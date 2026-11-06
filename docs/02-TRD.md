# 02 — Technical Requirements Document (TRD)

## 1. Architecture decision

**Modular monolith**: Next.js/TypeScript SaaS frontend, dedicated Node.js TypeScript API (NestJS or Fastify; choose one before implementation), PostgreSQL, Prisma or Drizzle (choose one), object storage for images and documents. Monorepo managed with pnpm workspaces. Separate deployables for `apps/saas`, `apps/api`, later `apps/website`, `apps/pos`, `apps/platform-admin`.

Avoid choosing two backend frameworks or two ORMs in the actual implementation. Suggested initial choice: **Fastify + Zod + Drizzle + PostgreSQL**, but validate team familiarity and tooling first.

## 2. Repository

```text
apps/
  saas/               # mobile-first dashboard
  api/                # HTTP API + domain services
  website/            # future
  pos/                # future
  platform-admin/     # future or isolated route
packages/
  contracts/          # Zod schemas, DTOs, typed errors
  database/           # schema and migrations
  ui/                 # design primitives
  config/             # lint, TS, tooling
infra/                # IaC/deployment, backup config
docs/
```

## 3. Domain modules

Identity, Tenancy, Memberships, Stores, Catalog, Suppliers, Purchasing, Inventory, Customers, Sales, Payments, Credit Ledger, Returns, Expenses, Reporting, Audit, Platform Operations. Each owns business services; modules communicate through explicit interfaces, not direct UI database writes.

## 4. Tenant isolation

- Identity is global; tenant membership is many-to-many.
- Every tenant-scoped table has `tenant_id NOT NULL`; store-scoped tables also `store_id`.
- Database RLS policies on tenant-scoped tables; set verified tenant context **per transaction** with `SET LOCAL` / equivalent safe parameterized mechanism, never pooled connection-global state.
- Runtime database role must not bypass RLS; do not use table owner or privileged service role for ordinary requests.
- Composite foreign keys `(tenant_id, parent_id)` prevent cross-tenant references.
- Admin/service jobs use explicit audited context; test cross-tenant joins, exports, background jobs, object-storage paths and cache keys.

## 5. API conventions

Base `/v1`; JSON UTF-8; stable error envelope `{code,message,details,requestId}`; server-generated IDs; cursor pagination; UTC timestamps with tenant-local display; decimal quantity strings for fractional units; money as minor units.

Sample endpoints:

```text
POST   /v1/tenants
GET    /v1/tenants/current
POST   /v1/stores
GET    /v1/products?cursor=&search=
POST   /v1/products
POST   /v1/purchases
POST   /v1/purchases/{id}/receive
GET    /v1/inventory/availability?storeId=&productId=
POST   /v1/sales
POST   /v1/sales/{id}/returns
POST   /v1/customers/{id}/collections
GET    /v1/reports/daily-sales?storeId=&date=
```

For financial POSTs require `Idempotency-Key` scoped to tenant + operation + actor; hash request payload to reject same key/different body.

## 6. Transaction design

- Purchase receive: lock purchase, validate state and batches, create receipt lines, stock movements, inventory balances, audit, commit.
- Sale post: validate membership/store, lock relevant balance rows in deterministic order, enforce available stock and batch status, compute authoritative price/tax/discount, create sale/lines, stock movements, payments/ledger, audit, commit.
- Retry on serialization conflicts only with safe idempotency semantics.
- Do not send external notifications before commit; use transactional outbox for async notifications/integration events.

## 7. Non-functional requirements

- Availability: define pilot SLO after hosting selection; no invented 99.99% guarantee.
- Performance: pilot test at realistic store volumes, p95 latency and concurrent sale scenarios; set numeric targets from device/network research.
- Accessibility: WCAG 2.2 AA as design target, test keyboard navigation, contrast, focus, text scaling.
- Localization: Bangla/English UI strings, Bengali text search, units, BDT, timezone Asia/Dhaka, locale-aware dates.
- Observability: request IDs, structured logs with PII redaction, error tracking, DB metrics, audit trails.
- Backup: automated encrypted backups, restore drills, defined RPO/RTO before go-live.
- Security: TLS, secure cookie/session strategy, CSRF where applicable, rate limits, input validation, upload scanning, secrets manager, dependency scanning.

## 8. Offline/mobile

Responsive web and installable PWA shell may be delivered first. **Do not claim offline financial posting.** Cached read-only catalog and local drafts can be P1 with explicit stale-data labels. Offline POS later requires device-local durable queue, unique transaction IDs, conflict handling, reconciliation, and policies for inventory/payment risk.

## 9. Integration boundaries

Future POS uses public internal sale command contract, never writes DB directly. External POS adapters map external product/store IDs, import transactions with provider-specific IDs, idempotency and reconciliation dashboard. CSV import first; API/webhook only when provider supports it. Per-tenant credentials encrypted and rotated.

## 10. Quality gates

CI: format, lint, TypeScript, unit tests, migration validation, integration tests with real PostgreSQL/RLS, E2E mobile workflows, authorization matrix, concurrency/idempotency tests. Staging data must be synthetic/anonymized.
