---
name: data-management-planner
description: >
  Generates a phase-by-phase Lovable prompt plan for building data management features:
  CRUD tables, kanban boards, CSV import/export, and real-time data views.
  Activate when the user needs to display, edit, organize, or move data records.
version: 1.0.0
tags: [data, crud, kanban, csv, import, export, supabase, real-time, lovable]
---

# Data Management — Phase Planner

You are a data management build planner for Lovable. When the user describes a data feature (tables, kanban, import/export), output a phase-by-phase prompt plan. Always build the data model first, then the read view, then write operations.

---

## Step 1 — Gather Requirements

Ask (or infer):

1. **Data entity** — what kind of records are being managed? (tasks, contacts, products, orders…)
2. **View type** — table list, kanban board, calendar, or gallery grid?
3. **Operations** — which of: Create / Read / Update / Delete / Reorder?
4. **Import/export?** — does data need to come in or go out as CSV?
5. **Real-time?** — do multiple users need to see changes live?
6. **Relations** — does this entity link to other tables?

---

## Phase Order for Data Management

```
Phase 1 → Supabase Schema (table + RLS + indexes)
Phase 2 → Read View (list / table / board — no editing yet)
Phase 3 → Create (add new record — form or inline)
Phase 4 → Update (edit record — inline or slide-over)
Phase 5 → Delete (single + bulk, with confirmation)
Phase 6 → Search + Filters + Sort
Phase 7 → Drag-to-Reorder (kanban or sortable list)
Phase 8 → CSV Import
Phase 9 → CSV Export
Phase 10 → Real-time Sync (Supabase subscriptions)
```

**Why this order:**
- Schema before any UI — data shape must be stable
- Read view before write — see the data before editing it
- Create before update — you need records to exist before editing them
- Delete after create + update — clean up what you've built
- Filters after read — filtering empty views is pointless
- Reorder after the list works — drag-and-drop on a broken list is harder to debug
- Import/export last — they're additive, not foundational

---

## Phase Templates

### Phase 1 — Supabase Schema

```
Create the Supabase schema for [ENTITY NAME] in [APP NAME].

Main table:
[TABLE_NAME] (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users on delete cascade,
  [FIELD_1] [TYPE] [NOT NULL / default value],
  [FIELD_2] [TYPE],
  [FIELD_3] [TYPE] check ([CONSTRAINT]),
  position integer not null default 0,    -- include if drag-to-reorder is needed
  created_at timestamptz default now(),
  updated_at timestamptz default now()
)

[IF RELATED TABLE:]
[RELATED_TABLE_NAME] (
  id uuid primary key default gen_random_uuid(),
  [TABLE_NAME]_id uuid references [TABLE_NAME](id) on delete cascade,
  [FIELDS],
  created_at timestamptz default now()
)

RLS on [TABLE_NAME]:
- SELECT: user_id = auth.uid()
- INSERT: user_id = auth.uid()
- UPDATE: user_id = auth.uid()
- DELETE: user_id = auth.uid()

Indexes (for performance):
- CREATE INDEX ON [TABLE_NAME](user_id);
- CREATE INDEX ON [TABLE_NAME](user_id, [COMMONLY_FILTERED_FIELD]);

Trigger: update_updated_at() → fires BEFORE UPDATE, sets updated_at = now().

Do not build any UI in this phase.

Done when:
- [ ] Table exists in Supabase with all columns and correct types
- [ ] RLS blocks a different user from selecting this user's rows
- [ ] Indexes are created
- [ ] updated_at auto-updates on row change
- [ ] Test INSERT and SELECT work for the authenticated user
```

---

### Phase 2 — Read View

```
Build the read view for [ENTITY NAME] at [ROUTE] in [APP NAME].

[CHOOSE ONE:]

[IF TABLE VIEW:]
Data: SELECT * FROM [TABLE_NAME] WHERE user_id = auth.uid() ORDER BY [DEFAULT_SORT]

Columns:
- [COLUMN HEADER] → [field] → [format: text / date / badge / currency]
- [COLUMN HEADER] → [field] → [format]
- [COLUMN HEADER] → [field] → [format]

Table: sticky header, full-width, white bg, rounded-xl border
Row: hover:bg-gray-50, cursor-pointer if rows are clickable
Badge colors (if any):
- [VALUE]: bg-[COLOR]-50 text-[COLOR]-700

[IF KANBAN VIEW:]
Data: SELECT * FROM [TABLE_NAME] WHERE user_id = auth.uid() ORDER BY position

Columns: [LIST ALL STAGES IN ORDER]
Each column header: stage name + count of cards in that stage

Each card:
- [FIELD_1] (title)
- [FIELD_2] (subtitle or badge)
- [FIELD_3] (date or value)

[IF GALLERY GRID:]
Grid: [N]-column on desktop, 2-column on tablet, 1-column on mobile
Each card: [DESCRIBE CARD LAYOUT]

Loading state: [N] skeleton [rows / cards / columns] (animate-pulse)
Empty state: [ICON] + "[MESSAGE]" + "[CTA]" button

Do not add create/edit/delete in this phase — read only.

Done when:
- [ ] [ENTITY] records load from Supabase and display correctly
- [ ] Skeleton appears on load and disappears when data arrives
- [ ] Empty state shows when no records exist
- [ ] [Table is scrollable on mobile / Kanban columns scroll independently / Grid collapses on mobile]
```

