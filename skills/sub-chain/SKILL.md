---
name: lovable-sub-chain-planner
description: "รับ idea แอปเป็นภาษาไทย แล้วสร้าง prompt แบบ sub-chain pattern สำหรับ Lovable: 1 prompt = 1 component เพื่อลด regression และให้ Lovable แตะแค่ไฟล์เดียวต่อครั้ง"
---

# Lovable Sub-Chain Planner (v2)

รับ idea แอปจาก user แล้วแตก prompt เป็น sub-chain ย่อยๆ แต่ละ sub-chain = 1 component เท่านั้น prompt สั้น regression น้อย ตอบกลับทุกอย่างเป็นภาษาไทย รวมถึง prompt ที่สร้างสำหรับ Lovable

## When to Use

- User ต้องการลด regression ใน Lovable
- User เคยใช้ 4-chain แล้ว Lovable พัง component เก่า
- User อยากควบคุม Lovable ได้ละเอียดขึ้น
- User สร้างแอปซับซ้อนที่มีหลาย component

## หลักการสำคัญ

**1 prompt = 1 component = Lovable แตะแค่ไฟล์เดียว**

Lovable regression เกิดเมื่อ prompt ใหญ่เกิน → Lovable แก้หลายไฟล์พร้อมกัน → component เก่าพัง

```
❌ แบบเก่า (regression สูง):
Chain 2 — ฟีเจอร์หลัก
  TaskList + TaskCard + TaskForm + TaskDetail ทั้งหมดในครั้งเดียว

✅ แบบ sub-chain (regression ต่ำ):
Chain 2.1 — TaskList     ← verify แล้วค่อยไปต่อ
Chain 2.2 — TaskCard     ← verify แล้วค่อยไปต่อ
Chain 2.3 — TaskForm     ← verify แล้วค่อยไปต่อ
Chain 2.4 — TaskDetail   ← verify แล้วค่อยไปต่อ
```

**ขนาด prompt ที่ดี:** 5-15 บรรทัด

## Input Schema

```yaml
app_name: string          # ชื่อแอป
app_description: string   # แอปทำอะไร สำหรับใคร
core_features: list       # ฟีเจอร์หลัก 3-5 อย่าง
has_auth: boolean         # ต้องการ login — default: true
has_payment: boolean      # มีชำระเงิน — default: false
app_type: string          # saas / landing / dashboard / ecommerce / form
```

## Workflow

### Step 1: ทำความเข้าใจ idea

ถ้าไม่รู้ `core_features` ถามก่อน — "ฟีเจอร์หลัก 3 อย่างที่ user จะทำในแอปนี้คืออะไร?"

### Step 2: แตก Component

ก่อนเขียน prompt ให้ list component ทั้งหมดก่อน:

```
ฟีเจอร์ "จัดการ Task" → แตกเป็น:
  TaskList    — แสดงรายการ
  TaskCard    — การ์ดแต่ละรายการ
  TaskForm    — ฟอร์มสร้าง/แก้ไข
  TaskDetail  — รายละเอียด

ฟีเจอร์ "Dashboard" → แตกเป็น:
  StatsCards  — KPI 4 ใบ
  TaskChart   — กราฟสถิติ
```

**กฎแตก component:**
- 1 component = 1 UI ชิ้น มี props และ state ของตัวเอง
- component ใหญ่เกิน (หลาย interaction) → แตกต่อ
- component เล็กมาก (แค่ text) → รวมกับ parent ได้

### Step 3: วาง Sub-Chain Plan

```
Chain 1 — โครงสร้าง
  Sidebar/Navbar + routes เปล่าทุกหน้า (1 prompt รวม)

Chain 2.1 — [Component 1 ของฟีเจอร์ A]
Chain 2.2 — [Component 2 ของฟีเจอร์ A]
Chain 2.3 — [Component 1 ของฟีเจอร์ B]
Chain 2.4 — [Component 2 ของฟีเจอร์ B]
...ต่อจนครบทุก component (mock data ทั้งหมด)

Chain 3.1 — Auth (Login/Signup + ProtectedRoute)
Chain 3.2 — เชื่อม Supabase: [ฟีเจอร์ A]
Chain 3.3 — เชื่อม Supabase: [ฟีเจอร์ B]
...ต่อจนครบทุก feature

Chain 4 — เก็บรายละเอียด
  Loading, error, empty states, mobile, polish (รวมทีเดียว)
```

**ลำดับที่ห้ามสลับ:**
- Chain 1 ก่อนเสมอ
- Chain 2.x ทั้งหมดก่อน Chain 3.x
- Chain 4 หลังสุดเสมอ
- ภายใน 2.x: component parent ก่อน child

### Step 4: เขียน Prompt แต่ละ Sub-Chain

**Template:**
```
สร้าง [ชื่อ Component] สำหรับ [ชื่อแอป]

- [spec ข้อ 1: layout หรือ UI หลัก]
- [spec ข้อ 2: ข้อมูลที่แสดง — mock data]
- [spec ข้อ 3: interaction ถ้ามี]
- [spec ข้อ 4: design token เช่น สี, ขนาด]

เสร็จเมื่อ:
- [ ] [ผลลัพธ์ตรวจสอบได้]
- [ ] [ผลลัพธ์ตรวจสอบได้]
```

**ห้ามใส่ในครั้งเดียว:**
- หลาย component
- UI + Supabase logic
- สร้างใหม่ + แก้ของเก่า

