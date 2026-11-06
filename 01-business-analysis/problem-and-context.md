# 01.1 — Business Problem & Market Context
**Document:** Problem Statement & Domain Context  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Domain:** Bangladesh Agricultural Retail & Input Dealerships  
**Classification:** Public Portfolio Specification  

---

## 1. Macro-Economic Context

`[DOCUMENTED]` Bangladesh’s agricultural input retail sector encompasses over **150,000 licensed and informal dealers** distributed across 495 upazilas, serving approximately **16.5 million farming families**. The cumulative annual market turnover across chemical fertilizers, hybrid/certified seeds, and crop protection chemicals exceeds **৳35,000 Crore ($3.2B USD)**.

Food security across Bangladesh depends on the unhindered distribution of four primary subsidized fertilizers:
- **Urea** (Procured from BCIC buffer godowns; Gazetted MRP: ৳27/kg)
- **Triple Superphosphate (TSP)** (Gazetted MRP: ৳27/kg)
- **Di-Ammonium Phosphate (DAP)** (Gazetted MRP: ৳21/kg)
- **Muriate of Potash (MoP)** (Gazetted MRP: ৳20/kg)

Despite the sector's immense national importance, over 90% of rural input retailers operate using traditional paper ledgers (লাল খাতা - Red Khata), informal dealer credit networks (মহাজন বাকি), and unverified physical stock logs.

---

## 2. Core Business Problems

### Problem 1: Seasonal Working Capital Illiquidity & High Bad-Debt Rates
- `[DOCUMENTED]` In Bangladesh's agricultural ecosystem, 60% to 75% of retail input transactions occur on credit during planting seasons (Boro, Aman, Aus, Rabi).
- `[INFERRED]` Traditional commercial ERP and POS software operates under urban retail assumptions (15 to 30-day billing cycles). Applying 30-day payment demands to rural farmers causes widespread operational friction and misclassifies creditworthy farmers as chronic defaulters.
- `[DOCUMENTED]` In reality, farmers liquidate debts only after crop harvest (120 to 150 days later). Without harvest-aligned aging schedules, dealers face a **14% to 22% delinquency rate**, causing severe liquidity crises during input restocking periods.

### Problem 2: Statutory Non-Compliance & Regulatory Vulnerability
- `[DOCUMENTED]` Under the **Fertilizer Management Act (2006)** and **Pesticide Act (2018)**, dealers face strict liability, mobile court fines (ভ্রাম্যমাণ আদালত), and dealership license revocation if:
  1. Fertilizer prices exceed gazetted government MRP ceilings.
  2. Subsidized quotas allocated to a union parishad are diverted to black markets.
  3. Hazardous pesticides (WHO Hazard Classes I and II) are dispensed without communicating Pre-Harvest Intervals (PHI) and mandatory personal protective equipment (PPE) requirements.
- `[VERIFIED]` Manual paper systems provide zero automated safeguards at the checkout counter, exposing dealers to catastrophic legal penalties.

### Problem 3: Inbound Procurement Mismatches & Broken Delivery Audits
- `[DOCUMENTED]` Input dealers purchase bulk cargo from remote buffer godowns (BCIC) and multinational distributors (Syngenta, Bayer, Auto Crop Care, BADC).
- `[INFERRED]` Due to rural riverine transit and road bottlenecks, shipments frequently arrive with torn bags, partial fulfillments, or missing challans.
- `[VERIFIED]` Lack of structured Goods Received Notes (GRN) leaves dealers unable to substantiate shortage claims against suppliers, resulting in direct inventory loss.

### Problem 4: Spoilage of Regulated Seed Germination Lots
- `[DOCUMENTED]` Under the **Seed Act (2018)**, certified seed varieties lose official germination clearance after their Seed Certification Agency (SCA) validity window expires.
- `[VERIFIED]` Selling sub-standard seeds (<80% germination rate) results in severe crop yield failures for farmers and criminal liability for the retailer.

