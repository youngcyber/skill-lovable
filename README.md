# Lovable Prompting Best Practices

A curated collection of prompting patterns, reusable templates, and real-world examples for building production-quality apps with [Lovable](https://lovable.dev).

## Why This Exists

Vague prompts produce vague apps. This repository captures battle-tested patterns that consistently yield clean, deployable results — from single components to full-stack SaaS products.

## Repository Structure

```
├── prompts/          # Category-specific prompt guides
│   ├── landing-pages.md
│   ├── saas-apps.md
│   ├── dashboards.md
│   └── e-commerce.md
├── templates/        # Reusable prompt scaffolds
│   ├── component-prompt.md
│   ├── feature-request.md
│   └── bug-fix.md
└── examples/         # Full real-world prompt → app walkthroughs
    ├── todo-app.md
    ├── blog-platform.md
    └── crm-dashboard.md
```

## Core Principles

### 1. Describe the User, Not the Widget
Tell Lovable who the user is and what they need to accomplish — not the exact UI components to render.

| Weak | Strong |
|------|--------|
| "Add a button" | "Add a primary CTA that lets a first-time visitor start a free trial without signing up" |
| "Make a form" | "Create a contact form for a B2B SaaS: company name, work email, team size dropdown, and a message field. Validate on submit." |

### 2. Anchor Every Request with Context

Always include:
- **Who** is using this feature
- **What** they are trying to accomplish
- **Where** this fits in the app flow (first visit, post-login, checkout, etc.)

### 3. Specify Constraints Upfront

Lovable will make choices for you. Override the important ones early:

```
Use Tailwind CSS for all styling.
Use shadcn/ui for components.
Store data in Supabase.
No external font imports — use the system font stack.
Mobile-first layout.
```

### 4. One Feature Per Prompt

Break large builds into sequential, focused prompts. Each prompt should ship one complete, testable feature.

### 5. Reference What Already Exists

When iterating, name the component or section you want changed:

```
In the <PricingCard> component, change the "Get Started" button to open a modal
instead of navigating to /signup.
```

### 6. End With Acceptance Criteria

Close every prompt with a short checklist Lovable can validate against:

```
When complete:
- [ ] The form submits without a page reload
- [ ] Errors display inline below each field
- [ ] A success toast appears after submission
- [ ] The form resets after success
```

## Quick-Start Templates

- [Component Prompt Template](templates/component-prompt.md)
- [Feature Request Template](templates/feature-request.md)
- [Bug Fix Template](templates/bug-fix.md)

## Real-World Examples

- [To-Do App](examples/todo-app.md) — Full build from blank canvas to deployed app
- [Blog Platform](examples/blog-platform.md) — Multi-author publishing with MDX
- [CRM Dashboard](examples/crm-dashboard.md) — Data-heavy app with filters, charts, and exports

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding new patterns and examples.

## License

MIT
# skill-lovable
