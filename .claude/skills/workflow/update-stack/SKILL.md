---
name: update-stack
description: Use this skill when safely updating all project dependencies, frameworks, or packages to their latest versions. Activate when the user mentions updating dependencies, upgrading packages, updating the tech stack, running dependency updates, or checking for outdated packages.
---

# Update Stack

Safely updates all project dependencies to their latest versions. Lists available updates, researches breaking changes in major version jumps, classifies migration effort, then applies updates based on user choice.

## When to Use This Skill

- User says "update stack", "update dependencies", "upgrade packages", or "check for outdated packages"
- User wants to update to latest versions of frameworks
- Periodic maintenance (recommended: monthly)

## Instructions

### Step 1: Detect Package Manager

Check for lock files:

| Lock File | Package Manager | Outdated Command | Update Command |
|-----------|-----------------|------------------|----------------|
| `bun.lockb` or `bun.lock` | bun | `bun outdated` | `bun update` |
| `pnpm-lock.yaml` | pnpm | `pnpm outdated` | `pnpm update` |
| `yarn.lock` | yarn | `yarn outdated` | `yarn upgrade` |
| `package-lock.json` | npm | `npm outdated` | `npm update` |

### Step 2: Audit Current State

1. **Read `package.json`** to understand the dependency list
2. **Run the outdated command** for the detected package manager
3. **Parse the output** - note current version, latest version, and whether it's a patch, minor, or major bump

### Step 3: Categorize Updates

Split updates into two groups:

**Safe (patch/minor):** `x.y.Z` or `x.Y.z` bumps - apply automatically, no review needed.

**Major version jumps:** `X.y.z` bumps - breaking changes likely, require changelog research before touching.

### Step 4: Research Major Version Changelogs

For each major version jump found, fetch its changelog and summarize:

1. **Look up the changelog URL** - use the reference table below, or derive it:
   - Check `package.json` homepage/repository field: `npm info {package} repository.url`
   - GitHub releases: `https://github.com/{owner}/{repo}/releases`
   - Package docs site migration guide

2. **Fetch and read the changelog** for the version range being jumped (e.g., all changes from v1.x to v2.0)

3. **Extract:**
   - What APIs, methods, or config options changed or were removed
   - Whether a migration script or CLI tool exists
   - Estimated code impact (how many files/patterns affected)

4. **Classify migration effort:**

| Effort | Time | Signal |
|--------|------|--------|
| Easy | 15-30 min | Only import paths, renamed config keys, or type-only changes. No component or hook logic changes. |
| Medium | 1-3 hours | Some API methods renamed, prop changes, a handful of files to update. |
| Hard | Half day+ | Architecture changes, major API redesign, migration script required, or many files affected. |

**Known changelog URLs for common packages:**

| Package | Changelog / Migration Guide |
|---------|----------------------------|
| `tailwindcss` | https://tailwindcss.com/docs/upgrade-guide |
| `@tanstack/react-query` | https://tanstack.com/query/latest/docs/framework/react/guides/migrating |
| `@tanstack/react-router` | https://tanstack.com/router/latest/docs/framework/react/guide/migrating |
| `@tanstack/react-start` | https://tanstack.com/start/latest/docs/framework/react/guide/migrating |
| `better-auth` | https://www.better-auth.com/docs/changelog |
| `drizzle-orm` | https://orm.drizzle.team/docs/changelog |
| `drizzle-kit` | https://orm.drizzle.team/docs/changelog |
| `next` | https://nextjs.org/docs/app/building-your-application/upgrading |
| `react` / `react-dom` | https://react.dev/blog (check major release post) |
| `supabase-js` | https://supabase.com/docs/reference/javascript/upgrade-guide |
| `payload` | https://payloadcms.com/docs/getting-started/changelog |
| `astro` | https://docs.astro.build/en/guides/upgrade-to/v5/ |
| `vite` | https://vite.dev/guide/migration |
| `vitest` | https://vitest.dev/guide/migration |
| `eslint` | https://eslint.org/docs/latest/use/migrate-to-9.0.0 |
| `stripe` | https://github.com/stripe/stripe-node/blob/master/CHANGELOG.md |
| `resend` | https://github.com/resendlabs/resend-node/releases |
| `zod` | https://zod.dev/changelog |
| `hono` | https://github.com/honojs/hono/releases |
| `lucia` | https://v3.luciaauth.com/upgrade-v2 |
| `uploadthing` | https://docs.uploadthing.com/changelog |

For packages not in this list: fetch `https://github.com/{owner}/{repo}/releases` or the npm package page.

### Step 5: Present Full Picture to User

Present a single consolidated view before taking any action:

