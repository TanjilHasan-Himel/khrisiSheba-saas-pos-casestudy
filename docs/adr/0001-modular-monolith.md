# ADR 0001 — Modular monolith, TypeScript, pnpm workspaces

- **Status:** Proposed — pending approval (implementation is frozen)
- **Date:** 2026-11-05
- **Deciders:** Product owner, Lead Architect

> **Revision 2 note:** the workspace and shells exist at commit `4b5c2a5`, but the decision is not
> approved, CI has never run (no remote), and no Phase 1 feature code may be added until
> `DECISION-APPROVAL-MATRIX.md` is signed off.

## Context

Krishi Sheba is a multi-tenant SaaS for small agricultural retailers. The spec pack requires a
SaaS app now, with a dedicated POS and website later, and explicitly forbids unnecessary
microservices. Team size is small.

## Decision

- Single repository, pnpm workspaces.
- `apps/saas` — Next.js (App Router) for the tenant-facing product.
- `apps/api` — Fastify 5 + Zod for all business operations and tenant data access.
- `packages/contracts` — shared Zod schemas, error codes, pagination.
- `packages/config` — environment parsing.
- `packages/database` — PostgreSQL access.
- `packages/ui` — design primitives (added when the first real screen is built).
- TypeScript 5.9 strict; Node 24; ESM.

## Rationale

- One deployable backend keeps transactions, authorization, and audit in one place.
- Fastify + Zod is small, explicit, and generates OpenAPI from the same schemas.
- Next.js gives server-side session handling and a mobile-first app shell.
- No Turborepo/Nx yet: six packages do not justify the extra layer.

## Consequences

- No browser-to-database path; the API is a mandatory hop.
- All money/stock logic lives in the API and SQL, never in the UI.
- Adding POS later means a second client of the same API, not a second backend.

## Alternatives considered

- NestJS: more structure and ceremony than needed.
- Next.js route handlers only: would couple deployment and business logic to the frontend.
- Microservices: unjustified operational cost.
