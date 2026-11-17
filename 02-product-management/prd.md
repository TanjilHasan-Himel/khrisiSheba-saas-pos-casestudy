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
|  - Farmer Trust Ratings & Manual Limit   |  - Damaged seed & pesticide quarantine   |
|  - DAE Monthly Return Form-1 Register    |  - Multi-branch stock transfers          |
|  - Agro-Advisory & Weather Alert Center  |  - Direct MFS automated payout webhook   |
|  - Engine-level multi-tenant isolation   |  - Offline-first draft sync engine       |
+------------------------------------------+------------------------------------------+
|                  COULD                   |                  WON'T                   |
|  - Bluetooth 2-inch thermal printer ESC  |  - Speculative AI crop yield prediction  |
|  - Camera barcode buffer scanner         |  - Algorithmic auto credit limit hike    |
|  - Regional cooperative rollup views     |  - Public e-commerce marketplace         |
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

### Journey 3: Farmer Credit Trust Evaluation & Manual Limit Control
1. Farmer Karim Mia requests ৳5,000 in additional fertilizer on credit for Boro season.
2. Cashier views Karim's profile: `নিয়মিত খরিদ্দার (TRUSTED)` badge appears based on 100% past Boro settlement.
3. Cashier notices current debt is approaching his ৳30,000 credit ceiling.
4. Store owner reviews the request, clicks `[✏️ লিমিট]`, and manually adjusts the limit to ৳35,000 after confirming crop acreage. The software prevents automatic increases while empowering dealer judgment.

### Journey 4: Statutory DAE Monthly Return (ছক-১) Submission
1. On the 2nd of the month, dealer navigates to Inventory or Analytics module.
2. Clicks `[🏛️ DAE সরকারি মাসিক রিটার্ন (PDF)]`.
3. System compiles opening stocks, BADC/BCIC buffer receipts, total sales, closing stocks, and gazetted values across Urea, TSP, DAP, MoP, and Gypsum into official Form-1.
4. Dealer clicks `[🖨️ A4 প্রিন্ট / PDF সংরক্ষণ]`, signs the statutory declaration, affixes shop seal, and submits it to Upazila Agriculture Office in minutes.

### Journey 5: Emergency Hailstorm & Rice Blast Advisory Broadcast
1. Weather alert indicates 65% chance of severe hailstorm and rainfall across Upazila within 48 hours during Boro harvesting.
2. Dealer clicks `[🌾 আবহাওয়া ও বালাইনাশক এলার্ট]` in Customer Khata or Dashboard.
3. Selects preset template: *"কালবৈশাখী ও শিলাবৃষ্টি সতর্কতা (৮০% পাকা ধান দ্রুত কর্তন)"*.
4. Selects target segment: *"ধান চাষী"* (filters all paddy farmers).
5. Dispatches 1-click WhatsApp messages via interactive dispatch queue and broadcasts SMS to non-smartphone farmers with official shop signature.

### Journey 3: Periodic Financial Accounting & Unit Economics
1. Dealer opens Analytics tab to review monthly P&L (Revenue, COGS, OpEx, Net Margin).
2. Analyzes crop season ROI: Boro (28.4%), Aman (25.0%), Rabi (28.0%).
3. Evaluates farmer acquisition cost (CAC: ৳185) against annual lifetime value (LTV: ৳45,200).

