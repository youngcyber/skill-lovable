---
name: lovable-ecommerce-planner
description: "รับ idea ร้านค้าออนไลน์ แล้วสร้าง prompt แบบ chain pattern 4 ชั้นสำหรับ Lovable: โครงสร้าง → catalog & cart → checkout & payment → เก็บรายละเอียด รองรับภาษาไทย"
---

# Lovable E-Commerce Planner

รับ idea ร้านค้าออนไลน์จาก user แล้วสร้าง chain prompt พร้อม paste เข้า Lovable ทีละ chain ตอบกลับทุกอย่างเป็นภาษาไทย รวมถึง prompt ที่สร้างสำหรับ Lovable

## When to Use

- User อยากสร้างร้านค้าออนไลน์หรือ e-commerce store ด้วย Lovable
- User พูดถึง product catalog, shopping cart, หรือ checkout
- User ต้องการรับชำระเงินผ่าน Stripe
- User ใช้ภาษาไทยในการอธิบายร้านค้า

## Input Schema

```yaml
store_name: string          # ชื่อร้าน
product_type: string        # ประเภทสินค้า เช่น "เทียนหอม", "คอร์สออนไลน์", "เสื้อผ้า"
product_fields: list        # field ของสินค้า เช่น [name, price, description, images, category, stock]
has_variants: boolean       # สินค้ามี variant เช่น สี/ขนาด — default: false
variant_options: list       # เช่น [color, size] ถ้า has_variants: true
has_categories: boolean     # มีหมวดหมู่สินค้า — default: true
payment_method: string      # "stripe" | "prompt_pay" | "both"
has_admin: boolean          # มีหน้า admin จัดการสินค้า — default: true
```

## Workflow

### Step 1: ทำความเข้าใจร้านค้า

ถ้าไม่รู้ `product_type` หรือ `payment_method` ให้ถามก่อน — สองอย่างนี้กำหนด chain 2 และ 3 ทั้งหมด

### Step 2: วาง 4 Chain

```
Chain 1 → โครงสร้าง: Navbar, footer, /shop grid เปล่า, /product/:id เปล่า
Chain 2 → ฟีเจอร์หลัก: Product cards, cart UI, checkout form (mock data)
Chain 3 → จัดการข้อมูล: Supabase products table, CartContext, Stripe checkout
Chain 4 → เก็บรายละเอียด: Image lazy load, stock badge, mobile, SEO
```

### Step 3: เขียน Prompt แต่ละ Chain

---

**Chain 1 — โครงสร้าง**

```
สร้าง layout shell สำหรับร้าน [ชื่อร้าน]

Tech stack:
- React + Vite + TypeScript
- Tailwind CSS + shadcn/ui
- React Router v6
- Lucide React

Navbar:
- โลโก้ "[ชื่อร้าน]" (font-bold) ซ้าย
- Nav links: [หมวดหมู่ถ้ามี], About, Contact
- Cart icon (ShoppingBag จาก Lucide) ขวาสุด พร้อม badge "0"
- Sticky เมื่อ scroll

Footer:
- โลโก้ + tagline
- Link columns: Shop, Company, Support
- Bottom bar: copyright + Privacy + Terms
- bg-gray-900 text-white

สร้าง routes เปล่า:
- /shop → Grid product placeholder 6 ใบ (gray-100 bg, h-64)
- /product/:id → placeholder centered
- /cart → placeholder centered
- /checkout → placeholder centered
[ถ้า has_admin:] /admin → placeholder centered (จะ protect ทีหลัง)

เสร็จเมื่อ:
- [ ] Navbar render พร้อม cart icon badge "0"
- [ ] Footer render ครบ
- [ ] /shop, /product/:id, /cart, /checkout render ได้
- [ ] Navbar sticky ทำงาน
- [ ] Mobile hamburger เปิด/ปิดได้
```

---

**Chain 2 — ฟีเจอร์หลัก**

