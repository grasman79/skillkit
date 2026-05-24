# Cloudflare Tunnel - Setup

Everything to go from zero to a working tunnel. Installation, quick tunnels for testing, and both dashboard and CLI tunnel creation.

## Prerequisites

- Server must reach Cloudflare on **port 7844** (TCP + UDP)
- For published applications: a domain already added to Cloudflare
- For quick tunnels: just `cloudflared` installed (no account needed)

## Install cloudflared

### macOS

```bash
brew install cloudflared
```

### Debian / Ubuntu

```bash
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflared.list
sudo apt-get update && sudo apt-get install cloudflared
```

### RHEL / CentOS

```bash
curl -fsSL https://pkg.cloudflare.com/cloudflared-ascii.repo | sudo tee /etc/yum.repos.d/cloudflared.repo
sudo yum install cloudflared
```

### Arch Linux

```bash
pacman -Syu cloudflared
```

### Windows

Download from Cloudflare downloads page, rename to `cloudflared.exe`.

```powershell
.\cloudflared.exe --version
```

### From Source

```bash
git clone https://github.com/cloudflare/cloudflared.git
cd cloudflared
make cloudflared
go install github.com/cloudflare/cloudflared/cmd/cloudflared
```

### Docker

```bash
docker pull cloudflare/cloudflared:latest
```

### Verify

```bash
cloudflared --version
```

## Quick Tunnel (TryCloudflare)

Fastest way to test. Generates a random `trycloudflare.com` subdomain pointing to your local service. No account needed.

```bash
cloudflared tunnel --url http://localhost:8080
```

Outputs a public URL like `https://random-words.trycloudflare.com`.

### Limitations

- **Testing only** - no SLA, no uptime guarantee
- Max 200 concurrent requests (returns 429 above that)
- No Server-Sent Events (SSE) support
- No custom domains
- Won't work if `config.yaml` exists in `~/.cloudflared/` - rename it temporarily
- Requires `cloudflared` version 2020.5.1+

## Dashboard Tunnel (Remotely-Managed)

Best for production. Configuration stored in Cloudflare.

### Step 1: Create the Tunnel

1. Go to **Zero Trust** > **Networks** > **Connectors** > **Cloudflare Tunnels**
2. Click **Create a tunnel**
3. Select **Cloudflared** as the connector type
4. Enter a descriptive name (e.g., `production-api`, `staging-vpc`)
5. Save - you get an install command with a token

### Step 2: Install the Connector

Copy the command from the dashboard and run it on your server:

```bash
cloudflared service install <TUNNEL_TOKEN>
```

Make sure the environment selector matches your OS. After execution, the connector appears in the dashboard.

### Step 3a: Publish an Application

1. Go to the **Published applications** tab
2. Enter a subdomain and select your domain
3. Under **Service**, select type and URL:
   - Type: `HTTP`, `HTTPS`, `SSH`, `RDP`, `TCP`, etc.
   - URL: `localhost:8000` (or wherever your service runs)
4. Add optional settings (TLS, headers, access policies)
5. Save

Your app is now accessible at `https://subdomain.yourdomain.com`.

### Step 3b: Connect a Private Network

1. Go to the **CIDR** tab
2. Enter the private IP or CIDR range (e.g., `10.0.0.1` or `10.0.0.0/24`)
3. Click **Complete setup**

Users with the Cloudflare One Client (WARP) enrolled in your Zero Trust org can now reach those IPs.

### Step 4: Verify

Check tunnel status on **Networks** > **Connectors**:

| Status | Meaning |
|--------|---------|
| **Healthy** | Working - 4 connections to Cloudflare |
| **Degraded** | Running but some connections failed |
| **Down** | Was connected, now disconnected |
| **Inactive** | Created but never connected |

## CLI Tunnel (Locally-Managed)

Best for dev/testing or config-as-code workflows.

### Step 1: Authenticate

```bash
cloudflared tunnel login
```

Opens browser for Cloudflare login and zone selection. Generates `~/.cloudflared/cert.pem`.

### Step 2: Create the Tunnel

```bash
cloudflared tunnel create my-tunnel
```

Registers the tunnel, generates a UUID and credentials file at `~/.cloudflared/<UUID>.json`.

Verify:

```bash
cloudflared tunnel list
```

### Step 3: Create config.yml

Create `~/.cloudflared/config.yml`:

**Single service:**

```yaml
url: http://localhost:8000
tunnel: <TUNNEL-UUID>
credentials-file: /root/.cloudflared/<TUNNEL-UUID>.json
```

**Private network:**

```yaml
tunnel: <TUNNEL-UUID>
credentials-file: /root/.cloudflared/<TUNNEL-UUID>.json
warp-routing:
  enabled: true
```

**Multiple services (ingress rules):**

```yaml
tunnel: <TUNNEL-UUID>
credentials-file: /root/.cloudflared/<TUNNEL-UUID>.json

ingress:
  - hostname: app.example.com
    service: http://localhost:8000
  - hostname: api.example.com
    service: http://localhost:3000
  - hostname: ssh.example.com
    service: ssh://localhost:22
  - service: http_status:404
```

The last rule with no hostname is the **catch-all** - required.

### Step 4: Route Traffic

**Published applications** (creates a DNS CNAME):

```bash
cloudflared tunnel route dns <UUID-or-NAME> <hostname>
# Example:
cloudflared tunnel route dns my-tunnel app.example.com
```

**Private networks** (adds IP route):

```bash
cloudflared tunnel route ip add <IP/CIDR> <UUID-or-NAME>
# Example:
cloudflared tunnel route ip add 10.0.0.0/24 my-tunnel
```

### Step 5: Run the Tunnel

```bash
cloudflared tunnel run <UUID-or-NAME>
```

With a custom config:

```bash
cloudflared tunnel --config /path/to/config.yml run <UUID-or-NAME>
```

### Step 6: Verify

```bash
cloudflared tunnel info <UUID-or-NAME>
```

## Terminology

| Term | Definition |
|------|------------|
| **Tunnel** | Persistent object (UUID) linking your origin to Cloudflare |
| **Connector (cloudflared)** | Daemon instance creating 4 connections to 2+ Cloudflare data centers |
| **Replica** | Additional cloudflared instance running the same tunnel on a different host |
| **Remotely-managed** | Tunnel created via dashboard, config stored in Cloudflare |
| **Locally-managed** | Tunnel created via CLI, config stored in local `config.yml` |
| **Quick tunnel** | Ephemeral tunnel on `trycloudflare.com` for testing |
| **Virtual network** | Logical segregation for overlapping IP routes |

## Permissions Files

| File | Created by | Scope | Expires |
|------|-----------|-------|---------|
| `cert.pem` | `cloudflared tunnel login` | Account-wide (create, delete, list tunnels) | 10+ years (revocable via API Tokens) |
| `<UUID>.json` | `cloudflared tunnel create` | Single tunnel (run only) | Never |

Both stored in `~/.cloudflared/` by default.

To revoke `cert.pem`: delete the API token at **My Profile** > **API Tokens** in the Cloudflare dashboard.
