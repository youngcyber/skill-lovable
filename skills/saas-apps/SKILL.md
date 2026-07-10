---
name: lovable-saas-planner
description: "รับ idea SaaS แล้วสร้าง prompt แบบ chain pattern 4 ชั้นสำหรับ Lovable: โครงสร้าง → ฟีเจอร์หลัก → จัดการข้อมูล → เก็บรายละเอียด รองรับภาษาไทย"
---

# Lovable SaaS Planner

รับ idea SaaS จาก user แล้วสร้าง chain prompt พร้อม paste เข้า Lovable ทีละ chain ตอบกลับทุกอย่างเป็นภาษาไทย รวมถึง prompt ที่สร้างสำหรับ Lovable

## When to Use

- User พูดถึงการสร้าง SaaS, web app, หรือแอปที่ต้องการ login
- User บอก idea เป็นภาษาไทยและต้องการแผน build ด้วย Lovable
- User ถามว่า "จะเริ่ม build SaaS ด้วย Lovable ยังไง?"
- แอปมี user accounts, ข้อมูลที่ต้องเก็บ, และ dashboard

## Input Schema

```yaml
app_name: string           # ชื่อแอป
app_description: string    # แอปทำอะไร สำหรับใคร
core_features: list        # ฟีเจอร์หลัก 3-5 อย่าง เช่น [สร้าง project, assign task, track progress]
auth_methods: list         # วิธี login: [email, google, magic-link] — default: [email]
has_payment: boolean       # มีระบบชำระเงินไหม — default: false
plan_names: list           # ชื่อแพ็กเกจ เช่น [Free, Pro] — ต้องการถ้า has_payment: true
user_roles: list           # บทบาท: [user, admin] — default: [user]
```

## Workflow

### Step 1: ทำความเข้าใจ idea

ถ้า `core_features` ยังไม่ชัด ให้ถามว่า "ฟีเจอร์หลัก 3 อย่างที่ user จะทำในแอปนี้คืออะไร?" อย่าสร้าง chain จนกว่าจะรู้ฟีเจอร์หลัก

### Step 2: วางโครงสร้าง 4 Chain

```
Chain 1 → โครงสร้าง
Chain 2 → ฟีเจอร์หลัก
Chain 3 → จัดการข้อมูล
Chain 4 → เก็บรายละเอียด
```

### Step 3: เขียน Prompt แต่ละ Chain

---

**Chain 1 — โครงสร้าง**

สร้าง visual shell ก่อน ไม่มี logic ไม่มี database ในชั้นนี้

```
สร้าง layout shell สำหรับ [ชื่อแอป]

Tech stack (ใช้ตามนี้ทุกอย่าง ห้ามเปลี่ยน):
- React + Vite + TypeScript
- Tailwind CSS
- shadcn/ui สำหรับทุก component
- React Router v6
- Lucide React สำหรับ icon

Sidebar (desktop, fixed, 240px):
- Logo "[ชื่อแอป]" ซ้ายบน (font-bold, indigo-600)
- Nav links: [Icon → ชื่อหน้า → /route] สำหรับทุกหน้า
- Active link: bg-indigo-50 text-indigo-700 rounded-lg
- ด้านล่าง: avatar + ชื่อ user + ปุ่ม sign-out
- ย่อได้เป็น 64px icon-only บันทึกใน localStorage

Header (h-16, sticky): ชื่อหน้าซ้าย, [ปุ่มเพิ่มเติม]ขวา
Mobile: hamburger → drawer ปิดเมื่อกด nav link
Main content: bg-gray-50 p-6 — ปล่อยว่างไว้ก่อน

สร้างหน้าเปล่าที่ทุก route:
[LIST ทุกหน้า เช่น /app/dashboard, /app/projects, /app/settings]
แต่ละหน้าแสดงแค่ h1 ชื่อหน้า

เสร็จเมื่อ:
- [ ] Layout render ถูกต้องบน 375px / 768px / 1280px
- [ ] Sidebar ย่อ/ขยายได้และจำสถานะ
- [ ] ทุก nav link navigate ถูกและแสดง active state
- [ ] ทุกหน้า render ได้โดยไม่ error
```

