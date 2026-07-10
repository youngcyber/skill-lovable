# SaaS App Prompts

Patterns for building full-stack SaaS applications with Lovable.

## Overview

SaaS apps in Lovable are built in layers: auth first, then data models, then UI, then business logic. Each layer is a separate prompt. Never combine auth setup and feature building in the same prompt — the result is unpredictable.

---

## Pattern 1: Auth Setup

### Prompt

```
Set up authentication for [APP NAME] using Supabase Auth.

Auth methods to enable:
- Email + password (primary)
- Google OAuth (secondary)
- Magic link (tertiary, for passwordless)

Pages to create:
- /login — email/password form + Google OAuth button + "Send magic link" link
- /signup — same fields as /login plus a "Full name" field
- /forgot-password — email field + submit button that triggers a password reset email
- /reset-password — new password + confirm password fields (reached via email link)

Post-auth behavior:
- After login/signup: redirect to /dashboard
- After logout: redirect to /login
- Unauthenticated access to any /app/* route: redirect to /login with ?next= param
- After password reset: redirect to /login with a success toast

Session handling:
- Persist session across page refreshes using Supabase's built-in session management
- Add a useAuth() hook that exposes: user, session, signIn, signUp, signOut, loading

Design:
- Centered card layout, max-width 400px, white background, subtle shadow
- Logo above the form
- Google button uses the official Google G icon (SVG inline)
- Error messages display inline below the relevant field, not in a toast

Acceptance criteria:
- [ ] Email/password login works and redirects to /dashboard
- [ ] Google OAuth opens the consent screen and returns the user to /dashboard
- [ ] Magic link email arrives and logs the user in
- [ ] Accessing /dashboard when logged out redirects to /login?next=/dashboard
- [ ] Password reset flow completes end-to-end
- [ ] useAuth() hook is importable from @/hooks/useAuth
```

### When to Use

Always build auth before any other feature. This pattern locks in the hook API (`useAuth`) so every subsequent feature prompt can reference it consistently.

---

## Pattern 2: Onboarding Flow

### Prompt

```
Build a multi-step onboarding flow for [APP NAME] that new users see once after signup.

Trigger: runs automatically the first time a user visits /dashboard if their profile.onboarded field is false

Steps:
1. Welcome — "Welcome to [APP NAME], [first name]!" + brief value prop (2 sentences) + "Let's get started" button
2. Create your first [ENTITY] — inline mini-form: name field + optional description. "Skip for now" link available.
3. Invite teammates — email input with "Add another" button (up to 5 invites). "Skip for now" link available.
4. Done — confetti animation + "Go to dashboard" button. Sets profile.onboarded = true via PATCH /api/user/profile.

Navigation:
- Back and Next buttons on each step (except Welcome which only has Next, and Done which only has the final CTA)
- A step indicator (1 of 4, 2 of 4, etc.) at the top
- Back button on step 1 navigates back to /dashboard, not further back

Persistence:
- Store current step in URL params (?step=2) so refreshing doesn't reset progress
- Only mark onboarded=true on final step completion — not on step navigation

Design:
- Full-screen modal overlay, not a separate page
- White card, max-width 560px, centered vertically and horizontally
- Progress bar at the top of the card (indigo fill, animates between steps)
- Confetti on step 4: use canvas-confetti library

Acceptance criteria:
- [ ] Flow only appears if profile.onboarded is false
- [ ] Refreshing on step 2 returns to step 2, not step 1
- [ ] "Skip for now" links advance to the next step without saving
- [ ] Confetti fires exactly once when the user reaches step 4
- [ ] profile.onboarded is set to true only after "Go to dashboard" is clicked
- [ ] Flow never appears again after onboarding is complete
```

### When to Use

Use this after auth is set up and the dashboard shell exists. Onboarding that runs before the dashboard shell is built will require a rebuild.

---

## Pattern 3: Settings Page

### Prompt

```
Build a /settings page for [APP NAME] with tabbed navigation.

Tabs and their content:

Profile tab:
- Avatar upload (circular, 96px, uploads to Supabase Storage bucket "avatars")
- Full name (text input)
- Email (read-only, with a "Change email" link that opens a modal)
- Bio (textarea, max 280 chars, live character counter)
- Save button (disabled until a field changes)

Security tab:
- Change password section: current password, new password, confirm new password
- Active sessions section: list of devices/browsers with "Revoke" button per session
- Two-factor authentication toggle (UI only for now, no backend required)

Billing tab:
- Current plan badge (e.g., "Pro — $49/mo")
- Next billing date
- "Manage billing" button → opens Stripe Customer Portal in a new tab
- Payment method summary (last 4 digits, expiry) — read from GET /api/billing/summary
- Cancel plan link (opens a confirmation modal before proceeding)

Danger Zone tab:
- "Delete account" button — opens a confirmation modal requiring the user to type their email address before deletion is enabled
- Deletion calls DELETE /api/user and signs the user out

Layout:
- Left sidebar tabs on desktop (min-w-48), horizontal scrollable tabs on mobile
- All tabs share the same white card container
- Unsaved changes on Profile tab show a sticky "Unsaved changes" banner at the bottom

Acceptance criteria:
- [ ] Tab navigation works without page reload
- [ ] Avatar upload previews immediately before saving
- [ ] Save button on Profile tab is disabled until a field is dirty
- [ ] "Delete account" button is only enabled after the user types their email correctly
- [ ] Billing tab shows a skeleton loader while /api/billing/summary is fetching
```

### When to Use

Build this after core product features are in place. Settings pages built too early often need to be rebuilt when auth or billing integrations are added later.
