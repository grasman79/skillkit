---
name: react-doctor
description: Use this skill when setting up React Doctor for code quality scanning, health scoring, or linting in any React project - web or mobile. Covers CLI scans, ESLint integration, CI gating, and framework-specific configs for Next.js, TanStack Start, React Native, and Expo. Trigger words - react doctor, react-doctor, code health score, react health scan, react linting, react quality scan, install react doctor, configure react doctor, react doctor config, react native linting.
---

# React Doctor

Deterministic code health scanner for React projects. Produces a 0-100 health score and actionable diagnostics across security, performance, correctness, architecture, accessibility, and bundle size.

Works for: Next.js, Vite, TanStack Start, React Native, Expo - any React stack.

## When to Use This Skill

- Setting up code quality tooling on a new React project
- Adding health scoring to CI/CD pipelines
- Integrating React-aware linting into ESLint config
- Installing as a coding agent skill for ongoing feedback
- Configuring framework-specific rules (RN, Next.js, TanStack)

## Quick Scan (No Install)

```bash
npx react-doctor@latest
```

Runs a one-off scan, outputs health score + diagnostics. Good for auditing existing projects.

## Install as Agent Skill (Claude Code / Cursor)

```bash
npx react-doctor@latest install
```

Adds React Doctor as a persistent coding agent skill - gives real-time feedback during development.

## ESLint Integration (Ongoing Linting)

### Install

```bash
# npm
npm install --save-dev eslint-plugin-react-doctor

# bun
bun add -d eslint-plugin-react-doctor
```

### Configure (`eslint.config.js`)

**Web React (Vite / TanStack Start)**
```js
import reactDoctor from "eslint-plugin-react-doctor";

export default [
  reactDoctor.configs.recommended,
  reactDoctor.configs["tanstack-start"], // if using TanStack Start
  reactDoctor.configs["tanstack-query"],  // if using TanStack Query
];
```

**Next.js**
```js
import reactDoctor from "eslint-plugin-react-doctor";

export default [
  reactDoctor.configs.recommended,
  reactDoctor.configs.next,
];
```

**React Native / Expo**
```js
import reactDoctor from "eslint-plugin-react-doctor";

export default [
  reactDoctor.configs.recommended,
  reactDoctor.configs["react-native"],
];
```

**All rules (strictest)**
```js
export default [reactDoctor.configs.all];
```

### Override Individual Rules

```js
export default [
  reactDoctor.configs.recommended,
  {
    rules: {
      "react-doctor/no-fetch-in-effect": "error",
      "react-doctor/no-derived-state-effect": "off",
    },
  },
];
```

## CLI Config (`react-doctor.config.json`)

Controls which rules fire when running `npx react-doctor@latest`.

```json
{
  "ignore": {
    "rules": [
      "react-doctor/rule-to-disable"
    ],
    "overrides": [
      {
        "files": ["src/routes/**/*.tsx"],
        "rules": ["react-doctor/only-export-components"]
      }
    ]
  }
}
```

Place at project root. See `reference/FRAMEWORKS.md` for per-framework rule suppression lists.

## CI / GitHub Actions

```yaml
name: React Doctor
on: [pull_request]
jobs:
  health:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npx react-doctor@latest
```

**Gate merges on score threshold:**
```yaml
- run: npx react-doctor@latest --min-score 80
```

**Post results as PR comment:**
```yaml
- run: npx react-doctor@latest --pr-comment
```

## Rule Categories (376 total)

| Category | Rules | Focus |
|----------|-------|-------|
| Performance | 66 | Memoization, animations, rendering |
| Correctness | 56 | Runtime safety, hydration, lifecycle |
| State & Effects | 50 | Anti-patterns, stale closures, derived state |
| Architecture | 47 | Code organization, duplication, type safety |
| Accessibility | 40 | ARIA, keyboard nav, semantic HTML |
| React Native | 28 | Animated, lists, gestures, Reanimated |
| React Compiler | 13 | Purity, ref safety, auto-memo compat |
| Next.js | 12 | Image, metadata, server/client boundaries |
| TanStack Start | 9 | Router, server functions, loaders |
| Dead Code | 10 | Unused exports, circular deps |
| Bundle Size | 7 | Tree-shaking, direct imports |
| Security | 7 | XSS, CSRF, env leaks |
| Server | 7 | Auth checks, cache keys, request isolation |
| TanStack Query | 5 | Cache invalidation, mutation semantics |
| Preact | 5 | API differences, event naming |

## Framework Notes

### React Native / Expo
- Use `reactDoctor.configs["react-native"]` - this is a first-class config, not a workaround
- The 28 RN-specific rules cover: GPU-friendly transforms, list virtualization, gesture handling, Reanimated integration
- Web-only rules (hydration, anchor tags, iframes, server components) do not fire when using the RN config
- See `reference/FRAMEWORKS.md` for the full suppression list if using the CLI scanner

### Next.js
- Use `reactDoctor.configs.next` for server component rules, metadata API, Image optimization
- The 7 server rules and 12 Next.js rules apply in addition to recommended

### TanStack Start
- Use `reactDoctor.configs["tanstack-start"]` + `reactDoctor.configs["tanstack-query"]`
- Covers type-safe RPC, loader coordination, server boundaries

## Verify It's Working

```bash
# Run scan and check score
npx react-doctor@latest

# Check ESLint integration
npx eslint src/ --ext .ts,.tsx
```

Look for `react-doctor/*` rule violations in ESLint output.
