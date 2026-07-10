---
name: lovable-data-management-planner
description: "รับ idea ระบบจัดการข้อมูล แล้วสร้าง prompt แบบ chain pattern 4 ชั้นสำหรับ Lovable: โครงสร้าง → ตาราง & view → CRUD & schema → เก็บรายละเอียด รองรับภาษาไทย"
---

# Lovable Data Management Planner

รับ idea ระบบจัดการข้อมูลจาก user แล้วสร้าง chain prompt พร้อม paste เข้า Lovable ทีละ chain ตอบกลับทุกอย่างเป็นภาษาไทย รวมถึง prompt ที่สร้างสำหรับ Lovable

## When to Use

- User ต้องการ CRUD table สำหรับจัดการ records เช่น contacts, products, tasks
- User ต้องการ kanban board ที่ drag การ์ดระหว่าง column ได้
- User ต้องการ filter, search, หรือ sort ข้อมูล
- User ต้องการ import/export CSV
- User ใช้ภาษาไทยในการอธิบายระบบ

## Input Schema

```yaml
entity_name: string         # ชื่อ entity เช่น "contacts", "งาน", "สินค้า"
view_type: string           # "table" | "kanban" | "gallery" | "list"
operations: list            # [create, read, update, delete, reorder]
data_fields: list           # fields พร้อม type เช่น [{name: "title", type: "text", required: true}]
kanban_stages: list         # ถ้า view_type: kanban เช่น ["รอดำเนินการ", "กำลังทำ", "เสร็จแล้ว"]
has_search: boolean         # มี search bar — default: true
has_filters: boolean        # มี filter ตาม field — default: false
has_import: boolean         # CSV import — default: false
has_export: boolean         # CSV export — default: false
is_realtime: boolean        # Supabase Realtime — default: false
```

## Workflow

### Step 1: ทำความเข้าใจ entity

ถ้าไม่รู้ `data_fields` ถามก่อน — schema เขียนไม่ได้ถ้าไม่รู้ columns

ถ้า `view_type: kanban` แต่ไม่รู้ `kanban_stages` — ถามก่อน

### Step 2: วาง 4 Chain

```
Chain 1 → โครงสร้าง: Layout, empty table/kanban shell, column headers
Chain 2 → ฟีเจอร์หลัก: Read view + mock data, search/filter UI
Chain 3 → จัดการข้อมูล: Supabase schema, CRUD operations, connect ทุก UI
Chain 4 → เก็บรายละเอียด: Loading skeleton, empty state, error, mobile, export
```

### Step 3: เขียน Prompt แต่ละ Chain

---

**Chain 1 — โครงสร้าง**

```
สร้าง layout shell สำหรับระบบจัดการ [entity_name]

Tech stack:
- React + Vite + TypeScript
- Tailwind CSS + shadcn/ui
- Lucide React
[ถ้า view_type: kanban หรือ reorder ใน operations:] - @dnd-kit/core + @dnd-kit/sortable (ติดตั้งด้วย)
[ถ้า has_import:] - papaparse (ติดตั้งด้วย)

Header section:
- ชื่อหน้า "[entity_name]" (h1, text-2xl font-bold) ซ้าย
- ขวา: [ถ้ามี search:] search input | ปุ่ม "เพิ่ม [entity]" (indigo-600)
  [ถ้า has_import:] ปุ่ม "Import CSV" | [ถ้า has_export:] ปุ่ม "Export CSV"

[ถ้า view_type: table:]
Table shell:
- shadcn Table: thead columns = [field names ตาม data_fields] + column "Actions" ขวาสุด
- tbody: แสดงแค่ placeholder row 1 ใบ "loading..."
- Pagination bar ด้านล่าง: "แสดง 1-10 จาก N รายการ" + ปุ่ม prev/next (placeholder ว่างไว้)

[ถ้า view_type: kanban:]
Kanban shell:
- Flex row: [N] columns ตาม kanban_stages
- แต่ละ column: header "[stage name]" + badge "0" + พื้นที่ว่าง min-h-[400px] bg-gray-50 rounded-xl
- "เพิ่มการ์ด" button ด้านล่างแต่ละ column — ว่างไว้ก่อน

[ถ้า has_filters:]
Filter bar ใต้ header (hidden บน mobile, toggle เปิด/ปิด):
- Filter ตาม [filter fields] — placeholder dropdowns ว่างไว้ก่อน

เสร็จเมื่อ:
- [ ] Header render พร้อมปุ่มทั้งหมด
- [ ] [Table:] thead columns ถูกต้อง, 1 placeholder row แสดง
- [ ] [Kanban:] N column render ตาม stages
- [ ] Layout responsive บน 375px / 1280px
- [ ] ยังไม่มี Supabase call ใดๆ
```

