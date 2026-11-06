# Krishi Sheba (কৃষি সেবা) — Architecture & Technical Specifications

## 1. System Overview

Krishi Sheba is built as a modular monorepo leveraging TypeScript throughout the stack. The platform decouples dealer shop operations from central multi-tenant administration while maintaining zero-trust database security policies.

```
                  ┌──────────────────────────────────────────────────┐
                  │                 Clients / Edge                   │
                  └──────────────┬───────────────────┬───────────────┘
                                 │                   │
                  ┌──────────────▼───────┐    ┌──────▼──────────────┐
                  │   Retailer Shop      │    │ Super Admin Portal  │
                  │   Next.js 16 (/)     │    │  (/superadmin)      │
                  └──────────────┬───────┘    └──────┬──────────────┘
                                 │                   │
                                 │      Zod Schema   │
                                 │      Contracts    │
                                 │                   │
                  ┌──────────────▼───────────────────▼──────────────┐
                  │          Fastify 5 Modular REST Engine          │
                  │        (/fertilizer, /pesticide, /seeds)        │
                  └──────────────────────┬──────────────────────────┘
                                         │
                  ┌──────────────────────▼──────────────────────────┐
                  │   Supabase PostgreSQL 17 + Row Level Security   │
                  │         (Tenant-Scoped Data Isolation)          │
                  └─────────────────────────────────────────────────┘
```

---

## 2. Multi-Tenancy & Data Isolation

Data isolation is guaranteed at the database engine level via PostgreSQL Row Level Security (RLS):

- **Tenant Identifier**: Each transaction, inventory item, customer, and purchase order includes a `tenant_id` UUID foreign key referencing the registered shop.
- **RLS Policy Enforcement**:
  ```sql
  CREATE POLICY "Tenant isolation for sales"
    ON public.sales
    FOR ALL
    USING (tenant_id = auth.jwt() ->> 'tenant_id')
    WITH CHECK (tenant_id = auth.jwt() ->> 'tenant_id');
  ```
- **Super Admin Isolation**: The `/superadmin` portal accesses tenant telemetry through dedicated administrative service roles authenticated via a cryptographic PIN/OAuth challenge, ensuring merchants cannot access central telemetry.

---

## 3. UI/UX Engineering & Design System

- **Typography**: Official **Tiro Bangla** Google Font ensures authentic Bengali glyph rendering for numbers, bills, and regulatory documents.
- **Fixed Sticky Navigation**: `<aside className="krishi-sidebar">` utilizes `position: sticky; top: 0; height: 100dvh; overflow-y: auto; flex-shrink: 0;` ensuring independent document scrolling without displacement.
- **Spring Physics**: Dynamic balance displays feature critically damped harmonic oscillation (`zeta = 1.0`) preventing visual jar during rapid telemetry refreshes.
- **A4 Document Print Engine**: Dedicated CSS print styles (`@media print`) format purchase orders, delivery challans, and cash memos precisely to A4 paper bounds without UI chrome.

---

## 4. Regulatory Data Models

### A. Fertilizers
- Subsidized quota tracking (`isSubsidized = true`)
- MRP ceiling enforcement (`gazettedMrpPerKg`)
- BCIC Buffer Godown Challan Reconciliation

### B. Pesticides
- WHO Hazard Classification: RED, YELLOW, BLUE, GREEN
- Pre-Harvest Interval (`phiDays`)
- Restricted Entry Interval (`reiHours`)
- Emergency Medical First Aid & Antidote Instructions

### C. Seeds
- Seed Certification Agency (SCA) Certificate Numbers
- Seed Class: Foundation, Certified, Hybrid
- Germination Rate % & Moisture Content % Verification
