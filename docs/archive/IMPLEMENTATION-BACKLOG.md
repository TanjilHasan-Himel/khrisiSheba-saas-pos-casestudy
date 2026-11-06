# IMPLEMENTATION BACKLOG — Krishi Sheba SaaS (P0)

**Status:** Phase 0 correction pass (revision 2) — **awaiting approval.**
**Date:** 2026-11-05
**Revision:** 2 — splits M6 into M6a/M6b so collections cannot precede posted sales; adds store-scope and RLS trust-boundary tests.
**Companion documents:** `ARCHITECTURE-REVIEW.md`, `SPECIFICATION-GAPS.md`, `PHASE-1-EXECUTION-PLAN.md`, `DECISION-APPROVAL-MATRIX.md`.

This is a dependency-ordered backlog for **SaaS Core (P0)** only. Website, dedicated POS, external POS connectors, billing, and AI are explicitly excluded (`01-PRD.md` §3–4, §7 delivery phases). Sizes are relative (S ≤ 1 day, M 2–3 days, L 4–5 days of focused work) and are **not calendar promises**.

---

## 1. Planning rules

1. One vertical slice at a time. No parallel work on Website/POS.
2. Every milestone ends in a working, tested increment — not scaffolding alone.
3. No milestone starts until its blocking gaps (`G-xx`) are decided.
4. Financial and stock correctness beats UI completeness.
5. Migrations are forward-only; destructive changes require an approved plan.
6. A task is not done because the UI renders; it is done when its tests pass.
7. New dependency or infrastructure requires a written justification in the PR.

---

## 2. Epics

| Epic | Domain              | Modules                                                             |
| ---- | ------------------- | ------------------------------------------------------------------- |
| E1   | Foundation          | repo, CI, config, env validation, request IDs, health               |
| E2   | Database foundation | schema, migrations, RLS, runtime role, isolation harness            |
| E3   | Identity & access   | Supabase Auth, session, users, memberships, roles, permissions      |
| E4   | Tenancy & stores    | tenant onboarding, stores, store membership, settings               |
| E5   | Catalog             | categories, units, products, attribute schemas, reorder levels      |
| E6   | Purchasing          | suppliers, purchase drafts, goods receipts, batches                 |
| E7   | Inventory           | movements, balances, availability, alerts, reconciliation           |
| E8   | Customers & credit  | customers, limits, ledger, collections, allocations                 |
| E9   | Sales               | manual sale, pricing, discounts, payments, numbering                |
| E10  | Corrections         | returns, adjustments, expenses                                      |
| E11  | Reporting           | dashboard, daily summaries, reports, CSV export                     |
| E12  | Platform ops        | platform operators, tenant status, platform audit                   |
| E13  | Hardening           | observability, backups, accessibility, performance, security review |

---

## 3. Milestones

### M0 — Decision lock and environment (blocking)

**Scope:** resolve `B-01`–`B-08`, `C-01`–`C-16`, `C-17`–`C-35`, and all BLOCKING gaps; provide a real PostgreSQL (Docker, local install, or dedicated staging); decide Supabase region; confirm paid tier/backups; rotate the exposed DB password; create `.env.example` policy.

**Dependencies:** product owner decisions; pilot geography.
**Acceptance:** Decision Log populated with dated approvals; Supabase project in the agreed region; a real PostgreSQL reachable for tests (`docker`/`psql`/staging connection succeeds); migrations reset from zero; no BLOCKING gap unresolved for M1–M3.
**Tests:** none (environment verification commands only).
**Exit gate:** no BLOCKING gap unresolved for M1–M3.

---

### M1 — Repository foundation and CI

**Scope:** pnpm workspaces, TypeScript strict, ESLint/Prettier, base tsconfig, `apps/api` skeleton with health endpoints, `apps/saas` skeleton, `packages/contracts`, `packages/config`, GitHub Actions (install, lint, typecheck, test, build), env validation.

