# Real-World Example: Blog Platform

A complete walkthrough of building a multi-author blog platform with Lovable. Features include a public-facing reader experience, a private author dashboard, MDX-rendered posts, and an admin panel for managing users.

**Total prompts used:** 6

---

## Prompt 1 — Public blog index

```
Build the public homepage for a blog platform called "Inkwell" at /.

This is a marketing + content page, not an app dashboard.

Hero:
- Headline: "Ideas worth reading."
- Subheadline: "Inkwell is a publishing platform for serious writers. Long-form, ad-free, by humans."
- CTA: "Start writing" → /signup, "Browse posts" → scrolls to the post grid below
- No hero image — use a typographic layout with a large serif font (use Google Fonts: Playfair Display for headings, Inter for body)

Featured post (above the fold, below the hero):
- Large card: cover image (16:9), category tag, title, excerpt (2 lines), author name + avatar, read time, publish date
- Data: GET /api/posts?featured=true&limit=1

Post grid (below featured):
- 3 columns on desktop, 2 on tablet, 1 on mobile
- Each card: cover image (3:2), category tag, title (2 lines max, truncated), author name + avatar (small), read time
- Data: GET /api/posts?page=1&limit=9 (exclude the featured post)
- "Load more" button at the bottom (not infinite scroll — explicit button)

Header:
- Logo ("Inkwell" in Playfair Display) left-aligned
- Nav: Browse, About, Sign in, "Start writing" (button)
- Sticky, white background with border-bottom on scroll

Footer:
- Logo + tagline
- Links: Browse, About, Privacy, Terms
- "© 2025 Inkwell. All rights reserved."

Acceptance criteria:
- [ ] Featured post loads from /api/posts?featured=true
- [ ] Post grid loads 9 posts on mount
- [ ] "Load more" fetches the next 9 posts and appends them (no duplicate posts)
- [ ] Header becomes sticky with border after scrolling 64px
- [ ] All fonts load correctly (Playfair Display for headings, Inter for body)
```

---

## Prompt 2 — Post detail page

```
Build the post detail page at /posts/:slug for Inkwell.

Data: GET /api/posts/:slug returns:
{
  id, title, slug, excerpt, coverImage, content (MDX string),
  publishedAt, readTime, category,
  author: { name, avatarUrl, bio, postCount }
}

Layout:
- Max content width: 680px, centered
- Full-width cover image above the title (aspect 21:9, object-cover)
- Title: Playfair Display, 40px desktop / 28px mobile, font-bold
- Byline below title: author avatar + name + "in [category]" + publish date + read time

MDX rendering:
- Render the content field as MDX using next-mdx-remote or mdx-bundler (whichever is already in the project)
- Custom MDX components:
  - h2: Playfair Display, 28px, with a top border-t-2 border-indigo-200 for visual separation
  - blockquote: left border (4px indigo), italic, indigo-100 background, rounded-r-lg
  - code (inline): monospace, indigo-50 background, indigo-700 text, rounded, px-1
  - pre (code block): dark background (#1e1e2e), syntax highlighted via Prism, copy button in top-right corner

Author card below the post:
- Avatar (64px circle), name (bold), bio (2 lines max), "X posts published"
- "Follow" button (UI only — no backend required yet)

Related posts:
- 3 posts from the same category, from GET /api/posts?category=:category&limit=4 (exclude current post)
- Small horizontal cards: thumbnail left (80px square), title right, read time below title

Acceptance criteria:
- [ ] Post content renders from the MDX string with correct custom component styles
- [ ] Code blocks have a copy button that copies to clipboard and shows "Copied!" for 2 seconds
- [ ] Author card appears below the post with correct data
- [ ] Related posts grid shows 3 posts from the same category
- [ ] Cover image spans the full page width (not capped at 680px)
```

---

## Prompt 3 — Author write + publish flow

```
Build the post editor at /write for authenticated authors in Inkwell.

Auth: only accessible to authenticated users. Redirect to /login if unauthenticated.

Editor:
- Title field at the top: large, plain text input, Playfair Display, 36px, placeholder "Post title"
- Body: a rich text editor using TipTap
  - Toolbar: Bold, Italic, Underline, Strikethrough | H2, H3 | Bullet list, Numbered list | Blockquote | Code block | Link | Image upload
  - Keyboard shortcuts: Cmd+B (bold), Cmd+I (italic), Cmd+K (link)
- Cover image upload: drag-and-drop zone above the title (uploads to Supabase Storage bucket "covers")

Right sidebar (publish panel):
- Status badge: "Draft" or "Published"
- Publish button (POST /api/posts): disabled until title and body are non-empty
- "Save draft" button: saves without publishing, shows "Saved" indicator
- Excerpt field (textarea, 160 char max, with live counter)
- Category selector (dropdown, populated from GET /api/categories)
- Estimated read time (auto-calculated from word count, read-only)

Auto-save:
- Save draft to Supabase every 30 seconds if there are unsaved changes
- Show "Saving…" while the request is in flight, "Saved X minutes ago" after

Editing existing posts: /write/:postId loads the existing post data into the editor.

Acceptance criteria:
- [ ] TipTap editor renders and all toolbar buttons work
- [ ] Cover image uploads to Supabase Storage and previews above the title
- [ ] Auto-save fires every 30 seconds with unsaved changes
- [ ] Publish button is disabled until title and body fields are non-empty
- [ ] Navigating to /write/:postId loads the existing post for editing
- [ ] Estimated read time updates as the user types
```

