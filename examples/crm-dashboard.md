# Real-World Example: CRM Dashboard

A complete walkthrough of building a data-heavy CRM dashboard with Lovable. Features include a contacts database, deal pipeline (kanban), activity feed, analytics with charts, and CSV export.

**Total prompts used:** 7

---

## Prompt 1 — Auth + layout shell

```
Build the foundation for a CRM app called "Pipeline" for B2B sales teams.

Tech stack:
- React + Vite + TypeScript
- Tailwind CSS + shadcn/ui
- Supabase for auth and database
- React Router for navigation

Authentication:
- Email + password only
- /login page: centered card, company logo, email + password fields, "Forgot password?" link
- Protected routes: all /app/* routes redirect to /login if unauthenticated
- useAuth() hook from @/hooks/useAuth: { user, session, signIn, signOut, loading }

Layout shell for /app/* routes:
- Fixed left sidebar (240px)
  - Logo "Pipeline" at top
  - Nav links: Dashboard, Contacts, Deals, Companies, Activities, Reports
  - User name + email at bottom + sign-out button
- Top header: page title (dynamic) + global search (Cmd+K shortcut) + notification bell
- Main content area: scrollable, gray-50 background

Mobile: hamburger icon in header opens a sidebar drawer.

Acceptance criteria:
- [ ] /login renders and auth works end-to-end
- [ ] All /app/* routes redirect to /login when unauthenticated
- [ ] Sidebar nav links are active-styled for the current route
- [ ] Cmd+K focuses the global search input
```

---

## Prompt 2 — Contacts database

```
Build the Contacts page at /app/contacts in Pipeline.

Supabase table:
contacts (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users on delete cascade,
  first_name text not null,
  last_name text not null,
  email text,
  phone text,
  company_name text,
  job_title text,
  status text default 'lead' check (status in ('lead','prospect','customer','churned')),
  owner_id uuid references auth.users,
  tags text[],
  created_at timestamptz default now(),
  updated_at timestamptz default now()
)

RLS: users can only access contacts where user_id = auth.uid().

Contacts table UI:
- Columns: checkbox, Avatar initials, Full name, Email, Company, Job title, Status badge, Owner, Created date
- Status badge colors: lead=gray, prospect=blue, customer=green, churned=red
- Sticky header row, virtualized rows if count > 500 (use react-virtual)
- Click a row to open a contact detail slide-over (build in a later prompt)

Filter and search bar above the table:
- Text search (searches name, email, company) — debounced 300ms, fetches from Supabase
- Status filter (multiselect dropdown checkboxes)
- Owner filter (multiselect, populated from the users table)
- Tags filter (multiselect)
- "Clear filters" link, visible only when any filter is active

Sorting:
- Clickable column headers: Name (asc/desc), Company (asc/desc), Created (asc/desc, default: desc)
- Sort state persists in URL params (?sort=created_at&dir=desc)

Bulk actions:
- Select all (header checkbox) + individual row checkboxes
- Sticky bulk action bar at the bottom (appears when ≥1 row is selected):
  - "X contacts selected"
  - "Change status" dropdown
  - "Assign owner" dropdown
  - "Delete" button (confirm dialog)
  - "Export CSV" button

Add contact:
- "Add contact" button top-right opens a modal with all fields
- Form validates required fields (first name, last name) on submit
- On success: closes modal, appends new contact to the top of the table

Acceptance criteria:
- [ ] Table loads contacts from Supabase on mount with pagination (50 per page)
- [ ] Text search filters results correctly with 300ms debounce
- [ ] Bulk status change updates all selected rows in Supabase in one request
- [ ] Sorting updates the URL param and re-fetches
- [ ] "Add contact" modal saves to Supabase and the new row appears at the top
- [ ] Export CSV downloads a .csv file with all visible (filtered) contacts
```

---

## Prompt 3 — Contact detail panel

