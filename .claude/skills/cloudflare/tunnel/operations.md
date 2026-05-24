# Cloudflare Tunnel - Operations

CLI commands, monitoring, metrics, notifications, and troubleshooting.

## CLI Command Reference

### cloudflared Management

| Command | Purpose |
|---------|---------|
| `cloudflared update` | Check for and install updates |
| `cloudflared version` | Print version and build date |
| `cloudflared help` | List all commands |

### Tunnel Management

| Command | Purpose |
|---------|---------|
| `cloudflared tunnel login` | Authenticate (generates `cert.pem`) |
| `cloudflared tunnel create <NAME>` | Create tunnel and credentials |
| `cloudflared tunnel list` | List active tunnels. `-d` includes deleted |
| `cloudflared tunnel info <NAME/UUID>` | Show connectors and connections |
| `cloudflared tunnel run <NAME/UUID>` | Run the tunnel |
| `cloudflared tunnel --config /path/config.yml run <NAME/UUID>` | Run with custom config |
| `cloudflared tunnel delete <NAME/UUID>` | Delete tunnel. `-f` for force |
| `cloudflared tunnel cleanup <NAME/UUID>` | Delete stale connections |
| `cloudflared tunnel cleanup --connector-id <ID> <NAME/UUID>` | Remove specific replica |
| `cloudflared tail <UUID>` | Livestream tunnel logs |

### Published Applications

| Command | Purpose |
|---------|---------|
| `cloudflared tunnel route dns <NAME/UUID> <hostname>` | Create CNAME to tunnel |
| `cloudflared tunnel route lb <NAME/UUID> <hostname> <pool>` | Add to load balancer pool |

### Private Networks

| Command | Purpose |
|---------|---------|
| `cloudflared tunnel route ip add <IP/CIDR> <NAME/UUID>` | Add network route |
| `cloudflared tunnel route ip show` | Show routing table |
| `cloudflared tunnel route ip delete <IP/CIDR>` | Remove route |
| `cloudflared tunnel route ip get <IP/CIDR>` | Check which route handles an IP |

### Virtual Networks

| Command | Purpose |
|---------|---------|
| `cloudflared tunnel vnet add <NAME>` | Create virtual network. `-d` for default |
| `cloudflared tunnel vnet delete <NAME/UUID>` | Delete virtual network |
| `cloudflared tunnel vnet list` | List all virtual networks |

### Config Validation

| Command | Purpose |
|---------|---------|
| `cloudflared tunnel ingress validate` | Validate ingress rules |
| `cloudflared tunnel ingress rule <URL>` | Test which rule matches a URL |

## Monitoring

### Prometheus Metrics

**Default:** `127.0.0.1:<PORT>/metrics` where PORT is first available in 20241-20245.
**In containers:** defaults to `0.0.0.0:<PORT>/metrics`.

Set a fixed address:

```bash
cloudflared tunnel --metrics 127.0.0.1:60123 run my-tunnel
```

Verify:

```bash
curl http://localhost:60123/metrics
```

### Key Metrics

**Tunnel Health:**

| Metric | Type | Measures |
|--------|------|----------|
| `cloudflared_tunnel_ha_connections` | Gauge | Active HA connections (expect 4 per connector) |
| `cloudflared_tunnel_concurrent_requests_per_tunnel` | Gauge | In-flight requests |
| `cloudflared_tunnel_total_requests` | Counter | Total requests served |
| `cloudflared_tunnel_request_errors` | Counter | Failed origin proxy attempts |
| `cloudflared_tunnel_timer_retries` | Gauge | Unacknowledged heartbeats |
| `cloudflared_tunnel_server_locations` | Gauge | Edge data center locations |

**Sessions:**

| Metric | Type | Measures |
|--------|------|----------|
| `cloudflared_tcp_active_sessions` | Gauge | Concurrent TCP sessions |
| `cloudflared_udp_active_sessions` | Gauge | Concurrent UDP sessions |

**QUIC:**

| Metric | Type | Measures |
|--------|------|----------|
| `quic_client_latest_rtt` | - | Most recent RTT |
| `quic_client_smoothed_rtt` | - | Smoothed RTT average |
| `quic_client_min_rtt` | - | Minimum observed RTT |
| `quic_client_lost_packets` | - | Lost packet count |

**System:** `build_info` (version/build details) plus standard Prometheus process and Go runtime metrics.

### Health Check Endpoint

