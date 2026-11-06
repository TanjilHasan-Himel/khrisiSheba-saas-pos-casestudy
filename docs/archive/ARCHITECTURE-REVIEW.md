# ARCHITECTURE REVIEW — Krishi Sheba SaaS

**Status:** Phase 0 correction pass (revision 2) — **awaiting approval; implementation is not authorized.**
**Date:** 2026-11-05
**Revision:** 2 — corrects revision 1 after the owner's conditional approval. See §0.
**Reviewer role:** Lead Software Architect
**Inputs reviewed:** `01-PRD.md`, `02-TRD.md`, `03-APP-FLOWS.md`, `04-UI-UX.md`, `05-DATABASE-SCHEMA.md`, `06-SECURITY-AND-COMPLIANCE.md`, `07-IMPLEMENTATION-PLAN.md`, `08-DECISIONS-AND-OPEN-QUESTIONS.md` (all read in full).

**Repository state at correction-pass time (verified, not assumed)**

- Git repository exists. Commit `4b5c2a5` contains the F1.0/F1.1 foundation: pnpm workspace, Fastify shell, Next.js shell, contracts/config/database packages, CI workflow, ADRs, and decision log.
- 14 unit/integration tests passed locally; `build` and a runtime smoke test passed. These are real results, reproduced during this pass.
- **No business features, no database migrations, no hosted-project changes.** `supabase/migrations` does not exist.
- The repository has **no remote**, so GitHub Actions has **never run**. "CI is configured" is true; "CI is green" would be false.
- Node `v24.13.0`; pnpm `10.34.9` installed under `C:\Users\Himel\.local\bin` (Program Files is not writable); git `2.53.0`.
- Docker Desktop **not installed**; `psql` **not installed**.
- Supabase CLI available through `npx supabase@3.0.0`; project `vqqjkigsumihfpmredpi` linked (Oceania / Sydney, created 2026-11-05).
- Hosted project verified read-only in this pass: PostgreSQL **17.11**, `public` schema has **0 tables**, `auth.users` has **0 rows**, transaction pooler on port 6543 is reachable from this machine. Details in §0.7.
- Direct DB host `db.<ref>.supabase.co` is **IPv6-only**; the Supavisor pooler is required from IPv4 networks.

**Agent skills verified on disk** (`%USERPROFILE%\.claude\skills`): `banner-design`, `brand`, `design`, `design-system`, `design-taste-frontend`, `ponytail`, `ponytail-audit`, `ponytail-debt`, `ponytail-gain`, `ponytail-help`, `ponytail-review`, `slides`, `ui-styling`, `ui-ux-pro-max`. No repository-local `AGENTS.md` or project skill directory exists. Relevant to this build: `design-system`, `ui-styling`, `ui-ux-pro-max` (UI milestones), `ponytail` / `ponytail-review` (implementation and review discipline). Not relevant: `sanity-*`, `slides`, `banner-design`, `report`. **Skills actually applied in this correction pass:** `ponytail-review` (review discipline: correctness, safety, load, tests, lean). No UI/design skill was applied because no UI was designed.

---

## 0. Correction pass (revision 2)

### 0.1 What this revision fixes

| Owner instruction                                              | Correction in this revision                                                                                                  | Where                          |
| -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| RLS tenant context vs trusted API authorization                | Explicit trust boundary, threat model, and corrected policy shape. RLS is defense-in-depth, **not** an authenticator.        | §0.2, §6                       |
| Supabase Auth / Next.js session / Fastify JWT / refresh / CSRF | End-to-end flow with exact responsibilities, failure paths, and the `service_role` exception.                                | §0.4, ADR 0003                 |
| M6 → M7 dependency                                             | M6 split into **M6a** (customers + credit policy, before sales) and **M6b** (collections + allocations, after posted sales). | `IMPLEMENTATION-BACKLOG.md` §3 |
| Store-scoped authorization with permission tests               | Precedence rules, enforcement points, and an endpoint × role × store-scope test matrix.                                      | §0.5, §7                       |
| Dedicated non-bypass runtime role                              | Role attributes, grants, ownership, pooling, TLS, and deployment feasibility.                                                | §0.6, §0.7                     |
| Verify infrastructure against the real environment             | Read-only host/database verification, plus an explicit list of what is still unverified.                                     | §0.7                           |

**What did not happen in this pass:** no application code was changed, no migration was created, and the hosted Supabase project was not modified. The only hosted access was a **read-only** transaction (`BEGIN READ ONLY`) used to verify facts.

### 0.2 RLS trust boundary — what RLS proves and does not prove

Revision 1 implied that RLS would prevent a forged tenant context. **That is not true by itself.** Any role that can execute SQL can call `set_config('app.tenant_id', '<other tenant>', true)`. A policy of the form `tenant_id = current_setting('app.tenant_id')` then returns the other tenant's rows. `set_config` is not a security primitive; it is a _transport_ for a decision the API already made.

The guarantee must be stated in layers:

| Layer                    | Owns                           | What it guarantees                                                                                                                                                                                                                                      |
| ------------------------ | ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Browser                  | Nothing security-critical      | Holds an httpOnly session cookie only. Never receives a database credential, `service_role` key, or raw access token.                                                                                                                                   |
| Next.js server           | Browser session, CSRF          | Authenticates the user with Supabase Auth, refreshes the session, and forwards a short-lived access token. Does not make authorization decisions about tenant data.                                                                                     |
| Fastify API              | Authentication + authorization | Verifies the JWT; derives `user_id` from `sub`; loads active memberships; derives the allowed tenant/store set; **never** treats a client-supplied tenant id, role, or store id as authority.                                                           |
| Database session context | API only                       | `app.user_id`, `app.tenant_id`, `app.store_id` are set **after** membership verification, inside the same transaction as the business statements.                                                                                                       |
| RLS                      | Database                       | Blocks statements that omit a tenant/store predicate and cross-tenant joins, **for the context the API set**. It does not authenticate the caller and cannot distinguish "the API set this" from "an attacker with `app_runtime` credentials set this". |
| Grants + network         | Platform                       | Browsers cannot reach PostgreSQL. `anon` / `authenticated` have no privileges on tenant tables. `app_runtime` is non-owner and `NOBYPASSRLS`.                                                                                                           |

**Threat model (explicit):**

| ID  | Threat                                                               | Mitigated by                                                                                                           | Residual risk                                                                                                                 |
| --- | -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| T1  | Untrusted browser forges `tenant_id`, `store_id`, `role`, or user id | Browser has no DB path; API derives all of these from the verified JWT + membership rows; cross-tenant ids return 404  | None if the API is correct                                                                                                    |
| T2  | Developer forgets `WHERE tenant_id` or writes a cross-tenant join    | RLS on every tenant table, `FORCE ROW LEVEL SECURITY`, composite FKs                                                   | Policy must exist on every table; tests must prove it                                                                         |
| T3  | SQL injection in the API                                             | Parameterized queries; RLS confines the blast radius to the current tenant/user                                        | A successful injection can still read the current tenant's data                                                               |
| T4  | Leaked `app_runtime` credential or compromised API host              | TLS, secret rotation, no public DB exposure, least-privilege grants, audit, anomaly detection                          | **RLS does not stop this attacker** - they can set the GUCs. This is an accepted residual risk only with the listed controls. |
| T5  | Forged or expired JWT                                                | JWKS signature, `iss`, `aud`, `exp`, `nbf`, `sub`; no roles/tenant trusted from the token                              | Key rotation/skew must be handled                                                                                             |
| T6  | Compromised Next.js server                                           | Can act only while a victim's access token is valid; cannot mint tokens; refresh rotation; short access-token lifetime | Session theft is possible if the server is fully compromised                                                                  |

**Consequence for acceptance criteria:** a test named "forging `app.tenant_id` without membership still yields no data" is misleading. The meaningful tests are:

1. A client cannot set `app.tenant_id` at all (no DB credential, no SQL surface).
2. With `app.tenant_id = A` and `app.user_id = U`, RLS returns only tenant A rows **and only if U is an active member of A**.
3. With `app.tenant_id = B` and `app.user_id = U` (U not a member of B), RLS returns 0 rows even if the caller knows B's UUID.
4. With no context set, RLS returns 0 rows.
5. Cross-tenant composite-FK inserts fail.
6. A leaked-runtime-credential test is documented as a residual risk, not claimed as mitigated by RLS.

### 0.3 Revised tenant isolation model

The design keeps one database, shared schema, `tenant_id` on every tenant-owned table, composite `(tenant_id, id)` keys, and composite FKs. Four changes are required:

1. **Policies must include membership, not just tenant equality.** A tenant equality check alone trusts the GUC too much. Add a `SECURITY DEFINER` predicate keyed by `app.user_id`:

   ```sql
   -- Owned by the migration role (postgres), not by app_runtime.
   -- Body is fixed; no dynamic SQL; search_path pinned.
   create or replace function public.is_tenant_member(p_user uuid, p_tenant uuid)
   returns boolean
   language sql
   stable
   security definer
   set search_path = public, pg_temp
   as $$
     select exists (
       select 1
       from public.tenant_memberships m
       where m.user_id = p_user
         and m.tenant_id = p_tenant
         and m.status = 'ACTIVE'
     );
   $$;

   revoke all on function public.is_tenant_member(uuid, uuid) from public, anon, authenticated;
   grant execute on function public.is_tenant_member(uuid, uuid) to app_runtime;
   ```

   ```sql
   create policy tenant_isolation on public.customers
     using (
       tenant_id = nullif(current_setting('app.tenant_id', true), '')::uuid
       and public.is_tenant_member(
         nullif(current_setting('app.user_id', true), '')::uuid,
         tenant_id
       )
     )
     with check (
       tenant_id = nullif(current_setting('app.tenant_id', true), '')::uuid
       and public.is_tenant_member(
         nullif(current_setting('app.user_id', true), '')::uuid,
         tenant_id
       )
     );

   alter table public.customers enable row level security;
   alter table public.customers force row level security;
   ```

