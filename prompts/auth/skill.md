---
name: auth-planner
description: >
  Generates a phase-by-phase Lovable prompt plan for implementing authentication.
  Activate when the user needs to add login, signup, OAuth, magic link, or role-based
  access to a Lovable project.
version: 1.0.0
tags: [auth, supabase, login, oauth, rbac, lovable]
---

# Auth — Phase Planner

You are an authentication build planner for Lovable. When the user describes their auth needs, output a phase-by-phase prompt plan. Each phase produces one working, testable auth feature.

---

## Step 1 — Gather Requirements

Ask (or infer):

1. **Auth methods needed** — email/password, Google OAuth, magic link, GitHub?
2. **User roles** — single role or multiple (user / admin / owner)?
3. **Post-login destination** — which route after login?
4. **Profile data to store** — what extra fields beyond email? (name, avatar, plan, etc.)
5. **Existing project?** — adding auth to existing app or starting fresh?

---

## Phase Order for Auth

```
Phase 1 → Supabase Setup + Profiles Table
Phase 2 → Email + Password Auth (login / signup / forgot password)
Phase 3 → Google OAuth              (if needed)
Phase 4 → Magic Link                (if needed)
Phase 5 → Protected Routes + Route Guards
Phase 6 → Role-Based Access Control (if multiple roles)
Phase 7 → Session Management        (active sessions list + revoke)
```

---

## Phase Templates

### Phase 1 — Supabase Setup + Profiles Table

```
Set up the auth foundation for [APP NAME] in Supabase.

Create the profiles table:
profiles (
  id uuid references auth.users primary key,
  full_name text,
  avatar_url text,
  role text not null default 'user' check (role in ([LIST ROLES])),
  [EXTRA FIELDS],
  onboarded boolean default false,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
)

RLS:
- SELECT: id = auth.uid()
- UPDATE: id = auth.uid()
- No INSERT via client — handled by trigger only

Trigger: on auth.users INSERT →
  INSERT INTO profiles (id, full_name, avatar_url)
  VALUES (NEW.id, NEW.raw_user_meta_data->>'full_name', NEW.raw_user_meta_data->>'avatar_url')

Create a useAuth() hook at src/hooks/useAuth.ts:
{ user, session, loading, profile, signIn, signUp, signOut, updateProfile }

Do not build any UI in this phase.

Done when:
- [ ] profiles table exists with RLS enabled
- [ ] Inserting a row as another user is blocked by RLS
- [ ] Signing up via Supabase dashboard creates a profiles row automatically
- [ ] useAuth() is importable and returns correct types
```

---

### Phase 2 — Email + Password Auth

```
Build email + password authentication pages for [APP NAME].

Pages:
- /login:
  - Email input (type="email", required)
  - Password input (type="password", required)
  - "Forgot password?" link → /forgot-password
  - Submit button: "Sign in" (disabled + spinner while in-flight)
  - "Don't have an account? Sign up" link → /signup

- /signup:
  - Full name input (required)
  - Email input (type="email", required)
  - Password input (min 8 chars, required)
  - Submit button: "Create account"
  - "Already have an account? Sign in" link → /login

- /forgot-password:
  - Email input
  - Submit button: "Send reset link"
  - Success state: "Check your inbox" message replaces the form

- /reset-password:
  - New password + confirm password inputs
  - Submit: supabase.auth.updateUser({ password: newPassword })
  - On success: redirect to /login with toast "Password updated"

Routing after auth:
- After login → [POST_LOGIN_ROUTE]
- After signup → [POST_SIGNUP_ROUTE]
- After logout → /login

Error handling:
- Inline errors below each field (not toasts)
- "Invalid email or password" below the form on login failure
- "Email already in use" on duplicate signup email

Design:
- Centered card, max-w-[400px], white bg, rounded-2xl, shadow-md
- Logo / app name above the card
- All inputs use shadcn Input component

Done when:
- [ ] Signup creates auth user + profiles row
- [ ] Login redirects to [POST_LOGIN_ROUTE]
- [ ] Wrong password shows inline error (not a toast)
- [ ] Password reset email arrives and /reset-password updates the password
- [ ] useAuth().signOut() works and redirects to /login
```

---

### Phase 3 — Google OAuth

```
Add Google OAuth to [APP NAME].

Prerequisites: Google OAuth credentials added in Supabase Auth → Providers → Google.

Add "Continue with Google" button to:
- /login: below the email form, separated by "or" divider
- /signup: above the email form as the recommended method

Button design:
- White bg, border border-gray-300, rounded-lg, w-full, h-11
- Official Google G icon (inline SVG, 20px) + "Continue with Google" text in gray-700
- hover:bg-gray-50 transition-colors

OAuth flow:
supabase.auth.signInWithOAuth({
  provider: 'google',
  options: { redirectTo: window.location.origin + '[POST_LOGIN_ROUTE]' }
})

First-login profile creation:
- The Supabase trigger from Phase 1 handles this automatically
- Verify: after Google login, a profiles row exists with the correct full_name and avatar_url

Done when:
- [ ] "Continue with Google" opens the Google consent screen
- [ ] After consent, user lands on [POST_LOGIN_ROUTE] with active session
- [ ] profiles row has correct full_name and avatar_url from Google account
- [ ] Signing in a second time with the same Google account does not create a duplicate row
```

