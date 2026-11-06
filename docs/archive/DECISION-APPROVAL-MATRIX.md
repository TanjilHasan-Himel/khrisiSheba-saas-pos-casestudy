# DECISION APPROVAL MATRIX — Krishi Sheba SaaS

**Status:** Phase 0 correction pass (revision 2) — **nothing in this document is approved.**
**Date:** 2026-11-05
**Purpose:** one place to approve, amend, or reject every decision that blocks Phase 1 implementation.
**Companion documents:** `ARCHITECTURE-REVIEW.md` (§0 corrections), `SPECIFICATION-GAPS.md` (§8.1), `IMPLEMENTATION-BACKLOG.md`, `PHASE-1-EXECUTION-PLAN.md`.

> **How to read this file:** every row is a **proposal**. The recommended option is what the architect believes is safest, not what the owner has agreed to. A decision becomes binding only when the owner records it in §6 (`Approval record`) and the status changes from **Proposed** to **Approved**, **Amended**, or **Rejected**. Until then, no code, migration, or hosted-project change is authorized.

**Owner approval legend**

- **Owner** — product owner must explicitly approve (money, legal, scope, destructive infrastructure).
- **Owner + Architect** — owner approves the product consequence; architect confirms the technical form.
- **Architect** — technical decision delegated by the owner; still recorded, still not assumed.

---

## 1. Summary matrix — blocking architecture decisions (`B-01`–`B-08`)

| ID       | Decision                                     | Recommended option                                                                                                                                    | Main alternative / trade-off                                                                                                      | Risk if wrong                                                                        | Approval          | Implementation impact                                                                |
| -------- | -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ----------------- | ------------------------------------------------------------------------------------ |
| **B-01** | Who talks to PostgreSQL                      | **API-only.** Browser never holds a DB-capable key. Revoke `anon`/`authenticated`/`service_role` grants and the default ACL, or disable the Data API. | Browser → PostgREST with RLS: less server code, but makes RLS the authenticator and exposes the publishable key to tenant tables. | Cross-tenant exposure; duplicated authorization logic; `apps/api` loses its purpose. | Owner + Architect | Auth shape, RLS policies, transaction boundaries, idempotency location, API surface. |
| **B-02** | Quantity model in P0                         | **Base units only.** Store purchase/sale quantities in base units; no unit conversion in P0.                                                          | Multi-unit conversion (box ↔ kg): better UX for some products, but conversion errors corrupt stock.                               | Silent stock and money errors; reconciliation never balances.                        | Owner             | Product/stock schema, sale and purchase line math, UI labels.                        |
| **B-03** | Batch tracking when a product has no batches | **Sentinel batch** per store/product for non-batch stock, with expiry `NULL`.                                                                         | Nullable batch everywhere: simpler writes, but availability and ledger queries need two code paths.                               | Stock cannot be reconciled; expiry queries drift.                                    | Owner + Architect | Inventory schema, availability query, stock movement posting.                        |
| **B-04** | Customer payment allocation                  | **Explicit allocation rows**, **oldest-first default**, **reject overpayment** in P0.                                                                 | Collector chooses invoices: more control, more UI and training; allow overpayment credit: needs a credit ledger (P1).             | Wrong dues, double-spend, or an unapproved credit liability.                         | Owner             | Collections module, ledger, receipt numbering, customer due view.                    |
| **B-05** | Migration authority                          | **Supabase CLI** (`supabase/migrations` + `db push`) is the only migration path; no dashboard DDL, no ORM auto-push.                                  | Drizzle Kit push: faster iteration, but drift and no reviewable history.                                                          | Schema drift; an unreviewed production change.                                       | Architect         | CI, migration files, `db diff` gate, rollback procedure.                             |
| **B-06** | Real database for tests                      | **Any real PostgreSQL** (Docker, local install, or dedicated staging). Docker is convenient, not mandatory.                                           | Docker only: simplest parity, but blocks work on machines without virtualization.                                                 | RLS/concurrency tests become theatre; isolation is unproven.                         | Architect         | Local setup, CI service container, F1.2a proof.                                      |
| **B-07** | Supabase region                              | **Recreate the empty project in Singapore or Mumbai** before any data exists.                                                                         | Keep Sydney: no migration cost now, permanent latency for Bangladesh users.                                                       | Latency and a later destructive migration.                                           | Owner             | Project URL/keys, connection strings, secrets, CI config.                            |
| **B-08** | Role and store-scope model                   | **Fixed roles** (`OWNER`, `MANAGER`, `SALES`, `STOCK`); role only on **tenant** membership; `store_memberships` has **no role** and only narrows.     | Per-store roles: flexible for chains, but two sources of truth and complex precedence.                                            | Privilege escalation or a user seeing another store's data.                          | Owner + Architect | Membership schema, authorization service, permission and store-scope tests.          |

