# Project Implementation Tasks

**Project:** [Will be populated during initialization]
**Last Updated:** [Will be populated during initialization]
**Status:** Not initialized

---

## Instructions

This file will be automatically populated when you initialize the project.

**To initialize:**
1. Say "Help me set up my project"
2. Answer the setup questions about what you're building

Claude will create implementation tasks based on your projectbrief.md features and requirements.

---

## Task Template

Each task will follow this structure:

```markdown
## Task N: [Task Name]
**Status:** pending | in progress | mostly complete | complete
**Priority:** High | Medium | Low
**Tags:** #domain #type #priority #stack
**Dependencies:** [List of task numbers this depends on]
**Estimated Time:** [Hours]
**Actual Time:** [Track as work progresses]

### Checklist:
- [ ] Sub-task 1
- [ ] Sub-task 2
- [ ] Sub-task 3

### Implementation Notes:
[Added during development - decisions made, approaches taken, etc.]

### Test Strategy:
- [ ] Test requirement 1
- [ ] Test requirement 2
- [ ] Test requirement 3

---
```

**Status Definitions:**
- `pending` - Not started, no work done
- `in progress` - Some checkboxes marked, active development
- `mostly complete` - Major work done, minor items/polish remaining
- `complete` - All checkboxes marked, fully tested, merged

**Tag Categories:**

**Domain Tags:**
- `#frontend` - UI components, pages, client-side code
- `#backend` - API routes, server actions, business logic
- `#database` - Schema, migrations, queries
- `#api` - External API integrations
- `#ui` - Design system, styling, UX
- `#devops` - Deployment, CI/CD, infrastructure

**Type Tags:**
- `#feature` - New functionality
- `#bug` - Bug fix
- `#refactor` - Code improvement without behavior change
- `#docs` - Documentation updates
- `#test` - Testing-related work
- `#security` - Security improvements
- `#performance` - Performance optimization

**Priority Tags:**
- `#critical` - Must be done immediately
- `#high` - Important, do soon
- `#medium` - Standard priority
- `#low` - Nice to have

**Stack Tags:**
(Use based on your tech stack)
- `#nextjs` - Next.js specific
- `#react` - React components
- `#prisma` - Prisma ORM
- `#drizzle` - Drizzle ORM
- `#shadcn` - shadcn/ui components
- `#vercel` - Vercel deployment
- `#stripe` - Stripe payments
- `#auth` - Authentication (Better Auth, Auth.js, etc.)
- `#ai` - AI features (Vercel AI SDK, etc.)

**Example Task with Tags:**
```markdown
## Task 3: User Authentication
**Status:** in progress
**Priority:** High
**Tags:** #backend #auth #security #feature #nextjs
**Dependencies:** Task 1 (Database Setup)
**Estimated Time:** 8 hours
**Actual Time:** 6 hours (in progress)

### Checklist:
- [x] Install Better Auth
- [x] Configure auth providers (email, Google)
- [x] Create auth API routes
- [ ] Add login/signup forms
- [ ] Implement session management
- [ ] Add protected route middleware

### Implementation Notes:
- Using Better Auth for simplified setup
- Email provider configured with Resend
- Google OAuth requires approved domain

### Test Strategy:
- [x] Test email authentication flow
- [ ] Test Google OAuth flow
- [ ] Test session persistence
- [ ] Test protected routes
```

---

## Project Metrics

**Current Week:** [Week of Date Range]
- **Tasks completed:** 0
- **Total time:** 0 hours
- **Estimation accuracy:** N/A
- **Blockers resolved:** 0

**Velocity trend:** → stable

**All-Time Stats:**
- **Total tasks completed:** 0
- **Average completion time:** N/A
- **Overall estimation accuracy:** N/A

**Lessons Learned:**
- [Will be added as patterns emerge]

---

## Archived Tasks

Completed tasks will be moved here automatically when using the `archive completed` command.

---

*This file will be automatically maintained by Claude as you complete work.*
*See `.claude/07-documentation-maintenance.md` for update protocols.*
