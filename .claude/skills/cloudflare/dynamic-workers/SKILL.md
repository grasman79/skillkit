---
name: cloudflare-dynamic-workers
description: "Cloudflare Dynamic Workers - spawn isolated Workers at runtime from code strings using the Worker Loader binding. Covers LOADER.load() vs LOADER.get(), WorkerCode config object, module types, wrangler.toml setup, network/egress control, custom bindings, observability via Tail Workers, Durable Object Facets, Dynamic Workflows, runtime bundling with @cloudflare/worker-bundler, and pricing. Trigger words - dynamic workers, worker loader, LOADER binding, env.LOADER, load worker at runtime, dynamic worker, spawn worker, runtime code execution, user code execution, sandboxed worker, dynamic code, multi-tenant workers, per-tenant worker"
---

# Cloudflare Dynamic Workers

Reference for spawning isolated Workers at runtime using the Worker Loader binding (`env.LOADER`). The lowest-level primitive for on-demand, sandboxed code execution on Cloudflare's edge.

## Source of Truth

Official documentation: https://developers.cloudflare.com/dynamic-workers/

Always check the official docs when skill content conflicts or when you need the latest details.

## When to Use This Skill

- Executing user-submitted or AI-generated code safely in isolation
- Multi-tenant platforms where each tenant runs their own worker logic
- AI agent "code mode" - letting an LLM write executable code instead of tool calls
- Preview/prototype environments with millisecond startup times
- Custom automations that run on-demand worker logic per request

## When NOT to Use This Skill

- Static deployment of a known worker - use standard Workers setup ([workers-core](../workers-core/SKILL.md))
- Stateful AI agents - use the Agents SDK ([agents](../agents/SKILL.md))
- Production Cloudflare Tunnel - use [tunnel](../tunnel/SKILL.md)

## Wrangler Configuration

Add a `worker_loaders` binding to `wrangler.toml`:

```toml
[[worker_loaders]]
binding = "LOADER"
```

Or in `wrangler.jsonc`:

```json
{
  "worker_loaders": [{ "binding": "LOADER" }]
}
```

The binding name (`LOADER`) is arbitrary - access it via `env.LOADER` in your Worker.

## Core API

### `env.LOADER.load(code)` - one-time execution

Creates a fresh Dynamic Worker. No caching - a new isolate spins up every call. Best for single-use scenarios like executing AI-generated tool calls or user-submitted snippets.

```ts
export default {
  async fetch(request: Request, env: Env) {
    const worker = env.LOADER.load({
      compatibilityDate: "2026-06-01",
      mainModule: "src/index.js",
      modules: {
        "src/index.js": `
          export default {
            fetch(request) {
              return new Response("Hello from a dynamic Worker");
            },
          };
        `,
      },
    });

    const entrypoint = worker.getEntrypoint();
    return entrypoint.fetch(request);
  },
};
```

### `env.LOADER.get(id, callback)` - cached execution

Loads or retrieves a cached Dynamic Worker by ID. The callback only runs on a cache miss. Best for multi-tenant apps or preview environments where the same worker receives multiple requests.

```ts
export default {
  async fetch(request: Request, env: Env) {
    const tenantId = new URL(request.url).hostname.split(".")[0];

    const worker = env.LOADER.get(tenantId, async () => {
      const code = await env.KV.get(`tenant:${tenantId}:code`);
      return {
        compatibilityDate: "2026-06-01",
        mainModule: "index.js",
        modules: { "index.js": code },
      };
    });

    const entrypoint = worker.getEntrypoint();
    return entrypoint.fetch(request);
  },
};
```

Cache reuse requires the same `id` across calls. Changing the `id` or the code creates a new isolate.

## WorkerCode Configuration Object

