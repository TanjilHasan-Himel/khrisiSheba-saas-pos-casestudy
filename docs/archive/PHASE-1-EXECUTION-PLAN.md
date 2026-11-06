# PHASE 1 EXECUTION PLAN — Foundation

**Status:** Phase 0 correction pass (revision 2) — **awaiting approval.** The F1.0/F1.1 foundation code exists at commit `4b5c2a5` but is **frozen**; no new application code is authorized.
**Date:** 2026-11-05
**Revision:** 2 — corrects RLS/auth/runtime-role steps, adds the M6a/M6b ordering, and records the verified environment.
**Scope:** `07-IMPLEMENTATION-PLAN.md` Phase 1 — monorepo, environments, CI, PostgreSQL, migrations, identity, tenant creation, membership, stores, RBAC, RLS, audit, request IDs.
**Out of scope:** catalog, purchasing, inventory, sales, customers, reports, platform UI, POS, website, billing.

**Phase 1 exit gate (unchanged in intent):** two-tenant isolation, RLS trust-boundary, permission, and store-scope tests pass, in CI, against real PostgreSQL.

---

## 1. Entry criteria (must be true before new implementation)

| #   | Criterion                                                                            | Current state                                   |
| --- | ------------------------------------------------------------------------------------ | ----------------------------------------------- |
| 1   | Product owner approved `B-01`–`B-08`                                                 | **Pending**                                     |
| 2   | Auth/session/refresh/CSRF model approved (`D-04`, `G-03`, `G-60`)                    | **Pending**                                     |
| 3   | Role/store-scope model approved (`D-05`, `G-06`, `G-07`, `C-20`)                     | **Pending**                                     |
| 4   | RLS trust boundary + default-grant revocation approved (`D-01`–`D-03`)               | **Pending**                                     |
| 5   | Supabase region decided; paid tier/backups confirmed                                 | **Pending** (currently Sydney, tier unverified) |
| 6   | A real PostgreSQL is available for local tests (Docker, local, or dedicated staging) | **Missing** (no Docker, no local Postgres)      |
| 7   | `app_runtime` + custom-role pooling feasibility verified                             | **Unverified**                                  |
| 8   | pnpm available                                                                       | Yes — `10.34.9` at `C:\Users\Himel\.local\bin`  |
| 9   | Repo initialized with `.gitignore` for `.env`, `supabase/.temp`                      | Yes — commit `4b5c2a5`, working tree clean      |
| 10  | Git remote + CI can run against a PostgreSQL service                                 | **Missing** (no remote; Actions never run)      |
| 11  | Custom SMTP configured for staging/production (not blocking local)                   | **Pending**                                     |
| 12  | DB password shared during setup rotated                                              | **Pending**                                     |

If any of 1–7 or 12 is unresolved, only documentation and read-only environment verification may proceed. No schema, API, auth, or RLS code.

### 1.1 Verified environment (correction pass)

| Item                           | Verified value / state                                                                           |
| ------------------------------ | ------------------------------------------------------------------------------------------------ |
| Node / pnpm / git              | `v24.13.0` / `10.34.9` / `2.53.0`                                                                |
| Docker / `psql`                | Not installed                                                                                    |
| Git                            | Repo exists; `4b5c2a5`; no remote; CI never ran                                                  |
| Supabase project               | `vqqjkigsumihfpmredpi`; region `ap-southeast-2` (Sydney); empty                                  |
| Database version               | PostgreSQL **17.11** through the pooler                                                          |
| Pooler                         | `aws-0-ap-southeast-2.pooler.supabase.com:6543` works from this machine                          |
| Direct DB host                 | AAAA-only; not usable from IPv4                                                                  |
| `postgres` role                | `LOGIN`, `BYPASSRLS`, `CREATEROLE`, `CREATEDB`, not superuser                                    |
| `anon` / `authenticated`       | `NOLOGIN`, `NOBYPASSRLS`, `USAGE` on `public`, no `CREATE`                                       |
| `service_role`                 | `NOLOGIN`, `BYPASSRLS`                                                                           |
| `app_runtime`                  | Does not exist                                                                                   |
| `public` tables / `auth.users` | 0 / 0 rows                                                                                       |
| Default ACL                    | Future `public` tables get **ALL** for `anon`, `authenticated`, `service_role` — must be revoked |
| TLS                            | Supabase chain not in Node's default store; pin the CA                                           |
| `supabase/migrations`          | Does not exist                                                                                   |
| Tests last run                 | 14 passed; build + runtime smoke passed (shell only)                                             |

### 1.2 Smallest safe Phase 1 starting milestone

After the decisions are approved, the first code-bearing milestone is **not** the full core schema. It is a narrow proof of the riskiest assumptions:

**F1.2a — RLS/runtime-role proof (throwaway scope)**

