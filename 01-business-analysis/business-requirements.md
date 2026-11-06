# 01.4 — Business Requirements Document (BRD)
**Document:** Canonical Business Requirements  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Public Portfolio Specification  

---

## 1. Business Objectives

| ID | Business Objective | Target Metric | Verification Status |
| :--- | :--- | :--- | :--- |
| **BO-01** | Eliminate mobile court fines for price gouging and non-compliant pesticide sales. | 0 fines across registered dealers | `[VERIFIED]` POS enforces MRP ceiling programmatically |
| **BO-02** | Shorten seasonal debt collection cycle from 165 days to 112 days. | 32% increase in working capital velocity | `[INFERRED]` Model-based projection from automated SMS schedules |
| **BO-03** | Prevent inventory write-offs caused by distributing expired certified seeds. | 0 expired seed dispatches | `[VERIFIED]` Germination viability & SCA expiry blocked at POS |
| **BO-04** | Automate daily cash drawer reconciliation and prevent cashier leakage. | End-of-day variance < 0.1% | `[VERIFIED]` Denomination counter audit with Z-report logging |

---

## 2. Functional Requirements (FR)

### FR-01: BCIC Fertilizer Quota & Price Ceiling Control
- **FR-01.1:** The system shall maintain gazetted maximum retail prices (MRP) per kilogram for Urea, TSP, DAP, and MoP.
- **FR-01.2:** The system shall reject any cash memo line item where `unitPrice > gazettedMrpPerKg`.
- **FR-01.3:** The system shall track the monthly store quota allocated by BADC/BCIC and deduct confirmed sales atomically.
- **Evidence:** `apps/api/src/modules/fertilizer/fertilizer.test.ts` (8 passing tests) `[VERIFIED]`.

### FR-02: Agrochemical Toxicity & Safety Mandate
- **FR-02.1:** Every pesticide product shall have a WHO Hazard Classification (Class I Red, Class II Yellow, Class III Blue, Class IV Green).
- **FR-02.2:** Cash memos for Class I and II chemicals shall print Pre-Harvest Intervals (PHI in days), Restricted Entry Intervals (REI in hours), and emergency antidote guidance.
- **Evidence:** `apps/api/src/modules/pesticides/pesticides.test.ts` (6 passing tests) `[VERIFIED]`.

### FR-03: Seed Viability & SCA Lot Validation
- **FR-03.1:** Certified seeds shall include Seed Certification Agency (SCA) lot numbers and germination percentages.
- **FR-03.2:** The system shall prevent sale of seeds with germination rates below 80% or past official validity dates.
- **Evidence:** `apps/api/src/modules/seeds/seeds.test.ts` (5 passing tests) `[VERIFIED]`.

### FR-04: Crop-Aligned Seasonal Khata Book
- **FR-04.1:** Debts shall be organized by biological crop harvest cohorts (Boro, Aman, Aus, Rabi) rather than calendar 30-day buckets.
- **FR-04.2:** The system shall provide 1-click Bengali SMS and WhatsApp reminder generation containing farmer name, exact balance in BDT, and store address.
- **FR-04.3:** The system shall enforce double-entry invariance: `Paid Amount + Outstanding Due = Grand Total`.
- **Evidence:** `apps/api/src/modules/khata/khata.test.ts` (7 passing tests) `[VERIFIED]`.

### FR-05: Inbound Procurement & A4 Goods Receipt Note (GRN)
- **FR-05.1:** Sourcing orders shall track ordered quantity versus received quantity, managing partial fulfillments.
- **FR-05.2:** Printable A4 delivery receipts shall include dealer DAE license numbers, itemized batches, and dual authorization signature blocks.
- **Evidence:** `apps/saas/src/components/SuppliersModule.tsx` & `@media print` CSS engine `[VERIFIED]`.

---

## 3. Non-Functional Requirements (NFR)

| NFR ID | Dimension | Specification | Verification Evidence |
| :--- | :--- | :--- | :--- |
| **NFR-01** | **Multi-Tenancy** | Zero-trust data isolation enforced at the PostgreSQL engine level via Row-Level Security (RLS). | `supabase/migrations/20261105000003_core_tenancy_and_pos.sql` `[VERIFIED]` |
| **NFR-02** | **Numerical Invariance** | Financial calculations must use integer minor units (BDT paisa) with zero floating-point drift. | `packages/contracts/src/contracts.test.ts` `[VERIFIED]` |
| **NFR-03** | **Animation Stability** | Dynamic balance telemetry must use critically damped harmonic oscillation (`zeta = 1.0`) without velocity explosion under +/-1000 step reversals. | `apps/saas/src/adversarial.test.ts` (51 passing tests) `[VERIFIED]` |
| **NFR-04** | **Typography & Legibility** | Authentic Bengali glyph rendering using official Google Font Tiro Bangla across mobile screens and thermal/A4 print receipts. | `apps/saas/src/app/globals.css` `[VERIFIED]` |
| **NFR-05** | **Offline Resilience** | Browser shell must maintain sticky sidebar navigation and zero-latency tab switching without unmounting skeletons. | `apps/saas/src/components/Navigation.tsx` `[VERIFIED]` |

