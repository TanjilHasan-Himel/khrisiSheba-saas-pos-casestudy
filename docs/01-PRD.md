# 01 — Product Requirements Document (PRD)

## 1. Product thesis

Krishi Sheba is a **multi-tenant agricultural retail operations SaaS** for Bangladesh. Its first paid-use hypothesis: small agricultural retailers will pay for a dependable way to manage purchases, stock, batch/expiry, sales recording, customer dues, and business visibility, including on low-end Android phones.

**Not yet validated:** willingness to pay, exact target segment, pricing, device mix, network quality, and regulatory obligations.

## 2. Primary personas and jobs

| Persona           | Primary job                                                               | Access                              |
| ----------------- | ------------------------------------------------------------------------- | ----------------------------------- |
| Tenant owner      | Know stock, cash, dues, purchasing and branch performance                 | Full business access                |
| Store manager     | Manage purchases, inventory, customers, sales and staff                   | Scoped store access                 |
| Sales clerk       | Record manual sales and receive due payments                              | Limited sales/customer access       |
| Stock clerk       | Receive goods, count stock, report damage/expiry                          | Inventory-only                      |
| Platform operator | Manage tenant lifecycle and support without routine access to tenant data | Platform-specific restricted access |

## 3. Release scope

### P0 — SaaS Core (first usable release)

- Account registration/invitation, email/phone verification approach to be selected, login/logout/session control.
- Create tenant, business profile, first store; invite staff; store-scoped roles.
- Product catalog: SKU, Bangla/English name, category, base unit, sale price, active status; optional barcode and image.
- Category-aware attributes (e.g., fertilizer pack weight, seed variety, pesticide registration metadata where applicable), not a hard-coded single product shape.
- Suppliers, purchase drafts, receive/confirm goods, batch/lot, manufacture/expiry dates where relevant.
- Stock movements with immutable history, available quantity and low-stock/near-expiry alerts.
- Customers, optional phone, credit limits/approval policy, ledger and collection receipts.
- Manual sales entry (not a full POS), line discounts, payment allocation, due creation; sales returns with review and reversal records.
- Expense capture, daily summaries, basic sales/purchase/stock/due reports, export CSV.
- Bangla and English-ready interface; mobile-responsive operational screens.
- Tenant-specific settings and minimal internal platform operations for suspending/reactivating tenants and viewing audit events.

### P1 — After initial pilot

- Stock counts/adjustments with approvals; multi-warehouse transfer; purchase returns; supplier payable ledger; detailed margin and COGS reports.
- More advanced branch analytics, receipt printing templates, offline draft capture (not offline authoritative sales).
- Subscription plans, trials, automated billing and lifecycle notifications after a payment-provider decision.

### P2 — Separate products/extensions

- Public marketing website, dedicated mobile-friendly POS, supported external POS connectors, tenant storefronts, delivery, wholesale pricing, sophisticated accounting, advisory AI.

## 4. Explicit non-goals for P0

No public social feed; no farmer advisory diagnosis; no payment gateway claim; no generalized e-commerce; no universal third-party POS compatibility; no offline-finalized financial transactions; no microservices; no speculative ML demand forecasts.

## 5. Core business rules

1. Every tenant-owned record belongs to one tenant; every store-specific record additionally has `store_id`.
2. Membership and role must be checked server-side; user-provided tenant/store identifiers are never sufficient authorization.
3. All money stored as integer minor units (BDT paisa) or fixed precision numeric with documented rounding; no floating point.
4. A confirmed purchase posts stock **once**. A confirmed sale deducts stock **once**; both operations are atomic and idempotent.
5. Batch selection is required when category/store policy requires it; expired, recalled, blocked or quarantined batches cannot be sold.
6. Stock cannot go negative unless an explicit audited policy is later approved; default is reject.
7. Credit sales require identified customer and permission; due balances derive from posted ledger entries, not mutable balance fields alone.
8. Posted financial documents cannot be silently edited or deleted; correction is via reversal/return/adjustment.
9. Drafts are editable; posted documents are versioned or immutable.
10. Customer contact data must not be visible across tenants; exports require permission and audit.
11. SaaS manual sales and future POS sales use the same sale-posting service and inventory ledger.
12. Inventory valuation/margin reports must use defined costing method; until selected, do not label gross sales as profit.

## 6. Example acceptance criteria

- A newly registered tenant sees no other tenant's products, even by guessing UUIDs or editing API paths.
- Two cashiers attempting to sell the last item: at most one succeeds; stock never becomes negative.
- Retrying a confirmed purchase request with same idempotency key never doubles received stock.
- Partial payment creates the correct due; subsequent collection reduces due without changing original sale amount.
- Sales return links original line, posts reverse stock movement only for approved restockable quantity, and posts financial reversal.
- Mobile user can create a product, receive stock, record sale, and collect due at 360px viewport without horizontal page overflow.
- Deactivated staff cannot call protected APIs after revocation takes effect.

## 7. Pilot KPIs (targets to be chosen, not invented)

Task completion rate; time to record sale; reconciliation error count; repeat weekly active shop owners; stock discrepancy rate; support tickets per tenant; retention and willingness to pay. Define baselines with real shops before numeric targets.

## 8. Dependencies

Field interviews, local product units and batch conventions, licensing/compliance verification, internet reliability, printing hardware, SMS/email delivery, business ownership model and payment provider choice.