1. Provision a real PostgreSQL (Docker, local, or a dedicated staging database — not the hosted project unless the owner explicitly allows it).
2. Create a minimal migration with only: `tenants`, `users`, `tenant_memberships`, `roles` (or role constants), `stores`, and one tenant-owned sample table.
3. Create `app_runtime` (`NOBYPASSRLS`, non-owner) and the `is_tenant_member` / `list_my_memberships` functions.
4. Revoke default privileges and all `anon`/`authenticated`/`service_role` table grants.
5. Add `ENABLE` + `FORCE RLS` and policies on every created table.
6. Run the isolation suite: no context → 0 rows; member of A querying B → 0 rows; cross-tenant FK insert fails; parallel transactions do not leak context.
7. Verify the same role/grants through the chosen pooler mode (or record the fallback if Supabase rejects a custom role).
8. Delete the throwaway database/role after the proof (or keep it as the staging base).

**Exit from F1.2a:** actual SQL output showing the isolation cases, plus a recorded decision on pooler/TLS. Only then expand to **F1.2b — the full core schema** (invitations, audit, platform operators, etc.).

This ordering prevents the team from building a large schema on top of an unverified isolation model.

---

## 2. Phase 1 work packages

Each package is independently testable and ends with a report using the Agent Execution Protocol (`ARCHITECTURE-REVIEW.md` §17 format from the addendum).

### F1.0 — Decision lock and environment verification (~0.5 unit)

**Deliverables**

- Decision Log entries for approved items; no proposal is silently promoted.
- `docs/adr/0001`–`0004` statuses corrected to **Proposed — pending approval**.
- Supabase region decided; project recreated only if the owner explicitly approves the destructive change.
- A real PostgreSQL available for tests: Docker Desktop, a local PostgreSQL, or a dedicated staging database.
- Git repository initialized; `.gitignore` covers `.env*`, `supabase/.temp/`, build output; remote added.
- DB password rotated; secrets stored outside the repo.
- `app_runtime`/pooler/TLS feasibility proven or explicitly deferred to F1.2a.

**Acceptance**

- `docker version` **or** `psql --version` **or** a documented staging connection succeeds. Docker is not the only acceptable answer; a real database is.
- `npx supabase start` works **if** Docker is chosen; otherwise the chosen database has the required extensions.
- `npx supabase db reset` completes **if** using the Supabase CLI locally.
- `git status` clean after the initial commit; a remote exists.
- No `.env` file committed.
- The DB password shared in chat is no longer valid.

**Validation**

```powershell
docker version            # optional path
psql --version            # optional path
corepack enable --install-directory "$env:USERPROFILE\.local\bin"
corepack prepare pnpm@10 --activate
pnpm --version
npx supabase@latest start # only if Docker path is chosen
npx supabase@latest db reset
git remote -v
```

**Correction:** revision 1 required Docker specifically. The actual requirement is a real PostgreSQL for RLS/concurrency tests; Docker is one implementation.

---

### F1.1 — Workspace and CI (~1 unit) — code exists, frozen

**Current state (verified):** F1.0/F1.1 is already implemented at commit `4b5c2a5`. This section is a **re-acceptance checklist**, not a greenfield build. Do not rewrite the workspace.

**Existing files (verified, representative):**

```text
package.json / pnpm-workspace.yaml / tsconfig.base.json / eslint.config.mjs / .prettierrc.json
.nvmrc / .gitignore / .gitattributes / .env.example
apps/api/src/{index,server,env}.ts
apps/api/src/plugins/{db,errors,request-id}.ts
apps/api/src/modules/health/routes.ts
apps/saas/src/app/{layout,page}.tsx + globals.css
packages/contracts/src/common/{error-codes,errors,http}.ts
packages/config/src/{env,index}.ts
packages/database/src/{client,index}.ts
.github/workflows/ci.yml
docs/adr/0001–0004, docs/DECISION-LOG.md
```

**Known defects to fix only after unfreeze**

- `package.json` `clean` uses `rm -rf`, which fails in PowerShell.
- API CORS enables `credentials: true` although the browser must not call the API.
- `.env.example` does not list the Supabase Auth/session/admin variables.
- `pnpm format:check` currently fails on the generated `apps/saas/next-env.d.ts`, which is not in
  `.prettierignore`. The repository's own docs and source pass; the generated file does not.
- `apps/saas` has no auth/session code yet (expected — F1.4).
- `packages/ui` and Drizzle are not installed yet (deliberate).

**Acceptance (re-verified locally; CI must be repeated on a remote)**

- `pnpm install --frozen-lockfile` succeeds on a clean checkout.
- `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build` pass.
- API starts and returns `{status:"ok"}` from `/health/live`.
- `/health/ready` returns 503 when the DB is not configured/unreachable.
- Error envelope shape matches `{code,message,details,requestId}` on a thrown route.
- Every request carries `x-request-id`.
- The CI workflow runs install → format → lint → typecheck → test → build → audit **and has actually executed on a remote** with a PostgreSQL service before it is called green.
- `.env.example` contains every variable used by `apps/api` and `apps/saas`, with no real secrets.

**Tests**

- Unit: env schema rejects missing/invalid variables; error envelope serializer; pagination bounds.
- Integration: health endpoints (live, ready not configured).
- CI: a deliberately failing test must fail the pipeline (verified once, then reverted).

