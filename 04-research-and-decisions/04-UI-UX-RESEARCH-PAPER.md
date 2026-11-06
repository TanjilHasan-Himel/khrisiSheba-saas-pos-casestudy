# UI/UX Research Notes: Real-World Retail & POS Benchmarks

**Document**: Field Research & UI/UX Benchmarks  
**Author**: Tanjil Hasan Himel  
**Benchmarked Tools**: Square POS, Toast POS, Khatabook, DeHaat  

---

## 1. Why Standard POS Software Fails in Rural Agri-Retail

Western retail POS systems (like standard Shopify or Square) assume constant barcode scanning, card payments, and simple retail inventories. Generic ledger apps (like Khatabook) track informal credit but lack inventory decrement, government fertilizer quotas, or chemical safety warnings.

In Upazila and Union bazaars across Bangladesh, agri-retail input shops have three specific operational realities:
1. **Cash dominance**: Over 85% of transactions are in cash, often with split payments (partial cash, partial MFS, remainder on seasonal credit).
2. **Seasonal crop credit**: Farmers settle debts when crops are harvested (Boro in May, Aman in November), not on standard 30-day billing cycles.
3. **Statutory enforcement**: DAE mobile courts penalize dealers for selling subsidized fertilizer above gazetted price caps or over the 500kg seasonal limit per farmer.

---

## 2. Benchmark Decisions Implemented in Krishi Sheba

### A. Fast Checkout & Cash Ergonomics (Inspired by Square & Toast POS)
* **Dual-Pane Layout**: Product selection on the left, sticky cart and settlement on the right.
* **Quick Cash Buttons**: One-tap buttons for common Bangladesh Bank notes (`৳500`, `৳1,000`, `৳2,000`, `হিসাব সমান`) so cashiers never have to type common bill amounts.
* **Change Due Calculation**: Automatic calculation showing exact cash to return to the customer, eliminating mental math errors during morning rushes.
* **Keyboard Hotkey**: Global `[/]` shortcut to focus product search instantly without touching the mouse.

### B. Seasonal Farmer Khata (Inspired by Khatabook & OkCredit)
* **Harvest-Linked Due Dates**: Due dates tied to real crop harvest cycles (Boro, Aman, Aus, Rabi) instead of arbitrary 30-day deadlines.
* **Debtor Aging**: Balances categorized into 0–30 days (Current), 31–60 days (Aging), and 60+ days / Past Harvest (Overdue).
* **Direct Notifications**: One-click WhatsApp and native SMS links with pre-formatted Bengali billing summaries sent directly from the browser.

### C. Chemical Safety & Quota Clamps (Inspired by DeHaat)
* **WHO Toxicity Badges**: Clear color-coded hazard bands (Red, Yellow, Blue, Green) on pesticide products.
* **Pre-Harvest Interval (PHI)**: Mandatory harvest waiting days printed directly on sales memos.
* **DAE Quota Limits**: Automated limit capping subsidized fertilizer sales to 500kg per farmer per season, requiring a valid Krishok Card or NID.

### D. Cash Drawer Balancing (Inspired by Toast POS)
* **Z-Report Shift Reconciliation**: Blind physical count of currency notes at closing time compared against system expected cash, flagging register over/short discrepancies.

---

## 3. Interface Design Principles

* **Typography**: Clean modern sans-serif (`Hind Siliguri` for Bengali, `Inter` for tabular numbers). No decorative or literary serif fonts that strain eyes on screens.
* **High Contrast**: Dark forest green (`#173D32`) on warm background (`#F7F7F1`) tested for outdoor daylight glare.
* **Mobile Thumb Zone**: 44px+ touch targets with bottom navigation on mobile viewports.
