---
name: feature-planner
description: Use this skill when creating detailed implementation plans for new features before building. Activate when the user mentions planning a feature, scoping work, writing an implementation spec, architecting a solution, or designing a feature before coding.
---

# Feature Planner

Creates comprehensive implementation plans before you start building. Ensures you understand the full scope, get approval on UI components, and have a clear roadmap.

## When to Use This Skill

- User says "plan feature", "plan [feature name]", or "design feature"
- User wants to understand scope before implementing
- User mentions starting work on a new feature
- Before any significant new functionality

## Instructions

### Step 0: Create Feature Branch

1. Derive branch name from feature description:
   - "user settings page" -> `feature/user-settings`
   - "stripe integration" -> `feature/stripe-integration`
2. Run `git checkout -b feature/[name]`
3. Confirm branch created before proceeding

### Step 1: Understand the Feature

1. Read `project/projectbrief.md` for project context and existing patterns
2. Ask clarifying questions if needed:
   - What specific functionality is needed?
   - Who is this for (user type)?
   - Are there related features already built?
3. Search for existing code that might be relevant

### Step 2: Generate the Plan

Present the plan using this structure:

```
FEATURE PLAN: [Feature Name]
Branch: feature/[name]

## Overview
[One paragraph describing what this feature does and why]

## User Stories
- As a [user type], I want [action] so that [benefit]
- As a [user type], I want [action] so that [benefit]

## UI Mockup
[ASCII representation - see format below]

## Required Components

| Component | Status | Source |
|-----------|--------|--------|
| Button | Exists | @/components/ui/button |
| Dialog | Needs install | shadcn/ui (or your premium alternative) |
| DataTable | Needs install | shadcn/ui (or your premium alternative) |

**Action needed:** Which components should I install? You may have premium alternatives you'd prefer to use.

## Implementation Steps

1. [ ] Step one description
2. [ ] Step two description
3. [ ] Step three description
...

## Database Changes

| Table | Change | Fields |
|-------|--------|--------|
| users | Modify | Add `settings` JSON column |
| preferences | Create | id, userId, theme, notifications |

## API Endpoints

| Method | Route | Purpose |
|--------|-------|---------|
| GET | /api/settings | Fetch user settings |
| PATCH | /api/settings | Update user settings |

## Dependencies

| Package | Purpose | Approval |
|---------|---------|----------|
| zod | Validation | Pending |

**Action needed:** Approve packages before installation.

## Acceptance Checks

**Machine-checked** (written before any implementation code):
- [ ] [Plain-language statement of something that must be true]
- [ ] [Rejection case: what must be refused, and how]
- [ ] [Boundary case: who must NOT be able to reach this]

**Human-judged** (you decide by looking):
- [ ] [Anything about feel, taste, wording, or visual result]

## Risks & Considerations

- [Edge case or security consideration]
- [Performance consideration]
- [Dependency on other features]
```

### Step 3: Get Approval

After presenting the plan:

1. **Ask about components:** "Which components should I use? You may have premium alternatives."
2. **Ask about packages:** "Should I install these dependencies?"
3. **Confirm the acceptance checks:** "These are the things I'll prove are true before calling this done. Is anything missing or wrong?"
4. **Confirm scope:** "Does this plan cover everything you need?"

Point 3 matters most. The acceptance checks are where a misunderstanding surfaces cheaply, before any code exists. If the user reads a check and says "no, that's not what I meant", the plan was wrong and the code would have been wrong too.

Only proceed to implementation after explicit approval.

### Step 4: Write the Checks Before the Code

Once the plan is approved, the approved acceptance checks become real, failing checks **before** any implementation code is written.

1. If the project has no test runner, use `tooling/test-harness` to set one up
2. Translate each machine-checked item into a real test
3. Run them and confirm they **fail** for the right reason (feature does not exist yet)
4. Only then start implementing

A check that passes before the feature is built is checking nothing. Confirming the red state is what proves the check is wired to the thing it claims to cover.

Skip this step only when every acceptance check is human-judged. Some work is purely visual and has nothing a machine can assert. Say so explicitly rather than inventing hollow tests to satisfy the process.

### Step 5: Build Until Green