---

**Chain 2 — ฟีเจอร์หลัก**

```
เพิ่ม read view และ interactions ให้ระบบจัดการ [entity_name] ด้วย mock data

ใช้ const mock[EntityName] = [...] hardcode ไว้ก่อน — 5-8 รายการ

[ถ้า view_type: table:]
Table rows:
[แต่ละ field ของ data_fields:]
- text: แสดงตรงๆ
- select/status: Badge component สีตาม value (เช่น active=green, inactive=gray)
- date: format เป็น DD/MM/YYYY (ไทย)
- boolean: CheckCircle (green) / XCircle (gray)
- long text: truncate text-ellipsis ไม่เกิน 60 ตัวอักษร

Actions column (ปุ่มใน row):
- Edit icon (Pencil): เปิด slide-over/modal — ว่างไว้ก่อน
- Delete icon (Trash2): เปิด confirm dialog — ว่างไว้ก่อน

Pagination:
- แสดง 10 รายการ/หน้า, slice mock data ตาม current page

[ถ้า view_type: kanban:]
Card component:
- title (ตัวหนา), description (text-sm text-gray-500 truncate)
- badges ตาม fields เช่น priority (High=red, Medium=yellow, Low=green), due_date
- Edit (Pencil), Delete (Trash2) icons บน card

Cards ใน column ตาม stage ของ mock data

[ถ้า has_search:]
Search:
- Filter mock data real-time เมื่อพิมพ์ (filter ใน [primary text field])
- Debounce 300ms

[ถ้า has_filters:]
Filters:
- Dropdown แต่ละ filter: filter mock data เมื่อเปลี่ยน
- "ล้าง filter" button เมื่อมี filter active

เสร็จเมื่อ:
- [ ] Mock data แสดงใน table/kanban ถูกต้อง
- [ ] Status/type badges สีถูกต้อง
- [ ] Search filter mock data ได้
- [ ] Pagination เปลี่ยนหน้าได้ (ถ้า table)
- [ ] ยังไม่มี Supabase call
```

---

**Chain 3 — จัดการข้อมูล**

