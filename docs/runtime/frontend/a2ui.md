# A2UI

<div class="language-support-tag">
  <span class="lst-supported">Generative UI</span>
</div>

A2UI is a protocol for structured agent-generated UI. Instead of returning a
string that describes a card, table, form, or dashboard, the agent returns UI
operations that reference a component catalog. A renderer uses that catalog to
turn the operations into real application UI.

The [A2UI integration](/integrations/a2ui/) page covers the A2UI agent SDK for
agents that emit A2UI directly, for example over A2A. This page covers A2UI over
AG-UI: the `ag-ui-adk` adapter carries the operations in the AG-UI stream and an
A2UI renderer in the frontend draws the surface. The protocol boundary is still
A2UI: catalog, surface, component tree, data model, and user actions.

Use this page when the agent should render structured UI. If you only need chat,
start with [AG-UI](ag-ui/index.md) and a client page first.

## Protocol model

| Part | Owner | Purpose |
|---|---|---|
| Catalog | Application | Defines the component names, prop schemas, and renderers that are allowed. |
| Surface | Agent | Names one renderable UI surface, such as `sales-dashboard`. |
| Operations | Agent | Create a surface and update its component tree or data model. |
| Renderer | Frontend | Validates operations against the catalog and renders the surface. |
| Actions | Frontend and agent | Send user interaction events back through the same runtime boundary. |

An A2UI response is data. A small response can look like this:

```json
{
  "a2ui_operations": [
    {
      "version": "v0.9",
      "createSurface": {
        "surfaceId": "sales-dashboard",
        "catalogId": "https://example.com/catalogs/sales.json"
      }
    },
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "sales-dashboard",
        "components": [
          { "id": "root", "component": "Column", "children": ["revenue"] },
          {
            "id": "revenue",
            "component": "MetricCard",
            "title": { "path": "/title" },
            "value": { "path": "/value" }
          }
        ]
      }
    },
    {
      "version": "v0.9",
      "updateDataModel": {
        "surfaceId": "sales-dashboard",
        "path": "/",
        "value": { "title": "Revenue", "value": "$124K" }
      }
    }
  ]
}
```

The frontend does not need to parse arbitrary natural language to build UI. It
receives structured operations, checks them against the catalog, and renders the
matching components.

## How it fits with ADK

1. An ADK agent decides that a visual response is useful.
2. The A2UI tool path creates or updates a surface from a known catalog.
3. The AG-UI stream carries the structured UI activity to the frontend.
4. The renderer displays the surface and forwards user actions back through the
   same runtime boundary.

Keep business data, credentials, and storage policy in ADK tools or backend
services. Keep component rendering and browser interaction in the frontend.

## Using A2UI over AG-UI

The example below uses CopilotKit, which ships an A2UI renderer and forwards the
catalog with each run. You provide the catalog on the client and configure the
ADK AG-UI wrapper for dynamic A2UI.

Install the renderer package alongside the React client. The renderer's catalog
types are built on Zod 3, so pin that major version:

```shell
npm install @copilotkit/react-core @copilotkit/a2ui-renderer zod@3
```

Create a catalog with the components your app can render. Each entry pairs a
Zod schema with a description the agent reads, and `includeBasicCatalog` adds
the built-in layout components such as `Column`:

```tsx title="a2ui-catalog.tsx"
import { createCatalog, type RendererProps } from "@copilotkit/a2ui-renderer";
import { z } from "zod";

const definitions = {
  MetricCard: {
    description: "A single KPI with a title and a formatted value.",
    props: z.object({ title: z.string(), value: z.string() }),
  },
};

function MetricCard({ props }: RendererProps<{ title: string; value: string }>) {
  return (
    <article>
      <h3>{props.title}</h3>
      <strong>{props.value}</strong>
    </article>
  );
}

export const salesCatalog = createCatalog(
  definitions,
  { MetricCard },
  {
    catalogId: "https://example.com/catalogs/sales.json",
    includeBasicCatalog: true,
  },
);
```

Pass the catalog to the CopilotKit provider:

```tsx title="app/providers.tsx"
"use client";

import { CopilotKit } from "@copilotkit/react-core/v2";
import "@copilotkit/react-core/v2/styles.css";

import { salesCatalog } from "./a2ui-catalog";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <CopilotKit
      runtimeUrl="/api/copilotkit"
      useSingleEndpoint={false}
      a2ui={{ catalog: salesCatalog }}
    >
      {children}
    </CopilotKit>
  );
}
```

Configure the ADK wrapper with the same catalog id and composition guidance:

```python title="app.py"
from google.adk.agents import Agent
from ag_ui_adk import ADKAgent

root_agent = Agent(
    name="a2ui_assistant",
    model="gemini-pro-latest",
    instruction=(
        "When visual UI helps, create an A2UI surface using the available "
        "catalog. Do not repeat the rendered data as plain text."
    ),
)

ag_ui_agent = ADKAgent(
    adk_agent=root_agent,
    app_name="a2ui_app",
    user_id="local_user",
    session_timeout_seconds=3600,
    use_in_memory_services=True,
    a2ui={
        "default_catalog_id": "https://example.com/catalogs/sales.json",
        "guidelines": {
            "composition_guide": "Use MetricCard for KPIs.",
        },
    },
)
```

The rest of the FastAPI setup is the same as the [AG-UI overview](ag-ui/index.md).

## What to read next

- [Declarative (A2UI)](patterns/generative-ui/a2ui.md): CopilotKit catalog wiring and ADK wrapper setup.
- [Controlled](patterns/generative-ui/controlled-generative-ui.md): Render app-owned components from
  tool calls.
- [Tool Rendering](patterns/generative-ui/tool-rendering.md): Render progress and results for
  ordinary tools.