**Commands**

```powershell
pnpm install --frozen-lockfile
pnpm lint
pnpm typecheck
pnpm test
pnpm build
```

---

### F1.2 — Database foundation (~2 units, split into F1.2a/F1.2b)

**Scope:** first real migration for identity, tenancy, stores, roles, audit. No catalog/inventory/sales tables.

#### F1.2a — RLS/runtime-role proof (do this first)

- Minimal migration: `tenants`, `users`, `tenant_memberships`, `roles` (or role constants), `stores`, and one sample tenant-owned table.
- Create `app_runtime`, the membership helper functions, default-ACL revocations, `FORCE RLS`, and policies.
- Run the isolation suite on a throwaway database (see §1.2) before expanding the schema.
- Exit: actual SQL output for the isolation cases, plus a recorded pooler/TLS decision.

#### F1.2b — Full core schema

**Tables (minimum for Phase 1; corrected)**

```text
users(id, auth_user_id, email, phone, display_name, status, created_at, updated_at)
tenants(id, legal_name, display_name, status, timezone, currency, created_at, updated_at)
tenant_memberships(id, tenant_id, user_id, role_id, status, invited_by, created_at)
roles(id, code, name, is_system)
permissions(id, code)
role_permissions(role_id, permission_id)
stores(id, tenant_id, code, name, address, phone, status, created_at, updated_at)
store_memberships(id, tenant_id, store_id, user_id, status)   -- NO role_id
invitations(id, tenant_id, email, role_id, token_hash, status, expires_at, invited_by, created_at)
audit_events(id, tenant_id, actor_id, entity_type, entity_id, action, timestamp, before_json, after_json)
platform_operators(id, user_id, status, created_at)            -- prefer a separate `platform` schema
```

**Constraints and grants**

- `users`: unique `auth_user_id`; unique email (case-insensitive); unique phone where not null. Verify the `auth.users` cross-schema FK on PostgreSQL 17.11; if unsupported, use a trigger-maintained `auth_user_id` unique column.
- Tenants/stores: composite unique `(tenant_id, id)` on tenant-scoped parents.
- Memberships: partial unique active `(tenant_id, user_id) WHERE status='ACTIVE'`.
- Store memberships: partial unique active `(tenant_id, store_id, user_id) WHERE status='ACTIVE'`; composite FK `(tenant_id, store_id)`; **no role column**.
- `permissions.code` unique; `roles.code` unique.
- `audit_events` append-only; `app_runtime` gets `SELECT, INSERT` only.
- All tenant tables `tenant_id NOT NULL`.
- Revoke default privileges and all `anon`/`authenticated`/`service_role` table, sequence, and function grants **in the first migration**.
- `app_runtime` is non-owner and `NOBYPASSRLS`; table owner is `postgres`.
- Do not apply DDL through the transaction pooler. Use session mode (5432) or a direct connection.

**Acceptance**

- `supabase db reset` applies all migrations from scratch.
- `supabase db diff --linked --schema public` returns no differences after push.
- Composite FKs reject cross-tenant store/user references.
- Inserting a store membership with a store from another tenant fails at the DB layer.
- `app_runtime` cannot read `tenant_memberships` directly; it uses `list_my_memberships`.
- Querying `pg_default_acl` and `has_table_privilege` proves `anon`/`authenticated`/`service_role` have no access to tenant tables.
- Migration files are ordered, named, and reviewed; no manual production edits.
- The `app_runtime` role connects through the chosen path with a pinned CA, or the fallback is recorded.

**Tests**

- Migration up/down where reversal is safe (or documented forward-fix).
- Constraint tests for every unique/partial unique/composite FK.
- Grant tests for default ACL and per-table privileges.
- Fixture: two tenants, two stores, three users.
- RLS proof is F1.3; F1.2b does not claim it.

**Commands**

```powershell
npx supabase@latest db reset
npx supabase@latest migration list --local
$env:SUPABASE_DB_PASSWORD='<from secret store>'
npx supabase@latest db diff --linked --schema public
# Run through a session/direct connection, not the transaction pooler:
#   psql "$env:DATABASE_MIGRATION_URL" -c "select ... from pg_default_acl;"
```

---

### F1.3 — Runtime role, RLS, and tenant context (~2 units)

**Scope**

- `app_runtime` database role (`LOGIN`, `NOBYPASSRLS`, non-owner), least-privilege grants.
- `FORCE ROW LEVEL SECURITY` on every tenant table.
- Policies that check **both** tenant equality and active membership:
  ```sql
  tenant_id = nullif(current_setting('app.tenant_id', true), '')::uuid
  and public.is_tenant_member(
        nullif(current_setting('app.user_id', true), '')::uuid,
        tenant_id
      )
  ```
- `SECURITY DEFINER` helpers `is_tenant_member` and `list_my_memberships`, owned by the migration role, fixed `search_path`, EXECUTE granted only to `app_runtime`.
- `withTenantTransaction(tenantId, userId, storeId, fn)` helper:
  ```sql
  BEGIN;
  SELECT set_config('app.user_id',   $1, true);
  SELECT set_config('app.tenant_id', $2, true);
  SELECT set_config('app.store_id',  $3, true); -- optional
  -- fn runs
  COMMIT;
  ```