**Dependencies:** M0.
**Acceptance:** fresh clone → `pnpm install && pnpm build` succeeds; the CI workflow is **configured and has executed on a remote** (not merely present); `/health/live` and `/health/ready` respond; no secrets committed; `.env.example` documents every variable.
**Tests:** unit test for env schema; API health integration test; CI runs all checks.
**Status note (revision 2):** the workspace/CI portion exists at commit `4b5c2a5` but is **frozen** and CI has **never run** (no remote). M1 is not complete until the workflow runs green on a remote with a real PostgreSQL service.
**Excludes:** any business domain.

---

### M2 — Database foundation, RLS, and isolation harness

**Scope:** first migration (identity/tenancy/stores/audit core), `app_runtime` role, default-ACL revocation, `FORCE ROW LEVEL SECURITY`, membership predicate (`SECURITY DEFINER`), tenant-context transaction helper (`set_config`), composite keys/FKs, audit table, migration workflow, two-tenant test fixture.

**Dependencies:** M1, `B-05`, `B-06`, `D-01`–`D-03`.
**Acceptance:** migrations apply from zero on a real PostgreSQL; `app_runtime` is non-owner with `NOBYPASSRLS`; a user who is a member of A but not B sees 0 rows when `app.tenant_id` is set to B; with no context set, every tenant table returns 0 rows; RLS enabled **and forced** on every tenant table; `anon`/`authenticated`/`service_role` have no tenant-table grants (default ACL included); `supabase db diff --linked` empty; isolation suite runs in CI on a remote.
**Tests:** cross-tenant SELECT/INSERT/UPDATE/DELETE attempts; missing-context and non-member cases; superuser-vs-runtime verification; grant/default-ACL assertions; migration reset from scratch; reconciliation of composite FKs.
**Excludes:** catalog/inventory/sales tables (added with their milestones).
**Correction (revision 2):** "cannot read another tenant's rows even with a forged `app.tenant_id`" is **not** a valid RLS guarantee. RLS enforces the context the API sets; the forged-context protection comes from the API deriving membership and from the membership predicate, not from the GUC being unforgeable.

---

### M3 — Identity, tenancy, onboarding, RBAC

**Scope:** Supabase Auth integration, server-only session with httpOnly/Secure/SameSite=Lax cookie, JWT verification via JWKS, user profile, tenant creation (ONBOARDING → ACTIVE), first store, invitations, memberships, fixed roles/permissions, **store scope**, authorization middleware, tenant/store switching, audit of role changes.

**Dependencies:** M2, `G-01`, `G-03`, `G-04`, `G-06`, `G-07`, `G-08`, `G-09`, `D-04`, `D-05`.
**Acceptance:** owner can register, verify, create tenant/store, invite staff; invited user joins with the correct role; a user in two tenants can switch with no data leakage; deactivated staff loses access on the next request; every protected endpoint has a permission test **and** a store-scope test; `store_memberships` carries no role; cross-tenant store ids return 404.
**Tests:** auth flow integration (sign-in/refresh/logout, one-retry on expiry); cookie flags; CSRF Origin check; invitation expiry/reuse; permission matrix per endpoint; store-scope matrix S-01…S-12; tenant switch leakage (API + cache); revocation + refresh-token revocation; audit rows written.
**Excludes:** product catalog, invitations by SMS, owner MFA.

---

### M4 — Catalog

**Scope:** categories (flat P0), units, products (bilingual, SKU, barcode, batch flag, base unit, sale price, status), category attribute schema validation, store reorder levels, product list/search/filter, mobile product screens.

**Dependencies:** M3, `G-11`–`G-18`, `G-24`, `G-55` (images decision).
**Acceptance:** create/edit/deactivate product; duplicate SKU rejected; barcode unique; Bangla/English names; search works at 360px without overflow; inactive products cannot be sold.
**Tests:** validation, uniqueness, attribute schema, permission (`products.manage`), mobile E2E for create.
**Excludes:** images (unless approved), variants, conversions.

---

### M5 — Suppliers, purchasing, inventory

**Scope:** suppliers; purchase drafts with lines; goods receipt (full/partial); batch creation; movement posting; balance materialization; purchase numbering; receive idempotency; low-stock and expiry queries; reconciliation view.

