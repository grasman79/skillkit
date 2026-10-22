---
name: wrap-up
description: Use this skill when ending a session or a lap to log progress, lint, test, commit, push, create a PR, have it reviewed, merge it into the test environment, and hand the user a "Try this" list. Also handles going live (/go-live). Activate when the user mentions /wrap-up, wrapping up, shipping work to staging, ending the session, pushing and merging completed work, /go-live, going live, releasing to production, or promoting staging to main.
---

# Wrap Up

Single command to close out a lap. Logs progress, lints, tests, commits, pushes, opens a PR, has a fresh reviewer check it, merges it into the **test environment**, and ends with a short "Try this" list so you can try the result yourself.

`/wrap-up` never puts a change live. Going live is a separate step (`/go-live`, see the end of this skill), so several laps can collect on the test environment and you decide when they reach your users.

```
plan -> build -> Claude tries it -> /wrap-up (merge to test) -> you try it -> next lap ... -> /go-live
```

## When to Use This Skill

- User says "wrap up", "wrap-up", "done for today", or "ship it to staging"
- A lap from `workflow/feature-planner` is built and checked
- Session is ending and user wants to save, commit, and push everything
- User says "/go-live", "go live", "release", or "push staging to production" (see Go Live)

## What This Does

1. Log the session (changelog + tasks)
2. Run linter and fix issues
3. Run the test suite (blocking - a failing suite stops the wrap-up)
4. Verify build passes
5. Check the lap was tried in the running app (flag, not blocker)
6. Check branch name matches the work (warn if mismatch)
7. Commit all changes and push
8. Create PR into the test branch
9. Code review: a fresh local reviewer agent by default, or Greptile if configured
10. Merge into the test branch
11. Make sure a testable version exists (staging URL, or a mobile test build)
12. Clean up and hand you a "Try this" list

## Workflow

### Step 1: Detect Package Manager

Check `project/dev-context.md` for a `## Package Manager` section to determine the runner command:

| Package Manager | Runner |
|-----------------|--------|
| bun | bunx |
| pnpm | pnpm dlx |
| npm | npx |
| yarn | yarn dlx |

If not specified, check for lock files (`bun.lockb`, `pnpm-lock.yaml`, etc.). Default to `bunx` if nothing found.

### Step 2: Log the Session

Review the conversation and identify:
- Features built or modified
- Bugs fixed
- Tasks completed
- Key decisions made

**Update `.claude/project/changelog.md`** - Add entry at the top:

```markdown
## [YYYY-MM-DD] - [Brief Session Title]

### Completed
- [What was done]

### Key Decisions
- [Important decisions, if any]

### Next Steps
- [What to work on next]
```

**Update `.claude/project/tasks.md`** - Update in-progress items. A lap that is merged to the test environment stays In Progress with its "Try this" note until the user has tried it (see `workflow/task-tracker`, "After You Tried It").

**Update dev-context.md** (if features are still in progress) - Capture state for next session.

### Step 3: Update README Date

If a `README.md` exists in the project root with a date field, update it to today's date.

### Step 4: Run Linter

```bash
[runner] ultracite fix
```

If there are errors that can't be auto-fixed, fix them manually before continuing. Do not skip or bypass the linter.

### Step 5: Run the Test Suite

If `package.json` has a `test` script:

```bash
[package-manager] run test
```

**A failing suite stops the wrap-up.** Fix the failures before continuing. Never commit over a red suite, and never report the session as done while tests fail.

If a check fails because the behaviour deliberately changed this session, update the check and say so plainly in the summary. Deleting or skipping a check to get green is not allowed. If a check genuinely needs to go, that is a decision to raise with the user, not to make silently.

**If there is no `test` script**, do not block. Note it once in the summary:

```
No test suite in this project. Ran lint and build only.
```

**If the session added a feature and wrote no checks for it**, say so in the summary rather than staying quiet:

```
Note: [feature] shipped without acceptance checks. Worth adding before this grows.
```