```
Add a contact detail slide-over panel to the Contacts page in Pipeline.

Trigger: clicking a contact row opens the panel.

Panel layout: 480px wide, slides from the right. Full-screen on mobile.

Panel sections:

Header:
- Avatar initials circle (64px, random indigo shade seeded by contact id)
- Full name (editable inline)
- Job title + company (editable inline)
- Status badge with a dropdown to change status

Contact info section:
- Email (with mailto: link + copy icon)
- Phone (with tel: link + copy icon)
- Each field is editable: click to open an inline input, Enter or blur to save

Activity timeline (scrollable, most recent first):
- Each activity: icon (type-based), description, timestamp relative ("2h ago", "3 days ago")
- Types: note, email, call, meeting, deal stage change
- "Add note" inline at the top of the timeline: textarea + submit button

Deals section:
- Lists all deals associated with this contact
- Each deal: name, stage badge, value, close date
- "Add to deal" button (dropdown of existing deals or "Create new deal")

Tabs at the top of the panel: Overview (default), Activities, Deals, Files

Auto-save:
- Edits to name, job title, company, email, phone save on blur with a "Saving…" / "Saved" indicator

Acceptance criteria:
- [ ] Panel opens on row click and shows correct contact data
- [ ] Inline edits save to Supabase on blur
- [ ] Status change updates the badge and the table row simultaneously
- [ ] Adding a note posts to the activities table and appears at the top of the timeline
- [ ] Tab navigation works without closing the panel
- [ ] Panel closes on Escape and outside click
```

---

## Prompt 4 — Deal pipeline (kanban)

```
Build the Deals page at /app/deals in Pipeline as a Kanban board.

Supabase table:
deals (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users on delete cascade,
  title text not null,
  value numeric default 0,
  currency text default 'USD',
  stage text not null default 'lead' check (stage in ('lead','qualified','proposal','negotiation','won','lost')),
  contact_id uuid references contacts(id),
  close_date date,
  probability smallint default 0 check (probability between 0 and 100),
  owner_id uuid references auth.users,
  position integer not null default 0,
  created_at timestamptz default now()
)

Kanban columns (in order): Lead, Qualified, Proposal, Negotiation, Won, Lost

Each column:
- Column header: stage name + deal count + total value sum (formatted as $12.4k)
- Deal cards (draggable, ordered by position within the column)
- "Won" column has a green header background; "Lost" column has a red header background

Each deal card:
- Deal title
- Contact name (if linked)
- Value (bold, formatted as currency)
- Close date (red if overdue)
- Probability badge
- Owner avatar (small, 24px)

Drag and drop:
- Library: @dnd-kit/core + @dnd-kit/sortable
- Cards can be dragged within a column (reorder by position) or between columns (changes stage)
- Dragging between columns: update stage + recalculate position in Supabase
- Optimistic update: move the card immediately, revert if Supabase update fails

Add deal:
- "+ Add deal" button at the bottom of each column opens an inline form within that column
- Fields: title (required), value, contact (searchable dropdown), close date, probability
- Saves to Supabase and appends to the bottom of the column

Click a deal card to open a deal detail slide-over (same pattern as contacts, build details later).

Board-level filters above the board:
- Owner filter (multiselect)
- Close date filter (this month / next month / this quarter / custom range)

Acceptance criteria:
- [ ] All 6 columns render with correct deal counts and value sums
- [ ] Dragging a card between columns updates the stage in Supabase
- [ ] Dragging within a column updates the position in Supabase
- [ ] Optimistic revert works: if Supabase update fails, the card returns to its original position
- [ ] Value sums update immediately when a card is moved
- [ ] "+ Add deal" form in each column inserts correctly into that column's stage
```

---

## Prompt 5 — Activity feed

```
Build the Activities page at /app/activities in Pipeline.

This is a unified feed of all CRM activity across contacts and deals.

Supabase table:
activities (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users on delete cascade,
  type text not null check (type in ('note','email','call','meeting','task','deal_stage_change')),
  title text,
  body text,
  contact_id uuid references contacts(id),
  deal_id uuid references deals(id),
  due_at timestamptz,
  completed boolean default false,
  created_at timestamptz default now()
)

Feed layout:
- Timeline: vertical line on the left, activity items to the right
- Each item: type icon (colored circle), description, linked contact/deal (clickable), timestamp
- Group by date: "Today", "Yesterday", "Monday, Jan 13", etc.

Filters above the feed:
- Type filter: All / Notes / Emails / Calls / Meetings / Tasks
- Date range: Last 7 days / Last 30 days / All time / Custom
- Owner filter

Log activity button (top-right):
- Opens a modal with fields:
  - Type (select)
  - Title
  - Body / notes (textarea)
  - Link to contact (searchable dropdown)
  - Link to deal (searchable dropdown)
  - Due date (for tasks only — show/hide based on type)
- On save: inserts into activities table and prepends to the feed

Tasks section (sidebar panel on desktop, below feed on mobile):
- "My tasks" list: activities where type = 'task' AND completed = false AND owner_id = current user
- Each task: title, linked contact/deal, due date (red if overdue), checkbox to complete
- Completing a task sets completed = true and removes it from the list with a checkmark animation

Acceptance criteria:
- [ ] Feed loads all activities grouped by date, most recent first
- [ ] Filtering by type shows only activities of that type
- [ ] Logging a new activity adds it to the top of the feed immediately
- [ ] Tasks in the sidebar show only incomplete tasks owned by the current user
- [ ] Completing a task removes it from the sidebar with an animation
- [ ] Clicking a linked contact name opens the contact detail panel
```