```ts
interface WorkerCode {
  compatibilityDate: string;           // Required - e.g. "2026-06-01"
  mainModule: string;                  // Required - entry point path
  modules: Record<string, ModuleValue>;// Required - virtual file system

  compatibilityFlags?: string[];       // e.g. ["python_workers"]
  allowExperimental?: boolean;
  globalOutbound?: ServiceStub | null; // Network access control
  env?: Record<string, ServiceStub>;  // Custom bindings for the dynamic worker
  tails?: ServiceStub[];              // Tail Workers for observability
  limits?: {
    cpuMs?: number;                   // Max CPU ms per request
    subRequests?: number;             // Max outbound fetch calls
  };
}
```

## Module Types

The `modules` object maps virtual file paths to code. Use the file extension or an explicit object:

| Extension | Object form | Type |
|-----------|-------------|------|
| `.js` | `{ js: string }` | ES module |
| `.cjs` | `{ cjs: string }` | CommonJS |
| `.py` | `{ py: string }` | Python (requires `"python_workers"` flag) |
| - | `{ text: string }` | Importable string |
| - | `{ data: ArrayBuffer }` | Binary data |
| - | `{ json: object }` | JSON module |

Multi-file example:

```ts
modules: {
  "index.js": `
    import { greet } from "./utils.js";
    export default { fetch: (req) => new Response(greet("world")) };
  `,
  "utils.js": `
    export function greet(name) { return "Hello, " + name; }
  `,
}
```

## Network / Egress Control

Control outbound network access from the dynamic worker via `globalOutbound`:

```ts
// Block ALL outbound fetch/connect calls
modules: { "index.js": code },
globalOutbound: null,

// Route through a custom interceptor (audit, filter, proxy)
globalOutbound: ctx.exports.GatewayClass(),

// Unset = inherits parent worker's network access (default)
```

## Custom Bindings

Pass host-worker resources (KV, R2, Durable Objects, custom classes) into the dynamic worker via `env`:

```ts
// In the host worker - define a custom binding
export class MyBinding extends WorkerEntrypoint {
  async getData() {
    return this.ctx.props.data;  // props passed at instantiation
  }
}

// Pass it to the dynamic worker
const worker = env.LOADER.load({
  compatibilityDate: "2026-06-01",
  mainModule: "index.js",
  modules: { "index.js": code },
  env: {
    MY_BINDING: ctx.exports.MyBinding({ props: { data: "secret" } }),
    KV_STORE: env.KV,  // pass through an existing binding
  },
});
```

Inside the dynamic worker, access via standard `env`:

```ts
// Dynamic worker code
export default {
  async fetch(request, env) {
    const data = await env.MY_BINDING.getData();
    return new Response(data);
  },
};
```

## Observability (Tail Workers)

Capture logs and execution events from dynamic workers by attaching Tail Workers:

```ts
// Tail Worker - defined in host worker file
export class TailWorker extends WorkerEntrypoint {
  async tail(events) {
    for (const event of events) {
      // event.logs contains console.log output
      console.log("Dynamic worker logs:", event.logs);
    }
  }
}

// Attach when loading
const worker = env.LOADER.load({
  compatibilityDate: "2026-06-01",
  mainModule: "index.js",
  modules: { "index.js": code },
  tails: [ctx.exports.TailWorker({ props: { workerId: "my-worker" } })],
});
```

Tail Workers capture platform-level events (every `console.log`). This is different from app-level logging that routes only through your own logger.

Enable observability in wrangler.toml for the host worker:

```toml
[observability]
enabled = true
```

## Durable Object Facets

Run dynamically-loaded code as a Durable Object with isolated SQLite storage. Access via `this.ctx.facets` inside a Durable Object class:

```ts
export class MyDurableObject extends DurableObject {
  async fetch(request: Request) {
    const code = await this.env.KV.get("tenant-code");
    const worker = this.env.LOADER.load({
      compatibilityDate: "2026-06-01",
      mainModule: "index.js",
      modules: { "index.js": code },
    });

    // Create a facet - isolated DO instance for this tenant
    const facet = this.ctx.facets.get("tenant-logic", async () => ({
      class: worker.getDurableObjectClass("TenantHandler"),
    }));

    return facet.fetch(request);
  }
}
```

