---
name: schema
description: >
  Database schema design skill that enforces data traceability conventions
  across any database engine or ORM. Every table must include audit/traceability
  columns (created_at, updated_at, deleted_at, created_by, updated_by,
  deleted_by) with consistent naming, nullability rules, and foreign-key
  relationships to user/entity tables. Soft-delete is the default deletion
  strategy. The agent unifies dispersed naming patterns in existing schemas,
  generates proper migration scripts using the project's revision tool, and
  records decisions in project-timeline.md.
license: MIT
compatibility: >
  Portable Agent Skill. Verified layouts for Claude Code (.claude/skills/),
  Cursor (.cursor/skills/), GitHub Copilot (.github/skills/), and Factory
  Droid (.factory/skills/). Works with any database (PostgreSQL, MySQL,
  SQLite, MongoDB, etc.) and any ORM/migration tool (Prisma, Knex, Alembic,
  Flyway, TypeORM, Sequelize, Django, Rails ActiveRecord, etc.).
metadata:
  version: "1.0.0"
  workflow: schema-traceability
---

# Schema — Data Traceability Conventions

A prescriptive skill that ensures every database table or collection carries
full audit trail columns with consistent naming, proper constraints, and
foreign-key relationships. Agents use this skill whenever creating or
modifying database schemas.

## When to use this skill

- Creating a new table, collection, or entity.
- Modifying an existing schema (adding columns, renaming, refactoring).
- Reviewing or auditing schema definitions for traceability gaps.
- Unifying inconsistent naming across an existing codebase.

## Required traceability columns

Every table/entity MUST include these columns:

| Column        | Type               | Nullable | Default          | Notes                                    |
|---------------|--------------------|----------|------------------|------------------------------------------|
| `created_at`  | timestamp with tz  | NOT NULL | current timestamp| Set once at row creation, never updated  |
| `updated_at`  | timestamp with tz  | NULL     | NULL             | Set on every update; NULL means never updated |
| `deleted_at`  | timestamp with tz  | NULL     | NULL             | Non-null means soft-deleted              |
| `created_by`  | FK to user/entity  | NOT NULL | —                | Who created the record                   |
| `updated_by`  | FK to user/entity  | NULL     | NULL             | Who last updated the record              |
| `deleted_by`  | FK to user/entity  | NULL     | NULL             | Who soft-deleted the record              |

### Rules

1. **`created_at` is never null.** It must have a server-side default
   (e.g., `NOW()`, `CURRENT_TIMESTAMP`). Application code must not override
   it after creation.
2. **Soft-delete is the default strategy.** Use `deleted_at IS NOT NULL` to
   filter out deleted rows. Hard-delete is only acceptable when legally
   required (e.g., GDPR right-to-erasure) or for ephemeral/cache tables —
   document the exception.
3. **`*_by` columns are foreign keys.** They reference the user table (or
   the relevant actor entity such as service account, API key, etc.). Never
   store free-text usernames — always use a referential key.
4. **Correlation ID** — if the project uses distributed tracing or event
   sourcing, add a `correlation_id` (UUID, NOT NULL) column to link the row
   mutation to the originating request/event.

## Naming conventions

1. **Learn from existing code first.** Scan the project's models, migrations,
   and schema files to detect the prevailing naming case:
   - `snake_case` (most SQL databases, Python/Ruby ORMs)
   - `camelCase` (some JS/TS ORMs like Prisma, TypeORM)
   - `PascalCase` (C#/Entity Framework)
2. **Adopt the existing convention.** Do not introduce a second convention.
3. **If naming is dispersed/inconsistent**, propose a unification migration
   that renames columns to a single chosen pattern. Present the options to
   the human (via the gated-agentic-development workflow if available)
   before executing.

### Standard column name mappings by case

| Concept      | snake_case     | camelCase    | PascalCase   |
|--------------|----------------|--------------|--------------|
| Created at   | `created_at`   | `createdAt`  | `CreatedAt`  |
| Updated at   | `updated_at`   | `updatedAt`  | `UpdatedAt`  |
| Deleted at   | `deleted_at`   | `deletedAt`  | `DeletedAt`  |
| Created by   | `created_by`   | `createdBy`  | `CreatedBy`  |
| Updated by   | `updated_by`   | `updatedBy`  | `UpdatedBy`  |
| Deleted by   | `deleted_by`   | `deletedBy`  | `DeletedBy`  |
| Correlation  | `correlation_id`| `correlationId` | `CorrelationId` |

## Workflow

### Step 1 — Discover existing patterns

1. Scan the project for schema definitions (models, migration files, DDL
   scripts, ORM entity files).
2. Identify the naming case, existing traceability columns (if any), and
   the migration/revision tool in use.
3. Note any inconsistencies (e.g., some tables use `createdAt`, others use
   `created_at`; some tables lack `deleted_at`).

### Step 2 — Align or create schema

- **New table**: Include all required traceability columns from the start.
- **Existing table missing columns**: Generate an additive migration that
  adds the missing columns with appropriate defaults for existing rows.
- **Inconsistent naming**: Propose a single rename migration. If renaming
  would break downstream consumers, add new columns + backfill + deprecate
  old ones in a phased approach.

### Step 3 — Generate migration

1. Use the project's existing revision/migration tool:
   - Alembic → `alembic revision --autogenerate -m "..."`
   - Knex → `npx knex migrate:make ...`
   - Prisma → `npx prisma migrate dev --name ...`
   - Flyway → create `V<next>__description.sql`
   - TypeORM → `npx typeorm migration:generate ...`
   - Django → `python manage.py makemigrations`
   - Rails → `rails generate migration ...`
   - Sequelize → `npx sequelize-cli migration:generate --name ...`
2. Review the generated migration for correctness.
3. Run it locally (`migrate up` / `migrate dev`) to verify it applies
   without errors.

### Step 4 — Document

- If `project-timeline.md` exists, append an entry describing the schema
  change, naming convention chosen, and any unification performed.
- If it does not exist, reference the code and migration as the source of
  truth.

## Notes for agents operating this skill

- Never skip `created_at` — it is the single non-negotiable column.
- When in doubt about whether to soft-delete or hard-delete, default to
  soft-delete and document the choice.
- If no migration tool exists in the repo, suggest adding one appropriate
  to the tech stack before writing raw DDL.
- If the existing schema is wildly inconsistent, do NOT silently rename
  columns. Present a unification plan to the human first.
- Always check for existing indexes on the traceability columns; add
  indexes on `deleted_at` (for filtering soft-deletes) and `created_at`
  (for time-range queries) if they are missing.
