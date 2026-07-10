---
name: lovable-landing-page-planner
description: "รับ idea product แล้วสร้าง prompt แบบ chain pattern 4 ชั้นสำหรับ landing page บน Lovable: โครงสร้าง → section หลัก → เชื่อม backend → เก็บรายละเอียด รองรับภาษาไทย"
---

# Lovable Landing Page Planner

สร้าง chain prompt สำหรับ landing page บน Lovable ใน 4 ชั้น สร้าง layout shell ก่อน แล้วเพิ่มทีละ section ตามลำดับความสำคัญ

## When to Use

- User อยากสร้าง landing page หรือ marketing site ด้วย Lovable
- User ต้องการ hero, pricing, testimonials, หรือ FAQ section
- User พูดถึงหน้าเว็บสาธารณะที่ไม่ต้อง login
- User ใช้ภาษาไทยในการอธิบาย product

## Input Schema

```yaml
product_name: string          # ชื่อ product
product_description: string   # ทำอะไร สำหรับใคร
primary_cta: string           # action หลัก เช่น "เริ่มใช้ฟรี"
primary_cta_route: string     # ลิงก์ CTA เช่น /signup
sections: list                # [hero, features, how-it-works, pricing, testimonials, faq, cta-banner]
has_pricing: boolean          # มี pricing section ไหม
plan_names: list              # ชื่อ plan ถ้ามี pricing
brand_color: string           # สีหลัก default: indigo
```

## Workflow

### Step 1: ทำความเข้าใจ product

ถ้าไม่รู้ `primary_cta_route` ถามก่อน — ทุก section ต้องการ route นี้

### Step 2: วาง 4 Chain

```
Chain 1 → โครงสร้าง: Navbar, footer shell, route เปล่าที่ /
Chain 2 → ฟีเจอร์หลัก: Hero, features, testimonials, pricing sections
Chain 3 → จัดการข้อมูล: Form submissions, waitlist, analytics (ถ้ามี)
Chain 4 → เก็บรายละเอียด: SEO meta, performance, mobile, animations
```

### Step 3: เขียน Prompt แต่ละ Chain

---

**Chain 1 — โครงสร้าง**

```
สร้าง layout shell สำหรับ landing page ของ [ชื่อ product]

Tech stack:
- React + Vite + TypeScript
- Tailwind CSS + shadcn/ui
- Lucide React
- react-helmet-async (ติดตั้งด้วย)

Navbar:
- โลโก้ "[ชื่อ product]" (font-bold, [brand_color]-600) ซ้าย
- Nav links กลาง/ขวา: [LIST links]
- CTA ปุ่มขวาสุด: "[primary_cta]" → [primary_cta_route] (indigo-600, white text)
- Sticky: เมื่อ scroll 64px → bg-white/90 backdrop-blur border-b border-gray-100
- Mobile: hamburger → drawer, ปิดเมื่อ click link

Footer:
- โลโก้ + tagline ซ้ายบน
- Link columns: [COLUMNS]
- Bottom bar: "© [ปี] [ชื่อ product]" + Privacy + Terms
- bg-gray-900 text-white

Route / : แสดงแค่ placeholder "[ชื่อ product] Landing Page" centered ก่อน

เสร็จเมื่อ:
- [ ] Navbar render พร้อม link และปุ่ม CTA
- [ ] Navbar sticky + backdrop blur หลัง scroll
- [ ] Footer render ครบทุก column
- [ ] Mobile hamburger เปิด/ปิดได้
- [ ] / route render ไม่ error
```

---

**Chain 2 — ฟีเจอร์หลัก**

```
เพิ่ม sections ให้ landing page ของ [ชื่อ product]

[สร้างทีละ section ตาม sections_needed:]

HERO SECTION (ถ้าต้องการ):
- Headline: "[HEADLINE]"
- Subheadline: "[SUBHEADLINE — ไม่เกิน 2 บรรทัด]"
- Primary CTA: "[primary_cta]" → [primary_cta_route] (indigo-600)
- Secondary CTA: "[text]" → [action]
- Social proof: "[ข้อความ เช่น ใช้โดย 2,000+ ทีม]"
- Hero image: placeholder 16:9 (gray-100 bg, rounded-xl) — สลับทีหลัง
- Layout: centered mobile / text-left image-right desktop (lg:grid-cols-2)
- มีแค่ 1 h1 ในทั้งหน้า

FEATURES SECTION (ถ้าต้องการ):
- Heading: "[HEADING]" (h2)
- [N] cards: icon (Lucide) + title + description 2 ประโยค
- Grid: 3 cols desktop, 2 tablet, 1 mobile
- Card: white, border, rounded-2xl, hover:shadow-md

PRICING SECTION (ถ้า has_pricing):
- Toggle monthly/annual (default: monthly)
- [N] plan cards — most popular: indigo bg, scale-105
- "Save X%" badge เมื่อเลือก annual
- Plan ราคา custom → CTA "ติดต่อเรา" → /contact

TESTIMONIALS SECTION (ถ้าต้องการ):
- 6 quote cards, 3-column grid
- แต่ละ card: 5 stars (fill-yellow-400), quote, ชื่อ + ตำแหน่ง + บริษัท
- bg-gray-50 section

FAQ SECTION (ถ้าต้องการ):
- shadcn Accordion, one open at a time
- [N] คำถาม-คำตอบ

FINAL CTA BANNER:
- bg-indigo-600, centered, ปุ่ม white

เสร็จเมื่อ:
- [ ] Hero แสดงครบ: headline, 2 CTAs, social proof, placeholder image
- [ ] CTA ทุกปุ่มลิงก์ถูกต้อง
- [ ] Pricing toggle เปลี่ยนราคาได้
- [ ] Stars ใน testimonials เป็น fill-yellow-400
- [ ] ทุก section heading เป็น h2 (ยกเว้น hero ที่เป็น h1)
```

