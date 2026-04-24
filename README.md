# Your GitHub Pages Blog

A clean, minimal Jekyll-based blog template inspired by [amitness.com](https://amitness.com).

## Features

- 🎨 Clean, minimalist design
- 🌓 Dark/light mode support
- 📱 Fully responsive
- 🏷️ Category filtering
- ✍️ Easy Markdown-based posting
- ⚡ Fast and lightweight

## Quick Start

### 1. Configure Your Site

Edit `_config.yml` and update:
- `title`: Your name/site title
- `description`: Your tagline
- `url`: Your GitHub Pages URL (e.g., `https://username.github.io`)

### 2. Add a New Blog Post

Create a new file in `_posts/` with the format: `YYYY-MM-DD-title.md`

Example front matter:

```yaml
---
layout: post
title: Your Post Title
category: tutorials
read_time: 5
thumbnail: /assets/images/your-image.jpg
---

Your content here in Markdown...
```

### 3. Available Categories

Add any category you want in the `category` field. Common ones:
- `tutorials`
- `thoughts`
- `news`
- `projects`

### 4. Test Locally

```bash
# Install Bundler if needed
gem install bundler

# Install dependencies
bundle install

# Start local server
bundle exec jekyll serve
```

Visit `http://localhost:4000`

### 5. Deploy to GitHub Pages

1. Push to your repository
2. Go to Settings > Pages
3. Select source: Deploy from a branch
4. Select branch: `main` or `master`
5. Your site will be live at `https://username.github.io`

## Project Structure

```
.
├── _config.yml        # Site configuration
├── _layouts/          # HTML templates
│   ├── default.html
│   └── post.html
├── _pages/            # Additional pages (About, etc.)
├── _posts/            # Your blog posts
├── assets/
│   ├── css/           # Stylesheets
│   └── images/        # Images
└── index.html         # Homepage
```

## Customization

### Styling

Edit `assets/css/style.css` to customize colors, fonts, and layout.

### Navigation

Add links to `_config.yml` under the `nav` section.

## Writing Tips

- Use clear, descriptive titles
- Add relevant categories for filtering
- Include estimated read time
- Add thumbnails for visual appeal
- Use proper Markdown for formatting

## Need Help?

- [Jekyll Docs](https://jekyllrb.com/docs/)
- [GitHub Pages Docs](https://docs.github.com/en/pages)
