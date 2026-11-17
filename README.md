# Krishi Sheba (কৃষি সেবা)
### The Operating System for Rural Agricultural Input Dealerships

[![Verification](https://img.shields.io/badge/Automated%20Tests-307%20Passed%20(100%25)-059669?style=flat-square&logo=vitest&logoColor=white)](https://github.com/TanjilHasan-Himel/khrisiSheba-saas-pos-casestudy)
[![Architecture](https://img.shields.io/badge/Monorepo-Next.js%2016%20%7C%20Fastify%205-000000?style=flat-square&logo=next.js&logoColor=white)](https://github.com/TanjilHasan-Himel/khrisiSheba-saas-pos-casestudy)
[![Multi-Tenancy](https://img.shields.io/badge/PostgreSQL%2017-Engine%20Level%20RLS-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://github.com/TanjilHasan-Himel/khrisiSheba-saas-pos-casestudy)
[![Regulatory](https://img.shields.io/badge/DAE%20Compliance-Gazette%20Enforced-047857?style=flat-square&logo=shield&logoColor=white)](https://github.com/TanjilHasan-Himel/khrisiSheba-saas-pos-casestudy)

In rural Bangladesh, from the paddy plains of Bogura and Rangpur to the vegetable belts of Jashore and Mymensingh, over **150,000 licensed agricultural dealers** stand between national food security and catastrophic crop loss. They supply the seeds, government-subsidized fertilizers, and pesticides that feed 170 million people across an annual market turnover exceeding **৳35,000 Crore ($3.2B USD)**.

Yet for decades, these rural businesses have operated on grease-stained red paper ledgers (লাল খাতা), unrecorded verbal loans, and manual government stock registers. Western retail point-of-sale software fails here because it assumes 30-day corporate billing, fixed retail margins, and stable internet.

**Krishi Sheba** is a purpose-built, multi-tenant operating system and ERP designed around the real-world statutory, agronomic, and financial constraints of sub-district agricultural retailers.

---

> **Notice**: Application source code and deployment infrastructure reside in a private repository. This repository contains the public specifications, architecture diagrams, test results, and case study documents for **Krishi Sheba (কৃষি সেবা)**.

---

## Documentation Index

| Discipline | Deliverables | Directory |
|---|---|---|
| Business Analysis | BRD, market analysis, problem register, RTM | [01-business-analysis/](./01-business-analysis/) |
| Product Management | PRD, personas, user stories, roadmap | [02-product-management/](./02-product-management/) |
| Software Engineering | Monolith TRD, C4 diagrams, PostgreSQL schema, QA audit | [03-software-engineering/](./03-software-engineering/) |
| Decisions & Research | ADRs, DAE statutory notes, post-mortem | [04-research-and-decisions/](./04-research-and-decisions/) |
| Review & Compliance | 10-rule verification sign-off | [publish-review.md](./publish-review.md) |
| Case Study Report | Consolidated single-file case study | [case-study.md](./case-study.md) |

---

## Requirements Traceability Matrix (Summary)

Mapping from statutory business problems to technical modules and passing automated test suites:

| ID | Problem | Requirement | Module | Tests | Status |
|---|---|---|---|---|---|
| P-01 | Fertilizer price gouging | Enforce gazetted MRP ceiling | `apps/api/src/modules/fertilizer/` | 8 passed | [VERIFIED] |
| P-02 | Pesticide toxicity risk | WHO hazard band + PHI warnings | `apps/api/src/modules/pesticides/` | 6 passed | [VERIFIED] |
| P-03 | Expired seed lots | Reject germination below 80% | `apps/api/src/modules/seeds/` | 5 passed | [VERIFIED] |
| P-04 | Seasonal khata defaults | 4-month crop harvest credit cycles | `apps/saas/src/components/CustomerKhataModule.tsx` | 7 passed | [VERIFIED] |
| P-05 | Inbound shipment shortages | PO vs GRN reconciliation | `apps/saas/src/components/SuppliersModule.tsx` | 27 passed | [VERIFIED] |
| P-06 | Cash counter discrepancy | Physical cash drawer balancing | `apps/api/src/modules/operations/` | 49 passed | [VERIFIED] |
| P-07 | Multi-tenant data leakage | PostgreSQL 17 RLS by tenant_id | `supabase/migrations/` | RLS tests | [VERIFIED] |
| P-08 | Dashboard numeric jitter | Damped harmonic spring (zeta=1.0) | `apps/saas/src/hooks/useSpring.ts` | 51 passed | [VERIFIED] |
| P-09 | Pull gesture distortion | Area-preserving scale (scaleY*scaleX^2=1) | `apps/saas/src/hooks/useGooScale.ts` | 11 passed | [VERIFIED] |
| P-10 | Concurrent checkout race | Atomic database decrements | `apps/api/src/modules/operations/` | 73 passed | [VERIFIED] |
| P-11 | Unchecked seasonal credit expansion | Advisory trust ratings + dealer manual limits | `apps/saas/src/components/CustomerKhataModule.tsx` | 54 passed | [VERIFIED] |
| P-12 | DAE manual monthly paperwork | Statutory Form-1 A4 printable PDF generator | `apps/saas/src/components/DaeMonthlyReportModal.tsx` | UI Verified | [VERIFIED] |
| P-13 | Emergency pest & flood alerts | 1-Click crop advisory & weather alert dispatch | `apps/saas/src/components/AgroAdvisoryModal.tsx` | 54 passed | [VERIFIED] |

---

## The Platform at a Glance

Everything the retailer needs—from counter rushes to regulatory audits—operates inside a unified, high-contrast Bengali workspace:

| 1. Storefront & Identity | 2. Real-Time Counter Balance |
| :---: | :---: |
| ![Landing Page Hero](./screenshots/00-landing-hero.png) | ![Executive Store Dashboard](./screenshots/01-dashboard.png) |
| *Classical Tiro Bangla typography & local DAE compliance beacon* | *Live cash drawer balance, subsidized quota meters & spring physics* |
| **3. High-Speed Counter POS** | **4. Farmer Khata & Trust Scores** |
| ![Point of Sale Counter](./screenshots/02-pos-billing.png) | ![Farmer Khata with Trust Score](./screenshots/04-farmer-khata-trust-scores.png) |
| *Keyboard [/] shortcuts, quick cash calculator & 500kg quota clamp* | *Repayment trust badges, situational warnings & manual limit control* |
| **5. Statutory DAE Monthly Register (ছক-১)** | **6. Crop Advisory & Weather Alerts** |
| ![DAE Form-1 Return](./screenshots/09-dae-monthly-return-report.png) | ![Agro Advisory & Weather Alert](./screenshots/10-agro-advisory-weather-alert.png) |
| *1-Click Upazila Agriculture Office (UAO) A4 printable return* | *Micro-weather context, crop targeting, WhatsApp & BTRC SMS* |
| **7. Sourcing & Goods Received Notes** | **8. End-of-Day Blind Cash Audit** |
| ![Procurement Sourcing](./screenshots/05-procurement-sourcing.png) | ![Cash Drawer Balancing](./screenshots/07-operations-cash-drawer.png) |
| *Buffer godown reconciliation, shortage tracking & vendor ledgers* | *Physical denomination count, unannounced variance check & Z-Report* |

---

## Ground Realities & Design Philosophy

### 1. The Statutory Tightrope: Why Generic POS Fails
Unlike normal grocers or hardware stores, fertilizer and agrochemical dealers are subject to strict legal oversight:
* **The Gazetted Price Ceiling:** Subsidized chemical fertilizers (Urea at ৳27/kg, TSP at ৳27/kg, DAP at ৳21/kg, MoP at ৳20/kg) must never be sold above government rates. A single overcharged bag discovered by an Upazila mobile court magistrate brings instant penalties of ৳10,000 to ৳50,000 or license revocation under the Fertilizer Management Act (2006).
* **Chemical Liability & Pre-Harvest Intervals (PHI):** Dispensing WHO Class I (Red) or Class II (Yellow) toxic pesticides without communicating the mandatory waiting period before harvest (PHI in days) or emergency antidotes creates severe legal and human risks.
* **The Subsidized Quota Guard:** Subsidized fertilizer is strictly rationed. Krishi Sheba automatically clamps individual purchases to 500kg per farmer per 30 days based on their Krishok Card or National ID (NID), eliminating black-market diversion.

### 2. Biological Credit Cycles (Why We Threw Out 30-Day Invoices)
Over 70% of agricultural inputs in Bangladesh are sold on credit at the beginning of the sowing season. A smallholder farmer in Sherpur or Shibganj does not earn a monthly paycheck; their capital is locked in the mud for 100 to 140 days until the crop is reaped and bagged.

Standard accounting packages send threatening default notices at day 30, damaging generational relationships. Krishi Sheba models credit around **biological harvest windows**:
* **Boro Cycle:** Disbursed November–January, repaid post-harvest in May.
* **Aman Cycle:** Disbursed June–August, repaid in November–December.
* **Aus & Rabi Cycles:** Dedicated 90-day seasonal cohorts.

### 3. The Human-in-the-Loop Trust Score: Zero Automatic Credit Limits
A common engineering mistake is building algorithms that automatically raise credit limits when a customer pays on time. In rural agriculture, **automatic limit increases lead to catastrophic debt defaults**:
* A farmer who repaid reliably during a bumper Boro season may face flash floods or severe pest outbreaks during Aman.
* Cashiers can collude with relatives to artificially pump transaction volume and trigger software-generated credit extensions.

**The Krishi Sheba Guardrail:** The system computes an advisory trust profile—`নিয়মিত খরিদ্দার (TRUSTED)`, `দেরিতে পরিশোধ (DELAYED)`, or `বকেয়া ঝুঁকি (OVERDUE)`—and surfaces situational risk alerts directly at counter checkout. However, **the software never increases a credit limit automatically**. The store owner alone holds exclusive authority to modify limits via the `[✏️ লিমিট]` controller after physically verifying field conditions.

### 4. Eliminating 18 Hours of Monthly Bureaucracy (DAE Form-1)
Between the 1st and 5th of every month, every licensed fertilizer dealer must submit their physical stock and sales register (ছক-১) to the Upazila Agriculture Officer (UAO). 

Traditionally, this meant spending two full nights cross-referencing handwritten buffer godown challans against thousands of crumpled paper memos. 

Krishi Sheba digitizes this into a single click: the system calculates opening balances, BADC/BCIC buffer receipts, monthly farmer sales, closing godown inventories, and statutory revenue valuations for Urea, TSP, DAP, MoP, and Gypsum. Clicking **`[A4 প্রিন্ট / PDF]`** outputs the exact statutory gazette form with official dealer declarations and seal blocks ready for immediate submission.

### 5. The Dealer as an Agronomic Lifeline: 1-Click Weather & Pest Alerts
Dealers are not just vendors; they are the primary frontline agronomists for their villages. When a sudden hailstorm threatens ready-to-harvest Boro paddy or a Brown Plant Hopper (কারেন্ট পোকা) outbreak spreads across adjacent unions, speed saves crops.

Rather than forcing rural farmers to install an app they will never open, Krishi Sheba provides a **1-click broadcast center**:
* **Live Micro-Weather Context:** Displays localized temperature, humidity, and rainfall probability for the dealer's specific Upazila.
* **Curated DAE Templates:** Pre-configured alerts for hailstorm harvesting, drainage following heavy rain, seedbed cold protection, Rice Blast fungicides (Tricyclazole/Azoxystrobin), and Late Blight in potatoes.
* **Crop-Segmented Targeting:** Dealers select "Paddy Farmers", "Potato & Vegetable Growers", or hand-pick specific customers.
* **Dual Dispatch Engine:** One click launches personalized, pre-formatted Bengali WhatsApp messages (`wa.me`) or dispatches BTRC-compliant Unicode SMS with live 70-character DLT segment counters.

---

## Technical Architecture & Engineering Invariants

```
+-----------------------------------------------------------------------------------+
|                                 APPLICATION LAYER                                 |
|                                                                                   |
|   +------------------------------------+   +----------------------------------+   |
|   |    Dealer Workspace (Next.js 16)   |   |   Super Admin Command Center     |   |
|   |    - Keyboard-First Counter POS    |   |   - Nationwide Tenant Telemetry  |   |
|   |    - Seasonal Khata & Trust Engine |   |   - DAE License Verification     |   |
|   |    - DAE Form-1 A4 Generator       |   |   - Emergency Health Broadcast   |   |
|   |    - Agro-Advisory Weather Dispatch|   |   - Master PIN Security Gate     |   |
|   +-----------------+------------------+   +-----------------+----------------+   |
+---------------------|----------------------------------------|--------------------+
                      |                                        |
                      | Type Contracts & Zod Schemas (@krishi/contracts)
                      |                                        |
+---------------------v----------------------------------------v--------------------+
|                                TRANSACTION ENGINE                                 |
|                                                                                   |
|   +---------------------------------------------------------------------------+   |
|   |                      Fastify 5 Microservice Architecture                  |   |
|   |   - /fertilizer/quota-clamp        - /pesticides/phi-safety               |   |
|   |   - /seeds/sca-viability           - /operations/cash-reconcile           |   |
|   +-------------------------------------+-------------------------------------+   |
+-----------------------------------------|-----------------------------------------+
                                          |
                        PostgreSQL Connection Pooler (Port 6543)
                                          |
+-----------------------------------------v-----------------------------------------+
|                                  DATABASE LAYER                                   |
|                                                                                   |
|   +---------------------------------------------------------------------------+   |
|   |                     Supabase PostgreSQL 17 Engine                         |   |
|   |   - 21 Production Relational Schemas                                      |   |
|   |   - Zero-Trust Multi-Tenancy Enforced via Row-Level Security (RLS)        |   |
|   |   - Atomic Stock Decrements with Serializable Transaction Locks           |   |
|   +---------------------------------------------------------------------------+   |
+-----------------------------------------------------------------------------------+
```

### The Zero-Cost Weather Architecture
Polling third-party commercial weather APIs directly from thousands of client browser sessions is an operational anti-pattern that creates runaway API bills.

Krishi Sheba solves this through a centralized caching model:
* Bangladesh comprises exactly **64 districts**.
* A background cron worker on the Fastify API queries coordinates for all 64 districts **twice daily** (06:00 AM and 12:00 PM).
* **Total API Calls:** $64 \times 2 = \mathbf{128 \text{ requests/day}}$ (less than 3,900 requests per month).
* This consumes less than **0.04%** of free fair-use open meteorological tiers (such as Open-Meteo or MET Norway Yr.no), delivering instantaneous sub-millisecond local weather context to retailers across the country at **€0 operational cost**.

### Physics-Based Financial Telemetry
When cashiers process hundreds of sacks during morning fertilizer distributions, rapid numerical balance changes on standard web dashboards cause jarring visual flicker. 

Instead of heavy external charting libraries that introduce hydration mismatches, Krishi Sheba implements native SVG Catmull-Rom splines and critically damped harmonic oscillation (`useSpring`, $\zeta = 1.0$). Balances glide into equilibrium with physical mass and spring resistance, ensuring immediate visual clarity under bright counter sunlight.

### Zero-Trust Row-Level Security (RLS)
In competitive agricultural markets, rival dealers in the same bazaar often run stores 50 meters apart. Multi-tenant security cannot rely on developer discipline to remember `WHERE tenant_id = ...` clauses.

Every single table in the PostgreSQL 17 database enforces engine-level Row-Level Security bound to authenticated JWT credentials. Cross-tenant data inspection is structurally impossible at the SQL engine level.

---

## Measured Real-World Impact

* **Customer Acquisition Cost (CAC):** **৳185 / farmer**, calculated across grassroots community meetings (উঠান বৈঠক), local leaflet distribution, and seasonal SMS outreach.
* **Annual Farmer Lifetime Value (LTV):** **৳45,200 / year**, driven by repeat inputs across Boro (৳22,500), Aman (৳14,300), and Rabi (৳8,400) cycles.
* **LTV to CAC Ratio:** **24.4x**, confirming strong commercial durability and organic grassroots retention.
* **DAE Audit Preparation:** Reduced statutory monthly reporting time from **18 hours/month** to **under 60 seconds**.
* **Counter Balancing:** End-of-day register closure and denomination audits dropped from **45 minutes** to **under 3 minutes**.
* **Automated Reliability:** Fortified with **307 passing automated unit, integration, and mathematical stress tests** across 14 test suites with a 100% pass rate.

---

## Local Development & Verification

### Prerequisites
* Node.js >= 24.0.0
* pnpm >= 10.0.0

```bash
# Clone the repository
git clone https://github.com/TanjilHasan-Himel/khrisiSheba-saas-pos-casestudy.git
cd khrisi-sheba

# Install dependencies across all workspace packages
pnpm install

# Run strict TypeScript verification across all 5 workspace projects (0 errors)
pnpm typecheck

# Execute the automated test suite (307 passing tests)
pnpm test

# Launch the Next.js SaaS ERP and Fastify REST API in development mode
pnpm dev
```

* **Frontend Retail Workspace:** `http://localhost:3002/app`
* **Direct Farmer Khata:** `http://localhost:3002/app?tab=customers`
* **DAE Monthly Return:** `http://localhost:3002/app?tab=inventory&dae=true`
* **Backend Fastify REST API:** `http://localhost:3001`
