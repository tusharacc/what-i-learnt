# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal Jekyll blog ("What I learnt!") hosted on GitHub Pages. Posts document real programming experiences — bugs, tools, experiments, and lessons from actual projects.

## Local development

```bash
bundle exec jekyll serve        # serve with live reload at http://localhost:4000
bundle exec jekyll build        # build static site to _site/
```

If `bundle` is not installed: `gem install bundler jekyll`, then `bundle install`.

## Adding a new post

- **Markdown posts** go in `_posts/` as `YYYY-MM-DD-slug.md` with front matter:
  ```yaml
  ---
  title: "Post Title"
  date: YYYY-MM-DD
  ---
  ```
- **HTML posts** (custom-styled, self-contained) also go in `_posts/` as `YYYY-MM-DD-slug.html` with the same front matter prepended before `<!DOCTYPE html>`.

## Post style and voice

Posts are written in first person, from direct personal experience. They describe real mistakes and what they taught — not tutorials or generic explainers. The writing is candid and often self-deprecating. Match this tone when drafting or editing posts.

## Site structure

- `_config.yml` — Jekyll config; uses `minima` theme with dark skin
- `_posts/` — all blog posts (markdown and HTML)
- `about.md`, `portfolio.md` — static pages linked in the nav header
- `assets/` — static assets
- `docs/` — built output (if GitHub Pages is configured to serve from `/docs`)

## HTML post conventions

Recent posts (`2026-02-13-claude-orchestrator-report.html`, `2026-03-18-does-code-assistant-need-large-models.html`) use a consistent self-contained dark-themed design with CSS variables (`--bg`, `--surface`, `--accent`, etc.), callout boxes, pipeline diagrams, and benchmark tables. Reuse this style for new HTML posts.
