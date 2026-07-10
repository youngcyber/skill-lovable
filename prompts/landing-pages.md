# Landing Page Prompts

Patterns for building high-converting landing pages with Lovable.

## Overview

Landing pages built with Lovable work best when each section is prompted separately, then connected in a final composition pass. Start with the hero, then add social proof, features, pricing, and CTA sections one at a time.

---

## Pattern 1: SaaS Hero Section

### Prompt

```
Build a hero section for [APP NAME], a [ONE-LINE DESCRIPTION].

Target visitor: [e.g., "a startup founder looking to automate their invoicing"]
Primary goal: get the visitor to click "Start free trial" without signing up first

Content:
- Headline: "[YOUR HEADLINE]"
- Subheadline: "[YOUR SUBHEADLINE — max 2 lines]"
- Primary CTA: "Start free trial" → links to /signup
- Secondary CTA: "See a demo" → opens a YouTube embed in a modal
- Social proof line below CTAs: "Trusted by 2,400+ teams at companies like Stripe, Linear, and Notion"
- Hero image: a product screenshot mockup (use a placeholder for now, I'll swap it later)

Design:
- Stack: React + Tailwind CSS + shadcn/ui
- Layout: centered text with the hero image below on mobile, side-by-side on desktop (text left, image right)
- Background: white with a subtle radial gradient behind the headline (indigo → transparent)
- Headline font size: 4xl on mobile, 6xl on desktop, font-bold
- No stock photos, no illustrations — only the product mockup

Acceptance criteria:
- [ ] Both CTAs are visible above the fold on a 375px mobile screen
- [ ] The demo modal opens and closes without page reload
- [ ] The hero image placeholder has correct aspect ratio (16:9)
- [ ] Section is fully accessible: heading hierarchy starts at h1, CTAs have aria-labels
```

### When to Use

Use this when starting a new SaaS marketing site from scratch. The side-by-side layout with a product screenshot is the highest-converting pattern for B2B SaaS.

---

## Pattern 2: Feature Grid Section

### Prompt

```
Add a features section below the hero on the [APP NAME] landing page.

Audience: visitors who scrolled past the hero and want to understand what the product actually does

Layout: 3-column grid on desktop, 1-column stack on mobile
Number of features: 6

Each feature card:
- Icon (use Lucide React icons, not custom SVGs)
- Title (bold, ~3 words)
- Description (2 sentences max)
- No CTA per card

Features to include:
1. Icon: Zap | Title: "Instant setup" | Description: "Connect your tools in under 5 minutes. No engineering required."
2. Icon: Shield | Title: "SOC 2 compliant" | Description: "Your data is encrypted at rest and in transit. We're audited annually."
3. Icon: BarChart2 | Title: "Real-time analytics" | Description: "See what's happening in your pipeline as it happens. No 24-hour delays."
4. Icon: Users | Title: "Team collaboration" | Description: "Invite unlimited teammates. Roles and permissions included."
5. Icon: RefreshCw | Title: "Automated syncs" | Description: "Set it once. Data syncs on your schedule — hourly, daily, or on trigger."
6. Icon: Headphones | Title: "24/7 support" | Description: "Chat with a human whenever you need one. No bots, no ticket queues."

Design:
- Card background: white with 1px border, rounded-xl, subtle shadow on hover
- Icon: indigo-600, size 24px, inside a rounded indigo-50 badge
- Section padding: py-24 on desktop, py-16 on mobile
- Section heading above the grid: "Everything you need to ship faster"

Acceptance criteria:
- [ ] All 6 cards render with correct icon, title, and description
- [ ] Grid collapses to single column below 640px
- [ ] Hover state on each card is smooth (transition-all duration-200)
- [ ] Section heading is an h2
```

### When to Use

Use after the hero section. Works for any product type. Six features is the sweet spot — enough to convince without overwhelming.

---

## Pattern 3: Pricing Section with Toggle

### Prompt

```
Build a pricing section for [APP NAME] with a monthly/annual billing toggle.

Plans:
- Starter: $19/mo (or $15/mo billed annually) — for individuals and freelancers
- Pro: $49/mo (or $39/mo billed annually) — for small teams, HIGHLIGHTED as most popular
- Enterprise: Custom pricing — for large orgs, CTA is "Contact sales" instead of "Get started"

Each plan card includes:
- Plan name and price (update dynamically based on toggle)
- "Save 20%" badge next to the annual price (only visible when annual is selected)
- 1-sentence description
- List of 5 features (checkmarks, Lucide CheckCircle icon, green)
- CTA button

Features per plan:
Starter: Up to 3 projects, 5GB storage, Email support, Basic analytics, API access
Pro: Unlimited projects, 50GB storage, Priority support, Advanced analytics, API access + webhooks
Enterprise: Unlimited everything, Dedicated storage, 24/7 phone support, Custom analytics, SSO + audit logs

Toggle behavior:
- Default to monthly
- Switching to annual updates all prices simultaneously with a smooth number transition
- The toggle pill slides with a CSS transition (no jump)

Design:
- Pro card: indigo background, white text, scaled up slightly (scale-105) to draw attention
- Starter and Enterprise cards: white background, dark text
- Section heading: "Simple, transparent pricing"
- Subheading: "Start free. No credit card required."

Acceptance criteria:
- [ ] Toggle switches between monthly and annual prices correctly
- [ ] "Save 20%" badge appears only when annual is active
- [ ] Pro card is visually distinct and slightly larger than the others
- [ ] Enterprise CTA links to /contact, not /signup
- [ ] All prices update without a page reload
- [ ] Section is responsive: stacked on mobile, 3-column on desktop
```

### When to Use

Use this for any SaaS pricing section. The annual/monthly toggle with a visible savings badge increases annual plan conversion by reducing sticker shock.
