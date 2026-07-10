# Real-World Example: To-Do App

A complete walkthrough of building a production-quality to-do app from blank canvas to deployed app using Lovable. Each prompt below was used in sequence.

**Final app features:** Authentication, multiple lists, drag-to-reorder tasks, due dates, priorities, and real-time sync via Supabase.

**Total prompts used:** 7

---

## Prompt 1 — Project scaffold + auth

```
Create a new React app called "Tasklane" — a personal task manager.

Tech stack:
- React + Vite
- Tailwind CSS
- shadcn/ui for components
- Supabase for auth and database
- React Router for navigation

Set up Supabase authentication with:
- Email + password sign up and login
- /login page (centered card, max-width 400px)
- /signup page (same layout, adds a "Full name" field)
- Protected routes: any route that starts with /app redirects to /login if unauthenticated
- After login/signup, redirect to /app/inbox
- A useAuth() hook exported from @/hooks/useAuth that exposes: user, signIn, signUp, signOut, loading

Create this Supabase table:
profiles (
  id uuid references auth.users primary key,
  full_name text,
  created_at timestamptz default now()
)

Row-level security: users can only read and update their own profile row.

Acceptance criteria:
- [ ] /login and /signup render at those routes
- [ ] Signing up creates a row in profiles
- [ ] Accessing /app/inbox when logged out redirects to /login
- [ ] useAuth() is importable and exposes the correct API
```

---

## Prompt 2 — App layout shell

```
Build the layout shell for the /app/* routes in Tasklane.

Layout:
- Fixed left sidebar, 256px wide
- Main content area fills the rest, scrollable

Sidebar contents:
- "Tasklane" logo (text, indigo-600, font-bold) at the top
- Navigation section labeled "My Lists":
  - Inbox (always present)
  - Today (always present)
  - Upcoming (always present)
  - Divider
  - User-created lists will be rendered here later (leave a placeholder comment)
- "Add list" button at the bottom of the nav section (icon: Plus, text: "New list")
- User section at the very bottom: avatar initials in a circle + full name + sign-out button

Mobile:
- Sidebar hidden, accessible via hamburger icon in a top header bar
- Drawer overlay, closes on nav click or outside tap

Acceptance criteria:
- [ ] Layout renders on /app/inbox without errors
- [ ] Sign-out button calls signOut() from useAuth() and redirects to /login
- [ ] Mobile drawer opens and closes correctly
- [ ] Inbox, Today, and Upcoming links navigate to their respective routes
```

---

## Prompt 3 — Task list + create task

```
Build the Inbox view at /app/inbox in Tasklane.

Data model — add this Supabase table:
tasks (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id) on delete cascade,
  list_id uuid references lists(id) on delete cascade nullable,
  title text not null,
  notes text,
  due_date date,
  priority smallint default 0, -- 0=none, 1=low, 2=medium, 3=high
  completed boolean default false,
  position integer not null default 0,
  created_at timestamptz default now()
)

RLS: users can only CRUD their own tasks (user_id = auth.uid()).

Inbox behavior: shows all tasks where list_id IS NULL and completed = false, ordered by position.

Task list UI:
- Each task row: checkbox (marks complete), title, priority color dot (gray/blue/yellow/red for none/low/medium/high), due date chip (red if overdue)
- Clicking the title opens an inline edit field
- Completing a task: row fades out and is removed from the list after 600ms
- Empty state: "No tasks in your inbox. Add one below."

Add task:
- Persistent input bar at the bottom of the task list
- Fields: task title (required), quick-set priority (P1/P2/P3 buttons toggle), due date (date picker popover)
- Pressing Enter or clicking the "Add" button creates the task via Supabase insert
- Input clears after successful creation

Acceptance criteria:
- [ ] Tasks load from Supabase on mount
- [ ] Creating a task inserts it into Supabase and it appears in the list immediately
- [ ] Completing a task updates completed=true in Supabase and removes it from view after 600ms
- [ ] Overdue tasks show the due date chip in red
- [ ] Empty state renders when no tasks exist
```

---

## Prompt 4 — Task detail panel

```
Add a task detail slide-over panel to Tasklane.

Trigger: clicking anywhere on a task row (except the checkbox) opens the panel.

Panel contents:
- Task title (editable inline, large text, no border until focused)
- Notes (textarea, placeholder "Add notes…", auto-resize to content)
- Due date picker (shadcn Popover with Calendar)
- Priority selector (segmented buttons: None / Low / Medium / High)
- "Move to list" dropdown (populated with user's lists + "Inbox" option)
- "Delete task" button at the bottom (red, opens a confirm dialog before deleting)

Auto-save:
- Changes to title, notes, due date, and priority save automatically 800ms after the user stops typing/selecting (debounced PATCH to Supabase)
- A "Saved" indicator fades in for 2 seconds after each successful save

Panel layout:
- Slides in from the right, 380px wide on desktop
- Full-screen on mobile (slides up from bottom)
- Clicking outside or pressing Escape closes the panel
- Closing does not discard unsaved changes — the debounce fires immediately on close

Acceptance criteria:
- [ ] Panel opens when a task row is clicked (not when the checkbox is clicked)
- [ ] All fields reflect the current task data on open
- [ ] Changes save to Supabase after 800ms of inactivity
- [ ] "Saved" indicator appears after each save
- [ ] Deleting a task removes it from the list and closes the panel
- [ ] Panel closes on Escape and outside click
```