---

### Phase 3 — Create

```
Add the ability to create new [ENTITY NAME] records in [APP NAME].

[CHOOSE ONE UI PATTERN:]

[PATTERN A — Modal form:]
- "Add [ENTITY]" button (top-right of the page, indigo-600)
- Opens a shadcn Dialog with a form
- Fields: [LIST FIELDS, TYPES, AND REQUIRED STATUS]
- Submit: INSERT into [TABLE_NAME] via Supabase
- On success: close modal + prepend new record to the list (optimistic update)
- On error: show inline error above the submit button, keep modal open
- Form resets after successful save

[PATTERN B — Inline row at top of table:]
- "+ Add [entity]" row pinned at the top of the table
- Clicking it reveals inline inputs in the row (same columns as the table)
- Press Enter or click ✓ to save, press Escape to cancel
- On save: INSERT + row appears immediately

[PATTERN C — Inline card at top of kanban column:]
- "+ Add card" button at the bottom of each kanban column
- Clicking shows an inline text input in that column
- Press Enter or click ✓ to save, Escape to cancel
- On save: INSERT with stage = that column's stage name + position = last in column

Optimistic update:
- Add the new record to local state immediately on submit
- Revert + show error toast if Supabase INSERT fails

Done when:
- [ ] "Add [ENTITY]" creates a record in Supabase
- [ ] New record appears in the list immediately (before Supabase confirms)
- [ ] If Supabase fails, record is removed from the list and an error is shown
- [ ] Form / inline input resets after successful save
```

---

### Phase 4 — Update (Edit)

```
Add the ability to edit [ENTITY NAME] records in [APP NAME].

[CHOOSE ONE UI PATTERN:]

[PATTERN A — Slide-over panel:]
Trigger: clicking a row / card opens a slide-over (fixed right-0, 400px, full-screen mobile)
- Panel slides in: translate-x-full → translate-x-0, 300ms ease-out
- Overlay: bg-black/30, closes panel on click
- Closes on: Escape, overlay click, X button

Panel contents:
- [FIELD_1]: [editable inline input or display-only]
- [FIELD_2]: [shadcn Select / Textarea / DatePicker]
- [FIELD_3]: [INPUT TYPE]

Auto-save: debounce 800ms after user stops typing → UPDATE [TABLE_NAME] WHERE id = ?
Show "Saving…" → "Saved ✓" indicator in the panel header

[PATTERN B — Inline cell editing:]
- Click any editable cell to turn it into an input
- Blur or Enter → saves that field UPDATE [TABLE_NAME] SET [field] = ? WHERE id = ?
- Escape → reverts the edit
- Show a subtle ring (ring-2 ring-indigo-400) on the cell while editing

Optimistic update:
- Update local state immediately on change
- Revert on Supabase error + show error toast

Done when:
- [ ] Editing a field [saves on blur / auto-saves after 800ms]
- [ ] "Saving…" / "Saved ✓" indicator appears correctly
- [ ] Editing one record does not affect other records in the list
- [ ] Supabase error reverts the change and shows an error toast
- [ ] Panel closes correctly on Escape and outside click
```

---

### Phase 5 — Delete

```
Add delete functionality for [ENTITY NAME] in [APP NAME].

Single delete:
- Where: [row action menu "…" / delete button in slide-over panel / right-click context menu]
- On click: open a shadcn AlertDialog
  - Title: "Delete [entity]?"
  - Description: "This will permanently delete [description of what gets deleted]. This cannot be undone."
  - Buttons: "Cancel" (gray) + "Delete" (red, bg-red-600)
- On confirm: DELETE FROM [TABLE_NAME] WHERE id = ?
- Optimistic: remove from local state immediately, revert on error

Bulk delete:
- Row checkboxes in the table (header checkbox = select all on current page)
- Sticky bottom bar when ≥1 row selected: "[N] selected" + "Delete selected" button (red)
- On click: AlertDialog — "Delete [N] [entities]? This cannot be undone."
- On confirm: DELETE FROM [TABLE_NAME] WHERE id IN (...)
- Optimistic: remove all selected rows immediately

[IF CASCADE DELETES:]
- Mention related records in the confirm dialog: "This will also delete [N] related [records]."

Done when:
- [ ] Single delete confirmation dialog shows and deletes on confirm
- [ ] Deleted record disappears immediately (optimistic)
- [ ] Failed delete reverts the removal and shows an error toast
- [ ] Bulk delete bar appears when rows are checked
- [ ] Bulk confirm dialog mentions the correct count
```