- Membership verification inside the transaction before the business statement.
- Default-privilege and per-table grant revocation for `anon`/`authenticated`/`service_role`.
- No `BYPASSRLS` for the runtime role; migrations run as `postgres`.
- **Trust boundary:** RLS enforces the context the API set; it does not authenticate the caller. Do not write acceptance criteria that claim otherwise.

**Acceptance**

- With no context set, every tenant table returns 0 rows.
- With `app.tenant_id = A` and `app.user_id = U` where U is an active member of A, only A rows are visible.
- With `app.tenant_id = B` and `app.user_id = U` where U is **not** a member of B, 0 rows are visible even if B's UUID is known.
- A tenant-equality-only policy is rejected in review; the membership predicate is required.
- `app_runtime` cannot `UPDATE`/`DELETE` audit rows.
- `app_runtime` cannot alter schema and cannot `SELECT` `tenant_memberships` directly.
- `anon`/`authenticated`/`service_role` have no table privileges on tenant tables (default ACL included).
- Policies do not cause full table scans on the hot paths (EXPLAIN checked on fixtures).
- `app_runtime` connects through the chosen pooler with a pinned CA, or the fallback is recorded.

**Tests**

- SQL isolation suite: SELECT/INSERT/UPDATE/DELETE across tenants for each table.
- Missing-context test (0 rows) and non-member test (0 rows).
- API-level isolation: request with tenant A token and tenant B id → 404.
- Role tests: `app_runtime` vs owner permissions; no `BYPASSRLS`; no DDL.
- Transaction-local context does not leak across pooled connections (parallel test).
- Grant tests: `has_table_privilege` for `anon`/`authenticated`/`service_role` is false.

**Commands**

```powershell
pnpm test:rls
pnpm test:integration
# SQL evidence, run as app_runtime through the chosen pooler:
#   select count(*) from public.stores; -- no context -> 0
#   select set_config('app.user_id', '<U>', true);
#   select set_config('app.tenant_id', '<B>', true);
#   select count(*) from public.stores; -- U not a member of B -> 0
```

---

### F1.4 — Auth, session, principal (~2 units)

**Scope**

- Supabase Auth in Next.js via a **server-only** `@supabase/ssr` client (PKCE), httpOnly/Secure/SameSite=Lax cookie, chunked-cookie handling.
- Middleware refresh using `supabase.auth.getUser()` (validates with the Auth server) — not `getSession()` alone.
- Logout (`scope: 'global'`), expired-session handling, one-refresh/one-retry on `TOKEN_EXPIRED`.
- API JWT verification via JWKS (`iss`, `aud = authenticated`, `exp`, `nbf`, `sub`; allow-listed algorithms).
- API accepts **Bearer only**; rejects cookies; no token in query strings.
- Principal loader: user + active memberships + store scope + permissions. Tenant hint validated against memberships; cross-tenant → 404.
- `authorize(principal, permission, storeScope)` helper.
- Deactivated user/membership rejection on every request; refresh-token revocation via a narrow server-side admin module.
- CSRF at the Next.js layer: Origin/Host check on every non-GET route handler and Server Action; SameSite=Lax; no state-changing GET.
- No browser-side Supabase client that reads the session.

**Acceptance**

- Unauthenticated request to a protected route → 401 with envelope.
- Expired/invalid/forged JWT → 401, no stack trace, stable code (`TOKEN_EXPIRED` vs `TOKEN_INVALID`).
- Deactivated membership → 403 immediately (next request).
- Permissionless request → 403 with stable code.
- Store-scope violation → 404 (cross-tenant) or 403 `STORE_SCOPE_REQUIRED` (no membership).
- No token is stored in `localStorage`, `sessionStorage`, or exposed to client JS.
- Cookie flags verified: `HttpOnly`, `Secure` (production), `SameSite=Lax`, `Path=/`.
- A `TOKEN_EXPIRED` response triggers exactly one refresh and one retry; a second failure logs out.
- A cross-origin POST without a valid Origin is rejected.
- `service_role` is absent from the client bundle and tenant request path.

**Tests**

- Integration: sign-in, refresh, logout, expired token, forged token, revoked refresh token.
- Cookie/session: flags, chunking, maxAge.
- Permission matrix: each Phase 1 endpoint × each role.
- Store-scope matrix: S-01…S-12 (`ARCHITECTURE-REVIEW.md` §0.5).
- Revocation: deactivate → next request denied.
- CSRF/origin check on mutations; missing/foreign Origin rejected.
- Token leakage scan: no access token in HTML, client JS, logs, or storage.

**Commands**

```powershell
pnpm test:auth
pnpm test:permissions
pnpm test:store-scope
```

---

### F1.5 — Tenant onboarding and stores (~2 units)

**Scope**

