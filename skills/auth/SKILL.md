---
name: lovable-auth-planner
description: "Generate a phase-by-phase Lovable prompt plan for implementing authentication — email/password, Google OAuth, magic link, protected routes, and role-based access control using Supabase Auth."
---

# Lovable Auth Planner

Turn an auth requirement into a sequenced set of Lovable prompts. Auth is always built in layers: foundation first, then login UI, then OAuth, then route guards, then roles.

## When to Use

- User needs to add login or signup to a Lovable app
- User asks about protecting routes or redirecting unauthenticated users
- User needs Google OAuth, magic link, or passwordless login
- User needs different access levels for different user types (admin, member, etc.)
- User says "how do I add auth to Lovable?"

## Input Schema

```yaml
app_name: string          # REQUIRED — name of the app
auth_methods: list        # REQUIRED — [email, google, magic-link] — pick one or more
post_login_route: string  # REQUIRED — where to redirect after login, e.g. /app/dashboard
user_roles: list          # OPTIONAL — e.g. [user, admin] — default: [user]
profile_fields: list      # OPTIONAL — extra fields beyond email, e.g. [full_name, avatar_url, plan]
existing_project: boolean # OPTIONAL — adding to existing app or starting fresh
```

## Workflow

### Step 1: Clarify Requirements

If `auth_methods` or `post_login_route` is missing, ask for them. Do not generate prompts until you know where the user lands after login.

### Step 2: Select Phases

```
Phase 1 → Supabase Setup + Profiles Table
Phase 2 → Email + Password Auth          (if email in auth_methods)
Phase 3 → Google OAuth                   (if google in auth_methods)
Phase 4 → Magic Link                     (if magic-link in auth_methods)
Phase 5 → Protected Routes + Guards
Phase 6 → Role-Based Access Control      (if user_roles has more than one role)
Phase 7 → Session Management             (optional — active sessions + revoke)
```

Skip any phase whose trigger condition is not met.

### Step 3: Write Each Phase Prompt

Write a complete, pasteable Lovable prompt for each selected phase. Include exact Supabase SQL, exact hook names, exact route paths, and specific "Done when" checkboxes.

### Step 4: Output the Plan

Present phases in order with complete prompts and checklists.

## Output Schema

```yaml
plan_title: string          # "[App Name] — Auth Build Plan"
auth_summary: string        # One sentence: which methods, which roles
phases:
  - number: integer
    name: string
    goal: string
    prompt: string           # Complete ready-to-paste Lovable prompt
    done_when: list          # 3-5 specific testable checkboxes
total_phases: integer
```

## Output Format

```
# [App Name] — Auth Build Plan
> [One sentence: auth methods selected, roles, post-login destination]

---

## Phase 1 — Supabase Setup + Profiles Table
**Goal:** Foundation before any auth UI.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] profiles table exists with RLS enabled
- [ ] Signup triggers auto-insert into profiles
- [ ] useAuth() is importable from src/hooks/useAuth

---

## Phase 2 — [Next Phase]
...

---

**Total phases: N**
Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2.
```

## Error Handling

- **No post_login_route given** — ask before generating any prompts; every auth phase depends on this
- **Multiple auth methods** — generate each as its own phase; never combine email + Google in one prompt
- **Existing project** — note which components already exist and skip recreating them

## Examples

### Example 1: Email + Google OAuth

**Input:** App name: Taskly, auth methods: [email, google], post_login_route: /app/dashboard, roles: [user, admin]

**Phases:**
1. Supabase Setup + Profiles Table (with role column)
2. Email + Password Auth (/login, /signup, /forgot-password)
3. Google OAuth (add button to existing login/signup pages)
4. Protected Routes + Guards (ProtectedRoute + AdminRoute components)
5. Role-Based Access Control (admin-only routes, RLS policies for admins)

**Total: 5 phases**

### Example 2: Magic Link Only

**Input:** App name: Brief, auth methods: [magic-link], post_login_route: /app/notes, roles: [user]

**Phases:**
1. Supabase Setup + Profiles Table
2. Magic Link Auth (/login with email input + "Send link" button)
3. Protected Routes + Guards

**Total: 3 phases**