---

## Prompt 6 — Reports and charts

```
Build the Reports page at /app/reports in Pipeline.

Layout: a dashboard-style grid of chart cards.

Charts to include:

1. Revenue by stage (bar chart)
   - X-axis: deal stages (Lead, Qualified, Proposal, Negotiation, Won, Lost)
   - Y-axis: total deal value in USD
   - Data: GET /api/reports/revenue-by-stage
   - Bar color: indigo, except "Won" = green and "Lost" = red

2. Deals won over time (line chart)
   - X-axis: months for the last 12 months
   - Y-axis: count of deals won
   - Data: GET /api/reports/won-over-time
   - Date range picker: "Last 3 months", "Last 6 months", "Last 12 months" (default)

3. Pipeline velocity (number + trend)
   - Large number: average days to close a won deal
   - Trend: "+2 days vs last period" (red if slower, green if faster)
   - Data: GET /api/reports/pipeline-velocity

4. Top contacts by deal value (horizontal bar chart)
   - Top 10 contacts, sorted by total value of their associated won deals
   - Each bar labeled with contact name
   - Data: GET /api/reports/top-contacts

5. Win rate by owner (grouped bar chart)
   - X-axis: team members
   - Y-axis: percentage of deals won
   - Data: GET /api/reports/win-rate-by-owner

Chart library: Recharts (already installed)

Global date range filter at the top of the page applies to all charts simultaneously.

Each chart is in a white card with a title, optional subtitle, and a "Download PNG" button (uses html2canvas).

Loading state: skeleton placeholder (same dimensions as the chart) while data fetches.

Acceptance criteria:
- [ ] All 5 charts render with correct data from their respective API endpoints
- [ ] Changing the global date range refetches all charts simultaneously
- [ ] "Download PNG" produces a downloadable image of the chart card
- [ ] Skeleton loaders appear on initial load and on date range change
- [ ] Charts are readable on a 1280px wide screen without horizontal scroll
```

---

## Prompt 7 — CSV export and data import

```
Add bulk CSV export and import functionality to Pipeline.

Export (on Contacts page):
- "Export" button above the contacts table (top-right, next to "Add contact")
- Exports all contacts matching the current filters (not just the current page)
- Columns in export: First Name, Last Name, Email, Phone, Company, Job Title, Status, Owner, Tags, Created At
- Format: UTF-8 CSV, with headers in the first row
- File name: pipeline-contacts-YYYY-MM-DD.csv
- Do not use a server route — generate the CSV in the browser from the fetched data

Import (on Contacts page):
- "Import" button next to Export opens a 3-step modal:

  Step 1 — Upload:
  - Drag-and-drop or click-to-browse for a .csv file
  - Parse the CSV in the browser (use papaparse)
  - Show a preview of the first 5 rows after upload

  Step 2 — Map columns:
  - Left column: detected CSV column headers
  - Right column: dropdown mapping to Pipeline contact fields
  - Auto-detect common header names (e.g., "Email Address" → email, "First" → first_name)
  - Required field indicators: first_name and last_name must be mapped before proceeding

  Step 3 — Import:
  - "Import X contacts" button (X = row count from the CSV)
  - Progress bar while inserting (batch inserts of 100 rows at a time via Supabase)
  - Results summary: "X imported successfully, Y skipped (duplicate email)"
  - "Done" closes the modal and refreshes the contacts table

Acceptance criteria:
- [ ] Export downloads a .csv with all matching contacts (not just visible rows)
- [ ] Import modal opens and accepts .csv files
- [ ] Column mapping auto-detects email, first_name, last_name correctly
- [ ] Import fails gracefully if required fields (first_name, last_name) are not mapped
- [ ] Progress bar advances during batch import
- [ ] Results summary shows correct counts for imported and skipped rows
- [ ] Contacts table refreshes after a successful import
```

---

## Result

After these 7 prompts, Pipeline is a production-ready CRM with:

- Role-protected auth and a consistent layout shell
- Contacts database with search, filters, bulk actions, and inline editing
- Kanban deal pipeline with drag-and-drop and optimistic updates
- Unified activity feed with task management
- Analytics reports with 5 chart types and PNG export
- CSV export and import with column mapping and progress tracking

Total build time: approximately 4–5 hours using Lovable.