- `POST /v1/tenants` (create ONBOARDING), `PATCH /v1/tenants/current`, `POST /v1/tenants/current/activate` (or explicit completion step).
- First store creation.
- `POST /v1/stores`, `GET /v1/stores`.
- Tenant settings: timezone, currency (BDT only), locale.
- Audit of tenant/store creation.
- Onboarding UI stepper (mobile-first).
- Tenant/store switcher with cache isolation.

**Acceptance**

- New user completes onboarding and lands on an empty dashboard.
- Interrupted onboarding resumes at the correct step.
- Creating a second store keeps the first intact.
- Tenant switch triggers a full data refetch; no previous tenant data remains in the client cache.
- Suspended tenant requests are rejected with a clear state.
- All writes are tenant-scoped and audited.

**Tests**

- Integration: create tenant, resume, activate, create store.
- Isolation: tenant A cannot see tenant B stores.
- E2E: onboarding at 360px, including validation errors.
- Cache test: switch tenant, assert no stale rows.

**Commands**

```powershell
pnpm test:integration
pnpm test:e2e -- --grep onboarding
```

---

### F1.6 — Invitations, memberships, RBAC, store scope (~2 units)

**Scope**

- Invite by email with hashed single-use token, 7-day expiry (proposed default, pending `D-08`/gap approval), resend invalidation.
- Accept invitation → **tenant** membership (`role_id`); optional store memberships (no role column).
- Role assignment/revocation; permission constants seeded.
- Staff list UI; invite/revoke actions; store-assignment UI.
- Audit of invites, role changes, store-scope changes, revocations.

**Acceptance**

- Invitation cannot be reused; expired invite fails with a clear message.
- Accepting with an existing account links to the tenant instead of duplicating the user.
- Role and store scope come from the verified principal, never from a client-supplied body field.
- A store-scoped SALES user cannot see another store's data; a store id from another tenant returns 404.
- Role/store-scope change takes effect on the next request.
- Owner cannot accidentally remove the last owner (guard).
- Revoking a membership disables access immediately and revokes refresh tokens via the narrow admin module.

**Tests**

- Integration: invite → accept → access; expiry; resend; reuse.
- Permission matrix for all staff endpoints.
- Store-scope matrix S-01…S-12 from `ARCHITECTURE-REVIEW.md` §0.5, including no-membership and revoked-membership cases.
- Isolation: store scope boundary; cross-tenant store id → 404.
- Aggregate/report restriction: a store-scoped actor never sees tenant-wide totals.
- Audit assertions.

**Commands**

```powershell
pnpm test:integration
pnpm test:permissions
pnpm test:store-scope
```

---

### F1.7 — Phase 1 hardening and exit review (~1 unit)

**Scope:** ADR updates, security review, dependency audit, runbook stub, Phase 1 report.

**Acceptance**

- All Phase 1 tests pass locally and in CI.
- `pnpm audit` reviewed; no unmitigated critical/high findings.
- Tenant isolation suite is part of CI (not a manual script).
- Rollback plan for migrations documented.
- Phase 1 report lists executed tests with actual output.

---

## 3. Dependency graph

```text
F1.0 ──► F1.1 (frozen, re-accept) ──► F1.2a (RLS/runtime-role proof)
                                          │
                                          ▼
                                      F1.2b (full core schema) ──► F1.3 ──► F1.4 ──► F1.5 ──► F1.6 ──► F1.7
                                                                       │
                                                                       └── migration gate
                                                                           (no F1.4+ data work on unstable schema)
```

- F1.2a is the **smallest safe starting milestone**. It proves the riskiest assumptions (RLS + membership predicate, `app_runtime`, pooler/TLS, default ACL) before a large schema exists.
- F1.2b and F1.3 are the critical path. Do not parallelize them with UI work.
- F1.4 depends on stable tenancy tables and a proven runtime role.
- F1.5/F1.6 can overlap only after F1.4 authorization is proven.

---

## 4. Environment matrix

| Variable                          | Local                   | Staging               | Production         | Notes                                                                    |
| --------------------------------- | ----------------------- | --------------------- | ------------------ | ------------------------------------------------------------------------ |
| `NODE_ENV`                        | development             | production            | production         |                                                                          |
| `DATABASE_URL`                    | local/staging pooler    | staging pooler (6543) | prod pooler (6543) | **`app_runtime`** credentials; `NOBYPASSRLS`; never `postgres`           |
| `DATABASE_MIGRATION_URL`          | local/session 5432      | staging session 5432  | prod session 5432  | `postgres` only, used by the CLI; not loaded by the API process          |
| `DATABASE_SSL_ROOT_CERT`          | Supabase CA path        | pinned CA path        | pinned CA path     | Required; `sslmode=verify-full`. Never `rejectUnauthorized: false`       |
| `SUPABASE_URL`                    | local API               | staging               | prod               |                                                                          |
| `SUPABASE_ANON_KEY` / publishable | local                   | staging               | prod               | Public by design; no tenant-table privileges                             |
| `SUPABASE_JWKS_URL`               | local Auth JWKS         | staging Auth JWKS     | prod Auth JWKS     | Cached; used by the API                                                  |
| `SUPABASE_SERVICE_ROLE_KEY`       | not loaded by default   | admin module only     | admin module only  | Server-side admin (GoTrue revocation) **only**; never client/tenant path |
| `API_BASE_URL`                    | `http://localhost:3001` | staging API           | prod API           | Server-to-server only; browser never calls the API                       |
| `APP_ORIGIN`                      | `http://localhost:3000` | staging app origin    | prod app origin    | Used for Origin/Host CSRF checks                                         |
| `SESSION_COOKIE_SECRET`           | generated               | secret manager        | secret manager     | If a cookie-signing layer is added; never committed                      |
| `LOG_LEVEL`                       | debug                   | info                  | info/warn          | No PII in logs                                                           |

