# ADR 0002 — Tenant isolation strategy

- **Status:** Proposed — pending approval (corrected in revision 2)
- **Date:** 2026-11-05
- **Deciders:** Product owner, Lead Architect, Security

> **Correction:** revision 1 implied RLS prevented a forged tenant context. It does not. Any role
> that can execute SQL can call `set_config('app.tenant_id', ...)`. RLS is **defense-in-depth**; the
> API is the authorization authority. See `ARCHITECTURE-REVIEW.md` §0.2–§0.3.

## Context

The system stores financial and customer data for multiple retailers. A cross-tenant leak is a
business-ending defect. The spec requires RLS plus service authorization, composite foreign keys,
and a runtime role that does not bypass RLS.

## Decision

1. Single PostgreSQL database, shared schema, `tenant_id` on every tenant-owned table.
2. Composite unique `(tenant_id, id)` on parents and composite FKs on children.
3. The API is the only data path. The browser never uses PostgREST or holds a DB-capable key.
4. The API connects as `app_runtime`, a non-owner role without `BYPASSRLS`. `postgres` is verified
   `BYPASSRLS` and is used only for migrations/DDL.
5. RLS is enabled and **forced** on every tenant table.
6. RLS is **defense-in-depth, not authentication.** Policies check tenant equality **and** an active
   membership for `app.user_id`, via a `SECURITY DEFINER` predicate owned by the migration role.
   Membership is resolved before the policy runs, inside the same transaction.
7. The API sets transaction-local context with
   `SELECT set_config('app.tenant_id', $1, true)` (and `app.user_id`, `app.store_id`). These values
   are derived from verified membership, never from a client-supplied field.
8. Membership and store scope are verified inside the same transaction as the write.
9. `store_memberships` carries **no role**. Tenant membership is the only role source; store
   membership only narrows the allowed set.
10. Platform operators are a separate principal and never get a blanket bypass.
11. The first migration revokes default table/sequence/function privileges from `anon`,
    `authenticated`, and `service_role`, and `app_runtime` gets only the per-table grants it needs.

## Rationale

- The database must remain safe even if a service forgets a `WHERE tenant_id` clause.
- A tenant-equality-only policy trusts the GUC, which is not a security boundary.
- `set_config(..., true)` is the only parameterizable, injection-safe way to set transaction-local
  context (a literal `SET LOCAL ... = $1` is not supported by PostgreSQL).

## Consequences

- Every repository function must receive a transaction with context; a single global pool query
  outside a tenant transaction is a defect.
- `tenant_memberships` is not directly readable by `app_runtime`; a narrow
  `list_my_memberships(user)` function is used before a tenant is selected.
- Isolation tests are mandatory in CI: two tenants, two stores, a member of A who is not a member
  of B, and a missing-context case.
- `set_config(..., true)` prevents GUC leakage between pooled transactions, but session state,
  `LISTEN`, session advisory locks, and temporary tables remain unsafe through a transaction pooler.
- A leaked `app_runtime` credential can set the GUCs. That is a **documented residual risk**, not
  something RLS mitigates. Controls are TLS, secret rotation, least privilege, audit, and anomaly
  detection.
- Some admin/migration queries must run as owner via the CLI, never through the API role.

## Verification

`tests/isolation/*` attempts SELECT/INSERT/UPDATE/DELETE across tenants with and without context,
plus a non-member case and a forged tenant header through the API. Grant tests assert that `anon`,
`authenticated`, and `service_role` have no tenant-table privileges.
