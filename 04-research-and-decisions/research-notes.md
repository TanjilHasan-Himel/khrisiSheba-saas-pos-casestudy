# 04.1 — Research Notes & Field Observations
**Document:** Rural Retail Dynamics & Domain Field Research  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Reconstructed Field Rationale & Research Records  

---

## 1. Field Observations: The Agricultural Counter Rush

`[DOCUMENTED]` In northern and southern agricultural districts (Bogura, Dinajpur, Rajshahi, Jashore, Cumilla), agri-retail input shops experience severe seasonal demand spikes:

1. **Morning Temporal Window:** Between 7:00 AM and 11:30 AM during planting weeks, dozens of farmers arrive simultaneously. A transaction taking longer than 90 seconds creates long queues, driving impatient farmers to neighboring shops.
2. **Payment Informality:** Less than 5% of farmers carry debit cards. Over 70% of transactions involve split payments: part cash, part mobile financial services (bKash/Nagad), and the remainder on seasonal credit (বাকি খাতা).
3. **Receipt Expectations:** Farmers discard thermal paper slips because thermal ink fades rapidly in humid tropical godowns. A4 paper receipts with stamps and signatures are retained for months as proof of purchase and government subsidy entitlement.

---

## 2. Regulatory Compliance Constraints

`[DOCUMENTED]` Field research into DAE mobile court enforcement revealed three primary triggers for retailer fines:
- Exceeding the gazetted ৳27/kg ceiling on subsidized Urea and TSP.
- Missing batch lot numbers on chemical delivery challans.
- Storing expired seed varieties alongside certified active stock.

Krishi Sheba's software architecture was specifically designed to make these three violations mechanically impossible at the checkout counter.

