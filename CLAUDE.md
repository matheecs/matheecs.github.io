# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based GitHub Pages blog (personal notebook) by Jixiang Zhang (matheecs), hosted at `matheecs.tech`. The content focuses on robotics, control theory, math, SLAM, state estimation, and programming — primarily in Chinese with some English.

## Commands

```bash
# Install dependencies (Ruby + Bundler required)
bundle install

# Run locally with live reload
bundle exec jekyll serve --livereload

# Build for production (outputs to _site/)
bundle exec jekyll build
```

Ruby, Bundler, and Jekyll are **not** installed in the current development environment. They must be installed first (`ruby`, `ruby-bundler`, and gems) before the above commands work.

## Architecture

- **Theme**: Uses the custom `moving` gem (`matheecs/moving` remote theme on GitHub). This is a fork/customization of the Minima theme. The `moving` config block in `_config.yml` controls theme-specific settings (avatar, date format, social links).
- **No local layouts or Sass**: The site has no `_layouts/` or `_sass/` directories — all layout and styling comes from the `moving` theme gem. To override a theme layout, create `_layouts/<name>.html` locally.
- **Only custom include**: `_includes/footer.html` — adds a Busuanzi visitor counter to the footer.
- **Content pages** (non-post, non-index):
  - `about.md` — large personal/about page (layout: `about`)
  - `booklist.md` — reading list (layout: `page`)
  - `buglist.md` — bugs/issues log (layout: `page`)
- **Posts**: ~190 posts in `_posts/`, standard Jekyll naming (`YYYY-MM-DD-slug.md`). Front matter uses `layout: post`, `categories`, and `author` fields.
- **Drafts**: ~24 drafts in `_drafts/` with non-standard filenames (some lack proper date prefixes, one has an accompanying `.png` and `.pdf`). Jekyll's `--drafts` flag will only pick up drafts with valid date-prefixed filenames.
- **Static files**: `files/` contains PDFs (lecture notes, CV), HTML animations, MP4 video, and a `cv/` subdirectory with LaTeX resume sources.
- **Images**: `images/` stores blog post images, organized by year/month subdirectories.

## Key Configuration

- **Domain**: `matheecs.tech` (set in `CNAME`; GitHub Pages enforces this)
- **Plugins**: Only `jekyll-feed` (generates RSS)
- **Jekyll version**: `~> 4.0.0` (pinned in Gemfile)
- **Permalinks**: Default Jekyll permalink scheme (date-based: `/:categories/:year/:month/:day/:title.html`)

## Content Conventions

- Posts are written in Markdown with Chinese as the primary language; some posts mix Chinese and English technical terms.
- Post categories include: `study`, `tech`, `robotics`, `math`, `life`, etc. Categories are used in URL paths.
- The `about.md` layout (`about`) is provided by the `moving` theme — it renders the author profile page.
- Drafts sometimes contain incomplete filenames (missing date format). Only date-prefixed drafts work with `jekyll serve --drafts`.

## Deployment

Deployment is handled by GitHub Pages. Pushing to the `master` branch triggers automatic build and deployment via GitHub's pages-build-deployment workflow (no custom Actions in `.github/`).