Rules:

- `.env` files are local only; `.env.example` lists names and safe placeholders. It must be updated when unfrozen (gap `G-95`).
- Startup validates env with Zod and fails fast. `DATABASE_MIGRATION_URL` must **not** be present in the API runtime environment.
- No production secret in CI; staging credentials only.
- The API must never connect with the Supabase `postgres` user (verified `BYPASSRLS`).
- The browser must never receive `DATABASE_URL`, `service_role`, or a raw access token.
- The DB password shared in chat during setup is treated as exposed and must be rotated (`D-09`).

---

## 5. Migration and rollback strategy

1. Every schema change is a numbered SQL file in `supabase/migrations`. The directory does not exist yet; F1.2a creates the first one.
2. Migrations run as `postgres` (owner) over a **session/direct connection (5432)**, not the transaction pooler (6543). DDL plus advisory locks assume session affinity.
3. The **first migration** must, before creating tenant tables:
   - revoke default table/sequence/function privileges from `anon`, `authenticated`, and `service_role`;
   - revoke existing privileges in `public`;
   - create `app_runtime` (`LOGIN`, `NOBYPASSRLS`, non-owner) and grant only what each table needs;
   - create the `SECURITY DEFINER` helpers (`is_tenant_member`, `list_my_memberships`) owned by `postgres`, with a fixed `search_path`.
4. Local: `supabase db reset` must succeed from zero. If Docker is not used, the chosen local/staging PostgreSQL must be reset by the same scripts.
5. CI: a PostgreSQL service applies all migrations from zero, then runs the isolation/grant/permission suites. `supabase db diff --linked --schema public` must be empty for the linked staging project.
6. Staging: apply with `supabase db push` (session/direct), then run integration and isolation suites.
7. Production: apply in a reviewed window; no destructive statement without an explicit approved plan and backup confirmation.
8. Rollback: prefer forward-fix migrations. Destructive rollback requires:
   - a verified backup,
   - explicit owner approval,
   - a documented data-loss assessment.
9. Never edit an applied migration file.
10. No `DROP`/`TRUNCATE` in production without written authorization.
11. A CI check fails if any tenant table has a grant to `anon` or `authenticated`, or if `pg_default_acl` grants them future tables.
12. Do not create application tables through the Supabase dashboard or as `supabase_admin` (its default ACL also exposes tables).

---

## 6. Validation commands (exact)

Run from the repository root. These are the commands that will be used to claim completion; none may be reported as passing unless executed.

```powershell
# Toolchain
node --version                  # expect v24.x
corepack enable
corepack prepare pnpm@10 --activate
pnpm --version

# Install
pnpm install --frozen-lockfile

# Static checks
pnpm lint
pnpm typecheck
pnpm format:check

# Unit + integration
pnpm test
pnpm test:integration
pnpm test:rls
pnpm test:auth
pnpm test:permissions
pnpm test:store-scope

# End-to-end (Phase 1 flows only)
pnpm test:e2e -- --grep "onboarding|invite|switch"

# Build
pnpm build

# Database
npx supabase@latest start          # only if Docker path chosen
npx supabase@latest db reset
npx supabase@latest migration list --local
$env:SUPABASE_DB_PASSWORD='<from secret store>'
npx supabase@latest db diff --linked --schema public

# Grant + role proof (session/direct connection, as postgres)
#   select rolname, rolbypassrls, rolsuper from pg_roles where rolname='app_runtime';
#   select * from pg_default_acl;
#   select grantee, privilege_type from information_schema.role_table_grants
#    where table_schema='public' and grantee in ('anon','authenticated','service_role');

# TLS proof: connect with the pinned CA and verify-full; must succeed without
# disabling verification. A connection that only works with
# rejectUnauthorized:false is a failure.

# CI evidence (required; the workflow has never run)
gh run list --limit 5

# Security
pnpm audit --prod
```

Expected results:

- All static checks exit 0.
- Test commands exit 0 with a count of executed tests reported.
- `db diff` prints no differences.
- The grant queries return **zero rows** for `anon`/`authenticated`/`service_role` on tenant tables.
- `app_runtime` shows `rolbypassrls = false`, `rolsuper = false`.
- `gh run list` shows at least one green run on a remote with a PostgreSQL service.
- `pnpm audit` has no unmitigated critical/high findings; anything found is documented with a decision.

