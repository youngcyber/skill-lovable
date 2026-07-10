---
name: lovable-forms-planner
description: "รับ idea form แล้วสร้าง prompt แบบ chain pattern 4 ชั้นสำหรับ Lovable: โครงสร้าง → UI & validation → บันทึกข้อมูล → เก็บรายละเอียด รองรับภาษาไทย"
---

# Lovable Forms Planner

รับ idea form จาก user แล้วสร้าง chain prompt พร้อม paste เข้า Lovable ทีละ chain ตอบกลับทุกอย่างเป็นภาษาไทย รวมถึง prompt ที่สร้างสำหรับ Lovable

## When to Use

- User อยากสร้างฟอร์มสมัคร, ฟอร์มติดต่อ, ฟอร์มกรอกข้อมูล, หรือ multi-step wizard
- User ต้องการ validation, conditional fields, หรือ file upload
- User ต้องการบันทึกข้อมูลฟอร์มเข้า Supabase หรือส่ง email notification
- User ใช้ภาษาไทยในการอธิบาย form

## Input Schema

```yaml
form_name: string           # ชื่อ form เช่น "ฟอร์มสมัครงาน"
form_purpose: string        # ใช้ทำอะไร เช่น "รับใบสมัครงาน"
fields: list                # field ทุกตัว พร้อม type + required/optional
  # - { name: string, type: text|email|tel|number|date|select|radio|checkbox|file|textarea, required: boolean, options: list (สำหรับ select/radio) }
is_multistep: boolean       # แบ่งเป็นหลาย step — default: false
steps: list                 # ถ้า is_multistep: true: [{step: 1, title: "", fields: []}]
submit_action: string       # "save_db" | "send_email" | "both"
email_recipient: string     # email ที่รับ notification ถ้า submit_action มี send_email
has_file_upload: boolean    # มี file upload ไหม — default: false
allowed_file_types: list    # เช่น [pdf, docx, jpg] ถ้า has_file_upload: true
```

## Workflow

### Step 1: ทำความเข้าใจ form

ถ้าไม่รู้ `fields` ให้ถามก่อน — "field ที่ต้องกรอกในฟอร์มนี้มีอะไรบ้าง?"

### Step 2: วาง 4 Chain

```
Chain 1 → โครงสร้าง: Form card UI เปล่าๆ ทุก field แต่ยังไม่มี logic
Chain 2 → ฟีเจอร์หลัก: Validation (react-hook-form + zod), submit ใน mock
Chain 3 → จัดการข้อมูล: บันทึก Supabase และ/หรือ ส่ง email (Resend)
Chain 4 → เก็บรายละเอียด: Loading, success state, error, accessibility, mobile
```

### Step 3: เขียน Prompt แต่ละ Chain

---

**Chain 1 — โครงสร้าง**

```
สร้าง [ชื่อ form] UI สำหรับ [form_purpose]

Tech stack:
- React + Vite + TypeScript
- Tailwind CSS + shadcn/ui
- Lucide React

[ถ้าไม่ใช่ multi-step:]
Layout:
- Card: white bg, rounded-2xl, border, max-w-[600px] mx-auto p-8
- ชื่อ form: h1 text-2xl font-bold บนสุด
- ทุก field ตาม fields ที่กำหนด:

[สร้างทีละ field:]
- TEXT field: Label + Input (shadcn Input) + placeholder เหมาะสม
- TEXTAREA field: Label + Textarea (shadcn) rows=4
- SELECT field: Label + shadcn Select + ตัวเลือก: [options]
- RADIO field: Label + shadcn RadioGroup + ตัวเลือก: [options] — horizontal layout
- CHECKBOX field: shadcn Checkbox + label text
- DATE field: Label + shadcn Input type="date"
- FILE field: Label + drag-drop zone (dashed border, rounded-xl, text-center) + หรือ click to browse
- EMAIL/TEL/NUMBER: Label + shadcn Input type ที่ถูกต้อง

- Error placeholder: พื้นที่ text-red-500 text-sm ใต้แต่ละ field (ว่างไว้ก่อน)
- ปุ่ม Submit: "[ชื่อ action]" เต็ม width indigo-600 — ยังไม่ต้องทำอะไร

[ถ้าเป็น multi-step:]
- Progress bar ด้านบน: [step_count] steps, filled ตาม current step
- Step indicator: "ขั้นตอน 1 จาก [N]: [step_title]"
- แสดงเฉพาะ fields ของ step ปัจจุบัน
- ปุ่ม "ถัดไป" / "ย้อนกลับ" / "ส่งฟอร์ม" ตาม step
- ปุ่ม "ถัดไป" ยังไม่ validate ใน chain นี้

เสร็จเมื่อ:
- [ ] ทุก field render ถูกต้องตาม type
- [ ] Error placeholder พื้นที่ว่างใต้ทุก field
- [ ] ปุ่ม submit แสดง (ยังไม่ทำอะไร)
- [ ] [ถ้า multi-step:] Progress bar และ step navigation render ได้
- [ ] Layout responsive บน 375px
```