```
Dependency audit complete. Here's what I found:

SAFE UPDATES (patch/minor - will apply automatically)
- @tanstack/react-query: 5.90.0 → 5.92.0
- tailwindcss: 4.0.6 → 4.0.8
- drizzle-orm: 0.45.0 → 0.45.2
- typescript: 5.4.0 → 5.5.2
(12 more patch/minor updates)

MAJOR VERSION JUMPS

📦 better-auth: 1.4.12 → 2.0.0  [Medium - ~2 hours]
  Breaking changes:
  - auth.api renamed to auth.handler in all server routes
  - Session cookie name changed - existing sessions will be invalidated
  - Plugin config moved from plugins[] to a new pluginConfig object
  Migration guide: https://www.better-auth.com/docs/changelog

📦 next: 14.2.0 → 15.0.0  [Medium - ~1-2 hours]
  Breaking changes:
  - fetch() caching now opt-in (no-store by default)
  - cookies() and headers() are now async - await required
  - Turbopack is stable, now default in dev
  Migration guide: https://nextjs.org/docs/app/building-your-application/upgrading

📦 tailwindcss: 3.4.17 → 4.0.0  [Hard - ~half day]
  Breaking changes:
  - Config format changed from JS (tailwind.config.js) to CSS (@import "tailwindcss")
  - Many utility class names renamed (e.g. shadow-sm → shadow-xs)
  - PostCSS plugin replaced with a new package (@tailwindcss/postcss)
  - Official upgrade tool available: npx @tailwindcss/upgrade
  Migration guide: https://tailwindcss.com/docs/upgrade-guide

How would you like to proceed?
A. Safe updates only (recommended - apply all patch/minor now, skip majors)
B. Safe updates + easy/medium majors (better-auth + next)
C. Everything including Tailwind v4 (longest, most work)
D. Let me pick specific packages
```

### Step 6: Perform Updates

Based on user choice:

**Applying safe (patch/minor) updates:**
```bash
# Bun - updates within semver range
bun update

# pnpm
pnpm update

# npm
npm update

# yarn
yarn upgrade
```

**Applying specific major version updates:**
```bash
# Bun
bun add package-name@latest

# pnpm
pnpm add package-name@latest

# npm
npm install package-name@latest

# yarn
yarn add package-name@latest
```

**If an official upgrade CLI exists, run it first** (e.g. `npx @tailwindcss/upgrade`, `npx @next/codemod`).

Update one major package at a time, then verify before moving to the next.

### Step 7: Verify After Each Major Update

After each major update (or after applying all safe updates):

1. **TypeScript check:**
```bash
bun run typecheck   # or pnpm typecheck / npx tsc --noEmit
```

2. **Linter:**
```bash
npx ultracite fix   # or biome check --write .
```

3. **Dev server:**
```bash
bun dev   # or pnpm dev
```

4. **Tests (if they exist):**
```bash
bun test   # or pnpm test
```

Fix any errors before moving to the next major update.

### Step 8: Handle Breaking Changes

If errors occur after updating:

1. **Read the error** - identify which package caused it
2. **Refer to the breaking changes summary** from Step 4
3. **Apply the migration** - update code to match the new API
4. **If stuck, rollback that package:**
```bash
# Restore lock file from git
git checkout -- bun.lockb   # or pnpm-lock.yaml / package-lock.json

# Reinstall at previous version
bun install
```

### Step 9: Final Report

```
Stack updated!

APPLIED (15 packages):
- typescript: 5.4.0 → 5.5.2
- @tanstack/react-query: 5.90.0 → 5.92.0
- drizzle-orm: 0.45.0 → 0.45.2
- better-auth: 1.4.12 → 2.0.0
- next: 14.2.0 → 15.0.0
[... full list ...]

VERIFICATION:
- TypeScript: no errors
- Linter: passed
- Dev server: running

SKIPPED (deferred for later):
- tailwindcss: 3.4.17 → 4.0.0 (Hard migration - set aside ~half day)

Commit suggested: "chore: update dependencies - patch/minor + better-auth v2, next v15"
```

## Tips

- Always commit before updating so you can rollback with `git checkout`
- Update monthly to avoid large version jumps accumulating
- Tackle one Hard migration per session - don't bundle them
- If a major version just released, wait a week for hotfixes before upgrading
- When NOT to update: right before a launch, during active feature development, or when under time pressure

## Adding npm Scripts

Suggest adding to `package.json` for regular use:

```json
{
  "scripts": {
    "deps:check": "bun outdated",
    "deps:update": "bun update",
    "deps:update-all": "bun update --latest"
  }
}
```
