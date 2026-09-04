# Angular

<div class="language-support-tag">
  <span class="lst-supported">AG-UI client</span>
</div>

This guide uses CopilotKit, one of the AG-UI clients listed in the
[Frontends overview](../index.md). The backend setup on the [AG-UI](index.md)
page is the same for every client.

Use CopilotKit Angular when an Angular application should connect to an ADK
agent through AG-UI. CopilotKit owns the runtime connection and AG-UI client
behavior. Your Angular app talks to `/api/copilotkit`, not directly to the
Python `/ag-ui` endpoint.

## Install

Complete the [AG-UI runtime setup](index.md) first, then install the Angular
package:

```shell
npm install @copilotkit/angular
```

The Angular package declares peer dependencies on Angular and Angular CDK
`^22.0.0` and RxJS `^7.8.0`, which your app provides.

## Runtime endpoint

The CopilotKit Runtime route from the
[AG-UI setup](index.md#connect-a-client) registers the ADK endpoint once and
serves every CopilotKit client at `/api/copilotkit`.

## Provide CopilotKit

Configure CopilotKit in your Angular app config:

```ts title="app.config.ts"
import { ApplicationConfig } from "@angular/core";
import { provideCopilotKit } from "@copilotkit/angular";

export const appConfig: ApplicationConfig = {
  providers: [
    provideCopilotKit({
      runtimeUrl: "/api/copilotkit",
    }),
  ],
};
```

## Render chat

Use the packaged `copilot-chat` component for the default chat surface.

```ts title="chat.component.ts"
import { Component } from "@angular/core";
import { CopilotChat } from "@copilotkit/angular";

@Component({
  selector: "app-chat",
  standalone: true,
  imports: [CopilotChat],
  template: `
    <section style="height: 100vh">
      <copilot-chat [agentId]="'default'"></copilot-chat>
    </section>
  `,
})
export class ChatComponent {}
```

Use the lower-level Angular services and `injectAgentStore("default")` only
when you need a fully custom Angular chat surface. Keep the runtime bridge and
ADK middleware the same.
