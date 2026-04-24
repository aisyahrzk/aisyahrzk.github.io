---
layout: post
title: Welcome to Your New Blog
category: general
read_time: 3
thumbnail: /assets/images/welcome-thumbnail.jpg
---

This is your first blog post! You can edit or delete this file and start writing your own content.

## Getting Started

To add a new blog post, create a new markdown file in the `_posts` folder with the following naming convention:

```
YYYY-MM-DD-your-post-title.md
```

## Front Matter

Each post should start with front matter like this:

```yaml
---
layout: post
title: Your Post Title
category: your-category
read_time: 5
thumbnail: /assets/images/your-image.jpg
---
```

## Supported Front Matter Fields

- `layout`: Always use "post" for blog posts
- `title`: Your post title
- `category`: A category for filtering (e.g., "tutorial", "thoughts", "news")
- `read_time`: Estimated reading time in minutes
- `thumbnail`: Path to a thumbnail image (optional)

## Writing Content

You can use standard Markdown syntax:

- **Bold text** with `**text**`
- *Italic text* with `*text*`
- [Links](https://example.com) with `[text](url)`
- Code with backticks

### Code Blocks

```python
def hello():
    print("Hello, World!")
```

### Lists

1. First item
2. Second item
3. Third item

### Blockquotes

> This is a blockquote. Great for highlighting important information.

Happy blogging!