2. **Bootstrap membership reads must not recurse.** `tenant_memberships` itself needs RLS, but the API must load a user's memberships before a tenant is selected. Resolve this with a narrow `SECURITY DEFINER` function owned by `postgres` (which has `BYPASSRLS`, verified) that returns only the caller's memberships:

   ```sql
   create or replace function public.list_my_memberships(p_user uuid)
   returns table (tenant_id uuid, role_code text, status text)
   language sql stable security definer
   set search_path = public, pg_temp
   as $$
     select m.tenant_id, r.code, m.status
     from public.tenant_memberships m
     join public.roles r on r.id = m.role_id
     where m.user_id = p_user;
   $$;
   ```

   The API calls this with the verified `sub`; the function never accepts a tenant id. Do not grant `SELECT` on `tenant_memberships` directly to `app_runtime`.

3. **Revoke the default Supabase grants.** This is not optional. Read-only verification found that future tables created in `public` by `postgres` currently grant **ALL** (`arwdDxtm`) to `anon`, `authenticated`, and `service_role` (see §0.7). If those grants remain, the publishable key can reach tenant tables through PostgREST before RLS is even considered. Every migration must include, or a single bootstrap migration must establish:

   ```sql
   alter default privileges for role postgres in schema public
     revoke all on tables from anon, authenticated, service_role;
   alter default privileges for role postgres in schema public
     revoke all on sequences from anon, authenticated, service_role;
   alter default privileges for role postgres in schema public
     revoke all on functions from anon, authenticated, service_role;

   revoke all on all tables in schema public from anon, authenticated, service_role;
   revoke all on all sequences in schema public from anon, authenticated, service_role;
   revoke all on all functions in schema public from anon, authenticated, service_role;
   ```

   Objects created by `supabase_admin` have a separate default ACL that also grants `anon`/`authenticated`. The migration process must not create application tables through the dashboard or as `supabase_admin`; a CI check should fail if any tenant table has a grant to `anon` or `authenticated`. Disabling the Data API is recommended if nothing needs it; if it stays enabled, the revokes above are mandatory.

4. **Store scope is enforced in the service layer and mirrored in RLS where the table is store-scoped.** `app.store_id` has the same trust caveat as `app.tenant_id`. It is set by the API after verifying that the store belongs to the verified tenant and is in the actor's allowed store set. Policies for store-scoped tables add `store_id = current_setting('app.store_id', true)::uuid`. Tables without `store_id` rely on tenant scope only.

**Transaction pattern (unchanged in shape, corrected in meaning):**

```text
BEGIN;
  SELECT set_config('app.user_id',   $verifiedUserId, true);
  SELECT set_config('app.tenant_id', $verifiedTenantId, true);
  SELECT set_config('app.store_id',  $verifiedStoreId, true); -- optional
  -- The membership/store checks that produced these values ran in this transaction.
  ... business statements ...
COMMIT;
```

`set_config(..., true)` remains correct for transaction pooling. It is a convenience for policies, not proof of identity.

### 0.4 Auth, session, refresh, and CSRF flow

**Decision under review:** Supabase Auth is the identity provider; the API is a Bearer-only resource server; the browser never talks to Supabase or PostgreSQL directly. All of this remains **proposed**, not approved.

```text
Browser ──HTTPS──► Next.js (apps/saas)
                     │  1. sign-in / sign-up / reset via server client (PKCE)
                     │  2. session in httpOnly cookie (server-only Supabase client)
                     │  3. middleware refreshes with supabase.auth.getUser()
                     │  4. server-side fetch: Authorization: Bearer <access_token>
                     ▼
                  Fastify API (apps/api)
                     │  5. verify JWT via JWKS (iss, aud, exp, nbf, sub)
                     │  6. load memberships/stores/permissions from DB
                     │  7. authorize, open transaction, set context, execute
                     ▼
                  PostgreSQL as app_runtime (RLS)
```

**Exact responsibilities and failure paths:**

1. **Sign-in.** The Next.js route handler creates a `@supabase/ssr` server client with a cookie adapter and calls `signInWithPassword` (or the OAuth/magic-link exchange). No browser-side Supabase client is required. Cookies are set with `httpOnly: true`, `secure: true` in production, `sameSite: 'lax'`, `path: '/'`. The access token is never written to `localStorage`, `sessionStorage`, a URL, or a client-readable cookie.
2. **Refresh.** Middleware runs on every request that can reach a protected page. It calls `supabase.auth.getUser()` — which validates the token with the Auth server — **not** `getSession()`, which only reads the cookie. If the access token is expired and a refresh token is present, the Supabase client refreshes and the cookie adapter writes the rotated tokens. If refresh fails (revoked, reused, expired), the server clears cookies and redirects to `/login`. Refresh-token reuse detection is handled by Supabase Auth; the app must treat a refresh failure as a full logout.
3. **Token forwarding.** The page/route handler reads the current session server-side and calls the API with `Authorization: Bearer <access_token>`. It may send `X-Tenant-Id` as a _hint_, never as authority. The access token is not forwarded to the browser.
4. **JWT verification (API).** Use `jose` with a cached remote JWKS (`https://<ref>.supabase.co/auth/v1/.well-known/jwks.json`). Verify signature, `iss` (`https://<ref>.supabase.co/auth/v1`), `aud` (`authenticated`), `exp`, `nbf` (small clock skew, e.g. 30 s), and `sub`. Reject `alg: none` and algorithms not in the allow-list. Return `401 TOKEN_INVALID` or `401 TOKEN_EXPIRED` in the stable envelope. Do not accept a cookie as authentication. Do not accept a token in a query string.
5. **Principal load.** Resolve `users.id` from `sub`. Load active tenant memberships (via `list_my_memberships`), active store memberships, and permissions. If `X-Tenant-Id` is absent, choose the only active membership or require a selection; if present, it must be in the verified set, otherwise `404 TENANT_NOT_FOUND` (not 403, to avoid enumeration). Never trust a role or permission claim in the JWT.
6. **Expiry during a request.** If the API returns `TOKEN_EXPIRED`, Next.js refreshes once and retries the API call once. Do not loop. If the retry still fails, log out.
7. **CSRF.** CSRF is a property of the **cookie-bearing Next.js layer**, not the Bearer-only API.
   - `SameSite=Lax` blocks cross-site POSTs from other origins.
   - Every non-GET route handler and every Server Action must verify `Origin` against the configured app origin (and `Host`); reject missing/foreign origins. Next.js Server Actions perform an origin check by default, but custom route handlers do not.
   - No state-changing GET. No JSONP. No wildcard CORS with credentials.
   - A double-submit CSRF token is optional defense-in-depth; if added, it must be bound to the session and checked server-side.
   - The API must **reject** cookie authentication and must **not** enable `credentials: true` CORS for browser use. The current F1.1 shell enables `credentials: true`; that is a defect to fix when implementation is unfrozen.
8. **Logout and revocation.** Logout calls `signOut({ scope: 'global' })`, clears cookies, and redirects. Deactivating a staff member must (a) mark the membership inactive so the next API request fails, and (b) revoke that user's refresh tokens through the GoTrue admin API. Step (b) requires `service_role` (or a server-side admin key) — see the exception below.
9. **`service_role` exception.** "Never use `service_role` in application code" is too absolute and contradicts deactivation/revocation. Correct rule: `service_role` may be used **only** in a narrow server-side admin module (or a one-off operational script) for GoTrue admin operations. It must never be in the browser bundle, never in a tenant request path, never logged, and never used to read tenant business data. Store it in the server host's secret store, not in `.env` committed to the repo. Prefer DB triggers for membership state and the admin API only for token revocation.
10. **Cookie size.** Supabase session cookies can be large; the adapter chunks them. Verify header size limits on the chosen host and set `maxAge` to the refresh-token lifetime. Do not assume a single cookie.
11. **Email verification.** Supabase's built-in SMTP has low rate limits. Custom SMTP is required before real onboarding; until then, staging/pilot onboarding is limited. This is a launch dependency, not a code detail.

### 0.5 Store-scoped authorization

**Model under review (revises `B-08` / `G-07`):**

- `tenant_memberships(tenant_id, user_id, role_code, status)` is the **only** source of role. `store_memberships(tenant_id, store_id, user_id, status)` has **no role column**; it only narrows visibility. Revision 1's `store_memberships.role_id` is removed from the design because it creates a second source of truth.
- Effective permission = tenant role permission **∩** store scope. Store membership can never add a permission.
- Precedence:
  1. Platform principal is not a tenant member and cannot call tenant endpoints.
  2. `OWNER`: all stores in the tenant, always. Store membership rows do not restrict the owner (they may set UI defaults only).
  3. `MANAGER`: all stores **if no active store membership rows exist**; if rows exist, only those stores.
  4. `SALES` / `STOCK`: require at least one explicit active store membership. None means no store access (`403 STORE_SCOPE_REQUIRED`).
  5. Revocation takes effect on the next request. No long-lived authorization cache.