Implement against the checks. The feature is done when every machine-checked item passes and nothing that passed before now fails.

Do not edit a check to make it pass. If a check turns out to be wrong, say so and get agreement on the change. Silently relaxing an assertion is how a suite becomes decorative.

### Step 6: Create Task Entry

After approval, add the feature to `project/tasks.md`:

```markdown
## Task N: [Feature Name]
**Status:** pending
**Priority:** [High/Medium/Low]
**Tags:** #feature [relevant tags]
**Branch:** feature/[name]

### Checklist:
[Convert implementation steps to checkboxes]

### Implementation Notes:
[Link back to this plan discussion if needed]

### Acceptance Checks:
[Copy from plan - keep the machine-checked / human-judged split]
```

## ASCII Mockup Format

Use this style for UI mockups:

```
+------------------------------------------+
|  Logo            Nav    Nav    [Avatar]  |
+------------------------------------------+
|          |                               |
| Sidebar  |     Main Content Area         |
| - Item   |                               |
| - Item   |   +---------------------+     |
| - Item   |   | Card Component      |     |
|          |   | - Content here      |     |
|          |   +---------------------+     |
|          |                               |
|          |   [Primary Button]            |
+------------------------------------------+
```

**Symbols:**
- `+--+` for borders/containers
- `|` for vertical separators
- `[ ]` for buttons
- `- Item` for list items
- Indent for nesting

## Component Handling Rules

**Critical:** Never install UI components without user consent.

1. **Check existing components first** - Scan `components/ui/` for what's already installed
2. **List all needed components** - Show which exist vs. need installation
3. **Mention premium alternatives** - User may have paid component libraries
4. **Wait for approval** - Do not run any install commands until confirmed

Example prompt:
```
These components need to be installed:
- Dialog (shadcn/ui default, or your premium alternative)
- DataTable (shadcn/ui default, or your premium alternative)

Which would you like me to install?
```

## Writing Acceptance Checks

The value of this step comes entirely from sorting honestly. Put something in the wrong column and the process either wastes effort or gives false confidence.

**Machine-checked** - one correct answer, no judgment involved:

- Data ownership: user A cannot read, edit, or delete user B's records
- Auth boundaries: logged-out requests are refused, not redirected into a half-state
- Money and credits: the arithmetic, and that it cannot go negative or be spent twice
- Contracts: what the endpoint returns on success, and the exact shape it returns on failure
- Persistence: what was saved is what comes back
- Names that must agree across places: config, database, deployed infrastructure
- The rejection cases: bad input, missing fields, expired tokens, duplicate submissions

**Human-judged** - depends on taste or context:

- Visual design, spacing, motion, whether it feels right
- Wording and tone
- Generated or AI-produced content quality
- Whether the flow makes sense to a real person

**The rule of thumb:** if breaking it would be invisible until a user complains, it belongs in the machine column. Silent breakage is exactly what a person reviewing by eye will miss, and exactly what a check catches for free forever.

**Write them in plain language first.** The user approving the plan may not read code. "A user who is not logged in cannot see another user's campaigns" is reviewable by anyone. `expect(res.status).toBe(403)` is not.

## Examples

**User:** "Plan feature: user settings page"

**Claude:**
1. Creates branch `feature/user-settings`
2. Reads project/projectbrief.md for context
3. Generates full plan with:
   - Overview of settings functionality
   - User stories for settings access
   - ASCII mockup of settings layout
   - Component list (existing + needed)
   - Implementation steps
   - Database changes for preferences
   - API endpoints
   - Acceptance checks, split into machine-checked and human-judged
4. Asks for component approval and confirms the acceptance checks read correctly
5. Writes the machine-checked items as failing tests, confirms they fail
6. Builds until they pass, then creates the task entry in project/tasks.md

---

**User:** "Plan the stripe integration"

**Claude:**
1. Creates branch `feature/stripe-integration`
2. Generates plan including:
   - Payment flow overview
   - User stories for checkout
   - ASCII mockup of payment UI
   - Components needed (pricing cards, checkout form)
   - Stripe SDK dependency (requires approval)
   - Webhook endpoints
   - Security considerations
3. Waits for approval before any installations
