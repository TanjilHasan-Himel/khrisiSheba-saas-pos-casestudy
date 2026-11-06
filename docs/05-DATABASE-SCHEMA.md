# 05 — Backend Database Schema (PostgreSQL, logical design)

## 1. Conventions

UUID primary keys, `created_at`, `updated_at`, `tenant_id` on all business tables, `store_id` where operational. Monetary values: `BIGINT` paisa (BDT minor units) with `CHECK >= 0` where appropriate; quantities: `NUMERIC(18,3)` or more precision after units review. Store all timestamps UTC. Prefer constrained enums/checks and explicit status transitions. Soft deletion only for appropriate master data; financial records are immutable/reversed.

## 2. Identity and tenancy

```text
users(id, email, phone, display_name, status, created_at)
tenants(id, legal_name, display_name, status, timezone, currency, created_at)
tenant_memberships(id, tenant_id, user_id, role_id, status, invited_by)
roles(id, tenant_id NULL for system template, code, name)
permissions(id, code)
role_permissions(role_id, permission_id)
stores(id, tenant_id, code, name, address, status)
store_memberships(id, tenant_id, store_id, user_id, role_id, status)
```

Unique active membership per `(tenant_id,user_id)`; store membership must reference same tenant. Roles must not grant access outside tenant context.

## 3. Catalog and procurement

```text
categories(id, tenant_id, parent_id, name_bn, name_en)
units(id, tenant_id, code, name_bn, precision)
products(id, tenant_id, sku, category_id, base_unit_id, name_bn, name_en,
         barcode, sale_price_minor, status, metadata_json)
product_batches(id, tenant_id, product_id, lot_code, manufacture_date,
                expiry_date, status, registration_metadata_json)
suppliers(id, tenant_id, name, phone, address, status)
purchases(id, tenant_id, store_id, supplier_id, purchase_no, status,
          ordered_at, posted_at, total_minor, created_by)
purchase_items(id, tenant_id, purchase_id, product_id, ordered_qty,
               unit_cost_minor, received_qty)
goods_receipts(id, tenant_id, store_id, purchase_id, receipt_no, posted_at)
goods_receipt_items(id, tenant_id, receipt_id, purchase_item_id,
                    batch_id, received_qty, unit_cost_minor, disposition)
```

Purchase cost and sale price are distinct. Batch `lot_code` uniqueness depends on product + supplier conventions, not global.

## 4. Inventory

```text
inventory_balances(id, tenant_id, store_id, product_id, batch_id,
                   on_hand_qty, reserved_qty, version)
inventory_movements(id, tenant_id, store_id, product_id, batch_id,
                    delta_qty, movement_type, source_type, source_id,
                    posted_at, actor_id)
inventory_adjustments(id, tenant_id, store_id, reason, status, approved_by)
```

Balance unique `(tenant_id,store_id,product_id,batch_id)` with non-null batch strategy (use sentinel batch or NULLS NOT DISTINCT PostgreSQL 15+). Movements are append-only; balance is transactional materialization. Stock availability = on_hand - reserved - blocked quantities as defined by policy.

## 5. Sales and finance

```text
customers(id, tenant_id, display_name, phone, address, status)
sales(id, tenant_id, store_id, sale_no, customer_id NULL, source,
      status, subtotal_minor, discount_minor, total_minor, posted_at, created_by)
sale_items(id, tenant_id, sale_id, product_id, batch_id, quantity,
           unit_price_minor, discount_minor, line_total_minor)
payments(id, tenant_id, store_id, sale_id NULL, customer_id NULL,
         method, amount_minor, reference, status, posted_at)
customer_ledger_entries(id, tenant_id, customer_id, sale_id NULL,
                        payment_id NULL, return_id NULL, amount_delta_minor,
                        entry_type, posted_at)
sale_returns(id, tenant_id, store_id, sale_id, return_no, status,
             refund_total_minor, posted_at)
sale_return_items(id, tenant_id, return_id, sale_item_id, quantity,
                  disposition, refund_minor)
expenses(id, tenant_id, store_id, category, amount_minor, occurred_at, notes)
```

Ledger convention: sale due = positive debit, payment = negative credit, refund/return adjustment signed by documented rule. Enforce posting with domain service and audit. `source` enum: MANUAL, KRISHI_POS, EXTERNAL_POS, IMPORT.

## 6. Platform, integration and audit

```text
plans(id, code, name, status)
subscriptions(id, tenant_id, plan_id, status, started_at, ends_at)
platform_audit_events(id, actor_id, action, tenant_id NULL, timestamp, metadata)
audit_events(id, tenant_id, actor_id, entity_type, entity_id, action,
             timestamp, before_json, after_json)
idempotency_records(id, tenant_id, operation, key, payload_hash,
                    response_code, response_body, created_at)
outbox_events(id, tenant_id, topic, payload_json, status, created_at)
# future
integration_connections(id, tenant_id, provider, status, encrypted_secret_ref)
external_record_mappings(id, tenant_id, provider, external_id, internal_id, type)
sync_runs(id, tenant_id, connection_id, status, cursor, started_at)
pos_devices(id, tenant_id, store_id, device_code, status, last_seen_at)
```

## 7. Referential integrity

Use composite unique keys `(tenant_id,id)` on tenant-scoped parents and composite foreign keys `(tenant_id,parent_id)` on children. Add `(tenant_id,store_id)` composite FKs where relevant. Global `users` and `plans` are exceptions; access still requires membership. Unique `(tenant_id,sku)` for active product SKUs, `(tenant_id,store_id,sale_no)` for sales, `(tenant_id,store_id,purchase_no)` for purchases. Index tenant/store/date/report filters. Enforce RLS as defense in depth.

## 8. Critical SQL transaction pseudocode

```sql
BEGIN;
-- server already authenticated actor and verified tenant/store membership
-- set tenant context transaction-locally, then acquire required row locks
SELECT ... FROM inventory_balances
 WHERE tenant_id = :tenant AND store_id = :store AND product_id = :product
 FOR UPDATE;
-- reject insufficient stock or blocked/expired batch
INSERT INTO sales (...);
INSERT INTO sale_items (...);
INSERT INTO inventory_movements (..., delta_qty) VALUES (..., -:qty);
UPDATE inventory_balances SET on_hand_qty = on_hand_qty - :qty, version = version + 1
 WHERE ... AND on_hand_qty - reserved_qty >= :qty;
-- check affected row count; write payments/ledger/audit/idempotency/outbox
COMMIT;
```

This is a **design sketch, not executable migration**. Real migrations must enforce cross-tenant constraints and test concurrency with actual PostgreSQL.

## 9. Unresolved modeling choices

Base-unit conversion (kg/g, litre/ml, packet), pricing/tax treatment, inventory costing FIFO vs weighted average, credit allocation across invoices, partial purchase returns, expiry policy, pesticide regulatory identifiers, and whether tenant needs wholesale/customer tiers.