- Enforcement:
  - Every route declares whether it is tenant-scoped, store-scoped, or platform-scoped.
  - The service resolves `store_id` from the path/body, then checks `(tenant_id, store_id)` belongs to the tenant and is in the principal's allowed set.
  - The client never sends an authoritative store list. `X-Store-Id` is a hint at most; the verified value is written to `app.store_id`.
  - A store id from another tenant returns `404`, not `403`, to avoid enumeration.
  - List endpoints filter by the allowed store set in SQL, not in the UI.
  - Aggregates/reports must group by the allowed set; a store-scoped user must not see tenant-wide totals.

**Required permission tests (endpoint × role × store-scope):**

| Case | Actor   | Store rows                                      | Expected                                             |
| ---- | ------- | ----------------------------------------------- | ---------------------------------------------------- |
| S-01 | OWNER   | none                                            | all tenant stores visible                            |
| S-02 | OWNER   | one                                             | still all stores; rows do not restrict               |
| S-03 | MANAGER | none                                            | all tenant stores visible                            |
| S-04 | MANAGER | store A only                                    | store B returns 404; lists exclude B                 |
| S-05 | SALES   | none                                            | `403 STORE_SCOPE_REQUIRED` on store-scoped routes    |
| S-06 | SALES   | store A                                         | store A allowed; store B returns 404                 |
| S-07 | STOCK   | store A                                         | stock writes allowed at A; store B 404; sales denied |
| S-08 | SALES   | store A (tenant A), request store from tenant B | 404, no data                                         |
| S-09 | SALES   | store A, direct object id from store B          | 404, no data                                         |
| S-10 | SALES   | store membership revoked mid-session            | next request denied                                  |
| S-11 | MANAGER | store A, tenant-wide report                     | report restricted to A                               |
| S-12 | any     | forged `X-Tenant-Id` / `X-Store-Id`             | ignored; 404 if not in verified set                  |

### 0.6 Dedicated runtime role: grants, ownership, pooling, deployment feasibility

**Role shape (proposed):**

```sql
create role app_runtime
  login
  password '<from secret store>'
  nosuperuser nocreatedb nocreaterole noinherit nobypassrls;

revoke all on schema public from app_runtime;
grant usage on schema public to app_runtime;

-- Example per-table grants; the exact list is generated by migration review.
grant select, insert, update on public.customers to app_runtime;
grant select, insert on public.audit_events to app_runtime;
revoke update, delete on public.audit_events from app_runtime;

grant usage, select on all sequences in schema public to app_runtime;
grant execute on function public.is_tenant_member(uuid, uuid) to app_runtime;
grant execute on function public.list_my_memberships(uuid) to app_runtime;
```

**Ownership and RLS:**

- Every table is owned by the migration role (`postgres`, verified `BYPASSRLS` + `CREATEROLE`). `app_runtime` owns nothing.
- `ENABLE` + `FORCE ROW LEVEL SECURITY` on every tenant table. Without `FORCE`, an owner bypasses RLS; `app_runtime` is not the owner, but `FORCE` prevents future mistakes (for example a table accidentally created as `app_runtime`).
- `app_runtime` must not have `CREATE` on `public` (verified: `anon`/`authenticated` already lack it; check `app_runtime` too).
- Append-only tables get `SELECT, INSERT` only (no `UPDATE`, no `DELETE`). Status transitions go through narrow `SECURITY DEFINER` functions or explicit `UPDATE` grants on headers, reviewed per table.
- `service_role` is not a runtime fallback and is not granted tenant data.

**Pooling and deployment — verified vs unverified:**

| Item                                                   | Status                    | Evidence / action                                                                                                                                                                  |
| ------------------------------------------------------ | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Transaction pooler reachable on 6543 from this machine | **Verified**              | Read-only `SELECT` succeeded through `aws-0-ap-southeast-2.pooler.supabase.com:6543`; backend `PostgreSQL 17.11`                                                                   |
| `set_config(..., true)` works in a transaction         | **Verified for SELECT**   | The read-only transaction used `BEGIN READ ONLY`; no cross-transaction leak test yet                                                                                               |
| Custom `app_runtime` role accepted by Supavisor        | **Unverified**            | Requires creating the role on the hosted project; not done in this pass. Verify on a staging/throwaway project before relying on it.                                               |
| `CREATE ROLE` privilege                                | **Verified**              | `postgres` has `rolcreaterole = true`                                                                                                                                              |
| Session-level `SET`, `LISTEN`, session advisory locks  | **Must not be used**      | Transaction pooling may hand the next transaction a different backend; use `set_config(..., true)` and `pg_advisory_xact_lock` only                                                |
| Named prepared statements                              | **Avoid**                 | The `pg` driver uses unnamed statements by default; do not add names that assume session affinity                                                                                  |
| Temporary tables across statements                     | **Avoid**                 | Session-scoped; not safe through a transaction pooler                                                                                                                              |
| Direct DB host from IPv4                               | **Unavailable**           | AAAA-only; use the pooler                                                                                                                                                          |
| TLS to the pooler                                      | **Needs a CA**            | Node's default trust store rejected the chain (`self-signed certificate in certificate chain`). Ship a pinned Supabase CA / `sslrootcert`; never ship `rejectUnauthorized: false`. |
| Migrations through transaction pooler                  | **Avoid**                 | Use a session-mode connection (5432) or direct connection for DDL/advisory locks. `supabase db push` as `postgres`.                                                                |
| `DATABASE_URL` role                                    | **Must be `app_runtime`** | The API must never connect as `postgres`; `postgres` has `BYPASSRLS` (verified)                                                                                                    |

**Fallback if Supavisor rejects a custom role** (decide only after verification):

1. Dedicated/session pooler with the custom role, if supported.
2. Direct IPv6 connection from a host with IPv6 egress (most Singapore hosts do).
3. A separate PostgreSQL instance for runtime traffic — not attractive at this scale.
4. Connecting as `postgres` and using `SET LOCAL ROLE app_runtime` per transaction — **rejected as a primary design** because the API would hold a `BYPASSRLS` credential and could reset the role. If used as a temporary fallback, it must be recorded as a residual risk and removed before pilot.

### 0.7 Infrastructure verification (read-only, this pass)

The following was verified against the linked hosted project using a `BEGIN READ ONLY` transaction. No writes were made.

| Check                                                 | Result                                                                       | Consequence                                                                    |
| ----------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| PostgreSQL version                                    | 17.11                                                                        | Compatible with the schema plan; verify extensions if any are added            |
| Connection path                                       | Supavisor transaction pooler, port 6543, succeeded from this Windows machine | Pooler works for reads from IPv4                                               |
| `postgres` attributes                                 | `LOGIN`, `BYPASSRLS`, `CREATEROLE`, `CREATEDB`, **not superuser**            | `postgres` is the migration/owner role; never the runtime role                 |
| `anon` / `authenticated`                              | `NOLOGIN`, `NOBYPASSRLS`, `USAGE` on `public`, **no `CREATE`**               | Safe only if table grants are revoked; they can still use PostgREST if granted |
| `service_role`                                        | `NOLOGIN`, `BYPASSRLS`                                                       | Use only server-side admin; never runtime                                      |
| `app_runtime`                                         | Does not exist                                                               | Must be created and verified in F1.2                                           |
| Tables in `public`                                    | 0                                                                            | No migration has been applied                                                  |
| `auth.users`                                          | 0 rows                                                                       | Project is empty                                                               |
| Default ACL for `postgres`-created tables in `public` | `anon`, `authenticated`, `service_role` receive **ALL** (`arwdDxtm`)         | **Critical:** revoke default privileges before creating any tenant table       |
| Default ACL for `supabase_admin`-created tables       | Also grants `anon`/`authenticated`                                           | Do not create app tables as `supabase_admin`; add a CI grant check             |
| TLS chain                                             | Not trusted by Node's default store                                          | Pin the Supabase CA; do not disable verification                               |
| Region                                                | Sydney (`ap-southeast-2`)                                                    | Still wrong for Bangladesh users; recreation remains pending approval          |
| Paid tier / PITR / backups                            | **Unverified**                                                               | Must be confirmed before pilot                                                 |
| Custom SMTP / Auth email limits                       | **Unverified**                                                               | Must be configured before real onboarding                                      |
| JWT expiry / refresh rotation settings                | **Unverified**                                                               | Read from the Auth settings before finalizing token lifetimes                  |
| Docker Desktop / `psql`                               | Not installed                                                                | No local Supabase stack; real-PostgreSQL tests still blocked locally           |
| GitHub remote / Actions                               | No remote; never run                                                         | CI claims must say "configured, not executed"                                  |
| `supabase/migrations`                                 | Does not exist                                                               | F1.2 has not started                                                           |

### 0.8 Unsupported or incorrect claims corrected

