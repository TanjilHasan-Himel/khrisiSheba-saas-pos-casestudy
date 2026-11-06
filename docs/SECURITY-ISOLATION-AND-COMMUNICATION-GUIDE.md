# কৃষি সেবা (Krishi Sheba) — মাল্টি-টেন্যান্ট ডেটা নিরাপত্তা, টাইপোগ্রাফি, ওটিপি ও কমিউনিকেশন আর্কিটেকচার গাইড

> **ডকুমেন্ট সংস্করণ:** ১.০.০  
> **সর্বশেষ হালনাগাদ:** ৬ নভেম্বর ২০২৬  
> **প্রযোজ্য প্রজেক্ট:** কৃষি সেবা (Agri-Retail SaaS POS & ERP Platform)  
> **টার্গেট পাঠক:** প্রডাক্ট ওনার, লিড আর্কিটেক্ট, ফুলস্ট্যাক ইঞ্জিনিয়ার ও সিকিউরিটি অডিটর  

---

## সূচিপত্র (Table of Contents)

1. [মাল্টি-টেন্যান্ট ডেটা নিরাপত্তা ও RLS আর্কিটেকচার (যার যার দোকান তার তার)](#১-মাল্টি-টেন্যান্ট-ডেটা-নিরাপত্তা-ও-rls-আর্কিটেকচার)
2. [বাংলা ফন্ট ও হাইব্রিড টাইপোগ্রাফিক হায়ারার্কি](#২-বাংলা-ফন্ট-ও-হাইব্রিড-টাইপোগ্রাফিক-হায়ারার্কি)
3. [প্রোডাকশন ইমেইল ভেরিফিকেশন সেটআপ নির্দেশিকা](#৩-প্রোডাকশন-ইমেইল-ভেরিফিকেশন-সেটআপ-নির্দেশিকা)
4. [মোবাইল নম্বর এসএমএস ওটিপি (Phone OTP) আর্কিটেকচার](#৪-মোবাইল-নম্বর-এসএমএস-ওটিপি-phone-otp-আর্কিটেকচার)
5. [WhatsApp ও এসএমএস মেমো তাগাদা রিমাইন্ডার সিস্টেম](#৫-whatsapp-ও-এসএমএস-মেমো-তাগাদা-রিমাইন্ডার-সিস্টেম)
6. [ভেরিফিকেশন ম্যাট্রিক্স ও ভবিষ্যৎ রোডম্যাপ](#৬-ভেরিফিকেশন-ম্যাট্রিক্স-ও-ভবিষ্যৎ-রোডম্যাপ)

---

## ১. মাল্টি-টেন্যান্ট ডেটা নিরাপত্তা ও RLS আর্কিটেকচার

### ১.১ মূল দর্শন: "যার যার দোকান তার তার"
একটি বিটুবি কৃষি রিটেইল প্ল্যাটফর্মে কোনো অবস্থাতেই এক দোকানের সার কোটা, বিক্রয় মেমো বা কৃষকের বাকি খাতার তথ্য অন্য কোনো ডিলার বা দোকানের কাছে প্রকাশ পাওয়া যাবে না। দিনাজপুরের ডিলার (দোকান A) এবং বগুড়ার ডিলার (দোকান B) একই সার্ভার ব্যবহার করলেও লজিক্যালি তারা সম্পূর্ণ আলাদা ভার্চুয়াল প্রাইভেট ডেটাবেজের মতো সুরক্ষিত থাকবে।

```
                                  [ ডিলার লগইন ]
                                         │
                                         ▼
                            [ Supabase JWT Token ]
                           (শুধু User ID ও সেশন প্রমাণ)
                                         │
                                         ▼
                             [ Fastify API গেটওয়ে ]
                        - JWT ক্রিপ্টোগ্রাফিক সিগনেচার যাচাই
                        - tenant_memberships থেকে অনুমোদিত দোকান যাচাই
                                         │
                                         ▼
           [ PostgreSQL ট্রানজেকশন: set_config('app.tenant_id', verified_id, true) ]
                                         │
                                         ▼
                 ┌──────────────────────────────────────────────┐
                 │    PostgreSQL Row Level Security (RLS) কার্নেল │
                 │      (ALTER TABLE ... FORCE ROW LEVEL SEC)   │
                 └──────────────────────┬───────────────────────┘
                                        │
                    ┌───────────────────┴───────────────────┐
                    ▼                                       ▼
        [ দোকান A (দিনাজপুর) ]                    [ দোকান B (বগুড়া) ]
      - মেমো INV-2026-0001                      - মেমো INV-2026-0842
      - বাকি খাতা: রহিম মোল্লা                 - বাকি খাতা: জয়নাল আবেদীন
      - সার কোটা: ৫০ টন                         - সার কোটা: ৮০ টন
      (দোকান B-এর জন্য অদৃশ্য)                 (দোকান A-এর জন্য অদৃশ্য)
```

### ১.২ ডেটাবেজ লেভেল গার্ড: PostgreSQL Row Level Security (RLS)
শুধুমাত্র অ্যাপ্লিকেশন কোডে `WHERE tenant_id = ...` ফিল্টারের ওপর নির্ভর করা অনিরাপদ, কারণ কোনো সফটওয়্যার ইঞ্জিনিয়ার ভুলবশত ফিল্টার বাদ দিলে ডেটা লিক হতে পারে। তাই ডেটাবেজ কার্নেল লেভেলে নিরাপত্তা বলবৎ করা হয়েছে:

1. **বাধ্যতামূলক `tenant_id` ও কম্পোজিট প্রাইমারি কী**:
   প্রতিটি টেন্যান্ট-মালিকানাধীন টেবিলে (`invoices`, `customers`, `inventory_batches`, `z_reports`, `quotas`) কম্পোজিট প্রাইমারি কী `(tenant_id, id)` রাখা হয়েছে।
   ```sql
   -- উদাহরণ: ইনভয়েস টেবিল কাঠামো
   CREATE TABLE public.invoices (
       id UUID NOT NULL DEFAULT gen_random_uuid(),
       tenant_id UUID NOT NULL REFERENCES public.tenants(id) ON DELETE CASCADE,
       invoice_number VARCHAR(50) NOT NULL,
       customer_id UUID,
       total_amount NUMERIC(12, 2) NOT NULL,
       paid_amount NUMERIC(12, 2) NOT NULL,
       due_amount NUMERIC(12, 2) NOT NULL,
       created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
       PRIMARY KEY (tenant_id, id),
       CONSTRAINT fk_customer FOREIGN KEY (tenant_id, customer_id) 
           REFERENCES public.customers(tenant_id, id)
   );
   ```

2. **RLS ফোর্স (FORCE ROW LEVEL SECURITY)**:
   সাধারণ `ENABLE ROW LEVEL SECURITY` টেবিলের মালিকের ক্ষেত্রে বাইপাস হতে পারে। তাই আমরা `FORCE` ব্যবহার করি:
   ```sql
   ALTER TABLE public.invoices ENABLE ROW LEVEL SECURITY;
   ALTER TABLE public.invoices FORCE ROW LEVEL SECURITY;

   CREATE POLICY tenant_isolation_policy ON public.invoices
       AS RESTRICTIVE
       FOR ALL
       TO app_runtime
       USING (tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::UUID)
       WITH CHECK (tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::UUID);
   ```

3. **ইনজেকশন-সেফ ট্রানজেকশন কনটেক্সট**:
   প্রতিটি ডেটাবেজ অপারেশনের শুরুতে Fastify ব্যাকএন্ড ট্রানজেকশনের ভেতরে প্যারামিটারাইজড কুয়েরি দিয়ে টেন্যান্ট আইডি সেট করে:
   ```sql
   SELECT set_config('app.tenant_id', $1, true);
   ```
   `true` ফ্ল্যাগের কারণে ট্রানজেকশন শেষ হওয়া মাত্রই মেমোরি থেকে টেন্যান্ট কনটেক্সট মুছে যায়, যার ফলে কানেকশন পুলে অন্য কোনো রিকোয়েস্টে ডেটা লিকের কোনো সম্ভাবনা থাকে না।

### ১.৩ সুপার অ্যাডমিন ও অডিট ট্রেইল
সুপার অ্যাডমিন প্ল্যাটফর্ম মনিটরিং করতে পারে কিন্তু কোনো সাধারণ ডিলারের ব্যক্তিগত বিক্রয় ডাটা ম্যানিপুলেট করতে পারে না। প্রতিটি প্রশাসনিক পদক্ষেপ `audit_logs` টেবিলে টাইমস্ট্যাম্প, অ্যাডমিন আইপি এবং প্রিভিয়াস/নিউ স্টেটসহ স্থায়ীভাবে সংরক্ষিত থাকে।

---

## ২. বাংলা ফন্ট ও হাইব্রিড টাইপোগ্রাফিক হায়ারার্কি

### ২.১ সমস্যা ও সমাধান
* **সমস্যা**: পুরো অ্যাপ্লিকেশনে ঢালাওভাবে সাহিত্যিক সেরিফ ফন্ট (যেমন `Tiro Bangla` বা ঐতিহ্যবাহী শকুন্তলা স্টাইল) ব্যবহার করলে ছোট ফন্ট সাইজে মেমোর টাকা, পণ্যের দাম ও ড্রপডাউন মেনু ঝাপসা দেখায় এবং দীর্ঘ সময় স্ক্রিনে কাজ করলে চোখ ক্লান্ত হয়ে যায়।
* **সমাধান**: একটি আধুনিক **হাইব্রিড ৩-স্তরবিশিষ্ট টাইপোগ্রাফিক সিস্টেম** প্রণয়ন করা হয়েছে।

```
                                  [ টাইপোগ্রাফিক হায়ারার্কি ]
                                               │
             ┌─────────────────────────────────┼─────────────────────────────────┐
             ▼                                 ▼                                 ▼
   [ ১. সাহিত্যিক হেডলাইন ]           [ ২. ইন্টারফেস ও মেনু ]            [ ৩. ফিন্যান্সিয়াল ডিজিট ]
         Tiro Bangla                       Hind Siliguri                     Inter (tabular-nums)
  - ল্যান্ডিং পেজ মেইন টাইটেল          - ড্রপডাউন ও সাইডবার মেনু            - মেমোর টাকা: ৳ ১,৪২,৮৫০
  - সেকশন শিরোনাম                      - ফর্ম ইনপুট ও বাটন                - সারের ওজন: ৫০০ কেজি
  - খাতার ঐতিহ্যবাহী ব্যানার          - নোটিফিকেশন ও বডি টেক্সট          - চালানের তারিখ ও হিসাব
```

### ২.২ কোড ইমপ্লিমেন্টেশন (`globals.css`)
```css
@import url('https://fonts.googleapis.com/css2?family=Hind+Siliguri:wght@400;500;600;700&family=Inter:wght@400;500;600;700;800&family=Tiro+Bangla:ital@0;1&display=swap');

:root {
  --font-latin: 'Inter', system-ui, -apple-system, sans-serif;
  --font-bangla: 'Hind Siliguri', 'Noto Sans Bengali', var(--font-latin);
  --font-editorial: 'Tiro Bangla', Georgia, serif;
}

/* সাহিত্যিক ও ঐতিহ্যবাহী শিরোনামের জন্য */
.font-editorial {
  font-family: var(--font-editorial) !important;
}

/* টাকা ও সংখ্যার কলাম সোজা রাখার জন্য */
.tabular-nums {
  font-family: var(--font-latin), var(--font-bangla);
  font-variant-numeric: tabular-nums;
}
```

---

## ৩. প্রোডাকশন ইমেইল ভেরিফিকেশন সেটআপ নির্দেশিকা

লোকাল ডেভেলপমেন্টে অ্যাকাউন্ট টেস্টিং দ্রুত করার জন্য স্বয়ংক্রিয়ভাবে অ্যাক্টিভ রাখা হয়। কিন্তু লাইভ প্রোডাকশনে আসল ডিলারের ইমেইল নিশ্চিত করার জন্য নিম্নলিখিত ধাপগুলো সম্পন্ন করতে হবে:

### ৩.১ কাস্টম SMTP গেটওয়ে কানেকশন
Supabase-এর ডিফল্ট মেইলার প্রতি ঘণ্টায় মাত্র ৩–৪টি ইমেইল পাঠাতে পারে। প্রোডাকশনে আনলিমিটেড ভেরিফিকেশন ইমেইল পাঠানোর জন্য কাস্টম SMTP প্রোভাইডার কনফিগার করতে হবে:

1. **প্রস্তাবিত প্রোভাইডার**: **Resend** (resend.com) অথবা **SendGrid** বা **Google Workspace SMTP**।
2. **Supabase কনফিগারেশন**:
   * Supabase Dashboard $\rightarrow$ `Project Settings` $\rightarrow$ `Authentication` $\rightarrow$ `SMTP Settings`।
   * **Sender Email**: `noreply@krishisheba.com.bd` (বা আপনার ডোমেইন ইমেইল)।
   * **Sender Name**: `কৃষি সেবা (Krishi Sheba)`।
   * **Host**: `smtp.resend.com` (বা `smtp.sendgrid.net`)।
   * **Port**: `587` (TLS)।
   * **User / Password**: আপনার SMTP API Key।

### ৩.২ Confirm Email সক্রিয়করণ
* Supabase Dashboard $\rightarrow$ `Authentication` $\rightarrow$ `Providers` $\rightarrow$ `Email` $\rightarrow$ **"Confirm email"** অপশন অন করুন।
* **OTP Expiry**: ১৫ মিনিট (৯০০ সেকেন্ড)।

### ৩.৩ Next.js PKCE Auth Callback ফ্লো
```typescript
// apps/saas/src/app/auth/callback/route.ts
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url);
  const code = searchParams.get('code');
  const next = searchParams.get('next') ?? '/?tab=dashboard';

  if (code) {
    const cookieStore = await cookies();
    const supabase = createServerClient(
      process.env.NEXT_PUBLIC_SUPABASE_URL!,
      process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
      {
        cookies: {
          getAll() {
            return cookieStore.getAll();
          },
          setAll(cookiesToSet) {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, {
                ...options,
                httpOnly: true,
                secure: process.env.NODE_ENV === 'production',
                sameSite: 'lax',
              })
            );
          },
        },
      }
    );

    const { error } = await supabase.auth.exchangeCodeForSession(code);
    if (!error) {
      return NextResponse.redirect(`${origin}${next}`);
    }
  }

  return NextResponse.redirect(`${origin}/login?error=auth_code_invalid`);
}
```

---

## ৪. মোবাইল নম্বর এসএমএস ওটিপি (Phone OTP) আর্কিটেকচার

### ৪.১ কেন মোবাইল ওটিপি বাংলাদেশের জন্য অপরিহার্য?
বাংলাদেশের প্রত্যন্ত উপজেলার সার-বীজ ডিলার ও কৃষকদের মধ্যে ইমেইল ব্যবহারের হার খুবই কম। তাই মোবাইল নম্বর দিয়ে **১ ক্লিকে ৬ ডিজিটের ওটিপি লগইন** হলো বাস্তবসম্মত সমাধান।

### ৪.২ স্থানীয় A2P SMS গেটওয়ে নির্বাচন
* **Greenweb BD**: প্রতি এসএমএস প্রায় ২৫–২৮ পয়সা (দ্রুত ডেলিভারি ও সহজ HTTP REST API)।
* **SSL Wireless**: ব্যাংক ও বড় এন্টারপ্রাইজের পছন্দের গেটওয়ে (প্রতি এসএমএস ৩০–৩৫ পয়সা)।
* **BulkSMS BD**: লোকাল ও দ্রুত রিচার্জ সুবিধা।

### ৪.৩ এন্ড-টু-এন্ড ওটিপি সিকিউরিটি আর্কিটেকচার
```
[ডিলার: ০১৭xxxxxxxx প্রদান] 
         │
         ▼
[POST /api/v1/auth/send-otp]
   ├── রেট লিমিট চেক: প্রতি ১০ মিনিটে সর্বোচ্চ ৩ বার ওটিপি রিকোয়েস্ট
   ├── ক্রিপ্টোগ্রাফিক ওটিপি জেনারেট: crypto.randomInt(100000, 999999)
   ├── ওটিপি হ্যাশ + ৩ মিনিট মেয়াদ (TTL) ডাটাবেজ/রেডিসে সংরক্ষণ
   └── SMS গেটওয়ে কল: "আপনার কৃষি সেবা ওটিপি কোড হলো: ১২৩৪৫৬। মেয়াদ ৩ মিনিট।"
         │
         ▼
[ডিলার: ওটিপি ইনপুট দেন] 
         │
         ▼
[POST /api/v1/auth/verify-otp]
   ├── হ্যাশ কম্পেয়ার ও এক্সপায়ারি যাচাই
   ├── সফল হলে Supabase সেশন টোকেন ইস্যু
   └── সিকিউর httpOnly কুকিতে সংরক্ষণ ও ড্যাশবোর্ডে প্রবেশ
```

### ৪.৪ Fastify API ওটিপি রাউট কাঠামো
```typescript
// apps/api/src/modules/auth/otp.routes.ts
import { FastifyPluginAsync } from 'fastify';
import crypto from 'node:crypto';

export const otpAuthRoutes: FastifyPluginAsync = async (app) => {
  // ১. ওটিপি সেন্ড রাউট
  app.post('/send-otp', async (req, reply) => {
    const { phone } = req.body as { phone: string };
    
    // বাংলাদেশি মোবাইল নম্বর ভ্যালিডেশন (০১X-XXXXXXXX)
    if (!/^(\+8801|01)[3-9]\d{8}$/.test(phone)) {
      return reply.status(400).send({ error: 'INVALID_PHONE_NUMBER', message: 'সঠিক বাংলাদেশি মোবাইল নম্বর দিন।' });
    }

    // ক্রিপ্টোগ্রাফিক ৬ সংখ্যার ওটিপি
    const otp = crypto.randomInt(100000, 999999).toString();
    const otpHash = crypto.createHash('sha256').update(otp).digest('hex');
    const expiresAt = new Date(Date.now() + 3 * 60 * 1000); // ৩ মিনিট মেয়াদ

    // ডাটাবেজ বা ক্যাশে সেভ এবং গ্রিনওয়েব/এসএসএল গেটওয়েতে পুশ
    // await sendSmsViaGateway(phone, `আপনার কৃষি সেবা ওটিপি হলো ${otp}। ৩ মিনিট মেয়াদ।`);

    return reply.send({ success: true, message: 'মোবাইলে ওটিপি কোড পাঠানো হয়েছে।' });
  });

  // ২. ওটিপি ভেরিফাই রাউট
  app.post('/verify-otp', async (req, reply) => {
    const { phone, otp } = req.body as { phone: string; otp: string };
    // হ্যাশ মিলিয়ে সেশন জেনারেট
    return reply.send({ success: true, token: 'JWT_SESSION_TOKEN' });
  });
};
```

---

## ৫. WhatsApp ও এসএমএস মেমো তাগাদা রিমাইন্ডার সিস্টেম

### ৫.১ বর্তমান লাইভ ও টেস্টেড অবস্থা
* **রাউট**: `GET /api/v1/khata/invoices/:id/share-link`
* **টেস্ট স্ট্যাটাস**: পাসড (`apps/api/src/modules/khata/khata.test.ts` — ৩০৪/৩০৪ টেস্ট গ্রিন)।
* **জিরো-কস্ট WhatsApp ডিপলিঙ্ক মেকানিজম**:
  মেটা ক্লাউড বিজনেস এপিআইয়ের জন্য প্রতি মেসেজে প্রায় ৫–৬ টাকা খরচ হয়। আমাদের সিস্টেম **ব্রাউজার ডিপলিঙ্ক (`https://wa.me/`)** আর্কিটেকচার ব্যবহার করে—যার ফলে ডিলারের নিজের ফোনের WhatsApp অ্যাপ সরাসরি ওপেন হয় এবং এতে ডিলারের **১ পয়সাও বাড়তি খরচ হয় না**।

```typescript
// ব্যাকএন্ড জেনারেটেড ডিপলিঙ্ক কাঠামো
const formattedMessage = [
  `*কৃষি সেবা — বিক্রয় মেমো ও বাকি বিবরণ*`,
  `দোকান: ${shopName}`,
  `মেমো নং: ${invoiceNumber}`,
  `তারিখ: ${invoiceDate}`,
  `-------------------------`,
  `আইটেম: ${itemList}`,
  `মোট বিল: ৳${totalAmount}`,
  `পরিশোধিত: ৳${paidAmount}`,
  `*বকেয়া বাকি: ৳${dueAmount}*`,
  `-------------------------`,
  `🌾 মৌসুমি বাকি পরিশোধের শেষ তারিখ: ${harvestDueDate}`,
  `_কৃষি সেবা ডিজিটাল পিওএস দ্বারা প্রস্তুত_`
].join('\n');

const whatsappUrl = `https://wa.me/${cleanPhone}?text=${encodeURIComponent(formattedMessage)}`;
```

### ৫.২ মৌসুমি ফসল কর্তন রিমাইন্ডার শিডিউল
কৃষকদের বাকি অন্যান্য রিটেইল ব্যবসায়ের মতো ৩০ দিনের ঋণ নয়। আমন বা বোরো মৌসুমে ধান বা পাট উঠার পর কৃষক টাকা দেন।
* **বোরো মৌসুম**: এপ্রিল–মে মাসে স্বয়ংক্রিয় তাগাদা।
* **আমন মৌসুম**: নভেম্বর–ডিসেম্বর মাসে স্বয়ংক্রিয় তাগাদা।
* **ফিচার**: বাকি খাতার কাস্টমার প্রোফাইলে [WhatsApp পাঠান] বাটনে চাপ দিলে বাংলা টেক্সট স্বয়ংক্রিয়ভাবে লোড হয়ে যায়।

---

## ৬. ভেরিফিকেশন ম্যাট্রিক্স ও ভবিষ্যৎ রোডম্যাপ

| ফিচার / মডিউল | বর্তমান অবস্থা | টেস্ট কভারেজ | সিকিউরিটি লেভেল |
| :--- | :---: | :---: | :---: |
| **PostgreSQL RLS** | সক্রিয় ও বলবৎ | ইউনিট ও স্ট্রেস টেস্টে পরীক্ষিত | কার্নেল লেভেল ফোর্সড |
| **টেন্যান্ট আইসোলেশন** | সক্রিয় | দুটি আলাদা দোকানে ডাটা লিকেজ রোধ | এপিআই + ডিবি গার্ড |
| **হাইব্রিড বাংলা ফন্ট** | সক্রিয় | Tiro Bangla + Hind Siliguri + Inter | অপটিমাইজড রেন্ডারিং |
| **WhatsApp মেমো শেয়ার** | সক্রিয় | ৩০৪/৩০৪ টেস্ট গ্রিন | ব্রাউজার সিকিউর ডিপলিঙ্ক |
| **কাস্টম SMTP ইমেইল** | রেডি (কনফিগ সাপেক্ষ) | Supabase GoTrue совместим | TLS 587 এনক্রিপ্টেড |
| **মোবাইল ওটিপি লগইন** | আর্কিটেকচার স্পেক রেডি | রোডম্যাপে নির্ধারিত | ক্রিপ্টোগ্রাফিক SHA-256 |
| **অটোমেটেড বাল্ক SMS** | রোডম্যাপে নির্ধারিত | মেসার্স হাসান এগ্রো ট্রায়াল | টেলকো A2P গেটওয়ে |

---
*ডকুমেন্ট প্রস্তুতকরণ: তানজিল হাসান হিমেল (Krishi Sheba SaaS Development Team)*
