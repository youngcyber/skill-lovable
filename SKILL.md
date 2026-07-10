---
name: lovable-vibe-coder
description: "รับ idea แอปเป็นภาษาไทย แล้วสร้าง prompt แบบ chain pattern สำหรับ Lovable ใน 4 ชั้น: โครงสร้าง → ฟีเจอร์หลัก → จัดการข้อมูล → เก็บรายละเอียด"
---

# Lovable Vibe Coder

รับ idea แอปจาก user แล้วแปลงเป็น chain prompt พร้อม paste เข้า Lovable ทีละ chain ตามลำดับ ทุกอย่างตอบกลับเป็นภาษาไทย

## When to Use

- User บอกว่า "อยากสร้างแอป..." หรือ "อยากทำเว็บ..." ด้วย Lovable
- User ถามว่าจะเริ่มต้น build อะไรก่อนใน Lovable
- User มี idea คร่าวๆ แล้วอยากให้แบ่งเป็นขั้นตอน
- User ใช้ภาษาไทยในการอธิบาย project

## Input Schema

```yaml
app_name: string          # ชื่อแอป
app_description: string   # แอปทำอะไร สำหรับใคร
core_features: list       # ฟีเจอร์หลัก 3-5 อย่าง
has_auth: boolean         # ต้องการระบบ login หรือไม่ — default: true
has_payment: boolean      # มีการชำระเงินหรือไม่ — default: false
user_roles: list          # บทบาทผู้ใช้ เช่น [user, admin]
app_type: string          # ประเภทแอป: saas / landing / dashboard / ecommerce / form
```

## Workflow

### Step 1: ทำความเข้าใจ idea

ถ้า user บอก idea ไม่ครบ ให้ถามคำถามเดียวที่สำคัญที่สุดก่อน เช่น "ฟีเจอร์หลักที่ user จะทำในแอปนี้คืออะไรบ้าง?" อย่าถามหลายคำถามพร้อมกัน

### Step 2: จำแนกประเภทแอป

- **SaaS** → ใช้ `skills/saas-apps/SKILL.md`
- **Landing Page** → ใช้ `skills/landing-pages/SKILL.md`
- **Dashboard** → ใช้ `skills/dashboards/SKILL.md`
- **E-Commerce** → ใช้ `skills/e-commerce/SKILL.md`
- **Form** → ใช้ `skills/forms/SKILL.md`
- **Data Management** → ใช้ `skills/data/SKILL.md`

### Step 3: สร้าง Chain Prompt ใน 4 ชั้น

แบ่ง prompt ออกเป็น 4 chain ตามลำดับนี้เสมอ:

```
Chain 1 → โครงสร้าง (Structure)
          Layout shell, navigation, หน้าเปล่าทุกหน้า, สีและ typography
          ห้าม implement logic หรือ backend ในชั้นนี้

Chain 2 → ฟีเจอร์หลัก (Core Features)
          UI ของแต่ละฟีเจอร์, interaction, การแสดงผลข้อมูล
          ใช้ข้อมูล mock/hardcode ได้ก่อน ยังไม่ต้อง connect database

Chain 3 → จัดการข้อมูล (Data Management)
          Supabase schema, auth, CRUD operations, connect UI กับ database
          ทำทีละฟีเจอร์ อย่า connect ทุกอย่างพร้อมกัน

Chain 4 → เก็บรายละเอียด (Details & Polish)
          Loading states, empty states, error handling, mobile responsive,
          animations, SEO — ทำหลังทุกฟีเจอร์ทำงานได้แล้ว
```

**กฎที่ห้ามละเมิด:**
- Chain 1 (โครงสร้าง) ต้องมาก่อนเสมอ — ต้องเห็น UI ก่อน connect database
- Chain 2 (ฟีเจอร์) ก่อน Chain 3 (ข้อมูล) — สร้าง UI ก่อน แล้วค่อย wire database
- Chain 4 เป็นชั้นสุดท้ายเสมอ — polish หลังทุกอย่างทำงานได้
- แต่ละ chain ต้องมี "เสร็จเมื่อ" checklist ที่ตรวจสอบได้จริง

### Step 4: เขียน Prompt แต่ละ Chain

รูปแบบ prompt ที่ถูกต้อง:

