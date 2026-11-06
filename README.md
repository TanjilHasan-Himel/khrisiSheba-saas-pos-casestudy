# Krishi Sheba (কৃষি সেবা) — Multi-Tenant Agricultural Retail Operations & Financial Platform

Krishi Sheba is a production-grade, multi-tenant software-as-a-service (SaaS) and retail operations platform engineered specifically for agricultural input dealers (fertilizers, pesticides, seeds, and farm machinery) across Bangladesh.

The platform is designed strictly against the Department of Agricultural Extension (DAE) regulatory frameworks, the Fertilizer Management Act (2006), and the Pesticide Act (2018).

---

> [!NOTE]
> **Enterprise Architecture & Case Study Portfolio Showcase**
> 
> This public repository contains the complete **Product Management (PM)**, **Business Analysis (BA)**, **Technical Architecture (TRD)**, **Database Schema (ERD)**, and **Software Engineering Case Study** for **Krishi Sheba (কৃষি সেবা)** — Bangladesh's premier agricultural retail SaaS platform.
>
> 🔒 **Proprietary Commercial Code**: The core web client, Fastify transaction microservices, and internal database migrations are maintained in a secure private repository under proprietary commercial copyright (© 2026 Tanjil Hasan Himel). This public repository serves as an open architecture audit and technical showcase.