---

**Chain 2 — ฟีเจอร์หลัก**

สร้าง UI และ interaction ของทุกฟีเจอร์ ใช้ mock data ก่อน ยังไม่ connect database

```
เพิ่มฟีเจอร์หลักให้ [ชื่อแอป] โดยใช้ mock data ก่อน

[ฟีเจอร์ที่ 1: ชื่อ] ที่ [route]:
- [รายละเอียด UI: component, layout, interaction]
- ข้อมูล: ใช้ const mockData = [...] hardcode ไว้ในไฟล์ก่อน
- [states: hover, click, empty]

[ฟีเจอร์ที่ 2: ชื่อ] ที่ [route]:
- [รายละเอียด]

[ฟีเจอร์ที่ 3: ชื่อ] ที่ [route]:
- [รายละเอียด]

ข้อกำหนด:
- ห้าม connect Supabase ในขั้นนี้
- ทุก button และ form ทำงานได้ใน UI (console.log ผลลัพธ์ได้)
- ต้อง responsive ทุก component

เสร็จเมื่อ:
- [ ] [ฟีเจอร์ 1] render และ interact ได้ด้วย mock data
- [ ] [ฟีเจอร์ 2] render และ interact ได้
- [ ] ทุก button/form ทำงานได้ใน UI
- [ ] ไม่มี Supabase call ในขั้นนี้
```

---

**Chain 3 — จัดการข้อมูล**

สร้าง database schema, auth, แล้ว connect UI กับ Supabase ทีละฟีเจอร์

```
เชื่อม [ชื่อแอป] กับ Supabase

1. Auth:
สร้างระบบ login ด้วย [method]
- หน้า /login, /signup, /forgot-password
- useAuth() hook ที่ src/hooks/useAuth.ts
  export: { user, session, loading, signIn, signUp, signOut }
- profiles table:
  profiles (id uuid references auth.users primary key,
            full_name text, avatar_url text,
            role text default 'user', onboarded boolean default false,
            created_at timestamptz default now())
  RLS: SELECT/UPDATE เฉพาะ row ของตัวเอง
- Route guard: /app/* → redirect /login ถ้าไม่ได้ login

2. Database Schema:
[TABLE 1]:
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users on delete cascade,
  [fields + types + constraints],
  created_at timestamptz default now()
  RLS: ทุก operation เฉพาะ user_id = auth.uid()

[TABLE 2 ถ้ามี]: [schema]

3. Connect ฟีเจอร์กับ database:
- [ฟีเจอร์ 1]: แทน mock data ด้วย Supabase query จาก [table]
- [ฟีเจอร์ 2]: INSERT เมื่อ submit form
- [ฟีเจอร์ 3]: UPDATE/DELETE ผ่าน row actions

เสร็จเมื่อ:
- [ ] Login/signup ทำงานได้และ redirect ถูก
- [ ] /app/* redirect ไป /login เมื่อไม่ได้ login
- [ ] [ฟีเจอร์ 1] ดึงข้อมูลจาก Supabase ได้จริง
- [ ] [ฟีเจอร์ 2] บันทึกข้อมูลเข้า Supabase ได้
- [ ] RLS block user อื่นจาก select ข้อมูล
```

---

**Chain 4 — เก็บรายละเอียด**

เพิ่ม loading, error, empty states และ polish หลังทุกฟีเจอร์ทำงานได้แล้ว

