# Cloudflare Workers Deployment - Astro Frontend

Step-by-step guide for deploying the Astro frontend to Cloudflare Workers. Cloudflare Pages functionality has fully merged into Workers - all new projects should use Workers.

## Prerequisites

- A GitHub repository with the monorepo structure:
  ```
  repo/
  ├── backend/          # Payload CMS (deployed separately - Railway or Cloudflare Workers)
  └── frontend/         # Astro (deployed to Cloudflare Workers)
  ```
- A Cloudflare account (free tier is sufficient for Workers static sites)
- Backend (Payload CMS) already deployed and accessible via a public URL

## Step 1: Install Dependencies

```bash
# Frontend (Astro) - uses bun
cd frontend && bun install

# Backend (Payload) - uses bun for installing, but pnpm for running
cd backend && bun install
```

### Dev Server Commands

The frontend and backend use different runtimes for development:

| Server | Command | Why |
|--------|---------|-----|
| Frontend (Astro) | `bun run dev` | Full Bun support, runs on localhost:4321 |
| Backend (Payload) | `pnpm run dev` | Payload requires Node.js runtime, runs on localhost:3000 |

**Important:** Payload does not support Bun as a runtime (only as a package manager). The backend keeps pnpm for running dev server and CLI commands like migrations and type generation. See the Payload deployment skill for full details on Bun/Node.js compatibility.

## Step 2: First Commit & Push to GitHub

Cloudflare Workers needs at least one branch pushed to connect a repository.

```bash
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

If the repo already has commits pushed, skip this step.

## Step 3: Deploy Backend First

The Payload backend must be live before connecting Cloudflare Workers, because the frontend needs `PUBLIC_PAYLOAD_URL` as an environment variable to fetch content at build time.

**Deploy the backend based on your choice:**

| Platform | Guide |
|----------|-------|
| Cloudflare Workers | See `cms/payload/reference/DEPLOYMENT.md` - Cloudflare Workers section |
| Railway | See [railway-payload-deploy.md](railway-payload-deploy.md) |

After deployment, note the backend URL (e.g., `https://my-cms.workers.dev` or `https://my-project-production.up.railway.app`).

## Step 4: Set Up Database

Database setup depends on your backend choice. If you followed the Railway guide, Postgres is already provisioned. If using Cloudflare Workers, D1 is configured via wrangler bindings.

Make sure the backend is running and accessible before proceeding.

## Step 5: Connect Cloudflare Workers to GitHub