| #    | Revision 1 claim                                                                               | Correction                                                                                                                                                                            |
| ---- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| U-01 | "RLS prevents a forged tenant context."                                                        | False as stated. RLS enforces the context the API sets; a DB-capable attacker can set it. See §0.2.                                                                                   |
| U-02 | "Forging `app.tenant_id` without a membership still yields no data (API verifies membership)." | Only true because the _API_ verifies membership, not because the database can tell who set the GUC. Add the membership predicate and state the residual risk.                         |
| U-03 | "No application code has been written."                                                        | False. Commit `4b5c2a5` contains the F1.0/F1.1 foundation. It is frozen, not approved.                                                                                                |
| U-04 | "PostgREST stays enabled only as a health/debug channel."                                      | If the Data API is enabled, the default grants expose new tables to `anon`/`authenticated` until revoked. Either disable the Data API or revoke defaults and verify in CI.            |
| U-05 | "`app_runtime` works through the pooler."                                                      | Role creation privilege is verified; Supavisor's acceptance of a custom role is **not**. Verify before F1.2 depends on it.                                                            |
| U-06 | "Docker is required."                                                                          | Docker is one way to get a real PostgreSQL. Local Postgres or a dedicated staging database is acceptable; what is mandatory is a real database for RLS/concurrency tests.             |
| U-07 | "CI is green / runs all checks."                                                               | The workflow is configured; the repository has no remote and Actions has never run.                                                                                                   |
| U-08 | "Phase 1 is in progress."                                                                      | Code exists but is frozen pending this correction pass and owner approval.                                                                                                            |
| U-09 | "`set_config(..., true)` makes pooling safe."                                                  | It removes cross-transaction GUC leakage, but session state, session advisory locks, `LISTEN`, and temp tables are still unsafe through a transaction pooler.                         |
| U-10 | "`httpOnly` cookie" (as if automatic).                                                         | `@supabase/ssr` must be configured server-only with explicit cookie options; verify flags in F1.4. A browser Supabase client would require JS-readable cookies and break this design. |
| U-11 | "`service_role` is never used in application code."                                            | Deactivation/refresh-token revocation needs it in a narrow server-side admin module. Define the exception; do not put it in the browser or tenant path.                               |
| U-12 | "30-day idempotency retention / 7-day returns / 90-day expiry / 50k export cap"                | These are **proposed defaults**, not approved requirements. They are not acceptance criteria until approved.                                                                          |

### 0.9 Remaining contradictions / acceptance criteria that cannot yet be accepted

These are additions to `C-01`–`C-16`; the existing IDs are kept stable.

| ID       | Contradiction or unsupported acceptance criterion                                                              | Resolution needed                                                                                                                                          |
| -------- | -------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **C-17** | F1.1's API enables CORS with `credentials: true`, while the architecture says the browser never calls the API. | Remove browser CORS/credentials when unfrozen; keep the API server-to-server only.                                                                         |
| **C-18** | "Never use `service_role`" vs "revoke refresh tokens on deactivation".                                         | Approve the narrow server-side admin exception (§0.4 step 9).                                                                                              |
| **C-19** | "httpOnly cookie" vs a browser-side Supabase client (which needs JS-readable cookies).                         | Commit to a server-only Supabase client; do not add a browser client that reads the session.                                                               |
| **C-20** | `store_memberships.role_id` vs "store membership only narrows".                                                | Remove the role column from store memberships; role comes only from tenant membership.                                                                     |
| **C-21** | "RLS forced on every tenant table" vs the need to read memberships before a tenant is chosen.                  | Approve the narrow `SECURITY DEFINER` bootstrap function (§0.3).                                                                                           |
| **C-22** | M6 includes collections, but posted sales exist only in M7.                                                    | Split M6a/M6b; M6b depends on M7.                                                                                                                          |
| **C-23** | "Docker required" vs "hosted DB may be used for development".                                                  | Choose one local/staging database strategy before F1.2; tests must use real PostgreSQL.                                                                    |
| **C-24** | Phase 1 exit gate says "isolation tests pass in CI", but CI has no database service and no remote.             | Add a Postgres service to CI and push the repo before claiming the gate.                                                                                   |
| **C-25** | "`app_runtime` logs in via the pooler" vs Supavisor custom-role support is unverified.                         | Verify on a throwaway/staging project; record the fallback.                                                                                                |
| **C-26** | "No production code written" vs commit `4b5c2a5`.                                                              | Correct the record; freeze the code until approval.                                                                                                        |
| **C-27** | "Email verification required" vs built-in SMTP rate limits and no custom SMTP.                                 | Configure custom SMTP before onboarding; otherwise acceptance is blocked.                                                                                  |
| **C-28** | "Platform MFA required in P0" vs platform ops scheduled at M10.                                                | Keep M10 before pilot; do not claim P0 complete without it.                                                                                                |
| **C-29** | "Store scope in RLS where feasible" vs `app.store_id` is forgeable.                                            | Enforce in the service layer first, mirror in RLS; do not claim RLS is the source of truth.                                                                |
| **C-30** | `public.users.id` references `auth.users.id`, but also stores email/phone; the two can drift.                  | Keep `public.users` as a profile table with a trigger or a view; do not treat it as the identity source. Verify cross-schema FK support on Supabase 17.11. |
| **C-31** | `roles.tenant_id NULL` breaks the `tenant_id NOT NULL` convention.                                             | Use system roles as code constants + a tenant-less reference table with explicit policy, or keep nullable with a documented exception.                     |
| **C-32** | `platform_operators` in `public` sits beside tenant tables.                                                    | Put platform data in a separate schema or exclude it from tenant RLS/grant rules explicitly.                                                               |
| **C-33** | Payment allocation "oldest-first" is a business rule, not a derived truth.                                     | Approve it; otherwise let the collector choose invoices.                                                                                                   |
| **C-34** | Credit limit `0 = no credit` vs `NULL = unlimited` is undefined.                                               | Define both values and the default before the customers migration.                                                                                         |
| **C-35** | "Overpayment rejected" vs real-world advance payments.                                                         | Confirm rejection; if advances are needed, add a customer credit ledger (P1 scope).                                                                        |

### 0.10 Implementation state and freeze

- F1.0/F1.1 code exists at `4b5c2a5`. It is **not approved** and must not be extended or deployed until the decisions in `DECISION-APPROVAL-MATRIX.md` are resolved.
- Known defects in the frozen code, to fix on unfreeze:
  - `package.json` `clean` uses `rm -rf`, which fails in PowerShell.
  - API CORS enables `credentials: true` despite the server-to-server-only design.
  - `.env.example` does not yet list the Supabase Auth/session/admin variables.
  - No database, no auth, no RLS, no migrations. Tests cover only the shell.
- The hosted Supabase project remains empty and unchanged. The DB password shared during setup should be **rotated** before any real deployment.

---

## 1. Executive verdict

The architecture direction is **sound and should be approved with corrections**. The specification already avoids the usual failure modes: no microservices, no AI features, no offline-authoritative financial posting, money as minor units, immutable posted documents, ledger-derived dues, tenant-scoped data, and a clear MVP/P1/P2 split.

The package is **not yet implementation-ready**. There are:

- **8 blocking decisions** that change the schema or the runtime shape (Section 3, `B-01`–`B-08`), now supplemented by correction-pass decisions `D-01`–`D-10`.
- **16 internal contradictions** between documents (Section 4, `C-01`–`C-16`) plus **19 new** contradictions/unsupported acceptance criteria (`C-17`–`C-35`, §0.9).
- **15 architectural risks**, two of which are environmental and block work today (`R-01` test database, `R-02` Supabase region).
- **Six security corrections** in §0: the RLS trust boundary was overstated, the auth/CSRF flow was underspecified, M6/M7 was inverted, store scope had two sources of truth, the runtime role/pooler was unverified, and the hosted environment had not been checked against reality.

Nothing here requires abandoning the design. It requires decisions before the first migration, because six of them are expensive to reverse after data exists.

**Recommendation:** approve the architecture direction, resolve the blocking and correction-pass decisions in `DECISION-APPROVAL-MATRIX.md`, keep the existing F1.1 code frozen, then run the smallest safe Phase 1 start described in `PHASE-1-EXECUTION-PLAN.md` §1.2.

---

## 2. What is validated and should not change

| #    | Decision                                                                           | Why it holds                                                                                                 |
| ---- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| V-01 | Modular monolith, single PostgreSQL database, shared schema with `tenant_id`       | Correct for this scale. Distributed services would multiply failure modes without a proven load requirement. |
| V-02 | Separate deployables for SaaS, API, and later POS/website/platform admin           | Keeps operational UI independent from tenant UI without microservices.                                       |
| V-03 | Server-authoritative pricing, totals, and stock                                    | Matches PRD rules 2, 4, 7 and the financial threat model.                                                    |
| V-04 | Append-only stock movements with materialized balances                             | Auditable, reconcilable, and testable. The alternative (balance-only) is unrecoverable after a bug.          |
| V-05 | Ledger-derived customer dues                                                       | Prevents the classic "mutable balance drifted from invoices" defect.                                         |
| V-06 | Idempotency keys on money/stock POSTs, stored response replay                      | Necessary for retries on unreliable mobile networks.                                                         |
| V-07 | Composite `(tenant_id, id)` keys and composite FKs                                 | The only reliable way to make cross-tenant references impossible at the database layer.                      |
| V-08 | RLS as defense-in-depth, not the only authorization layer                          | Correct: RLS cannot express all business rules, and API authorization cannot be trusted alone.               |
| V-09 | No negative stock by default; reversal instead of editing                          | Correct default for a money-handling product.                                                                |
| V-10 | Deferring subscription billing, external POS connectors, COGS, and offline posting | Keeps P0 shippable.                                                                                          |

---

## 3. Blocking decisions (resolve before the first migration)

Each is a decision that is cheap now and expensive after data exists. Full analysis and proposed defaults are in `SPECIFICATION-GAPS.md`; IDs match.

