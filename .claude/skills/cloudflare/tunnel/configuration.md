# Cloudflare Tunnel - Configuration

Config file structure, ingress rules, origin parameters, and run parameters.

## Config File Structure (Locally-Managed)

Default location: `~/.cloudflared/config.yml`

### Top-Level Keys

```yaml
tunnel: <TUNNEL-UUID>
credentials-file: /path/to/<TUNNEL-UUID>.json

# For private networks
warp-routing:
  enabled: true

# Global origin settings (apply to all ingress rules unless overridden)
originRequest:
  connectTimeout: 30s
  noTLSVerify: false

# Routing rules
ingress:
  - hostname: app.example.com
    service: http://localhost:8000
  - service: http_status:404    # catch-all (required)
```

## Ingress Rules

Each rule has:

- `hostname` - domain to match (supports wildcards like `*.example.com`)
- `path` - regex pattern for URL path (optional)
- `service` - backend service URL
- `originRequest` - per-rule origin config overrides (optional)

### Supported Service Types

| Type | Example | Use case |
|------|---------|----------|
| HTTP | `http://localhost:8000` | Web apps, APIs |
| HTTPS | `https://localhost:8443` | TLS-terminated services |
| SSH | `ssh://localhost:22` | SSH access |
| RDP | `rdp://localhost:3389` | Remote desktop |
| TCP | `tcp://localhost:5432` | Raw TCP (databases, etc.) |
| Unix socket | `unix:/path/to/socket` | Socket-based services |
| Test server | `hello_world` | Built-in test page |
| Status code | `http_status:404` | Return fixed HTTP status |

### Example: Multiple Services

```yaml
tunnel: abc123
credentials-file: /root/.cloudflared/abc123.json

ingress:
  # Web app
  - hostname: app.example.com
    service: http://localhost:8000

  # API with custom timeout
  - hostname: api.example.com
    service: http://localhost:3000
    originRequest:
      connectTimeout: 10s

  # SSH
  - hostname: ssh.example.com
    service: ssh://localhost:22

  # Wildcard
  - hostname: "*.staging.example.com"
    service: http://localhost:9000

  # Path-based routing
  - hostname: example.com
    path: "^/api/.*$"
    service: http://localhost:3000

  - hostname: example.com
    service: http://localhost:8000

  # Catch-all (required - must be last)
  - service: http_status:404
```

### Validate and Test

```bash
# Validate config
cloudflared tunnel ingress validate

# Test which rule matches a URL
cloudflared tunnel ingress rule https://app.example.com/path
```

## Origin Parameters

Control how cloudflared connects to your origin. Set globally under `originRequest:` or per-ingress-rule.

### TLS Settings

| Parameter | Default | Description |
|-----------|---------|-------------|
| `originServerName` | `""` | Expected hostname from origin's TLS cert. If empty, uses service URL hostname |
| `matchSNItoHost` | `false` | Auto-set SNI to the incoming request's hostname |
| `caPool` | `""` | Path to CA certificate for origin (e.g., `/root/certs/ca.pem`) |
| `noTLSVerify` | `false` | Disable TLS verification. Not for production |
| `tlsTimeout` | `10s` | TLS handshake timeout |
| `http2Origin` | `false` | Connect to origin using HTTP/2 instead of HTTP/1.1 |

### HTTP Settings

| Parameter | Default | Description |
|-----------|---------|-------------|
| `httpHostHeader` | `""` | Override the HTTP Host header sent to origin |
| `disableChunkedEncoding` | `false` | Disable chunked encoding. Useful for WSGI servers |

### Connection Settings

| Parameter | Default | Description |
|-----------|---------|-------------|
| `connectTimeout` | `30s` | TCP connection timeout |
| `noHappyEyeballs` | `false` | Disable Happy Eyeballs algorithm |
| `proxyType` | `""` | Empty for regular proxy, `"socks"` for SOCKS5 |
| `proxyAddress` | `127.0.0.1` | Proxy listen address (locally-managed only) |
| `proxyPort` | `0` | Proxy port. `0` = random unused port (locally-managed only) |
| `keepAliveTimeout` | `1m30s` | Idle keepalive timeout |
| `keepAliveConnections` | `100` | Max idle keepalive connections to origin |
| `tcpKeepAlive` | `30s` | TCP keepalive packet interval |