---

## 7. Security and data-integrity checklist for Phase 1

**Tenant isolation and RLS**

- [ ] RLS enabled **and forced** on every tenant table.
- [ ] Policies include the active-membership predicate, not tenant equality alone.
- [ ] With no context set, every tenant table returns 0 rows.
- [ ] A user who is a member of A but not B gets 0 rows when querying B.
- [ ] Runtime role (`app_runtime`) is not the table owner and has no `BYPASSRLS` and no `SUPERUSER`.
- [ ] `set_config(..., true)` used; no string interpolation of tenant/store ids into SQL.
- [ ] Membership verified in the same transaction as the write.
- [ ] Composite FKs proven with a cross-tenant insert test.
- [ ] `anon`, `authenticated`, and `service_role` have no table/sequence/function privileges on tenant tables, including the **default ACL**.
- [ ] `tenant_memberships` is not directly readable by `app_runtime`; `list_my_memberships` is used instead.
- [ ] Residual risk of a leaked `app_runtime` credential is documented, not claimed as mitigated by RLS.

**Store scope**

- [ ] Store scope enforced in the service layer and mirrored in RLS for store-scoped tables.
- [ ] `store_memberships` has no role column.
- [ ] OWNER/MANAGER/SALES/STOCK precedence matches `ARCHITECTURE-REVIEW.md` §0.5.
- [ ] Cross-tenant store ids return 404; a store-scoped user never sees tenant-wide aggregates.
- [ ] S-01…S-12 tests exist and pass.

**Auth and session**

- [ ] JWT verified with signature (JWKS), `iss`, `aud`, `exp`, `nbf`, and `sub`; algorithm allow-list enforced.
- [ ] API accepts Bearer only; rejects cookies and query-string tokens.
- [ ] Cookie flags: `HttpOnly`, `Secure` (production), `SameSite=Lax`, `Path=/`; chunking handled.
- [ ] `getUser()` used for refresh; a `TOKEN_EXPIRED` response triggers one refresh + one retry, then logout.
- [ ] Origin/Host check on every mutating route handler and Server Action.
- [ ] No access token in `localStorage`, `sessionStorage`, HTML, client JS, logs, or URLs.
- [ ] `service_role` exists only in a server-side admin module; absent from the client bundle and tenant path.

**General**

- [ ] Permission checks in services, not only routes.
- [ ] Audit rows written for invites, role changes, store-scope changes, suspensions.
- [ ] No secrets in repo, client bundle, or logs.
- [ ] Error responses contain no stack traces or SQL.
- [ ] Rate limiting on auth-adjacent endpoints.
- [ ] Structured logs redact email/phone where required.
- [ ] Dependency audit reviewed.
- [ ] Supabase CA pinned; TLS verification not disabled.
- [ ] DB password shared during setup rotated.

---

## 8. Test plan detail (Phase 1)

| ID    | Test                                            | Type        | Pass criteria                                                             |
| ----- | ----------------------------------------------- | ----------- | ------------------------------------------------------------------------- |
| PT-01 | Two-tenant SELECT isolation                     | SQL         | 0 rows from the other tenant                                              |
| PT-02 | Forged tenant/store header                      | API         | 404/403, no data                                                          |
| PT-03 | Cross-tenant FK insert                          | SQL         | Constraint violation                                                      |
| PT-04 | Runtime role cannot bypass RLS                  | SQL         | 0 rows without context; `rolbypassrls=false`                              |
| PT-05 | Context does not leak across pooled connections | Integration | Parallel requests isolated                                                |
| PT-06 | JWT expired/invalid/forged                      | API         | 401 envelope with stable code                                             |
| PT-07 | Deactivated membership                          | API         | 403 on next request                                                       |
| PT-08 | Permission matrix                               | API         | Expected allow/deny per role                                              |
| PT-09 | Store scope boundary                            | API         | Other store invisible; cross-tenant store id → 404                        |
| PT-10 | Invitation reuse/expiry                         | Integration | Rejected                                                                  |
| PT-11 | Tenant switch cache                             | E2E         | No stale data                                                             |
| PT-12 | Onboarding at 360px                             | E2E         | No horizontal overflow, all fields usable                                 |
| PT-13 | Audit completeness                              | Integration | Row per action with actor                                                 |
| PT-14 | Migration from zero                             | CI          | `db reset` success against a real PostgreSQL                              |
| PT-15 | Schema drift                                    | CI          | `db diff` empty                                                           |
| PT-16 | RLS membership predicate                        | SQL         | Member-of-A querying B returns 0 rows                                     |
| PT-17 | Default ACL + table grants                      | SQL/CI      | `anon`/`authenticated`/`service_role` have zero privileges                |
| PT-18 | `app_runtime` role attributes                   | SQL         | Non-owner, `NOBYPASSRLS`, no `SUPERUSER`, no DDL                          |
| PT-19 | Membership bootstrapping                        | SQL         | `list_my_memberships` works before tenant selection; direct SELECT denied |
| PT-20 | Token refresh + retry                           | Integration | Exactly one refresh and one retry; second failure logs out                |
| PT-21 | Cookie/session flags                            | Integration | HttpOnly, Secure, SameSite=Lax, Path=/; chunking handled                  |
| PT-22 | CSRF origin check                               | Integration | Missing/foreign Origin rejected on mutations                              |
| PT-23 | Store-scope precedence matrix                   | API         | S-01…S-12 all pass                                                        |
| PT-24 | Aggregate restriction                           | API         | Store-scoped actor cannot see tenant-wide totals                          |
| PT-25 | TLS with pinned CA                              | Integration | `verify-full` succeeds; no `rejectUnauthorized:false` in code             |
| PT-26 | Secret scan                                     | CI          | No `service_role`/DB password in bundle, logs, or repo                    |

