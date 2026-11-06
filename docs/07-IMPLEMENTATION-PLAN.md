# 07 — Implementation Plan and Delivery Gates

## Delivery approach

Build **one complete vertical slice at a time**; no parallel implementation of Website/POS/AI. Estimates below are sequencing, **not promised calendar durations**; actual time depends on developer capability, feedback, and tests.

### Phase 0 — Discovery and decision lock

Interview at least a few shop operators when access becomes possible; collect anonymized sample invoices/stock sheets; confirm units, credit, batch/expiry, hardware, internet, tax, legal and pricing assumptions. Produce decision log, approved PRD and wireframes.
**Gate:** no unresolved P0 business rule affecting money or stock.

### Phase 1 — Foundation

Monorepo, environments, CI, PostgreSQL, migrations, user identity, tenant creation, membership, stores, RBAC, RLS, audit, request IDs.
**Gate:** two-tenant isolation tests and role tests pass.

### Phase 2 — Catalog and stock

Products/categories/units, supplier master, purchase drafts, goods receipt, batch stock, movement ledger, low-stock/expiry alerts.
**Gate:** receive/retry/partial receipt/stock reconciliation tests pass.

### Phase 3 — Manual sales and dues

Sales command, price validation, inventory deduction, payments, credit ledger, collections, returns, invoice export.
**Gate:** concurrent last-item sale, idempotency, partial payment, return/reversal tests pass.

### Phase 4 — Mobile dashboard and reports

Daily KPIs, stock/customer/purchase/sale search, exports, 360px mobile workflows, Bangla content, accessibility testing.
**Gate:** owner can perform critical tasks end-to-end on target Android device.

### Phase 5 — Pilot hardening

Staging + pilot tenant, permission audit, backups/restore, support procedures, observability, error budgets, usability fixes, data migration plan.
**Gate:** reconciled trial transactions and explicit go-live signoff.

### Phase 6 — Marketing site + commercialization

Public pages, transparent pricing, signup/trial, privacy/terms, onboarding/support, subscription payment provider if chosen. No fake claims.
**Gate:** trial onboarding and plan enforcement tested.

### Phase 7 — Dedicated POS

Separate responsive PWA frontend, store/device registration, cashier sessions, barcode input, cart, payment, receipt, returns; reuse sale command API.
**Gate:** same financial and stock correctness as manual sale.

### Phase 8 — External POS integration

Provider discovery → CSV importer → field mapping → dry-run → reconciliation → optional API/webhook connector with provider-specific authorization.
**Gate:** duplicates, out-of-order events and mapping failures do not corrupt stock.

## Example initial task backlog

- Create workspace and shared TS config.
- Define permissions, roles, membership policy tests.
- Write migrations for tenants/stores/products.
- Add RLS policies and cross-tenant integration tests.
- Implement product creation/list API + mobile UI.
- Implement goods-receipt transaction with idempotency.
- Implement stock ledger and reconciliation query.
- Implement manual sale transaction with concurrency test.
- Implement customer due ledger and collection.
- Build end-to-end mobile owner workflow.

## Release gates

Functional acceptance; tenant isolation; stock/ledger reconciliation; security review; backup restore; usable Bangla/mobile UI; no critical unresolved bugs; regulatory verification; pilot operator signoff.

## Budget and hosting

Do not invent monthly cost. Collect expected tenant count, stores, daily invoices, images, retention, SMS, bandwidth, backup RPO/RTO, provider choices and required support hours; then calculate fixed + variable costs and pricing break-even.