```
เพิ่ม product catalog, cart, และ checkout UI ให้ [ชื่อร้าน] ด้วย mock data

ใช้ const mockProducts = [...] hardcode ในไฟล์ก่อน

หน้า /shop:
- Product grid: 3 cols desktop, 2 tablet, 1 mobile
- Product card: รูป 4:3 (placeholder gray-100), ชื่อ, ราคา "฿X,XXX", ปุ่ม "เพิ่มลงตะกร้า"
- [ถ้า has_categories:] Filter bar ด้านบน: All / [หมวดหมู่ต่างๆ] — filter ได้ใน mock
- Empty state เมื่อ filter ไม่มีสินค้า

หน้า /product/:id:
- รูปหลักขนาดใหญ่ + thumbnails row (placeholder)
- ชื่อสินค้า h1, ราคา, description
- [ถ้า has_variants:] Selector: [color/size options] — เลือกได้, อัปเดตราคาถ้าต่างกัน
- Quantity stepper: − [n] +
- ปุ่ม "เพิ่มลงตะกร้า" เต็ม width (indigo-600)
- In-stock / Out of stock badge

Cart (slide-over panel, เปิดจาก cart icon):
- รายการสินค้า: thumbnail + ชื่อ + ราคา + quantity stepper + ลบ
- Subtotal, Shipping (TBD), Total
- ปุ่ม "ดำเนินการชำระเงิน" → /checkout

หน้า /checkout:
- ฟอร์มที่อยู่: ชื่อ, เบอร์โทร, บ้านเลขที่, แขวง/ตำบล, เขต/อำเภอ, จังหวัด, รหัสไปรษณีย์
- Order summary sidebar (sticky บน desktop)
- ปุ่ม "ชำระเงิน" → console.log ก่อน (จะ connect Stripe ใน chain ถัดไป)

CartContext (src/context/CartContext.tsx):
- state: items[], addItem, removeItem, updateQty, clearCart, total
- persist ใน localStorage

เสร็จเมื่อ:
- [ ] /shop แสดง product cards จาก mock data
- [ ] Filter หมวดหมู่ทำงานใน mock
- [ ] เพิ่มสินค้า → cart badge อัปเดต
- [ ] Cart slide-over แสดงรายการและ total
- [ ] /checkout แสดงฟอร์มและ order summary
- [ ] Quantity +/- ทำงานใน cart
```

---

**Chain 3 — จัดการข้อมูล**

```
เชื่อม [ชื่อร้าน] กับ Supabase และ Stripe

1. Supabase Schema:

products (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  description text,
  price integer not null,           -- เก็บเป็น satang/สตางค์ (คูณ 100)
  images text[] default '{}',
  category text,
  stock integer not null default 0,
  is_active boolean default true,
  created_at timestamptz default now()
)
RLS: SELECT เปิดสำหรับทุกคน (public)
     INSERT/UPDATE/DELETE เฉพาะ role = 'admin'

[ถ้า has_variants:]
product_variants (
  id uuid primary key default gen_random_uuid(),
  product_id uuid references products on delete cascade,
  [option_fields เช่น color text, size text],
  price_modifier integer default 0,
  stock integer not null default 0
)

orders (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users,
  stripe_session_id text unique,
  status text default 'pending',   -- pending / paid / shipped / cancelled
  total integer not null,
  shipping_address jsonb,
  created_at timestamptz default now()
)
order_items (
  id uuid primary key default gen_random_uuid(),
  order_id uuid references orders on delete cascade,
  product_id uuid references products,
  quantity integer not null,
  unit_price integer not null
)
RLS: SELECT/UPDATE เฉพาะ user_id = auth.uid() สำหรับ orders/order_items

2. Connect /shop กับ Supabase:
- แทน mockProducts ด้วย query: SELECT * FROM products WHERE is_active = true
- Filter หมวดหมู่: .eq('category', selected)

3. Stripe Checkout:
- ติดตั้ง Stripe: Supabase Edge Function "create-checkout-session"
  - รับ: items[] (product_id, qty, price)
  - สร้าง Stripe Checkout Session (mode: 'payment')
  - line_items จาก cart
  - success_url: [domain]/checkout/success?session_id={CHECKOUT_SESSION_ID}
  - cancel_url: [domain]/checkout
- ปุ่ม "ชำระเงิน" → เรียก Edge Function → redirect ไป Stripe
- หน้า /checkout/success: verify session, บันทึก order ใน Supabase, เคลียร์ cart
- หน้า /checkout/cancel: แสดง "ยกเลิกการชำระเงิน" พร้อม back button

[ถ้า has_admin:]
4. Admin page /admin/products (protect เฉพาะ role = 'admin'):
- ตาราง product ทั้งหมด: ชื่อ, ราคา, stock, is_active toggle
- ปุ่ม "เพิ่มสินค้า" → modal form ทุก field
- Edit inline หรือ slide-over
- Upload รูป → Supabase Storage bucket "product-images"

เสร็จเมื่อ:
- [ ] /shop ดึงสินค้าจาก Supabase ได้
- [ ] Cart + Stripe → redirect ไป Stripe Checkout จริง
- [ ] หลัง payment สำเร็จ order บันทึกใน Supabase
- [ ] /checkout/success แสดง order summary
- [ ] Admin เพิ่ม/แก้ไขสินค้าได้ (ถ้า has_admin)
```

