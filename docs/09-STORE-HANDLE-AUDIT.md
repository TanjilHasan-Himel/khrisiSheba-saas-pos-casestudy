# 09 — Database Audit & Store Handle Architecture Analysis

**Krishi Sheba (কৃষি সেবা) — Multi-Tenant SaaS POS & ERP**  
**Audit Date:** 2026-11-06  
**Status:** Completed & Pending User Direction (No destructive schema changes applied per user constraint)  

---

## ১. ডাটাবেজ অডিট সারসংক্ষেপ (Database Schema Audit)

আমাদের তিনটি মাইগ্রেশন ফাইল (`supabase/migrations/`) এবং ডাটাবেজ আর্কিটেকচার পুঙ্খানুপুঙ্খ অডিট করা হয়েছে:

### ক) বর্তমান `public.tenants` টেবিলের গঠন:
```sql
CREATE TABLE IF NOT EXISTS public.tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    legal_name VARCHAR(255) NOT NULL,
    trade_license_no VARCHAR(100),
    tin_number VARCHAR(100),
    fertilizer_license_no VARCHAR(100),
    pesticide_license_no VARCHAR(100),
    seed_license_no VARCHAR(100),
    owner_name VARCHAR(255) NOT NULL,
    owner_phone VARCHAR(20) NOT NULL,
    owner_email VARCHAR(255),
    whatsapp_number VARCHAR(20),
    district VARCHAR(100) NOT NULL DEFAULT 'বগুড়া',
    upazila VARCHAR(100) NOT NULL DEFAULT 'শেরপুর',
    address TEXT,
    status VARCHAR(50) NOT NULL DEFAULT 'ACTIVE',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### খ) অডিট ফলাফল:
1. **হ্যান্ডেল ফিল্ডের বর্তমান অবস্থা:** বর্তমানে ডাটাবেজে কোনো `handle`, `slug` বা `store_code` কলাম **নেই**।
2. **আইসোলেশন ও আইডেন্টিফায়ার:** প্রতিটি ডিলারশিপ এবং দোকানকে সনাক্ত করা হচ্ছে **`id (UUID)`** এবং ডিলারের **মোবাইল নম্বর (`owner_phone`)** দিয়ে।
3. **সাইনআপে হ্যান্ডেল তৈরি:** বর্তমানে সাইনআপের সময়ও কোনো হ্যান্ডেল জেনারেট বা সেভ হচ্ছে না।

---

## ২. হ্যান্ডেল কি তৈরি হওয়া দরকার, নাকি দরকার নেই? (Trade-off Analysis)

### সুবিধা (Pros of having a Handle):
1. **পাবলিক মেমো ও অনলাইন বাকি ভেরিফিকেশন (WhatsApp / SMS Link):**
   * ডিলার যখন কৃষককে এক ক্লিকে WhatsApp বা SMS-এ মেমোর লিংক পাঠাবে:
     * হ্যান্ডেল থাকলে: `krishisheba.com.bd/m/hasan-agro/inv-1082` (খুব পরিষ্কার, বিশ্বাসযোগ্য ও ব্র্যান্ডেড)।
     * হ্যান্ডেল না থাকলে: `krishisheba.com.bd/m/a223c1a8-140c-4be5-a428/inv-1082` (খটমটে ও হিজিবিজি)।
2. **দোকানের ডিজিটাল কিউআর কোড (Store QR Code):**
   * দোকানে টানিয়ে রাখার জন্য ডিলারের নিজস্ব কিউআর কোড (যেমন: `krishisheba.com.bd/@hasanagro`)।
3. **সাব-ডোমেইন বা ফেসবুক পেজে দেওয়ার মতো ব্র্যান্ডেড লিংক:**
   * যেমন Shopify-তে থাকে: `hasan-agro.krishisheba.com.bd`।

### অসুবিধা ও ঝুঁকি (Cons & Friction):
1. **গ্রামাঞ্চলের সাধারণ ডিলারদের সাইনআপ বাধা:**
   * শেরপুর, গোদাগাড়ী বা তারাগঞ্জের অধিকাংশ সার-বীজ ডিলারকে সাইনআপে ইংরেজি ইউনিক হ্যান্ডেল লিখতে বললে তারা বানান ভুল করবে বা সাইনআপ বন্ধ করে দেবে।
2. **নামের সংঘর্ষ (Duplicate Slugs):**
   * বাংলাদেশে "সততা এগ্রো" বা "বিসমিল্লাহ সার ঘর" নামে শত শত দোকান আছে। ডুপ্লিকেট হ্যান্ডেল হ্যান্ডলিং ডিলারদের বিরক্ত করতে পারে।
3. **ইআরপি অপারেশনের জন্য হ্যান্ডেল অপ্রয়োজনীয়:**
   * ক্যাশ মেমো কাটা, স্টক ম্যানেজমেন্ট, সরকারি সার কোটা এবং বাকির হিসাবের জন্য `tenant_id (UUID)` একাই শতভাগ শক্তিশালী ও নিরাপদ।

---

## ৩. প্রস্তাবিত আধুনিক সল্যুশন (Zero-Friction Auto-Slug)

ডিলারকে দিয়ে কোনো ইংরেজি হ্যান্ডেল লেখানোর দরকার নেই:
1. **ব্যাকগ্রাউন্ডে স্বয়ংক্রিয় হ্যান্ডেল তৈরি (Auto-Generated Slug):**
   * ডিলার যখন বাংলায় নাম ও জেলা দেবে (যেমন: "মেসার্স হাসান এগ্রো", বগুড়া, ফোন: 01712345678):
   * সিস্টেম স্বয়ংক্রিয়ভাবে তৈরি করবে: `hasan-agro-5678` অথবা `hasan-agro-bogura`।
2. **লগইন থাকবে শুধু মোবাইল নম্বর দিয়ে:**
   * হ্যান্ডেল মনে রাখার কোনো দরকার নেই; ডিলার লগইন করবে মোবাইল ও পিন দিয়ে।
3. **সেটিংস থেকে এডিট করার অপশন:**
   * যারা স্মার্ট ডিলার, তারা চাইলে পরবর্তীতে সেটিংস থেকে নিজের পছন্দমতো হ্যান্ডেল পরিবর্তন করতে পারবে।

---

## ৪. ভবিষ্যৎ মাইগ্রেশন প্ল্যান (User Approval এর পর এক্সিকিউট করা হবে)

```sql
-- Migration Proposal: Add unique handle to tenants
ALTER TABLE public.tenants 
ADD COLUMN IF NOT EXISTS handle VARCHAR(80) UNIQUE;