**Dependencies:** M4, `G-18`–`G-30`, `B-03`.
**Acceptance:** receiving posts stock exactly once; retry with the same idempotency key does not double-post; partial receipt is correct; balance equals movement sum; expired batch cannot be marked sellable; over-receipt rejected.
**Tests:** idempotent receive, partial receive, concurrent receive, balance reconciliation, permission (`purchases.receive`), batch sentinel path.
**Excludes:** purchase returns, payables, transfers, approvals.

---

**Dependency note (revision 2):** customer setup and credit policy (**M6a**) must exist before sales. Collections and invoice allocation (**M6b**) require **posted sales from M7**, so M6b cannot start before M7's posting transaction and idempotency tests pass. This corrects revision 1, where M6 required posted sales that were only built in M7.

---

### M6a — Customers, credit policy, ledger schema

**Scope:** customers, contact/PII fields, credit limits and credit policy, customer status, ledger schema and opening balance, customer due **view definition** (no settlement flow), permissions.

**Dependencies:** M3, `G-36`, `C-34`.
**Acceptance:** create/edit/deactivate customer; duplicate/ambiguous identity handled; credit limit stored with defined `0` vs `NULL` semantics; only OWNER/MANAGER may change limits; opening balance is audited; PII access is permission-controlled.
**Tests:** validation, uniqueness, credit-policy unit tests, permission (`customers.manage`, `customers.credit_limit.manage`), PII access, ledger view definition.
**Excludes:** collection receipts, payment allocations, oldest-first settlement, receipt numbering. **M6a must not claim any M6b test.**

---

### M6b — Collections, allocations, dues settlement

**Scope:** collection receipts, payment allocations against **posted** sales (oldest-first), customer due view, receipt numbering, allocation math and concurrency guards.

**Dependencies:** M6a, **M7 (posted sales are mandatory)**, `G-34`, `G-35`.
**Acceptance:** partial payment produces the correct due; oldest-first allocation; overpayment rejected; concurrent collections cannot double-spend the same due; ledger sum matches invoice outstanding; a collection against a draft/cancelled sale is rejected.
**Tests:** allocation math, concurrent collection, overpayment, permission (`customers.collect_dues`), reconciliation (ledger = invoice outstanding), receipt numbering.
**Excludes:** overpayment credit balance, statements/PDF.

---

### M7 — Manual sales

**Scope:** sale wizard (product search, batch selection, quantity), server pricing and discount rules, payment capture **for the sale being posted**, credit sale, sale posting transaction, stock deduction, numbering, receipt view, timeout-safe retry.

**Dependencies:** M5, **M6a** (customer + credit policy), `G-31`–`G-42`, `B-04`. **Not M6b.**
**Acceptance:** concurrent sale of the last unit → exactly one succeeds; stock never negative; totals server-authoritative; credit sale above the customer's policy limit rejected under a customer lock; retry after timeout does not duplicate; 360px flow completes without horizontal scroll; an immediate payment is allocated to the sale being posted.
**Tests:** concurrency (last unit), idempotency, rounding, discount permissions, credit limit + lock, E2E mobile sale, permission matrix.
**Excludes:** offline posting, POS device auth, multi-currency, tax, collection against pre-existing invoices (that is M6b).

---

### M8 — Returns, adjustments, expenses

**Scope:** sale returns (eligibility, approval, restock/quarantine, refund), inventory adjustments with items and approval, expenses.

**Dependencies:** M7, **M6b**, `G-37`–`G-39`, `G-27`, `G-42`.
**Acceptance:** return quantity cannot exceed net sold; refund cannot exceed net paid; concurrent returns blocked; restock movement is correct; adjustment requires approval and is audited; posted documents never mutate.
**Tests:** over-return concurrency, refund math, quarantine path, adjustment audit, permission tests.
**Excludes:** supplier returns, exchanges.

---

### M9 — Dashboard, reports, exports

**Scope:** dashboard KPIs (sales, dues, low stock, near expiry, pending purchases), daily summary, sales/purchase/stock/due reports, CSV export, definitions page/tooltips.