---

## Prompt 5 — Drag-to-reorder tasks

```
Add drag-to-reorder functionality to the task list in Tasklane.

Library: use @dnd-kit/core and @dnd-kit/sortable (install if not present).

Behavior:
- Tasks in any list view can be dragged vertically to reorder
- Drag handle: a GripVertical icon (Lucide) on the left of each task row, visible on hover
- While dragging, the dragged item shows a shadow and slight scale (scale-102)
- Other items animate into their new positions as the dragged item moves over them
- On drop, update the position field for all affected tasks in Supabase in a single batch upsert

Optimistic update:
- Reorder the list in local state immediately on drop (do not wait for Supabase)
- If the Supabase update fails, revert to the previous order and show a toast error

Acceptance criteria:
- [ ] Drag handle appears on task row hover
- [ ] Dragging reorders tasks visually with smooth animation
- [ ] Dropping persists the new order to Supabase
- [ ] Local state updates immediately on drop (no flicker)
- [ ] On Supabase error, the order reverts and a toast appears
```

---

## Prompt 6 — Custom lists

```
Add custom task lists to Tasklane.

Data model — add this Supabase table:
lists (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id) on delete cascade,
  name text not null,
  color text default 'indigo',
  position integer not null default 0,
  created_at timestamptz default now()
)

RLS: users can only CRUD their own lists.

"Add list" flow:
- Clicking "New list" in the sidebar opens an inline input field directly in the sidebar nav (not a modal)
- User types the list name and presses Enter or clicks a checkmark to save
- Pressing Escape cancels without saving
- On save, insert into lists and add the new list to the sidebar nav immediately

List page at /app/list/:listId:
- Same layout as Inbox but filtered to tasks where list_id = :listId
- Page title = list name (editable inline — click to edit, blur or Enter to save)
- List color shown as a color dot next to the title with a color picker popover on click (8 preset colors)

Delete list:
- "…" menu on each list item in the sidebar: Rename, Change color, Delete
- Deleting a list opens a confirm dialog: "Delete [name]? All tasks in this list will also be deleted."
- On confirm, deletes the list (tasks cascade delete via FK)

Acceptance criteria:
- [ ] New list appears in the sidebar immediately after creation
- [ ] Navigating to /app/list/:listId shows only that list's tasks
- [ ] Renaming the list via the page title updates the sidebar label
- [ ] Deleting a list with 3 tasks shows the confirmation dialog mentioning the task count
- [ ] Sidebar list order matches the position field from Supabase
```

---

## Prompt 7 — Today and Upcoming views

```
Build the Today and Upcoming views in Tasklane.

Today view at /app/today:
- Shows all tasks where due_date = today AND completed = false, across all lists
- Groups tasks by list: each group has the list name as a header (or "Inbox" if list_id is null)
- Shows the count of tasks due today in the sidebar nav badge next to "Today"
- Empty state: "Nothing due today. Enjoy your day."

Upcoming view at /app/upcoming:
- Shows all tasks where due_date >= tomorrow AND completed = false, ordered by due_date asc
- Groups tasks by date: section headers formatted as "Tomorrow", "Monday, Jan 13", "Tuesday, Jan 14", etc.
- "This week" section and "Later" section if tasks span more than 7 days
- Empty state: "No upcoming tasks. Add a due date to a task to see it here."

Shared behavior for both views:
- Tasks can be completed inline (same checkbox behavior as Inbox)
- Clicking a task opens the detail panel
- Tasks cannot be drag-reordered in these views (they are sorted by due date)

Acceptance criteria:
- [ ] Today view shows only tasks due today across all lists
- [ ] Today badge count updates when a task is completed
- [ ] Upcoming view groups tasks by date with correct relative labels
- [ ] Completing a task in either view removes it from the view after 600ms
- [ ] Both views show their empty states when no tasks qualify
```

---

## Result

After these 7 prompts, Tasklane is a fully functional personal task manager with:

- Supabase auth and RLS-protected data
- Inbox, Today, Upcoming, and custom list views
- Task creation, editing, completion, and deletion
- Drag-to-reorder with optimistic updates
- Debounced auto-save on the detail panel
- Responsive layout with mobile drawer navigation

Total build time: approximately 90 minutes using Lovable.