```
เชื่อม [entity_name] กับ Supabase และ implement CRUD ทั้งหมด

1. Supabase Schema:

[entity_name_plural] (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users on delete cascade not null,
  [แต่ละ field จาก data_fields — type ที่ตรงกัน],
  [ถ้า reorder ใน operations:] position integer not null default 0,
  [ถ้า view_type: kanban:] stage text not null default '[kanban_stages[0]]',
  created_at timestamptz default now(),
  updated_at timestamptz default now()
)
RLS:
- SELECT: user_id = auth.uid()
- INSERT: user_id = auth.uid()
- UPDATE: user_id = auth.uid()
- DELETE: user_id = auth.uid()
Trigger: updated_at auto-update on row change
Indexes: CREATE INDEX ON [table](user_id); CREATE INDEX ON [table](created_at DESC);
[ถ้า reorder:] CREATE INDEX ON [table](user_id, position);

2. แทน mock data ด้วย Supabase:
- SELECT * FROM [table] WHERE user_id = auth.uid() ORDER BY [position หรือ created_at DESC]

3. Create:
[ถ้า table:] "เพิ่ม [entity]" ปุ่ม → modal form ด้วย react-hook-form + zod
[ถ้า kanban:] inline input ด้านล่างแต่ละ column → INSERT with stage = [column stage]
- INSERT into [table]

4. Update:
- Edit button → slide-over panel (400px, animate slide จากขวา)
- Fields ครบ pre-filled ด้วยค่าปัจจุบัน
- "บันทึก" → UPDATE, ปิด slide-over, refresh list

5. Delete:
- ปุ่ม Delete → shadcn AlertDialog ยืนยัน "ลบ [entity_name] นี้?"
- ยืนยัน → DELETE FROM [table] WHERE id = [id]

[ถ้า reorder ใน operations:]
6. Drag-to-Reorder (@dnd-kit):
- DndContext + SortableContext ครอบ list/column
- onDragEnd: update position ใน local state ทันที (optimistic)
  จากนั้น batch UPDATE position ใน Supabase
  ถ้า Supabase fail: revert ค่าเดิม + toast error

[ถ้า view_type: kanban:]
7. Cross-column drag:
- onDragEnd: ถ้า over !== active container → update stage field + recalculate position
  Optimistic update ทันที, revert ถ้า fail

[ถ้า has_import:]
8. CSV Import:
- ปุ่ม "Import CSV" → upload modal
- parse ด้วย papaparse
- Preview 5 rows แรก + map column headers
- "Import [N] รายการ" → batch INSERT 100 rows ต่อ batch
- ผล: "[X] รายการนำเข้าสำเร็จ, [Y] รายการข้าม (เหตุผล)"

[ถ้า has_export:]
9. CSV Export:
- ดึง ALL records ตาม filter ปัจจุบัน (ไม่ใช่แค่หน้าที่เห็น)
- Generate ใน browser, UTF-8 BOM
- ชื่อไฟล์: [entity]-export-YYYY-MM-DD.csv

[ถ้า is_realtime:]
10. Supabase Realtime:
- Subscribe to [table] changes
- INSERT → เพิ่ม row ใน list
- UPDATE → อัปเดต row
- DELETE → ลบ row
- หมายเหตุ: ต้อง enable Replication ใน Supabase dashboard → Table Editor → [table] → Enable Realtime

เสร็จเมื่อ:
- [ ] List/kanban ดึงข้อมูลจาก Supabase ได้
- [ ] Create บันทึกและแสดงทันที
- [ ] Update บันทึกและ slide-over ปิด
- [ ] Delete ลบและหายจาก list
- [ ] [ถ้า reorder:] drag เปลี่ยน position ได้ + persist หลัง refresh
- [ ] RLS block user อื่น
```

---

**Chain 4 — เก็บรายละเอียด**