```
เพิ่ม loading states, error handling, และ polish ให้ [ชื่อแอป]

Loading states:
- ทุก component ที่ fetch data: แสดง skeleton (animate-pulse bg-gray-100)
  ให้ shape ตรงกับ content จริง ห้ามใช้ spinner กลางหน้า
- ระหว่าง submit form: ปุ่ม disabled + Loader2 spinner ข้างใน

Empty states (ทีละหน้า):
- [หน้า 1]: ไอคอน + "[ข้อความ]" + ปุ่ม CTA
- [หน้า 2]: ไอคอน + "[ข้อความ]"

Error handling:
- Supabase error: แสดง inline error + ปุ่ม "ลองอีกครั้ง" ใน section นั้น
- ห้ามแสดง blank หน้าขาว ห้าม crash
- Success: toast สีเขียว 3 วินาที (sonner หรือ shadcn Toaster)
- Error: toast สีแดง ปิดเองไม่ได้

Mobile responsive:
- ตาราง → card list บน screen < 640px
- Grid หลายคอลัมน์ → single column บน mobile
- Modal → full-screen บน mobile

Micro-animations:
- Page transition: fade-in 150ms เมื่อ navigate
- ปุ่ม: active:scale-95
- ลบ row: fade-out 400ms ก่อนออกจาก DOM

เสร็จเมื่อ:
- [ ] ทุก data-fetch component แสดง skeleton บน load
- [ ] ทุก list มี empty state
- [ ] ทุก mutation แสดง success toast
- [ ] Supabase error แสดง inline ไม่ใช่ blank หน้า
- [ ] แอปใช้ได้บน iPhone 375px
```

### Step 4: ส่ง Plan

แสดงทั้ง 4 chain พร้อมบอก user ให้ verify ก่อนไป chain ถัดไป

## Output Schema

```yaml
plan_title: string         # "[ชื่อแอป] — SaaS Build Plan"
app_summary: string        # สรุปหนึ่งประโยค
chains:
  - number: integer        # 1-4
    name: string           # โครงสร้าง / ฟีเจอร์หลัก / จัดการข้อมูล / เก็บรายละเอียด
    goal: string           # เป้าหมายของ chain
    prompt: string         # Prompt พร้อม paste
    done_when: list        # Checklist 3-5 ข้อ
total_chains: 4
```

## Output Format

```
# [ชื่อแอป] — SaaS Build Plan
> [สรุปหนึ่งประโยค]

---

## Chain 1 — โครงสร้าง
**เป้าหมาย:** Layout shell และ navigation ก่อน logic ใดๆ

[PROMPT]

**เสร็จเมื่อ:**
- [ ] ...

---

## Chain 2 — ฟีเจอร์หลัก
**เป้าหมาย:** UI ทุกฟีเจอร์ด้วย mock data

[PROMPT]

**เสร็จเมื่อ:**
- [ ] ...

---

## Chain 3 — จัดการข้อมูล
**เป้าหมาย:** Auth + Supabase schema + connect UI กับ database

[PROMPT]

**เสร็จเมื่อ:**
- [ ] ...

---

## Chain 4 — เก็บรายละเอียด
**เป้าหมาย:** Loading, error, empty states, mobile, polish

[PROMPT]

**เสร็จเมื่อ:**
- [ ] ...

---

**รวม 4 Chains**
Paste Chain 1 เข้า Lovable ก่อน ✓ ทุก checkbox แล้วค่อยไป Chain 2
```

## Error Handling

- **ไม่รู้ฟีเจอร์หลัก** — ถามก่อน: "ฟีเจอร์หลัก 3 อย่างที่ user ทำในแอปนี้คืออะไร?"
- **User อยาก connect database ใน Chain 2** — แนะนำให้ทำ UI ให้สมบูรณ์ก่อน wire ง่ายกว่า
- **has_payment: true แต่ไม่บอกชื่อ plan** — ถามก่อนเขียน Chain 3

## Examples

### ตัวอย่าง: แอปจัดการโปรเจกต์ "Taskly"

**Input:** "อยากสร้างแอป SaaS ชื่อ Taskly สำหรับจัดการ task ในทีม login ด้วย email ไม่มีระบบชำระเงิน"

**Chain 1 — โครงสร้าง:** Sidebar (Dashboard, Projects, Tasks, Settings), layout shell, 4 หน้าเปล่า

**Chain 2 — ฟีเจอร์หลัก:** Project list + สร้าง project modal (mock), Task board kanban (mock), ฟอร์มสร้าง task

**Chain 3 — จัดการข้อมูล:** Auth (email), projects + tasks table + RLS, connect ทุก UI กับ Supabase

**Chain 4 — เก็บรายละเอียด:** Skeleton loading, empty state "ยังไม่มี project", error toast, mobile layout