---

**Chain 2 — ฟีเจอร์หลัก**

```
เพิ่ม validation และ interactions ให้ [ชื่อ form]

ติดตั้ง: react-hook-form + zod + @hookform/resolvers

Zod schema (src/lib/schemas/[form-name].ts):
[แต่ละ field ตามที่กำหนด:]
- text required: z.string().min(1, "[label] จำเป็นต้องกรอก")
- text optional: z.string().optional()
- email: z.string().email("รูปแบบ email ไม่ถูกต้อง")
- tel: z.string().regex(/^[0-9]{9,10}$/, "เบอร์โทรต้องเป็นตัวเลข 9-10 หลัก")
- number: z.coerce.number().min([min], "[message]")
- select required: z.string().min(1, "กรุณาเลือก[label]")
- checkbox required: z.boolean().refine(v => v, "กรุณายอมรับข้อกำหนด")
- file: z.instanceof(FileList).refine(f => f.length > 0, "กรุณาแนบไฟล์")

[ถ้ามี conditional field:]
- ใช้ .superRefine() หรือ watch() ซ่อน/แสดง field ตามค่า field อื่น

กฎ validation:
- แสดง error เฉพาะเมื่อกด submit ครั้งแรก
- หลังจากนั้น error หายทันทีเมื่อแก้ไข field นั้น
- Input ที่ error: border-red-500 ring-red-500
- Error text: text-red-500 text-sm mt-1 ใต้ field

Submit handler:
- onSubmit: console.log(data) ก่อน — ยังไม่ส่งไปไหน
- ปุ่ม: disabled + Loader2 spinner ขณะ submitting (setTimeout 1.5s mock)

[ถ้า multi-step:]
- ปุ่ม "ถัดไป": trigger validation เฉพาะ fields ของ step ปัจจุบัน
  (ใช้ trigger([...fieldNames]) จาก react-hook-form)
- ถ้า validate ผ่าน → ไป step ถัดไป
- "ย้อนกลับ" → ไม่ต้อง validate

เสร็จเมื่อ:
- [ ] Submit โดยไม่กรอก required field → error ทุก field ที่ขาด
- [ ] แก้ไข field ที่ error → error หายทันที
- [ ] Email format ผิด → แสดง error
- [ ] ปุ่มแสดง spinner เมื่อ submit
- [ ] console.log(data) แสดงข้อมูลถูกต้องหลัง validate ผ่าน
```

---

**Chain 3 — จัดการข้อมูล**

```
เชื่อม [ชื่อ form] กับ backend

[ถ้า submit_action: "save_db" หรือ "both":]
1. Supabase table:

[form_name]_submissions (
  id uuid primary key default gen_random_uuid(),
  [แต่ละ field เป็น column ที่ type ถูกต้อง],
  submitted_at timestamptz default now(),
  [ถ้า user login:] user_id uuid references auth.users
)
RLS:
- INSERT: เปิดสำหรับทุกคน (ถ้าเป็น public form) หรือ user_id = auth.uid()
- SELECT: เฉพาะ authenticated admin (role = 'admin')

Connect form:
- แทน console.log ด้วย supabase.from('[table]').insert([data])
- map form values → column ให้ถูกต้อง

[ถ้า has_file_upload:]
File upload:
- Upload ไฟล์ไปยัง Supabase Storage bucket "[form-name]-files"
- ชื่อไฟล์: [uuid]-[original-name]
- บันทึก storage path ในตาราง
- Validate ฝั่ง client: ขนาดไม่เกิน 5MB, type เฉพาะ [allowed_file_types]

[ถ้า submit_action: "send_email" หรือ "both":]
Email notification:
- Supabase Edge Function "send-form-notification"
  - รับ: form data
  - ส่ง email ผ่าน Resend ไปยัง [email_recipient]
  - Subject: "ฟอร์มใหม่: [ชื่อ form]"
  - Body: HTML table แสดงทุก field + ค่า
- เรียก Edge Function หลัง INSERT สำเร็จ (ไม่เรียกถ้า INSERT fail)

เสร็จเมื่อ:
- [ ] Submit บันทึกข้อมูลใน Supabase จริง
- [ ] ตรวจสอบ row ใน Supabase dashboard หลัง submit
- [ ] [ถ้ามี file:] ไฟล์ปรากฏใน Supabase Storage
- [ ] [ถ้ามี email:] ได้รับ email notification ที่ [email_recipient]
```

