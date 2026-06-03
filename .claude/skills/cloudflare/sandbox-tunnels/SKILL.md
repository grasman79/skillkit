---
name: cloudflare-sandbox-tunnels
description: "Cloudflare Sandbox SDK tunnels API - expose a service running inside a Cloudflare Sandbox container on a zero-config *.trycloudflare.com URL via sandbox.tunnels. Covers tunnels.get(), tunnels.list(), tunnels.destroy(), TunnelInfo type, transport requirements, image variant constraints, and all limitations (no SSE, no persistent hostname, DNS warm-up, WARP egress). Trigger words - sandbox tunnels, sandbox.tunnels, trycloudflare sandbox, expose sandbox port, sandbox url, sandbox service url, cloudflare sandbox sdk tunnel"
---

# Cloudflare Sandbox Tunnels

Reference for the `sandbox.tunnels` namespace in the `@cloudflare/sandbox` SDK. Exposes a service running inside a sandbox container on a public `*.trycloudflare.com` URL - no Cloudflare account, DNS record, or custom domain required.

## Source of Truth

Official documentation: https://developers.cloudflare.com/sandbox/api/tunnels/

Always check the official docs when skill content conflicts or when you need the latest details.

## When to Use This Skill

- Exposing a port inside a Cloudflare Sandbox container for quick development or testing
- Deploying to `.workers.dev` (wildcard DNS unavailable - tunnels work here, `exposePort()` does not)
- Listing or cleaning up active tunnels on a sandbox
- Debugging why a sandbox tunnel isn't reachable

## When NOT to Use This Skill

- Production-grade stable URLs - use `exposePort()` with a custom domain instead (see [Ports API](https://developers.cloudflare.com/sandbox/api/ports/))
- Exposing a local dev machine (use [local-dev-tunnels](../local-dev-tunnels/SKILL.md))
- Production cloudflared daemon setup (use [tunnel](../tunnel/SKILL.md))

## Requirements

**RPC transport required.** `sandbox.tunnels` throws `"RPC transport required"` on HTTP/WebSocket transports. Configure RPC transport first: https://developers.cloudflare.com/sandbox/configuration/transport/

**glibc image only.** `cloudflared` ships with the `default`, `python`, `opencode`, and `desktop` image variants. The `musl`/Alpine variant does not include `cloudflared` - no upstream build exists for musl.

## API

### `tunnels.get(port)`

Expose a port and get its public URL. Idempotent - repeated calls for the same port return the same record.

```ts
const tunnel = await sandbox.tunnels.get(port: number): Promise<TunnelInfo>
```

- `port` - Port inside the sandbox (1024-65535, excluding reserved ports). The service must already be listening on `0.0.0.0:<port>` inside the container.

```ts
import { getSandbox } from "@cloudflare/sandbox";
export { Sandbox } from "@cloudflare/sandbox";

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const sandbox = getSandbox(env.Sandbox, "my-sandbox");

    await sandbox.startProcess("python -m http.server 8080");

    const tunnel = await sandbox.tunnels.get(8080);
    console.log(tunnel.url); // https://random-words-here.trycloudflare.com

    // Same record on repeated calls
    const same = await sandbox.tunnels.get(8080);
    console.log(same.url === tunnel.url); // true

    return Response.json({ url: tunnel.url });
  },
};
```

### `tunnels.list()`

Return all tunnels currently tracked for this sandbox. Returns an empty array when no tunnels are active.

```ts
const tunnels = await sandbox.tunnels.list(): Promise<TunnelInfo[]>
```

```ts
const tunnels = await sandbox.tunnels.list();
for (const tunnel of tunnels) {
  console.log(`port ${tunnel.port} → ${tunnel.url}`);
}
```

### `tunnels.destroy(portOrInfo)`

Tear down a tunnel. Accepts either the port number or the `TunnelInfo` record. Idempotent - destroying an unknown port resolves successfully.

```ts
await sandbox.tunnels.destroy(portOrInfo: number | TunnelInfo): Promise<void>
```

```ts
const tunnel = await sandbox.tunnels.get(8080);

await sandbox.tunnels.destroy(8080);    // by port number
await sandbox.tunnels.destroy(tunnel);  // or by record
```

## TunnelInfo Type

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | SDK-assigned identifier (e.g. `quick-9f2c8a1d`) |
| `port` | number | Port inside the sandbox the tunnel proxies to |
| `url` | string | Public URL - `https://<random-words>.trycloudflare.com` |
| `hostname` | string | Hostname component of `url` |
| `createdAt` | string | ISO-8601 creation timestamp |

```ts
interface TunnelInfo {
  id: string;
  port: number;
  url: string;
  hostname: string;
  createdAt: string;
}
```

## Limitations

| Limitation | Detail |
|-----------|--------|
| **No persistent hostname** | Every container restart picks a new `<random-words>.trycloudflare.com`. The SDK clears its tunnel cache on restart - next `tunnels.get()` returns a fresh record. |
| **No uptime guarantee** | Cloudflare positions `trycloudflare.com` as a debug aid. Use `exposePort()` with a custom domain for production. |
| **No Server-Sent Events** | The edge buffers `text/event-stream` responses - SSE does not reach the client. WebSockets work normally. |
| **DNS warm-up** | First request through a brand-new URL can take a couple of seconds while DNS propagates, even after `get()` resolves. |
| **WARP / Zero Trust egress** | If the local machine runs Cloudflare WARP or another Zero Trust egress policy, traffic to `api.trycloudflare.com` and the cloudflared edge can be blocked. `tunnels.get()` hangs on the edge handshake and eventually times out. Disable WARP or add an egress exception. |
| **No musl/Alpine** | The `musl` image variant does not include `cloudflared`. Use `default`, `python`, `opencode`, or `desktop`. |

## Related Resources

- [Ports API](https://developers.cloudflare.com/sandbox/api/ports/) - `exposePort()` for production-grade stable URLs with custom domains
- [Preview URLs concept](https://developers.cloudflare.com/sandbox/concepts/preview-urls/) - Worker-fronted preview URLs vs. quick tunnels
- [Expose services guide](https://developers.cloudflare.com/sandbox/guides/expose-services/) - End-to-end walkthrough
- [Transport configuration](https://developers.cloudflare.com/sandbox/configuration/transport/) - RPC vs. route-based transport
