# CONCEPT.md

## Project Name

personal-website

## Status

Draft proposal. Confirm with user before treating as validated.

## Objective

A personal website that showcases coding skills and projects, effectively migrating and enhancing the content from the current GitHub profile README (`CoderMan73/README.md`) into a polished standalone site. Hosted entirely free on GitHub Pages.

## Goal / Outcome

- A clean, fast-loading personal website deployed at `CoderMan73.github.io` (the `username.github.io` repo convention).
- A dedicated project showcase page that auto-syncs with GitHub metadata (repo stars, descriptions, languages, status badges) at build time, so the portfolio stays current without manual edits.
- Migration of the README's core content (about, tech stack badges, GitHub stats, featured projects) into a structured multi-page site.
- Optional blog for technical writing, built on the same stack.

## Technical Goals

1. **Static site generation**: Use a modern static site generator that builds at deploy time and outputs clean static HTML, avoiding heavy client-side JavaScript.
2. **GitHub API integration at build time**: Fetch live repository data (name, description, stars, forks, primary language, topics, last updated, license) during the build step to populate the project showcase page dynamically.
3. **Project showcase page**: Dedicated page listing repositories grouped by category/status (featured, active, archived), with links to live demos, source, and real-time stats.
4. **Optional blog**: Support for Markdown-based blog posts with automatic routing, no database or server required.
5. **No JavaScript runtime required for visitors**: Site should be navigable and readable without client-side hydration, though minimal interactivity (dark mode toggle) is acceptable.
6. **Deploy pipeline**: GitHub Actions workflow that builds the site and deploys to GitHub Pages automatically on push to `main`.

## Scope

### MVP

- Homepage with about section, tech stack display, quick facts (remote, work auth, relocation).
- GitHub stats display (streak, top languages) via existing badge services or API.
- Project showcase page auto-synced with GitHub repos at build time.
- Contact links (GitHub, email, LinkedIn).
- Responsive design (mobile-friendly).
- GitHub Actions CI/CD deploy to GitHub Pages.

### Post-MVP (optional, not committed)

- Blog with Markdown posts, tags, and archive page.
- Dark/light mode toggle with persistence.
- Interactive project filtering (by language, topic, status).
- Performance optimizations (image optimization, font optimization).
- Resume download and detailed experience page.

## Design Decisions

- **Hosting**: GitHub Pages via the standard `username.github.io` repository convention, deploying from `main` branch (root or `/docs`).
- **Static site generator**: Astro. Chosen because:
  - Outputs zero-JS static HTML by default (clean and simple, not React-heavy).
  - Native Markdown/MDX support for the optional blog.
  - Build-time data fetching integrates naturally with the GitHub API.
  - Excellent developer experience and ecosystem.
  - Deploys cleanly to GitHub Pages via Actions.
- **Data source**: GitHub REST API v3 at build time. Fetched repository data is cached/generated into static files, so no client-side API calls are needed.
- **Styling**: Lightweight CSS, no heavy UI framework. Tailwind CSS is the preferred option if a utility framework is used; otherwise, clean hand-written CSS.
- **Blog**: Built-in Astro Markdown (MD) support, no CMS. Posts authored as local Markdown files.

## Unresolved Decisions

- Exact color scheme and visual identity (beyond migrating badge aesthetics from the README).
- Whether to include the full collapsible career archive from the README, or restructure it into dedicated pages (experience, education, awards, presentations).
- Whether GitHub stats should use third-party badge services (current README approach) or be fetched from the GitHub API directly at build time for more control.
- Whether the site should support i18n (the README mentions multi-language versions exist in the `career-ops` project, but the personal README is English-only).
- Project status categorization logic: how to map repos to featured/active/archived on the showcase page (tags, topics, manual override?).

## Constraints & Restrictions

- Must be free to host on GitHub Pages (no paid services, no serverless functions).
- Build step is acceptable (user is fine with CI/CD workflow).
- Client-side JavaScript should be minimal (React-level frameworks avoided).
- No custom domain initially; `CoderMan73.github.io` is acceptable.
- GitHub API rate limits apply to build-time fetching (unauthenticated: 60 req/hour; authenticated: 5000 req/hour — a GitHub token in Actions is preferred).

## User Preferences

- Clean and simple, not React-heavy.
- Auto-sync with GitHub repo data is valued over manual curation.
- Dedicated project showcase page that pulls live repo info.
- Blog is optional/nice-to-have.
- Standard GitHub Pages deployment (`username.github.io` convention).

---

This document is a draft proposal. It has not been validated against the user's intent and should be reviewed and confirmed before being treated as finalized or used to guide development.