---

## 2. Summary matrix — correction-pass decisions (`D-01`–`D-10`)

| ID       | Decision                                          | Recommended option                                                                                                                                                               | Main alternative / trade-off                                                                                         | Risk if wrong                                                                                  | Approval          | Implementation impact                                      |
| -------- | ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ----------------- | ---------------------------------------------------------- |
| **D-01** | RLS trust boundary                                | **RLS is defense-in-depth, not authentication.** The API derives user/tenant/store from verified membership. RLS blocks missing-predicate and cross-tenant statements.           | Treat RLS as the authenticator: fewer API checks, but a forged GUC or leaked runtime credential defeats it silently. | A false sense of security; cross-tenant reads after one API mistake.                           | Owner + Architect | Policy wording, acceptance criteria, tests PT-04/PT-16.    |
| **D-02** | Membership predicate and bootstrap read           | Add `is_tenant_member(user, tenant)` to every policy; add `list_my_memberships(user)` as a `SECURITY DEFINER` function owned by the migration role.                              | Tenant-equality-only policies: faster, but trust the GUC; direct membership SELECT: recursion and over-grant.        | Non-members can read a tenant if they can set the GUC; or membership loading deadlocks itself. | Architect         | RLS SQL, function grants, `app_runtime` privileges.        |
| **D-03** | Default grants / Data API                         | **Revoke default table/sequence/function privileges** from `anon`, `authenticated`, `service_role` in the first migration; add a CI grant check. Disable the Data API if unused. | Leave the Data API on for debugging: convenient, but every new table is exposed until revoked.                       | The publishable key reaches tenant tables before RLS is considered.                            | Owner + Architect | First migration, CI checks, Supabase settings.             |
| **D-04** | Auth, session, refresh, CSRF, `service_role`      | Server-only Supabase client; httpOnly/Secure/SameSite=Lax cookie; `getUser()` refresh; Bearer-only API; Origin checks; `service_role` only in a narrow server-side admin module. | Browser Supabase client: simpler realtime later, but JS-readable session and weaker CSRF posture.                    | Session theft, CSRF, or a leaked admin key; refresh loops.                                     | Owner + Architect | `apps/saas` auth, middleware, API JWT plugin, env/secrets. |
| **D-05** | Store-scope precedence and tests                  | OWNER all stores; MANAGER all unless narrowed by rows; SALES/STOCK require explicit rows; cross-tenant store id → 404; S-01…S-12 tests.                                          | MANAGER also requires rows: stricter, more admin work; 403 instead of 404: leaks existence.                          | Store-scoped staff see other stores or tenant-wide totals.                                     | Owner + Architect | Authorization service, list/report queries, test matrix.   |
| **D-06** | Runtime role, pooling, TLS                        | `app_runtime` (`LOGIN`, `NOBYPASSRLS`, non-owner); transaction pooler 6543 for runtime; session/direct 5432 for DDL; pinned Supabase CA.                                         | Connect as `postgres`: zero setup, but `BYPASSRLS` silently defeats RLS; custom role may be rejected by Supavisor.   | Isolation is cosmetic; or the API cannot connect in production.                                | Architect         | Role creation, grants, connection strings, TLS config.     |
| **D-07** | Milestone order for customers, sales, collections | Split **M6a** (customers + credit policy) before M7; **M6b** (collections + allocations) after posted sales in M7.                                                               | Keep one M6: simpler plan, but collections cannot be tested without invoices.                                        | Collections are built against draft sales; allocation logic is rewritten.                      | Owner             | Backlog order, dependencies, test fixtures.                |
| **D-08** | Transactional email / verification                | Configure **custom SMTP** with SPF/DKIM and Bangla templates before onboarding real users; built-in SMTP is staging-only.                                                        | Built-in SMTP: no setup, but low limits and poor deliverability.                                                     | Users cannot verify or reset; onboarding stalls at launch.                                     | Owner             | Auth settings, email templates, launch checklist.          |
| **D-09** | Exposed database password                         | **Rotate** the password shared during setup before any deployment; store it only in a secret manager.                                                                            | Keep it: no immediate work, but it is already outside the secret boundary.                                           | Unauthorized database access with a `BYPASSRLS` role.                                          | Owner             | Supabase settings, `.env`, deployment secrets, runbook.    |
| **D-10** | F1.1 code freeze                                  | Keep the existing shell **frozen**; fix only the documented defects after the decisions above are approved.                                                                      | Continue building on it: faster, but cements unapproved defaults and known defects.                                  | Rework on top of wrong assumptions; a false "Phase 1 in progress" claim.                       | Owner             | Commit policy, README status, re-acceptance checklist.     |

---

## 3. Correction-pass contradictions that need a resolution (`C-17`–`C-35`)

