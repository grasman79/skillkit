---
name: cloudflare-workers-deploy
description: Use this skill when deploying an Astro + Payload CMS monorepo where the Astro frontend deploys to Cloudflare Workers. Activate when the user mentions deploying to Cloudflare Workers, Railway + Cloudflare, monorepo deployment, deploy hooks, or static frontend deployment. Cloudflare Pages functionality has merged into Workers - do not use Pages terminology.
---

# Astro + Payload CMS Monorepo Deployment

Deploy pattern for projects using Payload CMS backend + Astro frontend. The frontend deploys to Cloudflare Workers (formerly Pages - same underlying platform). Backend options: Cloudflare Workers (recommended) or Railway.

## Monorepo Structure

```
project/
├── package.json              # Root (dev scripts only)
├── backend/                  # Payload CMS (Next.js)
│   ├── Dockerfile            # Multi-stage Node 22 Alpine (Railway option)
│   ├── package.json          # pnpm, engines, start script
│   ├── next.config.mjs       # output: 'standalone' (Railway) or standard (Workers)
│   ├── wrangler.jsonc        # Cloudflare Workers config (Workers option)
│   └── src/
│       ├── payload.config.ts # CMS config, D1/R2 bindings or S3 storage
│       ├── endpoints/deploy.ts
│       └── components/DeployButton.tsx
├── frontend/                 # Astro (static)
│   ├── astro.config.mjs      # No adapter for static, or cloudflare adapter for SSR
│   ├── wrangler.jsonc        # Cloudflare Workers config
│   ├── .env.production       # Build-time env vars
│   └── public/_headers       # Security headers
```

## Option A: Full Cloudflare Stack (Recommended)

Both Payload backend and Astro frontend on Cloudflare Workers. See `cms/payload/reference/DEPLOYMENT.md` for the complete Payload on Workers setup (D1 + R2 + OpenNext).

**Cost:** ~$5/month total (Workers paid plan covers both)

**Frontend wrangler.jsonc (static Astro):**
```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "my-site",
  "compatibility_date": "2026-04-21",
  "assets": {
    "directory": "./dist"
  },
  "vars": {
    "PAYLOAD_URL": "https://my-cms.workers.dev"
  }
}
```

**Environment variables:**

Workers (Payload backend):
- `PAYLOAD_SECRET` - Set via `wrangler secret put PAYLOAD_SECRET`
- D1 and R2 bindings configured in wrangler.jsonc

Workers (Astro frontend):
- `PAYLOAD_URL` - Your Payload Worker URL (set via wrangler.jsonc vars or dashboard)

---

## Option B: Railway Backend + Cloudflare Workers Frontend

Payload CMS on Railway (Docker + PostgreSQL), Astro frontend on Cloudflare Workers.

**Cost:** ~$28/month

## Step 1: Railway (Backend) - Option B only

### 1.1 Create Railway Project

1. New Project > Deploy from GitHub repo
2. Select the repo
3. Set **Root Directory** to `backend` (Settings > Source)

### 1.2 Add PostgreSQL

1. New Service > Database > PostgreSQL
2. Railway auto-provisions the database

### 1.3 Set Environment Variables

All 10 variables needed for the Payload backend on Railway:

| Variable | Value | When to set |
|----------|-------|-------------|
| `DATABASE_URI` | `${{Postgres.DATABASE_URL}}` | Initial setup |
| `PAYLOAD_PUBLIC_SERVER_URL` | `https://{service}-production.up.railway.app` | Initial setup |
| `PAYLOAD_SECRET` | `openssl rand -hex 32` | Initial setup |
| `R2_BUCKET` | `{project}-media` | After R2 setup (Step 2) |
| `R2_ENDPOINT` | `https://{account-id}.r2.cloudflarestorage.com` | After R2 setup (Step 2) |
| `R2_PUBLIC_URL` | `https://pub-{hash}.r2.dev` or custom domain | After R2 setup (Step 2) |
| `R2_ACCESS_KEY_ID` | From R2 API token | After R2 setup (Step 2) |
| `R2_SECRET_ACCESS_KEY` | From R2 API token | After R2 setup (Step 2) |
| `CLIENT_URI` | `https://{project}.workers.dev` | After Workers setup (Step 3) |
| `CLOUDFLARE_DEPLOY_HOOK_URL` | Webhook URL | After deploy hook setup (Step 3) |

### 1.4 How the Dockerfile Works

Multi-stage build (no configuration needed):

1. **deps** - Detects pnpm from lockfile, runs `pnpm i --frozen-lockfile`
2. **builder** - Copies deps + source, runs `pnpm run build` (Next.js)
3. **runner** - Copies standalone output, runs as non-root user on port 3000

