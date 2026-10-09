---
name: write-blog-post
description: Draft a new Markdown blog post for Negar's Nook with the correct frontmatter, tags, and voice. Use when the user asks to write, draft, or add a blog post, article, or note in src/content/blog.
---

# Write a Blog Post

Create a new file in `src/content/blog/` as `{kebab-case-slug}.md`.

## Frontmatter

Match existing posts. Required fields:

```yaml
---
author: Negar Baharmand
pubDatetime: 2026-08-21T09:00:00.000Z
title: Clear, specific title
slug: kebab-case-slug
featured: false
draft: true
tags:
  - javascript
description: One or two sentences summarizing the post. Shown on cards and in RSS.
---
```

- `pubDatetime` must be an ISO date (the collection schema requires `z.date()`).
- `slug` is the URL segment under `/posts/`.
- Set `draft: true` unless the user asks to publish.
- Prefer existing tag names when they fit (`javascript`, `java`, `react`, `database`, `project`). Add a new tag only when none of those apply.
- Do not invent `ogImage` unless the user supplies one (min 1200×630).

## Body

- First-person, practical, and specific. Open with why the topic matters, then walk through what you built or learned.
- Use `##` / `###` headings. Keep paragraphs short.
- Include concrete code or commands when they help; skip filler intros.

## After writing

Tell the user the file path and that they can preview with `npm run dev`.