These are not new features; they are places where two documents or two claims disagree. Each one needs a one-line decision.

| ID       | Contradiction                                                               | Recommended resolution                                                                     | Approval          |
| -------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ----------------- |
| **C-17** | API enables `credentials: true` CORS while the browser must not call it     | Remove browser CORS/credentials when unfrozen; API is server-to-server only.               | Architect         |
| **C-18** | "Never use `service_role`" vs revoke refresh tokens on deactivation         | Approve the narrow server-side admin exception (D-04).                                     | Owner + Architect |
| **C-19** | httpOnly cookie vs a browser Supabase client that needs JS-readable cookies | Commit to a server-only Supabase client; no browser client reads the session.              | Architect         |
| **C-20** | `store_memberships.role_id` vs "store scope only narrows"                   | Remove the role column (B-08, D-05).                                                       | Architect         |
| **C-21** | RLS forced on memberships vs reading memberships before a tenant is chosen  | Approve `list_my_memberships` bootstrap function (D-02).                                   | Architect         |
| **C-22** | M6 includes collections, but posted sales exist only in M7                  | Approve M6a/M6b split (D-07).                                                              | Owner             |
| **C-23** | "Docker required" vs "hosted DB may be used"                                | Any real PostgreSQL is acceptable; choose per-environment (B-06).                          | Architect         |
| **C-24** | Exit gate says "isolation tests in CI", but CI has no DB and no remote      | Add a remote + PostgreSQL service before claiming the gate.                                | Architect         |
| **C-25** | `app_runtime` via pooler vs Supavisor custom-role support unverified        | Verify on a throwaway/staging project in F1.2a; record the fallback.                       | Architect         |
| **C-26** | "No production code written" vs commit `4b5c2a5`                            | Correct the record; code is frozen (D-10).                                                 | Owner             |
| **C-27** | Email verification required vs built-in SMTP limits                         | Custom SMTP before onboarding (D-08).                                                      | Owner             |
| **C-28** | "Platform MFA required in P0" vs platform ops scheduled at M10              | Keep M10 before pilot; do not call P0 complete without it.                                 | Owner             |
| **C-29** | "Store scope in RLS where feasible" vs `app.store_id` is forgeable          | Enforce in the service layer first; mirror in RLS; do not call RLS the source of truth.    | Architect         |
| **C-30** | `public.users.id → auth.users.id` vs storing profile email/phone            | Treat `public.users` as a profile keyed by `auth_user_id`; verify cross-schema FK support. | Architect         |
| **C-31** | `roles.tenant_id NULL` vs the `tenant_id NOT NULL` convention               | System roles as code constants or a tenant-less reference table with an explicit policy.   | Architect         |
| **C-32** | `platform_operators` in `public` beside tenant tables                       | Separate `platform` schema, or an explicit RLS/grant exclusion.                            | Architect         |
| **C-33** | "Oldest-first" allocation is a business rule, not a derived truth           | Approve it as the P0 default (B-04); otherwise let the collector choose.                   | Owner             |
| **C-34** | Credit limit `0 = no credit` vs `NULL = unlimited` is undefined             | Define both before the customers migration; state the default.                             | Owner             |
| **C-35** | "Overpayment rejected" vs real-world advance payments                       | Reject in P0; add a customer credit ledger in P1 if advances are needed.                   | Owner             |

---

## 4. Product and scope decisions still pending (`C`/`G`)

These do not all block Phase 1 code, but they block the milestone named beside them. They are listed so the owner does not have to hunt through the gap register.