**Dependencies:** M8, `G-43`–`G-48`.
**Acceptance:** reports reconcile with ledger; attendance to tenant timezone; no profit/margin labels; export permission + audit; empty states honest; filters work on mobile.
**Tests:** report SQL fixtures, timezone boundary, export auth + audit, large-export limit, snapshot reconciliation.
**Excludes:** COGS, valuation, charts without a business question.

---

### M10 — Platform operations

**Scope:** platform operator principal, MFA, tenant metadata list, suspend/reactivate, platform audit view, support access policy.

**Dependencies:** M3, `G-10`, `G-61`, `G-62`.
**Acceptance:** operator cannot read tenant sales/customer data by default; suspend blocks tenant requests immediately; every action audited; MFA enforced.
**Tests:** cross-tenant access denial, suspend enforcement, audit completeness, MFA gate.
**Excludes:** billing/plan management, impersonation.

---

### M11 — Pilot hardening

**Scope:** observability (structured logs, request IDs, health), error tracking decision, backup/restore drill, permission audit, accessibility pass, target-device performance pass, security review, runbooks, staging + pilot tenant.

**Dependencies:** M9, M10, `G-51`, `G-65`, `G-66`, `G-76`, `G-79`.
**Acceptance:** restore drill recorded; WCAG 2.2 AA checks pass on critical flows; performance measured on the agreed device/network; no critical/high security findings open; support runbook exists.
**Tests:** E2E suite on staging, a11y automated + manual keyboard, performance trace, restore rehearsal, dependency audit.
**Exit gate:** §7 of `07-IMPLEMENTATION-PLAN.md` satisfied.

---

## 4. Backlog (dependency-ordered)

