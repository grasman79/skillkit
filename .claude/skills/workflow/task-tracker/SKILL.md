---
name: task-tracker
description: Use this skill when tracking project tasks, checking progress, or deciding what to work on next. Activate when the user mentions task status, project progress, what to do next, marking tasks complete, or viewing the current task list.
---

# Task Tracker

Keeps track of what you're building - what's done, what you're working on, what's coming next, and what's blocked.

## When to Use This Skill

- User asks about project status or progress
- User asks "what should I work on" or "what's next"
- User says a task is done or finished
- User asks about blocked tasks or dependencies
- User wants to see their task list
- User reports what they saw after trying a change on staging or a test build ("I tried it", "on my phone", "feedback on the last change")

## Instructions

### Show Status

**Triggers:** "status", "progress", "how am I doing", "where are we"

1. Read `project/tasks.md` to find current task and progress
2. Calculate completion percentage from checkboxes
3. Run `git status` for uncommitted changes
4. Show summary:

```
PROJECT STATUS

Current Task: User Authentication
Progress: 60% (3/5 items complete)

Completed:
- [x] Login form UI
- [x] Password validation
- [x] Database schema

Remaining:
- [ ] OAuth integration
- [ ] Session management

Next up: Dashboard Layout (after this task)
```

### Suggest Next Task

**Triggers:** "what's next", "what should I do", "suggest a task"

1. Read `project/tasks.md` for pending tasks
2. Check `project/projectbrief.md` for feature dependencies
3. Find tasks with all dependencies completed
4. Sort by priority (high → medium → low)
5. Recommend with reasoning:

```
NEXT TASK RECOMMENDATION

Recommended: Dashboard Layout

Why this task?
- All dependencies complete (Auth is done)
- High priority
- Natural follow-up to authentication work

Alternatives:
1. User Settings - medium priority, small scope
2. Email Notifications - medium priority, can do in parallel

Would you like to start "Dashboard Layout"?
```

### Mark Task Complete

**Triggers:** "done", "finished", "completed", "I finished [task]"

1. Find task in `project/tasks.md` (use task name if provided)
2. Update status to "complete" and check all boxes
3. Check what tasks are now unblocked
4. Suggest next task:

```
TASK COMPLETED!

Marked complete: User Authentication

Now unblocked:
- Dashboard Layout (was waiting on auth)
- User Settings (was waiting on auth)

Suggested next: Dashboard Layout
- High priority
- Natural continuation

Start working on Dashboard Layout?
```

### After You Tried It

**Triggers:** "I tried it", "tried it on my phone", "on staging", "here's what I saw", "feedback", or a screenshot from a test build

This closes the loop: what the user notices on a real device becomes the next small lap. It does not write code. The next lap goes through `workflow/feature-planner`.

1. Find the task with a "Try this" note that the user is talking about (the most recent one if unclear)
2. Split what the user said into separate findings. Ask for a screenshot only if a finding cannot be understood without one
3. For each finding, add a small task to `project/tasks.md`, linked back to the tried task:
   ```markdown
   ## Task N: [Short finding]
   **Status:** pending
   **Priority:** [High if it blocks going live, else Medium]
   **Tags:** #lap [relevant tags]
   **From:** Task M ([tried task name]), tried on [date]

   ### What was seen
   [The user's words]
   ```
4. Update the tried task:
   - User is happy with it -> mark it complete
   - Findings must be fixed before users see it -> keep it In Progress and note "blocks go-live"
5. If a finding is really a standing rule ("translations are broken again"), offer to add it to `## Must Keep Working` in dev-context.md
6. Recommend the next lap:

```
GOT IT

Tried: Widget redesign
- Works: sizes switch correctly, queue shows up
- Found: image aspect ratio looks off for portrait articles

Added:
- Task 12: Fix portrait image aspect ratio in widget (High - blocks go-live)

Widget redesign stays In Progress until that's fixed.
Next lap: plan the aspect ratio fix?
```

If the user is happy with everything on staging, suggest `/go-live`.

### Show Blocked Tasks

**Triggers:** "blocked", "what's blocking", "dependencies"

1. Find tasks with status "blocked" or unmet dependencies
2. Show what's blocking each and blocker progress
3. Recommend which blocker to prioritize:

```
BLOCKED TASKS

3 tasks waiting on dependencies:

1. Payment Integration
   Blocked by: User Settings (40% complete)
   Needs: User profile with billing info

2. Email Notifications
   Blocked by: Dashboard Layout (not started)
   Needs: Notification preferences UI

Recommendation:
Focus on User Settings to unblock Payment Integration

Available now: 4 tasks (ask "what's next" to see them)
```

## Examples

**Checking Progress:**
```
User: "How's the project going?"

Claude: "PROJECT STATUS

Current Task: User Authentication
Progress: 60% (3/5 items complete)
..."
```

---

**Completing a Task:**
```
User: "I finished the login page"

Claude: "TASK COMPLETED!

Marked complete: User Authentication

Now unblocked:
- Dashboard Layout
- User Settings

Suggested next: Dashboard Layout"
```

---

**Finding Next Work:**
```
User: "What should I work on?"

Claude: "NEXT TASK RECOMMENDATION

Recommended: Dashboard Layout

Why? All dependencies complete, high priority..."
```