| ID       | Decision                                                                              | Why it blocks                                                                                                                                | Proposed default                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| -------- | ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **B-01** | Who talks to PostgreSQL: a dedicated API only, or browser → PostgREST/Supabase client | Determines auth, RLS policy shape, transaction boundaries, idempotency location, and the entire `apps/api` justification                     | **API-only.** Browser never holds a DB-capable key; all tenant reads and writes go through the API. Revoke `anon`/`authenticated`/`service_role` grants on tenant tables **and** the default privileges that grant future tables, or disable the Data API. PostgREST is never a client path. See §0.3.                                                                                                                                            |
| **B-02** | Unit-of-measure strategy: base-unit-only vs full conversions (carton → packet → kg)   | Catalog, purchasing, and sales quantities cannot be modelled correctly without it; it changes `products`, `units`, and every quantity column | **P0: one base unit per sellable SKU, purchase and sale in that unit.** No carton/pack conversion in P0. Revisit with pilot data before Phase 2 ends.                                                                                                                                                                                                                                                                                             |
| **B-03** | Batch tracking: mandatory sentinel batch vs nullable batch with `NULLS NOT DISTINCT`  | Determines inventory balance uniqueness, FKs, and every stock query                                                                          | **`batch_id NOT NULL` with a lazily created sentinel batch per product** where `batch_tracked = false`. Simpler joins, no NULL semantics, works with composite FKs.                                                                                                                                                                                                                                                                               |
| **B-04** | Payment allocation and credit rules                                                   | Without an allocation table, partial payments and dues across multiple invoices cannot be represented correctly                              | **Add `payment_allocations`.** P0: collection applies **oldest posted invoice first**, overpayment **rejected**. Credit limit checked under customer row lock.                                                                                                                                                                                                                                                                                    |
| **B-05** | Migration source of truth                                                             | Two generators (Drizzle Kit and Supabase CLI) will drift and corrupt history                                                                 | **Authoritative SQL lives in `supabase/migrations`.** Drizzle is the typed query layer; `drizzle-kit generate` output is reviewed and committed into that folder, never applied independently.                                                                                                                                                                                                                                                    |
| **B-06** | Local environment/CI runtime                                                          | Docker is absent; `supabase start`, Testcontainers, and local Postgres tests cannot run                                                      | **Provide a real PostgreSQL for tests** — Docker Desktop, a local PostgreSQL, or a dedicated staging database. `supabase start`, Testcontainers, and RLS/concurrency tests are not credible without a real PostgreSQL.                                                                                                                                                                                                                            |
| **B-07** | Supabase region and paid tier                                                         | Project is in **Sydney**; target users are in Bangladesh. Region cannot be changed after creation                                            | **Recreate the (empty) project in Singapore or Mumbai before any data**, and confirm plan level supports daily backups/PITR before pilot.                                                                                                                                                                                                                                                                                                         |
| **B-08** | Role model and store-scope precedence                                                 | Two membership tables with independent `role_id` have no defined precedence; permissions can disagree                                        | **Fixed system roles for P0** (`OWNER`, `MANAGER`, `SALES`, `STOCK`), tenant membership defines the role, and `store_memberships` has **no role column** — it can only narrow data scope. Effective permission = tenant role ∩ store scope. OWNER always sees all stores; MANAGER sees all unless explicit store rows narrow it; SALES/STOCK require explicit rows. Platform operator is a separate principal, not a tenant membership. See §0.5. |

---

## 4. Contradictions found across documents

| ID       | Conflict                                                                                                                                                       | Documents                                              | Why it matters                                                                             | Proposed resolution                                                                                                                                                                                            |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **C-01** | Identity/verification is "to be selected", yet the security doc mandates MFA for platform operators and "ideally owners", and the TRD mandates a dedicated API | PRD §3, §8; SEC §3; TRD §1, §4                         | Auth choice determines session model, RLS claim source, invite flow, and API contract      | Use **Supabase Auth** (already provisioned) for identity. Next.js holds an httpOnly cookie session; API verifies the Supabase JWT. Platform operators require TOTP MFA in P0; owner MFA deferred and recorded. |
| **C-02** | RBAC is modelled as dynamic `roles`/`role_permissions` rows, but the security doc defines a fixed role matrix                                                  | DB §2; SEC permission matrix; PRD "store-scoped roles" | Custom roles multiply authorization test surface and UI work with no P0 demand             | Fixed role codes + permission constants seeded into the DB. No tenant-authored roles in P0.                                                                                                                    |
| **C-03** | Plans/subscriptions appear in the schema and platform flow, but subscription billing is explicitly P1                                                          | DB §6; APP-FLOWS H; PRD §3 P1                          | Building billing tables in P0 creates unused schema and false expectations                 | Keep the tables out of the P0 migration. Platform ops in P0 = suspend/reactivate tenant + view audit only.                                                                                                     |
| **C-04** | TRD requires a transactional outbox, but P0 has no async consumer                                                                                              | TRD §6; PRD P0                                         | An outbox with no worker is dead weight and extra write per transaction                    | Do not create `outbox_events` until notifications or POS sync exist.                                                                                                                                           |
| **C-05** | Availability is defined as `on_hand − reserved − blocked`, but there is no reservation flow and no blocked column                                              | DB §4; PRD rule 6; APP-FLOWS C                         | Implementers will either invent a reservation feature or silently ignore the formula       | P0 availability = `on_hand` (batch must be sellable). `reserved_qty` kept at 0 and documented as unused, or omitted. Quarantine is expressed by batch status.                                                  |
| **C-06** | "Soft deletion only for appropriate master data" is stated, but no `deleted_at` exists anywhere                                                                | DB §1; DB §2–6                                         | Status enums and soft delete will be mixed ad hoc                                          | Use explicit `status` values (`ACTIVE`, `INACTIVE`, `ARCHIVED`) for master data. No `deleted_at`; deletions are status changes.                                                                                |
| **C-07** | Runtime role must not bypass RLS, but Supabase's default `postgres` role does bypass it                                                                        | TRD §4; SEC §2                                         | Using the linked `postgres` user from the API silently defeats the entire isolation design | Create a dedicated `app_runtime` role with `NOLOGIN`-derived login, explicit grants, and `FORCE ROW LEVEL SECURITY`. Migrations run as owner; requests run as `app_runtime`.                                   |
| **C-08** | P0 includes optional product images; security requires upload validation/scanning; no storage design exists                                                    | PRD §3; SEC §8; TRD §7                                 | Images add Storage RLS, MIME/size validation, and a public-vs-signed URL decision          | Defer product images to P1 unless the pilot explicitly needs them. If kept: private bucket, tenant-prefixed paths, signed URLs, 2 MB limit, JPEG/PNG/WebP only.                                                |
| **C-09** | Purchase flow has "APPROVED (optional)" but no permission, rule, or approver is defined                                                                        | APP-FLOWS A/B; SEC matrix                              | Optional states without rules become inconsistent code paths                               | P0: no approval state. `DRAFT → PARTIALLY_RECEIVED → RECEIVED`, `DRAFT → CANCELLED`. Approval is P1.                                                                                                           |
| **C-10** | Platform operator scope is "suspend/reactivate + audit" in the PRD but "manage plan/status" in the app flow                                                    | PRD §3; APP-FLOWS H; DB §6                             | Scope creep into billing and tenant data browsing                                          | P0 platform scope: list tenant metadata, suspend/reactivate, read platform audit. No plan/billing, no tenant transaction data.                                                                                 |
| **C-11** | `inventory_adjustments` has a header but no line items, while the stock-correction flow implies multiple products                                              | DB §4; APP-FLOWS F                                     | Multi-line corrections cannot be stored                                                    | Add `inventory_adjustment_items` (product, batch, qty delta, reason) in the P0 migration if adjustments are in P0; otherwise defer both.                                                                       |
| **C-12** | Dues derive from ledger entries, but payments cannot be allocated to specific invoices                                                                         | PRD rule 7; DB §5; APP-FLOWS D                         | Partial payment and multi-invoice dues will be implemented incorrectly or inconsistently   | Add `payment_allocations(payment_id, sale_id, amount_minor)` with a constraint that allocations never exceed the payment or the invoice's outstanding amount.                                                  |
| **C-13** | "Bangla and English-ready" vs a language toggle vs "localization readiness"                                                                                    | PRD §3; UI §2; TRD §7                                  | The team will either hardcode strings or over-build four locales                           | P0: Bangla default, English toggle, all strings externalized, `Intl` for dates/numbers, Latin digits for quantities/SKUs, Bengali labels for UI text.                                                          |
| **C-14** | TRD asks for a "safe parameterized" `SET LOCAL` tenant context; PostgreSQL does not accept bind parameters in `SET`                                            | TRD §4                                                 | Naive implementation either string-interpolates (SQL injection) or drops the mechanism     | Use `SELECT set_config('app.tenant_id', $1, true)` inside the request transaction, with the parameter bound normally.                                                                                          |
| **C-15** | "Store-scoped roles" vs two independent membership role columns with no precedence                                                                             | PRD §2; DB §2                                          | Two sources of truth for authorization                                                     | Tenant membership holds the role; store membership only scopes which stores are visible. Explicitly document that a store membership cannot elevate.                                                           |
| **C-16** | P0 promises low-stock alerts; no reorder point exists in the schema                                                                                            | PRD §3; DB §3                                          | The alert cannot be computed per product/store                                             | Add `store_product_settings(tenant_id, store_id, product_id, reorder_level_qty)` or a tenant-wide `reorder_level_qty` on `products`. Recommend per-store table.                                                |

---

## 5. Architectural risks (ranked)

