# Krishi Sheba (কৃষি সেবা) — Comprehensive SaaS Benchmarking, Gap Analysis & Agricultural Retail Architecture

**Document Reference:** `DOC-KS-M1-BENCHMARK-2026-11`  
**Classification:** Product Requirements Document (PRD) & Domain Architecture Specification  
**Scope:** Milestone 1 — Requirement R1 (SaaS Benchmarking and Agricultural Feature Matrix)  
**Target Systems:** `apps/saas` (Next.js 16 App Router POS), `apps/api` (Fastify 5 TypeScript Backend), `packages/contracts` (Zod Schemas), `supabase` (PostgreSQL 17)  
**Target User Base:** 15,000+ Registered BCIC/BADC Fertilizer Dealers, Pesticide Retailers, and Seed Merchants across Bangladesh  
**Status:** Authoritative Architectural Standard  

---

## 1. Executive Summary & Domain Mission

### 1.1 The High-Stakes Reality of Bangladeshi Agricultural Input Retail
In Bangladesh, agricultural input retailers (সার, বালাইনাশক ও বীজের ডিলার) are not ordinary FMCG shopkeepers. They are the frontline stewards of national food security, operating across approximately 495 Upazilas and 4,500+ Unions. In a country of 170+ million people sustained by intensive, tri-annual cropping cycles on smallholder land parcels (averaging under 0.6 hectares), an agro-dealer’s shop serves as both a commercial hub and an agronomic triage center.

Agro-dealers handle three highly regulated, mission-critical product categories:
1. **Subsidized Chemical Fertilizers:** Essential inputs (Urea, DAP, TSP, MOP) procured under state monopoly, allocated via rigorous monthly quotas by sub-district administrations, and sold under criminal statute at gazetted Maximum Retail Prices (MRPs).
2. **Crop Protection Agrochemicals:** Highly potent, regulated toxins governed by the Pesticides Act 2018 and monitored by the Department of Agricultural Extension (DAE), requiring strict hazard categorization, application protocols, Pre-Harvest Intervals (PHI), and emergency antidote directives.
3. **Certified Agricultural Seeds:** High-yield varieties (HYV) and hybrid seeds whose viability is subject to statutory laboratory germination and moisture thresholds overseen by the Seed Certification Agency (SCA).

Currently, over 90% of these 15,000+ licensed dealers and independent sub-dealers rely on manual, paper-bound accounting: ink-stained red cloth ledgers (লাল খাতা - *Lal Khata*), fragile paper cash memos, and physical stock registers. This manual approach leads to severe operational vulnerabilities:
- **Regulatory Sanctions:** Dealers risk license cancellation, confiscation of security deposits, and mobile court penalties when physical books fail to reconcile with official Delivery Orders (DOs) during sudden inspections by the Upazila Agriculture Officer (UAO) or Executive Magistrate.
- **Capital Depletion from Unsettled Seasonal Credit:** Because smallholder farmers experience seasonal liquidity (cash only arrives after the paddy or potato harvest), dealers extend millions of Taka in unsecured credit. Without systematic tracking tied to crop seasons (Boro, Aman, Aus, Rabi), unpaid balances compound into toxic bad debts.
- **Commercial Squeeze from Agribusiness Mohajons:** Retailers purchase branded pesticides and seeds from corporate agribusiness distributors (e.g., ACI, Square, Petrochem, Syngenta, Bayer) under strict 30-to-60-day credit lines. Without visibility into inventory turnover and distributor dues, retailers frequently suffer liquidity freezes.

### 1.2 Mission Statement: Krishi Sheba (কৃষি সেবা)
Krishi Sheba is purpose-built as the specialized operating system for rural Bangladeshi agricultural retailers. It bridges the gap between high-velocity counter commerce and stringent regulatory compliance. By codifying government quota formulas, DAE toxicity hazard bands, SCA seed viability rules, and harvest-synchronized credit ledgers (*Khata*) into an ultra-fast, offline-tolerant web application, Krishi Sheba protects the dealer’s legal standing, stabilizes their cash flow, and protects farmers from inappropriate chemical usage.

---

## 2. In-Depth Platform Profiles & Technical Capabilities

To design Krishi Sheba with global best practices while respecting rural Bangladeshi realities, five global platforms representing agronomy, grain origination, commodity traceability, agribusiness ERP, and modern counter point-of-sale were benchmarked.

```
+---------------------------------------------------------------------------------------------------+
|                               GLOBAL PLATFORMS BENCHMARKED FOR KRISHI SHEBA                       |
+---------------------------------------------------------------------------------------------------+
|  [ AGWORLD ]            [ BUSHEL ]              [ AGRIDIGITAL ]       [ ERPNEXT AGRI ] [ SQUARE/CLOVER ] |
|  Agronomic prescriptions Grain accounting       Commodity grading     Double-entry     Rapid counter POS |
|  Chemical safety checks  Scale tickets & grower Batch provenance      Crop cycles      Offline queuing   |
|  Pre-harvest intervals   Digital settlements    Real-time payment     Input budgeting  Split-tender      |
+---------------------------------------------------------------------------------------------------+
                                                  |
                                                  v
                      +-------------------------------------------------------+
                      |         KRISHI SHEBA SYNTHESIS & LOCALIZATION         |
                      | - DAE 4-Color Toxicity Hazard Bands & Antidote Notes  |
                      | - BCIC / DAE Fertilizer Quota, Subsidies & UAO Recon  |
                      | - BADC / SCA Seed Lab Viability & Expiry Countdown    |
                      | - Customer & Mohajon Seasonal Khata (Boro/Aman/Rabi)  |
                      | - Frictionless WhatsApp / SMS Bilingual Digital Memos |
                      +-------------------------------------------------------+
```

---

### 2.1 Agworld (Collaborative Farm Management & Agronomic Safety)
* **Company Profile:** Australian-founded, globally adopted agronomic data platform operating in the US, Australia, New Zealand, and South Africa. Agworld connects farmers, independent agronomists, and retail input dealers into a unified workflow.
* **Core Technical Capabilities:**
  - **Standardized Agronomic Prescriptions:** Certified crop advisors generate structured digital recommendations containing exact tank-mix formulations, water volumes per hectare, application methods (foliar, broadcast, soil injection), and targeted weed/pest pressures.
  - **Regulatory Crop Protection Guardrails:** Integrates with national chemical databases (such as APVMA in Australia and EPA/CDMS in North America). Automatically blocks illegal tank mixes, enforces statutory maximum dosage rates, and validates mandatory Pre-Harvest Intervals (PHI) and Worker Re-Entry Intervals (REI).
  - **Farm Plan to Retail Inventory Allocation:** When an agronomist formulates a seasonal spray plan, required chemical volumes are automatically converted into retailer demand forecasts, reserving stock at local warehouses.
* **Limitations for Bangladeshi Micro-Retail:**
  - *Excessive Operational Complexity:* Agworld assumes enterprise-scale farms with GPS-mapped shapefiles, tractor telemetry, and corporate agronomists. A village shopkeeper in Bogura or Dinajpur needs to complete a retail transaction within 15 seconds.
  - *Desktop-Centric Workflow:* Agworld’s extensive recommendation builders require desktop screens and high-bandwidth internet connections, making them unsuitable for dusty, mobile-first rural retail counters.
* **Architectural Adaptations for Krishi Sheba:**
  - Standardize product-level chemical metadata in `product_pesticide_specs`: Active Ingredient, DAE Registration Number (`AP-XXXX`), Target Crops, Target Pests, Application Dilution (e.g., `২ মিলি / লিটার`), PHI (days), and REI (hours).
  - Implement instant toxicity alerts on the POS counter: When a cashier adds a Class I or II pesticide to an invoice, the UI prominently displays the statutory warning, withholding period, and required protective equipment.

---

### 2.2 Bushel (Agribusiness Infrastructure, Grain Accounting & Grower Portal)
* **Company Profile:** North American ag-fintech platform powering grain elevators, cooperatives, agricultural input retailers, and food processors. Bushel processes over $22 billion in annual grain transactions.
* **Core Technical Capabilities:**
  - **Scale Tickets & Grain Settlement:** Ingests scale weight tickets directly from truck scale hardware, calculates automated grade moisture/dockage discounts, and executes multi-party settlements (dividing payments between landowners and tenant sharecroppers).
  - **Zero-Friction Grower Portal:** Growers log into lightweight mobile apps to view unbilled scale tickets, commodity contracts, historical grain settlements, and input finance balances.
  - **Digital Agribusiness Contracts:** Issues legally binding crop purchase contracts and input financing agreements with e-signatures directly on mobile phones.
