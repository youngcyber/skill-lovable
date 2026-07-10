---
name: lovable-saas-planner
description: "Generate a phase-by-phase Lovable prompt plan for building a full-stack SaaS web application — from stack setup through auth, core features, billing, and admin panel."
---

# Lovable SaaS Planner

Turn a SaaS app idea into a complete, sequenced build plan. SaaS apps are built in strict layers: stack → auth → schema → layout → features → billing. Skipping or combining layers produces apps that need expensive refactors.

## When to Use

- User wants to build a SaaS product with Lovable
- User describes an app with user accounts, persistent data, and subscription plans
- User asks "how do I build [SaaS idea] with Lovable?"
- User has a SaaS in progress and wants to know what phase to build next

## Input Schema

```yaml
app_name: string          # REQUIRED — name of the SaaS
app_description: string   # REQUIRED — what it does and who it's for
core_features: list       # REQUIRED — 3-5 main things users do
auth_methods: list        # OPTIONAL — [email, google, magic-link] — default: [email]
paid_plans: boolean       # OPTIONAL — default: false
plan_names: list          # OPTIONAL — e.g. [Free, Pro, Business] — required if paid_plans: true
user_roles: list          # OPTIONAL — [user, admin] — default: [user]
onboarding_steps: list    # OPTIONAL — steps in the first-run wizard
```

## Workflow

### Step 1: Clarify the Idea

If `core_features` is missing or vague, ask: "What are the 3 main things a user does in this app?" Do not generate phases until you have at least 3 concrete features.

### Step 2: Select Phases

```
Phase 1  → Stack Declaration
Phase 2  → Authentication + Profiles
Phase 3  → Database Schema
Phase 4  → App Layout Shell
Phase 5  → Onboarding Flow              (recommended for all SaaS)
Phase 6  → Core Feature #1
Phase 7  → Core Feature #2
Phase 8  → Core Feature #3              (repeat for each core feature)
Phase 9  → Settings Page
Phase 10 → Billing + Stripe            (if paid_plans: true)
Phase 11 → Admin Panel                 (if user_roles has admin)
Phase 12 → Empty States + Error Handling
Phase 13 → Polish + Animations
```

**Never break these rules:**
- Phase 1 (Stack) always first
- Phase 2 (Auth) always before any feature
- Phase 3 (Schema) always before the first feature that reads data
- Phase 5 (Onboarding) always after Phase 4 (Layout) — it runs inside the shell
- Phases 9–11 always after all core features work
- Phase 12–13 always last

### Step 3: Write Each Phase Prompt

**Phase 1 — Stack Declaration template:**
```
Set up a new project called "[APP NAME]" — [ONE-LINE DESCRIPTION].

Tech stack (use exactly this, do not substitute):
- React + Vite + TypeScript
- Tailwind CSS
- shadcn/ui for all components
- Supabase for database and auth
- React Router v6
- Lucide React for icons

Create:
- App.tsx with React Router configured
- Placeholder /app/dashboard route showing "[APP NAME]" centered on screen
- Tailwind with default neutral palette

Done when:
- [ ] Project runs without TypeScript errors
- [ ] /app/dashboard renders "[APP NAME]" centered
- [ ] shadcn/ui Button imports and renders without error
```

**Phase 2 — Auth template:**
```
Add authentication to [APP NAME] using Supabase Auth.

Auth method: [METHOD]
Pages: /login, /signup, /forgot-password
Post-login route: [ROUTE]

Hook: useAuth() at src/hooks/useAuth.ts
Returns: { user, session, loading, signIn, signUp, signOut }

Supabase profiles table:
profiles (
  id uuid references auth.users primary key,
  full_name text,
  avatar_url text,
  role text default 'user' check (role in ([ROLES])),
  onboarded boolean default false,
  created_at timestamptz default now()
)
RLS: SELECT and UPDATE only for own row.
Trigger: on auth.users INSERT → INSERT into profiles(id, full_name).

Design: centered card, max-w-[400px], white bg, shadow-md.
Inline field errors. Submit button disabled while in-flight.

Done when:
- [ ] Signup creates auth user + profiles row
- [ ] Login redirects to [POST_LOGIN_ROUTE]
- [ ] /app/dashboard when logged out → /login?next=/app/dashboard
- [ ] useAuth() importable from src/hooks/useAuth
```

