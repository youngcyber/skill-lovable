---
name: lovable-dashboard-planner
description: "รับ idea dashboard แล้วสร้าง prompt แบบ chain pattern 4 ชั้นสำหรับ Lovable: โครงสร้าง → widgets & charts → เชื่อม Supabase → เก็บรายละเอียด รองรับภาษาไทย"
---

# Lovable Dashboard Planner

สร้าง chain prompt สำหรับ analytics dashboard บน Lovable ใน 4 ชั้น สร้าง layout + widget shell ก่อน แล้วค่อย connect data จริง

## When to Use

- User อยากสร้าง analytics dashboard, admin panel, หรือ reporting page
- User พูดถึง charts, KPI cards, หรือ metric ต่างๆ
- User ต้องการดูข้อมูลสรุปของแอป เช่น ยอดขาย, จำนวน user, conversion rate
- User ใช้ภาษาไทยในการอธิบาย dashboard

## Input Schema

```yaml
app_name: string           # ชื่อแอป
dashboard_title: string    # ชื่อ dashboard เช่น "Sales Overview"
kpi_cards: list            # KPI แต่ละตัว เช่น [total_revenue, active_users, churn_rate]
charts: list               # charts ที่ต้องการ: [line, bar, pie, area, donut]
chart_descriptions: list   # อธิบายแต่ละ chart เช่น "รายได้รายเดือน 12 เดือนหลัง"
data_source: string        # ที่มาของข้อมูล เช่น "orders table", "events table"
time_filter: boolean       # มี filter เลือกช่วงเวลาไหม — default: true
has_table: boolean         # มีตารางข้อมูลด้านล่างไหม — default: false
```

## Workflow

### Step 1: ทำความเข้าใจ dashboard

ถ้าไม่รู้ `kpi_cards` หรือ `charts` ให้ถามก่อน — "metric หลักที่อยากเห็นบน dashboard มีอะไรบ้าง?"

### Step 2: วาง 4 Chain

```
Chain 1 → โครงสร้าง: Header, KPI card shells, chart placeholders
Chain 2 → ฟีเจอร์หลัก: Charts ด้วย Recharts + mock data, time filter UI
Chain 3 → จัดการข้อมูล: Query Supabase จริง, aggregate functions, replace mock
Chain 4 → เก็บรายละเอียด: Loading skeletons, empty states, responsive, export
```

### Step 3: เขียน Prompt แต่ละ Chain

---

**Chain 1 — โครงสร้าง**

```
สร้าง dashboard shell สำหรับ [ชื่อ dashboard] ของ [ชื่อแอป]

Tech stack:
- React + Vite + TypeScript
- Tailwind CSS + shadcn/ui
- Recharts (ติดตั้งด้วย)
- Lucide React

Layout:
- Header: ชื่อ "[dashboard_title]" ซ้าย, time range selector ขวา
  (Today / Last 7 days / Last 30 days / Last 3 months / Custom)
- bg-gray-50 p-6 min-h-screen

KPI Cards (grid: 4 cols desktop, 2 tablet, 1 mobile):
[สร้างทีละ card ตาม kpi_cards:]
- Card: white bg, rounded-2xl, border, p-6
- Icon (Lucide ที่เหมาะสม) + ชื่อ metric text-sm text-gray-500
- Value: placeholder "--" text-3xl font-bold
- Change badge: "+X%" text-sm (สีเขียว = บวก, สีแดง = ลบ) — ว่างไว้ก่อน

Chart placeholders (grid ตาม charts ที่มี):
- แต่ละ chart: card white, rounded-2xl, border, p-6
- ชื่อ chart text-lg font-semibold บน
- พื้นที่ h-64 bg-gray-50 rounded-lg — placeholder ว่างไว้ก่อน

เสร็จเมื่อ:
- [ ] KPI cards [N] ใบ render บน grid
- [ ] Chart placeholder cards render ครบ
- [ ] Time range selector แสดงได้ (ยังไม่ต้อง filter จริง)
- [ ] Layout responsive บน 375px / 768px / 1280px
- [ ] ไม่มี Supabase call หรือ chart library ใดๆ ในชั้นนี้
```

---

**Chain 2 — ฟีเจอร์หลัก**

```
เพิ่ม charts และ interactions ให้ [ชื่อ dashboard] ด้วย mock data

ใช้ Recharts สำหรับทุก chart ข้อมูลเป็น const mockData hardcode ในไฟล์ก่อน

[สร้างทีละ chart ตามที่กำหนด:]

CHART: [ชื่อ chart 1] — [ประเภท: Line/Bar/Area/Pie/Donut]
- ข้อมูล: [อธิบาย x-axis, y-axis, หรือ segments]
- Mock data: 12 จุด (รายเดือน) หรือ 7 จุด (รายวัน)
- สี: stroke/fill ใช้ indigo-500, emerald-500, rose-500 ตามลำดับ
- Tooltip: custom — แสดงค่าพร้อม unit เช่น "฿12,500" หรือ "1,234 users"
- Legend: ด้านล่าง chart
- CartesianGrid: stroke="#f0f0f0" strokeDasharray="3 3"
- Axes: text-xs text-gray-500, ไม่มี tick marks

[CHART 2, 3 ตามที่มี]

KPI Values (mock):
[แต่ละ KPI card อัปเดต placeholder "--" เป็นค่า mock เช่น "฿124,500"]
Change badge: "+12.5% vs เดือนก่อน"

Time range UI:
- Select ทำงานได้ (เปลี่ยน state แต่ mock data ยังไม่เปลี่ยน)

เสร็จเมื่อ:
- [ ] ทุก chart render ด้วย mock data
- [ ] Tooltip แสดงค่าเมื่อ hover
- [ ] KPI values แสดงค่าและ % change
- [ ] ไม่มี Supabase call ยังคง mock ทั้งหมด
```