This is a flag, not a blocker. Prototype work legitimately ships without checks. The point is that the user knows, rather than assuming coverage that does not exist.

### Step 6: Verify Build (if applicable)

If the project has a build command in package.json:

```bash
[package-manager] run build
```

If build fails, fix the errors before committing. Do not push broken code.

### Step 7: Check the Lap Was Tried in the Running App

If this lap came from a plan with "Claude checks in the running app" items (`workflow/feature-planner` Step 8), confirm they were run and their results reported this session.

If they were not run, run them now if the tools are available. If they cannot be run, do not block, but say so in the summary:

```
Note: Claude did not try this in the running app ([reason]). Check these yourself on staging: [list]
```

### Step 8: Branch Relevance Check

**If on a feature branch**, before committing, verify the branch name relates to the work being committed:

1. Get the current branch name: `git branch --show-current`
2. Review the changes being committed
3. Compare the branch name against the actual changes

**If the branch name does NOT match the work being done** (e.g., branch is `feature/add-payments` but changes are about authentication), warn the user:

```
Warning: Current branch "feature/add-payments" doesn't seem related to the changes being committed (authentication updates).

Options:
1. Create a new branch (recommended) - stash changes, create a properly named branch, and continue
2. Continue on this branch anyway
3. Abort wrap-up
```

**If option 1:** Stash changes, create a new branch from the test branch, pop stash, and continue the workflow on the new branch.

**If on main/staging:** Skip this check.

### Step 9: Commit and Push

```bash
git add .
git status
```

Review what's being committed. Then generate a commit message based on all changes:

```bash
git commit -m "$(cat <<'EOF'
[Type]: Brief description

- Detail 1
- Detail 2

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Commit types:** Feature, Fix, Update, Refactor, Chore

```bash
git push -u origin [current-branch]
```

### Step 10: Create PR (feature branches only)

**If on a feature branch** (not main/staging/master):

Resolve the PR target with the same rule `workflow/create-branch` uses (Step 6): `## Deployment` -> `PR target for feature branches` in dev-context.md, else `staging` if it exists on the remote, else ask if only `dev` exists, else `main` with a one-time warning that merges go live directly. Never default to a branch that does not exist on the remote.

Create the PR:

```bash
gh pr create --base [pr-target] --title "[Type]: Brief description" --body "$(cat <<'EOF'
## Summary
[What changed and why]

## Changes
- Change 1
- Change 2

## Definition of Done
[Results of the plan's checks, if this lap had a plan]

Created with Claude Code
EOF
)"
```

Note the PR number from the output.

**If on main/staging:** Skip PR creation, just push directly. Skip to Step 13.

### Step 11: Code Review

Check `project/dev-context.md` for a `## Code Review` section:

| Setting | Review |
|---------|--------|
| No section, or `Provider: Local` | Step 11a - local reviewer agent |
| `Provider: Greptile` | Step 11b - Greptile cycle only (no double review) |
| `Provider: None` | Skip review, go to Step 12 |

#### Step 11a: Local Reviewer Agent (default)

A reviewer in a fresh context catches what the builder no longer sees. It runs on your machine, costs nothing extra, and needs no setup.

1. Spawn one reviewer subagent with the Agent tool. It does not inherit this conversation, so its prompt must include:
   - The PR diff (`gh pr diff [pr_number]`)
   - The plan's Intent and Definition of Done, if this lap had a plan
   - The `## Must Keep Working` section from dev-context.md
   - The checklist from `workflow/code-reviewer`
   - The instruction: return concrete findings ranked Critical / Warning / Suggestion with `file:line`, no rewrites. "No material findings" is a valid answer.
2. **Vet every finding yourself** against the code. Reviewers over-report. Reject findings that are by-design, point at the wrong line, or duplicate each other.
3. Fix accepted Critical and Warning findings. Leave Suggestions for the summary.
4. Re-run lint, tests, and build. Commit (`Fix: Address review findings`) and push.
5. Maximum 2 review cycles. If Critical findings remain after that, stop and show them to the user instead of merging.