-- Backfill default demo shop handle
UPDATE public.tenants 
SET handle = 'hasan-agro-sherpur' 
WHERE owner_phone = '01712345678' AND handle IS NULL;

CREATE INDEX IF NOT EXISTS idx_tenants_handle ON public.tenants(handle);
```

---

## ৫. সার্বিক সিস্টেম স্বাস্থ্য ও কোয়ালিটি গেট (Health Matrix)

| উপাদান | স্ট্যাটাস | প্রমাণ / মেট্রিক |
| :--- | :---: | :--- |
| **TypeScript Typecheck** | ✅ PASS | ৫টি প্যাকেজে (SaaS, API, Contracts, Config, DB) ০ এরর |
| **Automated Tests** | ✅ PASS | **৩০৪টি টেস্ট উত্তীর্ণ** (১৪টি টেস্ট ফাইল) |
| **Fastify API Server** | ✅ PASS | `GET /` -> `{"name":"Krishi Sheba API","status":"ok"}` |
| **API Domain Routes** | ✅ PASS | `GET /api/v1/fertilizer/quotas` এবং `GET /api/v1/pesticides` লাইভ ডেটা রিটার্ন করছে |
| **Decoupled SaaS Routing** | ✅ PASS | `/` (পাবলিক ল্যান্ডিং পেজ) এবং `/app` (এসএএএস পিওএস ওয়ার্কস্পেস) |
| **Unified AuthModal** | ✅ PASS | ডিলার লগইন + ২-ধাপের সাইনআপ (রিয়েল ছবি আপলোড, NID, ট্রেড ও সার লাইসেন্স) |
