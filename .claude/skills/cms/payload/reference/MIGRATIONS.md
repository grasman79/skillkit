# Migrations & Versioned Collections

Reusable lessons for Payload CMS (Drizzle ORM) + PostgreSQL migrations, beyond the seed-guard issue covered in [CONTENT-SYNC.md](CONTENT-SYNC.md).

## Payload does NOT auto-run migrations on dev start

After any schema change, you MUST run migrations manually. Restarting the backend alone causes `relation "table_name" does not exist` errors.

```bash
# From the backend directory:
pnpm payload migrate:create   # Generate migration from schema diff
pnpm payload migrate           # Run pending migrations
pnpm run dev                   # NOW restart
```

| Command | What it does |
|---------|-------------|
| `pnpm payload migrate:create` | Generates migration by diffing schema against database |
| `pnpm payload migrate` | Runs all pending migrations |
| `pnpm payload migrate:status` | Shows applied/pending migrations |
| `pnpm payload migrate:down` | Rolls back last migration |
| `pnpm payload migrate:fresh` | Drops all tables, re-runs everything (destructive) |
| `pnpm payload migrate:reset` | Rolls back all migrations (destructive) |

## Make schema changes incrementally

Never add many fields/tables at once. Add ONE section/group at a time, verify the dev server starts and admin saves correctly, then add the next.

Large schema changes (50+ fields) cause migration nightmares: wrong table naming (single vs double underscores for blocks), missing columns (`block_name`), and broken migrations that are hard to revert.

When `migrate:create` triggers too many interactive drizzle-kit prompts (blocking the server), write manual SQL migrations instead:

```typescript
import { MigrateUpArgs, sql } from '@payloadcms/db-postgres'

export async function up({ db }: MigrateUpArgs): Promise<void> {
  await db.execute(sql`
    ALTER TABLE "pages" ADD COLUMN IF NOT EXISTS "new_field" varchar;
  `)
}
```

Register manual migrations in `migrations/index.ts`. Use `IF NOT EXISTS` / `IF EXISTS` for idempotency.

## Never use Payload API in migrations

Never call `payload.*` methods (`updateGlobal`, `find`, `create`, `update`, `delete`) inside a migration `up()` function. Always use `db.execute(sql`...`)` with raw PostgreSQL.

Payload compiles its Drizzle ORM from the **final** schema state at startup. When a migration calls `payload.updateGlobal()`, Payload queries columns from later migrations that haven't run yet, causing "column does not exist" errors on fresh databases.

**Correct pattern:**
- Migrations = schema changes + minimal row creation via raw SQL
- `onInit` hook = rich content seeding via Payload ORM (runs after ALL migrations complete)

```typescript
// CORRECT - raw SQL in migration
await db.execute(sql`
  INSERT INTO "pages" ("slug", "_status", "updated_at", "created_at")
  SELECT 'home', 'published', NOW(), NOW()
  WHERE NOT EXISTS (SELECT 1 FROM "pages" WHERE "slug" = 'home')
`)

// WRONG - Payload API in migration
await payload.create({ collection: 'pages', data: { slug: 'home' } })
```

### Version array tables need `_uuid` column

When writing manual migrations for array fields, version array tables (`_collection_v_*`) need an extra `"_uuid" varchar` column that regular array tables do NOT have. Without it, queries fail with "column _uuid does not exist".

## Versioned collections

Payload versioned collections (drafts/published) have three critical behaviors beyond what's in [COLLECTIONS.md](COLLECTIONS.md#versioning--drafts):

### 1. Drizzle uses delete+insert, not UPDATE

For versioned collections (pages, posts), saves do a DELETE then INSERT. If the INSERT omits `created_at`, the NOT NULL constraint fires (same root cause as the `payload_migrations` issue in [CONTENT-SYNC.md](CONTENT-SYNC.md)).

Fix: add a database trigger:
```sql
CREATE OR REPLACE FUNCTION set_created_at()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.created_at IS NULL THEN
    NEW.created_at = NOW();
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_pages_created_at
BEFORE INSERT OR UPDATE ON pages
FOR EACH ROW EXECUTE FUNCTION set_created_at();
```

### 2. Admin reads from `_pages_v`, not `pages`

SQL-seeded pages are invisible in the admin list unless they also have a row in `_pages_v`. Backfill version rows in `onInit` for pages without versions.

### 3. Backfill `_pages_v` with the Payload API, not raw SQL

Never use raw SQL to insert into `_pages_v`. Payload reads page titles from `_pages_v_locales`, not `_pages_v`. A raw SQL INSERT creates a version row without the locales row, so titles show blank in admin.

Always use `payload.update()` to create version rows - this populates both `_pages_v` and `_pages_v_locales` correctly. The backfill must run AFTER all content seeding in `onInit` - if it runs before, pages get duplicate version rows.

### 4. `updateGlobal({ draft: false })` may not publish

Due to the delete+insert pattern, `_status` can stay as `'draft'`. Force-publish via raw SQL after seeding:
```sql
UPDATE header SET _status = 'published'
```

## R2 / S3 media storage

### Configure R2 before first dev start

R2 credentials must be in `.env` BEFORE starting dev servers for the first time. Without them, Payload falls back to local disk storage. Files uploaded locally won't exist in production (most hosts have an ephemeral filesystem) and need re-uploading.

### `s3Storage` plugin crashes with empty credentials

When R2 credentials are empty strings, the `s3Storage` plugin still initializes and constructs `endpoint: "https://"` which breaks the upload handler. This causes a 500 error on `/admin/login`.

Fix: conditionally initialize the plugin - only include it when credentials are set:
```typescript
// payload.config.ts
plugins: [
  ...(process.env.R2_ENDPOINT ? [s3Storage({ /* config */ })] : []),
]
```

## Instructions for Claude

When working on a Payload CMS project:
- After schema changes, tell the user to run `pnpm payload migrate:create && pnpm payload migrate` from the backend directory, THEN restart the dev server
- Never say "restart the backend and Payload will run the migration automatically" - this is false for local development
- Make schema changes ONE section at a time - never batch 50+ field changes into one migration
- Never use `payload.*` API calls inside migration files - use raw SQL via `db.execute(sql`...`)`
- For versioned collections, always add `created_at` triggers and backfill `_*_v` tables via the Payload API (not raw SQL)
