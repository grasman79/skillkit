# AI Tooling for Expo

Expo ships first-party tooling that lets an AI agent build, debug, and validate Expo apps more autonomously. This file covers the Expo-specific technical pieces: the Expo MCP, `expo doctor`, and the Expo skills registry. Generic agent-workflow hygiene (context management, threading conversations) is out of scope - that belongs to your general Claude Code setup, not this skill.

Source: Expo official docs (expo.dev).

## Run `expo doctor` Before Debugging

Before burning time (and tokens) chasing a build failure, run:

```bash
npx expo-doctor
```

It flags version mismatches between your installed packages and what the Expo SDK expects (a common one: `react-native-reanimated` expected-vs-found). Many "mysterious" build errors are just a doctor-detectable mismatch. Run it first, fix what it reports, then debug anything that remains.

## Expo MCP

The Expo MCP connects an agent to your Expo account and project so it can read real data (build logs, project state) and act on it. It is free for everyone. Install from `expo.dev/ai` (a script per agent, or the Expo plugin which bundles both the MCP server and Expo's skills). After install, authenticate via `/mcp` in Claude Code.

**Rule of thumb:** if you can do it in the Expo dashboard, the MCP can probably do it too. The highest-value workflow is autonomous build-failure debugging - the agent reads the failing build's logs from the dashboard, diagnoses, fixes, and verifies, without you copy-pasting logs.

### Server vs local capabilities

The MCP exposes two kinds of tools:

- **Server capabilities** (installed by default) - anything that talks to the Expo dashboard/cloud: reading build logs, project and build info, etc.
- **Local capabilities** (opt-in) - control your local simulator. These require an extra dev dependency plus a start-command change:
  1. Install the local-tools dependency (adds it to `devDependencies`).
  2. Replace the `start` script in `package.json` with the MCP-enabled start command, then run it.

  Local tools let the agent take **screenshots**, **tap/click** in the simulator, read **native and JavaScript logs**, open dev tools, and inspect expo-router sitemaps - the pieces needed to actually validate that a change works, not just that it compiles.

### Cost awareness

MCP servers load **all** their tool definitions into context whether or not you use them, so they are heavier than skills. Only connect the MCPs a project actually needs - e.g. add the Supabase MCP only if the project uses Supabase; do not install it globally. Keep the Expo MCP scoped to Expo projects.

## Expo Skills Registry

Expo publishes agent skills at `expo.dev/skills`. Install the ones you need (or all with a wildcard), choosing the coding agent and scope (keep them local to a project unless you build Expo apps daily). A skill is just a `SKILL.md` (optionally with reference docs and scripts) that the agent reads on demand.

Notable: the **Expo UI skill** implements the native iOS liquid-glass effect on bottom tabs and modal buttons correctly in one shot. (This skill's `advanced-router.md` also covers native tab bars with liquid-glass support - see the Expo UI section of the main `SKILL.md` for the SwiftUI/Jetpack Compose native-component layer.)

## How to Verify

- `npx expo-doctor` runs clean (no version mismatches) before you debug a build.
- Expo MCP authenticated (`/mcp` shows it connected); server tools can read your latest build's logs.
- If you need simulator validation, local capabilities are installed (dev dependency present, start command swapped) and the agent can screenshot/tap.
- Only project-relevant MCPs are connected, not a global pile.
