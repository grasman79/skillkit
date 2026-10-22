---
description: End a lap - log, lint, test, commit, push, PR, review, merge to staging, hand over a "Try this" list
---

Follow the wrap-up workflow to close out this lap.

## Instructions

1. Read `.claude/skills/workflow/wrap-up/SKILL.md` for the full workflow

2. Steps performed:
   - Log session progress (changelog + tasks)
   - Run linter, tests, and build
   - Confirm the lap was tried in the running app
   - Commit, push, and create a PR into the test branch (staging)
   - Code review: local reviewer agent (or Greptile if configured)
   - Merge into the test branch and confirm a testable version exists
   - Clean up branches

3. Hand the user a "Try this" list. Never go live from here - that is `/go-live`