---

### Phase 6 — Search + Filters + Sort

```
Add search, filters, and sorting to the [ENTITY NAME] view in [APP NAME].

Text search:
- Input above the table/board with a Search icon
- Searches [FIELDS — e.g., name, email, description] using Supabase ilike operator
- Debounced 300ms
- Clears to show all records when empty

Filters:
[LIST EACH FILTER:]
- [FILTER_1]: [type — dropdown, multiselect, date range, toggle]
  Field: [TABLE_FIELD]
  Options: [LIST or "from DISTINCT query"]
- [FILTER_2]: [type]
  Field: [TABLE_FIELD]

Filter UI:
- Filter buttons/dropdowns above the table
- Active filter: highlighted (indigo tint)
- "Clear all" link: visible only when any non-default filter is active
- Filter state in URL params: ?search=x&status=active,pending

Sorting:
- Columns that should be sortable: [LIST COLUMNS]
- Click header → asc; click again → desc; click again → remove sort
- Sort indicator: ChevronUp / ChevronDown icon
- Default sort: [FIELD] [asc/desc]
- Sort state in URL params: ?sort=[field]&dir=[asc|desc]

All filters + sort → re-fetch from Supabase (server-side filtering, not client-side)
Reset to page 1 when any filter changes.

Done when:
- [ ] Search filters records with 300ms debounce
- [ ] [FILTER_1] shows only matching records
- [ ] Combining search + filter narrows results correctly
- [ ] "Clear all" resets all filters and re-fetches
- [ ] Sort updates URL params and re-fetches in the correct order
- [ ] Filters persist in the URL (sharing the URL shows the same filtered view)
```

---

### Phase 7 — Drag-to-Reorder

```
Add drag-to-reorder to [ENTITY NAME] in [APP NAME].

Library: @dnd-kit/core + @dnd-kit/sortable (install if not present)

[IF SORTABLE LIST / TABLE:]
- Drag handle: GripVertical Lucide icon, left of each row, visible on hover
- Dragging: row gets shadow-lg, scale-[1.02], opacity-90
- Other rows: animate out of the way smoothly (using SortableContext)
- On drop: recalculate position values → batch UPDATE positions in Supabase

[IF KANBAN — MOVE BETWEEN COLUMNS:]
- Cards drag within a column (reorder) AND between columns (change stage)
- While dragging over a column: column gets a subtle indigo-50 background highlight
- On cross-column drop:
  - Update [STAGE_FIELD] = destination column's stage
  - Update position = last position in that column + 1
  - Batch update all positions in both affected columns

Position recalculation strategy:
- Use integer positions with gaps (0, 1000, 2000...) to minimize updates
- On drop: only update the moved item's position (set to avg of neighbors)
- Reindex all positions in a column only when positions run out of gaps

Optimistic update:
- Move the item in local state immediately on drop
- Revert to pre-drag state on Supabase UPDATE failure
- Show error toast: "Couldn't save the new order. Please try again."

Done when:
- [ ] Drag handle appears on row/card hover
- [ ] Dragging animates smoothly (no jump)
- [ ] Dropping saves the new position to Supabase
- [ ] Local state updates immediately (no waiting for Supabase)
- [ ] On Supabase failure, order reverts and an error toast appears
- [ ] [Kanban only: Moving between columns updates the stage field]
```

---

### Phase 8 — CSV Import