---

**Chain 4 — เก็บรายละเอียด**

```
เพิ่ม UX, success/error states, และ accessibility ให้ [ชื่อ form]

Success state:
- หลัง submit สำเร็จ: แทน form ด้วย success card
  - CheckCircle icon (indigo-600)
  - "[ข้อความยืนยัน เช่น 'ส่งฟอร์มสำเร็จแล้ว!']"
  - "[ขั้นตอนต่อไป เช่น 'เราจะติดต่อกลับภายใน 3 วันทำการ']"
  - [ถ้าต้องการ:] ปุ่ม "ส่งฟอร์มอีกครั้ง" reset กลับ form ว่าง
- ห้าม redirect — แสดง success ในหน้าเดิม

Error state:
- Supabase INSERT fail: แสดง error card สีแดงเหนือปุ่ม submit
  "เกิดข้อผิดพลาด กรุณาลองใหม่อีกครั้ง"
  + ปุ่ม "ลองอีกครั้ง" — ไม่เคลียร์ข้อมูลที่กรอกไว้

Loading:
- ระหว่าง submit: ทุก field disabled + ปุ่ม "กำลังส่ง..." + Loader2 spinner
- [ถ้ามี file upload:] progress bar แสดง % upload

Accessibility:
- ทุก input มี id + htmlFor เชื่อมกัน
- Required field มี aria-required="true"
- Error message มี role="alert" และ aria-describedby เชื่อมกับ input
- ปุ่ม submit ที่ disabled มี aria-disabled="true"
- ใช้ Tab navigate ผ่านทุก field ได้ตามลำดับ

Mobile:
- ทุก field แสดงชัดเจนบน 375px
- ปุ่มสูงอย่างน้อย 48px (touch target)
- File upload drop zone แสดงชัดเจน

เสร็จเมื่อ:
- [ ] Submit สำเร็จ → form เปลี่ยนเป็น success card
- [ ] Submit fail → error card แสดง + ข้อมูลยังอยู่ใน field
- [ ] ทุก field disabled ระหว่าง submit
- [ ] Tab key navigate ผ่านทุก field ได้
- [ ] ใช้งานได้บน 375px mobile
```

## Output Schema

```yaml
plan_title: string          # "[Form Name] — Build Plan"
form_summary: string        # สรุปหนึ่งประโยค
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
# [ชื่อ Form] — Build Plan
> [สรุปหนึ่งประโยค]

---

## Chain 1 — โครงสร้าง
**เป้าหมาย:** Form UI shell ทุก field ก่อน logic

[PROMPT]

**เสร็จเมื่อ:** ...

---
(Chain 2, 3, 4)
---

**รวม 4 Chains**
Paste Chain 1 เข้า Lovable ✓ ทุก checkbox แล้วไป Chain 2
```

## Error Handling

- **ไม่รู้ fields** — ถามก่อน: "field ที่ต้องกรอกในฟอร์มนี้มีอะไรบ้าง?"
- **Multi-step แต่ไม่รู้ step** — ถามว่าแบ่งเป็นกี่ขั้นตอน และแต่ละขั้นมี field อะไร
- **ต้องการ send email แต่ไม่มี Resend** — แนะนำใช้ Supabase Edge Function + Resend (free tier 100 email/วัน)

## Examples

### ตัวอย่าง: ฟอร์มสมัครงาน 3 ขั้นตอน

**Input:** form: ฟอร์มสมัครงาน, purpose: รับใบสมัครตำแหน่ง Developer, multi-step: true, steps: [{1: "ข้อมูลส่วนตัว", fields: [ชื่อ, นามสกุล, email, tel]}, {2: "ประสบการณ์", fields: [ตำแหน่งที่สมัคร (select), ประสบการณ์ (number), ทักษะ (checkbox multi)]}, {3: "เอกสาร", fields: [Resume (file), portfolio URL (text optional)]}], submit: both, email: hr@company.com, file: [pdf, docx]

- Chain 1: 3-step form UI, progress bar, ทุก field ตาม step, navigation "ถัดไป"/"ย้อนกลับ"
- Chain 2: Validate แต่ละ step ก่อนไปหน้าถัดไป, file type/size validation, submit → console.log
- Chain 3: Supabase applications table, Supabase Storage สำหรับ resume, Resend email notification
- Chain 4: Success card "ส่งใบสมัครสำเร็จ", error ไม่เคลียร์ข้อมูล, accessibility ครบ, mobile 375px