1. Go to the [Cloudflare dashboard](https://dash.cloudflare.com)
2. Navigate to **Workers & Pages**
3. Click **Create**
4. Select **Workers** tab > **Workers Builds** (Git-connected deployment)
5. Click **Connect to Git**
6. Select your GitHub repository
7. Click **Begin setup**

## Step 6: Configure Build Settings

On the "Set up builds and deployments" screen, enter these exact values:

| Setting | Value |
|---------|-------|
| Project name | Your project name (auto-filled from repo, can customize) |
| Production branch | `main` |
| Framework preset | `None` (leave as None - we set the build command manually) |
| Build command | `bun install && bun run build` |
| Build output directory | `dist` |
| Root directory (advanced) | `frontend` |

**Important:** The build output directory is controlled by `assets.directory` in `frontend/wrangler.jsonc` (set to `./dist`). If the dashboard locks this field, modify `wrangler.jsonc` instead.

### Environment Variables

Click **+ Add variable** and add:

| Type | Name | Value |
|------|------|-------|
| Plaintext | `PUBLIC_PAYLOAD_URL` | Your deployed Payload backend URL |

**Note:** The build command includes `bun install` because Cloudflare Workers needs to install the frontend dependencies within the `frontend/` root directory before building.

## Step 7: Save and Deploy

Click **Save and Deploy**. The first build starts immediately.

- Cloudflare Workers installs dependencies, runs the build, and deploys to its global CDN
- Your site will be available at `{project-name}.workers.dev`
- Automatic deployments are enabled by default - every push to `main` triggers a new build
- Pull requests get preview deployments automatically

## Step 8: Set Up Deploy Hook (Required)

The deploy-tracker plugin in Payload needs a Cloudflare deploy hook to trigger rebuilds when content changes.

1. In your Workers project, go to **Settings > Builds > Deploy hooks**
2. Click **Add deploy hook**
3. Name it `Payload` and select the `main` branch
4. Copy the generated webhook URL
5. In Railway (or wherever Payload is hosted), add the environment variable:
   ```
   CLOUDFLARE_DEPLOY_HOOK_URL={webhook-url}
   ```
6. Redeploy the backend so it picks up the new env var

Now editors can click "Deploy Website" in the Payload admin panel to trigger a Cloudflare Workers rebuild.

## Step 9: Verify Post-Deploy Settings

After the first deploy completes, go to the Workers project **Settings** tab and verify:

### Build Settings

| Setting | Expected Value |
|---------|---------------|
| Git repository | Your GitHub repo |
| Build command | `bun install && bun run build` |
| Root directory | `frontend` |
| Branch control | Production: main, Automatic deployments: Enabled |

### Runtime Settings

| Setting | Value |
|---------|-------|
| Compatibility flags | `nodejs_compat` |
| Compatibility date | Recent date |

### Variables and Secrets

| Type | Name | Value |
|------|------|-------|
| Plaintext | `PUBLIC_PAYLOAD_URL` | Your Payload backend URL |

## Step 10: Configure Staging Environment

Cloudflare Workers automatically deploys every branch. The `staging` branch gets a preview URL.

### Set up separate environment variables for staging

1. In your Workers project, go to **Settings > Environment variables**
2. Switch to the **Preview** tab
3. Add `PUBLIC_PAYLOAD_URL` pointing to your staging Payload backend (or the same production backend if you only have one)

The Production tab should already have the production values from Step 6.

### Verify staging works

1. Push to the staging branch: `git checkout staging && git merge main && git push`
2. Wait for the Cloudflare build to complete
3. Visit the preview URL - the site should load

### Branch workflow

| Branch | Deploys to | Purpose |
|--------|-----------|---------|
| `main` | Production domain | Live site |
| `staging` | Auto-generated preview URL | Testing before production |
| Feature branches | Auto-generated preview URLs | PR review |

Merge flow: `feature/my-change` -> `staging` (test) -> `main` (production)

## Step 11: Enable CI Workflows

The template includes two GitHub Actions workflows that run on PRs and staging pushes.

### Add GitHub Actions secret

1. Go to your GitHub repo **Settings > Secrets and variables > Actions**
2. Click **New repository secret**
3. Name: `PAYLOAD_URL`, Value: your Payload backend URL (same as `PUBLIC_PAYLOAD_URL`)

### Lighthouse CI (`.github/workflows/lighthouse.yml`)

Runs on pushes to `staging` and PRs to `main`/`staging`. Tests performance, accessibility, best practices, and SEO with minimum thresholds (Performance >= 90, Accessibility >= 95, Best Practices = 100, SEO >= 95).

### Performance Budget (`.github/workflows/performance-budget.yml`)

Runs on the same triggers. Enforces:
- Total JS bundle < 50KB
- Any HTML page < 200KB

Fails the PR if budgets are exceeded. Writes a summary table to the GitHub Actions job summary.

### Verify CI works

Push a change to the staging branch or open a PR. Check the Actions tab in GitHub - both workflows should run and pass.

## Step 12: Custom Domain (Optional)

To add a custom domain:

1. Go to your Workers project **Custom domains** tab
2. Click **Add custom domain**
3. Enter your domain (e.g., `www.example.com`)
4. Cloudflare handles DNS and SSL automatically if the domain is already on Cloudflare

## Quick Reference

All settings at a glance:

```
Project name:           {project-name}
Production branch:      main
Framework preset:       None
Build command:          bun install && bun run build
Build output directory: dist
Root directory:         frontend
Environment variable:   PUBLIC_PAYLOAD_URL = https://{your-payload-backend-url}
```

## wrangler.jsonc (Static Astro)

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

For SSR Astro, see `cloudflare/astro` skill.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Build fails with "command not found: bun" | Verify Build system version supports Bun natively |
| Build succeeds but pages are empty | Check that `PUBLIC_PAYLOAD_URL` is set correctly and the Payload backend is accessible |
| Build fails with module not found | The build command should be `bun install && bun run build` (includes install step) |
| Content not updating after Payload changes | Click "Deploy Website" in Payload admin, or push a commit to main |
| "Deploy Website" button doesn't trigger build | Check `CLOUDFLARE_DEPLOY_HOOK_URL` env var on the backend |
| Custom domain shows old content | Check zone-level cache rules - they may be caching HTML. Add cache exclusion for your Workers domain or purge the zone cache |
| 404 on page routes | Check `assets.directory` in wrangler.jsonc points to `./dist` |
| Font 404 errors | Check that font files exist in `public/fonts/` and the paths in CSS match |
| Staging branch not deploying | Push at least one commit to `staging` branch. Cloudflare auto-deploys all branches by default |
| Lighthouse CI not running | Add `PAYLOAD_URL` as GitHub Actions secret. Workflow triggers on `staging` push and PRs to `main`/`staging` |
| Performance budget failing | Check JS bundle size in `dist/_astro/*.js`. Budget is 50KB total JS, 200KB per HTML page |
