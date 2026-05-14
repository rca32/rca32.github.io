# AGENTS.md

## Site Purpose

This repository is the GitHub Pages source for `blog.chartagent.net`.
It is a Jekyll-based blog using the `minima` theme.

## Publishing Rules

- Keep the site source at the repository root.
- Do not use the `docs/` directory for GitHub Pages output or custom domain settings.
- Keep the root `CNAME` set to `blog.chartagent.net`.
- Do not create another `/blog/` URL prefix because the subdomain already identifies the site as the blog.

## Post Structure

Blog posts should live in `_posts/`.

Use language-specific post files when publishing translated content.

Use this filename format:

```text
YYYY-MM-DD-title.lang.md
```

Example:

```text
_posts/2026-05-14-first-post.ko.md
_posts/2026-05-14-first-post.en.md
_posts/2026-05-14-first-post.ja.md
_posts/2026-05-14-first-post.zh.md
```

Use this front matter for regular posts:

```markdown
---
layout: post
title: "Post title"
date: 2026-05-14 10:00:00 +0900
lang: ko
ref: first-post-2026-05-14
permalink: /ko/2026/05/14/first-post/
---
```

Use categories only when they describe the actual subject, such as `ai`,
`chart`, `market`, or `data`. Do not use `blog` as a category.

Use the same `ref` value for every translation of the same post.

## URL Policy

Use language prefixes for public content and never add a `/blog/` prefix.

The root `/` page detects the visitor's browser language and redirects to the
best supported language home. Supported language homes are:

```text
/ko/
/en/
/ja/
/zh/
```

Expected post URL formats:

```text
https://blog.chartagent.net/ko/2026/05/14/first-post/
https://blog.chartagent.net/en/2026/05/14/first-post/
https://blog.chartagent.net/ja/2026/05/14/first-post/
https://blog.chartagent.net/zh/2026/05/14/first-post/
```

The root language detection uses `localStorage` key
`chartagent.preferredLang`. Manual language selection should update this key.

## Visibility And Safety

- `AGENTS.md` is excluded from Jekyll output via `_config.yml`.
- Excluding a file from Jekyll output does not hide it from GitHub if the repository is public.
- Do not commit secrets, API keys, tokens, private prompts, unpublished business plans, or private data.
- Public blog content should be treated as fully public once committed and pushed.

## Writing Guidance

- Write posts in Korean unless there is a clear reason to use another language.
- Keep titles concrete and searchable.
- Prefer Markdown content over custom HTML unless the post needs a specific layout.
- Store images and other public assets under `assets/`.
- Use relative links for local site assets where possible.
