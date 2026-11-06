# 02.1 — Product Requirements Document (PRD)
**Product:** Krishi Sheba (কৃষি সেবা) — Multi-Tenant Agricultural Retail Operations & Financial Platform  
**Lead Product Architect:** Tanjil Hasan Himel  
**Classification:** Canonical Public Specification  

---

## 1. Product Vision & Value Proposition

Krishi Sheba transforms rural agricultural dealerships across Bangladesh from vulnerable, paper-bound shops into compliant, automated, and bank-fundable retail enterprises.

By embedding Bangladesh statutory regulations directly into the point-of-sale checkout engine and structuring customer credit around biological crop harvest cohorts (Boro, Aman, Aus, Rabi), Krishi Sheba protects retailers from mobile court penalties while accelerating working capital recovery.

---

## 2. MoSCoW Feature Prioritization

```
+------------------------------------------+------------------------------------------+
|                  MUST                    |                  SHOULD                  |
|  - Gazetted MRP price ceiling guard      |  - A4 printable GRN & Challans           |
|  - WHO toxicity safety advisories on POS |  - 1-click Bengali WhatsApp balance SMS  |
|  - SCA seed germination viability block  |  - End-of-day cash drawer balancing      |
|  - Seasonal crop credit cohorts (Khata)  |  - Physics-based damped spring charts    |
|  - Engine-level multi-tenant isolation   |  - Damaged seed & pesticide quarantine   |
+------------------------------------------+------------------------------------------+
|                  COULD                   |                  WON'T                   |
|  - Multi-branch stock transfers          |  - Speculative AI crop yield prediction  |
|  - Direct MFS automated payout webhook   |  - Unregulated P2P social chatter        |
|  - Offline-first draft sync engine       |  - Public e-commerce marketplace         |
+------------------------------------------+------------------------------------------+
```

---

## 3. Core User Journeys

### Journey 1: Morning Counter Rush (Peak Fertilizer & Seed Dispatch)
1. Farmer presents Krishok Card or National ID (NID).
2. Cashier selects subsidized Urea (50kg bag) and certified BRRI Dhan 28 seeds.
3. System verifies price does not exceed ৳27/kg and confirms seed lot germination is above 80%.
4. Cashier records split payment: ৳2,050 cash, ৳1,600 added to Boro 2026 Seasonal Khata.
5. System prints bilingual invoice with Tiro Bangla font and dispatches SMS confirmation.

### Journey 2: Supplier Sourcing & Inbound Delivery Receipt
1. Dealer creates purchase order for 200 bags of fertilizer from BCIC buffer depot.
2. Truck arrives with 180 bags (20 bags damaged/short).
3. Manager enters 180 received bags; system logs 20-bag shortage with supplier challan reference.
4. Generates high-contrast A4 Goods Received Note (GRN) with dual signature authorization blocks.

### Journey 3: Periodic Financial Accounting & Unit Economics
1. Dealer opens Analytics tab to review monthly P&L (Revenue, COGS, OpEx, Net Margin).
2. Analyzes crop season ROI: Boro (28.4%), Aman (25.0%), Rabi (28.0%).
3. Evaluates farmer acquisition cost (CAC: ৳185) against annual lifetime value (LTV: ৳45,200).