| ID       | Risk                                             | Severity    | Impact                                                                                            | Mitigation                                                                                                                                                |
| -------- | ------------------------------------------------ | ----------- | ------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **R-01** | No real PostgreSQL available for local tests     | **Blocker** | No local Supabase, no Testcontainers, no credible RLS/concurrency tests                           | Provide Docker Desktop, a local PostgreSQL, or a dedicated staging database before RLS work. CI gets a Postgres service container.                        |
| **R-02** | Supabase project in Sydney for Bangladesh users  | **High**    | 100–150 ms extra per round trip on every screen and transaction; worse on 3G                      | Recreate the empty project in Singapore (`ap-southeast-1`) or Mumbai (`ap-south-1`) before any data. Decide with pilot geography.                         |
| **R-03** | API connected as `postgres` bypasses RLS         | **High**    | A single missed `WHERE tenant_id` becomes a cross-tenant leak; `postgres` is verified `BYPASSRLS` | `app_runtime` role, `FORCE RLS`, grants only on required tables, isolation tests in CI.                                                                   |
| **R-04** | Concurrent sale of the last unit                 | **High**    | Negative stock or double sale                                                                     | Lock balance rows in deterministic product order; `INSERT .. ON CONFLICT` for the first sale of a product; explicit concurrency test.                     |
| **R-05** | Locking a non-existent balance row does nothing  | **High**    | Two first-time sales both insert/oversell                                                         | Pre-create balance rows on receipt; for sales, lock the product/store advisory key or use `INSERT .. ON CONFLICT DO NOTHING` then `SELECT .. FOR UPDATE`. |
| **R-06** | Over-return under concurrency                    | **High**    | Refund exceeds sold/paid quantity                                                                 | Lock the sale and its returnable lines inside the return transaction; test two parallel returns.                                                          |
| **R-07** | Credit-limit race                                | **High**    | Two credit sales exceed the customer limit                                                        | Lock the customer row (or a per-customer advisory lock) before computing ledger outstanding.                                                              |
| **R-08** | Dues/ledger drift                                | **High**    | Wrong customer balance, collections applied incorrectly                                           | All dues from posted ledger + allocations; reconciliation view; property tests.                                                                           |
| **R-09** | Idempotency replay semantics undefined           | Medium      | Duplicate stock/payment or client cannot recover from a timeout                                   | Unique `(tenant_id, actor_id, operation, key)`; store response code/body; 409 for in-flight duplicates; document TTL.                                     |
| **R-10** | Migration drift between Drizzle and Supabase CLI | Medium      | Staging/prod schema diverges from code                                                            | Single migration folder, CI `db diff` gate, generated schema check.                                                                                       |
| **R-11** | Tenant-scoped caching in Next.js                 | Medium      | User sees another tenant's cached data after switching                                            | No shared route cache for tenant data; session and fetches keyed by tenant; explicit cache invalidation on switch.                                        |
| **R-12** | Low-end Android performance undefined            | Medium      | "Mobile-first" is asserted but never measured                                                     | Define a device/network baseline and budgets before Phase 4 (see gaps `G-51`). Do not invent numbers now.                                                 |
| **R-13** | Backups/RPO/RTO not defined                      | Medium      | Data loss is unrecoverable in a money product                                                     | Decide plan tier and RPO/RTO before pilot; run a restore drill.                                                                                           |
| **R-14** | Platform admin cross-tenant access               | Medium      | Support workflow becomes a privacy hole                                                           | Separate principal, MFA, audited actions, metadata-only default, no bulk customer/sales browsing.                                                         |
| **R-15** | Regulatory and tax assumptions                   | Medium      | Legal exposure; reports may imply tax compliance                                                  | Keep tax/COGS out of P0 reports; obtain local verification; label reports "gross sales", never "profit".                                                  |

---

## 6. Tenant isolation assessment

**The design is conceptually correct but under-specified at the database boundary.**

Strengths:

- `tenant_id` on every business table, `store_id` where operational.
- Composite `(tenant_id, id)` unique keys and composite FKs.
- RLS as defense-in-depth.
- Explicit warning not to trust client-provided tenant IDs.

Gaps (corrected in §0.2–§0.3):

1. **No runtime role definition.** The owner/superuser bypass is not addressed operationally. `postgres` is verified `BYPASSRLS`; `app_runtime` does not exist yet.
2. **No policy shape, and tenant equality alone is not enough.** Policies must compare `tenant_id` to the transaction context **and** verify that `app.user_id` is an active member of that tenant. A policy that only checks `tenant_id` trusts the GUC, which is not a security boundary. Use the `SECURITY DEFINER` membership predicate in §0.3.
3. **Platform-operator bypass path** is undefined. It must be a separate, audited principal, never a blanket `BYPASSRLS` and never a tenant membership.
4. **Store-level isolation** is not expressed in RLS, and `app.store_id` is forgeable in the same way as `app.tenant_id`. Enforce store scope in the service layer first, mirror it in policies for store-scoped tables, and test both.
5. **No isolation tests defined at SQL level.** Tests must use two tenants, two stores, direct object references by UUID, a user who is a member of A but not B, and a missing-context case. See §0.2 and §0.5.
6. **Default grants are live.** Verified: future `public` tables created by `postgres` grant ALL to `anon`, `authenticated`, and `service_role` until the default ACL is revoked. This must be fixed in the first migration.

Required request pattern (proposed; meaning corrected):

```text
BEGIN;
  SELECT set_config('app.user_id',   $verifiedUserId,   true);
  SELECT set_config('app.tenant_id', $verifiedTenantId, true);
  SELECT set_config('app.store_id',  $verifiedStoreId,  true); -- optional
  -- The API verified membership and store scope in this transaction.
  ... business statements ...
COMMIT;
```

Notes:

- `set_config(..., true)` is transaction-local, so it does not leak across pooled transactions. It does **not** prove identity. Do not describe it as a security control.
- Membership verification must happen inside the same transaction as the business write, or be re-checked there, to avoid a revocation race.
- `app_runtime` must be granted `SELECT/INSERT/UPDATE` only on required tables and **must not own them**. Append-only tables get no `UPDATE`/`DELETE`.
- The API must never connect as `postgres` (`BYPASSRLS` verified).

---

## 7. Permission model assessment

The baseline matrix (SEC §2) is a good start but incomplete for the actual P0 surface.

Required additions:

- `sales.discount.apply` (and any max-discount rule) — missing entirely.
- `customers.credit_limit.manage` and `customers.credit_sale.create` — the matrix only says "collect dues".
- `inventory.adjust` vs `inventory.adjust.approve` — P1 but must be reserved.
- `reports.export` — implied by PRD but absent from the matrix.
- `staff.invite` vs `staff.role.manage` — "manage staff" conflates them.
- `products.manage` — absent.
- `platform.tenant.status` — separate principal.

Precedence rules (proposed, requires approval; revised in §0.5):

1. Platform principal is not a tenant member and cannot use tenant endpoints.
2. Tenant membership grants the role's permission set.
3. `store_memberships` has **no role column**; it restricts the actor to listed stores and can never grant a permission the tenant role lacks.
4. `OWNER` always sees all tenant stores. `MANAGER` sees all stores when no active store rows exist; explicit rows narrow a MANAGER. `SALES`/`STOCK` require at least one explicit active store membership; none means no store access.
5. Revocation takes effect on the next request (no long-lived authorization cache).
6. Every store-scoped endpoint has a test for: allowed store, other store (404), other tenant's store (404), no store membership, and revoked membership. See §0.5.

---

## 8. Data model assessment

### 8.1 Missing entities required for P0 correctness

| Entity                                              | Needed by                         | Why                                                                |
| --------------------------------------------------- | --------------------------------- | ------------------------------------------------------------------ |
| `invitations`                                       | PRD P0 staff invites; APP-FLOWS A | `invited_by` alone cannot model token, expiry, status, or resend   |
| `payment_allocations`                               | PRD rule 7; APP-FLOWS D           | Partial payments and dues across invoices                          |
| `store_product_settings` (reorder level)            | PRD P0 low-stock alerts           | No place to store the threshold                                    |
| `inventory_adjustment_items`                        | APP-FLOWS F                       | Multi-line corrections                                             |
| `product_attribute_schemas` / validation contract   | PRD P0 category-aware attributes  | `metadata_json` alone is unvalidated                               |
| `platform_operators` or a principal flag            | APP-FLOWS H; SEC MFA              | No platform identity model exists                                  |
| `sale_return_approvals` (or an approval column set) | APP-FLOWS E; SEC matrix           | "Approval if required" has no record                               |
| `user_sessions` (optional)                          | SEC session control               | Revocation and device list; Supabase Auth can partially cover this |

### 8.2 Fields that will cause defects if left as written

| Field                                                    | Problem                                                                    | Recommendation                                                                                               |
| -------------------------------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `purchase_items.received_qty`                            | Denormalized against `goods_receipt_items`; will drift                     | Derive from receipt lines via view, or maintain with trigger + reconciliation test                           |
| `products.metadata_json`                                 | Unvalidated, unqueryable by business rules                                 | JSONB validated against a per-category Zod schema; store schema version                                      |
| `inventory_balances.reserved_qty`                        | No writer in P0; availability formula references it                        | Keep at 0 with a documented "reserved for future orders" note, or omit                                       |
| `inventory_balances` uniqueness with nullable `batch_id` | `NULLS NOT DISTINCT` works on PG15+ but complicates composite FKs          | Sentinel batch (`B-03`)                                                                                      |
| `units.precision`                                        | Ambiguous; quantity scale is fixed at 3                                    | Rename/clarify to `quantity_scale` and enforce per unit                                                      |
| `roles.tenant_id NULL`                                   | Breaks the "tenant_id NOT NULL" convention and RLS assumptions             | System roles are code constants seeded with a fixed tenant-less table, or keep nullable with explicit policy |
| `payments.sale_id NULL`                                  | Cannot represent one payment covering several invoices without allocations | Allocation table                                                                                             |
| `products.sku` uniqueness only for ACTIVE                | Reusing a SKU breaks history and barcode mapping                           | Unique per tenant regardless of status                                                                       |
| `sale_no`, `purchase_no`, `return_no`, `receipt_no`      | No generation strategy; concurrent inserts collide                         | Per-store counter row locked in-transaction, or `(store_id, date, sequence)` with a lock                     |
| `product_batches.lot_code`                               | No uniqueness policy at all                                                | Unique `(tenant_id, product_id, lot_code, expiry_date)` where lot code is present                            |
| `products.barcode`                                       | No uniqueness                                                              | Partial unique `(tenant_id, barcode)` where barcode is not null                                              |
| `sale_return_items`                                      | No returnable-quantity constraint                                          | Enforce via locked sum in the service + DB check where possible                                              |