* **Limitations for Bangladeshi Micro-Retail:**
  - *North American Grain Elevator Focus:* Bushel is architected for bulk grain elevators operating 80,000-bushel silo bins, whereas Bangladeshi input merchants sell packaged 50kg fertilizer bags, 100ml pesticide bottles, and 1kg seed packets.
  - *High Native App Barrier:* Expecting smallholder farmers in rural Bangladesh to download, authenticate, and update native iOS/Android apps fails due to device storage constraints, low digital literacy, and variable network coverage.
* **Architectural Adaptations for Krishi Sheba:**
  - **Zero-Install Shareable Grower Statements:** Rather than requiring farmers to install native apps, Krishi Sheba adapts the grower portal paradigm into frictionless, web-standard **SMS and WhatsApp digital memos** generated directly from URI schemes (`wa.me` and `sms:`).
  - Transparent balance breakdowns showing: Current Sale, Government Subsidy Contribution, Prior Khata Balance, Total Due, and Seasonal Repayment Date.

---

### 2.3 AgriDigital (Cloud Commodity Management & Quality Grading)
* **Company Profile:** Australian commodity management SaaS managing grain supply chains, handling inventory logistics, storage bin grading, forward contracts, and automated escrow payments.
* **Core Technical Capabilities:**
  - **Statutory Quality Grading & Dockage Engine:** Real-time laboratory intake recording of moisture %, physical purity %, protein content, test weight, and foreign matter dockage, dynamically adjusting inventory valuation based on quality grades.
  - **Batch Provenance & Traceability:** Tracks grain lots from specific harvest paddocks through transit manifests to end-use millers or export ports.
  - **Automated Settlement & Payment Guarantees:** Instant calculation of storage fees, levies, freight deductions, and net supplier settlements.
* **Limitations for Bangladeshi Micro-Retail:**
  - *Heavy Enterprise Footprint:* Geared toward industrial bulk grain terminals with on-site laboratory technicians and automated grain analyzers.
* **Architectural Adaptations for Krishi Sheba:**
  - **Seed Lot Viability Verification:** Adapt AgriDigital’s quality grading engine into the seed inventory module (`product_seed_specs` and `seed_batch_test_records`).
  - Track official BADC/SCA laboratory metrics: Germination Rate (%), Moisture Content (%), Physical Purity (%), and SCA Test Certificate Number.
  - Automatically calculate the statutory 9-month test validity window. If a seed batch passes 9 months without re-testing, the system flags the batch as non-compliant and blocks sale at the POS.

---

### 2.4 ERPNext Agriculture Module (Open-Source Agribusiness ERP)
* **Company Profile:** Open-source ERP platform built on the Python/Frappe framework, featuring a dedicated Agriculture module designed for commercial farming operations, plantations, and agricultural processing enterprises.
* **Core Technical Capabilities:**
  - **Crop Cycles & Seasonal Periods:** Organizes operations around structured crop cycles (e.g., Planting Date, Vegetative Stage, Flowering, Harvest Window, Projected Yield) linked to analytical cost centers.
  - **Input Tracking per Crop Cycle:** Allocates disbursements of seed, fertilizer, and crop protection chemicals to specific plots, tracking unit costs against projected output.
  - **Double-Entry Farmer Credit Accounting:** Manages farmer advances as accounts receivable, settling debts through delivered harvest produce or cash repayments.
* **Limitations for Bangladeshi Micro-Retail:**
  - *High Latency and Server Resource Consumption:* The Python/Frappe stack is resource-heavy, resulting in sluggish page loads on 3G/poor 4G rural mobile connections.
  - *Over-Engineered Enterprise Forms:* Requires dozens of non-mandatory form fields (land coordinates, soil pH, irrigation pump serial numbers) that impede fast retail counter sales.
* **Architectural Adaptations for Krishi Sheba:**
  - **Seasonal Crop Cycle Architecture:** Adopt ERPNext’s seasonal period abstraction directly into the Customer and Mohajon Khata ledgers.
  - Structure credit and inventory tracking around the four core Bangladeshi seasons: **Boro (বোরো)**, **Aman (আমন)**, **Aus (আউশ)**, and **Rabi (রবি)**.
  - Enable dealers to view outstanding receivables grouped by harvest cycle, matching repayments to when farmers actually liquidate their crops.

---

### 2.5 Clover & Square POS (Rapid Counter Checkout & Offline Resilience)
* **Company Profile:** Market-leading point-of-sale platforms powering millions of small-to-medium retail merchants globally with rapid touch-screen checkouts and robust payment processing.
* **Core Technical Capabilities:**
  - **High-Velocity Counter Ergonomics:** Sub-second item search, rapid barcode scanning, 44px+ touch targets, and streamlined keyboard shortcuts designed for high customer volume.
  - **Offline Resilience:** Local transaction queuing, optimistic UI rendering, and background synchronization when network connections drop.
  - **Split-Tender Payment Capabilities:** Allows a single sale to be paid across multiple tenders: partial cash, debit/credit cards, gift cards, and "House Account" (store credit).
  - **Instant Digital Receipts:** Generates receipts sent directly via SMS and email without thermal paper printers.
* **Limitations for Bangladeshi Micro-Retail:**
  - *Generic Retail Focus:* Completely devoid of agricultural domain models. No support for government fertilizer quotas, maximum retail price ceilings, toxicity warning bands, seed germination testing, or seasonal harvest credit terms.
  - *Proprietary Hardware Lock-In:* Designed for proprietary payment terminals not widely distributed or affordable in rural Bangladesh.
* **Architectural Adaptations for Krishi Sheba:**
  - Emulate the ergonomic, keyboard-navigable, high-contrast POS counter layout for low-cost Android tablets, desktop PCs, and budget smartphones (360px+ viewport support).
  - Implement multi-tender split payments tailored to rural Bangladesh: **Cash (নগদ)**, **bKash (বিকাশ)**, **Nagad (নগদ এমএফএস)**, **Bank Transfer/Cheque (ব্যাংক)**, and **Khata Credit (বাকির খাতা)**.
  - Implement client-side optimistic UI state updates for immediate checkout feedback even during intermittent network drops.

---

## 3. Structured Comparative Feature Matrix

The following multi-column matrix evaluates the five global benchmark platforms against Krishi Sheba across 18 specialized agricultural, regulatory, and retail operational dimensions:

| # | Capability Dimension | Agworld | Bushel | AgriDigital | ERPNext Agri | Clover / Square | **Krishi Sheba (Bangladeshi Operating System)** |
|:---|:---|:---:|:---:|:---:|:---:|:---:|:---|
| 1 | **BCIC/BADC Fertilizer Quota Tracking** | ❌ None | ❌ None | ❌ None | ⚠️ Manual BOM | ❌ None | **✅ Native Upazila/Union monthly quota ledger & balance tracking** |
| 2 | **Gazetted Farmer MRP & Fixed Margin** | ❌ None | ❌ None | ❌ None | ⚠️ Generic pricing | ❌ None | **✅ Mandatory MRP caps (Urea 27, DAP 21, TSP 27, MOP 20) & Tk 3/kg margin** |
| 3 | **Farmer Krishok Card / NID Validation** | ❌ None | ❌ None | ❌ None | ❌ None | ❌ None | **✅ 10-digit Krishok Card & 10/17-digit NID quota validation at checkout** |
| 4 | **DAE/WHO 4-Color Toxicity Hazard Bands** | ⚠️ Text notes | ❌ None | ❌ None | ❌ None | ❌ None | **✅ Statutory Red, Yellow, Blue, Green visual bands + toxic class icons** |
| 5 | **Pre-Harvest Interval (PHI) Guardrails** | ✅ Full APVMA/EPA | ❌ None | ❌ None | ❌ None | ❌ None | **✅ Mandatory PHI (days) countdown & consumer harvest safety warnings** |
| 6 | **Re-Entry Interval (REI) & PPE Directives** | ✅ Full worker safety | ❌ None | ❌ None | ❌ None | ❌ None | **✅ Statutory REI (hours) & bilingual PPE icon checklist at point of sale** |
| 7 | **Emergency Antidote & First-Aid Protocols** | ⚠️ SDS link only | ❌ None | ❌ None | ❌ None | ❌ None | **✅ Clinical antidote instructions (Atropine, 2-PAM, lavage) in Bengali/English** |
| 8 | **Seed Certification Class Tagging** | ❌ None | ❌ None | ⚠️ Grain specs | ❌ None | ❌ None | **✅ BADC/SCA classes: Foundation (White), Certified (Blue), TLS (Green/Yellow)** |
| 9 | **Seed Lab Viability Thresholds** | ❌ None | ❌ None | ✅ Lab metrics | ❌ None | ❌ None | **✅ Statutory Paddy thresholds: Min 80% Germination, Max 12% Moisture, Min 98% Purity** |
| 10 | **9-Month Seed Test Validity Tracking** | ❌ None | ❌ None | ⚠️ Manual shelf | ❌ None | ❌ None | **✅ Automated 9-month test expiry countdown with POS sale-blocking rules** |
| 11 | **Annual Regulatory License Tracking** | ❌ None | ❌ None | ❌ None | ❌ None | ❌ None | **✅ BCIC, DAE, and SCA dealer license expiry alerts (30-day renewal warnings)** |
| 12 | **Seasonal Customer Credit (Khata)** | ❌ None | ⚠️ Grain contracts | ⚠️ Grower dues | ✅ Crop cycle AR | ⚠️ Generic tabs | **✅ Boro, Aman, Aus, Rabi harvest repayment schedules with credit limits** |
| 13 | **Supplier Payables (Mohajon Khata)** | ❌ None | ❌ None | ❌ None | ✅ AP module | ⚠️ Generic bills | **✅ Distributor ledger for ACI, Square, Petrochem, Syngenta, Bayer terms** |
| 14 | **Rapid Counter POS & Barcode Scanner UX** | ❌ Complex agronomy | ❌ Portal only | ❌ Back-office | ⚠️ Slow ERP forms | ✅ Industry benchmark | **✅ Sub-second checkout, barcode search, 44px touch targets, keyboard nav** |
| 15 | **Multi-Tender Split Payments** | ❌ None | ❌ Settlement | ❌ Wire only | ⚠️ Multi-line journal | ✅ Native split | **✅ Cash, bKash, Nagad, Bank, and Khata credit combined in a single sale** |
| 16 | **Offline-Tolerant Low-Bandwidth Engine** | ⚠️ Partial mobile | ❌ High latency | ❌ Cloud only | ❌ Python/Frappe lag | ✅ Offline queue | **✅ Lightweight bundle, client-side optimistic updates, resilient sync** |
| 17 | **Zero-Install Shareable Invoices** | ❌ Email/PDF only | ⚠️ Mobile app | ❌ Portal only | ❌ Email only | ⚠️ SMS / Email | **✅ One-click bilingual WhatsApp (`wa.me`) & GSM SMS (`sms:`) receipts** |
| 18 | **UAO Monthly Stock Reconciliation Report** | ❌ None | ❌ None | ❌ None | ❌ None | ❌ None | **✅ One-click statutory Upazila Agriculture Officer opening/closing report** |

---

## 4. Bangladeshi Agricultural Retail Realities & Localized Workflows

```
+----------------------------------------------------------------------------------------------------+
|                                BANGLADESHI AGRI-RETAIL ECOSYSTEM                                   |
+----------------------------------------------------------------------------------------------------+
|                                                                                                    |
|    GOVERNMENT BODIES / REGULATORS               DISTRIBUTORS & MANUFACTURERS ("MOHAJON")           |
|    - BCIC (Ministry of Industries)              - ACI Agribusiness / ACI Formulations              |
|    - BADC (Ministry of Agriculture)             - Square Agribusiness                              |
|    - DAE (Plant Protection Wing)                - Petrochem (Bangladesh) Ltd                       |
|    - SCA (Seed Certification Agency)            - Auto Crop Care Ltd / Bayer / Syngenta            |
|    - Upazila Agri Officer (UAO / UNO)                                                              |
|                     |                                             |                                |
|          Quotas, Subsidies & Approvals                 Dealer Credit, Consignments                 |
|                     v                                             v                                |
|        +-------------------------------------------------------------------------+                 |
|        |                   KRISHI SHEBA AGRI-DEALER OPERATING HUB                |                 |
|        |                                                                         |                 |
|        |  [1. Fertilizer Quota]      [2. Pesticide Safety]     [3. Seed Trace]   |                 |
|        |  - Subsidized MRP caps      - DAE Color Bands         - SCA Class (TLS) |                 |
|        |  - Farmer NID/Card log      - PHI days, PPE icons     - Germination %   |                 |
|        |  - UAO Monthly Recon        - Antidote/Usage notes    - Expiry tracking |                 |
|        |                                                                         |                 |
|        |  [4. Customer Seasonal Khata]        [5. Mohajon Distributor Khata]     |                 |
|        |  - Boro/Aman/Aus/Rabi credit         - Company credit lines/terms       |                 |
|        |  - Harvest settlement scheduling     - Reconcile inward shipments       |                 |
|        |  - SMS / WhatsApp bill sharing       - Distributor payables ledger      |                 |
|        +-------------------------------------------------------------------------+                 |
|                                             |                                                      |
|                                  Counter Sales & Credit                                            |
|                                             v                                                      |
|                             FARMERS / GROWERS (কৃষক ও চাষী)                                        |
|                             - National ID / Krishok Card holders                                   |
|                             - Seasonal harvest cash settlement                                     |
+----------------------------------------------------------------------------------------------------+
```

---

### 4.1 Fertilizer Quota Management & Subsidy Allocation Framework

#### 1. Statutory Governance & Distribution Hierarchy
The fertilizer distribution mechanism in Bangladesh is one of the most strictly supervised state-controlled commodity operations in South Asia:
- **BCIC (Bangladesh Chemical Industries Corporation):** Under the Ministry of Industries, BCIC manages the domestic manufacturing (KAFCO, Jamuna Fertilizer, Shahjalal Fertilizer Company, Ghorashal-Polash Fertilizer PLC) and international importation of **Urea**. BCIC appoints certified primary dealers at the Upazila level.
- **BADC (Bangladesh Agricultural Development Corporation):** Under the Ministry of Agriculture, BADC oversees the procurement, buffer stocking, and distribution of non-urea fertilizers: **TSP (Triple Super Phosphate)**, **DAP (Di-Ammonium Phosphate)**, and **MOP (Muriate of Potash)**.
- **District & Upazila Fertilizer and Seed Monitoring Committee (জেলা ও উপজেলা সার ও বীজ মনিটরিং কমিটি):** Chaired by the Deputy Commissioner (DC) at the district level and the Upazila Nirbahi Officer (UNO) at the sub-district level, with the Upazila Agriculture Officer (UAO) serving as Member Secretary. This statutory committee calculates monthly fertilizer quotas for every Union based on net cultivable area, target crop acreages, and seasonal fertilizer demand.

#### 2. Gazetted Subsidized Price Schedule & Dealer Margin Economics
The Ministry of Agriculture establishes nationwide fixed prices for both dealer acquisition from buffer godowns and retail sale to farmers. To prevent black-market extortion and ensure retail viability, dealers are granted a statutory regulated margin of **Tk 3.00 per kg (equivalent to Tk 150.00 per 50kg bag)**.

| Fertilizer Type (রাসায়নিক সার) | Primary Nutrient Composition | Dealer Acquisition Price | Farmer MRP (Gazetted) | Bag Size | Subsidized Farmer Bag Price | Dealer Margin / Bag | Market Cost Without Subsidy (Est.) |
|:---|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| **Urea (ইউরিয়া)** | 46% Nitrogen (N) | Tk 24.00 / kg | **Tk 27.00 / kg** | 50 kg | **Tk 1,350.00** | Tk 150.00 | ~Tk 4,500.00 |
| **DAP (ডিএপি)** | 18% N, 46% P₂O₅ | Tk 18.00 / kg | **Tk 21.00 / kg** | 50 kg | **Tk 1,050.00** | Tk 150.00 | ~Tk 5,200.00 |
| **TSP (টিএসপি)** | 46% P₂O₅ | Tk 24.00 / kg | **Tk 27.00 / kg** | 50 kg | **Tk 1,350.00** | Tk 150.00 | ~Tk 4,800.00 |
| **MOP (এমওপি / পটাশ)** | 60% K₂O | Tk 17.00 / kg | **Tk 20.00 / kg** | 50 kg | **Tk 1,000.00** | Tk 150.00 | ~Tk 3,800.00 |

