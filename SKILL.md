---
name: lovable-vibe-coder
description: "Transform any web app idea into a sequenced, phase-by-phase Lovable prompt plan following vibe coding best practices. Each phase outputs a ready-to-paste prompt that produces one working, testable result."
---

# Lovable Vibe Coder

Turn a web app idea into a complete build plan — a numbered sequence of Lovable prompts, each scoped to one phase, each ending with a testable checkpoint. The user pastes one prompt at a time and verifies it works before moving to the next.

## When to Use

- User says "I want to build [app] with Lovable"
- User has a Lovable project and wants to know what to build next
- User asks "how do I start building X in Lovable?"
- User wants a vague idea broken into actionable steps
- User wants to know the right order to build features

## Input Schema

```yaml
app_name: string          # REQUIRED — name of the app
app_description: string   # REQUIRED — what the app does and who it's for
core_features: list       # REQUIRED — 3-5 main things users do in the app
auth_required: boolean    # OPTIONAL — default: true
paid_plans: boolean       # OPTIONAL — default: false
user_roles: list          # OPTIONAL — e.g. [user, admin] — default: [user]
stack_override: string    # OPTIONAL — override default stack if specified
```

## Workflow

### Step 1: Clarify the Idea

If any required input is missing, ask one focused question to fill the gap. Do not ask more than one question at a time. If enough context exists, skip to Step 2.

### Step 2: Identify the App Type

Classify the app into one or more categories:
- **SaaS** — authenticated users, persistent data, subscriptions → use `skills/saas-apps/SKILL.md`
- **Landing Page** — public marketing site, no auth → use `skills/landing-pages/SKILL.md`
- **Dashboard** — data visualization, admin views → use `skills/dashboards/SKILL.md`
- **E-Commerce** — products, cart, checkout → use `skills/e-commerce/SKILL.md`
- **Forms** — data collection, multi-step wizards → use `skills/forms/SKILL.md`
- **Data Management** — CRUD, kanban, import/export → use `skills/data/SKILL.md`

For apps that span multiple types, combine phases from both in the master order below.

### Step 3: Select Phases

Use this master phase order. Skip phases that do not apply:

```
Phase 1  → Stack Declaration
Phase 2  → Authentication + Profiles    (skip if auth_required = false)
Phase 3  → Database Schema
Phase 4  → App Layout Shell
Phase 5  → Onboarding Flow              (SaaS only)
Phase 6  → Core Feature #1
Phase 7  → Core Feature #2
Phase 8  → Core Feature #3              (repeat as needed)
Phase 9  → Settings Page               (skip if auth_required = false)
Phase 10 → Billing + Stripe            (skip if paid_plans = false)
Phase 11 → Admin Panel                 (skip if single user role)
Phase 12 → Empty States + Error Handling
Phase 13 → Polish + Animations
Phase 14 → SEO + Performance           (public-facing apps only)
```

**Ordering rules (never break these):**
- Stack always Phase 1 — locks in tech before Lovable guesses
- Auth before any feature — no protected-route refactor later
- Schema before UI — data shape must be stable before components read it
- Core features before settings/billing — build the product before wrapping it
- Polish and SEO always last

### Step 4: Write Each Phase Prompt

For every selected phase, write a complete Lovable prompt:

```
[WHAT TO BUILD — one clear sentence]

[CONTEXT — who uses it, what it does, where it lives in the app]

[TECHNICAL SPEC — tables, endpoints, components, stack constraints]

[BEHAVIOR — interactions, states, edge cases]

[DESIGN — layout, responsive breakpoints, component names from shadcn/ui]

Done when:
- [ ] [Specific testable outcome]
- [ ] [Specific testable outcome]
- [ ] [Specific testable outcome]
```

Rules:
- Complete and pasteable — no `[FILL THIS IN]` remaining
- One concern per prompt — never combine two phases
- 3–5 "Done when" checkboxes, each specific and verifiable
- Include loading, empty, and error states for every data-fetching feature
- Reference exact component names from previous phases

### Step 5: Output the Full Plan

Present all phases in order. End with total phase count and the instruction to verify each phase before advancing.

## Output Schema

```yaml
plan_title: string        # "[App Name] — Build Plan"
app_summary: string       # One sentence confirming the understood idea
phases:
  - number: integer       # Phase number
    name: string          # Phase name
    goal: string          # One-sentence goal
    prompt: string        # Complete ready-to-paste Lovable prompt
    done_when: list       # 3-5 specific testable checkboxes
total_phases: integer
closing_note: string      # "Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2."
```

## Output Format

```
# [App Name] — Build Plan
> [One sentence confirming the understood idea]

---

## Phase 1 — Stack Declaration
**Goal:** Lock in the tech stack before Lovable makes choices.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] ...
- [ ] ...
- [ ] ...

---

## Phase 2 — [Name]
**Goal:** ...

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] ...

---

**Total phases: N**
Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2.
```

## Error Handling

- **Idea too vague** — ask one clarifying question, do not generate phases until the app purpose is clear
- **App spans multiple types** — combine relevant phase lists, maintain the master ordering, do not duplicate phases
- **User wants to skip a phase** — warn which later phase depends on it and what will break if skipped

## Examples

### Example 1: SaaS Habit Tracker

**Input:** "I want to build a habit tracker SaaS called Streaks"

**Selected phases:**
1. Stack Declaration
2. Authentication + Profiles
3. Database Schema (habits, completions)
4. App Layout Shell
5. Onboarding (name your first habit)
6. Habit list + daily check-in
7. Streak counter + calendar view
8. Settings Page
9. Empty States + Error Handling
10. Polish + Animations

**Total: 10 phases**

### Example 2: E-Commerce Store

**Input:** "Build an online candle store called Wax & Wick"

**Selected phases:**
1. Stack + Store Layout (nav, footer, CartContext)
2. Product Catalog
3. Product Detail Page
4. Shopping Cart Slide-over
5. Checkout (Stripe)
6. Order Confirmation + History
7. Admin — Order Management
8. SEO + Performance

**Total: 8 phases**
