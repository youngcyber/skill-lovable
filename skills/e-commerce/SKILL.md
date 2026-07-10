---
name: lovable-ecommerce-planner
description: "Generate a phase-by-phase Lovable prompt plan for building e-commerce storefronts — product catalog, shopping cart with CartContext, Stripe checkout, and order management."
---

# Lovable E-Commerce Planner

Turn an online store idea into a sequenced build plan. E-commerce apps follow a strict order: cart context before product pages (because "Add to cart" reads from it), and Stripe after the cart total exists.

## When to Use

- User wants to build an online store with Lovable
- User needs a product catalog, shopping cart, or checkout flow
- User asks about Stripe integration in a Lovable project
- User wants to add e-commerce features to an existing Lovable app

## Input Schema

```yaml
store_name: string          # REQUIRED — name of the store
product_type: string        # REQUIRED — what is being sold, e.g. "handmade jewelry"
has_variants: boolean       # OPTIONAL — do products have sizes, colors, etc. — default: false
payment_method: string      # OPTIONAL — "stripe-checkout" | "stripe-elements" — default: stripe-elements
auth_required: boolean      # OPTIONAL — guest checkout or login required — default: false
needs_admin: boolean        # OPTIONAL — seller needs order + product management — default: false
catalog_size: string        # OPTIONAL — "small" (< 50) | "large" (50+) — default: small
```

## Workflow

### Step 1: Clarify Requirements

If `product_type` is missing, ask before generating phases. If `needs_admin` is true but no roles are specified, default to adding an `admin` role to the profiles table.

### Step 2: Select Phases

```
Phase 1 → Stack + Store Layout (navbar, footer, CartContext)
Phase 2 → Product Catalog (/shop with filters)
Phase 3 → Product Detail Page
Phase 4 → Shopping Cart Slide-over
Phase 5 → Checkout Flow (Stripe)
Phase 6 → Order Confirmation + History   (if auth_required: true)
Phase 7 → Admin — Order Management      (if needs_admin: true)
Phase 8 → Admin — Product Management    (if needs_admin: true)
Phase 9 → Empty States + Error Handling
Phase 10 → SEO + Performance
```

**Ordering rules that must never be broken:**
- CartContext (Phase 1) before product detail (Phase 3) — "Add to cart" reads from context
- Cart slide-over (Phase 4) before checkout (Phase 5) — checkout reads from CartContext
- Stripe only after the cart total is computable
- Admin panels always after the customer storefront
- SEO always last

### Step 3: Write Each Phase Prompt

**CartContext rules (Phase 1):**
- Persist to localStorage — cart survives page refresh
- addItem: adding same product increases qty, never creates a duplicate row
- Checkout button disabled when cart is empty

**Product catalog rules (Phase 2):**
- Infinite scroll — load PAGE_SIZE products, fetch next when 300px from bottom
- "SALE" badge only when compare_at_price > price
- Out-of-stock overlay disables "Add to cart"
- Filter state in URL params

**Checkout rules (Phase 5):**
- 3 steps: Shipping → Payment → Confirmation
- Full-screen spinner overlay during Stripe submit — prevents double-submission
- On Stripe error: show error.message inline above "Place order" — do not clear the form
- clearCart() called only on reaching step 3 (Confirmation)
- Order inserted into Supabase orders table on success

### Step 4: Output the Plan

List all phases in order with complete prompts and checklists.

## Output Schema

```yaml
plan_title: string          # "[Store Name] — E-Commerce Build Plan"
store_summary: string       # One sentence: product type, key features
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
# [Store Name] — E-Commerce Build Plan
> [One sentence: product type and key features]

---

## Phase 1 — Stack + Store Layout
**Goal:** CartContext and store shell before any product pages.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] Navbar renders with cart icon (badge hidden when 0)
- [ ] CartContext persists to localStorage across page refreshes
- [ ] Footer renders with all links
- [ ] / route renders with "Shop now" link → /shop

---

## Phase 4 — Shopping Cart Slide-over
**Goal:** Working cart before building checkout.

[COMPLETE LOVABLE PROMPT]

**Done when:**
- [ ] Cart persists across page refreshes
- [ ] Adding same product twice increases qty, not item count
- [ ] Checkout button disabled when cart is empty
- [ ] Escape and overlay click close the cart

---

**Total phases: N**
Paste Phase 1 into Lovable. Verify every checkbox before pasting Phase 2.
```

## Error Handling

- **User wants to build checkout before cart** — refuse and explain that checkout reads from CartContext which must exist first
- **Stripe not connected** — remind user to add Stripe keys in Lovable project settings before writing Phase 5
- **has_variants: true** — add a variant selector section to Phase 3 prompt and a variants table to Phase 1 schema

## Examples

### Example 1: Handmade Jewelry Store

**Input:** Store: Goldenleaf, product: handmade jewelry, variants: true (ring sizes), payment: stripe-elements, auth: false, admin: true

**Phases:**
1. Stack + Store Layout + CartContext
2. Product Catalog (filter by category, price, in-stock)
3. Product Detail (image gallery, size selector, "Add to cart")
4. Shopping Cart Slide-over
5. Checkout (shipping + Stripe Elements + confirmation)
6. Admin — Order Management
7. Admin — Product Management (with image upload)
8. Empty States + Error Handling
9. SEO + Performance

**Total: 9 phases**

### Example 2: Digital Products Store

**Input:** Store: DesignKit, product: Figma UI kits (digital downloads), variants: false, payment: stripe-checkout, auth: true, admin: false

**Phases:**
1. Stack + Store Layout + CartContext
2. Product Catalog (grid, category filter)
3. Product Detail (preview images, license info, "Buy now")
4. Shopping Cart Slide-over
5. Checkout (Stripe Checkout redirect)
6. Order Confirmation + Download links
7. Empty States + Error Handling
8. SEO + Performance

**Total: 8 phases**
