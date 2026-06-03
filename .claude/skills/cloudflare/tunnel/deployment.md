# Cloudflare Tunnel - Deployment

Docker, Kubernetes, system services, replicas, and firewall configuration.

## Docker

### Remotely-Managed (Token-Based)

```bash
docker run -d \
  --name cloudflared \
  --restart unless-stopped \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate run --token <TUNNEL_TOKEN>
```

### Docker Compose

```yaml
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    command: tunnel --no-autoupdate run --token ${TUNNEL_TOKEN}
    environment:
      - TUNNEL_TOKEN=${TUNNEL_TOKEN}
```

### With Metrics Exposed

```yaml
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    command: tunnel --no-autoupdate --metrics 0.0.0.0:2000 run --token ${TUNNEL_TOKEN}
    ports:
      - "2000:2000"  # Prometheus metrics
```

### Docker Networking

- Use `host.docker.internal` or the Docker network gateway IP to reach host services
- For other containers, use container name or Docker network alias
- In Compose, services reach each other by service name

```yaml
services:
  app:
    image: my-app:latest
    expose:
      - "8000"

  cloudflared:
    image: cloudflare/cloudflared:latest
    command: tunnel --no-autoupdate run --token ${TUNNEL_TOKEN}
    depends_on:
      - app
    # In dashboard, set service URL to http://app:8000
```

## Kubernetes

### Architecture

Deploy cloudflared as a **separate Deployment** from your application. Each pod can access all cluster services via service discovery.

**Do not autoscale cloudflared** - downscaling breaks active connections.

### Step 1: Store Token

```yaml
# tunnel-token.yaml
apiVersion: v1
kind: Secret
metadata:
  name: tunnel-token
stringData:
  token: <YOUR_TUNNEL_TOKEN>
```

```bash
kubectl apply -f tunnel-token.yaml
```

### Step 2: Deploy cloudflared

```yaml
# cloudflared-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloudflared
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cloudflared
  template:
    metadata:
      labels:
        app: cloudflared
    spec:
      securityContext:
        sysctls:
          - name: net.ipv4.ping_group_range
            value: "65532 65532"
      containers:
        - name: cloudflared
          image: cloudflare/cloudflared:latest
          env:
            - name: TUNNEL_TOKEN
              valueFrom:
                secretKeyRef:
                  name: tunnel-token
                  key: token
          command:
            - cloudflared
            - tunnel
            - --no-autoupdate
            - --loglevel
            - info
            - --metrics
            - 0.0.0.0:2000
            - run
          livenessProbe:
            httpGet:
              path: /ready
              port: 2000
            failureThreshold: 1
            initialDelaySeconds: 10
            periodSeconds: 10
```

```bash
kubectl apply -f cloudflared-deployment.yaml
```

### Step 3: Configure Routes

In the Cloudflare dashboard, use Kubernetes service names as service URLs:

- `http://my-service` or `http://my-service.default.svc.cluster.local`

### Key Details

- `securityContext.sysctls` enables ICMP for diagnostics
- Liveness probe hits `/ready` on the metrics port
- `--no-autoupdate` required in containers - update via image tag
- `--metrics 0.0.0.0:2000` binds to all interfaces (required in containers)
- 2 replicas = HA; each opens 4 connections to 2+ data centers

## System Service

### Linux (systemd)

```bash
sudo cloudflared service install
```

Creates a systemd unit. Tunnel runs on boot.

```bash
sudo systemctl start cloudflared
sudo systemctl stop cloudflared
sudo systemctl restart cloudflared
sudo systemctl status cloudflared
journalctl -u cloudflared -f
```

For remotely-managed tunnels, the dashboard install command handles service setup automatically.

### macOS (launchd)

```bash
sudo cloudflared service install
```

### Windows

```powershell
cloudflared.exe service install
```

### Uninstall

```bash
sudo cloudflared service uninstall
```

**Only one cloudflared service per machine.** If you see `"cloudflared service is already installed"`, either add routes to the existing tunnel or uninstall first.

## Replicas (High Availability)

Multiple cloudflared instances running the **same tunnel** on different hosts.

### How They Work

- All replicas serve the same tunnel
- If one host goes down, remaining replicas continue
- Cloudflare routes to the geographically closest replica
- **Replicas do NOT load-balance** - they're for failover only

### Use Cases

- Redundancy across hosts or availability zones
- Zero-downtime config updates (update one replica at a time)
- Surviving individual host failures

### Deploy

**Remotely-managed:** Run the same `cloudflared service install <TOKEN>` on multiple hosts.

**Locally-managed:** Run `cloudflared tunnel run <NAME>` on multiple hosts with the same tunnel UUID and credentials.

**Kubernetes:** Set `replicas: 2+` in the Deployment spec.

### Verify

```bash
cloudflared tunnel info <NAME-or-UUID>
```

### Remove a Specific Replica

```bash
cloudflared tunnel cleanup --connector-id <CONNECTOR-ID> <NAME-or-UUID>
```

## Firewall Configuration

cloudflared requires **outbound** access only. No inbound ports needed.

### Required: Port 7844 (TCP + UDP)

#### Global Region IPs

**region1.v2.argotunnel.com:**
- IPv4: 198.41.192.7, .27, .37, .47, .57, .67, .77, .107, .167, .227
- IPv6: 2606:4700:a0::1 through ::10

**region2.v2.argotunnel.com:**
- IPv4: 198.41.200.13, .23, .33, .43, .53, .63, .73, .113, .193, .233
- IPv6: 2606:4700:a8::1 through ::10

#### US Region IPs (`--region us`)

**us-region1:** 198.41.218.1-10, IPv6: 2606:4700:a1::1-10
**us-region2:** 198.41.219.1-10, IPv6: 2606:4700:a9::1-10

#### FedRAMP High IPs

**fed-region1:** 162.159.234.1-10, IPv6: 2a06:98c1:4d::1-10
**fed-region2:** 172.64.234.1-10, IPv6: 2606:4700:f6::1-10

### SNI-Based Firewall Rules (Port 7844)

| Hostname | Protocol |
|----------|----------|
| `_v2-origintunneld._tcp.argotunnel.com` | TCP (http2) |
| `cftunnel.com` | TCP + UDP |
| `h2.cftunnel.com` | TCP (http2) |
| `quic.cftunnel.com` | UDP (quic) |

### Optional: Port 443 (HTTPS)

| Hostname | Purpose |
|----------|---------|
| `api.cloudflare.com` | API calls |
| `update.argotunnel.com` | Auto-updates |
| `github.com` | Source-based updates |
| `<team-name>.cloudflareaccess.com` | Access auth |
| `pqtunnels.cloudflareresearch.com` | Post-quantum setup |

### Minimal iptables Example

```bash
# Allow outbound to Cloudflare
iptables -A OUTPUT -p tcp --dport 7844 -j ACCEPT
iptables -A OUTPUT -p udp --dport 7844 -j ACCEPT

# Block all inbound (origin protection)
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -j DROP
```