### 8.3 Missing constraints / indexes to add in the first migration

- Partial unique active memberships: `(tenant_id, user_id) WHERE status = 'ACTIVE'`, `(tenant_id, store_id, user_id) WHERE status = 'ACTIVE'`.
- `CHECK (amount_minor >= 0)`, `CHECK (quantity > 0)` on movement/sale/purchase lines.
- `CHECK (on_hand_qty >= 0)` — if the no-negative-stock rule is absolute (recommended).
- Idempotency unique `(tenant_id, actor_id, operation, key)`.
- Ledger indexes: `(tenant_id, customer_id, posted_at)`, `(tenant_id, store_id, posted_at)`.
- Inventory indexes: `(tenant_id, store_id, product_id)`, `(tenant_id, product_id, expiry_date)` for alerts.
- Reporting indexes: `(tenant_id, store_id, posted_at)` on sales/payments/movements.

### 8.4 Immutability and grants

Posted financial documents and stock movements must be **append-only at the database layer**, not only by convention:

- `REVOKE UPDATE, DELETE` on `inventory_movements`, `customer_ledger_entries`, `payments` (posted), `audit_events` from `app_runtime`.
- Sales/purchase headers may only transition status through service functions; no arbitrary column updates.
- Corrections are new rows (`returns`, `adjustments`, `reversals`).

---

## 9. Inventory calculation assessment

**Validated:** movement-first ledger + materialized balance + no negative stock + batch status gating.

**Required definitions before coding:**

1. **Availability**
   `available = on_hand_qty` for sellable batches only. `reserved_qty` is unused in P0. Quarantined/expired/blocked batches are excluded from sellable availability, not subtracted from `on_hand`.
2. **Batch requirement**
   Per product/category flag `batch_tracked`. If false, use the sentinel batch. If true, the sale line must reference a batch and the batch must be sellable.
3. **Balance creation**
   Balances are created on receipt (and for opening stock) so that sale-time locking always finds a row. Never rely on `SELECT ... FOR UPDATE` to serialize a row that does not exist.
4. **Lock ordering**
   Sort locked balance rows by `(store_id, product_id, batch_id)` to prevent deadlocks in multi-line sales.
5. **Adjustments**
   Every correction is a movement with a reason and an approver; no direct balance writes.
6. **Transfers**
   Out of P0. Do not add transfer tables now.
7. **Reconciliation**
   A SQL view must prove `sum(movements.delta_qty) = balances.on_hand_qty` per key; run it in CI and staging.
8. **Expiry**
   Near-expiry threshold is per category (proposed: 90 days, configurable). Expired batches are blocked from sale but remain countable.
9. **Valuation**
   No costing method selected. **Do not compute or display COGS/profit in P0.** Inventory value reports are out of scope.

---

## 10. Financial records assessment

**Validated:** immutable postings, reversal-based corrections, ledger-derived dues, server-calculated totals, idempotency.

**Required definitions:**

1. **Rounding**
   `BIGINT` paisa everywhere. Line total = `round(unit_price_minor × quantity)`; discount per line in paisa; order total = sum of line totals minus order-level discount. One documented rounding mode (proposed `HALF_UP`), applied per line, never on the aggregate only.
2. **Quantity representation**
   `NUMERIC(18,3)` in the database, decimal **string** in JSON contracts. Reject >3 decimal places at the boundary.
3. **Payments**
   Payment amount, method, reference, status, posted time. Methods for P0: `CASH`, `BKASH`, `NAGAD`, `BANK`, `CARD`, `OTHER` (proposed; needs approval).
4. **Allocation**
   A payment may allocate to one or more posted sales. Collection flow allocates oldest-first. Unallocated amount must be zero in P0 (overpayment rejected).
5. **Credit limit**
   `credit_limit_minor` on customer (proposed). Checked inside the sale transaction with the customer row locked. Limit 0 means no credit.
6. **Refunds**
   Return creates a credit or a refund payment, never a negative sale. The refund method must be recorded. The total refund cannot exceed the net paid on the returned quantities.
7. **Idempotency**
   Scope: tenant + actor + operation + key. Store request hash, response status, response body, created time. Retention proposed 30 days (needs approval). Duplicate with a different body → 409.
8. **Expenses**
   Simple posting with category, amount, date, notes. No approval workflow in P0.
9. **Reports**
   Derived from posted rows. No profit/margin labels until costing is approved.

---

## 11. API boundary assessment

**Validated:** `/v1`, JSON, stable error envelope, cursor pagination, OpenAPI, `Idempotency-Key`.

**Required additions:**

- **Single access path** (`B-01`). Browser → Next.js server → API. No direct table access from the browser.
- **Tenant context** as an explicit, validated path or header (`X-Tenant-Id`), never inferred from a cookie alone and never trusted without a membership lookup.
- **Store context** for store-scoped endpoints, validated against store membership.
- **Error envelope** exactly `{ code, message, details, requestId }`, with stable machine codes (e.g. `INSUFFICIENT_STOCK`, `CREDIT_LIMIT_EXCEEDED`, `IDEMPOTENCY_CONFLICT`).
- **Idempotency** on: receive purchase, post sale, post return, post collection, post adjustment, post expense.
- **Authorization** enforced in the service layer, not just route guards; permission matrix tests per endpoint.
- **Cursor pagination** with stable sort `(created_at, id)`; no offset pagination for large tables.
- **Rate limiting** in-process per IP/user for P0 (single instance). Document the limitation; no Redis.
- **OpenAPI** generated from the same Zod schemas used for validation; CI fails if the spec drifts.
- **Request IDs** propagated to logs and error responses.
- **Health endpoints**: `/health/live`, `/health/ready` (DB check).
- **POS compatibility**: the sale command contract must be UI-agnostic and accept a client-generated idempotency key, so a future POS/offline queue can replay safely. Do not build device auth or sync endpoints now.

---

## 12. Frontend, mobile, and responsive assessment

Validated: mobile-first intent, 360px baseline, bottom nav, no fake live data, accessibility target.

Gaps to close before UI work:

- **Performance budgets** are absent. Define device/network baseline first (e.g. a specific low-end Android and a throttled 3G profile), then set LCP/INP/JS-budget targets. Do not invent numbers in the PRD.
- **Localization mechanics** are unspecified: i18n library, string ownership, pluralization, Bengali digit policy, date format, currency display (`৳1,250.00` vs `৳১,২৫০`).
- **Table strategy**: dense desktop tables cannot be reused on mobile; the brief already says cards, but the component contract must enforce it.
- **Offline behaviour**: show connectivity state and disable money-moving actions when offline; never queue authoritative postings in P0.
- **Accessibility specifics**: WCAG 2.2 AA adds focus-not-obscured, minimum target size (24px), dragging alternatives, and accessible authentication. Add these to UI acceptance tests.
- **Contrast verification**: the proposed amber `#B77A23` on warm background is likely below 4.5:1 for body text; validate all token pairs programmatically before use.

---

## 13. Infrastructure findings (Supabase)

| Finding                                                         | Impact                                                                             | Action                                                                                                      |
| --------------------------------------------------------------- | ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Project region is **Sydney**                                    | Added latency for Bangladesh users; cannot be changed in place                     | Decide pilot geography, then recreate the empty project in Singapore/Mumbai (`B-07`)                        |
| Direct DB host resolves **IPv6 only**                           | `psql`/ORM direct connections fail on IPv4-only networks                           | Use the Supavisor pooler connection string everywhere; document both ports (6543 transaction, 5432 session) |
| Pooler reachable from this machine                              | Transaction pooler on 6543 succeeded (verified read-only)                          | Use 6543 for runtime; use 5432 session or direct for DDL/migrations                                         |
| PostgreSQL **17.11**                                            | Verified through the pooler                                                        | Re-verify extensions and cross-schema FK support before F1.2                                                |
| `postgres` has **BYPASSRLS**                                    | Connecting the API as `postgres` silently defeats RLS                              | Create `app_runtime` with `NOBYPASSRLS`; never use `postgres` at runtime                                    |
| Default ACL grants ALL to `anon`/`authenticated`/`service_role` | New `public` tables are exposed to the publishable key until revoked               | Revoke default privileges + per-table grants in the first migration; add a CI grant check                   |
| TLS chain not trusted by Node default                           | `sslmode=require` failed with "self-signed certificate in certificate chain"       | Pin the Supabase CA (`sslrootcert`); never ship `rejectUnauthorized: false`                                 |
| Custom role through Supavisor                                   | Unverified; `CREATE ROLE` privilege is verified                                    | Test on a throwaway/staging project before F1.2 depends on it; record the fallback                          |
| Project tier not verified                                       | Free tier pauses after inactivity and lacks PITR; unacceptable for a money product | Confirm paid plan + backups/PITR before pilot; record RPO/RTO                                               |
| Auth email uses built-in SMTP                                   | Low rate limits; onboarding emails may be throttled                                | Configure custom SMTP before onboarding real users                                                          |
| Phone OTP not configured                                        | PRD's "email/phone verification approach to be selected" is unresolved             | Decide email-first for P0 (`C-01`, `G-03`)                                                                  |
| Data API root requires a secret key                             | Not a defect; new gateway policy                                                   | Ignore the root endpoint; revoke table grants and do not use PostgREST from clients                         |
| Storage not designed                                            | Product images would need bucket RLS and signed URLs                               | Defer images (`C-08`)                                                                                       |
| Migrations via `supabase db push`                               | Works without Docker, but DDL should not go through the transaction pooler         | Use `postgres` via session/direct connection; CI gate: `supabase db diff --linked` must be empty            |

