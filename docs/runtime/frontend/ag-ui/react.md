# React

<div class="language-support-tag">
  <span class="lst-supported">AG-UI client</span>
</div>

This guide uses CopilotKit, one of the AG-UI clients listed in the
[Frontends overview](../index.md). The backend setup on the [AG-UI](index.md)
page is the same for every client.

Use CopilotKit React when a browser UI should connect to an ADK agent through
AG-UI. CopilotKit owns the client-side protocol handling, streaming messages,
tool calls, state updates, and chat UI. Your React code talks to
`/api/copilotkit`, not directly to the Python `/ag-ui` endpoint.

## Install

Complete the [AG-UI runtime setup](index.md) first, then install the React
client package:

```shell
npm install @copilotkit/react-core zod
```

## Runtime endpoint

The CopilotKit Runtime route from the
[AG-UI setup](index.md#connect-a-client) registers the ADK endpoint once and
serves every CopilotKit client at `/api/copilotkit`.

## Add the provider

Mount `CopilotKit` once near the React root.

```tsx title="app/providers.tsx"
"use client";

import { CopilotKit } from "@copilotkit/react-core/v2";
import "@copilotkit/react-core/v2/styles.css";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <CopilotKit runtimeUrl="/api/copilotkit" useSingleEndpoint={false}>
      {children}
    </CopilotKit>
  );
}
```

Use the provider from your root layout:

```tsx title="app/layout.tsx"
import { Providers } from "./providers";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

## Render chat

Add `CopilotChat` anywhere below the provider:

```tsx title="app/page.tsx"
"use client";

import { CopilotChat } from "@copilotkit/react-core/v2";

export default function Page() {
  return (
    <main style={{ height: "100vh" }}>
      <CopilotChat agentId="default" />
    </main>
  );
}
```

The `CopilotChat` component manages message state, input state, streaming, tool calls,
attachments, and suggestions internally.

## Add browser tools

Use `useFrontendTool` when the ADK agent should call a browser-side capability.
The `AGUIToolset()` toolset on the ADK side exposes these tools to the agent.

```tsx title="SearchTool.tsx"
"use client";

import { useFrontendTool } from "@copilotkit/react-core/v2";
import { z } from "zod";

export function SearchTool() {
  useFrontendTool({
    name: "searchDocs",
    description: "Search the current application documentation.",
    parameters: z.object({
      query: z.string(),
    }),
    handler: async ({ query }, { signal }) => {
      const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`, {
        signal,
      });
      return response.text();
    },
  });

  return null;
}
```