---

## Prompt 4 — Author dashboard

```
Build the author dashboard at /dashboard for authenticated authors in Inkwell.

Layout: same sidebar shell as /write — add this as the main content for /dashboard.

Stats row (top of content area):
- Total posts (published + draft)
- Total views (sum across all posts, from GET /api/author/stats)
- Total reads (estimated: users who read > 50% of a post)
- Followers count

Posts table:
- Columns: Title, Status (Published/Draft badge), Views, Published date, Actions
- Actions: Edit (→ /write/:postId), Unpublish / Publish toggle, Delete (confirm dialog)
- Filter tabs above the table: All, Published, Drafts
- Empty state per tab: "You haven't published any posts yet. Start writing."

Data: GET /api/author/posts returns all posts for the authenticated user.

Acceptance criteria:
- [ ] Stats row shows correct totals for the authenticated user
- [ ] Toggling publish status updates the post in Supabase and the badge changes immediately
- [ ] Deleting a post requires confirmation and removes the row after success
- [ ] Filter tabs show the correct subset of posts
- [ ] Edit link opens /write/:postId with the correct post loaded
```

---

## Prompt 5 — Admin panel

```
Add an admin panel at /admin for users with role = 'admin' in Inkwell.

Auth: redirect to /dashboard if the user is authenticated but not an admin. Redirect to /login if unauthenticated.

Admin sidebar (replaces the author sidebar for admin users):
- Dashboard, Users, Posts, Categories, Settings

Users page (/admin/users):
- Table: Avatar, Name, Email, Role, Posts published, Joined date, Actions
- Actions: Edit role (dropdown: reader / author / admin), Suspend account, Delete account
- Filter by role (tabs: All, Admin, Author, Reader)
- Search by name or email

Posts page (/admin/posts):
- Same structure as the author dashboard posts table but shows all posts from all authors
- Extra column: Author name (links to their profile)
- Bulk actions: Publish selected, Unpublish selected, Delete selected

Categories page (/admin/categories):
- CRUD table: Name, Slug (auto-generated from name), Post count, Actions (Edit, Delete)
- "Add category" opens an inline row at the top of the table

Acceptance criteria:
- [ ] /admin redirects non-admins to /dashboard
- [ ] Role changes take effect immediately (optimistic update + Supabase patch)
- [ ] Suspending a user prevents them from accessing /write and /dashboard
- [ ] Bulk delete on the posts page requires a confirmation dialog naming the count
- [ ] Category slug auto-generates from the name (lowercased, hyphenated)
```

---

## Prompt 6 — SEO and performance

```
Add SEO metadata and performance optimizations to Inkwell.

SEO (using react-helmet-async or the existing meta solution):
- Home page: title "Inkwell — Ideas worth reading", description from config
- Post detail: title = post title + " | Inkwell", description = post excerpt (max 160 chars)
- Open Graph tags on every page: og:title, og:description, og:image (cover image for posts, default OG image for other pages), og:type
- Twitter Card tags: twitter:card = summary_large_image
- Canonical URL on every page

Images:
- All <img> tags that don't already have it: add loading="lazy" and explicit width + height attributes
- Cover images on the post detail page: add fetchpriority="high" (above the fold, most important image)
- Author avatars: serve at 2x resolution (128px source for 64px display)

Sitemap:
- Generate a sitemap at /sitemap.xml listing all published posts with their lastmod date
- Fetch post list from GET /api/posts?limit=1000&status=published

Robots.txt:
- Allow all crawlers
- Point to /sitemap.xml

Acceptance criteria:
- [ ] Post detail page has correct og:title, og:description, og:image, and twitter:card tags
- [ ] Home page has its own distinct og:title and description
- [ ] All post card images have loading="lazy"
- [ ] Cover image on post detail has fetchpriority="high"
- [ ] /sitemap.xml returns valid XML with all published post URLs
- [ ] /robots.txt is accessible and references the sitemap
```

---

## Result

After these 6 prompts, Inkwell is a production-ready multi-author blog platform with:

- Public homepage with featured post and paginated grid
- MDX post rendering with syntax highlighting and a copy button
- TipTap rich text editor with auto-save and cover image uploads
- Author dashboard for managing and publishing posts
- Role-based admin panel for user and content management
- Full SEO metadata, Open Graph tags, sitemap, and lazy loading

Total build time: approximately 3 hours using Lovable.