*Legal Enforcement:* Under the Essential Commodities Act and Fertilizer Control Order, selling fertilizer above the gazetted MRP or hoarding subsidized stock is a non-bailable offense resulting in immediate cancellation of dealership licenses, forfeiture of bank guarantees, and criminal prosecution via executive mobile courts.

#### 3. Farmer Krishok Card (কৃষি উপকরণ সহায়তা কার্ড) & NID Quota Validation
To prevent illegal hoarding, bulk smuggling across borders, or diversion to commercial aquaculture and industrial buyers:
- Agro-dealers are legally required to verify and log the farmer’s **10-digit Krishok Card Number** (কৃষি কার্ড) or **10/17-digit National Identification Number (NID)** for every subsidized fertilizer transaction.
- DAE establishes recommended agronomic application benchmarks per Bigha (33 decimals / ~0.133 hectare):
  * **Boro Rice:** 35–40 kg Urea, 12–15 kg DAP, 15–18 kg MOP per Bigha.
  * **Aman Rice:** 25–30 kg Urea, 10–12 kg DAP, 10–12 kg MOP per Bigha.
  * **Potato (আলু):** 40–45 kg Urea, 30–35 kg DAP, 40–50 kg MOP per Bigha.
- **POS Validation Rules in Krishi Sheba:**
  1. Default single-transaction ceiling: Maximum **3 bags (150 kg)** per fertilizer type per farmer without special written endorsement from the Sub-Assistant Agriculture Officer (SAAO).
  2. Duplicate purchase alert: Warns the cashier if the same Krishok Card / NID purchased fertilizer within the preceding 14 days in the same Union.
  3. Real-time quota balance deduction: Every completed bag sale deducts immediately from the store's active monthly allocation.

#### 4. UAO Monthly Stock Reconciliation Formula
On the first day of each calendar month, every licensed dealer must submit an official stock return to the Upazila Agriculture Office. The reconciliation formula is:

$$\text{Closing Balance} = \text{Opening Balance} + \text{Received Allocation (DO/Challan)} - \text{Total Verified Farmer Sales} - \text{Permitted Spillage/Damage}$$

Krishi Sheba automates this return, allowing dealers to generate and print the exact government reconciliation return with one click, eliminating hours of manual ledger calculations.

---

### 4.2 Pesticide Safety & Toxicity Hazard Classifications (DAE & WHO)

Crop protection chemicals are strictly regulated under the **Pesticides Act 2018** (বালাইনাশক আইন ২০১৮) and the **Pesticide Rules 1985 (amended 2020)** by the **Plant Protection Wing of the Department of Agricultural Extension (DAE)**.

```
+----------------------------------------------------------------------------------------------------+
| DAE / WHO PESTICIDE TOXICITY HAZARD BANDS & STATUTORY SAFETY SPECIFICATIONS                         |
+----------------------------------------------------------------------------------------------------+
| Band Color   | WHO Class     | Toxicity Rating      | Oral LD50 (mg/kg) | Statutory Bengali Signal |
+--------------+---------------+----------------------+-------------------+--------------------------+
| [ RED ]      | Class Ia / Ib | Extremely / Highly   | < 50 mg/kg        | বিপদ / বিষ (POISON)      |
|              |               | Hazardous            |                   | Skull & Crossbones       |
+--------------+---------------+----------------------+-------------------+--------------------------+
| [ YELLOW ]   | Class II      | Moderately Hazardous | 50 – 500 mg/kg    | সাবধান / ক্ষতিকর (WARNING)|
+--------------+---------------+----------------------+-------------------+--------------------------+
| [ BLUE ]     | Class III     | Slightly Hazardous   | 500 – 2000 mg/kg  | সতর্কতা (CAUTION)        |
+--------------+---------------+----------------------+-------------------+--------------------------+
| [ GREEN ]    | Class IV / U  | Unlikely Acute Hazard| > 2000 mg/kg      | তুলনামূলক নিরাপদ (SAFE) |
+----------------------------------------------------------------------------------------------------+
```

#### Detailed Breakdown of DAE Hazard Classes

1. **Red Band (Category Ia / Ib — Extremely / Highly Hazardous):**
   - *Toxicological Criteria:* Oral $LD_{50}$ in rats under 50 mg/kg body weight; dermal $LD_{50}$ under 200 mg/kg.
   - *Active Ingredient Examples:* Carbofuran (Furadan), Phorate, Phosphamidon, Monocrotophos. (Note: DAE has phased out many Class I substances; remaining active approvals are restricted to granular formulations for soil application against severe nematode/stem borer infestations).
   - *Statutory Retail Guardrails:* Prohibited from open shelf display; must be stored in locked steel cabinets. Explicitly banned for use on leafy vegetables, fruits, and tea gardens.
   - *Krishi Sheba UI Treatment:* High-contrast red badge (`#DC2626`), skull & crossbones indicator, mandatory cashier verbal confirmation prompt before adding to cart.

2. **Yellow Band (Category II — Moderately Hazardous):**
   - *Toxicological Criteria:* Oral $LD_{50}$ between 50 and 500 mg/kg.
   - *Active Ingredient Examples:* Cypermethrin (Ripcord, Ustad), Chlorpyrifos, Dimethoate (Rogor), Cartap (Cadan), Lambda-cyhalothrin (Karate), Diazinon.
   - *Health Hazards:* Severe neurotoxicity (cholinesterase inhibition), acute dermal burning, high aquatic and pollinator toxicity.
   - *Mandatory PPE Requirements:* Chemical-resistant nitrile gloves, particulate face mask, splash goggles, full long-sleeved clothing.
   - *Krishi Sheba UI Treatment:* Amber/Yellow warning badge (`#D97706`), exclamation warning icon, automated display of Pre-Harvest Intervals.

3. **Blue Band (Category III — Slightly Hazardous):**
   - *Toxicological Criteria:* Oral $LD_{50}$ between 500 and 2000 mg/kg.
   - *Active Ingredient Examples:* Mancozeb (Dithane M-45, Indofil), Hexaconazole, Thiamethoxam, Imidacloprid (Admire, Tido), Carbendazim (Autostin).
   - *Health Hazards:* Mild skin and mucosal irritant; toxic if ingested in large quantities; moderate environmental persistence.
   - *Mandatory PPE Requirements:* Protective face mask, rubber gloves, immediate post-application handwashing.
   - *Krishi Sheba UI Treatment:* Blue safety badge (`#2563EB`), caution shield icon.

4. **Green Band (Category IV / Class U — Unlikely to Present Acute Hazard):**
   - *Toxicological Criteria:* Oral $LD_{50}$ exceeding 2000 mg/kg.
   - *Active Ingredient Examples:* Biologicals (*Trichoderma viride*, *Bacillus thuringiensis*), Azadirachtin (neem oil formulations), Azoxystrobin + Difenoconazole (Amistar Top), Wettable Sulphur (Kumulus).
   - *Health Hazards:* Negligible acute systemic toxicity under normal usage conditions. Compatible with Integrated Pest Management (IPM).
   - *Krishi Sheba UI Treatment:* Green eco badge (`#16A34A`), leaf icon.

#### Critical Operational Chemical Attributes
Every pesticide catalog record in Krishi Sheba mandates six critical attributes:
- **DAE Registration Number:** Official government approval number (e.g., `AP-1892` or `AP-2451` issued under Form 17/10).
- **Target Approved Crops (অনুমোদিত ফসল):** e.g., ধান (Paddy), আলু (Potato), বেগুন (Brinjal), মরিচ (Chilli), ভুট্টা (Maize).
- **Target Pests & Diseases (দমনযোগ্য পোকা ও রোগ):** e.g., মাজরা পোকা (Stem Borer), বাদামি গাছফড়িং / কারেন্ট পোকা (Brown Planthopper - BPH), নাবি ধসা (Late Blight), ব্লাস্ট রোগ (Rice Blast).
- **Pre-Harvest Interval (PHI) in Days (ফল তোলার পূর্ববর্তী অপেক্ষমাণ সময়):** The legally required minimum waiting period between the last chemical application and food harvest:
  * Virtako (Chlorantraniliprole + Thiamethoxam): 14 days
  * Ripcord 10 EC (Cypermethrin): 21 days
  * Mancozeb 80 WP: 7 days
  * Amistar Top 325 SC: 10 days