#### Step 11b: Greptile Review Cycle (optional)

**Greptile File Limit Check** - Greptile has a 100-file limit per review:

```bash
gh pr diff {pr_number} --name-only | wc -l
```

If the count exceeds 100, warn the user and offer: (1) use the local reviewer agent instead (Step 11a), or (2) cancel and split into smaller PRs.

**Poll for review** every 30 seconds, up to 10 minutes:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews
```

Look for a review from Greptile (check the `user.login` field for greptile or greptileai).

**If no Greptile review appears after 10 minutes:** fall back to the local reviewer agent (Step 11a). Show message: "Greptile review not detected - using the local reviewer instead."

**If Greptile review appears:** read the confidence score (e.g. 3/5, 4/5, 5/5) and all review comments:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
```

**If confidence score >= threshold (default 4/5):** go to Step 12.

**If confidence score < threshold:**

1. Read each Greptile comment and understand the issue
2. Fix the code based on the feedback
3. Run linter, tests, and build again
4. Commit (`Fix: Address code review feedback`) and push
5. Go back to the top of Step 11b and poll for a new review

**Maximum review cycles:** 3 attempts. If still below threshold after 3 fix cycles, stop and show the remaining comments to the user instead of merging.

### Step 12: Merge Into the Test Branch

```bash
gh pr merge [pr_number] --squash --delete-branch
```

If the repo requires status checks before merging, use `--auto` instead so it merges once they pass.

If the PR target is `main` and the project has no test branch (the user accepted this when warned), this merge goes live. Say so in the summary.

### Step 13: Make Sure a Testable Version Exists

The point of merging is that the user can try the change. Confirm there is something to try.