---

**Chain 3 — จัดการข้อมูล**

*ใช้เฉพาะถ้ามี form หรือ backend เช่น waitlist, contact form*

```
เพิ่มระบบ backend ให้ [ชื่อ product] landing page

[ถ้ามี waitlist form:]
Supabase table:
waitlist (
  id uuid primary key default gen_random_uuid(),
  email text unique not null,
  created_at timestamptz default now()
)
RLS: INSERT เปิดสำหรับทุกคน (public), SELECT เฉพาะ authenticated

Connect form:
- Email input + ปุ่ม "เข้าร่วมรายการรอ"
- INSERT into waitlist on submit
- Success: เปลี่ยนเป็น "สมัครแล้ว! เราจะแจ้งเตือนคุณ"
- Duplicate email: แสดง "Email นี้สมัครไว้แล้ว"

[ถ้ามี contact form:]
- POST /api/contact → ส่ง email ด้วย Resend
- fields: ชื่อ, email, ข้อความ
- ต้อง validate ก่อน submit

เสร็จเมื่อ:
- [ ] Form submit บันทึกข้อมูลจริง
- [ ] Success state แสดงหลัง submit
- [ ] Duplicate email แสดง error ถูกต้อง
```

---

**Chain 4 — เก็บรายละเอียด**

```
เพิ่ม SEO, performance, และ polish ให้ [ชื่อ product] landing page

SEO (react-helmet-async):
- Title: "[ชื่อ product] — [tagline]"
- Description: "[max 160 ตัวอักษร]"
- og:title, og:description, og:image, og:type: website
- twitter:card: summary_large_image
- Canonical URL: https://[domain]/

Images:
- Hero image: fetchpriority="high" (above the fold)
- ทุกรูปอื่น: loading="lazy" + explicit width + height

Performance:
- Google Fonts: <link rel="preconnect"> + <link rel="dns-prefetch">
- ห้ามมี render-blocking scripts

Animations:
- Section scroll-in: fade-up (translateY(20px) → 0, opacity 0 → 1) เมื่อ scroll เข้า viewport
- Pricing toggle: price crossfade 200ms
- CTA button hover: scale-105 transition

Mobile audit:
- ทุก section อ่านง่ายบน 375px iPhone
- Pricing cards stack vertically (most popular ขึ้นก่อน)
- Navbar mobile drawer ปิดเรียบร้อย

เสร็จเมื่อ:
- [ ] Meta tags ครบ og: และ twitter:
- [ ] Hero image มี fetchpriority="high"
- [ ] รูปอื่นทุกรูปมี loading="lazy"
- [ ] Scroll animation ทำงานได้
- [ ] ทุก section อ่านง่ายบน 375px
```

## Output Schema

```yaml
plan_title: string          # "[Product Name] Landing Page — Build Plan"
product_summary: string     # สรุปหนึ่งประโยค
chains:
  - number: integer
    name: string
    goal: string
    prompt: string
    done_when: list
total_chains: integer       # 3 ถ้าไม่มี backend, 4 ถ้ามี
```

## Output Format

```
# [ชื่อ Product] Landing Page — Build Plan
> [สรุปหนึ่งประโยค]

---

## Chain 1 — โครงสร้าง
**เป้าหมาย:** Navbar + footer shell ก่อน section ใดๆ

[PROMPT]

**เสร็จเมื่อ:** ...

---
(Chain 2, 3, 4)
---

**รวม [N] Chains**
Paste Chain 1 เข้า Lovable ✓ ทุก checkbox แล้วไป Chain 2
```

## Error Handling

- **ไม่รู้ primary_cta_route** — ถามก่อน ทุก CTA ต้องการ route
- **อยากรวม hero + features ใน chain เดียว** — แยกเป็น sub-section ใน Chain 2 ได้ แต่ยังอยู่ใน chain เดียว
- **ไม่มี backend** — ข้าม Chain 3 เลย รวม 3 chains

## Examples

### ตัวอย่าง: SaaS Landing Page

**Input:** product: Compose (AI เขียน content), CTA: "ทดลองฟรี" → /signup, sections: [hero, features, testimonials, pricing, faq, cta-banner], has_pricing: true, plans: [Starter, Pro]

- Chain 1: Navbar (Features, Pricing, Sign in, "ทดลองฟรี"), footer
- Chain 2: Hero + 6 feature cards + 6 testimonials + pricing toggle + FAQ + CTA banner
- Chain 3: ข้าม (ไม่มี form/waitlist)
- Chain 4: SEO meta, lazy loading, scroll animations, mobile audit
