---
name: railway
description: Railway deployment platform for TanStack Start with Nitro SSR. Trigger words - railway, deploy to railway, railway deployment, ssr deployment, nitro deployment, railway.app
---

# Railway

Modern deployment platform with one-click database provisioning, automatic SSL, and preview environments for every PR.

## When to Use This Skill

- Deploying TanStack Start applications with Nitro SSR
- Need PostgreSQL, MySQL, Redis, or MongoDB alongside your app
- Want automatic preview deployments for PRs
- Building full-stack SSR applications
- Need WebSocket support or long-running connections
- Require background jobs and persistent processes

## When NOT to Use

- Building static sites (use Vercel or Netlify instead)
- Need specific cloud provider (AWS/GCP/Azure)
- Require Kubernetes or complex container orchestration

## Prerequisites

- TanStack Start application with Nitro configured
- GitHub repository
- Railway account ([railway.com](https://railway.com))

## 1. Initial Setup

### Install Nitro (Nightly)

Add to `package.json`:

```json
{
  "dependencies": {
    "nitro": "npm:nitro-nightly@latest"
  }
}
```

Or pin to a specific nightly version:
```json
{
  "dependencies": {
    "nitro": "npm:nitro-nightly@^3.0.1-20260209-102041-99f7fe3d"
  }
}
```

### Configure Vite

In `vite.config.ts`, ensure plugins are in the correct order:

```typescript
import { tanstackStart } from '@tanstack/react-start/plugin/vite'
import { nitro } from 'nitro/vite'
import viteReact from '@vitejs/plugin-react'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [
    tanstackStart(),  // MUST be first
    nitro(),          // Nitro for server deployment
    viteReact(),
  ],
})
```

### Configure Package Scripts

```json
{
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "start": "node .output/server/index.mjs"
  }
}
```

## 2. Railway Configuration

### Critical Environment Variable

Railway's Railpack builder auto-detects projects. For SSR apps (not SPAs), you MUST add:

**`RAILPACK_NO_SPA=1`**

This tells Railway to use your Node.js server instead of serving static files through Caddy.

### Service Settings

Railway will auto-detect these from your `package.json`:

- **Build Command**: `bun run build` (or `npm run build`)
- **Start Command**: `bun run start` (runs `node .output/server/index.mjs`)
- **Port**: Railway automatically sets `PORT` env var (typically 8080)

**Do NOT** set a custom start command unless necessary. Let Railway use your `package.json` scripts.

### Environment Variables

Set these in Railway > Service > Variables:

#### Required for SSR
```
RAILPACK_NO_SPA=1
```

#### Your Application Variables
```
# Database (Supabase/Postgres)
DATABASE_URL=postgresql://...
SUPABASE_SERVICE_ROLE_KEY=...

# Application URLs (use your Railway domain)
VITE_APP_URL=https://your-app.up.railway.app
BETTER_AUTH_URL=https://your-app.up.railway.app

# API Keys
OPENROUTER_API_KEY=...
RESEND_API_KEY=...
# ... etc
```

**Important**: `VITE_*` prefixed variables are baked into the client bundle at build time. Make sure they're set before building.

## 3. Deployment Steps

1. **Push to GitHub**
   ```bash
   git push origin main
   ```

2. **Connect to Railway**
   - Go to [railway.com](https://railway.com)
   - Click "New Project"
   - Select "Deploy from GitHub repo"
   - Choose your repository

3. **Configure Environment Variables**
   - Go to your service > Variables tab
   - Add `RAILPACK_NO_SPA=1`
   - Add all your application environment variables

4. **Trigger Deployment**
   - Railway will automatically deploy on push to your main branch
   - Or manually redeploy from the Railway dashboard

## 4. Common Issues & Solutions

### 502 Bad Gateway

**Symptom**: Server logs show "Listening on port 8080" but browser shows 502

**Cause**: Railway's Railpack detected your app as an SPA and is serving it through Caddy instead of your Node.js server

**Solution**: Add `RAILPACK_NO_SPA=1` environment variable and redeploy

### Port Configuration

The Nitro server automatically reads Railway's `PORT` environment variable. No configuration needed.

```typescript
// Nitro automatically uses process.env.PORT or process.env.NITRO_PORT
// Defaults to 3000 if neither is set
```

### Database Connection Issues

**Symptom**: Server crashes immediately after "Listening on port..."

**Cause**: `pg` Pool emits unhandled errors if database connection fails

**Solution**: Add error handler to your Pool:

```typescript
import { Pool } from 'pg'

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false }
})

// Prevent unhandled pool errors from crashing the process
pool.on('error', (err) => {
  console.error('Unexpected pg pool error:', err.message)
})
```

### Build Failures

**Module not found errors**: Check that paths use correct aliases:
- `@/*` for shared code in project root
- `~/*` for TanStack Start-specific code in `src/`

**Server-only modules leaking to client**: Create a Vite plugin to stub them:

```typescript
function serverOnlyModules(): Plugin {
  const serverOnlyPackages = new Set(['pg', 'node:async_hooks'])

  return {
    name: 'server-only-modules',
    enforce: 'pre',
    applyToEnvironment(env) {
      return env.name === 'client'
    },
    resolveId(id) {
      const pkgName = id.split('/')[0]
      if (serverOnlyPackages.has(pkgName)) {
        return { id: `\0server-only:${id}` }
      }
    },
    load(id) {
      if (id.startsWith('\0server-only:')) {
        return 'export default {}'  // Empty stub
      }
    }
  }
}
```

## 5. Performance Optimization

### FastResponse (Node.js Only)

For ~5% throughput improvement, use srvx's optimized Response:

```bash
npm install srvx
```

In your server entry (`src/server.ts`):

```typescript
import { FastResponse } from 'srvx'
globalThis.Response = FastResponse
```

This only works with Node.js deployments using Nitro/srvx.

## 6. Railway Features

### Automatic Deployments
- Every push to `main` triggers a new deployment
- Pull requests get preview environments

### Built-in Services
- PostgreSQL, MySQL, Redis, MongoDB available
- One-click provisioning
- Connection strings auto-injected as env vars

### Monitoring
- **Deploy Logs**: Build output
- **HTTP Logs**: Request/response logs
- **Metrics**: CPU, memory, network usage

### Custom Domains
- Add your own domain in Railway > Settings > Domains
- SSL automatically provisioned

## 7. Project Structure

```
project/
├── src/
│   ├── routes/          # TanStack Router routes
│   ├── client.tsx       # Client entry
│   ├── ssr.tsx          # Server entry
│   └── router.tsx       # Router config
├── lib/                 # Shared utilities
├── components/          # React components
├── vite.config.ts       # Vite + Nitro config
├── nitro.config.ts      # Nitro runtime config (optional)
├── package.json         # Scripts and dependencies
└── .output/             # Built output (gitignored)
    └── server/
        └── index.mjs    # Production server
```

## 8. Troubleshooting Checklist

Before opening a support ticket:

- [ ] `RAILPACK_NO_SPA=1` is set
- [ ] All `VITE_*` variables are set in Railway
- [ ] Build command is `bun run build` or `npm run build`
- [ ] Start command is `bun run start` or `npm run start`
- [ ] Database connection string is correct
- [ ] Server-only modules don't leak to client bundle
- [ ] Pool error handler is added (if using `pg`)
- [ ] Railway domain is in auth trustedOrigins

## Tips

- Always set `RAILPACK_NO_SPA=1` for SSR applications
- Use Railway's built-in database services for zero-config setup
- Preview environments automatically created for PRs
- Environment variable changes require redeploy
- Check Deploy Logs for build errors, HTTP Logs for runtime issues
- Railway automatically handles SSL certificates

## How to Verify

### Quick Checks
- Railway deployment shows "Success" status
- Application accessible at `*.up.railway.app` URL
- Server logs show "Listening on port 8080"
- No 502 Bad Gateway errors

### Common Issues
- "502 Bad Gateway": Missing `RAILPACK_NO_SPA=1`
- "Application failed to respond": Check port configuration
- "Build failed": Review Deploy Logs for errors
- "Database connection refused": Verify `DATABASE_URL` is set
- "Server crashes on start": Add error handler to database pool

## Resources

- [TanStack Start Hosting Docs](https://tanstack.com/start/latest/docs/framework/react/guide/hosting)
- [Railway TanStack Start Help Thread](https://station.railway.com/questions/502-tan-stack-start-project-e00eb065)
- [Railway Error Reference](https://docs.railway.com/reference/errors/application-failed-to-respond)
- [Railway Deploy Templates](https://railway.com/deploy/trustworthy-smile)
- [Railway Help Station](https://station.railway.com)
- [TanStack Discord](https://discord.gg/tanstack)
