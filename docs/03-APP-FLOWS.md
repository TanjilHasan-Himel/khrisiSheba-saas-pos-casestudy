# 03 — App Flows and State Transitions

## A. Tenant onboarding

2-Stage Progressive Onboarding:
- **Stage 1 (Personal & Account Security)**: Owner Full Name, Mobile Number (017xxxxxxxx), NID Number, Email, Password, Passport size Photo, NID Card upload with instant preview.
- **Stage 2 (Shop & Regulatory DAE Licenses)**: Shop Name, Business Type (Retailer/Wholesale Dealer), District, Upazila, Market Address, Agricultural Categories (Fertilizer, Pesticide, Seeds, Machinery), Trade License No. + Document upload, TIN Number, DAE License numbers (Fertilizer License, Pesticide Form 10/17, Seed Dealer Registration), Shop Front / Godown photo.
- **Completion**: Account created in Supabase Auth, documents stored in Supabase Storage (`shop-documents`), shop profile confirmed with setup checklist.

(Note: Google SSO deferred to a future phase per owner instruction).

**Exceptions:** existing account, expired invitation, user belonging to multiple tenants, tenant suspended, duplicate business name (allowed; UUID identity), interrupted onboarding resumes draft.

## B. First stock receipt

Create category/unit → create product → supplier → purchase draft → line items + cost + expected batch/expiry → review → receive actual quantities → server posts inventory movements → updated availability and purchase status.

**Exceptions:** partial receipt, unexpected quantity, duplicate receive retry, batch missing when required, expired goods, damaged/quarantined stock, cancellation before posting.

## C. Manual sale (SaaS core, not POS)

Select store → search product → choose quantity/batch → optional customer → choose paid/partial/credit → server previews calculated total → confirm → transactional posting → receipt + stock and ledger update.

**Exceptions:** stock exhausted during confirmation, credit not permitted, unknown customer, partial payment exceeds total, permission denied, network timeout after commit (retry same idempotency key).

## D. Customer due collection

Search customer → view posted ledger and outstanding amount → enter collection amount/method/reference → review → confirm → post collection ledger and payment → issue receipt.

**Exceptions:** overpayment handling policy unresolved, duplicate submission, stale balance, collection reversal requires manager permission.

## E. Sale return

Find posted sale → choose eligible items/quantities → reason + condition → approval if required → decide restock vs quarantine → post return financial and stock movements → linked receipt.

**Exceptions:** return exceeds original net quantity, sale in another tenant, product non-returnable under policy, refund greater than paid amount, credit-only refund adjustments.

## F. Stock correction

Manager requests adjustment → reason + evidence → approver review (where required) → posted adjustment movement → audit log. Never overwrite stock balance silently.

## G. Tenant / store switching

User selects accessible tenant/store → API checks active membership → session tenant context changes → clear tenant-scoped client cache → reload authorized data. No leakage through browser back or stale cached requests.

## H. Platform operator

Platform admin login with stronger auth → list tenant metadata → manage plan/status → audited support workflow → no default unrestricted browsing of tenant sales/customer data.

## I. Future POS attachment

Tenant owner opens Integrations/Add-ons → choose Krishi Sheba POS → provision device/store association → cashier authenticates → device syncs catalog → posts sales via shared sales API → dashboard reflects committed transactions. Existing POS: select supported connector/import, authorize credentials, map products, run dry-run, review exceptions, enable sync.

## User state examples

Purchase: DRAFT → APPROVED (optional) → PARTIALLY_RECEIVED → RECEIVED; DRAFT → CANCELLED.
Sale: DRAFT → POSTED → PARTIALLY_RETURNED → RETURNED (as applicable); never `POSTED → DRAFT`.
Tenant: ONBOARDING → ACTIVE → SUSPENDED → CLOSED (with retention policy).
