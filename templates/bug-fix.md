# Bug Fix Template

Use this template when asking Lovable to fix a specific, reproducible bug.

## Template

```
Fix: [SHORT BUG DESCRIPTION] in [COMPONENT OR PAGE NAME]

Environment:
- Where it happens: [URL path or component name]
- Who sees it: [e.g., "all users" or "only users on mobile" or "only after login"]
- How often: [e.g., "every time" or "only when X condition is true"]

Steps to reproduce:
1. [Step 1]
2. [Step 2]
3. [Step 3]

Current behavior:
[Exactly what happens — be specific. Include error messages verbatim if applicable.]

Expected behavior:
[Exactly what should happen instead.]

Suspected cause (optional):
[If you have a hunch — name the file, function, or logic you think is wrong.]

Do not change:
- [Component or behavior to leave untouched]
- [Component or behavior to leave untouched]

Acceptance criteria:
- [ ] The bug no longer reproduces following the steps above
- [ ] [Any related edge case to verify]
- [ ] No regressions in [RELATED FEATURE]
```

## When to Use

Use this template when:
- You have a specific, reproducible bug
- You want to prevent Lovable from refactoring unrelated code while fixing it
- The fix involves a non-obvious root cause you want to communicate

## Example Output

**Filled prompt:**

```
Fix: dropdown menu closes immediately after opening on iOS Safari in <UserNav>

Environment:
- Where it happens: the avatar dropdown in the top-right of the main nav (UserNav component)
- Who sees it: all users on iOS Safari (iPhone and iPad)
- How often: every time the avatar is tapped

Steps to reproduce:
1. Open the app on an iPhone using Safari
2. Log in so the nav bar appears
3. Tap the avatar icon in the top-right corner

Current behavior:
The dropdown opens for a split second and then immediately closes. No menu item
can be selected. The bug does not occur on Chrome for iOS or any desktop browser.

Expected behavior:
The dropdown opens and stays open until the user taps a menu item or taps outside
the dropdown.

Suspected cause:
iOS Safari fires a blur event on the trigger button before the click event on menu
items registers, causing the onOpenChange handler to close the menu prematurely.
The issue is likely in the onOpenChange or onBlur logic inside UserNav.tsx.

Do not change:
- The visual design or positioning of the dropdown
- Any behavior on non-iOS browsers
- The list of menu items or their navigation targets

Acceptance criteria:
- [ ] Tapping the avatar on iOS Safari opens the dropdown and it stays open
- [ ] Tapping a menu item on iOS Safari navigates correctly
- [ ] Tapping outside the dropdown on iOS Safari closes it
- [ ] Dropdown behavior on Chrome (desktop and mobile) is unchanged
- [ ] Dropdown behavior on Firefox (desktop) is unchanged
```

**What gets built:** A targeted fix for the iOS Safari event-ordering bug in the nav dropdown, with no changes to visual design, menu items, or behavior on other platforms.