**Web:** the test branch auto-deploys (Railway staging environment, Cloudflare Workers preview URL). Find the staging URL in dev-context.md `## Deployment`, wait for the deploy, and confirm the URL loads. If it does not load within a few minutes, say so and show where to look (the platform's deploy logs).

**Mobile:** test builds are opt-in, because they take time, use build quota, and need store credentials. Check dev-context.md `## Deployment` for a `Mobile test builds:` line:

- **Not set:** ask once: "Should `/wrap-up` make a test build for your phone after merging (TestFlight internal testing on iOS, internal testing track on Android)?" Record the answer as `Mobile test builds: TestFlight internal` or `Mobile test builds: off`.
- **Set to TestFlight internal:** ask "Start a test build now?" before every build. If yes, follow `platform/expo` (EAS Build, then EAS Submit to TestFlight / internal track).
- **On any failure** (missing credentials, failed build, failed submit): stop and report the error and where to fix it. Never retry automatically; each retry spends build quota.
- **If the change is JavaScript-only** and the project uses EAS Update, an update to the internal channel is a faster alternative. Offer it; do not choose it silently.

### Step 14: Clean Up Local Branch

**If on a feature branch:**

```bash
git checkout [pr-target]
git pull origin [pr-target]
git branch -d [feature-branch-name]
```

**If on main/staging:** Skip this step.

### Step 15: Hand Over

End with what was done and what the user should try. The "Try this" list comes from the task's "You check on your device" items.

```
Lap wrapped up and on staging!

Code:
- Linter: passed
- Tests: X passed, 0 failed (or "no test suite")
- Build: passed (or N/A)
- Claude tried it: 4/4 checks passed (or what was not verified)
- Review: local reviewer - 2 findings fixed, 1 suggestion noted
- PR: #XX merged into staging

Try this:
- Staging: https://staging.yourapp.com (or: TestFlight build 35)
- [ ] [Device check 1]
- [ ] [Device check 2]

When you've tried it, tell me what you saw. Each issue becomes the next small lap.
When staging is ready for your users: /go-live
```

## Go Live

Triggered by `/go-live`, "go live", "release", or "push staging to production". Moves everything on the test branch to production.

### Step 1: Show What Will Go Live

```bash
git fetch origin
git log --oneline origin/main..origin/[test-branch]
```

Summarize the laps in plain language (use the changelog), and flag any lap whose task is still waiting for the user's device check:

```
Going live will release these changes to your users:
1. Widget redesign - all sizes (you tried it on 21/10)
2. Search filters (not tried yet on staging)

Go live now?
```

### Step 2: Get Explicit Confirmation

Never go live without a clear yes in this conversation. If any lap was not tried yet, recommend trying it first.

### Step 3: Release

**Web:**

```bash
gh pr create --base main --head [test-branch] --title "Release: [summary]" --body "[list of laps]"
gh pr merge [pr_number] --merge
```

Use a merge commit (not squash) so `main` and the test branch keep the same history. Then wait for the production deploy and confirm the production URL loads.

**Mobile:** hand off to `platform/app-store` for store assets and review readiness, and to `platform/expo` for submitting the tested build for store review. Confirm with the user before submitting.

### Step 4: Log It

Add a changelog entry `## [YYYY-MM-DD] - Release: [summary]` and mark the released tasks done.

## Error Handling

**Linter fails and can't auto-fix:**
Fix the issues before continuing. Show what needs fixing and resolve it.

**Tests fail:**
Stop. Fix the code, or fix the check if the behaviour changed on purpose and say so in the summary. Never skip, delete, or comment out a check to reach green.

**Build fails:**
Fix build errors before committing. Do not push broken code.

**Push rejected:**
```bash
git pull --rebase origin [branch]
git push origin [branch]
```

If conflicts exist, resolve them and retry.

**No changes to commit:**
```
No code changes to commit, but session was logged.

Changelog and tasks have been updated.
```

Skip the commit/push/PR steps.

**Merge blocked (branch protection or conflicts):**
Show the reason and the PR URL. Do not force-merge.

**Staging does not load after merge:**
Report it with the deploy log location. The lap is merged but not ready to try; say so rather than handing over a broken "Try this" list.

**gh CLI not available:**
Push the branch and provide the manual PR creation URL instead.

## Examples

**Wrap-up after a planned lap (local reviewer):**

```
User: "/wrap-up"

Claude: "Wrapping up...

Code:
- Linter: passed (2 auto-fixes applied)
- Tests: 14 passed, 0 failed
- Build: passed
- Claude tried it: 6/6 checks passed (screenshots above)
- PR: #47 into staging
- Review: local reviewer found a missing null check in the search handler - fixed
- Merged into staging, local branch cleaned up

Try this:
- Staging: https://staging.yourapp.com/contacts
- [ ] Search feels instant while typing
- [ ] Filter labels read naturally in German

When you've tried it, tell me what you saw."
```

---

**Wrap-up with Greptile configured:**

```
User: "/wrap-up"

Claude: "Wrapping up...

[... lint, tests, build, commit, push, PR #48 into staging ...]

Waiting for Greptile review...
Greptile review: 2/5 - 3 issues found. Fixing...
Greptile review: 5/5 - all issues resolved

Merged into staging. Try this:
- Staging: https://staging.yourapp.com
- [ ] ..."
```

---

**Go live:**

```
User: "/go-live"

Claude: "Going live will release these changes to your users:
1. Contact search with filters (you tried it on staging)
2. Sidebar collapse fix (you tried it on staging)

Go live now?"

User: "Yes"

Claude: "Release PR #49 merged into main. Production deploy finished and
https://yourapp.com loads. Changelog updated, both tasks marked done."
```

---

**Wrap-up with no code changes:**

```
User: "wrap up"

Claude: "Wrapping up your session...

Logged:
- Changelog updated (2026-02-11)
- Tasks: planning notes updated

No code changes to commit.

Next session: Start building the Products module"
```

## How to Verify

- The PR targets the test branch, never a branch that does not exist on the remote
- The review step ran (local agent or Greptile) and its findings were vetted, not applied blindly
- The summary ends with a "Try this" list and a working staging URL or build number
- Nothing reached `main` without `/go-live` and an explicit yes (unless the project has no test branch and the user accepted that)
