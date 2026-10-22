---
name: feature-planner
description: Use this skill when planning a change before building it - a new feature, an improvement, a redesign, or the next lap after the user tried a test build. Activate when the user mentions planning a feature, planning a change or improvement, scoping work, writing an implementation spec, designing a feature before coding, or "plan [something]".
---

# Feature Planner

Plans one improvement at a time, builds it, and proves it works before handing it to the user. This is the first half of the build-test-try loop:

```
plan -> approve -> tests first -> build -> Claude tries the running app -> /wrap-up -> you try it -> next lap
```

The user's job is to define what "done" means and judge the result. Claude's job is to build against that definition and check its own work in the real app before saying it is done. Without a check, an agent just declares itself done.

## When to Use This Skill

- User says "plan feature", "plan [change]", "improve [thing]", or "design [feature]"
- User wants to understand scope before implementing
- Starting the next lap after feedback from a test build (see `workflow/task-tracker`, "After You Tried It")
- Before any change that touches more than a line or two

## Keep Each Lap Small

One improvement per plan. If the request contains several independent changes, propose splitting them into separate laps and plan the most important one first. Small laps get tried sooner, and a wrong assumption costs less.

## Instructions

### Step 1: Read the Context

1. Read `project/projectbrief.md` for the product and stack
2. Read `project/dev-context.md`, especially `## Must Keep Working` and `## Deployment`
3. Search for the existing code this change touches

**If dev-context.md has no `## Must Keep Working` section**, ask once:

```
Before we plan: what must keep working every time we change something?
Common examples: translations into all supported languages, accessibility
(screen readers, keyboard), dark mode, phone and desktop widths, existing
features on this screen.
```

Create the section in dev-context.md from the answer (format in `templates/dev-context-template.md`). If the user says "nothing yet", create it with a note saying so, and do not ask again.

### Step 2: Capture the Intent

Get three things before drafting anything:

1. **What to improve, and what good looks like.** In the user's words. "The widget looks cropped and only supports two sizes; it should look good at every size" is a usable intent.
2. **References, when they would help.** Ask for them if missing and the change is visual or uses an unfamiliar API:
   - Screenshots of the current state
   - Examples the user likes (another app, a site, a design)
   - Links to official documentation for the platform or API involved. Fetch and read them. Official docs beat a general web search.
3. **The product questions that change the outcome.** Ask the few that matter (use AskUserQuestion when available), for example "which sizes should this support?" or "crop images or keep them whole?". Do not ask technical questions; decide those yourself.

### Step 3: Create the Branch

If not already on a feature branch for this work:

1. Derive the branch name from the intent ("widget redesign" -> `update/widget-redesign`), using the prefix rules in `workflow/create-branch`
2. Resolve the base branch with `workflow/create-branch` Step 6 (Deployment config, else staging, else ask, else main)
3. Create the branch from that base

Do not send the user to `/branch` first. The plan is the task; there may not be a tasks.md entry yet.

### Step 4: Draft the Plan

Use this structure. Leave out sections that do not apply to this change.

```
PLAN: [Change Name]
Branch: [prefix]/[name]

## Intent
[What is being improved and what good looks like, in the user's words]

## References
[Screenshots, examples, docs links used - omit if none]

## Design
How it looks and behaves:
- [Behaviour, layout, states: empty, loading, error]
[ASCII mockup if the layout changes - see format below]

How it fits the existing code:
- [Files and functions touched or added, and why there]

## Must Keep Working
[Items from dev-context.md that this change could affect, and how the plan respects each]

## Definition of Done

Automated tests (written before the code):
- [ ] [Plain-language statement that must be true]
- [ ] [Rejection case: what must be refused, and how]

Claude checks in the running app (with screenshots):
- [ ] [What Claude will look at, and in which variants - e.g. phone and desktop width, each language, light and dark]

You check on your device:
- [ ] [Feel, looks, wording - what only a person can judge]

## Only When Relevant
Components to install / Database changes / API endpoints / Dependencies needing approval

## Risks
- [Edge case, security or performance consideration]
```

### Step 5: Review and Approve

1. For any non-trivial plan, run `workflow/plan-review` before showing it
2. Present the plan and ask:
   - "Is this the right change, and does the Definition of Done match what you mean by done?"
   - Approval for any components or packages to install
3. Only proceed after explicit approval

The Definition of Done is where a misunderstanding surfaces cheaply. If the user reads a check and says "that's not what I meant", the plan was wrong and the code would have been wrong too.

After approval, add the change to `project/tasks.md` as In Progress, with the branch name and the Definition of Done (all three columns).

### Step 6: Write the Tests Before the Code

1. If the project has no test runner, use `tooling/test-harness` to set one up
2. Translate each automated-test item into a real test
3. Run them and confirm they **fail** for the right reason (the change does not exist yet)
4. Only then start implementing

A check that passes before the change is built is checking nothing. Skip this step only when the change has no automated-test items (purely visual work). Say so explicitly rather than inventing hollow tests.

### Step 7: Build Until Green

Implement against the plan. Done with this step when every automated test passes and nothing that passed before now fails.

Do not edit a test to make it pass. If a test turns out to be wrong, say so and get agreement on the change.

### Step 8: Try It in the Running App

