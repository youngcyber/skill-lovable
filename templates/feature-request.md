# Feature Request Template

Use this template when adding a complete, end-to-end feature to an existing Lovable project.

## Template

```
Add [FEATURE NAME] to [APP NAME].

Background:
[1-2 sentences explaining why this feature exists and who asked for it.]

User story:
As a [USER ROLE], I want to [DO SOMETHING] so that [OUTCOME].

Scope — include:
- [Thing 1 to build]
- [Thing 2 to build]
- [Thing 3 to build]

Scope — exclude (build separately later):
- [Thing to leave out]
- [Thing to leave out]

Technical notes:
- [Any API endpoints, auth requirements, or data models to use]
- [Any components that already exist and should be reused]
- [Performance or security constraints]

Acceptance criteria:
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]
- [ ] No regressions in [RELATED EXISTING FEATURE]
```

## When to Use

Use this template when:
- Adding a feature that spans multiple components or routes
- The feature touches authentication, data mutation, or backend logic
- You need to clearly define what is in and out of scope for this iteration

## Example Output

**Filled prompt:**

```
Add email notification preferences to Notify SaaS.

Background:
Users currently receive all notifications by email. Several users have churned citing
notification overload. This feature lets users opt in or out per notification type.

User story:
As a registered user, I want to control which email notifications I receive so that
my inbox only contains alerts I actually care about.

Scope — include:
- A "Notifications" tab inside the existing /settings page
- Toggle switches for each notification type: Weekly digest, New comment, Mention, Security alerts
- Save button that persists choices via PATCH /api/user/notification-preferences
- Success toast on save; inline error if the request fails
- Default all toggles to ON for existing users on first visit

Scope — exclude (build separately later):
- Push notification preferences
- Per-project notification overrides
- Notification frequency controls (e.g., "max 1 email per hour")

Technical notes:
- Reuse the existing <SettingsTabs> component; add a new "Notifications" tab entry
- Preferences are stored in the user_preferences table, column notification_settings (JSONB)
- Security alerts toggle must always default to ON and cannot be turned off
- Use the existing useUserPreferences() hook for reading current settings

Acceptance criteria:
- [ ] The Notifications tab appears in /settings without breaking existing tabs
- [ ] All four toggles render with correct default values from the API
- [ ] Security alerts toggle is visually distinct (lock icon) and disabled
- [ ] Saving shows a success toast within 1 second on 200 response
- [ ] Saving shows an inline error message (not a toast) on 4xx/5xx
- [ ] Unsaved changes show a "You have unsaved changes" banner at the top of the tab
- [ ] No regressions in Profile and Password tabs
```

**What gets built:** A fully functional notification preferences tab wired to a real API, with per-type toggles, a locked security alerts control, unsaved-changes detection, and proper loading/error/success states — all integrated into the existing settings page without touching other tabs.