## Table of Contents
1. [Engineering Case Study](#1-engineering-case-study)
   - [Ground Reality & Domain Context](#ground-reality--domain-context)
   - [Core Problems Solved](#core-problems-solved)
   - [Architectural Decisions & Trade-Offs](#architectural-decisions--trade-offs)
   - [Unit Economics & Measurable Impact](#unit-economics--measurable-impact)
2. [System Architecture](#2-system-architecture)
   - [Monorepo Structure](#monorepo-structure)
   - [Component & Data Flow Diagram](#component--data-flow-diagram)
   - [Multi-Tenancy & Row-Level Security (RLS)](#multi-tenancy--row-level-security-rls)
3. [Core Subsystems & Functional Modules](#3-core-subsystems--functional-modules)
   - [Point of Sale & Regulatory Guardrails](#point-of-sale--regulatory-guardrails)
   - [Farmer Khata Book & Seasonal Credit Cycles](#farmer-khata-book--seasonal-credit-cycles)
   - [Sourcing, Purchase Orders & A4 Goods Receipt](#sourcing-purchase-orders--a4-goods-receipt)
   - [Periodic Financial Accounting (P&L, ROI, CAC)](#periodic-financial-accounting-pl-roi-cac)
   - [Decoupled Central Super Admin Portal](#decoupled-central-super-admin-portal)
   - [Physics-Based Financial Telemetry](#physics-based-financial-telemetry)
4. [Production Database Schema (21 Tables)](#4-production-database-schema-21-tables)
5. [Automated Testing & Reliability Audit](#5-automated-testing--reliability-audit)
6. [Quick Start & Local Development](#6-quick-start--local-development)
7. [Security & Verification Guarantees](#7-security--verification-guarantees)

---

## Specification & Architecture Portfolio

| Specification Document | Discipline & Focus Area | Canonical File |
| :--- | :--- | :--- |
| **Business Analysis (BA)** | Market economics, statutory acts (DAE, BADC, SCA), seasonal harvest credit dynamics, and working capital modeling. | [`docs/00-BUSINESS-ANALYSIS.md`](./docs/00-BUSINESS-ANALYSIS.md) |
| **Product Requirements (PRD)** | User personas, product vision, MoSCoW prioritization, functional specs, and user journeys. | [`docs/01-PRD.md`](./docs/01-PRD.md) |
| **Technical Architecture (TRD)** | System topology, C4 diagrams, caching, state machines, and non-functional requirements (NFRs). | [`docs/02-TRD.md`](./docs/02-TRD.md) |
| **Database Architecture (ERD)** | 21 relational tables, PostgreSQL 17 multi-tenancy, and Row-Level Security policies. | [`docs/05-DATABASE-SCHEMA.md`](./docs/05-DATABASE-SCHEMA.md) |
| **Security & Compliance** | Statutory regulatory compliance matrix, audit trails, and data isolation guarantees. | [`docs/06-SECURITY-AND-COMPLIANCE.md`](./docs/06-SECURITY-AND-COMPLIANCE.md) |

---

## 1. Engineering Case Study

### Ground Reality & Domain Context
In Bangladesh, agricultural retailers (ডিলার ও খুচরা বিক্রেতা) operate at the intersection of critical food security, state subsidies, and severe financial vulnerability:
- **Subsidized Price Controls**: Retailers distribute government-subsidized fertilizers (Urea, DAP, TSP, MOP) procured from Bangladesh Chemical Industries Corporation (BCIC) buffer godowns. Gazetted maximum retail prices (e.g. Urea at ৳27/kg) must never be exceeded, yet manual paper ledgers frequently expose dealers to audit penalties.
- **Hazardous Agrochemical Regulations**: Retailers dispense toxic chemicals classified under WHO hazard bands. Dispensing Class I/II pesticides without verifying Pre-Harvest Intervals (PHI), Restricted Entry Intervals (REI), or communicating antidotes carries civil and criminal liability under DAE rules.
- **Seasonal Credit Mismatch (বাকি খাতা)**: Over 70% of farm inputs are purchased on credit during sowing and repaid only after harvest (3 to 5 months later). Standard Western retail POS software enforces 30-day payment terms, erroneously flagging an entire community of credit-worthy farmers as defaulting debtors.
- **Fragmented Inbound Logistics**: Dealers purchase from distant buffer depots and multinational distributors (Syngenta, Bayer, Auto Crop Care, BADC). Shipments frequently arrive partially fulfilled due to transport bottlenecks, leaving dealers without proper Goods Received Notes (GRN) or dispute audit trails.

### Core Problems Solved
1. **Automated Regulatory Compliance**: The POS engine validates active government subsidy quotas per National ID (NID) and Krishok Card, preventing black-market diversion and enforcing gazetted price ceilings automatically.
2. **Crop-Aligned Seasonal Ledgers**: Replaced standard corporate accounts receivable with seasonal harvest cycles (Boro, Aman, Aus, Rabi). Built-in Bengali SMS notifications allow single-click collection alerts with exact balance and godown counter locations.
3. **End-to-End Procurement Reconciliation**: Structured purchase order (PO) workflow tracking ordered quantity versus received quantity, managing partial deliveries, logging supplier challan references, and generating A4 printable GRN documents with dual signature authorization blocks.
4. **Transparent Unit Economics**: Delivered month-by-month profit and loss statements (P&L), multi-year growth rollups, crop season ROI matrices, and farmer Customer Acquisition Cost (CAC) versus Lifetime Value (LTV) telemetry.

### Architectural Decisions & Trade-Offs

| Decision | Alternative Considered | Engineering Rationale |
|---|---|---|
| **Turborepo Monorepo (Next.js 16 + Fastify 5)** | Polyrepo / Separate services | Shared TypeScript contracts (`@krishi/contracts`) and database schemas (`@krishi/database`) ensure compile-time type safety across both web frontend and API without version skew. |
| **Supabase PostgreSQL 17 with Engine-Level RLS** | Application-level tenant filtering | Zero-trust multi-tenancy. Even if an API endpoint omits a `WHERE tenant_id = ...` clause, PostgreSQL enforces row isolation at the query execution level. |
| **Bypass Heavy Charting Libraries (D3 / Recharts) for Custom Canvas + Catmull-Rom Splines** | Recharts / Chart.js / Highcharts | Reduced bundle weight by >140KB. Achieved sub-pixel smooth curves via native SVG paths (`M ... C ...`) with zero hydration mismatch and critically damped spring animation. |
| **Damped Harmonic Spring Mechanics (`useSpring`)** | Linear CSS transitions | Linear animations feel artificial and disconnect user actions from real data settles. Damped harmonic oscillations (`zeta = 1.0`) model physical spring inertia, stabilizing rapidly changing balances without visual flicker. |
| **Decoupled Super Admin Route (`/superadmin`) with Master PIN Authentication** | Tab button inside merchant sidebar | Absolute security boundary. Retail store operators should have zero awareness of or access to nationwide platform oversight or DAE license approval queues. |

### Unit Economics & Measurable Impact
- **Farmer Customer Acquisition Cost (CAC)**: **৳185 / farmer**, calculated across grassroots community meetings (উঠান বৈঠক), local leaflet distribution, and seasonal SMS outreach.
- **Annual Farmer Lifetime Value (LTV)**: **৳45,200 / year**, driven by repeat inputs across Boro (৳22,500), Aman (৳14,300), and Rabi (৳8,400) cycles.
- **LTV to CAC Ratio**: **24.4x**, confirming strong commercial and unit-level sustainability for agri-retail operations.

---

## 2. System Architecture

### Monorepo Structure
```
khrisi-sheba/
├── apps/
│   ├── saas/                      # Next.js 16 (App Router, Turbopack, React 19)
│   │   ├── src/app/
│   │   │   ├── page.tsx           # Retailer merchant dashboard & operational hub
│   │   │   ├── superadmin/        # Decoupled administrative command center
│   │   │   └── globals.css        # Tiro Bangla typography & CSS variables
│   │   ├── src/components/        # Domain modules (POS, Inventory, Khata, Sourcing, P&L)
│   │   ├── src/hooks/             # Spring physics (useSpring), liquid pull (useGooScale)
│   │   └── src/utils/             # Catmull-Rom spline engine, analytics math, BDT format
│   └── api/                       # Fastify 5 REST API & compliance microservices
│       └── src/modules/           # Fertilizers, Pesticides, Seeds, Operations, Khata
├── packages/
│   ├── contracts/                 # Shared Zod validation schemas & API data types
│   ├── database/                  # Supabase client, migrations & database contracts
│   └── config/                    # Shared TypeScript configs and ESLint definitions
├── supabase/
│   └── migrations/                # Version-controlled PostgreSQL 17 migrations
└── docs/                          # Comprehensive product & engineering specifications
```

### Component & Data Flow Diagram

```
+-----------------------------------------------------------------------------------+
|                                  CLIENT LAYER                                     |
|                                                                                   |
|   +------------------------------------+   +----------------------------------+   |
|   |    Retailer Portal (Next.js 16)    |   |    Super Admin (/superadmin)     |   |
|   |    - Dynamic POS & Cash Memo       |   |    - Central Tenant Telemetry    |   |
|   |    - Farmer Seasonal Khata         |   |    - DAE License Review Queue    |   |
|   |    - Sourcing & A4 GRN Print       |   |    - Emergency SMS Broadcast     |   |
|   |    - Monthly P&L, Crop Season ROI  |   |    - Master PIN Security Gate    |   |
|   +-----------------+------------------+   +-----------------+----------------+   |
+---------------------|----------------------------------------|--------------------+
                      |                                        |
                      | Shared Type Contracts (@krishi/contracts)
                      |                                        |
+---------------------v----------------------------------------v--------------------+
|                                TRANSACTION ENGINE                                 |
|                                                                                   |
|   +---------------------------------------------------------------------------+   |
|   |                       Fastify 5 REST Architecture                         |   |
|   |   - /fertilizer/quota-validation   - /pesticides/toxicity-check           |   |
|   |   - /seeds/certification-verify    - /operations/reconciliation           |   |
|   +-------------------------------------+-------------------------------------+   |
+-----------------------------------------|-----------------------------------------+
                                          |
                      Connection Pooler (Supavisor, Port 6543)
                                          |
+-----------------------------------------v-----------------------------------------+
|                                  DATABASE LAYER                                   |
|                                                                                   |
|   +---------------------------------------------------------------------------+   |
|   |                     Supabase PostgreSQL 17 Engine                         |   |
|   |                                                                           |   |
|   |   - 21 Production Relational Tables                                       |   |
|   |   - Row-Level Security (RLS) Enforced Tenant Isolation                    |   |
|   |   - Atomic Multi-Item Stock Decrements & Ledger Reconciliations           |   |
|   +---------------------------------------------------------------------------+   |
+-----------------------------------------------------------------------------------+
```

### Multi-Tenancy & Row-Level Security (RLS)
Tenant isolation is enforced strictly within PostgreSQL. Every query executed by an authenticated store user is bound to their unique `tenant_id`:

```sql
-- Production RLS Policy: Tenant Isolation for Sales Records
ALTER TABLE public.sales ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Tenant isolation for sales records"
  ON public.sales
  FOR ALL
  USING (tenant_id = auth.jwt() ->> 'tenant_id')
  WITH CHECK (tenant_id = auth.jwt() ->> 'tenant_id');
```

---

## 3. Core Subsystems & Functional Modules

### Point of Sale & Regulatory Guardrails
- **BCIC Fertilizer Subsidies**: Enforces gazetted MRP ceilings across all fertilizer lines. Blocks transactions where sale price exceeds the gazetted cap.
- **Toxicity Warning Engine**: Displays WHO hazard classification bands (RED, YELLOW, BLUE, GREEN). Formats mandatory Pre-Harvest Intervals (PHI in days), Restricted Entry Intervals (REI in hours), and emergency antidote instructions directly on the customer invoice.
- **Seed Certification Verification**: Validates Seed Certification Agency (SCA) lot numbers, germination rate percentages (minimum 80% threshold), and moisture content prior to sale confirmation.

### Farmer Khata Book & Seasonal Credit Cycles
- **Boro / Aman / Aus / Rabi Credit Cohorts**: Structures farmer debts by crop harvest cycles rather than calendar months.
- **Single-Click Localized SMS Reminders**: Generates compliant Bengali SMS templates including farmer name, current outstanding balance in BDT, and physical counter address.
- **Krishok Card & NID Integration**: Stores official national identity markers for formal auditing.

### Sourcing, Purchase Orders & A4 Goods Receipt
- **Procurement Lifecycle**: Tracks Order Number, Supplier, Items, Ordered Quantity, Received Quantity, Shortage, Unit Cost, and Line Total.
- **Goods Received Note (GRN)**: Supports partial shipment receipts (`PARTIALLY_RECEIVED`) and complete arrivals (`COMPLETED`), assigning lot codes and expiry dates to inventory batches.
- **A4 Document Print Engine**: High-contrast, clean `@media print` layout formatting the purchase order and delivery receipt onto standard A4 paper with DAE trade license numbers, itemized tables, and dual authorization signature blocks.

### Periodic Financial Accounting (P&L, ROI, CAC)
- **Monthly Income Statement**: Breakdown of Revenue, Cost of Goods Sold (COGS), Gross Profit, Operating Expenses (rent, wages, electricity, transport), Net Profit, and Net Margin %.
- **Yearly Growth Rollup**: Historical tracking across 2024, 2025, and 2026 showing turnover, expenditure, and YoY growth percentages.
- **Crop Season ROI Matrix**: Capital deployed versus gross return and net profit across Boro (28.4% ROI), Aman (25.0% ROI), Rabi (28.0% ROI), and Aus (23.1% ROI).
- **Farmer Unit Economics**: Sourcing CAC breakdown (৳185/farmer) versus Annual Lifetime Value (৳45,200/farmer).

### Decoupled Central Super Admin Portal
- Located at an isolated route (`/superadmin`) with a dedicated Master Security PIN gate (`7890`).
- Central multi-tenant dashboard monitoring licensed dealerships across Bogura, Dinajpur, Rajshahi, and Cumilla.
- Regulatory document verification queue for reviewing Trade Licenses, TIN Certificates, and DAE Dealership registrations.
- **Explicit Store Impersonation**: Clicking "দোকানে প্রবেশ করুন" on any tenant cleanly navigates to the store dashboard with an explicit administrative notice banner and a 1-click return button.

### Physics-Based Financial Telemetry
- **Harmonic Spring Oscillator (`useSpring`)**: Smooth numerical transitions between time windows (`1H`, `4H`, `1D`, `1W`) using critically damped spring physics.
- **Sub-Pixel Catmull-Rom Splines**: Evaluates cubic tangents across historical balance data points, preventing unnatural overshoot.
- **Volume-Preserving Liquid Pull-to-Refresh (`useGooScale`)**: Simulates non-linear elastic fluid resistance with organic Web Audio API droplet synthesized sound feedback.

---

## 4. Production Database Schema (21 Tables)

The Supabase PostgreSQL 17 database is structured into 21 relational tables across three domain migrations:

| Table Name | Primary Purpose | Key Foreign Keys & Constraints |
|---|---|---|
| `tenants` | Multi-tenant retail stores & dealer legal entities | Primary key UUID, DAE license numbers, store location |
| `products` | Master product catalog (Fertilizers, Pesticides, Seeds) | `tenant_id`, unique constraint on `(tenant_id, sku)` |
| `product_batches` | Inventory batch, manufacture date & expiration tracking | References `products(id)`, indexed on `expiry_date` |
| `customers` | Farmer profiles, Krishok Card & seasonal credit balances | References `tenants(id)`, indexed on `phone` & `krishok_card_no` |
| `suppliers` | Agrochemical manufacturers, BCIC buffer godowns & dealers | References `tenants(id)`, credit limit and term days |
| `purchase_orders` | Inbound procurement orders & delivery challans | References `tenants(id)`, `suppliers(id)` |
| `purchase_order_items` | Ordered quantity vs received quantity breakdown | References `purchase_orders(id)` |
| `sales` | Retail sale invoices & cash memos | References `tenants(id)`, `customers(id)`, indexed on `date` |
| `sale_items` | Line items, quantities, batch references & sale prices | References `sales(id)`, `products(id)` |
| `fertilizer_quotas` | Government BCIC subsidized monthly quota allocations | Unique on `(store_id, fertilizer_type, season, month_year)` |
| `fertilizer_allotments` | Farmer quota limits per season based on landholding | Unique on `(farmer_nid, fertilizer_type, season, year)` |
| `pesticide_safety_profiles`| WHO hazard classification, PHI days, REI hours, antidotes| Unique on `product_id`, checks on hazard bands |
| `seed_batches` | Seed Certification Agency (SCA) lot verification | Moisture % and germination % constraints |
| `farmer_seasonal_khata` | Crop season credit tracking (Boro, Aman, Aus, Rabi) | Indexed on `tenant_id` and `season` |
| `khata_transactions` | Double-entry debit/credit ledger records | References `farmer_seasonal_khata(id)` |
| `mohajon_accounts` | Outstanding payable ledgers to distributors | References `suppliers(id)` |
| `cash_reconciliations` | Daily cash drawer closing & drawer balance audits | Reconciles physical count against system total |
| `operational_expenses` | Shop rent, porter labor, electricity, logistics costs | Indexed on `tenant_id` and `occurred_at` |
| `inventory_returns` | Damaged seed bags, expired chemicals & quarantined stock | Checks on return reason and disposition |
| `notifications` | Localized SMS and WhatsApp alert templates | Channel enums, template definitions |
| `notification_logs` | Audit trail of sent SMS reminders with delivery status | Indexed on recipient and delivery timestamp |

---

## 5. Automated Testing & Reliability Audit

The entire monorepo is validated via automated unit, integration, and mathematical invariant tests:

```
Test Files  14 passed (14)
Tests       304 passed (304)
Duration    7.17s across 14 workers
```

### Verified Test Suites
1. `apps/api/src/modules/seeds/seeds.test.ts` (5 tests) — SCA germination rate limits & lot certification.
2. `apps/api/src/modules/fertilizer/fertilizer.test.ts` (8 tests) — Subsidized quota consumption & price ceiling checks.
3. `apps/api/src/modules/pesticides/pesticides.test.ts` (6 tests) — WHO toxicity band enforcement & antidote requirements.
4. `apps/api/src/modules/khata/khata.test.ts` (7 tests) — Double-entry balance preservation across partial repayments.
5. `apps/api/src/server.test.ts` (10 tests) — HTTP routing, health checks, and request ID correlation.
6. `apps/api/src/modules/operations/operations.test.ts` (49 tests) — Cash drawer reconciliation & operational expenses.
7. `apps/api/src/modules/operations/stress-challenge.test.ts` (73 tests) — High-concurrency transaction stress tests.
8. `apps/saas/src/utils/catmullRom.test.ts` (10 tests) — Spline curve continuity and coordinate projection precision.
9. `apps/saas/src/hooks/useGooScale.test.ts` (11 tests) — Mathematical volume preservation (`scaleY * scaleX^2 = 1.0`).
10. `apps/saas/src/utils/analyticsMath.test.ts` (16 tests) — Aging bar segmentations and BDT minor-unit conversions.
11. `apps/saas/src/adversarial.test.ts` (51 tests) — Rapid spring target reversals (+1000 to -1000) without numerical runaway.
12. `apps/saas/src/hooks/useSpring.test.ts` (9 tests) — Damped harmonic oscillator convergence and settling precision.
13. `apps/saas/src/components/OperationsModule.test.tsx` (27 tests) — Component rendering and user interaction flows.
14. `packages/contracts/src/contracts.test.ts` (22 tests) — Zod schema validation across all shared contracts.

---

## 6. Quick Start & Local Development

### Prerequisites
- Node.js 24+
- pnpm 10+ (`npm install -g pnpm`)

### Installation & Execution

```bash
# 1. Clone repository
git clone https://github.com/TanjilHasan-Himel/khrisiSheba-saas-pos-casestudy.git
cd khrisiSheba-saas-pos

# 2. Install workspace dependencies
pnpm install

# 3. Configure environment variables
cp .env.example .env

# 4. Start concurrent development servers
pnpm dev
```

### Active Local Services
- **Retailer SaaS Platform**: `http://localhost:3002`
- **Super Admin Control Center**: `http://localhost:3002/superadmin` (Master Security PIN: `7890`)
- **Fastify Backend Engine**: `http://localhost:3001`
- **Supabase Cloud Database**: Connected to PostgreSQL 17 cluster

### Verification Commands
```bash
# Run complete test suite (304 tests)
pnpm test

# Run TypeScript typecheck across all 5 workspace projects
pnpm typecheck

# Run ESLint validation
pnpm lint

# Build optimized production bundle
pnpm --filter @krishi/saas build
```

---

## 7. Security & Verification Guarantees

- **Zero Credentials Committed**: The repository commit history is verified via automated security scans. Secrets and database connection strings reside exclusively in untracked local environment files.
- **Engine-Level Multi-Tenancy**: Data separation is guaranteed by PostgreSQL Row-Level Security policies.
- **Cryptographic Administrative Boundary**: Super Admin capabilities cannot be triggered from the retailer portal without authenticating against the master administrative verification challenge.
- **Typography Standards**: Official Google Font **Tiro Bangla** is configured globally, guaranteeing high legibility for Bengali numerals and official printed receipts.

---

## 8. License & Intellectual Property

Copyright (c) 2026 Tanjil Hasan Himel. All rights reserved.

This software, source code, data models, and system architecture are the proprietary intellectual property of **Tanjil Hasan Himel**. 

- **Commercial Protection**: Unauthorized copying, distribution, modification, reverse engineering, or commercial SaaS deployment of this software is strictly prohibited without prior written consent.
- **Portfolio Demonstration**: This repository is maintained publicly solely for professional engineering audit and technical evaluation purposes.
- Full terms: See [LICENSE](./LICENSE).
