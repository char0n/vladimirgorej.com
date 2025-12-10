# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is Vladimír Gorej's personal website built with Jekyll and hosted on GitHub Pages. It includes a blog, certifications showcase, and book reviews.

## Development Commands

```bash
# Install dependencies
bundle install

# Run development server (auto-reloads on changes, except _config.yml)
bundle exec jekyll serve

# Build the site
bundle exec jekyll build
```

**Note:** Changes to `_config.yml` require restarting the server.

## Architecture

### Collections
The site uses three Jekyll collections (defined in `_config.yml`):
- `_posts/` - Blog posts (`/blog/:title/`)
- `_certifications/` - Certification entries (`/certifications/:title/`)
- `_book-reviews/` - Book reviews (`/book-reviews/:title/`)

### Layout Hierarchy
- `_layouts/base.html` - Base template with head, header, content, footer
- `_layouts/default.html` - Extends base (wrapper)
- `_layouts/post.html` - Blog post template
- `_layouts/certification.html` - Certification entry template
- `_layouts/book-review.html` - Book review template

### Key Includes (`_includes/`)
- `head.html` - Meta tags, SEO, styles
- `header.html` - Navigation
- `footer.html` - Site footer with social links
- `card.html` - Reusable card component for homepage
- `google-analytics.html` - Analytics tracking
- `disqus_comments.html` - Comment system

### Pages (`pages/`)
- `homepage.html` - Main landing page with projects and social links
- `blog.html`, `certifications.html`, `book-reviews.html` - List pages
- `privacy-policy.html` - Privacy policy

### Static Assets
- `assets/` - CSS, images, PDFs (certification files in `assets/pdf/certifications/`)
- Favicon files in root directory

## Creating Certification Preview Images

To create a webp preview image from a certification PDF:

```bash
# Step 1: Convert PDF to PNG (150 DPI)
pdftoppm -png -r 150 -singlefile assets/pdf/certifications/CERT_NAME.pdf /tmp/CERT_NAME

# Step 2: Convert PNG to WebP
ffmpeg -i /tmp/CERT_NAME.png -quality 80 assets/img/certifications/CERT_NAME.webp

# Step 3: Get image dimensions for the markdown front matter
file assets/img/certifications/CERT_NAME.webp
```

## Technical Details

- Ruby version: 3.0.4 (specified in `.ruby-version`)
- Theme: minima (~> 2.5)
- GitHub Pages gem: ~> 215
- Plugins: jekyll-feed, jekyll-sitemap