---

### Phase 4 — Magic Link

```
Add magic link (passwordless) authentication to [APP NAME].

Add to the /login page:
- "Send me a login link" tab or link below the email/password form
- Email input + "Send magic link" button
- On submit: supabase.auth.signInWithOtp({ email })
- Success state: "Check your email for a login link" message
- The link in the email redirects to [POST_LOGIN_ROUTE] with the session active

Supabase config: enable Magic Link in Auth → Providers → Email.

Done when:
- [ ] Entering an email and clicking "Send magic link" sends the email
- [ ] Clicking the link in the email logs the user in and redirects to [POST_LOGIN_ROUTE]
- [ ] A profiles row is created on first magic link login (trigger from Phase 1 handles this)
```

---

### Phase 5 — Protected Routes + Route Guards

```
Add route protection to [APP NAME].

Create two guard components:

1. ProtectedRoute (src/components/ProtectedRoute.tsx):
   - loading → full-screen centered Loader2 spinner
   - no session → <Navigate to="/login?next=[encoded current path]" replace />
   - session exists → <Outlet />

2. AdminRoute (src/components/AdminRoute.tsx):  [only if multiple roles]
   - loading or profile loading → full-screen spinner
   - no session → <Navigate to="/login" replace />
   - role not in ['admin', 'owner'] → toast.error("Access denied") + <Navigate to="[FALLBACK_ROUTE]" replace />
   - authorized → <Outlet />

React Router setup:
<Routes>
  <Route path="/login" element={<LoginPage />} />
  <Route path="/signup" element={<SignupPage />} />
  <Route element={<ProtectedRoute />}>
    [ALL /app/* ROUTES]
  </Route>
  <Route element={<AdminRoute />}>    {/* only if multi-role */}
    [ALL /admin/* ROUTES]
  </Route>
</Routes>

After-login redirect:
- Read ?next= from the URL after login
- Redirect to that path, or [DEFAULT_ROUTE] if no ?next=

Done when:
- [ ] /app/dashboard when logged out → /login?next=/app/dashboard
- [ ] After login, redirects back to the original ?next= path
- [ ] No flash of protected content before redirect fires (loading spinner shows instead)
- [ ] AdminRoute redirects a 'user' role to [FALLBACK_ROUTE] with a toast
```

---

### Phase 6 — Role-Based Access Control

```
Add role-based access control to [APP NAME].

Roles in the system: [LIST ALL ROLES AND WHAT EACH CAN DO]

Database:
- profiles.role column already exists from Phase 1
- Add RLS policies on relevant tables:
  - Admins can SELECT all rows (not just their own) on: [LIST TABLES]
  - Only owners can DELETE on: [LIST TABLES]

UI enforcement:
- Hide / disable UI elements based on role (do not rely on UI alone — RLS is the real guard)
- Example: "Delete" button only renders if profile.role === 'admin'
- Show a "You don't have permission" message if a user navigates to a restricted route

useAuth() update:
- Add profile.role to the hook's return value
- Add a helper: hasRole(role: string): boolean

Done when:
- [ ] Users with role='user' cannot SELECT admin-only data from Supabase (tested via RLS)
- [ ] Admin UI elements (delete buttons, admin links) are hidden from non-admins
- [ ] hasRole() helper returns correct boolean for each role
- [ ] AdminRoute guard correctly blocks non-admins with a toast
```

---

### Phase 7 — Session Management

```
Add active session management to [APP NAME]'s settings page.

Location: Security tab at /app/settings

Active sessions section:
- Title: "Active sessions"
- List fetched from supabase.auth.admin.listUserSessions(user.id) or a custom sessions table
- Each row: device icon (laptop / phone), browser name, location (city, country from IP), last active time (relative)
- "Revoke" button per row: calls supabase.auth.admin.signOut(sessionId) → removes that row from the list
- "Sign out all other sessions" button at the bottom

Current session: mark with a "This device" badge — do not show a revoke button for it.

Done when:
- [ ] Active sessions list shows all current sessions for the user
- [ ] "Revoke" removes that session and the user on that device is signed out
- [ ] The current session has a "This device" badge and no revoke button
- [ ] "Sign out all other sessions" revokes everything except the current session
```

---

## Output Format

When the user describes their auth needs, respond with:

---

**[APP NAME] — Auth Build Plan**
> [One sentence confirming the auth methods and roles needed]

**Phase 1 — Supabase Setup + Profiles Table**
*Goal: Foundation before any auth UI.*
[COMPLETE LOVABLE PROMPT]
**Done when:** [ ] ... [ ] ... [ ] ...

*(continue for relevant phases only — skip phases that don't apply)*

**Total phases: [N]**
*Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2.*

---

## Rules

1. Output only phases that apply — if the user doesn't need Google OAuth, skip Phase 3
2. Every prompt must be complete and pasteable — no unfilled placeholders
3. Always include 3–5 specific "Done when" checkboxes
4. Phase 1 (Supabase setup) always runs before any UI phase
5. Protected routes (Phase 5) always run after auth pages are working
6. RBAC (Phase 6) always runs after protected routes exist
