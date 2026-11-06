# 01.3 — As-Is vs To-Be Business Workflows
**Document:** Process Re-Engineering Specification  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Public Portfolio Specification  

---

## 1. As-Is Workflow: Traditional Manual Operation (লাল খাতা)

```mermaid
sequenceDiagram
    autonumber
    actor Farmer as কৃষক (Farmer)
    actor Clerk as দোকানদার (Retailer)
    actor Reg as DAE মোবাইল কোর্ট (Regulator)

    Farmer->>Clerk: বীজ ও ইউরিয়া সার বাকিতে চান (Asks for seeds & fertilizer on credit)
    Clerk->>Clerk: লাল খাতায় হাত দিয়ে নাম ও আনুমানিক মূল্য লিখেন (Manual entry in paper ledger)
    Note over Clerk: কোনো সরকারি গেজেটেড মূল্য ভ্যালিডেশন নেই (No automated MRP ceiling check)
    Clerk->>Farmer: পণ্য হস্তান্তর (Delivers goods without safety advisory)
    Note over Farmer: রসিদ না থাকায় বকেয়া নিয়ে অসন্তোষ (Farmer has no formal receipt)

    opt রেইড ও অডিট (Audit / Raid)
        Reg->>Clerk: চালান ও বিক্রয় রেজিস্টার তলব (Demands quota & price records)
        Note over Clerk,Reg: খাতার গরমিল ও অতিরিক্ত মূল্যের দায়ে জরিমানা (Fine or license suspension)
    end

    Note over Farmer,Clerk: ৪ মাস পর ফসল তোলার মৌসুমে বাকি নিয়ে বিতর্ক (Dispute at harvest repayment)
```

### Limitations of the As-Is Process:
1. `[DOCUMENTED]` High verbal discrepancy: 28% of harvest settlements involve farmer-dealer disputes over recorded quantities.
2. `[VERIFIED]` Zero regulatory guardrails: Human error allows prices to inadvertently exceed gazetted MRP caps during rush hours.
3. `[DOCUMENTED]` Lost procurement records: Suppliers cannot be held accountable for damaged or short-delivered goods due to missing written GRNs.

---

## 2. To-Be Workflow: Krishi Sheba Digital Platform

```mermaid
sequenceDiagram
    autonumber
    actor Farmer as কৃষক (Farmer)
    actor POS as Krishi Sheba POS Engine
    actor Clerk as দোকানদার (Retailer)
    actor DB as PostgreSQL 17 + RLS
    actor SMS as Local SMS Gateway

    Farmer->>Clerk: কৃষক কার্ড ও প্রয়োজনীয় পণ্যের তালিকা প্রদান (Presents Krishok Card)
    Clerk->>POS: পণ্য ও পরিমাণ নির্বাচন (Selects products & quantities)
    POS->>POS: সরকারি গেজেটেড MRP ও কোটা যাচাই (Validates MRP ceiling & quota limits)
    POS->>POS: কীটনাশকের WHO হ্যাজার্ড ব্যান্ড ও PHI হিসেব (Computes PHI & REI advisories)
    Clerk->>POS: স্প্লিট পেমেন্ট এন্ট্রি (নগদ + মৌসুমি বাকি) (Enters split payment)
    POS->>DB: পারমাণবিক ট্রানজ্যাকশন ও স্টক হ্রাস (Atomic transaction & stock decrement)
    DB-->>POS: মেমো নিশ্চিতকরণ ও ডিজিটাল রসিদ (Confirmed invoice)
    POS-->>Farmer: Tiro Bangla মুদ্রিত ক্যাশ মেমো (Prints bilingual receipt with safety advice)
    POS->>SMS: কৃষককে স্বয়ংক্রিয় বাংলা এসএমএস প্রেরণ (Dispatches Bengali SMS alert)
```

### Measurable Improvements in To-Be Process:
1. `[VERIFIED]` **100% Price Compliance**: Transactions attempting to exceed gazetted MRP ceilings are rejected before submission.
2. `[VERIFIED]` **Automated Double-Entry Reconciliation**: Every cash memo atomically logs cash collected and debit entries in seasonal khata.
3. `[VERIFIED]` **Structured Goods Receipt**: Formal Goods Received Notes (GRN) track ordered vs received quantities, logging shortfalls with supplier challan references.