---

## 9. Risks specific to Phase 1

| Risk                                               | Mitigation                                                                                                                                  |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| RLS policies slow on large tables                  | `EXPLAIN` fixtures; index `tenant_id`; membership predicate resolved via `SECURITY DEFINER` function, not a per-row join in the policy body |
| RLS mistaken for authentication                    | Documented trust boundary; API verifies membership; tests assert non-member → 0 rows                                                        |
| `set_config` omitted in one code path              | Single `withTenantTransaction` helper; lint/review rule; tests on every endpoint                                                            |
| Supabase pooler mode changes transaction semantics | Use transaction mode with `set_config(..., true)`; avoid session state/LISTEN/temp tables; test parallel requests                           |
| Custom role rejected by Supavisor                  | Verify in F1.2a on a throwaway/staging project; record the fallback before F1.2b                                                            |
| TLS verification quietly disabled                  | Pin the Supabase CA; code review + test reject `rejectUnauthorized:false`                                                                   |
| Default grants expose new tables                   | Revoke default privileges in the first migration; CI grant check                                                                            |
| Auth double-source (Supabase + local users)        | `public.users.auth_user_id` is a profile reference; identity source is `auth.users`; verify cross-schema FK support                         |
| Migration drift from ORM                           | Single migration folder; `db diff` CI gate                                                                                                  |
| Region latency discovered late                     | Decide before F1.2; recreating later costs a full migration                                                                                 |
| No real PostgreSQL                                 | Blocking entry criterion; do not fake DB tests                                                                                              |
| CI never executed                                  | Add a remote and a PostgreSQL service; do not call the exit gate met before it runs                                                         |

---

## 10. Phase 1 exit review

Present, with evidence:

1. Objective and scope delivered.
2. Files created/modified.
3. Skills actually used, named honestly (`design-system`, `ui-styling`, `ui-ux-pro-max` for UI work; `ponytail-review` for review). Do not list a skill that was not applied.
4. Decisions taken, with their approval record and ADR references.
5. Tests executed + actual results (counts and command output, not adjectives).
6. Isolation, RLS trust-boundary, permission, store-scope, auth/session, and grant evidence.
7. Known limitations and remaining risks, including the leaked-runtime-credential residual risk.
8. Recommendation for Phase 2 (catalog).

**Gate:** two-tenant isolation, membership-predicate, permission, and store-scope tests pass **in CI, on a remote, against real PostgreSQL**. If they do not, Phase 1 is not complete and Phase 2 does not start.

Any of the following means the gate is **not** met, regardless of local results:

- CI has never run.
- No real PostgreSQL was used.
- The runtime role is the owner or has `BYPASSRLS`.
- `anon`/`authenticated`/`service_role` still have tenant-table grants.
- Policies check tenant equality without membership.
- Store scope was tested only in the UI.

---

## 11. What I will not do without explicit approval

- Write catalog/inventory/sales code.
- Change the Supabase region or recreate the project (destructive).
- Apply any migration to the hosted project.
- Add billing, POS, website, offline, or integrations.
- Install Redis, queues, Kubernetes, or a second backend framework.
- Run destructive database operations against the hosted project.
- Mark any test as passing without executing it.
- Modify `01`–`08` documents.
- Unfreeze or extend the existing F1.1 code.

---

## 12. Immediate next action on approval

1. Record approved decisions in `DECISION-APPROVAL-MATRIX.md`, `docs/DECISION-LOG.md`, and the ADRs.
2. Rotate the exposed DB password; store secrets outside the repo.
3. Provision a real PostgreSQL (Docker, local, or dedicated staging) and push the repository to a remote with a PostgreSQL service.
4. Run **F1.2a** — the minimal RLS/runtime-role proof — and present actual SQL output.
5. Only then run F1.2b (full core schema), then F1.3–F1.7 in order.

**Nothing in steps 1–5 is authorized by this document. It is a proposed sequence.**

---

## 13. Document control

| Field                 | Value                                                  |
| --------------------- | ------------------------------------------------------ |
| Revision              | 2 (correction pass)                                    |
| Approved by           | — (pending)                                            |
| Approval date         | —                                                      |
| Implementation status | Frozen; existing F1.1 shell only; no migration applied |
| Next review           | After `DECISION-APPROVAL-MATRIX.md` is decided         |
