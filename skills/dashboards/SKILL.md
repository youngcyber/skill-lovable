---
name: lovable-dashboard-planner
description: "Generate a phase-by-phase Lovable prompt plan for building admin dashboards and analytics views — layout shell, KPI metrics, Recharts charts, data tables, and export."
---

# Lovable Dashboard Planner

Turn a dashboard requirement into a sequenced build plan. Dashboards are built in layers: layout first, then summary stats, then charts, then detail tables. Never add export or real-time before the data views are working.

## When to Use

- User wants to build an admin dashboard or analytics view with Lovable
- User needs KPI metric cards, charts, or sortable data tables
- User asks about Recharts integration in a Lovable project
- User needs a reporting or monitoring interface

## Input Schema

```yaml
dashboard_name: string     # REQUIRED — name of the dashboard
dashboard_purpose: string  # REQUIRED — what decisions does it help users make?
key_metrics: list          # REQUIRED — 3-5 most important numbers to show
data_sources: list         # REQUIRED — Supabase tables or API endpoints
charts_needed: list        # OPTIONAL — [area, bar, line, pie, donut]
tables_needed: list        # OPTIONAL — entities that need list views
has_export: boolean        # OPTIONAL — CSV or PDF export needed — default: false
is_realtime: boolean       # OPTIONAL — live updates via Supabase Realtime — default: false
```

## Workflow

### Step 1: Clarify Requirements

If `key_metrics` is missing, ask: "What are the 3 most important numbers this dashboard needs to show?" Do not write chart prompts without knowing the data source and format.

### Step 2: Select Phases

```
Phase 1 → Stack + Layout Shell
Phase 2 → KPI Metrics Row
Phase 3 → Primary Chart
Phase 4 → Secondary Charts          (if charts_needed has more than one)
Phase 5 → Data Table #1
Phase 6 → Data Table #2             (one phase per table)
Phase 7 → Detail Slide-over Panel   (if table rows are clickable)
Phase 8 → Global Filters + Date Range
Phase 9 → Export                    (if has_export: true)
Phase 10 → Real-time Sync           (if is_realtime: true)
Phase 11 → Empty States + Error Handling
```

**Ordering rules:**
- Layout always Phase 1
- KPI row always Phase 2 — most-read part of any dashboard
- Charts before tables — high-level before row-level
- Detail slide-over only after its table exists
- Global filters after all individual data views work
- Export and real-time always last

### Step 3: Write Each Phase Prompt

**KPI metrics rules:**
- Use `animate-pulse` skeleton cards while fetching — never a full-page spinner
- If a single metric fails: show "—" + "Unavailable" — do not fail the whole row
- Delta: green + TrendingUp if positive, red + TrendingDown if negative, gray if zero

**Chart rules:**
- Library: Recharts always — do not use Chart.js or other libraries
- AreaChart: gradient fill, opacity 0.2 at top → 0 at bottom
- On date range change: keep previous chart at opacity-40 while fetching — never unmount
- Tooltip: white card with shadow-lg, formatted values

**Table rules:**
- Sort state and filter state always in URL params
- Server-side filtering via Supabase — never client-side filtering of a full dataset
- Bulk action bar appears in a sticky bottom bar when ≥1 row is selected

### Step 4: Output the Plan

List all phases in order with complete prompts and checklists.

## Output Schema

```yaml
plan_title: string            # "[Dashboard Name] — Build Plan"
dashboard_summary: string     # One sentence: what it shows and who uses it
phases:
  - number: integer
    name: string
    goal: string
    prompt: string             # Complete ready-to-paste Lovable prompt
    done_when: list
total_phases: integer
```

## Output Format

```
# [Dashboard Name] — Build Plan
> [One sentence: what it shows, who uses it, main data source]

---

## Phase 1 — Stack + Layout Shell
**Goal:** Navigation frame before any data.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] Layout renders on 375px / 768px / 1280px without overflow
- [ ] Sidebar collapse persists across page refreshes
- [ ] All nav links work with correct active state
- [ ] Sign-out redirects to /login

---

## Phase 2 — KPI Metrics Row
**Goal:** Top-level summary numbers with deltas.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] All metric cards render with correct values and formatting
- [ ] Skeleton cards show on first load
- [ ] Delta is green/red/gray based on direction
- [ ] A single failing metric shows "—" without breaking others

---

**Total phases: N**
Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2.
```

## Error Handling

- **No data source specified** — ask which Supabase table or API endpoint the metric comes from before writing the prompt
- **Chart type unclear** — ask "Is this time-series data or a category comparison?" to pick AreaChart vs BarChart
- **Too many metrics (6+)** — suggest grouping into two KPI rows (primary and secondary) rather than one 6-column row

## Examples

### Example 1: E-Commerce Analytics Dashboard

**Input:** Dashboard: StoreIQ, metrics: [revenue, orders, AOV, conversion rate], data: Supabase orders table, charts: [area (revenue over time), bar (revenue by category)], tables: [orders], export: true

**Phases:**
1. Stack + Layout Shell (sidebar: Overview, Orders, Products, Customers, Reports)
2. KPI Row (Revenue, Orders, AOV, Conversion Rate — all from Supabase)
3. Revenue Over Time Area Chart (30D default, date range filter)
4. Revenue by Category Bar Chart
5. Orders Data Table (sortable, filterable, paginated)
6. Order Detail Slide-over
7. Global Date Range Filter
8. CSV Export (orders matching current filters)
9. Empty States + Error Handling

**Total: 9 phases**

### Example 2: HR Dashboard

**Input:** Dashboard: PeopleOS, metrics: [headcount, open roles, avg tenure, attrition rate], charts: [bar (headcount by dept), line (hiring over time)], tables: [employees], realtime: false

**Phases:**
1. Stack + Layout Shell
2. KPI Row (4 metrics from Supabase employees table)
3. Headcount by Department Bar Chart
4. Hiring Over Time Line Chart
5. Employees Data Table
6. Employee Detail Slide-over
7. Empty States + Error Handling

**Total: 7 phases**
