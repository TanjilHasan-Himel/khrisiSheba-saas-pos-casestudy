# 02.3 — Product Roadmap & Evolution Strategy
**Document:** Product Roadmap & Milestones  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Canonical Public Portfolio Specification  

---

## 1. Release Timeline & Milestone Status

```mermaid
gantt
    title Krishi Sheba Platform Evolution Roadmap
    dateFormat  YYYY-MM-DD
    section Phase 1 (Completed)
    Foundational Monorepo & Fastify API Shell     :done, 2026-11-01, 2026-11-03
    Core Retail Modules & Supabase Migrations     :done, 2026-11-03, 2026-11-04
    Regulatory Domain (DAE, BCIC, SCA) Validation:done, 2026-11-04, 2026-11-05
    Physics-Based Spring Telemetry & Spline Charts:done, 2026-11-05, 2026-11-06
    section Phase 2 (P1 Near-Term)
    Multi-Branch Warehouse Transfers              :active, 2026-11-15, 2026-12-15
    Direct bKash / Nagad MFS Merchant Webhooks    :2026-12-15, 2027-01-15
    Offline Draft Sync with IndexedDB             :2027-01-15, 2027-02-15
    section Phase 3 (P2 Future)
    DAE Central Regulatory API Direct Connector   :2027-02-15, 2027-04-01
    Automated SME Bank Credit Underwriting Package:2027-04-01, 2027-06-01
```

---

## 2. Milestone Breakdown

### Phase 1: SaaS Core & Regulatory Integrity `[VERIFIED / SHIPPED]`
- Multi-tenant dealership onboarding and DAE license recording.
- BCIC subsidized fertilizer price ceiling and quota consumption tracking.
- WHO pesticide toxicity classification (Red/Yellow/Blue/Green), PHI, and REI advisories.
- SCA certified seed germination rate verification (80% minimum viability threshold).
- Farmer seasonal khata book structured around biological harvest credit cycles.
- End-of-day cash drawer balancing, denomination counters, and Z-reports.
- 304 automated tests with 100% pass rate.

### Phase 2: Operations Scale & Payment Integration `[PROPOSED]`
- Multi-godown inventory transfers with transit gate passes.
- Direct automated webhooks for bKash and Nagad payment reconciliation.
- Offline-first draft creation using browser IndexedDB for rural areas with intermittent connectivity.

### Phase 3: National Institutional Integration `[PROPOSED]`
- Direct government API gateway integration with DAE and MoA quota databases.
- One-click verified financial statement export for SME bank loan applications.

