# Vue

<div class="language-support-tag">
  <span class="lst-supported">AG-UI client</span>
</div>

This guide uses CopilotKit, one of the AG-UI clients listed in the
[Frontends overview](../index.md). The backend setup on the [AG-UI](index.md)
page is the same for every client.

Use CopilotKit Vue when a Vue 3 application should connect to an ADK agent
through AG-UI. CopilotKit owns the runtime connection, streaming, and chat
state. Your Vue app talks to `/api/copilotkit`.

## Install

Complete the [AG-UI runtime setup](index.md) first, then install the Vue
package:

```shell
npm install @copilotkit/vue @copilotkit/core
```

## Runtime endpoint

The CopilotKit Runtime route from the
[AG-UI setup](index.md#connect-a-client) registers the ADK endpoint once and
serves every CopilotKit client at `/api/copilotkit`.

## Add the Vue client

Import the stylesheet once in your app entry:

```ts title="src/main.ts"
import { createApp } from "vue";
import App from "./App.vue";
import "@copilotkit/vue/styles.css";

createApp(App).mount("#app");
```

Wrap the assistant surface with `CopilotKitProvider` and render `CopilotChat`:

```vue title="src/App.vue"
<script setup lang="ts">
import { CopilotKitProvider, CopilotChat } from "@copilotkit/vue";
</script>

<template>
  <CopilotKitProvider runtime-url="/api/copilotkit">
    <main style="height: 100vh">
      <CopilotChat agent-id="default" />
    </main>
  </CopilotKitProvider>
</template>
```

Use Vue slots and composables when you need custom message, activity, or tool
rendering. Keep the CopilotKit runtime route and the ADK AG-UI backend the same.