- **Re-Entry Interval (REI) in Hours:** Waiting period before farm laborers may safely enter treated fields without protective gear (default 24 to 48 hours).
- **Antidote & First-Aid Protocol (বিষক্রিয়ার লক্ষণ ও প্রতিষেধক):** Immediate clinical directives printed on receipts (e.g., *"If organophosphate poisoning: Administer 2-4 mg Atropine Sulphate intravenously every 10-15 mins until atropinization. Pralidoxime (2-PAM) may be used. Induce gastric lavage immediately."*).

---

### 4.3 Seed Certification & Traceability (BADC & SCA)

Seed production, conditioning, and distribution in Bangladesh are governed by the **Seeds Act 2018** (বীজ আইন ২০১৮) and the **Seed Rules 2020**, administered by the **Seed Certification Agency (SCA)** under the Ministry of Agriculture.

#### 1. Official Seed Generations & Color-Coded Certification Tags
All legally packaged seed lots in Bangladesh must carry an official tag matching their generation:
- **Breeder Seed (প্রজনন বীজ):** Produced directly by specialized research institutions (BRRI, BARI, BINA, BJRI). Carries a **Yellow Tag**. Never distributed via retail counters; exclusively allocated to licensed foundation seed multipliers.
- **Foundation Seed (ভিত্তি বীজ):** Multiplied directly from breeder seed by BADC and accredited corporate seed enterprises. Carries a **White Tag (সাদা ট্যাগ)**.
- **Certified Seed (প্রত্যয়িত বীজ):** Multiplied from foundation seed, subject to rigorous field inspections by SCA seed inspectors and certified by official laboratory analysis. Carries an **Azure Blue Tag (নীল ট্যাগ)**.
- **Truthfully Labelled Seed (TLS / টিএলএস - মান ঘোষিত বীজ):** Multiplied and packaged by licensed private seed enterprises (e.g., ACI Seeds, Lal Teer Seed, Supreme Seed Company, Syngenta, Metal Agro). Carries an **Opal Green / Yellow Tag (সবুজ-হলুদ ট্যাগ)**. Under the Seeds Act 2018, the merchant and producer bear joint civil and criminal liability for the accuracy of declared germination, moisture, and purity specs.

#### 2. Statutory Quality Thresholds & Laboratory Parameters
To protect smallholders from defective, dead, or adulterated seed, the SCA enforces strict statutory thresholds. Seed batches that fail any of these benchmarks cannot legally be sold:

| Crop Category | Statutory Minimum Germination Rate (%) | Statutory Maximum Moisture Content (%) | Statutory Minimum Physical Purity (%) |
|:---|:---:|:---:|:---:|
| **Paddy (উফশী ও হাইব্রিড ধান)** | **≥ 80.0%** | **≤ 12.0%** | **≥ 98.0%** |
| **Hybrid Maize (ভুট্টা)** | **≥ 85.0%** | **≤ 12.0%** | **≥ 98.0%** |
| **Wheat (গম)** | **≥ 85.0%** | **≤ 12.0%** | **≥ 98.0%** |
| **Jute (পাট - তোষা ও দেশী)** | **≥ 75.0%** | **≤ 10.0%** | **≥ 97.0%** |
| **Oilseeds (সরিষা, তিল)** | **≥ 80.0%** | **≤ 8.0%** | **≥ 97.0%** |
| **Vegetables (আলু, বেগুন, টমেটো)** | **≥ 70.0% – 75.0%** | **≤ 8.0%** | **≥ 98.0%** |

#### 3. The 9-Month Laboratory Test Validity Rule
Under Section 14 of the Seed Rules 2020:
- Laboratory germination test certificates remain legally valid for a maximum of **nine (9) calendar months** from the official date of testing.
- If a seed batch remains unsold at the 9-month mark, its commercial sale is illegal. The dealer must either:
  1. Have the batch re-sampled and re-tested by an authorized SCA regional laboratory.
  2. Withdraw and destroy the batch.
- **Krishi Sheba Traceability Engine:**
  * Tracks `testDate`, `expiryDate`, and calculates dynamic `daysToExpiry`.
  * Triggers visual warnings when a batch is within 30 days of the 9-month expiration.
  * Disables addition to cart and displays an error if an expired batch is selected at checkout.

#### 4. Seed Dealer Licensing & Annual Renewal Tracking
Every retailer selling seeds must hold an active **Seed Dealer Registration Certificate** (বীজ ডিলার নিবন্ধনপত্র) issued by the SCA District Seed Committee. This license requires **annual renewal** upon payment of statutory treasury fees. Krishi Sheba tracks dealer license numbers, issuing dates, and automated 30-day renewal reminders.

---

### 4.4 Agricultural Credit Ledger (Customer & Mohajon Khata)

Rural commerce in Bangladesh revolves around seasonal, informal, and semi-formal credit arrangements:

```
+----------------------------------------------------------------------------------------------------+
|                                    AGRICULTURAL SEASONAL CREDIT TIMELINE                           |
+----------------------------------------------------------------------------------------------------+
|  PLANTING & VEGETATIVE PHASE                          HARVEST & POST-HARVEST SETTLEMENT            |
|  - Farmers collect seed & fertilizer on credit        - Crop harvested, dried & sold at local Hat  |
|  - Dealer logs transaction to Customer Khata          - Farmer visits shop with crop revenue       |
|  - Mohajon delivers stock on 60-90 day terms          - Customer Khata fully or partially settled  |
|  - Dealer tracks accounts payable in Mohajon Khata    - Dealer clears Mohajon wholesale bills      |
+----------------------------------------------------------------------------------------------------+
```

#### 1. Customer (Farmer) Khata: Seasonal Crop Cycles
Smallholder farmers experience extreme seasonal cash-flow asymmetry: they must purchase seed, basal fertilizer, and pest protection during land preparation and vegetative growth, but have zero liquidity until after harvest. Consequently, agricultural credit must be structured around crop seasons rather than conventional monthly billing:

- **Boro Season (বোরো মৌসুম):**
  * Sowing/Transplanting: November – January (কার্তিক – পৌষ).
  * Harvesting/Threshing: April – May (বৈশাখ – জ্যৈষ্ঠ).
  * *Commercial Impact:* The single largest credit and input consumption window in Bangladesh, representing over 60% of national Urea, DAP, and hybrid paddy seed sales. Credit extended in December is scheduled for settlement in May.
- **Aman Season (রোপা আমন / খরিপ-২):**
  * Sowing/Transplanting: July – August (আষাঢ় – শ্রাবণ).
  * Harvesting: November – December (অগ্রহায়ণ – পৌষ).
  * *Commercial Impact:* Heavy demand for top-dressing urea, granular insecticides (Cartap/Carbofuran), and BPH sprays.
- **Aus Season (আউশ মৌসুম / খরিপ-১):**
  * Sowing: March – April (চৈত্র – বৈশাখ); Harvesting: July – August (আষাঢ় – শ্রাবণ).
- **Rabi Season (রবি মৌসুম — আলু, ভুট্টা, সরিষা, শীতকালীন সবজি):**
  * Sowing: October – November; Harvesting: January – March.
  * *Commercial Impact:* High capital requirements for seed potatoes, DAP, MOP, and fungicides (Mancozeb, Difenoconazole).

*Krishi Sheba Seasonal Khata Capabilities:*
- **Season-Tagged Credit Invoicing:** Every credit transaction is explicitly tagged to a crop season (e.g., `BORO-2026`).
- **Promised Harvest Settlement Date:** Cashiers record the expected crop liquidation date (e.g., 20 May 2026).
- **Grace Period & Overdue Status:** Grace periods (typically 15 to 30 days post-harvest) prevent premature badgering of loyal farmers during market price dips.
- **Credit Limits & Warnings:** Configurable credit ceilings per farmer (default Tk 25,000) trigger visual warnings at POS when exceeded.
- **Partial Payment Allocation:** Automatically applies partial repayments to the oldest outstanding invoice.

#### 2. Mohajon (Wholesale/Distributor) Khata
Input retailers purchase branded pesticides, specialty micronutrients, and hybrid seeds from regional sales depots of major agribusiness conglomerates (*Mohajons*):
- **Key Agribusiness Manufacturers in Bangladesh:**
  * **ACI Agribusiness / ACI Formulations:** Ripcord, Floro, Karate, ACI Dhan seeds.
  * **Square Agribusiness / Square Agro:** Agro-chemicals, animal health, specialty foliar feeds.
  * **Petrochem (Bangladesh) Ltd:** Major importer of pesticides, soluble fertilizers, and hybrid seeds.
  * **Auto Crop Care Ltd:** Leading domestic formulator of fungicides and insecticides (Autostin, Mitac, Relay).
  * **Syngenta Bangladesh Ltd:** Premium market leader (Virtako, Amistar Top, Score, Cruiser, hybrid corn).
  * **Bayer CropScience Bangladesh:** Admire, Nativo, Confidor, hybrid brassica seeds.
  * **Ispahani Agro Ltd:** Bio-pesticides, organic amendments, and certified cereal seeds.
  * **McDonald Crop Care Ltd:** Foliar nutrients, plant growth regulators, and acaricides.