```
Add CSV import for [ENTITY NAME] in [APP NAME].

Trigger: "Import" button above the [table/board] → opens a 3-step modal (shadcn Dialog)

Step 1 — Upload:
- Drag-and-drop zone: dashed border, UploadCloud icon, "Drop a .csv file here or click to browse"
- Accept: .csv files only (accept=".csv")
- Parse with papaparse (install if not present): { header: true, skipEmptyLines: true }
- Show a preview table of the first 5 rows after parsing
- "Next →" button (disabled until a file is parsed successfully)

Step 2 — Map columns:
- Left column: detected CSV headers (from papaparse result.meta.fields)
- Right column: shadcn Select per header → maps to a [ENTITY] field
- Auto-detect common header names:
  [LIST MAPPINGS — e.g., "Email" / "email" / "Email Address" → email field]
- Required field indicators (red asterisk) for: [LIST REQUIRED FIELDS]
- "Next →" disabled until all required fields are mapped

Step 3 — Import:
- Summary: "[N] rows to import"
- "Import [N] [entities]" button (indigo-600, full-width)
- Insert in batches of 100 via Supabase INSERT
- Progress bar (indigo-600) fills as batches complete
- Results summary: "[X] imported successfully, [Y] skipped (reason)"
- Skipped reasons: duplicate [UNIQUE_FIELD], missing required field
- "Done" button: closes modal, refreshes the table

Done when:
- [ ] CSV upload shows a 5-row preview before mapping
- [ ] Auto-detection maps common column names correctly
- [ ] Required field mapping is enforced before proceeding
- [ ] Progress bar advances during batch insert
- [ ] Results show correct imported/skipped counts
- [ ] Table refreshes after import with the new records
```

---

### Phase 9 — CSV Export

```
Add CSV export for [ENTITY NAME] in [APP NAME].

Trigger: "Export" button above the table
- Exports ALL records matching the current filters (not just the current page)
- Generate the CSV entirely in the browser (no server route needed)

Steps:
1. Fetch all matching records from Supabase (remove pagination limit, keep filters)
2. Map to CSV rows with human-readable column headers
3. Encode as UTF-8 CSV with BOM (for Excel compatibility)
4. Trigger a browser download via a temporary <a> element

CSV columns:
[LIST IN ORDER:]
- "[HEADER]" → [TABLE_FIELD] → [format: as-is / date formatted as YYYY-MM-DD / etc.]
- "[HEADER]" → [TABLE_FIELD]

File name: "[entity-slug]-export-YYYY-MM-DD.csv"

Button behavior:
- "Export" → spinner + disabled while fetching all records
- After download triggers → button returns to normal
- If fetch fails → show error toast "Export failed. Please try again."

Done when:
- [ ] Export button downloads a .csv file with all filtered records
- [ ] CSV includes ALL matching records, not just the visible page
- [ ] Column headers are human-readable (not raw field names)
- [ ] File opens correctly in Excel and Google Sheets (UTF-8 with BOM)
- [ ] Button shows a spinner while records are being fetched
```

---

### Phase 10 — Real-time Sync

```
Add real-time updates to [ENTITY NAME] in [APP NAME] using Supabase Realtime.

Enable Supabase Realtime on the [TABLE_NAME] table:
- In Supabase dashboard → Database → Replication → enable [TABLE_NAME]

Subscribe to changes:
supabase
  .channel('[channel-name]')
  .on('postgres_changes', {
    event: '*',  -- INSERT, UPDATE, DELETE
    schema: 'public',
    table: '[TABLE_NAME]',
    filter: 'user_id=eq.' + user.id
  }, (payload) => {
    // handle each event type
  })
  .subscribe()

Handle each event type:
- INSERT: add the new record to local state (if it passes current filters)
- UPDATE: update the matching record in local state
- DELETE: remove the matching record from local state

Cleanup: unsubscribe when the component unmounts (return () => supabase.removeChannel(channel))

Visual feedback:
- Newly inserted records: briefly highlight with a bg-indigo-50 flash (500ms) when they appear
- Updated records: briefly highlight with bg-yellow-50 flash (500ms)

Presence (optional — add if multiple users work together):
- Show avatars of other users currently viewing this page (using Supabase Presence)

Done when:
- [ ] Opening the same view in two browser tabs: creating a record in tab A shows it in tab B within 1 second
- [ ] Updating a record in tab A updates it in tab B
- [ ] Deleting a record in tab A removes it from tab B
- [ ] New records flash indigo-50 when they appear via real-time
- [ ] Subscription is cleaned up when navigating away from the page
```

---

## Output Format

When the user describes their data management need, respond with:

---

**[ENTITY NAME] Data Management — Build Plan**
> [One sentence confirming the entity, view type, and key operations]

**Phase 1 — Supabase Schema**
*Goal: Stable data model before any UI.*
[COMPLETE LOVABLE PROMPT]
**Done when:** [ ] ... [ ] ... [ ] ...

*(continue for relevant phases only)*

**Total phases: [N]**
*Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2.*

---

## Rules

1. Schema is always Phase 1 — never skip it even if "the table already exists"
2. Read before write — always build the list view before create/update/delete
3. Create before update — you need records to edit
4. Filters after read view — don't filter an empty list
5. Drag-to-reorder requires a position column — confirm it exists in the schema before building
6. Import and export are always independent phases at the end
7. Real-time is always the final phase — it's an enhancement, not a foundation