---

**Chain 4 — เก็บรายละเอียด**

```
เพิ่ม UX, performance, และ polish ให้ [ชื่อร้าน]

Images:
- Product card: loading="lazy", width + height explicit
- Product detail หลัก: fetchpriority="high"
- Supabase Storage URL: ใช้ transform API สำหรับ resize (width: 400, quality: 80)

Stock:
- stock > 10: ไม่แสดงอะไร
- stock 1-10: badge "เหลือ [n] ชิ้น" สีเหลือง
- stock 0: badge "สินค้าหมด" สีแดง, disable ปุ่ม "เพิ่มลงตะกร้า"

Loading states:
- /shop: skeleton cards 6 ใบ animate-pulse ขณะโหลด
- Product detail: skeleton ทั้งหน้า
- "เพิ่มลงตะกร้า" ปุ่ม: spinner ขณะ process

Mobile:
- Product grid: 2 cols บน mobile 375px
- Cart slide-over: full-screen บน mobile
- Checkout: ฟอร์ม stacks, summary อยู่บนสุดบน mobile

SEO (react-helmet-async):
- /shop: title "[ชื่อร้าน] — ร้านค้าออนไลน์"
- /product/:id: title "[ชื่อสินค้า] — [ชื่อร้าน]", og:image: product image URL
- Canonical URL ทุกหน้า

Error:
- สินค้าหมด + ยังอยู่ใน cart → แจ้ง "สินค้าหมดแล้ว" เมื่อเปิด cart
- Stripe error → แสดง "ชำระเงินไม่สำเร็จ กรุณาลองใหม่" ไม่ใช่ blank หน้า

เสร็จเมื่อ:
- [ ] รูปทุกรูปมี lazy loading (ยกเว้น above-the-fold)
- [ ] Stock badge แสดงถูกต้อง
- [ ] Skeleton loading แสดงบน /shop
- [ ] Mobile layout ใช้ได้บน 375px
- [ ] SEO meta tags ครบทุกหน้า
```

## Output Schema

```yaml
plan_title: string          # "[Store Name] — E-Commerce Build Plan"
store_summary: string       # สรุปหนึ่งประโยค
chains:
  - number: integer
    name: string
    goal: string
    prompt: string
    done_when: list
total_chains: 4
```

## Output Format

```
# [ชื่อร้าน] — E-Commerce Build Plan
> [สรุปหนึ่งประโยค]

---

## Chain 1 — โครงสร้าง
**เป้าหมาย:** Navbar, footer, routes เปล่าทุกหน้า

[PROMPT]

**เสร็จเมื่อ:** ...

---
(Chain 2, 3, 4)
---

**รวม 4 Chains**
Paste Chain 1 เข้า Lovable ✓ ทุก checkbox แล้วไป Chain 2
```

## Error Handling

- **ไม่รู้ payment method** — ถามก่อน เพราะกำหนด Chain 3 ทั้งหมด
- **ต้องการ PromptPay** — ใช้ Stripe Payment Element (รองรับ PromptPay) แทน Stripe Checkout
- **ไม่มี admin** — ข้ามส่วน admin ใน Chain 3, รวม 4 chains เหมือนเดิม

## Examples

### ตัวอย่าง: ร้านขายเทียนหอม "Wax & Wick"

**Input:** store: Wax & Wick, product_type: เทียนหอม, variants: [scent, size], categories: true, payment: stripe, admin: true

- Chain 1: Navbar (logo, หมวดหมู่ dropdown, cart icon), footer, /shop /product/:id /cart /checkout /admin
- Chain 2: Product grid 3 cols mock, Scent/Size selector, Cart slide-over, Checkout address form
- Chain 3: Supabase products + product_variants + orders + order_items, Stripe Checkout Edge Function, Admin CRUD + image upload
- Chain 4: Lazy loading รูป, stock badges, skeleton loading, mobile 2-col grid, SEO meta