```bash
curl http://localhost:20241/ready
```

Use for Kubernetes liveness probes or external health monitoring.

### Notifications

Available on all Zero Trust plans. Set up in the Cloudflare dashboard.

| Type | Triggers on |
|------|-------------|
| **Tunnel Creation or Deletion** | Tunnel created or deleted |
| **Tunnel Health Alert** | Status changes: Healthy, Degraded, Down, Inactive |

Delivery: email, webhook, third-party integrations.

### Log Streaming

```bash
cloudflared tail <UUID>                                      # Livestream
cloudflared tunnel --logfile /var/log/cloudflared.log run my-tunnel  # File
cloudflared tunnel --loglevel debug run my-tunnel             # Verbose
```

**Warning:** `debug` logs request URLs, headers, and content length. May expose sensitive data.

## Troubleshooting

### Tunnel Status

| Status | Meaning | Action |
|--------|---------|--------|
| **Healthy** | 4 connections active | None |
| **Degraded** | Some connections failed | Check logs, firewall rules |
| **Down** | Was connected, disconnected | Verify process is running |
| **Inactive** | Created, never connected | Run the tunnel |

**Tunnel status only shows cloudflared-to-Cloudflare connectivity.** A "Healthy" tunnel can still 502 if the origin is down.

### 502 Bad Gateway

Tunnel connected to Cloudflare, but cloudflared can't reach your origin.

**Origin not running:**

```
dial tcp [::1]:8080: connect: connection refused
```

Verify service is listening:

```bash
# Linux
ss -tlnp | grep <PORT>
# macOS
lsof -iTCP -sTCP:LISTEN | grep <PORT>
# Test
curl -v http://localhost:8080
```

**Wrong protocol:**

```
net/http: HTTP/1.x transport connection broken: malformed HTTP response
```

Origin expects HTTPS but route uses `http://`, or vice versa. Update the service URL.

**Wrong port:** Verify config port matches what the app binds to.

**TLS cert mismatch:**

```
x509: certificate is valid for example.com, not localhost
```

Fix (in preference order):
1. `originServerName: app.example.com`
2. `caPool: /path/to/ca-cert.pem`
3. `noTLSVerify: true` (not for production)

### Common Errors

**"cloudflared service is already installed"**
One service per machine. Add routes to the existing tunnel or `sudo cloudflared service uninstall` first.

**"An A, AAAA, or CNAME record with that host already exists"**
Choose a different hostname or delete the existing DNS record.

**Credentials file does not exist**
The `credentials-file` path in `config.yml` is wrong. Check `/root/` vs your actual home directory.

**Tunnel fails to authenticate**
Permission changes rotated the API key but `cert.pem` has the old one. Run `cloudflared tunnel login` again.

**x509: certificate signed by unknown authority**
Self-signed cert or SSL inspection proxy. Add CA to system trust store, use `--origin-ca-pool`, or `--no-tls-verify`.

**Error 1033**
No healthy connector. Check tunnel status, ensure cloudflared is running.

### SSL/TLS Issues

**ERR_SSL_VERSION_OR_CIPHER_MISMATCH**
Multi-level subdomain (e.g., `a.b.example.com`) needs an Advanced Certificate. Universal SSL covers one level only.

**ERR_TOO_MANY_REDIRECTS with Access app**
Set `originServerName` to the hostname on the origin certificate.

### WebSocket Issues

**"websocket: bad handshake"** - check in order:
1. Tunnel running and connected?
2. WebSockets enabled in Cloudflare dashboard?
3. SSL/TLS mode not "Off"?
4. Super Bot Fight Mode not blocking automated requests?
5. Binding Cookie enabled on SSH/RDP Access app? (Disable it)
6. Workers route overlapping tunnel hostname?

### Network Issues

**Streaming responses buffered:** Origin must include `Content-Type: text/event-stream` header.

**Random disconnects:** Normal during maintenance. Long-lived connections last up to 8 hours. For idle connections, configure app-layer keepalives or test `--protocol http2`.

**ping/traceroute don't work:** ICMP not proxied by default. Configure ICMP proxy separately.

**"network inside existing subnet '100.96.0.0/12'":** Your CIDR overlaps with WARP client's CGNAT range. Change your private network range.

### System Resources

**"too many open files":** Increase `ulimit` on the host.

**"failed to increase receive buffer size":** QUIC library limitation. Safe to ignore. Optional: increase buffer in `/etc/sysctl.d/`.
