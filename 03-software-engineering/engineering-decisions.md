# 03.2 — Architecture Decision Records (ADR)
**Document:** Technical Decisions & Engineering Trade-Offs  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Canonical Public Portfolio Specification  

---

## ADR 0001: Modular Monolith vs Microservices

### Status: APPROVED `[VERIFIED]`
### Context:
Agricultural retail software in Bangladesh operates with limited engineering headcount and tight latency constraints during morning counter rushes.
### Decision:
Adopt a **Modular Monolith** using Turborepo workspaces:
- `apps/saas`: Next.js 16 (React 19) frontend.
- `apps/api`: Fastify 5 REST transactional API.
- `packages/contracts`: Shared Zod validation contracts and DTOs.
- `packages/database`: Shared database migrations and types.
### Rationale & Trade-Offs:
- **Pros:** Zero network serialization overhead between modules, guaranteed compile-time type safety across frontend and backend, single repository deployment, zero version skew.
- **Cons:** Shared deployment pipeline; mitigated by independent sub-project build filters (`pnpm --filter @krishi/saas build`).

---

## ADR 0002: PostgreSQL 17 Engine-Level RLS vs App-Level Filtering

### Status: APPROVED `[VERIFIED]`
### Context:
Multi-tenant SaaS platforms frequently suffer catastrophic data leakage when developers omit a `WHERE tenant_id = ...` clause in new API endpoints.
### Decision:
Enforce **PostgreSQL Row-Level Security (RLS)** on all 21 production tables. Application database roles connect with `NOBYPASSRLS`.
### Rationale & Trade-Offs:
- **Pros:** Zero-trust security guarantee. PostgreSQL rejects unauthorized cross-tenant data access at query plan compilation.
- **Cons:** Slight query execution overhead (~1-2ms); completely negligible compared to the commercial liability of exposing rival dealer prices.

---

## ADR 0003: Custom SVG Splines vs Third-Party Charting Libraries

### Status: APPROVED `[VERIFIED]`
### Context:
Off-the-shelf chart libraries (Recharts, Chart.js, Highcharts) introduce heavy runtime bundles (>140KB) and frequently suffer React 19 hydration mismatches.
### Decision:
Implement custom SVG path generators using **Catmull-Rom to Cubic Bezier** conversion algorithms coupled with critically damped harmonic spring hooks (`useSpring`).
### Rationale & Trade-Offs:
- **Pros:** Saved >140KB client bundle weight, zero external charting dependencies, 100% testable mathematical functions, sub-pixel rendering accuracy.
- **Cons:** Required authoring and maintaining spline mathematics and canvas projection code; validated by 26 automated unit and adversarial tests.

---

## ADR 0004: Decoupled Super Admin Route with Master PIN Gate

### Status: APPROVED `[VERIFIED]`
### Context:
Store cashiers sharing a shop tablet must never be able to access nationwide dealer lists, GMV totals, or DAE license verification queues.
### Decision:
Decouple the Super Admin command center into an isolated route (`/superadmin`) protected by a Master Security PIN challenge (`7890`) and separate session token.
### Rationale & Trade-Offs:
- **Pros:** Complete isolation of administrative surface area, clean dealer impersonation banner with 1-click return.
- **Cons:** Requires distinct login flow; highly desirable for security.

