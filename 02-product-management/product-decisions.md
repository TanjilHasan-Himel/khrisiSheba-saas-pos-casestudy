# 02.2 — Product Decisions & Trade-Offs Log
**Document:** Product Management Strategic Decisions  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Canonical Public Portfolio Record  

---

## 1. Key Product Decisions & Rationale

### Decision 1: Crop-Harvest Credit Cycles vs 30-Day Calendar Invoicing
- **Alternative Considered:** Standard corporate accounts receivable (Net 15, Net 30, Net 60).
- **Decision:** Categorize all farmer credit balances into biological harvest cycles (Boro, Aman, Aus, Rabi).
- **Trade-Off:** Required custom ledger aggregation math instead of standard off-the-shelf accounting plugins.
- **Product Rationale:** Western 30-day billing treats creditworthy farmers as chronic defaulters during planting. Crop-aligned ledgers reflect ground reality, allowing dealers to forecast cash inflows around regional harvest months.

### Decision 2: Hard Block on Price Ceilings vs Soft Warning
- **Alternative Considered:** Non-blocking warning dialog allowing cashiers to override price with a manager passcode.
- **Decision:** Hard blocking validation at both POS UI and API backend when price exceeds gazetted MRP.
- **Trade-Off:** Reduced cashier flexibility when local transport costs increase.
- **Product Rationale:** In Bangladesh, mobile court magistrate raids impose strict liability. A single overcharged bag can lead to ৳50,000 in fines or shop closure. Eliminating overrides protects the dealer unconditionally.

### Decision 3: Decoupled Super Admin Portal vs In-App Tab Switcher
- **Alternative Considered:** An "Admin Mode" toggle button inside the retailer merchant sidebar.
- **Decision:** Completely isolate Super Admin at a separate route (`/superadmin`) guarded by Master Security PIN challenge (`7890`).
- **Trade-Off:** Requires separate authentication flow and isolated layout structure.
- **Product Rationale:** Retail cashiers and counter clerks must have zero access to or awareness of nationwide multi-tenant telemetry, platform revenues, or regulatory license approval queues.

### Decision 4: High-Contrast A4 Print Engine vs Mobile-Only Receipts
- **Alternative Considered:** SMS-only receipts or standard 58mm thermal receipts.
- **Decision:** Support both thermal cash memos and full A4 printed purchase orders/GRNs with DAE trade license numbers and dual signature blocks.
- **Trade-Off:** Required complex `@media print` CSS rules, page-break safeguards, and ink-friendly high-contrast styling.
- **Product Rationale:** Official government audits, buffer godown disputes, and formal bank loan applications mandate physical A4 paper trails with handwritten authorization signatures.