Before saying the change is done, Claude uses the app itself and works through every "Claude checks" item. Tests prove the logic; this proves the experience.

**Pick the best tool available in this session:**

| Platform | Tool |
|----------|------|
| Web | Start the app (`workflow/dev-server`), then drive it with whatever browser automation the session has: the `run` skill, a browser extension or built-in browser, or the project's Playwright journeys |
| Expo / React Native | Expo MCP local capabilities: simulator screenshots, taps, logs (see `platform/expo/ai-tooling.md`) |
| Native iOS | Simulator screenshots via `xcrun simctl io booted screenshot`; taps only if the session has a tool that can drive the simulator |

**For each check:**

1. Put the app into the state the check describes, in each listed variant
2. Take a screenshot and compare it against the plan's Design section
3. If it is wrong, fix it, re-run the automated tests, and check again
4. Stop after 3 attempts on the same check and report what still fails

**Never claim a check you could not run.** If no tool can reach the app, or a variant cannot be produced, say exactly which checks were not verified and why. "I could not check the dark mode variant because the session has no browser tool" is a correct report. "Verified" without evidence is not.

**Show the evidence:** a short list of each check with pass/fail and the screenshots.

This is not the user's test. The user still tries the result on a real device after `/wrap-up`.

### Step 9: Hand Off

1. Add a "Try this" note to the task in tasks.md, built from the "You check on your device" items
2. Tell the user the lap is built and checked, and that `/wrap-up` will merge it to the test environment
3. If this lap revealed a rule that should apply to every future change ("every new button needs all translations"), offer to add it to `## Must Keep Working`

## Writing the Definition of Done

Sort honestly. Put something in the wrong column and the process either wastes effort or gives false confidence.

**Automated tests** - one correct answer, no judgment involved:

- Data ownership: user A cannot read, edit, or delete user B's records
- Auth boundaries: logged-out requests are refused, not redirected into a half-state
- Money and credits: the arithmetic, and that it cannot go negative or be spent twice
- Contracts: what the endpoint returns on success, and the exact shape on failure
- Persistence: what was saved is what comes back
- The rejection cases: bad input, missing fields, expired tokens, duplicate submissions

**Claude checks in the running app** - visible, but checkable without taste:

- The screen renders in every listed size, language, and theme without overflow or clipped text
- Each state appears when it should: empty, loading, error, success
- The flow can be completed end to end
- Nothing on the Must Keep Working list broke on the screens touched

**You check on your device** - depends on taste or context:

- Whether it looks and feels right
- Wording and tone
- Generated or AI-produced content quality
- Whether the flow makes sense to a real person

**Rule of thumb:** if breaking it would be invisible until a user complains, it belongs in automated tests. If it is visible but objective, Claude checks it. If it needs taste, the user checks it.

**Write every item in plain language.** "A logged-out visitor cannot see another user's campaigns" is reviewable by anyone. `expect(res.status).toBe(403)` is not.

**List variants only when relevant.** A change to a settings form does not need every screen size and language. A change to a widget layout does.

## ASCII Mockup Format

```
+------------------------------------------+
|  Logo            Nav    Nav    [Avatar]  |
+------------------------------------------+
|          |                               |
| Sidebar  |     Main Content Area         |
| - Item   |   +---------------------+     |
| - Item   |   | Card Component      |     |
|          |   +---------------------+     |
|          |   [Primary Button]            |
+------------------------------------------+
```

`+--+` borders, `|` separators, `[ ]` buttons, `- Item` list items, indent for nesting.

## Component Handling Rules

Never install UI components or packages without user consent.

1. Check existing components first (`components/ui/`)
2. List which exist and which need installing
3. Mention that the user may have premium alternatives
4. Wait for approval before running any install command

## Examples

**User:** "The home widget looks bad - the image is cropped and we only support two sizes. Plan an improvement."

**Claude:**
1. Reads projectbrief and dev-context; Must Keep Working lists translations and accessibility
2. Asks for a screenshot of the current widget, an example the user likes, and the official widget-size docs; reads the docs
3. Asks: "Which sizes should we support?" and "Crop images or keep them whole?"
4. Creates `update/widget-redesign` from staging
5. Drafts the plan: Design covers every size and the image treatment; Definition of Done has a test for the size-to-layout logic, Claude checks for each size in each language, and a device check for how it feels
6. Runs plan-review, then gets approval
7. Writes the failing test, builds until green
8. Opens the simulator, screenshots every size in every language, fixes a clipped title at the smallest size, re-checks
9. Reports all checks with screenshots and adds the "Try this" note. Next step: `/wrap-up`

---

**User:** "Plan the stripe integration"

**Claude:**
1. Suggests splitting it into laps: checkout first, then webhooks and receipts
2. Plans the checkout lap: Stripe SDK needs approval; automated tests cover the price arithmetic and that a failed payment is refused; Claude checks the checkout flow in the browser at phone and desktop width; the user checks it feels trustworthy
3. Waits for approval before any installation

## How to Verify

- The plan has a three-column Definition of Done written in plain language
- Automated tests were seen failing before the code and passing after
- Every "Claude checks" item has a screenshot or an explicit "not verified, because..."
- The task in tasks.md has the Definition of Done and a "Try this" note
