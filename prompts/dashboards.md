# Dashboard Prompts

Patterns for building data-heavy admin dashboards and analytics views with Lovable.

## Overview

Dashboards require a clear separation between layout, charts, and data fetching. Prompt for the layout shell first, then individual chart/widget blocks, then wire up real data last. Mixing all three in one prompt produces brittle results.

---

## Pattern 1: Dashboard Layout Shell

### Prompt

```
Build the layout shell for the [APP NAME] admin dashboard at /dashboard.

Layout structure:
- Fixed left sidebar (240px wide on desktop, collapsible to icon-only at 64px)
- Top header bar (64px tall, sticky)
- Main content area (scrollable, fills remaining space)

Sidebar contents:
- Logo at top
- Navigation links: Dashboard, [SECTION 2], [SECTION 3], [SECTION 4], [SECTION 5]
- Active link style: indigo background pill, white text
- Collapse toggle button at the bottom of the sidebar (chevron icon)
- User avatar + name at the very bottom with a dropdown for Settings and Sign out

Header contents:
- Page title (dynamic, changes per route)
- Global search input (cmd+K shortcut to focus)
- Notification bell icon with unread badge
- User avatar (same as sidebar — clicking opens the same dropdown)

Mobile behavior:
- Sidebar hidden by default, opens as a full-screen drawer via a hamburger icon in the header
- Drawer closes when a nav link is clicked or when the overlay is tapped

Tech:
- React + Tailwind CSS + shadcn/ui
- Use React Router for navigation
- Sidebar state (expanded/collapsed) persists in localStorage

Acceptance criteria:
- [ ] Layout renders correctly on 375px (mobile), 768px (tablet), 1280px (desktop)
- [ ] Sidebar collapse toggle persists across page refreshes
- [ ] Cmd+K focuses the search input
- [ ] Mobile drawer closes on nav link click
- [ ] All nav links render correctly and navigate to their routes
- [ ] Active link is visually distinct from inactive links
```

### When to Use

Always build the layout shell before adding dashboard content. Every subsequent dashboard prompt will reference the main content area of this shell.

---

## Pattern 2: KPI Metrics Row

### Prompt

```
Add a KPI metrics row at the top of the /dashboard main content area.

Metrics to display (4 cards in a row):
1. Total Revenue — value from GET /api/metrics/revenue, format as currency ($12,430)
2. Active Users — value from GET /api/metrics/users, format as integer with comma separator
3. Churn Rate — value from GET /api/metrics/churn, format as percentage (4.2%)
4. MRR Growth — value from GET /api/metrics/growth, format as percentage with + or - prefix (+8.1%)

Each card:
- Icon (Lucide): Revenue → DollarSign, Users → Users, Churn → TrendingDown, Growth → TrendingUp
- Metric label (small, muted text above the number)
- Value (large, bold)
- Delta vs last period below the value: "+12% vs last month" — green if positive, red if negative
- Subtle skeleton loader while fetching

Layout:
- 4 columns on desktop (lg:grid-cols-4)
- 2 columns on tablet (md:grid-cols-2)
- 1 column on mobile
- Cards: white background, 1px border, rounded-xl, no shadow

Error state:
- If any individual metric fails to load, show "—" as the value with a muted "Unavailable" label
- Do not fail the entire row if one card errors

Acceptance criteria:
- [ ] All 4 metrics render with correct formatting
- [ ] Skeleton loaders appear on initial load
- [ ] Delta is green for positive, red for negative, neutral gray for zero
- [ ] A single card error shows "—" without breaking the other cards
- [ ] Row is responsive across all three breakpoints
```

### When to Use

Use this as the first content block inside any dashboard. KPI cards establish the visual language for the rest of the dashboard.

---

## Pattern 3: Chart Block with Date Range Filter

### Prompt

```
Add a revenue over time line chart to the /dashboard page, below the KPI row.

Chart:
- Library: Recharts (already installed)
- Type: AreaChart with a gradient fill (indigo, opacity 0.15 at the bottom)
- X-axis: dates, formatted as "Jan 12" or "Feb 3" depending on the selected range
- Y-axis: dollar values, formatted as "$12k" (abbreviated)
- Tooltip: appears on hover, shows date + exact revenue value formatted as $12,430
- Data source: GET /api/metrics/revenue-chart?from=YYYY-MM-DD&to=YYYY-MM-DD

Date range filter:
- Preset buttons: "7D", "30D", "90D", "12M" (default: 30D)
- Custom range: a date picker that opens when "Custom" is clicked
- Selecting a range refetches the chart data and updates the URL params (?range=30d or ?from=&to=)
- Active preset button is visually highlighted (indigo background)

Loading state:
- Replace chart with a pulsing gray placeholder (same dimensions as the chart) while fetching
- Do not unmount the chart on refetch — keep the previous data visible with reduced opacity while new data loads

Layout:
- Full width of the content area
- Chart card: white background, 1px border, rounded-xl
- Chart height: 280px on desktop, 200px on mobile
- Filter buttons right-aligned in the card header

Acceptance criteria:
- [ ] Default 30D data loads on first render
- [ ] Selecting a preset updates the chart and the URL param
- [ ] Custom date range updates the chart on selection
- [ ] Previous data stays visible (at 40% opacity) while new data loads
- [ ] Tooltip shows correct date and formatted revenue value
- [ ] Chart is readable on mobile (no clipped axes)
```

### When to Use

Use after the KPI row is in place. This chart block is reusable for any time-series metric — swap the endpoint and Y-axis format for MRR, signups, or any other measure.
