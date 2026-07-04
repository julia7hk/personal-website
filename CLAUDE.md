# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website at julia7hk.com, built with Astro (static output) and deployed to GitHub Pages via GitHub Actions. Only dependency is Astro itself — no UI frameworks, CSS libraries, or CMS.

## Development Commands

- `npm run dev` — Start dev server at localhost:4321
- `npm run build` — Build to ./dist/
- `npm run preview` — Preview production build locally

## Architecture

### Two-Layout System

- **Layout.astro** — Used by all main pages (home, about, experience, projects, contact). Contains the fixed navbar with profile image, mobile hamburger menu, and footer.
- **BlogPost.astro** — Used by blog posts. Extends the navbar with a "Blog" link and adds comprehensive markdown content styling. Supports series navigation (prev/next) via `dayNumber` frontmatter.

Navbar code is duplicated between both layouts (not extracted into a shared component).

### Blog System

Blog posts are **plain markdown files** in `src/pages/blog/`, NOT Astro Content Collections. Posts are organized into series folders:

- `src/pages/blog/spotify-project/` — multi-part series
- `src/pages/blog/sase-sniping-discord-bot/` — multi-part series
- `src/pages/blog/fixing-the-sase-website/` — multi-part series
- `src/pages/blog/miscellaneous/` — standalone posts

**Blog index** (`blog.astro`) discovers posts via `Astro.glob()` and groups them by folder.
**Dynamic routing** (`[...slug].astro`) uses `getStaticPaths()` with `import.meta.glob()`.

Blog post frontmatter: `title`, `date`, `description`, `dayNumber` (optional, for series ordering), `tags` (optional).

### Styling

- Pure scoped CSS in `<style>` tags within each `.astro` file; global styles use `is:global`
- Font: Atkinson Hyperlegible Mono (Google Fonts, loaded in layouts)
- Color palette: #fafaf8 background, #2a2a2a primary text, #5372aa accent, #a9b4cd accent light
- Responsive breakpoint at 768px
- Card pattern: bordered containers with left accent stripe (`border-left: 3px solid #5372aa`)

## Deployment

- GitHub Pages via `.github/workflows/deploy.yml` (triggers on push to `main`)
- Custom domain configured via `public/CNAME` — this file must stay in `public/` so Astro includes it in the build output
- `astro.config.mjs` sets `site: 'https://julia7hk.com'`
