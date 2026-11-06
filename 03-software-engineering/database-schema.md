# 03.3 — Database Schema & Data Models
**Document:** PostgreSQL 17 Relational Data Models (21 Tables)  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Canonical Public Portfolio Specification  

---

## 1. Schema Overview

The database is built on PostgreSQL 17, organized across 21 relational tables with strict foreign key constraints, composite unique indexes, and Row-Level Security (RLS) policies:

| Table Name | Primary Purpose | Key Foreign Keys & Constraints |
| :--- | :--- | :--- |
| `tenants` | Multi-tenant retail stores & dealer legal entities | UUID PK, DAE license numbers, store location |
| `products` | Master product catalog (Fertilizers, Pesticides, Seeds) | `tenant_id`, unique on `(tenant_id, sku)` |
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

## 2. Invariant & Integrity Rules

1. **Integer Minor Units:** All monetary values are stored as 64-bit integers (`BIGINT` in BDT paisa). Decimal amounts are converted strictly at the presentation layer.
2. **Double-Entry Equality:** Every sale invoice atomically creates ledger debits and credits:
   $$\text{Cash Drawer Inflow} + \text{Khata Debit} = \text{Invoice Line Items Total}$$
3. **Immutable Posting:** Financial transactions, sales invoices, and stock movements are append-only. Cancellations require explicit reversal transactions (`REVERSAL`).

