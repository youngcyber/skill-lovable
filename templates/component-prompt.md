# Component Prompt Template

Use this template when asking Lovable to build a single UI component.

## Template

```
Build a [COMPONENT NAME] for [APP NAME].

Context:
- User: [WHO is using this — e.g., "an authenticated admin user"]
- Goal: [WHAT they need to do — e.g., "review pending support tickets"]
- Placement: [WHERE in the app — e.g., "on the /admin/tickets page, below the page header"]

Design requirements:
- Stack: [e.g., React + Tailwind CSS + shadcn/ui]
- Responsive: [e.g., "mobile-first; collapses to a stacked layout below 768px"]
- Theme: [e.g., "match the neutral gray palette already used in the app"]

Behavior:
- [Describe interaction 1 — e.g., "Clicking a row opens a slide-over panel with ticket details"]
- [Describe interaction 2 — e.g., "Status badge changes color: yellow = open, green = resolved, red = urgent"]
- [Describe interaction 3]

Data:
- [Describe the data shape — e.g., "Each ticket has: id, subject, status, createdAt, assignee (name + avatar URL)"]
- [State data source — e.g., "Fetch from GET /api/tickets. Show a skeleton loader while fetching."]

Acceptance criteria:
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]
```

## When to Use

Use this template when:
- Building a new, self-contained component (table, card, modal, form, etc.)
- The component has meaningful interactivity beyond static display
- You want to lock in tech stack and responsive behavior upfront

## Example Output

**Filled prompt:**

```
Build a TicketTable component for HelpDesk Pro.

Context:
- User: an authenticated admin logged into the support dashboard
- Goal: scan all open tickets and reassign or resolve them quickly
- Placement: on the /admin/tickets page, below the page header and filter bar

Design requirements:
- Stack: React + Tailwind CSS + shadcn/ui
- Responsive: mobile-first; below 640px show cards instead of a table
- Theme: neutral gray background, indigo accent for selected rows

Behavior:
- Clicking a row opens a shadcn Sheet (slide-over) with full ticket details
- Status badge colors: yellow = open, green = resolved, red = urgent, gray = closed
- Sortable columns: Subject, Created At, Status. Click header to toggle asc/desc.
- Bulk-select via checkboxes; "Assign to me" button appears in a sticky footer when ≥1 row is selected

Data:
- Each ticket: { id, subject, status, createdAt, assignee: { name, avatarUrl } }
- Fetch from GET /api/tickets?page=1&limit=25. Show a skeleton loader (5 rows) while fetching.
- On error, show an inline error state with a "Retry" button.

Acceptance criteria:
- [ ] Table renders within a Card with a visible border and subtle shadow
- [ ] Skeleton loader appears on initial load and disappears when data arrives
- [ ] Clicking a row opens the slide-over without a full page reload
- [ ] Sorting updates the displayed order without a new network request
- [ ] Bulk select footer animates in from the bottom when rows are checked
```

**What gets built:** A fully interactive support ticket table with sortable columns, row-level slide-over detail panel, color-coded status badges, responsive card fallback on mobile, and a bulk-action footer — all wired to a real API endpoint with loading and error states.