**Phase 3 — Schema template:**
```
Create the database schema for [APP NAME] in Supabase.

[TABLE definitions with types, constraints, and foreign keys]

RLS on every table: SELECT/INSERT/UPDATE/DELETE for user_id = auth.uid() only.
Trigger: update_updated_at() on any table with updated_at column.

Do not build any UI in this phase.

Done when:
- [ ] All tables exist with correct column types
- [ ] RLS is enabled on every table
- [ ] A different user's SELECT is blocked by RLS
- [ ] updated_at auto-updates on row change
```

**Phase 4 — Layout Shell template:**
```
Build the layout shell for /app/* routes in [APP NAME].

Sidebar (fixed, [WIDTH]px, desktop):
- Logo top-left
- Nav links: [Icon → Label → /route]
- Active: bg-indigo-50 text-indigo-700 rounded-lg
- Bottom: user avatar + name + sign-out
- Collapsible to 64px icon-only; state in localStorage

Header (h-16, sticky): page title left, [extras] right
Mobile: hamburger → left drawer, closes on nav click
Main content: flex-1 overflow-y-auto bg-gray-50 p-6 — leave empty

Done when:
- [ ] Layout renders on 375px / 768px / 1280px
- [ ] Collapse persists across page refreshes
- [ ] All nav links work with correct active state
- [ ] Sign-out redirects to /login
```

For core feature phases, write a full prompt including Supabase queries, component names, loading/empty/error states, and 3–5 done-when checkboxes.

### Step 4: Output the Plan

List all phases in order with complete prompts and checklists.

## Output Schema

```yaml
plan_title: string          # "[App Name] — SaaS Build Plan"
app_summary: string         # One sentence confirming the understood idea
total_phases: integer
phases:
  - number: integer
    name: string
    goal: string
    prompt: string           # Complete ready-to-paste Lovable prompt
    done_when: list          # 3-5 specific testable checkboxes
closing_note: string
```

## Output Format

```
# [App Name] — SaaS Build Plan
> [One sentence confirming app purpose and key features]

---

## Phase 1 — Stack Declaration
**Goal:** Lock in the tech stack before Lovable makes choices.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] ...

---

**Total phases: N**
Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2.
```

## Error Handling

- **Vague features** — ask "What are the 3 main things a user does in this app?" before generating phases
- **paid_plans: true but no plan names** — ask for plan names and prices before writing Phase 10
- **Many features (6+)** — generate one phase per feature; do not combine multiple features into one prompt

## Examples

### Example 1: Project Management SaaS

**Input:** App: Taskly, features: [create projects, assign tasks, track progress], auth: email, paid: false, roles: [user]

**Phases:** 1. Stack, 2. Auth, 3. Schema (projects, tasks), 4. Layout Shell, 5. Onboarding, 6. Projects list, 7. Task board, 8. Progress tracking, 9. Settings, 10. Empty States, 11. Polish

**Total: 11 phases**

### Example 2: Paid SaaS with Admin

**Input:** App: Formly, features: [build forms, collect responses, view analytics], auth: [email, google], paid: true, plans: [Free, Pro], roles: [user, admin]

**Phases:** 1. Stack, 2. Auth (email + Google), 3. Schema, 4. Layout, 5. Onboarding, 6. Form builder, 7. Response collection, 8. Analytics, 9. Settings, 10. Billing (Stripe), 11. Admin Panel, 12. Empty States, 13. Polish

**Total: 13 phases**
