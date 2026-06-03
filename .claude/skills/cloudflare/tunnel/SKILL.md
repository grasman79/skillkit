---
name: cloudflare-tunnel
description: "Production Cloudflare Tunnel (cloudflared) - secure outbound-only connections from origin servers to Cloudflare's network. Covers cloudflared installation, dashboard (remote) and CLI (local) tunnel creation, config.yml with ingress rules and origin parameters, run parameters, Docker and Kubernetes deployment, system service installation, replicas for high availability, firewall rules (port 7844), Prometheus metrics, notifications, and troubleshooting (502, TLS, WebSocket errors). For local dev tunnels with Wrangler/Vite, use cloudflare/local-dev-tunnels instead. Trigger words - cloudflare tunnel, cloudflared, tunnel ingress, tunnel config, expose service, zero trust tunnel, private network tunnel, tunnel docker, tunnel kubernetes, tunnel replica, tunnel firewall, tunnel metrics, tunnel troubleshoot, argo tunnel, cfargotunnel"
---

# Cloudflare Tunnel (cloudflared)

Production reference for Cloudflare Tunnel - the `cloudflared` daemon that creates secure, outbound-only connections from your origin server to Cloudflare's global network.

## Source of Truth

Official documentation: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/

Always check the official docs when the skill content conflicts or when you need the latest details.

## When to Use This Skill

- Exposing a production service (web app, API, SSH, RDP) through Cloudflare without opening inbound ports
- Setting up `cloudflared` daemon for the first time (quick tunnel, dashboard, or CLI)
- Configuring tunnel routing with ingress rules, origin parameters, or run flags
- Deploying cloudflared in Docker, Kubernetes, or as a system service
- Setting up replicas for high availability
- Configuring firewall rules for tunnel connectivity
- Monitoring tunnels with Prometheus metrics or notifications
- Troubleshooting tunnel connectivity, 502 errors, or TLS certificate issues

## When NOT to Use This Skill

- Exposing a local dev server temporarily (use [local-dev-tunnels](../local-dev-tunnels/SKILL.md) for Wrangler/Vite integration)
- Deploying a Worker or Pages project (use [workers-core](../workers-core/SKILL.md))
- Configuring wrangler, bindings, or secrets (use [workers-core](../workers-core/SKILL.md))

## Reference Files

| File | When to read |
|------|--------------|
| [setup.md](setup.md) | Installation, quick tunnels, dashboard tunnel creation, CLI tunnel creation, permissions, terminology |
| [configuration.md](configuration.md) | config.yml structure, ingress rules, origin parameters, all run parameters with env vars |
| [deployment.md](deployment.md) | Docker, Kubernetes manifests, system service, replicas for HA, firewall IP tables |
| [operations.md](operations.md) | CLI commands, Prometheus metrics, notifications, troubleshooting common errors |

## How It Works

```
User -> Cloudflare Edge -> cloudflared -> Your Origin Service
```

- **No public IP needed** - cloudflared initiates all connections outbound
- **No inbound firewall ports** - block all inbound, allow only outbound to Cloudflare on port 7844
- **Supports HTTP, HTTPS, SSH, RDP, TCP, Unix sockets**
- **Built-in redundancy** - each connector opens 4 connections to 2+ data centers

## Two Management Modes

| Mode | Created via | Config stored | Best for |
|------|-------------|---------------|----------|
| **Remotely-managed** | Cloudflare dashboard | Cloudflare (dashboard/API) | Production, teams, GitOps |
| **Locally-managed** | `cloudflared tunnel create` CLI | Local `config.yml` | Dev, testing, legacy setups |

## Key Gotchas

1. **Port 7844 must be open outbound** (TCP + UDP). This is the only network requirement. See [deployment.md](deployment.md).

2. **Quick tunnels are for testing only** - no SLA, 200 concurrent request limit, no custom domains. See [setup.md](setup.md).

3. **Only one cloudflared service per machine.** Add routes to the existing tunnel or use ingress rules. See [configuration.md](configuration.md).

4. **Replicas don't load-balance** - traffic goes to the geographically closest replica. They're for HA failover only. See [deployment.md](deployment.md).

5. **Don't autoscale cloudflared** - downscaling breaks active connections. See [deployment.md](deployment.md).

6. **Tunnel status only shows cloudflared-to-Cloudflare connectivity** - it does NOT tell you if cloudflared can reach your origin service. A "Healthy" tunnel can still produce 502s if the origin is down.