| ID / area       | Decision needed                                                 | Recommended default                                          | Blocks             | Approval  |
| --------------- | --------------------------------------------------------------- | ------------------------------------------------------------ | ------------------ | --------- |
| `C-01` / `G-03` | Supabase Auth; owner MFA deferred; platform TOTP in P0          | Supabase Auth, email-first, platform MFA                     | M3                 | Owner     |
| `C-03`          | Plans/subscriptions in P0?                                      | No — billing is P1                                           | M3                 | Owner     |
| `C-08`          | Product images in P0?                                           | No — defer to P1                                             | M4                 | Owner     |
| `C-13`          | Language and digits                                             | Bangla default + English toggle; Latin digits for quantities | M4/M7              | Owner     |
| `G-06` / `G-07` | Fixed roles and store-scope precedence                          | Fixed roles; store membership narrows only                   | M3                 | Owner     |
| `G-08`          | Who approves returns, discounts, credit limits                  | Role rules (OWNER/MANAGER)                                   | M6a/M8             | Owner     |
| `G-31`–`G-42`   | Sales rules: pricing, discounts, credit sale, rounding, returns | Server-authoritative; documented defaults                    | M7/M8              | Owner     |
| `G-34`–`G-36`   | Customer identity, credit policy, PII access                    | Defined in `SPECIFICATION-GAPS.md`                           | M6a                | Owner     |
| `G-47`          | Costing method / gross profit                                   | Defer; no profit labels in P0                                | P1                 | Owner     |
| `G-51` / `G-58` | Target devices, scanners, printers                              | Pilot input                                                  | M7 onward          | Owner     |
| `G-69` / `B-07` | Region and paid tier/PITR                                       | Singapore/Mumbai; confirm paid plan + backups                | F1.0               | Owner     |
| `G-74`          | External POS vendors                                            | Defer to P2                                                  | —                  | Owner     |
| `G-75`          | Pricing model                                                   | Defer to P1                                                  | —                  | Owner     |
| `G-76` / `G-63` | Legal, licensing, privacy                                       | External verification                                        | Pilot              | Owner     |
| `G-89`          | CI has no remote and never ran                                  | Push the repo; add a PostgreSQL service                      | F1.1/F1.7          | Architect |
| `G-91` / `D-08` | Custom SMTP                                                     | Configure before onboarding                                  | Pilot              | Owner     |
| `G-92` / `D-09` | Rotate the exposed DB password                                  | Rotate before any deployment                                 | F1.0               | Owner     |
| `G-93`          | JWT lifetime / refresh rotation                                 | Access token ≤ 1 h; confirm rotation/reuse detection         | F1.4               | Architect |
| `G-94`          | `auth.users` cross-schema FK support                            | Verify in F1.2b; fall back to `auth_user_id`                 | F1.2b              | Architect |
| `G-95`          | F1.1 defects (CORS, `.env.example`, `clean` script)             | Fix on unfreeze                                              | F1.1 re-acceptance | Architect |

---

## 5. Decisions that are explicitly **not** approved by "start building"

The owner's instruction to start building authorized foundation work. It did **not** approve any of the following, and this document does not treat them as approved:

- RLS as an authenticator.
- Tenant-equality-only policies.
- The default Supabase grants.
- `app_runtime` working through Supavisor.
- Any Sydney-region production deployment.
- `store_memberships.role_id`.
- Any retention period, expiry window, return window, credit-limit semantics, or pricing rule marked "proposed" in `SPECIFICATION-GAPS.md`.
- The existing F1.1 code as "done" or "production-ready".
- Docker as the only acceptable local database.
- `service_role` being unused anywhere.

---

## 6. Approval record (to be completed by the owner)

Use one row per decision. Do not mark a decision approved without a date and a named approver.

| ID   | Status (Proposed / Approved / Amended / Rejected) | Date | Approved by | Amendment / condition | Notes |
| ---- | ------------------------------------------------- | ---- | ----------- | --------------------- | ----- |
| B-01 | Proposed                                          | —    | —           | —                     | —     |
| B-02 | Proposed                                          | —    | —           | —                     | —     |
| B-03 | Proposed                                          | —    | —           | —                     | —     |
| B-04 | Proposed                                          | —    | —           | —                     | —     |
| B-05 | Proposed                                          | —    | —           | —                     | —     |
| B-06 | Proposed                                          | —    | —           | —                     | —     |
| B-07 | Proposed                                          | —    | —           | —                     | —     |
| B-08 | Proposed                                          | —    | —           | —                     | —     |
| D-01 | Proposed                                          | —    | —           | —                     | —     |
| D-02 | Proposed                                          | —    | —           | —                     | —     |
| D-03 | Proposed                                          | —    | —           | —                     | —     |
| D-04 | Approved                                          | 2026-11-05 | Owner       | Google SSO from day 1; phone/WhatsApp post-auth | Supabase Auth Google OAuth provider |
| D-05 | Proposed                                          | —    | —           | —                     | —     |
| D-06 | Proposed                                          | —    | —           | —                     | —     |
| D-07 | Proposed                                          | —    | —           | —                     | —     |
| D-08 | Amended                                           | 2026-11-05 | Owner       | Google SSO removes SMTP blocker for initial onboarding | Custom SMTP deferred to transactional notifications |
| D-09 | Proposed                                          | —    | —           | —                     | —     |
| D-10 | Proposed                                          | —    | —           | —                     | —     |

For `C-17`–`C-35`, record the resolution in the same table using the `C-xx` id.

---

## 7. What happens after approval

1. Approved rows are copied to `docs/DECISION-LOG.md` with date and approver; ADR statuses change from **Proposed — pending approval** to **Accepted** (or **Amended**).
2. `SPECIFICATION-GAPS.md` §12 and `PHASE-1-EXECUTION-PLAN.md` §1 are updated in the same change.
3. Only then does **F1.2a** (the minimal RLS/runtime-role proof) start.
4. Rejected or amended rows are recorded, and the affected milestone stays blocked.

**Until step 1 happens, this project has an approved architecture direction and no approved implementation.**