---

**Chain 3 — จัดการข้อมูล**

```
เชื่อม [ชื่อ dashboard] กับ Supabase จริง

[ถ้าต้องการ aggregate query:]
ใช้ Supabase Postgres function (rpc) สำหรับ aggregate ที่ซับซ้อน

KPI Queries (แต่ละ KPI):
[KPI 1 — total_revenue]:
SELECT SUM(amount) FROM [data_source]
WHERE user_id = auth.uid()
AND created_at >= [start_date]

[KPI 2 ...] และต่อไป

Chart Queries:
[Chart 1 — revenue by month]:
SELECT date_trunc('month', created_at) as month, SUM(amount) as total
FROM [data_source]
WHERE user_id = auth.uid()
AND created_at >= [start_date]
GROUP BY month ORDER BY month

[Chart 2 ...]

Connect:
- แทน mockData ทุกตัวด้วย Supabase query
- Time range selector เชื่อมกับ query WHERE clause
- แต่ละ query เป็น React hook แยก เช่น useRevenueData(range)

RLS บน [data_source]:
SELECT: user_id = auth.uid() เท่านั้น

เสร็จเมื่อ:
- [ ] KPI cards แสดงค่าจริงจาก Supabase
- [ ] Charts แสดงข้อมูลจริง
- [ ] เปลี่ยน time range → ข้อมูล re-query และ chart อัปเดต
- [ ] RLS block query จาก user อื่น
```

---

**Chain 4 — เก็บรายละเอียด**

```
เพิ่ม loading states, error handling, และ polish ให้ [ชื่อ dashboard]

Loading states:
- KPI card: skeleton pulse (w-16 h-8 bg-gray-100 rounded animate-pulse) แทน "--"
- Chart area: skeleton h-64 rounded-lg animate-pulse แทน chart
- ห้ามแสดง chart แบบ "ค่อยๆ โผล่" — รอ data ครบแล้วค่อย render ทีเดียว

Empty states:
- ถ้าไม่มีข้อมูลในช่วงเวลาที่เลือก: icon + "ยังไม่มีข้อมูลในช่วงนี้" centered ใน chart card
- KPI ที่เป็น 0 แสดง 0 ไม่ใช่ "--"

Error handling:
- Supabase error: แสดง "โหลดข้อมูลไม่ได้ ลองอีกครั้ง" + ปุ่ม Retry ใน card นั้น
- ห้าม crash ทั้งหน้า

[ถ้า has_table: true:]
- ตารางใต้ charts: เพิ่ม export CSV ปุ่มขวาบน
- Export ดึงข้อมูลตาม time range + filter ปัจจุบัน
- ชื่อไฟล์: [dashboard-name]-[date].csv UTF-8 BOM

Responsive:
- mobile: KPI 2 cols, charts stack เป็น single column
- Chart ย่อ height เป็น h-48 บน mobile

Print / export:
- "Export PDF" ปุ่ม: เรียก window.print() พร้อม print CSS ซ่อน navbar + sidebar

เสร็จเมื่อ:
- [ ] KPI skeleton แสดงระหว่างโหลด
- [ ] Chart skeleton แสดงระหว่างโหลด
- [ ] Empty state แสดงเมื่อไม่มีข้อมูล
- [ ] Supabase error แสดง Retry button
- [ ] Mobile layout ใช้ได้บน 375px
```

## Output Schema

```yaml
plan_title: string          # "[Dashboard Title] — Build Plan"
dashboard_summary: string   # สรุปหนึ่งประโยค
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
# [Dashboard Title] — Build Plan
> [สรุปหนึ่งประโยค]

---

## Chain 1 — โครงสร้าง
**เป้าหมาย:** KPI card shells + chart placeholder layout ก่อน data ใดๆ

[PROMPT]

**เสร็จเมื่อ:** ...

---
(Chain 2, 3, 4)
---

**รวม 4 Chains**
Paste Chain 1 เข้า Lovable ✓ ทุก checkbox แล้วไป Chain 2
```

## Error Handling

- **ไม่รู้ metric หลัก** — ถามก่อน: "metric หลักที่อยากเห็นบน dashboard มีอะไรบ้าง?"
- **ต้องการ real-time** — เพิ่ม Supabase Realtime subscription เป็น sub-step ใน Chain 3 ระวัง enable Replication ใน Supabase dashboard ด้วย
- **ข้อมูลจาก API ภายนอก** — ใช้ Supabase Edge Function เป็น middleware ดึงข้อมูลมาเก็บก่อน ไม่ call ตรงจาก client

## Examples

### ตัวอย่าง: E-Commerce Analytics Dashboard

**Input:** app: ShopFlow, KPIs: [total_revenue, orders, avg_order_value, conversion_rate], charts: [revenue by month (line), orders by category (bar), traffic sources (pie)], data_source: orders table, time_filter: true

- Chain 1: Header + time range selector, 4 KPI card shells, 3 chart placeholders
- Chain 2: Line chart รายได้รายเดือน mock, Bar chart หมวดหมู่ mock, Pie chart traffic mock, KPI values mock
- Chain 3: Query SUM(amount), COUNT(*) จาก orders, aggregate by month/category, เชื่อม time range filter
- Chain 4: Skeleton loading, empty state ช่วงที่ไม่มีออเดอร์, Retry button, mobile single-column