Facet lifecycle methods:
- `this.ctx.facets.get(name, callback)` - create or resume facet
- `this.ctx.facets.abort(name, reason)` - shut down facet
- `this.ctx.facets.delete(name)` - permanently delete facet and its storage

## Dynamic Workflows

Run different Workflow logic per tenant using `@cloudflare/dynamic-workflows`. Install it first:

```bash
npm i @cloudflare/dynamic-workflows
```

Configure in `wrangler.toml`:

```toml
[[worker_loaders]]
binding = "LOADER"

[[workflows]]
name = "my-workflow"
binding = "WORKFLOWS"
class_name = "DynamicWorkflowBinding"
```

Host worker:

```ts
import { wrapWorkflowBinding, DynamicWorkflowBinding } from "@cloudflare/dynamic-workflows";
export { DynamicWorkflowBinding };  // must be exported

export default {
  async fetch(request: Request, env: Env) {
    const tenantId = "tenant-123";
    const code = await env.KV.get(`workflow:${tenantId}`);

    const worker = env.LOADER.load({
      compatibilityDate: "2026-06-01",
      mainModule: "index.js",
      modules: { "index.js": code },
      env: {
        // Wrap the workflow binding with tenant context
        // Note: metadata must NOT contain secrets (it is persisted publicly)
        WORKFLOWS: wrapWorkflowBinding(env.WORKFLOWS, { tenantId }),
      },
    });

    const entrypoint = worker.getEntrypoint();
    return entrypoint.fetch(request);
  },
};
```

Dynamic worker implements `TenantWorkflow`:

```ts
// Tenant's workflow code (stored in KV)
import { WorkflowEntrypoint } from "cloudflare:workers";

export class TenantWorkflow extends WorkflowEntrypoint {
  async run(event, step) {
    const result = await step.do("process", async () => {
      return { processed: true };
    });
    return result;
  }
}
```

## Runtime Bundling

Bundle TypeScript or npm dependencies at runtime using `@cloudflare/worker-bundler`:

```ts
import { createWorker } from "@cloudflare/worker-bundler";

// Bundle TypeScript + npm deps from strings at request time
const { mainModule, modules } = await createWorker({
  files: {
    "src/index.ts": userCode,         // TypeScript is compiled
    "package.json": JSON.stringify({
      dependencies: { "some-package": "^1.0.0" },
    }),
  },
});

const worker = env.LOADER.load({
  compatibilityDate: "2026-06-01",
  mainModule,
  modules,
});
```

## Helper Packages

| Package | Use case |
|---------|----------|
| `@cloudflare/codemode` | `DynamicWorkerExecutor` - consolidates AI tool calls into single code execution runs |
| `@cloudflare/worker-bundler` | Bundle TypeScript + npm deps at runtime before loading |
| `@cloudflare/dynamic-workflows` | Per-tenant/per-agent durable workflow orchestration |

## Pricing

Billing started May 26, 2026. Dynamic Worker requests/CPU reuse standard Workers rates.

| Metric | Included (monthly) | Overage |
|--------|-------------------|---------|
| Unique Dynamic Workers | 1,000 | $0.002 per worker/day |
| Requests | 10M | $0.30 per million |
| CPU time | 30M ms | $0.02 per million ms |

A worker counts as unique if its ID or code changes. Multiple requests to the same cached worker (`get()`) count as one creation.

Unlike standard Workers, billing includes both isolate startup time and execution time (but not I/O wait).

## Key Constraints

- Python workers have substantially slower startup than JavaScript - avoid for latency-sensitive paths
- `globalOutbound: null` blocks all outbound fetch/connect from the dynamic worker, including to Cloudflare services
- Metadata in `wrapWorkflowBinding` must not contain secrets - it is persisted publicly
- `get()` cache reuse requires a consistent `id` - changing the ID creates a new isolate even for identical code
- Dynamic Workers are isolated by default - they have no external access unless you pass bindings via `env`
