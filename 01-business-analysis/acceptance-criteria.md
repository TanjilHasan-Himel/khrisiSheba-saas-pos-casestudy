# 01.6 — Acceptance Criteria & Quality Gates
**Document:** System Acceptance Criteria  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Public Portfolio Specification  

---

## 1. Domain Quality Gates

### AC-01: BCIC Fertilizer Price Ceiling
- **Given:** A cashier creates a sale invoice line item for subsidized fertilizer (e.g. Urea, TSP, DAP, MoP).
- **When:** The cashier enters a unit sale price higher than the gazetted MRP (e.g., entering ৳28/kg when MRP is ৳27/kg).
- **Then:** The POS engine immediately flags a red validation warning, disables the checkout button, and refuses invoice creation with code `PRICE_CEILING_EXCEEDED`.
- **Status:** `[VERIFIED]` in `apps/api/src/modules/fertilizer/fertilizer.test.ts`.

### AC-02: Agrochemical Toxicity Advice Printing
- **Given:** A customer purchases a WHO Class I or Class II pesticide (e.g. Furadan 5G, Virtako 40WG).
- **When:** The invoice is confirmed and rendered for printing.
- **Then:** The printed receipt must include:
  1. The official WHO Hazard Color Band (Red = Class I, Yellow = Class II).
  2. Pre-Harvest Interval (PHI) clearance date formatted in Bengali.
  3. Restricted Entry Interval (REI) in hours.
  4. First-aid antidote instructions.
- **Status:** `[VERIFIED]` in `apps/api/src/modules/pesticides/pesticides.test.ts`.

### AC-03: Seed Certification Agency (SCA) Viability
- **Given:** A seed product batch with germination rate below 80% or past official validity date.
- **When:** A cashier attempts to add this lot to an active cart.
- **Then:** The system prevents selection, displays a seed quarantine alert, and routes the batch to the damaged goods audit queue.
- **Status:** `[VERIFIED]` in `apps/api/src/modules/seeds/seeds.test.ts`.

### AC-04: Double-Entry Split Payment Ledger
- **Given:** A sale total of ৳3,650 where a farmer pays ৳2,050 in cash and requests ৳1,600 on seasonal credit.
- **When:** The transaction is completed.
- **Then:**
  1. Cash drawer balance increments by exactly ৳2,050.
  2. The farmer's seasonal khata ledger increments by exactly ৳1,600.
  3. `Cash Inflow + Due Debt = Invoice Total` (Double-entry balance check passes).
- **Status:** `[VERIFIED]` in `apps/api/src/modules/khata/khata.test.ts`.

### AC-05: Inbound Procurement Shortage Tracking
- **Given:** A purchase order for 200 bags of Urea.
- **When:** The supplier delivery challan arrives with only 180 bags.
- **Then:** The system marks the order as `PARTIALLY_RECEIVED`, records a shortage of 20 bags, and generates an A4 Goods Received Note (GRN) with supplier dispute references.
- **Status:** `[VERIFIED]` in `apps/saas/src/components/SuppliersModule.tsx`.

### AC-06: Mathematical Spring Convergence
- **Given:** Rapid balance updates causing numerical reversals (+1000 to -1000 step change).
- **When:** The harmonic spring simulator (`useSpring`) converges.
- **Then:** The velocity must decay to zero without numerical runaway or coordinate NaN/Infinity.
- **Status:** `[VERIFIED]` in `apps/saas/src/adversarial.test.ts` (51 passing tests).

