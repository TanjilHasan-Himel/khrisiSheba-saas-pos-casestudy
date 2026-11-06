# ADR 0004 — Migration authority

- **Status:** Proposed — pending approval (corrected in revision 2)
- **Date:** 2026-11-05
- **Deciders:** Lead Architect, Database Engineer

> **Correction:** revision 1 implied hosted pushes are the whole story and did not specify the
> connection mode or default-privilege revocation. See `ARCHITECTURE-REVIEW.md` §0.6–§0.7.

## Context

The TRD leaves the ORM open and suggests Drizzle. Supabase CLI also manages migrations. Running two
migration engines against one database causes drift and unreviewable history.

## Decision

- **`supabase/migrations/*.sql` is the single source of truth.**
- Migrations are applied with the Supabase CLI (`db reset` locally, `db push` to hosted) as the
  `postgres` owner role over a **session/direct connection (5432)**. DDL is not applied through the
  transaction pooler (6543).
- SQL is hand-authored and reviewed. Generated SQL (from Drizzle Kit or any tool) is reviewed and
  committed into the same folder; no tool applies migrations independently.
- The **first migration** revokes default and existing privileges from `anon`, `authenticated`, and
  `service_role`, creates the `app_runtime` role, and installs the membership helper functions.
  No tenant table is created before that.
- Drizzle ORM will be introduced only as a typed query layer when the schema grows, with a CI
  check that `supabase db diff --linked` is empty.
- CI applies all migrations from zero against a real PostgreSQL and runs the isolation, grant, and
  permission suites.
- Migrations are forward-only. Destructive rollback requires a verified backup and explicit owner
  authorization.
- Application tables are never created through the Supabase dashboard or as `supabase_admin`.

## Rationale

- One history, one tool, reviewable SQL, no ORM lock-in.
- Session/direct DDL avoids transaction-pooler assumptions about advisory locks and session state.
- The default Supabase ACL grants new `public` tables to `anon`/`authenticated`/`service_role`;
  revoking it in the first migration is the only way to keep that from becoming a live exposure.

## Consequences

- CI must gate on schema drift and on tenant-table grants.
- Applied migration files are immutable; fixes are new migrations.
- ORM schema definitions must be kept in sync manually or generated and reviewed.
- A local/staging PostgreSQL is required for `db reset`; Docker is one way to provide it, not the
  only one.
- No migration has been applied to the hosted project yet; `supabase/migrations` does not exist.