### Access Settings

| Parameter | Default | Description |
|-----------|---------|-------------|
| `access` | `""` | Require cloudflared to validate Cloudflare Access JWT. Fields: `required`, `teamName`, `audTag` |

### Example: TLS Config

```yaml
tunnel: abc123
credentials-file: /root/.cloudflared/abc123.json

originRequest:
  originServerName: app.example.com
  caPool: /etc/ssl/certs/internal-ca.pem
  connectTimeout: 10s

ingress:
  - hostname: app.example.com
    service: https://localhost:8443
  - service: http_status:404
```

## Run Parameters

Flags for `cloudflared tunnel run`. Most also settable via environment variables.

### Connection and Protocol

| Flag | Default | Env Var | Description |
|------|---------|---------|-------------|
| `--protocol` | `auto` | `TUNNEL_TRANSPORT_PROTOCOL` | `auto`, `http2`, or `quic`. Auto = quic with http2 fallback |
| `--edge-ip-version` | `4` | `TUNNEL_EDGE_IP_VERSION` | `auto`, `4`, or `6` |
| `--edge-bind-address` | - | `TUNNEL_EDGE_BIND_ADDRESS` | Outgoing IP for Cloudflare connections (multi-NIC) |
| `--region` | (global) | `TUNNEL_REGION` | `us` for US-only. Omit for global |
| `--retries` | `5` | `TUNNEL_RETRIES` | Max retries. Exponential backoff: 1, 2, 4, 8, 16s |
| `--grace-period` | `30s` | `TUNNEL_GRACE_PERIOD` | Shutdown drain timeout after SIGINT/SIGTERM |
| `--post-quantum` | - | `TUNNEL_POST_QUANTUM` | Enforce post-quantum crypto (QUIC only, no fallback) |

### Logging and Metrics

| Flag | Default | Env Var | Description |
|------|---------|---------|-------------|
| `--loglevel` | `info` | `TUNNEL_LOGLEVEL` | `debug`, `info`, `warn`, `error`, `fatal` |
| `--logfile` | - | `TUNNEL_LOGFILE` | Write logs to file |
| `--metrics` | `127.0.0.1:20241-20245` | `TUNNEL_METRICS` | Prometheus metrics endpoint |
| `--pidfile` | - | `TUNNEL_PIDFILE` | Write PID after first connection |

### Updates and Config

| Flag | Default | Env Var | Description |
|------|---------|---------|-------------|
| `--no-autoupdate` | - | `NO_AUTOUPDATE` | Disable auto-updates |
| `--autoupdate-freq` | `24h` | - | Update check frequency |
| `--config` | `~/.cloudflared/config.yml` | - | Config file path (locally-managed only) |
| `--origincert` | `~/.cloudflared/cert.pem` | `TUNNEL_ORIGIN_CERT` | Account cert path (locally-managed only) |

### Authentication

| Flag | Default | Env Var | Description |
|------|---------|---------|-------------|
| `--token` | - | `TUNNEL_TOKEN` | Tunnel token (remotely-managed only) |
| `--token-file` | - | `TUNNEL_TOKEN_FILE` | Token file path (requires 2025.4.0+) |
| `--tag` | - | `TUNNEL_TAG` | Custom tags as `KEY=VAL`. Repeat for multiple |

### DNS

| Flag | Default | Env Var | Description |
|------|---------|---------|-------------|
| `--dns-resolver-addrs` | (system) | `TUNNEL_DNS_RESOLVER_ADDRS` | Custom DNS as `ip:port`. Max 10. Requires 2025.7.0+ |

### Example: Production Run

```bash
cloudflared tunnel \
  --protocol quic \
  --loglevel info \
  --logfile /var/log/cloudflared.log \
  --metrics 0.0.0.0:20241 \
  --no-autoupdate \
  --grace-period 60s \
  run my-tunnel
```
