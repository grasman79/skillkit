---
description: Release everything on staging to production, after showing what will go live and getting a yes
---

Follow the Go Live section of the wrap-up workflow.

## Instructions

1. Read `.claude/skills/workflow/wrap-up/SKILL.md`, section "Go Live"

2. Steps performed:
   - List the laps on the test branch that are not live yet, flagging any the user has not tried
   - Ask for explicit confirmation
   - Web: release PR from the test branch to main, merge, confirm the production URL loads
   - Mobile: hand off to the app store submission flow, confirming before submitting
   - Log the release and mark released tasks done

3. Report what went live
