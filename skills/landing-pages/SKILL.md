---
name: lovable-landing-page-planner
description: "Generate a phase-by-phase Lovable prompt plan for building a high-converting marketing landing page — hero, features, pricing, testimonials, FAQ, and SEO."
---

# Lovable Landing Page Planner

Turn a product description into a sequenced set of Lovable prompts, one section at a time. Landing pages are built section by section: nav/footer shell first, then hero, then supporting sections, then SEO last.

## When to Use

- User wants to build a marketing or landing page with Lovable
- User needs a public homepage for their product or service
- User asks about hero sections, pricing tables, testimonials, or FAQ sections
- User wants to build a pre-launch waitlist page

## Input Schema

```yaml
product_name: string        # REQUIRED — name of the product
product_description: string # REQUIRED — what it does and who it's for
primary_cta: string         # REQUIRED — the action visitors should take, e.g. "Start free trial"
primary_cta_route: string   # REQUIRED — where the CTA links to, e.g. /signup
sections_needed: list       # OPTIONAL — [hero, features, how-it-works, pricing, testimonials, faq, cta-banner]
                            # default: all sections
has_pricing: boolean        # OPTIONAL — default: false
plan_names: list            # OPTIONAL — required if has_pricing: true
brand_color: string         # OPTIONAL — e.g. "indigo" — default: indigo
```

## Workflow

### Step 1: Clarify Requirements

If `primary_cta` or `primary_cta_route` is missing, ask for them before writing any prompts. Every section links to the primary CTA.

### Step 2: Select Sections

Build sections in this order:

```
Phase 1 → Stack + Base Layout (navbar + footer shell)
Phase 2 → Hero Section
Phase 3 → Features / How It Works
Phase 4 → Social Proof (logo strip + testimonials)
Phase 5 → Pricing Section           (if has_pricing: true)
Phase 6 → FAQ Section               (if requested)
Phase 7 → Final CTA Banner
Phase 8 → SEO + Performance
```

**Ordering rules:**
- Navbar and footer always Phase 1 — every section shares the same wrapper
- Hero always Phase 2 — sets the visual direction before building more
- SEO always last — after all content is in place
- Never combine two sections in one prompt

### Step 3: Write Each Phase Prompt

Each prompt must specify: exact headline and copy, CTA routes, Lucide icon names, Tailwind classes for key elements, responsive breakpoints, and "Done when" checkboxes.

**Hero section rules:**
- One h1 per page — only the hero has h1, all other sections use h2
- Both CTAs must be visible above the fold on 375px mobile
- Always include a product screenshot placeholder (can be swapped later)

**Pricing section rules:**
- Monthly/annual toggle with smooth price crossfade
- Most popular plan: scale-105, indigo background
- Enterprise/custom plan CTA always links to /contact, not /signup

**Testimonial rules:**
- Stars: Lucide Star, fill-yellow-400, not outline
- Section background: gray-50 to contrast with white page sections

### Step 4: Output the Plan

Present all phases in order with complete prompts and checklists.

## Output Schema

```yaml
plan_title: string            # "[Product Name] Landing Page — Build Plan"
product_summary: string       # One sentence confirming product and primary CTA
sections: list                # Which sections will be built
phases:
  - number: integer
    name: string
    goal: string
    prompt: string             # Complete ready-to-paste Lovable prompt
    done_when: list            # 3-5 specific testable checkboxes
total_phases: integer
```

## Output Format

```
# [Product Name] Landing Page — Build Plan
> [One sentence: product, target visitor, primary CTA]

---

## Phase 1 — Stack + Base Layout
**Goal:** Navbar and footer before any section content.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] Navbar renders with all links and CTA button
- [ ] Navbar becomes sticky with backdrop blur after scrolling
- [ ] Footer renders with correct columns and copyright
- [ ] Mobile hamburger menu opens and closes

---

## Phase 2 — Hero Section
**Goal:** Above-the-fold first impression with primary CTA.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] Both CTAs visible above fold on 375px mobile
- [ ] Desktop: text left, screenshot right (lg:grid-cols-2)
- [ ] Section has exactly one h1 tag
- [ ] Social proof line appears below CTAs

---

**Total phases: N**
Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2.
```

## Error Handling

- **No primary CTA route** — ask before generating any prompts; every section's button needs a destination
- **Wants to combine hero + features in one prompt** — refuse; visual direction must be confirmed on hero before adding more
- **No product description** — ask for a one-line description; without it the headline and copy cannot be written

## Examples

### Example 1: SaaS Landing Page with Pricing

**Input:** Product: Compose (AI writing tool), CTA: "Try Compose free" → /signup, has_pricing: true, plans: [Starter, Pro, Business]

**Phases:**
1. Stack + Base Layout (navbar: Features, Pricing, Sign in, "Try free" button)
2. Hero (headline, subheadline, primary + secondary CTA, product screenshot)
3. Features Grid (6 cards: Drafts in seconds, Tone control, etc.)
4. Testimonials (6 quotes from marketing professionals)
5. Pricing Section (monthly/annual toggle, 3 plan cards)
6. FAQ (6 questions about the product)
7. Final CTA Banner ("Start writing faster today")
8. SEO + Performance

**Total: 8 phases**

### Example 2: Pre-Launch Waitlist

**Input:** Product: Nova (AI calendar), CTA: "Join the waitlist" → /waitlist, no pricing, sections: [hero, features, cta-banner]

**Phases:**
1. Stack + Base Layout
2. Hero (countdown or "launching soon" badge, email capture CTA)
3. Features (3-card teaser of upcoming features)
4. Final CTA Banner
5. SEO

**Total: 5 phases**