| ID     | Epic | Task                                                  | Size | Depends on     | Milestone |
| ------ | ---- | ----------------------------------------------------- | ---- | -------------- | --------- |
| KS-001 | E1   | Initialize pnpm workspace, TS strict, lint/format     | S    | M0             | M1        |
| KS-002 | E1   | `packages/config` (tsconfig, eslint), env validation  | S    | KS-001         | M1        |
| KS-003 | E1   | Fastify skeleton + error envelope + request IDs       | M    | KS-002         | M1        |
| KS-004 | E1   | Next.js skeleton + design tokens + base layout        | M    | KS-002         | M1        |
| KS-005 | E1   | Health endpoints + CI pipeline                        | S    | KS-003         | M1        |
| KS-006 | E2   | Core migration: users/tenants/roles/stores/audit      | L    | KS-005         | M2        |
| KS-007 | E2   | `app_runtime` role, grants, FORCE RLS, policies       | L    | KS-006         | M2        |
| KS-008 | E2   | Tenant-context transaction helper (`set_config`)      | M    | KS-007         | M2        |
| KS-009 | E2   | Isolation test harness (2 tenants, 2 stores)          | M    | KS-008         | M2        |
| KS-010 | E2   | Migration workflow + `db diff` CI gate                | S    | KS-009         | M2        |
| KS-011 | E3   | Supabase Auth wiring, httpOnly session, logout        | M    | KS-009         | M3        |
| KS-012 | E3   | JWT verification plugin + principal loader            | M    | KS-011         | M3        |
| KS-013 | E3   | Permission constants + role seeds + authorize helper  | M    | KS-012         | M3        |
| KS-014 | E3   | Tenant onboarding (draft → active) + first store      | L    | KS-013         | M3        |
| KS-015 | E3   | Invitations (create/accept/expire/resend)             | M    | KS-014         | M3        |
| KS-016 | E3   | Memberships, store scope, staff screen                | L    | KS-015         | M3        |
| KS-017 | E3   | Tenant/store switcher + cache isolation               | M    | KS-016         | M3        |
| KS-018 | E3   | Permission matrix tests for all endpoints             | M    | KS-017         | M3        |
| KS-019 | E4   | Categories (flat) + units + defaults seeding          | M    | KS-014         | M4        |
| KS-020 | E4   | Products CRUD + bilingual fields + status             | L    | KS-019         | M4        |
| KS-021 | E4   | Barcode/SKU uniqueness + search/filter                | M    | KS-020         | M4        |
| KS-022 | E4   | Category attribute schema + Zod validation            | M    | KS-020         | M4        |
| KS-023 | E4   | Store reorder levels + low-stock query                | S    | KS-020         | M4        |
| KS-024 | E4   | Mobile product list/editor screens                    | L    | KS-021         | M4        |
| KS-025 | E5   | Suppliers CRUD                                        | M    | KS-020         | M5        |
| KS-026 | E5   | Purchase draft + lines + numbering                    | L    | KS-025         | M5        |
| KS-027 | E5   | Goods receipt + batches + partial receipt             | L    | KS-026         | M5        |
| KS-028 | E5   | Receive idempotency + state machine guards            | M    | KS-027         | M5        |
| KS-029 | E6   | Movement posting + balance materialization            | L    | KS-027         | M5        |
| KS-030 | E6   | Availability query + batch sellability                | M    | KS-029         | M5        |
| KS-031 | E6   | Reconciliation view + test                            | M    | KS-029         | M5        |
| KS-032 | E6   | Expiry/low-stock alert queries                        | M    | KS-030         | M5        |
| KS-033 | E7   | Customers CRUD + credit policy/limits                 | M    | KS-020         | M6a       |
| KS-034 | E7   | Ledger schema + opening balance + due view            | L    | KS-033         | M6a       |
| KS-036 | E7   | Collections + payment allocations (oldest-first)      | L    | KS-042, KS-034 | M6b       |
| KS-057 | E7   | Receipt numbering + due settlement view               | M    | KS-036         | M6b       |
| KS-037 | E8   | Sale draft/wizard data contract                       | M    | KS-030         | M7        |
| KS-038 | E8   | Sale posting transaction (lock, deduct, post)         | L    | KS-037         | M7        |
| KS-039 | E8   | Pricing, discount rules, rounding                     | M    | KS-038         | M7        |
| KS-035 | E7   | Credit limit check + customer row lock (sale path)    | M    | KS-038         | M7        |
| KS-040 | E8   | Payment capture + credit sale + sale-level allocation | L    | KS-039, KS-035 | M7        |
| KS-041 | E8   | Sale numbering + receipt view                         | M    | KS-038         | M7        |
| KS-042 | E8   | Concurrency + idempotency tests (last unit, retry)    | M    | KS-040         | M7        |
| KS-043 | E9   | Sale returns + approvals + restock/quarantine         | L    | KS-042, KS-036 | M8        |
| KS-044 | E9   | Refund math + ledger reversal                         | M    | KS-043         | M8        |
| KS-045 | E9   | Inventory adjustments (items + approval)              | L    | KS-031         | M8        |
| KS-046 | E9   | Expenses                                              | S    | KS-013         | M8        |
| KS-047 | E10  | Dashboard KPIs + empty states                         | L    | KS-042         | M9        |
| KS-048 | E10  | Daily summary + report queries (timezone)             | L    | KS-047         | M9        |
| KS-049 | E10  | CSV export + permission + audit                       | M    | KS-048         | M9        |
| KS-050 | E11  | Platform operator principal + MFA                     | M    | KS-018         | M10       |
| KS-051 | E11  | Tenant metadata + suspend/reactivate                  | M    | KS-050         | M10       |
| KS-052 | E11  | Platform audit view                                   | S    | KS-051         | M10       |
| KS-053 | E12  | Observability + error tracking                        | M    | KS-049         | M11       |
| KS-054 | E12  | Backup/restore drill + runbooks                       | M    | KS-053         | M11       |
| KS-055 | E12  | Accessibility + device performance pass               | L    | KS-049         | M11       |
| KS-056 | E12  | Security review + dependency audit                    | M    | KS-053         | M11       |

---

## 5. Test matrix (applies to every milestone)

