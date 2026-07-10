---
name: lovable-data-management-planner
description: "Generate a phase-by-phase Lovable prompt plan for building data management features — CRUD tables, kanban boards, drag-to-reorder, CSV import/export, and real-time sync."
---

# Lovable Data Management Planner

Turn a data management requirement into a sequenced build plan. Always build schema first, then read view, then write operations — in that order. Never add export or real-time before the data views are working.

## When to Use

- User needs a CRUD table for managing records in Lovable
- User wants a kanban board with drag-and-drop
- User needs CSV import or export
- User asks about Supabase Realtime in a Lovable project
- User wants sortable, filterable, or paginated data lists

## Input Schema

```yaml
entity_name: string         # REQUIRED — what kind of records, e.g. "contacts", "tasks"
view_type: string           # REQUIRED — "table" | "kanban" | "gallery" | "list"
operations: list            # REQUIRED — [create, read, update, delete, reorder]
data_fields: list           # REQUIRED — fields with types, e.g. [{name: title, type: text, required: true}]
has_import: boolean         # OPTIONAL — CSV import needed — default: false
has_export: boolean         # OPTIONAL — CSV export needed — default: false
is_realtime: boolean        # OPTIONAL — Supabase Realtime live updates — default: false
has_relations: boolean      # OPTIONAL — links to other tables — default: false
related_table: string       # OPTIONAL — required if has_relations: true
```

## Workflow

### Step 1: Clarify Requirements

If `data_fields` is missing, ask for the full list of fields before generating any phases. The schema (Phase 1) cannot be written without knowing what columns are needed.

If `view_type` is "kanban", ask: "What are the stages/columns?" and "What field controls which column a card is in?"

### Step 2: Select Phases

```
Phase 1  → Supabase Schema (table + RLS + indexes)
Phase 2  → Read View (list / table / kanban board — no editing)
Phase 3  → Create (add new record)
Phase 4  → Update (edit record — inline or slide-over)
Phase 5  → Delete (single + bulk with confirmation)
Phase 6  → Search + Filters + Sort
Phase 7  → Drag-to-Reorder          (if reorder in operations)
Phase 8  → CSV Import               (if has_import: true)
Phase 9  → CSV Export               (if has_export: true)
Phase 10 → Real-time Sync           (if is_realtime: true)
Phase 11 → Empty States + Error Handling
```

**Ordering rules that must never be broken:**
- Schema always Phase 1
- Read (Phase 2) before any write operation
- Create (Phase 3) before Update (Phase 4) — need records to edit
- Filters (Phase 6) after read view — do not filter an empty list
- Drag-to-reorder requires a `position integer` column — confirm it exists in Phase 1 schema
- Import/export and real-time always last

### Step 3: Write Each Phase Prompt

**Schema rules (Phase 1):**
- Always include `user_id uuid references auth.users on delete cascade`
- Always include `created_at timestamptz default now()`
- Include `position integer not null default 0` if reorder is in operations
- Include `updated_at` + trigger if any field will be edited
- RLS: SELECT/INSERT/UPDATE/DELETE for `user_id = auth.uid()` on every table
- Indexes: always add `CREATE INDEX ON [table](user_id)` at minimum

**Kanban rules:**
- Cards drag within a column (position reorder) AND between columns (stage change)
- On cross-column drop: update stage field + recalculate position
- On Supabase failure: revert to pre-drag state + show error toast

**Drag-to-reorder rules:**
- Library: `@dnd-kit/core` + `@dnd-kit/sortable` always
- Optimistic update: move in local state immediately on drop
- Revert + toast on Supabase failure

**CSV import rules:**
- Parse with `papaparse` — install if not present
- Upload → preview 5 rows → map columns → import with progress bar
- Auto-detect common column names (e.g., "Email Address" → email)
- Insert in batches of 100 rows
- Show results: "X imported, Y skipped (reason)"

**CSV export rules:**
- Export ALL records matching current filters — not just the visible page
- Generate in browser — no server route
- UTF-8 with BOM for Excel compatibility
- File name: `[entity]-export-YYYY-MM-DD.csv`

### Step 4: Output the Plan

List all phases in order with complete prompts and checklists.

## Output Schema

```yaml
plan_title: string          # "[Entity Name] Data Management — Build Plan"
entity_summary: string      # One sentence: entity, view type, key operations
phases:
  - number: integer
    name: string
    goal: string
    prompt: string           # Complete ready-to-paste Lovable prompt
    done_when: list
total_phases: integer
```

## Output Format

```
# [Entity Name] Data Management — Build Plan
> [One sentence: entity, view type, operations]

---

## Phase 1 — Supabase Schema
**Goal:** Stable data model before any UI.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] Table exists with all columns and correct types
- [ ] RLS blocks a different user's SELECT
- [ ] Indexes are created
- [ ] updated_at auto-updates on row change (if applicable)

---

## Phase 2 — Read View
**Goal:** Display records before adding any editing.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] Records load from Supabase and display correctly
- [ ] Skeleton shows on load, disappears when data arrives
- [ ] Empty state shows when no records exist
- [ ] No create/edit/delete yet — read only

---

**Total phases: N**
Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2.
```

## Error Handling

- **data_fields missing** — ask for the full field list before writing Phase 1; schema cannot be written without it
- **kanban but no stages defined** — ask "What are the column names?" before generating Phase 2
- **reorder requested but no position field** — add `position integer not null default 0` to Phase 1 schema and flag it in the Phase 7 prompt
- **is_realtime: true** — warn that real-time requires enabling Supabase Replication on the table in the Supabase dashboard

## Examples

### Example 1: CRM Contacts Table

**Input:** Entity: contacts, view: table, operations: [create, read, update, delete], fields: [first_name (text, required), last_name (text, required), email (email), company (text), status (select: lead/prospect/customer)], import: true, export: true, realtime: false

**Phases:**
1. Supabase Schema (contacts table + RLS)
2. Read View (table with sortable columns, status badges)
3. Create (modal form with all fields)
4. Update (slide-over panel, auto-save on blur)
5. Delete (single via row action, bulk via checkboxes)
6. Search + Filters (name/email search, status multiselect)
7. CSV Import (upload → map → batch insert)
8. CSV Export (all filtered contacts)
9. Empty States + Error Handling

**Total: 9 phases**

### Example 2: Task Kanban Board

**Input:** Entity: tasks, view: kanban, operations: [create, read, update, delete, reorder], fields: [title (text, required), description (textarea), priority (select: low/medium/high), due_date (date), assignee (text)], stages: [Backlog, In Progress, Review, Done], import: false, export: false, realtime: true

**Phases:**
1. Supabase Schema (tasks table with stage + position columns + RLS)
2. Read View (4-column kanban, cards with title, priority badge, due date)
3. Create (inline card input at bottom of each column)
4. Update (slide-over panel, auto-save)
5. Delete (trash icon on card, confirm dialog)
6. Drag-to-Reorder (within column + between columns via @dnd-kit)
7. Real-time Sync (Supabase Realtime — changes appear in all open tabs)
8. Empty States + Error Handling

**Total: 8 phases**