```
[สิ่งที่จะสร้าง — ประโยคเดียวชัดเจน]

Context:
- ใครใช้: [บทบาท user]
- เป้าหมาย: [user ต้องการทำอะไร]
- อยู่ที่ไหน: [route หรือ section ในแอป]

Tech Stack:
- React + Vite + TypeScript
- Tailwind CSS + shadcn/ui
- [เพิ่มเติมตามที่จำเป็น]

[รายละเอียด spec: columns, fields, behavior, design]

เสร็จเมื่อ:
- [ ] [ผลลัพธ์ที่ตรวจสอบได้]
- [ ] [ผลลัพธ์ที่ตรวจสอบได้]
- [ ] [ผลลัพธ์ที่ตรวจสอบได้]
```

### Step 5: ส่ง Plan ทั้งหมด

แสดงทั้ง 4 chain พร้อม prompt ที่ paste ได้เลย พร้อมบอก user ให้ verify checklist ก่อนไป chain ถัดไป

## Output Schema

```yaml
plan_title: string        # "[ชื่อแอป] — Build Plan"
app_summary: string       # สรุปหนึ่งประโยคว่าเข้าใจ idea ถูกต้อง
chains:
  - number: integer       # Chain 1-4
    name: string          # ชื่อ chain (โครงสร้าง / ฟีเจอร์หลัก / จัดการข้อมูล / เก็บรายละเอียด)
    goal: string          # เป้าหมายของ chain นี้
    prompt: string        # Prompt ที่พร้อม paste เข้า Lovable
    done_when: list       # Checklist 3-5 ข้อ
total_chains: integer
```

## Output Format

```
# [ชื่อแอป] — Build Plan
> [สรุปหนึ่งประโยค]

---

## Chain 1 — โครงสร้าง
**เป้าหมาย:** สร้าง layout shell และ navigation ก่อน implement logic ใดๆ

[PROMPT ที่พร้อม paste]

**เสร็จเมื่อ:**
- [ ] ...
- [ ] ...

---

## Chain 2 — ฟีเจอร์หลัก
**เป้าหมาย:** UI และ interaction ของแต่ละฟีเจอร์ด้วยข้อมูล mock

[PROMPT]

**เสร็จเมื่อ:**
- [ ] ...

---

## Chain 3 — จัดการข้อมูล
**เป้าหมาย:** Connect database และ wire ฟีเจอร์ทุกตัวเข้ากับ Supabase

[PROMPT]

**เสร็จเมื่อ:**
- [ ] ...

---

## Chain 4 — เก็บรายละเอียด
**เป้าหมาย:** Loading states, error handling, mobile, polish

[PROMPT]

**เสร็จเมื่อ:**
- [ ] ...

---

**รวม 4 Chains**
Paste Chain 1 เข้า Lovable ก่อน ✓ ทุก checkbox แล้วค่อยไป Chain 2
```

## Error Handling

- **Idea ไม่ชัดเจน** — ถามคำถามเดียวที่สำคัญที่สุด อย่าสร้าง prompt จน user ตอบแล้ว
- **User อยากรวม database กับ UI ในชั้นเดียว** — อธิบายว่า chain pattern แยก front-end ออกก่อน แล้วค่อย wire backend ใน Chain 3 ได้ผลลัพธ์ดีกว่า
- **มีหลาย app type** — รวม chain จากทั้งสอง type ตามลำดับที่ถูกต้อง ไม่ duplicate

## Examples

### ตัวอย่าง 1: SaaS จัดการ Task

**Input:** "อยากทำแอป task manager ชื่อ Taskly สำหรับทีมเล็กๆ"

```
Chain 1 — โครงสร้าง
สร้าง layout shell: sidebar, navbar, หน้า dashboard เปล่า, หน้า tasks เปล่า

Chain 2 — ฟีเจอร์หลัก
Task list ที่แสดง mock data, form สร้าง task, drag-to-complete checkbox

Chain 3 — จัดการข้อมูล
Supabase: tasks table + auth + RLS, connect form กับ database, real CRUD

Chain 4 — เก็บรายละเอียด
Skeleton loading, empty state "ยังไม่มี task", error toast, mobile layout
```

### ตัวอย่าง 2: ร้านค้าออนไลน์

**Input:** "อยากทำเว็บขายเทียนหอมชื่อ Wax & Wick"

```
Chain 1 — โครงสร้าง
Navbar, footer, หน้า /shop grid เปล่า, หน้า product detail เปล่า

Chain 2 — ฟีเจอร์หลัก
Product cards ด้วย mock data, cart slide-over UI, checkout form UI

Chain 3 — จัดการข้อมูล
Supabase products table, CartContext + localStorage, Stripe checkout

Chain 4 — เก็บรายละเอียด
Lazy loading รูป, "Out of stock" state, mobile-first layout, SEO meta tags
```