- **Commercial Credit Terms:**
  * Distributors deliver inventory on 30-, 60-, or 90-day credit terms, often backed by undated security cheques.
  * Dealers qualify for volume-based seasonal rebates, cash-early discounts, and target bonuses upon harvest clearing.
  * The **Mohajon Khata** module tracks: Distributor Account, Challan / DO Number, Billed Amount, Paid Amount (Bank Transfer, Cheque, Cash), Outstanding Payable, and Upcoming Due Dates.

#### 3. Shareable Bilingual Receipts & Notifications (SMS & WhatsApp)
Rural farmers rarely use email and often discard physical thermal paper slips. Krishi Sheba provides zero-install, instant bilingual receipts generated via native device URI protocols:

*Bilingual GSM-Compliant SMS Template (160 characters):*
```text
Krishi Sheba-Bhai Bhai Traders
Inv:#INV-1042 Boro-26
T:1350 Tk, Paid:500 Tk
Due:850 Tk (Total Due:4200 Tk)
Govt Subsidy:3150 Tk
Pay after harvest. Help:01711XXXXXX
```

*Rich WhatsApp Invoice Template (Bengali / English with Emoji Hierarchy):*
```text
🌾 *কৃষি সেবা — ভাই ভাই ট্রেডার্স* 🌾
ঠিকানা: শিবগঞ্জ বাজার, বগুড়া | মোবা: ০১৭১১-XXXXXX
ডিলার লাইসেন্স: BCIC-1049 | বীজ লাইসেন্স: SCA-BOG-2024-88

🧾 *ক্যাশ মেমো / বিক্রয় চালান*: #INV-2026-0418
📅 তারিখ: ০৫ নভেম্বর ২০২৬ | সময়: বিকাল ০৫:৩০
👤 ক্রেতা: মোঃ রফিকুল ইসলাম (চকপাড়া)
💳 কৃষক কার্ড / NID: 1985101234567890
🌱 মৌসুম: বোরো ২০২৬ (Boro Season)
----------------------------------------
আইটেম বিবরণ:
১. ইউরিয়া সার (BCIC) — ৫০ কেজি (১ বস্তা)
   লট: UR-2026-B1 | মূল্য: ১,৩৫০ টাকা
২. স্কোর ২৫০ ইসি (সিনজেন্টা) — ১০০ মিলি (১ বোতল)
   লট: SC-2025-11 | AP-1892 | মূল্য: ৩২০ টাকা
   ⚠️ সতর্কতা: নীল ব্যান্ড (Blue Band) | PHI: ১৪ দিন
----------------------------------------
উপমোট (Subtotal): ১,৬৭০ টাকা
ছাড় (Discount): ২০ টাকা
সর্বমোট (Grand Total): ১,৬৫০ টাকা
পরিশোধ (Cash Paid): ১,০০০ টাকা
বর্তমান চালানে বাকি: ৬৫০ টাকা
----------------------------------------
📌 *পূর্বের বকেয়া*: ৩,৫০০ টাকা
🔴 *মোট বাকি (Total Khata Due)*: ৪,১৫০ টাকা
🌾 *পরিশোধের প্রতিশ্রুতি*: বোরো ধান কাটার পর (মে ২০২৬)

✨ *সরকারি ভর্তুকি সঞ্চয়*: প্রায় ৩,১৫০ টাকা!
(সরকার কৃষকের জন্য সারে প্রতি কেজিতে সর্বোচ্চ ভর্তুকি দিচ্ছে)

ধন্যবাদ, আবার আসবেন! Krishi Sheba
```

---

## 5. Technical Architecture & Schema Specification for Krishi Sheba

To support these agricultural workflows, the underlying schema, contracts, and APIs must be explicitly typed and validated.

### 5.1 Relational Data Models & DDL (PostgreSQL / Supabase)

