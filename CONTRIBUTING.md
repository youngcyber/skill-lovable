# Contributing

Contributions are welcome. This guide explains how to add new patterns, templates, or examples.

## What to Contribute

- **Prompt patterns** — A repeatable prompt structure that produces reliable results for a specific use case
- **Templates** — Scaffold prompts with clearly marked fill-in sections
- **Examples** — Real prompts you used to build a real feature, with the outcome described

## What Not to Contribute

- Prompts that only work for one specific project
- Screenshots of generated UIs without the prompts that produced them
- Partial examples with TODO placeholders

## How to Add a Pattern

1. Fork this repository
2. Add your file in the appropriate directory (`prompts/`, `templates/`, or `examples/`)
3. Follow the format of an existing file in that directory
4. Update the `README.md` index table if you add a new file
5. Open a pull request with a title like: `Add: e-commerce checkout prompt pattern`

## Format Requirements

Every file must include:

- A `## Overview` section explaining what the pattern is for
- At least one complete, copy-pasteable prompt
- A `## When to Use` section
- A `## Example Output` description (not a screenshot — describe what gets built)

## Commit Style

Use clear, imperative commit messages:

```
Add landing page prompt for SaaS hero sections
Fix typo in CRM dashboard example
Update component-prompt template with validation criteria
```
