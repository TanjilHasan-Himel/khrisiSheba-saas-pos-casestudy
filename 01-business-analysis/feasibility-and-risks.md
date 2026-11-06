# 01.5 — Feasibility Analysis & Risk Register
**Document:** Technical, Operational & Financial Feasibility  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Public Portfolio Specification  

---

## 1. Feasibility Assessment

### A. Technical Feasibility: HIGH `[VERIFIED]`
- **Monorepo Architecture:** Next.js 16 (App Router) + Fastify 5 REST API sharing validation contracts (`@krishi/contracts`) via pnpm workspace ensures compile-time type safety across frontend and backend.
- **Database Engine:** PostgreSQL 17 Row-Level Security (RLS) guarantees tenant isolation without fragile application-level `WHERE tenant_id = ...` clauses.
- **Micro-Interactions & Physics:** Custom Catmull-Rom splines and `useSpring` oscillators replace bulky third-party chart libraries, shedding 140KB of bundle weight while delivering sub-pixel curve rendering.

### B. Operational Feasibility: HIGH `[VERIFIED]`
- **Low-Literacy Friendly:** Designed with Tiro Bangla typography, clear color-coded hazard bands (Red/Yellow/Blue/Green), and automated SMS generators that eliminate manual Bengali typing by counter staff.
- **Hardware Agnostic:** Optimized for low-cost Android smartphones and standard desktop Google Chrome browsers commonly found in union parishad bazaars.

### C. Financial Feasibility: HIGH `[INFERRED]`
- **SaaS Unit Economics:** Projected customer acquisition cost (CAC) of **৳185 per farmer** against an annual lifetime value (LTV) of **৳45,200**, yielding a **24.4x LTV:CAC ratio**.
- **Subscription Payback:** A monthly retailer subscription fee of ৳2,000 is recovered by preventing a single mobile court penalty (৳10,000 - ৳50,000) or recovering 2 overdue seasonal debts.

---

## 2. Risk Register & Mitigation Strategy

| Risk ID | Risk Description | Severity | Probability | Mitigation Strategy in Krishi Sheba | Evidence |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **RSK-01** | Price gouging fines due to cashier arithmetic error during counter rush. | CRITICAL | HIGH | POS automatically rejects checkout if unit price exceeds gazetted government MRP. | `fertilizer.test.ts` `[VERIFIED]` |
| **RSK-02** | Cross-tenant data leakage between rival input dealers in the same Upazila. | CRITICAL | LOW | PostgreSQL RLS policies enforce isolation at the database engine level. | `core_tenancy_and_pos.sql` `[VERIFIED]` |
| **RSK-03** | Farmer debt default due to harvest crop failure (flood/drought). | HIGH | MEDIUM | Organizes credit into seasonal cohorts (Boro, Aman); allows structured refinancing and roll-forward agreements. | `CustomerKhataModule.tsx` `[VERIFIED]` |
| **RSK-04** | Disputed short deliveries from buffer godowns or private distributors. | MEDIUM | HIGH | Enforces structured Goods Received Notes (GRN) with shortage calculation and dual signature blocks. | `SuppliersModule.tsx` `[VERIFIED]` |
| **RSK-05** | UI jitter and client layout shift during rapid tab switching on slow networks. | MEDIUM | HIGH | App shell locked to 100dvh, `scrollbar-gutter: stable`, and client skeleton swaps eliminated on instant tab clicks. | `page.tsx` & `globals.css` `[VERIFIED]` |

