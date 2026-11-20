# aisyahrzk.github.io

Personal site, built with Jekyll and hosted on GitHub Pages at https://aisyahrzk.github.io.

## Writing a post

Add a markdown file to `_posts/` named `YYYY-MM-DD-slug.md`:

```markdown
---
layout: post
title: "My post title"
date: 2026-11-20 10:00 +0800
categories: research
---

Post body in markdown. Images go in `assets/images/` and are linked as `/assets/images/name.png`.
```

Commit and push; the site rebuilds in about a minute. To keep a draft out of the site, add `published: false` to its front matter.

## Pages

| File | Page |
|---|---|
| `index.html` | Home: short bio, recent posts, publications |
| `blog.html` | All posts |
| `cv.html` | Selected work, experience, education, skills |
| `_layouts/` | Site shell (`default.html`) and post template (`post.html`) |
| `assets/css/style.css` | Styles, light and dark mode |
| `assets/images/profile.jpg` | Optional photo (initials shown if missing) |

## Local preview

```bash
bundle install
bundle exec jekyll serve
```

Then open http://localhost:4000.