```sql
-- ============================================================================
-- 1. ENUMS FOR AGRICULTURAL DOMAIN SPECIFICATIONS
-- ============================================================================

CREATE TYPE pesticide_hazard_band AS ENUM (
  'RED',      -- WHO Class Ia/Ib: Extremely/Highly Hazardous (Skull & Crossbones)
  'YELLOW',   -- WHO Class II: Moderately Hazardous (Warning)
  'BLUE',     -- WHO Class III: Slightly Hazardous (Caution)
  'GREEN'     -- WHO Class IV/U: Unlikely to Present Acute Hazard (Safe)
);

CREATE TYPE seed_certification_class AS ENUM (
  'BREEDER',    -- Breeder Seed (প্রজনন বীজ - Yellow Tag)
  'FOUNDATION', -- Foundation Seed (ভিত্তি বীজ - White Tag)
  'CERTIFIED',  -- Certified Seed (প্রত্যয়িত বীজ - Blue Tag)
  'TLS'         -- Truthfully Labelled Seed (মান ঘোষিত বীজ - Green/Yellow Tag)
);

CREATE TYPE crop_season AS ENUM (
  'BORO',       -- বোরো মৌসুম (November - May)
  'AMAN',       -- আমন মৌসুম (June - December, Kharif-2)
  'AUS',        -- আউশ মৌসুম (March - August, Kharif-1)
  'RABI',       -- রবি মৌসুম (October - March)
  'YEAR_ROUND'  -- বারোমাসি
);

CREATE TYPE tender_payment_method AS ENUM (
  'CASH',
  'BKASH',
  'NAGAD',
  'BANK_TRANSFER',
  'KHATA_CREDIT'
);

-- ============================================================================
-- 2. DEALER REGULATORY LICENSING TABLE
-- ============================================================================

CREATE TABLE dealer_regulatory_licenses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  store_id UUID NOT NULL REFERENCES stores(id) ON DELETE CASCADE,
  license_type VARCHAR(50) NOT NULL, -- 'BCIC_FERTILIZER', 'BADC_FERTILIZER', 'DAE_PESTICIDE', 'SCA_SEED'
  license_number VARCHAR(100) NOT NULL,
  issuing_authority VARCHAR(150) NOT NULL,
  issue_date DATE NOT NULL,
  expiry_date DATE NOT NULL,
  upazila VARCHAR(100) NOT NULL,
  district VARCHAR(100) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE', -- 'ACTIVE', 'EXPIRED', 'SUSPENDED'
  metadata_json JSONB DEFAULT '{}'::jsonb,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(tenant_id, store_id, license_type, license_number)
);

-- ============================================================================
-- 3. FERTILIZER QUOTA & SUBSIDY ALLOCATIONS TABLE
-- ============================================================================

CREATE TABLE fertilizer_monthly_quotas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  store_id UUID NOT NULL REFERENCES stores(id) ON DELETE CASCADE,
  fertilizer_type VARCHAR(20) NOT NULL, -- 'UREA', 'DAP', 'TSP', 'MOP'
  allotment_year INT NOT NULL,
  allotment_month INT NOT NULL, -- 1 to 12
  season crop_season NOT NULL,
  allocated_qty_bags NUMERIC(12,2) NOT NULL,
  received_qty_bags NUMERIC(12,2) NOT NULL DEFAULT 0,
  sold_qty_bags NUMERIC(12,2) NOT NULL DEFAULT 0,
  dealer_acquisition_price_paisa BIGINT NOT NULL, -- e.g. 120000 paisa (Tk 1200) for Urea
  subsidized_mrp_paisa BIGINT NOT NULL,          -- e.g. 135000 paisa (Tk 1350) for Urea
  subsidized_savings_per_bag_paisa BIGINT NOT NULL, -- difference vs unsubsidized global price
  challan_number VARCHAR(100),
  uao_approval_ref VARCHAR(100),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(tenant_id, store_id, fertilizer_type, allotment_year, allotment_month)
);

-- ============================================================================
-- 4. PESTICIDE SAFETY & SPECIFICATION ATTRIBUTES
-- ============================================================================

CREATE TABLE product_pesticide_specs (
  product_id UUID PRIMARY KEY REFERENCES products(id) ON DELETE CASCADE,
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  dae_reg_number VARCHAR(100) NOT NULL, -- e.g. 'AP-1892'
  active_ingredient VARCHAR(200) NOT NULL,
  hazard_band pesticide_hazard_band NOT NULL,
  phi_days INT NOT NULL, -- Pre-Harvest Interval in days
  rei_hours INT NOT NULL DEFAULT 24, -- Re-entry interval in hours
  target_crops TEXT[] NOT NULL DEFAULT '{}',
  target_pests TEXT[] NOT NULL DEFAULT '{}',
  application_dosage VARCHAR(200) NOT NULL, -- e.g. '২ মিলি / লিটার পানি'
  antidote_instructions TEXT NOT NULL,
  requires_ppe BOOLEAN NOT NULL DEFAULT TRUE,
  ppe_mask BOOLEAN NOT NULL DEFAULT TRUE,
  ppe_gloves BOOLEAN NOT NULL DEFAULT TRUE,
  ppe_goggles BOOLEAN NOT NULL DEFAULT FALSE,
  is_restricted_class_one BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================================
-- 5. SEED QUALITY, BATCH CERTIFICATION & VIABILITY
-- ============================================================================

CREATE TABLE product_seed_specs (
  product_id UUID PRIMARY KEY REFERENCES products(id) ON DELETE CASCADE,
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  seed_class seed_certification_class NOT NULL,
  crop_variety VARCHAR(150) NOT NULL, -- e.g. 'BRRI Dhan 28', 'NK-40'
  sca_license_number VARCHAR(100),
  min_germination_percentage NUMERIC(5,2) NOT NULL DEFAULT 80.00,
  max_moisture_percentage NUMERIC(5,2) NOT NULL DEFAULT 12.00,
  min_purity_percentage NUMERIC(5,2) NOT NULL DEFAULT 98.00,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE seed_batch_test_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  batch_id UUID NOT NULL REFERENCES product_batches(id) ON DELETE CASCADE,
  test_report_number VARCHAR(100) NOT NULL,
  testing_laboratory VARCHAR(150) NOT NULL, -- e.g. 'SCA Regional Lab Gazipur'
  test_date DATE NOT NULL,
  test_expiry_date DATE NOT NULL, -- statutory 9 months max from test_date
  actual_germination_percentage NUMERIC(5,2) NOT NULL,
  actual_moisture_percentage NUMERIC(5,2) NOT NULL,
  actual_purity_percentage NUMERIC(5,2) NOT NULL,
  is_certified_fit_for_sale BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================================
-- 6. EXTENDED CUSTOMER (FARMER) PROFILE & SEASONAL KHATA
-- ============================================================================

ALTER TABLE customers ADD COLUMN IF NOT EXISTS krishok_card_number VARCHAR(30);
ALTER TABLE customers ADD COLUMN IF NOT EXISTS nid_number VARCHAR(30);
ALTER TABLE customers ADD COLUMN IF NOT EXISTS cultivated_land_bigha NUMERIC(8,2) DEFAULT 0;
ALTER TABLE customers ADD COLUMN IF NOT EXISTS primary_crop_season crop_season DEFAULT 'BORO';

CREATE TABLE customer_seasonal_credit_ledgers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  customer_id UUID NOT NULL REFERENCES customers(id) ON DELETE CASCADE,
  season crop_season NOT NULL,
  calendar_year INT NOT NULL,
  credit_limit_paisa BIGINT NOT NULL DEFAULT 2500000, -- Tk 25,000 default
  total_credit_purchased_paisa BIGINT NOT NULL DEFAULT 0,
  total_repaid_paisa BIGINT NOT NULL DEFAULT 0,
  current_due_paisa BIGINT NOT NULL DEFAULT 0,
  promised_harvest_settlement_date DATE,
  status VARCHAR(20) NOT NULL DEFAULT 'OPEN', -- 'OPEN', 'SETTLED', 'OVERDUE'
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(tenant_id, customer_id, season, calendar_year)
);

-- ============================================================================
-- 7. MOHAJON (DISTRIBUTOR / WHOLESALE) KHATA
-- ============================================================================

CREATE TABLE mohajon_credit_ledgers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  supplier_id UUID NOT NULL REFERENCES suppliers(id) ON DELETE CASCADE,
  company_name VARCHAR(150) NOT NULL, -- e.g. 'ACI Agribusiness', 'Syngenta Bangladesh'
  payment_terms_days INT NOT NULL DEFAULT 60,
  total_invoiced_paisa BIGINT NOT NULL DEFAULT 0,
  total_paid_paisa BIGINT NOT NULL DEFAULT 0,
  current_payable_paisa BIGINT NOT NULL DEFAULT 0,
  last_payment_date DATE,
  next_settlement_due_date DATE,
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(tenant_id, supplier_id)
);
```

---

### 5.2 API Route Interfaces & Zod Validation Schemas (`packages/contracts` & `apps/api`)

To maintain strict type safety between the Fastify backend and the Next.js frontend, Zod schemas define the contracts:

#### 1. Fertilizer Quotas Contract
```typescript
import { z } from 'zod';

export const FertilizerTypeEnum = z.enum(['UREA', 'DAP', 'TSP', 'MOP']);
export const CropSeasonEnum = z.enum(['BORO', 'AMAN', 'AUS', 'RABI', 'YEAR_ROUND']);

export const FertilizerQuotaResponseSchema = z.object({
  id: z.string().uuid(),
  fertilizerType: FertilizerTypeEnum,
  allotmentYear: z.number().int(),
  allotmentMonth: z.number().int(),
  season: CropSeasonEnum,
  allocatedQtyBags: z.number(),
  receivedQtyBags: z.number(),
  soldQtyBags: z.number(),
  remainingBags: z.number(),
  dealerAcquisitionPricePaisa: z.number(),
  subsidizedMrpPaisa: z.number(),
  dealerMarginPerBagPaisa: z.number(),
});

export const CheckFarmerQuotaRequestSchema = z.object({
  farmerNid: z.string().min(10).max(17),
  krishokCardNo: z.string().optional(),
  fertilizerType: FertilizerTypeEnum,
  quantityKg: z.number().positive(),
});

export const CheckFarmerQuotaResponseSchema = z.object({
  allowed: z.boolean(),
  remainingQuotaKg: z.number(),
  gazettedMrpPerKg: z.number(),
  maxAllowedSaleKg: z.number(),
  warningMessage: z.string().optional(),
});
```

#### 2. Pesticide Safety Contract
```typescript
export const PesticideHazardBandEnum = z.enum(['RED', 'YELLOW', 'BLUE', 'GREEN']);

export const PesticideProductSchema = z.object({
  id: z.string().uuid(),
  name: z.string(),
  activeIngredient: z.string(),
  daeRegistrationNo: z.string(),
  hazardBand: PesticideHazardBandEnum,
  hazardClassification: z.string(),
  phiDays: z.number().int(),
  reiHours: z.number().int(),
  targetCrops: z.array(z.string()),
  targetPests: z.array(z.string()),
  applicationDosage: z.string(),
  antidoteInstructions: z.string(),
  ppeRequired: z.array(z.string()),
  isRestrictedClassOne: z.boolean(),
});
```

#### 3. Seed Certification Contract
```typescript
export const SeedClassEnum = z.enum(['BREEDER', 'FOUNDATION', 'CERTIFIED', 'TLS']);

export const SeedBatchRecordSchema = z.object({
  id: z.string().uuid(),
  productId: z.string().uuid(),
  batchNumber: z.string(),
  seedClass: SeedClassEnum,
  cropVariety: z.string(),
  scaCertificateNo: z.string(),
  testDate: z.string(), // ISO date
  expiryDate: z.string(), // ISO date (testDate + 9 months)
  germinationRatePercent: z.number(),
  moisturePercent: z.number(),
  purityPercent: z.number(),
  daysToExpiry: z.number().int(),
  isExpired: z.boolean(),
  isCompliant: z.boolean(),
});
```

