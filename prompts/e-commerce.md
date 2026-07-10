# E-Commerce Prompts

Patterns for building e-commerce storefronts and checkout flows with Lovable.

## Overview

E-commerce builds follow a strict sequence: product catalog → product detail → cart → checkout → confirmation. Never prompt for checkout before the cart exists — Lovable will invent a cart model you'll have to rebuild.

---

## Pattern 1: Product Catalog Grid

### Prompt

```
Build a product catalog page at /shop for [STORE NAME].

Data source: GET /api/products returns an array of products with:
{ id, name, slug, price, compareAtPrice, images: [{ url, alt }], category, inStock, rating, reviewCount }

Layout:
- Filter sidebar on the left (240px, desktop only) + product grid on the right
- Product grid: 3 columns on desktop, 2 on tablet, 1 on mobile
- Infinite scroll (load 12 products at a time, fetch next page when the user scrolls within 300px of the bottom)

Filter sidebar:
- Category multiselect (checkboxes, values from GET /api/products/categories)
- Price range slider ($0 – $500, step $10)
- In stock only toggle
- Rating filter: "4★ & up", "3★ & up", "Any"
- "Clear all filters" link at the top of the sidebar
- On mobile: filters open in a bottom drawer via a "Filters" button above the grid

Each product card:
- Primary image (aspect-ratio 3/4, object-cover, lazy loaded)
- Sale badge ("SALE") if compareAtPrice > price
- Out of stock overlay if inStock is false
- Product name (1 line, truncated)
- Price (bold) + compareAtPrice struck through in muted gray if it exists
- Star rating (Lucide Star icons, half-star not required) + review count
- "Add to cart" button (hidden on mobile, visible on hover on desktop)

Sorting:
- Sort dropdown above the grid (right-aligned): Featured, Price: low to high, Price: high to low, Newest, Best rated
- Changing sort order re-fetches from the API with the new sort param

Acceptance criteria:
- [ ] 12 products load on first render
- [ ] Scrolling to the bottom loads 12 more (no duplicate products)
- [ ] Applying a filter re-fetches and resets to page 1
- [ ] Clearing all filters returns to the unfiltered catalog
- [ ] Out-of-stock products show an overlay and the "Add to cart" button is disabled
- [ ] Sale badge appears only when compareAtPrice is set and higher than price
- [ ] Mobile filter drawer opens and closes correctly
```

### When to Use

Use this for any grid-based product browsing experience. The infinite scroll + sidebar filter pattern scales from 10 to 10,000 products.

---

## Pattern 2: Shopping Cart Sidebar

### Prompt

```
Add a shopping cart as a slide-over sidebar in [STORE NAME].

Trigger: opens when the cart icon in the header is clicked, or when "Add to cart" is clicked anywhere

Cart state:
- Store in React context (CartContext) + localStorage for persistence across sessions
- Cart context exposes: items, addItem(product, quantity), removeItem(id), updateQuantity(id, qty), clearCart, itemCount, subtotal

Sidebar contents:
- Header: "Your Cart" + item count badge + close (X) button
- Empty state: illustration placeholder + "Your cart is empty" + "Continue shopping" link → /shop
- Item list (scrollable if overflow):
  - Each item: product image (48px square), name, variant if applicable, quantity stepper (+/-), unit price, remove (trash) icon
  - Quantity stepper: min 1, max 99; removing last unit removes the item after a 500ms confirmation delay
- Order summary section (sticky at the bottom):
  - Subtotal
  - Shipping: "Calculated at checkout" (gray, italic)
  - "Proceed to checkout" button (full width, indigo)
  - "Continue shopping" text link centered below the button

Animation:
- Sidebar slides in from the right (translate-x-0 ← translate-x-full, 300ms ease-out)
- Background overlay fades in (opacity-0 → opacity-50, same duration)
- Pressing Escape or clicking the overlay closes the sidebar

Acceptance criteria:
- [ ] Cart persists across page refreshes
- [ ] Adding the same product twice increases the quantity, not the item count
- [ ] Quantity stepper cannot go below 1 (trash icon removes the item instead)
- [ ] Subtotal updates immediately on any quantity change
- [ ] Empty state renders when all items are removed
- [ ] Sidebar closes on Escape key and on overlay click
- [ ] "Proceed to checkout" is disabled if the cart is empty
```

### When to Use

Build this before the checkout page. The cart context defined here (`CartContext`) is what the checkout page reads from to populate the order summary.

---

## Pattern 3: Checkout Flow

### Prompt

```
Build a multi-step checkout page at /checkout for [STORE NAME].

Prerequisite: CartContext is available from @/context/CartContext.

Steps:
1. Contact & Shipping
   - Email address
   - Full name
   - Address line 1, Address line 2 (optional)
   - City, State (dropdown), ZIP, Country (dropdown, default US)
   - Shipping method radio: Standard (Free, 5-7 days), Expedited ($9.99, 2-3 days), Overnight ($24.99, next day)

2. Payment
   - Stripe Elements (Card element — number, expiry, CVC in one integrated block)
   - Billing address: "Same as shipping" checkbox (default checked); if unchecked, show address fields
   - "Place order" button

3. Confirmation
   - Order number (returned from POST /api/orders)
   - Summary of items ordered
   - Estimated delivery date based on selected shipping method
   - "Continue shopping" button → /shop
   - Clears the cart on mount

Layout:
- Two-column on desktop: form on the left (60%), order summary on the right (40%, sticky)
- Single column on mobile: order summary collapses to an accordion above the form
- Step indicator at the top: "1. Shipping → 2. Payment → 3. Confirmation"
- Breadcrumb-style: completed steps are clickable (can go back), future steps are not

Validation:
- Validate on "Continue" / "Place order" click, not on every keystroke
- Show inline errors below each invalid field
- ZIP code: US format only (5 digits or 5+4)
- Email: standard format validation

On submit:
- POST /api/orders with { cartItems, shippingAddress, shippingMethod, stripePaymentMethodId }
- Show a full-page spinner overlay during submission
- On 4xx: display the API error message above the "Place order" button
- On 5xx: display "Something went wrong. Please try again." — do not clear the form

Acceptance criteria:
- [ ] Completing step 1 advances to step 2; clicking the step 1 breadcrumb returns to step 1 with data intact
- [ ] "Same as shipping" checkbox pre-fills billing fields and disables them
- [ ] Stripe card element is fully functional in test mode
- [ ] Submitting with an invalid card shows a Stripe error message inline
- [ ] Confirmation page renders with the correct order number and clears the cart
- [ ] Full-page spinner prevents double-submission
```

### When to Use

Build this only after the cart sidebar is complete and Stripe is connected in the Lovable project settings. Checkout without a working CartContext will require manual wiring.