| Layer            | Tool                      | Required for                                                                                    |
| ---------------- | ------------------------- | ----------------------------------------------------------------------------------------------- |
| Unit             | Vitest                    | Money math, rounding, permissions, state machines, validators                                   |
| Integration (DB) | Vitest + real PostgreSQL  | Migrations, RLS, transactions, constraints, idempotency                                         |
| Contract         | Vitest + Zod/OpenAPI      | Request/response schemas, error envelope                                                        |
| Isolation        | SQL + API suite           | Two tenants, two stores, forged IDs, cross-tenant joins                                         |
| RLS trust bound. | SQL                       | No context → 0 rows; member-of-A querying B → 0 rows; default grants revoked                    |
| Permission       | API suite                 | Every protected endpoint × each role                                                            |
| Store scope      | API suite                 | Every store-scoped endpoint × role × store set; cross-tenant 404; no membership 403; revocation |
| Auth/session     | API + Next.js integration | JWT expiry/forgery, refresh rotation, one-retry, cookie flags, Origin/CSRF                      |
| Concurrency      | Integration               | Last-unit sale, receive retry, over-return, collection race                                     |
| Reconciliation   | SQL                       | Movement sum = balance; ledger = invoice outstanding                                            |
| E2E              | Playwright                | Owner critical path at 360px, staff flows                                                       |
| Accessibility    | axe + manual              | Keyboard, focus, labels, contrast, target size                                                  |
| Performance      | Trace on agreed device    | Phase 4/M11 budgets                                                                             |

**No milestone is complete if a required test is skipped.** Test results must be reported as actually executed.

---

## 6. Definition of Done (per task)

- [ ] Acceptance criteria met and demonstrated.
- [ ] Server-side authorization implemented and tested (permission **and** store scope).
- [ ] Tenant isolation verified (where data is touched), including the membership predicate and a missing-context case.
- [ ] Money/stock math covered by unit tests with edge cases.
- [ ] Migration reversible or forward-fix documented.
- [ ] Validation and error envelope consistent.
- [ ] Mobile layout verified at 360px (where UI).
- [ ] No secret, mock response, or placeholder logic in production path.
- [ ] Lint, typecheck, tests, build pass locally and in CI.
- [ ] ADR/decision log updated if a decision was made.
- [ ] Known limitations recorded.

---

## 7. Traceability: PRD P0 → milestone

| PRD P0 requirement                                               | Milestone           |
| ---------------------------------------------------------------- | ------------------- |
| Registration, verification, login/logout/session                 | M3                  |
| Tenant, profile, first store, invites, store roles               | M3                  |
| Product catalog, bilingual names, category, unit, price, barcode | M4                  |
| Category-aware attributes                                        | M4                  |
| Suppliers, purchase drafts, receive, batch/expiry                | M5                  |
| Stock movements, availability, low-stock/near-expiry             | M5                  |
| Customers, credit limits, ledger, collections                    | M6a, M6b            |
| Manual sales, line discounts, allocation, dues, returns          | M7, M6b, M8         |
| Expenses, daily summaries, reports, CSV                          | M8, M9              |
| Bangla/English, mobile-responsive                                | M3–M11 (continuous) |
| Tenant settings, minimal platform ops, audit                     | M3, M10             |

---

## 8. Explicitly out of scope for P0

Website, dedicated POS app, POS device management, external POS connectors, offline posting, subscriptions/billing, plan enforcement, supplier payables, purchase returns, multi-warehouse transfers, product variants, unit conversions, product images (unless approved), COGS/valuation, tax invoicing, customer store credit, owner MFA, dark mode, mobile native apps, AI/forecasting, Redis/queues/microservices.

---

## 9. Backlog risks

| Risk                           | Watch point                                                         |
| ------------------------------ | ------------------------------------------------------------------- |
| M2 RLS work underestimated     | Budget for policy tests, not just policies                          |
| RLS treated as authentication  | API verifies membership; RLS is defense-in-depth only               |
| M6b starts before posted sales | Keep collections gated on M7; customer/credit setup is M6a          |
| M5 concurrency defects         | Write the last-unit/receive-retry tests before UI polish            |
| M7 scope creep toward POS      | Reject device sessions, barcode hardware, offline                   |
| M9 report semantics drift      | Freeze definitions before coding queries                            |
| Local environment              | A real PostgreSQL must exist before M2; otherwise tests are theatre |
| Decisions slipping             | Blocking gaps are gates, not suggestions                            |