**Critical**: `next.config.mjs` must have `output: 'standalone'`. The Dockerfile copies `.next/standalone` and `.next/static` for the production image.

### 1.5 Verify

- Backend URL: `https://{service}-production.up.railway.app/admin`
- Create first admin account
- Health check: `GET /api/globals/site-settings` returns JSON

---

## Step 2: Cloudflare R2 Storage (Option B)

### 2.1 Create R2 Bucket

1. Cloudflare dashboard > R2 Object Storage > Create bucket
2. Bucket name: `{project}-media` (lowercase, hyphens)
3. Location: Automatic (or choose region close to users)

### 2.2 Enable Public Access

1. Open the bucket > Settings > Public access
2. Enable R2.dev subdomain (quick option) or set up a custom domain
3. Note the public URL (e.g., `https://pub-{hash}.r2.dev` or your custom domain)
4. This URL becomes `R2_PUBLIC_URL`

### 2.3 Create R2 API Token

1. R2 overview > Manage R2 API Tokens > Create API Token
2. Token name: `Payload - {project}`
3. Permissions: Object Read & Write
4. Specify bucket: select the bucket from 2.1
5. Create and **save immediately** (credentials shown only once):
   - Access Key ID -> `R2_ACCESS_KEY_ID`
   - Secret Access Key -> `R2_SECRET_ACCESS_KEY`
6. Note the Account ID from the Cloudflare dashboard URL or R2 overview page
7. Endpoint: `https://{account-id}.r2.cloudflarestorage.com` -> `R2_ENDPOINT`

### 2.4 Set R2 Variables on Railway

```bash
railway variable set "R2_BUCKET={project}-media" --service {service-name}
railway variable set "R2_ENDPOINT=https://{account-id}.r2.cloudflarestorage.com" --service {service-name}
railway variable set "R2_PUBLIC_URL=https://pub-{hash}.r2.dev" --service {service-name}
railway variable set "R2_ACCESS_KEY_ID={access-key}" --service {service-name}
railway variable set "R2_SECRET_ACCESS_KEY={secret-key}" --service {service-name}
```

---

## Step 3: Cloudflare Workers (Frontend)

### 3.1 Create Workers Project from Git

1. Cloudflare dashboard > Workers & Pages > Create
2. Select **Workers** tab > **Workers Builds** (Git-connected deployment)
3. Connect to Git and select the repo

### 3.2 Build Settings

| Setting | Value |
|---------|-------|
| **Framework preset** | None |
| **Build command** | `bun install && bun run build` |
| **Build output directory** | `dist` |
| **Root directory** (advanced) | `frontend` |

### 3.3 Environment Variable

| Variable | Value |
|---------|-------|
| `PAYLOAD_URL` | `https://{service}-production.up.railway.app` |

### 3.4 wrangler.jsonc (static Astro)

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "project-name",
  "compatibility_date": "2026-04-21",
  "assets": {
    "directory": "./dist"
  },
  "vars": {
    "PAYLOAD_URL": "https://{service}-production.up.railway.app"
  }
}
```

**Note:** For SSR Astro, add `main`, `compatibility_flags: ["nodejs_compat"]`, and `assets.binding: "ASSETS"`. See `cloudflare/astro` skill.

### 3.5 .env.production

Also committed to git (no secrets here):

```
PUBLIC_PAYLOAD_URL=https://{service}-production.up.railway.app
PUBLIC_SITE_URL=https://your-domain.com
PUBLIC_SITE_NAME=Site Name
```

### 3.6 Security Headers

`frontend/public/_headers` is served by Cloudflare Workers automatically for static assets:

```
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: SAMEORIGIN
  Referrer-Policy: strict-origin-when-cross-origin
  X-DNS-Prefetch-Control: on
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
  Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.hugeicons.com; img-src 'self' data: https:; font-src 'self' https://fonts.gstatic.com https://cdn.hugeicons.com; connect-src 'self'; frame-ancestors 'self'; base-uri 'self'; form-action 'self'
```

Adjust `style-src`, `font-src`, and `img-src` per project (e.g., add R2 domain to `img-src` for uploaded images).

### 3.7 Verify

- Workers URL: `https://{project}.workers.dev`
- Check that content loads from CMS
- Check security headers with browser DevTools (Network tab > Response Headers)

---

## Step 4: Connect Services (Option B)

### 4.1 Set CLIENT_URI on Railway

```bash
railway variable set "CLIENT_URI=https://{project}.workers.dev" --service {service-name}
```

This configures CORS so the Astro frontend can access the Payload API.

### 4.2 Create Deploy Hook

