---
name: plan-review
description: Use this skill when a plan needs to be stress-tested by independent critics before implementation starts. Activate when the user mentions reviewing a plan, stress-testing a plan, red-teaming a plan, attacking a plan, poking holes in a plan, or as the automatic step between drafting a plan and asking for approval.
---

# Plan Review

Takes a drafted plan and dispatches fresh-context critic subagents to attack it - checking blast radius, over-engineering, and convention compliance - before the plan is presented for approval. Turns a solo draft into a hardened plan with a visible paper trail of what was challenged and fixed.

## When to Use This Skill

- Right after drafting any non-trivial plan (3+ steps, multiple files, or an architectural decision), before presenting it for approval
- User says "review the plan", "stress-test this", "red-team the plan", "attack this plan", or "poke holes in this"
- Automatically, as the step between a Plan Mode draft and calling `ExitPlanMode`
- On an existing plan file (e.g. from `workflow/feature-planner` or `workflow/improve`) that hasn't been challenged yet

## What This Skill Does NOT Do

- Never edits or implements code. Input is a plan; output is a hardened plan.
- Critic subagents return objections only - never fixes, never rewrites, never file dumps.
- Does not replace user approval. It happens before approval, not instead of it.

## Instructions

### Step 1: Take the Draft Plan as Input

The draft can be:
- A plan just written in this conversation
- A Plan Mode draft, before calling `ExitPlanMode`
- An existing `plans/*.md` file (e.g. from `workflow/feature-planner` or `workflow/improve`)

Have the full plan text in hand before dispatching critics.

### Step 2: Dispatch Critic Subagents (fresh contexts, in parallel)

Use the Agent tool to spawn one critic per lens, in parallel, in fresh contexts. **Subagents do not inherit this skill's context**, so every critic prompt must inline:

- The full draft plan text
- The relevant project rules (from `CLAUDE.md`): ask-don't-assume, simplest-solution-first, don't-touch-unrelated-code, verify-before-done
- Its single assigned lens (see Step 3) - one lens per critic, not all three at once
- An explicit instruction: return concrete, ranked objections only, with `file:line` references where relevant. No fixes, no rewrites, no file dumps. "No material objections" is a valid, complete answer.

### Step 3: Default Lenses (one critic each)

1. **Blast radius / what breaks** - existing behavior and files touched, hidden coupling, data/migration/rollback risk, what could regress.
2. **Over-engineering / simplicity** - is there a materially simpler solution, unnecessary abstraction or flexibility, scope creep beyond what was asked.
3. **Convention & principle compliance** - matches existing patterns and naming, respects the project's own rules, missing verification/tests/error handling, unstated assumptions.

### Step 4: Vet Before Accepting

**Subagents over-report - vet everything yourself.** For every objection a critic returns, confirm it against the actual code or plan before accepting it. Reject or downgrade:
- By-design behavior flagged as a problem
- Objections attributed to the wrong file, step, or line
- Duplicate objections across critics

Do not invent objections to look thorough. A lens returning nothing material is a good outcome, not a failed pass.

### Step 5: Fold in Accepted Objections

Tighten scope, add missing steps, simplify, or add verification directly into the plan based on what survived vetting.

### Step 6: Present the Hardened Plan

Show the revised plan plus a short **Review Summary**:

```
## Review Summary

| Objection | Lens | Resolution |
|-----------|------|------------|
| [what was flagged] | Blast radius / Simplicity / Convention | [how the plan changed] |

**Considered and rejected:** [objection] - [why it didn't hold up]
```

Then hand off to normal approval - in Plan Mode this is calling `ExitPlanMode`; otherwise, ask the user to confirm before implementation starts.

## Effort Levels

| | `quick` | `standard` (default) | `deep` |
|---|---|---|---|
| Critics | 1, combined on the highest-risk lens for this plan | 3 (one per lens above) | 3 lenses + a 4th "cold reader" critic |
| Cold reader | - | - | Reads the plan with zero context beyond the plan text itself and CLAUDE.md rules; reports ambiguities an executor with no session history would trip on |

Default to `standard` unless the user says `quick` or `deep`, or the plan's size/risk clearly calls for it.

## Composition

- `workflow/feature-planner` drafts a plan -> `plan-review` hardens it -> user approves.
- Plan Mode: explore and draft -> `plan-review` attacks the draft -> `ExitPlanMode` presents the hardened version.
- Standalone: point this skill at an existing `plans/*.md` file (e.g. from `workflow/improve`) that hasn't been challenged yet.
- Does not replace `workflow/feature-planner` or `workflow/improve` - it's the review step that runs on their output.

## Examples

**User:** "Plan feature: user settings page" (feature-planner drafts the plan) -> "Review the plan before we build it"

**Claude:**
1. Takes the feature-planner output as the draft plan
2. Dispatches 3 critics in parallel: blast radius, over-engineering, convention compliance
3. Vets their objections - e.g. accepts "the preferences table needs a migration rollback step," rejects "this duplicates an existing settings hook" after confirming no such hook exists
4. Folds the accepted objection into the plan's Implementation Steps
5. Presents the hardened plan with a Review Summary table, then asks for approval

---

**User:** (mid Plan Mode session) "Before you exit plan mode, red-team this"

**Claude:**
1. Takes the current Plan Mode draft as input
2. Dispatches critics with the full draft plan and project rules inlined
3. One critic flags that the plan touches a shared utility file with no mention of other callers - vetted and confirmed real
4. Adds a step to check other callers before editing the shared file
5. Calls `ExitPlanMode` with the hardened plan and Review Summary