#### 4. Agricultural Khata Contract
```typescript
export const CustomerKhataLedgerSchema = z.object({
  customerId: z.string().uuid(),
  customerName: z.string(),
  phone: z.string(),
  krishokCardNumber: z.string().optional(),
  nidNumber: z.string().optional(),
  totalDuePaisa: z.number(),
  creditLimitPaisa: z.number(),
  seasonalBreakdown: z.array(
    z.object({
      season: CropSeasonEnum,
      calendarYear: z.number().int(),
      creditPurchasedPaisa: z.number(),
      repaidPaisa: z.number(),
      currentDuePaisa: z.number(),
      harvestSettlementDate: z.string().optional(),
      status: z.enum(['OPEN', 'SETTLED', 'OVERDUE']),
    })
  ),
});

export const MohajonDistributorSchema = z.object({
  supplierId: z.string().uuid(),
  companyName: z.string(),
  contactPerson: z.string().optional(),
  phone: z.string(),
  paymentTermsDays: z.number().int(),
  totalInvoicedPaisa: z.number(),
  totalPaidPaisa: z.number(),
  currentPayablePaisa: z.number(),
  nextSettlementDueDate: z.string().optional(),
});
```

---

### 5.3 Authentic Human-Centric UI/UX Design System

To ensure adherence to Requirement R3 and avoid synthetic "AI slop":
1. **Elimination of Artificial Tropes:**
   - No generic sparkles, magic-wand icons, glowing gradient borders, or synthetic robot heads.
   - All visual elements represent physical agricultural realia: 50kg fertilizer sacks (বস্তা), official color-band indicator bars (Red, Yellow, Blue, Green), seed certification stamps, and red ledger books (লাল খাতা).
2. **High-Contrast Touch Targets (44px Minimum):**
   - Designed for rural shops subject to direct sunlight, dust, and non-ideal touch inputs. All buttons, tender toggles, and form inputs maintain a minimum clickable bounding box of 44×44px.
3. **Bilingual Typography:**
   - Uses **Hind Siliguri** and **Noto Sans Bengali** for Bengali scripts, paired with clean tabular numerals for monetary amounts, batch codes, and weights.
4. **Keyboard-Driven Rapid POS Ergonomics:**
   - Supports single-key actions: `F2` to focus barcode/name search, `F8` for split tender, `F10` for cash settlement, and `Esc` to cancel.
5. **Print Layouts:**
   - Native CSS `@media print` stylesheets configured for both 80mm thermal receipt rolls and standard A5 cash memos.

---

## 6. Gap Analysis & Strategic Product Roadmap

### 6.1 Capability Gap Analysis

| Functional Domain | Generic Retail POS (Square / Clover) | Enterprise AgTech (Agworld / AgriDigital) | Traditional Manual Dealer (Lal Khata) | **Krishi Sheba (Target State)** |
|:---|:---|:---|:---|:---|
| **Subsidized Fertilizer Quotas** | Complete absence of quota concepts | Enterprise grain/crop planning only | Physical register prone to arithmetic errors | **Automated BCIC quota balances, NID check, UAO report** |
| **Pesticide Safety & DAE Bands** | Generic retail barcodes | High-level chemical tank-mix validation | Handwritten names without toxicity warnings | **Color bands (Red/Yellow/Blue/Green), PHI countdown, PPE** |
| **Seed Viability & Expiry** | Simple inventory expiry date | Bulk grain grading dockages | Visual inspection; frequent missed 9-month expiries | **Lab thresholds (Germination %, Moisture %), 9-mo block** |
| **Seasonal Farmer Credit** | Generic 30-day "House Account" | Formal grain contracts & grower equity | Unstructured paper Khata with high bad debt | **Harvest-linked Khata (Boro/Aman/Rabi) with SMS/WhatsApp** |
| **Supplier Credit (Mohajon)** | Generic Accounts Payable | Bulk commodity trading settlements | Unreconciled delivery challans and disputes | **Branded distributor ledgers (ACI, Square, Petrochem)** |
| **Receipt Delivery** | Email or native SMS | Farmer portal logins | Faded carbon copies, frequently lost | **Zero-install bilingual WhatsApp (`wa.me`) & GSM SMS** |

---

### 6.2 Phased Delivery Roadmap

```
+---------------------------------------------------------------------------------------------------+
| KRISHI SHEBA PHASED DELIVERY ROADMAP                                                              |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|  [MILESTONE 1: BENCHMARKING & PRD]                                                                |
|  - SaaS Benchmarking & Feature Matrix (docs/saas-benchmarking-and-feature-matrix.md) [COMPLETED]   |
|                                                                                                   |
|  [MILESTONE 2: CONTRACTS, DB & API BACKEND]                                                       |
|  - PostgreSQL Migration (supabase/migrations/20261105000001_agricultural_domain.sql)              |
|  - Zod Schemas in packages/contracts (fertilizer, pesticides, seeds, khata)                       |
|  - Fastify Modular Routes in apps/api/src/modules/                                                |
|  - Vitest Unit & Integration Test Suite in apps/api (test script added to package.json)          |
|                                                                                                   |
|  [MILESTONE 3: FRONTEND DOMAIN WORKFLOWS & POS]                                                   |
|  - Domain UI in apps/saas: Quota badges, DAE color bands, seed certification tags                 |
|  - Counter POS: Quota validation, split-tender (Cash, MFS, Khata), seasonal harvest picker        |
|  - Customer & Mohajon Khata views with harvest-cycle filtering                                    |
|                                                                                                   |
|  [MILESTONE 4: AUTHENTIC HUMAN UI POLISH & SHAREABLE RECEIPTS]                                    |
|  - Zero generic AI slop: Remove wand/sparkle icons, ensure 44px touch targets                     |
|  - Resolve all ESLint violations in apps/saas components                                          |
|  - Modal for WhatsApp (wa.me) & SMS (sms:) formatted receipts with government subsidy display     |
|                                                                                                   |
|  [MILESTONE 5: VERIFICATION & AUDIT]                                                              |
|  - Full workspace build: pnpm --filter saas build, pnpm --filter api build                        |
|  - Automated test runs: pnpm --filter api test                                                    |
|  - Independent multi-agent gate verification & Forensic Integrity Audit                           |
+---------------------------------------------------------------------------------------------------+
```

#### Detailed Milestone Tasks

* **Milestone 1 — SaaS Benchmarking & Feature Research Document (Current):**
  - Author comprehensive research document `docs/saas-benchmarking-and-feature-matrix.md` synthesizing global benchmarks with Bangladeshi agricultural retail realities.
* **Milestone 2 — Shared Contracts, Database Migration & Backend Fastify APIs:**
  - Execute PostgreSQL migration adding enums, licenses, fertilizer quotas, pesticide specs, seed specs, customer seasonal ledgers, and mohajon credit tables.
  - Implement Zod contracts in `packages/contracts/src/` (`fertilizer.ts`, `pesticides.ts`, `seeds.ts`, `khata.ts`).
  - Implement Fastify route plugins in `apps/api/src/modules/` with request tracing and error envelopes.
  - Add `"test": "vitest run"` in `apps/api/package.json` and author comprehensive test suites verifying API responses, quota checks, and seed viability logic.
* **Milestone 3 — Frontend Agricultural Domain Modules & POS Enhancements:**
  - Expand `apps/saas` TypeScript interfaces and mock/live data integration.
  - Build POS counter controls: fertilizer quota deduction, Krishok Card/NID validation, DAE hazard band indicator badges, and split-tender payment modal.
  - Implement Customer Khata and Mohajon Khata dashboards supporting seasonal harvest scheduling.
* **Milestone 4 — Authentic UI Polish, ESLint Clean-up & Shareable Receipts:**
  - Audit and fix all 8 ESLint violations in `apps/saas`.
  - Perform visual de-slopping: purge AI sparkles/magic wands; apply authentic agricultural domain cues and 44px minimum touch targets.
  - Integrate WhatsApp (`wa.me`) and SMS (`sms:`) bilingual shareable receipt generator.
* **Milestone 5 — End-to-End Verification & Forensic Integrity Audit:**
  - Execute comprehensive builds across `apps/saas` and `apps/api`.
  - Pass all automated unit and integration tests.
  - Undergo independent adversarial review and forensic verification.

---

## 7. Conclusion

By systematically adapting the agronomic safety guardrails of **Agworld**, the grower statement and settlement clarity of **Bushel**, the laboratory quality grading of **AgriDigital**, the seasonal crop-cycle ledger accounting of **ERPNext**, and the rapid counter ergonomics of **Clover and Square**, Krishi Sheba establishes a resilient, localized operational foundation. 

This platform transforms the Bangladeshi agricultural input retailer from a vulnerable, paper-dependent merchant into a compliant, efficient, and technologically empowered hub for rural food security.