```
เพิ่ม loading states, empty states, error handling, และ mobile ให้ระบบ [entity_name]

Loading states:
[ถ้า table:] Skeleton rows: 5 rows, แต่ละ row มี gray-100 pulse สำหรับทุก column
[ถ้า kanban:] Skeleton cards: 2-3 cards ต่อ column, animate-pulse
- ห้าม flash empty state ก่อนข้อมูลโหลด

Empty states:
- ยังไม่มีข้อมูล: icon เหมาะสม + "ยังไม่มี [entity_name]" + ปุ่ม "เพิ่ม [entity] แรก"
- search/filter ไม่เจอ: "ไม่พบ [entity_name] ที่ตรงกับเงื่อนไข" + ปุ่ม "ล้าง filter"
- [ถ้า kanban:] แต่ละ column empty: "ยังไม่มีรายการ"

Error handling:
- Supabase fetch fail: "โหลดข้อมูลไม่ได้" + ปุ่ม "ลองอีกครั้ง" กลางหน้า
- Create/Update fail: toast สีแดง "บันทึกไม่สำเร็จ กรุณาลองใหม่" — ข้อมูลที่กรอกยังอยู่
- Delete fail: toast สีแดง "ลบไม่สำเร็จ" — row กลับมาใน list
- ทุก success action: toast สีเขียว 3 วินาที

Bulk actions (ถ้า view_type: table):
- Checkbox ทุก row + checkbox ใน header (select all)
- Toolbar ปรากฏเมื่อเลือก 1+ รายการ: "เลือก [N] รายการ" + ปุ่ม "ลบที่เลือก"
- ยืนยันก่อน bulk delete

Mobile responsive:
[ถ้า table:] mobile → card list แทน table (แต่ละ row → card แสดง field สำคัญ)
[ถ้า kanban:] scroll horizontal แสดงทีละ 1 column บน < 640px
- Slide-over เต็ม screen บน mobile
- ปุ่ม minimum 48px touch target

เสร็จเมื่อ:
- [ ] Skeleton loading แสดงขณะโหลด
- [ ] Empty state แสดงเมื่อไม่มีข้อมูล
- [ ] Empty search state แสดงเมื่อไม่พบ
- [ ] Supabase error แสดง Retry button
- [ ] Toast แสดงหลังทุก CRUD action
- [ ] Mobile layout ใช้ได้บน 375px
```

## Output Schema

```yaml
plan_title: string          # "[Entity Name] Data Management — Build Plan"
entity_summary: string      # สรุปหนึ่งประโยค
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
# [Entity Name] Data Management — Build Plan
> [สรุปหนึ่งประโยค]

---

## Chain 1 — โครงสร้าง
**เป้าหมาย:** Layout + table/kanban shell ก่อน data ใดๆ

[PROMPT]

**เสร็จเมื่อ:** ...

---
(Chain 2, 3, 4)
---

**รวม 4 Chains**
Paste Chain 1 เข้า Lovable ✓ ทุก checkbox แล้วไป Chain 2
```

## Error Handling

- **ไม่รู้ data_fields** — ถามก่อน schema เขียนไม่ได้
- **kanban แต่ไม่รู้ stages** — ถามก่อน: "column ของ kanban มีอะไรบ้าง?"
- **reorder แต่ลืม position column** — เพิ่ม position integer ใน Chain 3 schema อัตโนมัติ
- **is_realtime: true** — เตือน user ต้อง enable Replication ใน Supabase dashboard

## Examples

### ตัวอย่าง 1: ตาราง CRM Contacts

**Input:** entity: contacts, view: table, operations: [create, read, update, delete], fields: [first_name (text, required), last_name (text, required), email (email), company (text), status (select: lead/prospect/customer)], search: true, filters: true (status), import: true, export: true

- Chain 1: Header + search + filter bar + ปุ่ม "เพิ่ม contact" + "Import" + "Export", table shell 6 columns
- Chain 2: Mock contacts 8 รายการ, status badges สีต่างกัน, search filter แบบ real-time
- Chain 3: Supabase contacts table + RLS, Create modal, Edit slide-over, Delete confirm, batch CSV import, CSV export
- Chain 4: Skeleton 5 rows, empty state, bulk select + delete, mobile card view, toasts ทุก action

### ตัวอย่าง 2: Kanban Tasks

**Input:** entity: tasks, view: kanban, operations: [create, read, update, delete, reorder], fields: [title (text, required), priority (select: high/medium/low), due_date (date)], stages: [รอดำเนินการ, กำลังทำ, รีวิว, เสร็จแล้ว], realtime: true

- Chain 1: 4-column kanban shell, header + card area ว่าง
- Chain 2: Mock task cards พร้อม priority badges, inline create บน column
- Chain 3: Supabase tasks table (stage + position columns) + @dnd-kit drag within + cross-column, Realtime subscription
- Chain 4: Skeleton cards, column empty state, drag fail revert + toast, mobile single-column scroll
