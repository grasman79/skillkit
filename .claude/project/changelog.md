# Changelog

**Last Updated:** 2025-12-31

This file tracks work completed across sessions to help maintain context.

---

## Session Log

### 2025-12-31 - Added Electric SQL Stack

**Summary:** Created a comprehensive Electric SQL stack guide. Electric SQL is a Postgres sync engine for building local-first applications with real-time data synchronization.

**Completed:**
- Created new stack file: `.claude/stacks/database/database-electric-sql.md`
- Documented core concepts: Shapes, ShapeStream, Shape classes
- Added TypeScript client API with all configuration options
- Included React integration with useShape hook
- Documented API proxy pattern for production security
- Added custom parsing, error handling, and deployment sections
- Integrated TanStack DB documentation with collections, live queries, and transactional mutations
- Referenced key blog post: "Local-first sync with Electric and TanStack DB"

**Files Changed:**
- `.claude/stacks/database/database-electric-sql.md` (new)

**Notes for Future Sessions:**
- Electric SQL works well with TanStack DB for local-first React apps
- The API proxy pattern is recommended for production deployments
- Shapes are the core primitive for selective data syncing

---
