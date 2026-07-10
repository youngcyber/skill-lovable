---
name: lovable-forms-planner
description: "Generate a phase-by-phase Lovable prompt plan for building forms — contact forms, multi-step wizards, file uploads, and validated data collection using react-hook-form and Zod."
---

# Lovable Forms Planner

Turn a form requirement into a sequenced build plan using react-hook-form and Zod. Forms are built in layers: schema first, then UI, then validation, then complexity (multi-step, file upload, conditionals).

## When to Use

- User needs a contact form, signup form, or any data-collection UI
- User wants a multi-step wizard or onboarding flow
- User needs file upload in a form
- User asks about form validation in a Lovable project
- User wants conditional fields that show/hide based on other answers

## Input Schema

```yaml
form_name: string           # REQUIRED — name of the form, e.g. "Contact Form"
form_purpose: string        # REQUIRED — what data is collected and where it goes
fields: list                # REQUIRED — list of fields with type and required status
submission_target: string   # REQUIRED — "supabase" | "api-endpoint" | "email"
is_multistep: boolean       # OPTIONAL — default: false
steps: list                 # OPTIONAL — required if is_multistep: true
has_file_upload: boolean    # OPTIONAL — default: false
file_types: list            # OPTIONAL — required if has_file_upload: true
has_autosave: boolean       # OPTIONAL — default: false
has_conditional_fields: boolean  # OPTIONAL — default: false
```

## Workflow

### Step 1: Clarify Requirements

If `fields` or `submission_target` is missing, ask for them before generating phases. Without the field list, the Zod schema cannot be written.

### Step 2: Select Phases

```
Phase 1 → Data Schema + Submission Target
Phase 2 → Form UI (fields + layout, no validation)
Phase 3 → Validation (react-hook-form + Zod)
Phase 4 → Multi-step Navigation    (if is_multistep: true)
Phase 5 → File Upload              (if has_file_upload: true)
Phase 6 → Submission Handling (loading / success / error)
Phase 7 → Conditional Fields       (if has_conditional_fields: true)
Phase 8 → Autosave / Draft         (if has_autosave: true)
```

**Ordering rules:**
- Schema always Phase 1 — Zod schema drives field types and validation
- Basic UI (Phase 2) before validation (Phase 3) — see the form before adding rules
- Multi-step navigation only after single-step form is fully working
- File upload is always its own phase — never combined with other fields
- Submission handling (loading/success/error) always before conditional fields or autosave
- Conditional fields and autosave always last — they enhance a working form

### Step 3: Write Each Phase Prompt

**Schema rules (Phase 1):**
- Define the Zod schema at `src/lib/schemas/[form-name].ts` before building any UI
- If submitting to Supabase: include the CREATE TABLE statement with RLS policy
- If submitting to email: include the API route definition

**Validation rules (Phase 3):**
- Use `zodResolver` from `@hookform/resolvers/zod`
- Validate on submit first — do not show errors on every keystroke
- After first submit attempt, switch to `mode: 'onChange'` so fixed fields clear errors immediately
- Error messages appear inline below the field in `text-red-500 text-sm`
- Field with error: `border-red-500` on the input

**Multi-step rules (Phase 4):**
- Validate only the current step's fields on "Continue" — not the whole form
- Store current step in URL param `?step=N` — refresh-safe
- Previous step data must be pre-filled when navigating back
- `shouldUnregister: false` in useForm to preserve data across steps

**File upload rules (Phase 5):**
- Upload happens on form submit, not on file select
- Show progress bar per file during upload
- Wrong file type or size: inline error below the drop zone — do not add file to the queue

**Submission handling rules (Phase 6):**
- Submit button: disabled + Loader2 spinner during submission
- Never clear the form on error
- Success: either replace form with success UI (Option A) or toast + reset (Option B — specify which)

### Step 4: Output the Plan

List all phases in order with complete prompts and checklists.

## Output Schema

```yaml
plan_title: string          # "[Form Name] — Build Plan"
form_summary: string        # One sentence: what it collects and where it goes
phases:
  - number: integer
    name: string
    goal: string
    prompt: string           # Complete ready-to-paste Lovable prompt
    done_when: list
total_phases: integer
```

## Output Format

```
# [Form Name] — Build Plan
> [One sentence: fields collected, submission target]

---

## Phase 1 — Data Schema + Submission Target
**Goal:** Define where data goes before building any UI.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] Zod schema is defined and importable from src/lib/schemas/[form-name]
- [ ] Supabase table / API endpoint accepts a test submission
- [ ] No UI built yet

---

## Phase 3 — Validation with Zod
**Goal:** Field-level validation before submission.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] Submitting empty required fields shows inline errors
- [ ] Error messages appear below the field (not as toasts)
- [ ] Fixing an errored field removes its error immediately
- [ ] Valid data passes through to handleSubmit

---

**Total phases: N**
Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2.
```

## Error Handling

- **Fields list missing** — ask for the full list of fields before writing any phase
- **Submission target unclear** — ask "Does this go to a Supabase table, an API endpoint, or an email?" before Phase 1
- **User wants to add validation in Phase 2** — separate into Phase 2 (UI only) then Phase 3 (validation); never combine

## Examples

### Example 1: Contact Form

**Input:** Form: Contact Form, purpose: send inquiry emails, fields: [name (text, required), email (email, required), subject (text, required), message (textarea, required)], target: email via Resend, single-step, no file upload

**Phases:**
1. Schema + Email endpoint (Zod schema + /api/contact route using Resend)
2. Form UI (4 fields, single column, max-w-lg)
3. Validation (required fields, email format)
4. Submission Handling (spinner, success message replaces form, inline error on failure)

**Total: 4 phases**

### Example 2: Multi-step Job Application

**Input:** Form: Job Application, fields: [name, email, role (select), cover letter (textarea), resume (PDF upload), linkedin (url, optional)], target: Supabase applications table, multi-step: [Personal Info, Cover Letter, Resume Upload, Review], file: PDF max 5MB

**Phases:**
1. Schema + Supabase table (applications table + RLS)
2. Form UI (all fields laid out, no validation, no steps)
3. Validation (required fields, URL format for LinkedIn, PDF type for resume)
4. Multi-step Navigation (4 steps, URL param, current-step validation only)
5. File Upload (PDF drop zone, 5MB limit, Supabase Storage, progress bar)
6. Submission Handling (spinner, success page with application ID, error inline)
7. Conditional Fields (show LinkedIn field only when role = "Engineering")

**Total: 7 phases**
