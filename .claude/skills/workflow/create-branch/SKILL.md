---
name: create-branch
description: Use this skill when starting a new task by creating a feature branch from the current task list. Activate when the user mentions /branch, creating a branch, starting a new task, or beginning work on a feature.
---

# Create Branch

Automatically creates a feature branch based on your current task in `tasks.md`. The branch name is generated from the task description with proper prefixes (feature/, fix/, etc.).

## When to Use This Skill

- User says "branch", "create branch", or "start task"
- Beginning work on a new task
- Need to switch from main/staging to a feature branch

## Workflow

### Step 1: Check Current Branch

```bash
git branch --show-current
```

If already on a feature branch (not main/staging/master), ask if they want to:
- Stay on current branch
- Create a new branch anyway
- Switch to a different existing branch

### Step 2: Find Current Task

Read `project/tasks.md` to find the task marked as "in progress" or the highest priority "pending" task.

**If no task found:**
```
No task found in tasks.md.

Would you like to:
1. Create a branch manually (provide a name)
2. Check task status first (/tasks)
```

**If task found:**
Show the task and suggest branch name based on it.

### Step 3: Determine Branch Type

Analyze the task description to determine branch prefix:

**Keywords → Prefix:**
- "add", "create", "build", "implement", "new" → `feature/`
- "fix", "bug", "error", "broken" → `fix/`
- "update", "improve", "enhance", "modify" → `update/`
- "refactor", "restructure", "reorganize" → `refactor/`
- "optimize", "performance", "speed up" → `optimize/`
- "security", "vulnerability", "secure" → `security/`
- Default → `feature/`

### Step 4: Generate Branch Name

From task name: "Add user authentication"
1. Remove common words: "Add", "Create", "Implement", "Fix"
2. Convert to lowercase
3. Replace spaces with hyphens
4. Remove special characters
5. Limit to 50 characters

**Examples:**
- "Add user authentication" → `feature/user-authentication`
- "Fix login redirect bug" → `fix/login-redirect-bug`
- "Update dashboard layout" → `update/dashboard-layout`
- "Refactor API calls" → `refactor/api-calls`

### Step 5: Check for Uncommitted Changes

```bash
git status --porcelain
```

**If changes exist:**
```
You have uncommitted changes:
- [list changed files]

What would you like to do?
1. Commit changes first (recommended)
2. Stash changes and create branch
3. Create branch anyway (changes come with you)
```

### Step 6: Resolve the Base Branch

Feature branches start from the **test branch** - the branch that deploys to the test environment. Resolve it in this order:

1. `project/dev-context.md` has `## Deployment` with `PR target for feature branches: [branch]` -> use that branch
2. Otherwise, `staging` exists on the remote (`git ls-remote --heads origin staging`) -> use `staging`
3. Otherwise, `dev` exists on the remote -> ask: "Is `dev` your test branch, where changes go before they go live?" Record the answer as `PR target for feature branches: [branch]` under `## Deployment` in dev-context.md so it is never asked again
4. Otherwise -> use `main`, and warn once:
   ```
   This project has no test branch, so merged changes go live directly.
   I recommend a `staging` branch that deploys to a test URL, so you can try
   changes before your users see them. Set it up now, or continue with main?
   ```
   Record the answer under `## Deployment` in dev-context.md.

`/wrap-up` uses the same rule for the PR target, so a branch always merges back where it started.

### Step 7: Create and Switch to Branch

```bash
git checkout [base-branch]
git pull origin [base-branch]
git checkout -b feature/branch-name
```

### Step 8: Confirm Creation

```
Branch created and checked out!

Branch: feature/user-authentication
Based on: staging
Task: Add user authentication

Ready to start working. When done, use /wrap-up to ship everything.
```

## Examples

**Starting a new task:**

```
User: "/branch"

Claude: "Looking at your tasks...

Current task (in progress): Add Stripe payment integration
Status: pending
Priority: High

I'll create a feature branch for this.

Suggested branch name: feature/stripe-payment-integration

Create this branch?"

User: "Yes"

Claude: "Branch created and checked out!

Branch: feature/stripe-payment-integration
Based on: staging
Task: Add Stripe payment integration

Ready to start working. When done, use /wrap-up to ship everything."
```

---

**Already on a feature branch:**

```
User: "/branch"

Claude: "You're currently on: feature/auth-system

This is already a feature branch. Would you like to:
1. Stay on this branch and continue working
2. Create a new branch for a different task
3. Switch to staging"

User: "Create new"

Claude: "Looking at pending tasks...

Next pending task: Dashboard layout
Priority: Medium

Suggested branch name: feature/dashboard-layout

Note: You have uncommitted changes on feature/auth-system.
Commit them first?"
```

---

**No task found:**

```
User: "/branch"

Claude: "I don't see any in-progress or pending tasks in tasks.md.

Would you like to:
1. Check your task list (/tasks)
2. Provide a branch name manually
3. Create a task first (/feature-planner)"

User: "user-settings"

Claude: "Creating branch with your name...

Branch: feature/user-settings
Based on: staging

Branch created!

Note: Consider adding this as a task in tasks.md to track progress."
```

---

**Custom branch name:**

```
User: "branch hotfix-production-error"

Claude: "Creating branch: fix/hotfix-production-error

Branch created and checked out!

Branch: fix/hotfix-production-error
Based on: staging

Ready to work on the hotfix."
```

## Error Handling

**Branch already exists:**
```
Branch 'feature/user-auth' already exists.

Would you like to:
1. Switch to existing branch
2. Create with different name (feature/user-auth-v2)
3. Delete and recreate (destructive!)
```

**Uncommitted changes blocking switch:**
```
You have uncommitted changes that would be overwritten:
- src/components/Auth.tsx
- lib/api.ts

Options:
1. Commit changes first (git commit)
2. Stash changes (git stash)
3. Cancel branch creation
```

**Not in a git repository:**
```
This directory is not a git repository.

Initialize git first:
git init
git remote add origin [your-repo-url]
```

## Integration with Other Skills

**With /tasks:**
Use task-tracker to find and start the next recommended task, then create its branch automatically.

**With /wrap-up:**
Once work is done, /wrap-up handles everything: logs session, lints, commits, pushes, creates PR, handles code review, merges, and cleans up the branch.

**With /log:**
Use /log for mid-session progress saves when you're not ready to push yet.

## Branch Naming Best Practices

**Good names:**
- `feature/stripe-checkout`
- `fix/auth-redirect`
- `update/dashboard-ui`

**Bad names:**
- `feature/fix-stuff` (vague)
- `test` (no context)
- `manus-branch` (not descriptive)
- `feature/this-is-a-very-long-branch-name-that-goes-on-forever` (too long)
