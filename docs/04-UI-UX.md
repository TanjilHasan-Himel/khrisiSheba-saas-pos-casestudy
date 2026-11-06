# 04 — UI/UX Brief

## 1. Product personality

Trustworthy, fast, calm, practical, agricultural but not rustic cliché. No random gradients, stock farming hero overload, fake analytics, floating glass cards or excessive icons. Data accuracy and legibility take precedence.

## 2. Visual language

- Color tokens (proposal): primary forest `#173D32`, accent leaf `#3E7358`, warm background `#F7F7F1`, ink `#18221D`, warning amber `#B77A23`, danger `#B42318`.
- Typography: **Inter** or **Manrope** for Latin; **Hind Siliguri** or **Noto Sans Bengali** for Bangla. Use licensed/hosted sources, avoid shipping font files in user-facing artifacts. No decorative AI-looking type.
- Spacing: 4px scale; 8px radius for controls, 12px cards; consistent 1px dividers.
- Minimum tap targets 44×44 CSS px; visible focus; labels above fields; numeric tabular alignment; no icon-only critical actions.
- Language toggle Bangla/English; do not translate SKU, brand or lot codes automatically.

## 3. Responsive navigation

**Mobile 320–767px:** compact header with tenant/store switcher; bottom nav Dashboard, Inventory, Sales, Customers, More. Long forms use progressive sections; search-first lists; filter drawer; sticky save/confirm action. Avoid wide tables; use cards with expandable detail.
**Tablet 768–1023px:** collapsible side navigation, split detail panels.
**Desktop 1024px+:** left rail, top context bar, dense but readable data tables, keyboard shortcuts for power users.

## 4. Screen inventory

| Screen         | Key elements                                     | Critical state         |
| -------------- | ------------------------------------------------ | ---------------------- |
| Sign in        | credentials, recovery, session notice            | invalid/locked account |
| Onboarding     | stepper, business/store fields                   | interrupted resume     |
| Dashboard      | sales, dues, low stock, expiry, pending purchase | no transactions        |
| Products       | search, category filters, stock, add/edit        | duplicate SKU          |
| Product editor | bilingual name, unit, price, category fields     | validation             |
| Suppliers      | list, contact, history                           | empty                  |
| Purchases      | drafts, received, status, detail                 | partial receive        |
| Inventory      | availability by store/batch, movement history    | negative attempt       |
| Sales          | manual sale wizard, invoice detail, returns      | conflict/timeout       |
| Customers      | profile, due ledger, collections                 | privacy/overpayment    |
| Reports        | date/store filters, export, reconciliation       | partial data           |
| Staff          | invites, roles, store access                     | revoked permission     |
| Settings       | tenant/store profile, units, policy              | unsaved changes        |

## 5. Detailed mobile sale flow

1. Header shows active store clearly.
2. Search field autofocus optional; recent products only if permission and performance allow.
3. Each product shows unit, availability, price and batch warning.
4. Cart drawer shows line total and discounts; avoid accidental remove.
5. Customer optional unless credit.
6. Payment selection; confirmation review clearly displays payable, paid, due.
7. Submit disabled while in-flight, with idempotent retry if timeout.
8. Success view shows invoice ID, receipt/share action; never assume success before server confirmation.

## 6. Interaction and error standards

- Skeleton for loading, helpful empty states with one clear CTA, field-level validation, retryable network errors, explicit 403/tenant-suspended states.
- Destructive actions require confirmation and role check; posted financial docs use reversal flow.
- Show data freshness and sync state; do not fake live updates.
- Use design tokens; dark mode optional after functional light mode; support system reduced motion.

## 7. Design deliverables

Figma component library, 360px/768px/1440px layouts, all critical states, interaction notes, content strings in Bangla/English, keyboard/accessibility annotations, handoff acceptance checklist.
