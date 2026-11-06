# 08 — Decision Register and Open Questions

## Confirmed by user

- Krishi Sheba is a **multi-tenant SaaS from the start**.
- Website, SaaS, POS and API are distinct responsibilities/apps.
- SaaS core is developed before the marketing website and dedicated POS.
- POS is an optional add-on; supported existing POS integrations are a future goal.
- Mobile-friendly design is mandatory across customer-facing operational applications.
- Initial agricultural categories include fertilizer, seeds, pesticides; tools may be added.
- No AI-slop design or inflated claims.
- **Authentication & Onboarding**: Initial phase uses a **Two-Stage Onboarding Form** (Email + Password + NID + Mobile Number + Documents/Photo Storage). Google SSO is deferred until later. Document images (Trade License, NID, Shop Photo, Owner Photo) are stored in Supabase Storage (`shop-documents`). Agricultural licenses follow DAE regulations (Fertilizer License, Pesticide Form 10/17, Seed Dealer Registration).

## Proposed defaults, NOT user-confirmed

- Agricultural retail first, single-country Bangladesh first.
- Online-first authoritative sales; offline drafts later.
- PostgreSQL + TypeScript modular monolith.
- BDT minor-unit money representation.
- SaaS includes manual sales; dedicated POS deferred.
- Minimal platform operations initially, separate platform admin frontend later.

## Questions to resolve before implementation

1. Which shop is pilot and what exact products/units are stocked?
2. Are sales primarily cash, credit, mobile financial service, or mixed?
3. How are dues settled: oldest invoice first, chosen invoice, or general customer balance?
4. Does the shop need batch/expiry on all items or only specific categories?
5. Are purchases and stock tracked by carton/packet/kg/litre, including conversions?
6. Are there wholesale prices, variable prices, negotiable discounts or tax requirements?
7. Is the first POS online-only or must it post sales offline?
8. What target Android devices, barcode scanners and receipt printers exist?
9. [RESOLVED - User Confirmed] How will accounts be verified: 2-stage onboarding form (Email, Password, NID, Mobile, Documents) for launch; Google SSO deferred to later phase.
10. Will SaaS pricing be per store, user, transaction, or plan? What free trial?
11. Which external POS vendors are candidates and do they expose APIs/exports?
12. Which legal licences, regulated-product records and privacy obligations apply?
13. What is the inventory costing method and definition of gross profit?
14. Is supplier credit/payables P0 or P1?
15. Who may approve returns, discounts, write-offs and credit limits?

## Change control

Every material decision should record date, owner, alternatives, rationale, affected modules and migration implications. Update PRD/TRD/schema together; avoid silent scope drift.
