# Decision Log

**Status:** Phase 0 correction pass (revision 2) — **no decision below is approved.**
Every row is a proposal. Approval requires a dated entry by the named owner in
`DECISION-APPROVAL-MATRIX.md` §6.

| ID          | Decision                                                                  | Date       | Status                      | Notes                                                                  |
| ----------- | ------------------------------------------------------------------------- | ---------- | --------------------------- | ---------------------------------------------------------------------- |
| ADR-0001    | Modular monolith: Next.js SaaS + Fastify API + pnpm workspaces            | 2026-11-05 | Proposed — pending approval | `docs/adr/0001-modular-monolith.md`                                    |
| ADR-0002    | API-only data path, `app_runtime`, forced RLS, membership predicate       | 2026-11-05 | Proposed — pending approval | `docs/adr/0002-tenant-isolation.md`; corrected in revision 2           |
| ADR-0003    | Supabase Auth with Google SSO; server-only session; Bearer-only API | 2026-11-05 | Approved by Owner           | `docs/adr/0003-auth.md`; Google SSO from day 1, phone post-auth        |
| ADR-0004    | Supabase CLI migrations are authoritative                                 | 2026-11-05 | Proposed — pending approval | `docs/adr/0004-migrations.md`; corrected in revision 2                 |
| B-01        | API-only access; revoke default grants / Data API decision                | 2026-11-05 | Proposed — pending approval | See `DECISION-APPROVAL-MATRIX.md`                                      |
| B-02        | One base unit per SKU in P0; no carton/pack conversions                   | 2026-11-05 | Proposed — pending approval | Revisit before Phase 2 exit with pilot invoices                        |
| B-03        | `batch_id NOT NULL` with sentinel batch for non-batch products            | 2026-11-05 | Proposed — pending approval | To be implemented in F1.2b/M5                                          |
| B-04        | `payment_allocations`, oldest-first, reject overpayment                   | 2026-11-05 | Proposed — pending approval | To be implemented in M6b/M7                                            |
| B-05        | Supabase CLI is the only migration authority                              | 2026-11-05 | Proposed — pending approval | See ADR-0004                                                           |
| B-06        | A real PostgreSQL is required; Docker is one option, not a requirement    | 2026-11-05 | Proposed — pending approval | No Docker and no local PostgreSQL on this machine today                |
| B-07        | Supabase region (Sydney) should be Singapore/Mumbai                       | 2026-11-05 | Proposed — pending approval | Destructive; awaits explicit approval to recreate the empty project    |
| B-08        | Fixed roles; tenant membership is the only role source; store rows narrow | 2026-11-05 | Proposed — pending approval | Removes `store_memberships.role_id`                                    |
| D-01 … D-10 | Correction-pass decisions                                                 | 2026-11-05 | Proposed — pending approval | See `DECISION-APPROVAL-MATRIX.md` §2                                   |
| C-17 … C-35 | Contradictions to resolve                                                 | 2026-11-05 | Proposed — pending approval | See `ARCHITECTURE-REVIEW.md` §0.9 and `DECISION-APPROVAL-MATRIX.md` §3 |
| G-47        | No COGS/profit labels in P0                                               | 2026-11-05 | Proposed — pending approval | Costing method unresolved                                              |
| G-51        | Performance budgets set from a measured baseline, not invented            | 2026-11-05 | Proposed — pending approval | Phase 4/M11                                                            |

## What changed in revision 2

Revision 1 marked several rows **Accepted** on the basis of the owner's instruction to start
building. That was wrong: the instruction authorized foundation work, not the technical defaults.
The rows are now **Proposed — pending approval**, and the code that was written under them is
**frozen**.

Corrections reflected above:

- ADR-0002 no longer claims RLS authenticates the caller. It is defense-in-depth; the API derives
  identity and membership, and policies include a membership predicate.
- ADR-0002/the schema no longer include a role on `store_memberships`.
- ADR-0003 specifies the server-only session, refresh path, CSRF location, and the narrow
  `service_role` exception.
- ADR-0004 records that DDL runs through a session/direct connection, not the transaction pooler,
  and that the first migration revokes default grants.
- B-06 no longer says "Docker required".

## How a decision becomes final

1. The owner records **Approved / Amended / Rejected** with a date in
   `DECISION-APPROVAL-MATRIX.md` §6.
2. The status in this file changes in the same edit.
3. If the decision changes a schema or runtime shape, the migration/implementation note is recorded
   before the code is written.
4. Until then, this log is a list of proposals and the implementation remains frozen.
