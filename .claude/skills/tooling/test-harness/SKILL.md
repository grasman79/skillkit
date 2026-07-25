---
name: test-harness
description: Use this skill when a project needs a test runner installed and wired up, or when acceptance checks need somewhere to live. Activate when the user mentions setting up tests, adding a test runner, "we have no tests", test infrastructure, Vitest, Playwright, e2e tests, browser tests, or when feature-planner needs to write acceptance checks into a project that has no test setup.
---

# Test Harness

Installs and wires up a test runner so acceptance checks have somewhere to live. This is infrastructure setup only. Writing the checks themselves belongs to `workflow/feature-planner`.

## When to Use This Skill

- Project has no test runner and a feature needs acceptance checks
- User says "set up tests", "add a test runner", "we have no tests"
- User wants browser-level end-to-end tests
- `feature-planner` reaches Step 3 and finds no place to put checks

## Before Installing Anything

1. Check `package.json` for an existing runner (`vitest`, `jest`, `bun:test`, `playwright`, `cypress`)
2. Check for config files: `vitest.config.*`, `jest.config.*`, `playwright.config.*`
3. Look for a `tests/`, `__tests__/`, or `e2e/` directory

If a runner already exists, use it. Never add a second runner alongside one that works.

## Two Layers, Different Jobs

| Layer | Runner | Answers | Runs in |
|-------|--------|---------|---------|
| **Fast checks** | Vitest | Does this function/route/rule behave correctly? | Milliseconds, on every commit |
| **Journey checks** | Playwright | Can a real user complete this path in a real browser? | Minutes, before merge or on deploy |

Set up the fast layer first. Only add the journey layer when the app has a running URL to point at.

## Layer 1: Fast Checks (Vitest)

Default runner for anything TypeScript or JavaScript. Fast, no config for the common case, same API as Jest.

```bash
[package-manager] add -D vitest
```

Add to `package.json`:

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest"
  }
}
```

`vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [tsconfigPaths()],
  test: {
    environment: 'node',
    include: ['tests/**/*.test.ts'],
  },
});
```

**If the project renders React components**, add the DOM environment:

```bash
[package-manager] add -D @testing-library/react @testing-library/user-event jsdom
```

```typescript
test: {
  environment: 'jsdom',
  include: ['tests/**/*.test.{ts,tsx}'],
}
```

**Directory layout** - mirror the source tree so a check is findable from the file it covers:

```
tests/
  lib/
    credits.test.ts        <- covers lib/credits.ts
    auth.test.ts           <- covers lib/auth.ts
  components/
    attach-menu.test.tsx
```

## Layer 2: Journey Checks (Playwright)

Add this when the user does not test locally, when a staging deploy can silently break, or when the app has flows that span several pages.

```bash
[package-manager] add -D @playwright/test
[runner] playwright install chromium
```

`playwright.config.ts`:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  timeout: 60_000,
  retries: 1,
  use: {
    baseURL: process.env.E2E_BASE_URL ?? 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
});
```

Add to `package.json`:

```json
{
  "scripts": {
    "test:e2e": "playwright test"
  }
}
```

**Point it at a real environment.** If the user deploys to staging and reviews there rather than locally, `E2E_BASE_URL` should be the staging URL. Testing localhost when nobody uses localhost catches nothing.

### How Many Journeys

Five to eight. Not thirty, not two hundred.

Pick the paths where breakage would be embarrassing or expensive:

1. Sign up and log in
2. The core action the product exists for
3. Anything that moves money or credits
4. Anything that creates the primary object (a project, a campaign, a document)
5. One path that spans several pages end to end

A large journey suite is slow, flaky, and gets muted. A small one that stays green is worth more than a big one nobody trusts.

### Journey Check Shape

```typescript
import { test, expect } from '@playwright/test';

test('user can create a project', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.E2E_USER!);
  await page.getByLabel('Password').fill(process.env.E2E_PASSWORD!);
  await page.getByRole('button', { name: 'Sign in' }).click();

  await page.getByRole('button', { name: 'New project' }).click();
  await page.getByLabel('Name').fill('Test project');
  await page.getByRole('button', { name: 'Create' }).click();

  await expect(page.getByText('Test project')).toBeVisible();
});
```

**Select by role and label, not by CSS class.** Class names change every time the design changes. Roles and labels change when the product changes, which is when a journey check should break.

## Test Credentials

Journey checks need a real account. Never hardcode credentials.

1. Create a dedicated test user in the target environment
2. Put the credentials in `.env.test` or the CI secret store
3. Add `.env.test` to `.gitignore`
4. Read them via `process.env` in the config or the check itself

If the environment is shared or production-adjacent, the test user must be scoped so a failing check cannot damage real data.

## Wiring Into CI

If the project has GitHub Actions, add the fast layer to the existing workflow. It costs seconds and catches most regressions.

```yaml
- run: [package-manager] install
- run: [package-manager] run test
```

Journey checks are slower. Run them after deploy against the deployed URL, not on every push.

## What Not To Do

- Do not add a coverage threshold. It rewards checking trivial code and tells you nothing about whether the important paths work.
- Do not add a second test runner alongside a working one.
- Do not write checks that assert on the shape of generated or AI-produced content. Assert that the call was made, the response was stored, and errors are handled. A human judges the content.
- Do not mock the thing under test. A check that mocks the function it is checking passes forever and means nothing.
- Do not test framework behaviour. The router works. The ORM works. Check your own logic.

## How to Verify

### Quick Checks
- `[package-manager] run test` runs and reports zero failures on a fresh setup
- A deliberately broken assertion makes the run fail (prove the runner actually runs)
- `[package-manager] run test:e2e` opens a browser and reaches the app

### Common Issues
- **"Cannot find module '@/...'"**: add `vite-tsconfig-paths` to the Vitest config plugins
- **"document is not defined"**: component checks need `environment: 'jsdom'`
- **Playwright times out immediately**: `baseURL` points at nothing running, check `E2E_BASE_URL`
- **Journey check passes locally, fails in CI**: usually the test user does not exist in the CI target environment