1. Cloudflare dashboard > Workers & Pages > your Worker project > Settings > Builds
2. Add deploy hook: Name "Payload", Branch "main"
3. Copy the webhook URL
4. Set on Railway:

```bash
railway variable set "CLOUDFLARE_DEPLOY_HOOK_URL={webhook-url}" --service {service-name}
```

### 4.3 How the Deploy Hook Works

```
Admin clicks "Deploy Website" in Payload sidebar
  -> POST /api/deploy (authenticated, backend)
  -> fetch(CLOUDFLARE_DEPLOY_HOOK_URL, { method: 'POST' })
  -> Cloudflare Workers triggers new build
  -> Astro fetches latest content from Payload API during build
  -> Static site deployed with fresh content
```

The deploy endpoint (`backend/src/endpoints/deploy.ts`) requires authentication - only logged-in admin users can trigger deploys.

### 4.4 Verify End-to-End

- Media uploads: Upload an image in Payload admin, verify it's stored in R2 and loads via the public URL
- Deploy hook: Click "Deploy Website" in Payload admin, verify Cloudflare Workers build triggers
- Content flow: Create a test post with an image, trigger rebuild, verify it appears on the frontend

---

## Checklist for New Projects

### From Template

1. [ ] Clone astro-payload-template
2. [ ] Update `frontend/wrangler.jsonc` - project name + PAYLOAD_URL
3. [ ] Update `frontend/.env.production` - PUBLIC_PAYLOAD_URL
4. [ ] Update `backend/src/payload.config.ts` - CORS origins if custom domain

### Railway (Option B only)

1. [ ] Create project + PostgreSQL
2. [ ] Set root directory to `backend`
3. [ ] Set initial env vars: DATABASE_URI, PAYLOAD_PUBLIC_SERVER_URL, PAYLOAD_SECRET
4. [ ] Verify admin panel loads
5. [ ] Create admin account

### Cloudflare R2 (Option B only)

1. [ ] Create R2 bucket (`{project}-media`)
2. [ ] Enable public access (R2.dev subdomain or custom domain)
3. [ ] Create R2 API token (Object Read & Write, scoped to bucket)
4. [ ] Save Access Key ID and Secret Access Key
5. [ ] Set R2_* env vars on Railway (5 variables)
6. [ ] Regenerate Payload import map: `cd backend && pnpm run payload generate:importmap`
7. [ ] Commit the updated import map and redeploy
8. [ ] Test media upload in Payload admin

### Cloudflare Workers (Frontend)

1. [ ] Create Workers project from Git (Workers & Pages > Create > Workers Builds)
2. [ ] Set build command: `bun install && bun run build`
3. [ ] Set output directory: `dist`
4. [ ] Set root directory: `frontend`
5. [ ] Set `PAYLOAD_URL` env var
6. [ ] Verify site loads with CMS content

### Connect Services (Option B only)

1. [ ] Set `CLIENT_URI` on Railway (Cloudflare Workers URL)
2. [ ] Create deploy hook in Cloudflare Workers settings (Settings > Builds)
3. [ ] Set `CLOUDFLARE_DEPLOY_HOOK_URL` on Railway
4. [ ] Test: click "Deploy Website" in Payload admin
5. [ ] Verify Cloudflare Workers build triggers
6. [ ] Add R2 public domain to CSP `img-src` in `_headers`

---

## Troubleshooting

### Railway: "No start command found"

Root directory not set to `backend`. Railway is reading the root package.json.

### Railway: "Module not found" during build

Dependencies in `package.json` but not in `pnpm-lock.yaml`. Run `cd backend && pnpm install` and commit the updated lockfile.

### Cloudflare: Build succeeds but pages show fallback content

`PAYLOAD_URL` points to wrong backend or backend isn't reachable during build. Check Railway URL and ensure backend is deployed and running.

### Deploy button: "Deploy hook URL not configured"

`CLOUDFLARE_DEPLOY_HOOK_URL` not set in Railway. Add the webhook URL from Cloudflare Workers settings (Settings > Builds > Deploy hooks).

### Media images return 404

R2 env vars not set or R2 public URL not matching. Check `R2_PUBLIC_URL` matches the URL used in `generateFileURL`. Also ensure R2 bucket CORS allows the frontend domain.

### Railway: S3 storage plugin client component not in import map

After adding the S3/R2 storage plugin, Payload fails because the plugin's client component isn't registered in the import map. Regenerate it and commit:

```bash
cd backend && pnpm run payload generate:importmap
```

This updates `src/app/(payload)/importMap.js`. Commit the file and redeploy.

### R2 upload fails with 403

R2 API token doesn't have write permissions, or is scoped to the wrong bucket. Verify the token in Cloudflare dashboard > R2 > Manage R2 API Tokens.