---

## 14. Complexity to remove from P0 (YAGNI)

| Item                                                                              | Reason                                         |
| --------------------------------------------------------------------------------- | ---------------------------------------------- |
| `plans`, `subscriptions` tables                                                   | Billing is P1; no consumer                     |
| `outbox_events`                                                                   | No async worker in P0                          |
| `integration_connections`, `external_record_mappings`, `sync_runs`, `pos_devices` | P2 integrations                                |
| Custom tenant-authored roles                                                      | Fixed roles suffice (`C-02`)                   |
| `reserved_qty` writes / reservation flow                                          | No orders in P0                                |
| Product images                                                                    | Defer unless pilot requires (`C-08`)           |
| Multi-warehouse transfers                                                         | P1                                             |
| Supplier payables ledger                                                          | P1                                             |
| COGS / inventory valuation                                                        | Costing method unresolved; P1                  |
| Offline drafts                                                                    | P1                                             |
| Dark mode                                                                         | After functional light mode                    |
| Turborepo/Nx                                                                      | pnpm workspaces is enough at this size         |
| Redis / queues / Kubernetes                                                       | No proven requirement                          |
| Custom auth service                                                               | Supabase Auth already exists; do not build two |

---

## 15. Proposed Phase 1 architecture

### 15.1 Stack (all decisions proposed, not yet approved)

| Layer       | Choice                                                           | Rationale                                                   | Alternative considered                      |
| ----------- | ---------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------- |
| Monorepo    | pnpm workspaces + TypeScript strict                              | Matches TRD; minimal tooling                                | Turborepo (defer), Nx (too heavy)           |
| SaaS app    | Next.js (App Router), server components, httpOnly session cookie | Mobile-first SaaS, SSR, fast first load                     | Remix, SvelteKit                            |
| API         | Fastify 5 + Zod + generated OpenAPI                              | Explicit, small, typed; matches suggested TRD option        | NestJS (more ceremony)                      |
| ORM / query | Drizzle ORM + SQL migrations in `supabase/migrations`            | SQL-first, transaction-friendly, no second migration engine | Prisma (heavier, separate migration engine) |
| Database    | Supabase PostgreSQL (single DB, shared schema, RLS)              | Managed Postgres, Auth, Storage available                   | Self-hosted Postgres                        |
| Auth        | Supabase Auth (GoTrue), JWT verified by API                      | Already provisioned; avoids two identity systems            | Custom auth in API                          |
| UI          | Tailwind + Radix primitives + design tokens                      | Accessible primitives, no generic template look             | Full component library                      |
| Validation  | Zod shared in `packages/contracts`                               | One source for runtime + types + OpenAPI                    | io-ts, manual                               |
| Tests       | Vitest + Playwright + SQL isolation suite                        | Unit, integration, E2E, RLS                                 | Jest                                        |
| CI          | GitHub Actions                                                   | Standard                                                    | GitLab CI                                   |

### 15.2 Runtime topology

```text
Browser (mobile-first)
   │  HTTPS, httpOnly session cookie
   ▼
apps/saas  (Next.js server)  ── server-side fetch ──►  apps/api (Fastify, /v1)
                                                        │
                                                        │  pg (app_runtime)
                                                        ▼
                                              Supabase PostgreSQL
                                              ├─ RLS (defense in depth)
                                              ├─ Supabase Auth (JWT issuer)
                                              └─ Storage (P1)
```

No browser → database path. No direct PostgREST usage by clients. `packages/database` is imported only by `apps/api`.

### 15.3 Repository structure (only what Phase 1 needs)

```text
krishi-sheba/
├── apps/
│   ├── api/
│   │   └── src/
│   │       ├── modules/{identity,tenancy,stores,audit,health}/
│   │       ├── plugins/{auth,db,errors,request-id}.ts
│   │       └── server.ts
│   └── saas/
│       └── src/app/{(auth)/,(onboarding)/,(app)/}
├── packages/
│   ├── contracts/src/{auth,tenancy,stores,common}/
│   ├── database/
│   │   ├── src/{schema,client,tenant-context}.ts
│   │   └── drizzle.config.ts
│   ├── ui/src/{tokens,components}/
│   └── config/{tsconfig,eslint}/
├── supabase/
│   ├── config.toml
│   └── migrations/
├── tests/{isolation,e2e}/
├── docs/adr/
└── .github/workflows/ci.yml
```

Modules are created by domain, not by technical layer. Phase 1 creates only `identity`, `tenancy`, `stores`, `audit`, `health`.

### 15.4 Request auth flow (Phase 1)

```text
1. User signs in via Supabase Auth (server-side PKCE); httpOnly/Secure/SameSite=Lax cookie set.
2. Next.js middleware calls supabase.auth.getUser(), refreshing the session when needed.
3. SaaS server calls API with Authorization: Bearer <access_token>; X-Tenant-Id is a hint only.
4. API verifies JWT signature via JWKS, plus iss/aud/exp/nbf/sub. Rejects cookies.
5. API loads principal: user, active memberships, store scope, permissions. Never trusts token roles.
6. API validates the hinted tenant/store against the verified set; cross-tenant ids -> 404.
7. API opens a transaction and sets app.user_id / app.tenant_id / app.store_id (transaction-local).
8. Service enforces permission + store scope; DB enforces RLS + membership predicate as a second layer.
9. TOKEN_EXPIRED -> Next.js refreshes once and retries once; otherwise logout.
10. Response includes requestId; errors use the stable envelope.
```

CSRF is enforced at the Next.js cookie layer (Origin/Host check on mutations + SameSite=Lax), not at the Bearer-only API. See §0.4.

### 15.5 What Phase 1 deliberately does not include

Products, suppliers, purchases, inventory, sales, customers, reports, platform ops UI, POS, website, subscriptions, images, offline.

---

## 16. Validation and quality gates

Exact commands are listed in `PHASE-1-EXECUTION-PLAN.md` §6. Phase 1 cannot exit until:

1. Two-tenant isolation tests pass at API and SQL level, including a user who is a member of A but not B, and a missing-context case.
2. RLS membership predicate tests pass; a tenant-equality-only policy is not accepted.
3. Permission **and store-scope** matrix tests pass for every Phase 1 endpoint (S-01…S-12 in §0.5).
4. `app_runtime` is verified non-owner, `NOBYPASSRLS`, `FORCE RLS`, and able to connect through the chosen pooler with a pinned CA.
5. `anon`/`authenticated`/`service_role` have **no** table privileges on tenant tables, including the default ACL.
6. JWT verification, token refresh, one-retry-on-expiry, cookie flags, and Origin/CSRF checks are tested.
7. `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build` pass locally and in CI **on a remote that has a real PostgreSQL service**.
8. `supabase db diff --linked` reports no drift.
9. No secret in the repository; `.env.example` documents every variable; the DB password shared in chat is rotated.
10. A restore drill has been performed or is scheduled with a recorded RPO/RTO.

---

## 17. Approval checklist

Approve or amend each item before new Phase 1 implementation. **Nothing below is approved by default.** The detailed alternatives and trade-offs are in `DECISION-APPROVAL-MATRIX.md`.

**Blocking architecture decisions**

- [ ] `B-01` API-only data access, with default-grant revocation / Data API decision
- [ ] `B-02` base-unit-only P0 quantities
- [ ] `B-03` sentinel batch strategy
- [ ] `B-04` payment allocations + oldest-first + reject overpayment
- [ ] `B-05` Supabase CLI as the only migration authority
- [ ] `B-06` real-PostgreSQL test environment (Docker, local, or staging)
- [ ] `B-07` recreate Supabase project in Singapore/Mumbai
- [ ] `B-08` fixed roles + store membership narrows only; no role on store membership

**Correction-pass decisions**

- [ ] `D-01` RLS is defense-in-depth; API is the authorization authority (§0.2)
- [ ] `D-02` membership predicate + `SECURITY DEFINER` bootstrap function (§0.3)
- [ ] `D-03` revoke default grants and per-table `anon`/`authenticated`/`service_role` access (§0.3)
- [ ] `D-04` auth/session/refresh/CSRF flow and the narrow `service_role` exception (§0.4)
- [ ] `D-05` store-scope precedence and the S-01…S-12 test matrix (§0.5)
- [ ] `D-06` `app_runtime` role, pooling, TLS CA, and the custom-role fallback (§0.6)
- [ ] `D-07` split M6 into M6a/M6b (`IMPLEMENTATION-BACKLOG.md` §3)
- [ ] `D-08` custom SMTP / email deliverability before onboarding
- [ ] `D-09` rotate the DB password shared during setup
- [ ] `D-10` unfreeze F1.1 code only after the above are approved

**Existing product decisions still pending**

- [ ] `C-01` Supabase Auth, owner MFA deferred, platform MFA required
- [ ] `C-03` no plans/subscriptions in P0
- [ ] `C-08` images deferred to P1
- [ ] `C-13` Bangla default + English toggle, Latin digits for quantities
- [ ] Platform-admin surface location for P0 (route group vs separate app)
- [ ] Pilot geography and target device/network profile
- [ ] Payment method list for P0
- [ ] Idempotency retention period

**The existing F1.1 code remains frozen until these are resolved. No new production code, migration, or hosted-project change will be made.**