### Step 5: ส่ง Plan

แสดง sub-chain ทั้งหมด พร้อม prompt แต่ละตัว บอก user ให้ verify ทุก checkbox ก่อนไป sub-chain ถัดไป

## Output Schema

```yaml
plan_title: string
app_summary: string
chains:
  - id: string             # "1", "2.1", "2.2", "3.1" ฯลฯ
    name: string           # ชื่อ component
    goal: string           # 1 ประโยค
    prompt: string         # พร้อม paste, 5-15 บรรทัด
    done_when: list        # 2-3 ข้อ
total_chains: integer
```

## Output Format

```
# [ชื่อแอป] — Sub-Chain Build Plan
> [สรุปหนึ่งประโยค]

---

## Chain 1 — โครงสร้าง
**เป้าหมาย:** Layout shell + routes เปล่าทุกหน้า

[PROMPT]

**เสร็จเมื่อ:**
- [ ] ...

---

## Chain 2.1 — [Component]
**เป้าหมาย:** [1 ประโยค]

[PROMPT สั้น 5-15 บรรทัด]

**เสร็จเมื่อ:**
- [ ] ...

---

## Chain 2.2 — [Component]
**เป้าหมาย:** [1 ประโยค]

[PROMPT]

**เสร็จเมื่อ:**
- [ ] ...

---

(ต่อจนครบทุก sub-chain)

---

**รวม [N] Sub-Chains**
Paste Chain 1 → ✓ ทุก checkbox → Chain 2.1 → ✓ → Chain 2.2 → ...
อย่าข้าม sub-chain ถ้ายัง verify ไม่ผ่าน
```

## Error Handling

- **Idea ไม่ชัด** — ถามก่อน อย่าแตก component จนกว่าจะรู้ฟีเจอร์หลัก
- **User อยากรวม 2 component** — อธิบายว่า regression จะสูงขึ้น แนะนำให้แยก
- **Component มี sub-component ลึกมาก** — แตกได้เป็น 2.1.1, 2.1.2 ถ้าจำเป็น
- **Chain 2.x มีเยอะมาก** — ปกติ 6-12 sub-chains คือดี ถ้าเกิน 15 อาจ scope แอปใหญ่เกิน

## Examples

### ตัวอย่าง 1: Task Manager "Taskly"

**Input:** "อยากทำแอป task manager ชื่อ Taskly"

**Component list:**
```
Layout: Sidebar, routes
Tasks: TaskList, TaskCard, TaskForm, TaskDetail
Dashboard: StatsCards, TaskChart
```

**Sub-Chain Plan:**
```
Chain 1    — Layout: Sidebar + /tasks, /dashboard, /settings routes

Chain 2.1  — TaskList: แสดง mock tasks[], filter All/Active/Done
Chain 2.2  — TaskCard: card + priority badge + checkbox complete + due date
Chain 2.3  — TaskForm: modal สร้าง task (ชื่อ, priority, due date)
Chain 2.4  — TaskDetail: slide-over แสดง field ทุกตัว + Edit/Delete

Chain 2.5  — StatsCards: 4 KPI cards (total, active, done, overdue) mock
Chain 2.6  — TaskChart: Bar chart tasks รายวัน 7 วัน (Recharts, mock)

Chain 3.1  — Auth: /login, /signup, useAuth(), ProtectedRoute /app/*
Chain 3.2  — Tasks → Supabase: tasks table + RLS, แทน mock ด้วย query จริง, CRUD
Chain 3.3  — Dashboard → Supabase: query COUNT/aggregate จาก tasks table จริง

Chain 4    — Skeleton loading, empty states, error toasts, mobile 375px
```

**รวม 11 Sub-Chains**

---

### ตัวอย่าง 2: ระบบบันทึกเวลา

**Input:** "อยากทำระบบ Clock In/Out คำนวณค่าแรง"

**Sub-Chain Plan:**
```
Chain 1    — Layout: Sidebar + /clock, /history, /summary, /settings

Chain 2.1  — ClockButton: ปุ่ม Clock In/Out + สถานะ + นาฬิกา real-time
Chain 2.2  — TodaySummary: ชั่วโมงวันนี้ + ค่าแรงวันนี้ (mock)
Chain 2.3  — HistoryTable: ตารางประวัติ วันที่/เข้า/ออก/ชม/ค่าแรง (mock)
Chain 2.4  — HistoryFilter: month picker + summary bar (รวมชม/เงิน)
Chain 2.5  — SummaryKPI: 4 KPI cards วันทำงาน/ชม/เฉลี่ย/ค่าแรงรวม (mock)
Chain 2.6  — SummaryCharts: Bar chart ค่าแรงรายวัน + Line chart ชม. (Recharts)
Chain 2.7  — SettingsForm: ฟอร์มค่าแรง/ชม, OT rate, วันหยุด, เวลาเข้า-ออก

Chain 3.1  — Auth: login + profiles table (hourly_rate, ot_multiplier ฯลฯ)
Chain 3.2  — ClockIn/Out → Supabase: time_entries table + คำนวณ OT จริง
Chain 3.3  — History → Supabase: query จริงตาม month filter + edit entry
Chain 3.4  — Summary → Supabase: aggregate query KPI + chart data จริง
Chain 3.5  — Settings → Supabase: UPDATE profiles ค่าแรง/OT/วันหยุด

Chain 4    — Skeleton, empty states, mobile, error handling
```

**รวม 15 Sub-Chains**
