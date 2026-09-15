# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website at julia7hk.com, built with Astro (static output) and deployed to GitHub Pages via GitHub Actions. Only dependency is Astro itself — no UI frameworks, CSS libraries, or CMS.

## Development Commands

- `npm run dev` — Start dev server at localhost:4321
- `npm run build` — Build to ./dist/
- `npm run preview` — Preview production build locally
- `npm run astro check` — Type-check (TS `strict` via `astro/tsconfigs/strict`)

There is no test suite or linter configured — `astro check` is the only static verification.

## Architecture

### Two-Layout System

- **Layout.astro** — Used by all main pages (home, about, experience, projects, contact, blog index). Renders `<Navbar />`, a `<slot />` inside `<main>`, and `<Footer />`.
- **BlogPost.astro** — Used by blog posts. Same `<Navbar />` / `<Footer />` shell, plus comprehensive markdown content styling, a breadcrumb back to `/blog`, series navigation (prev/next via `dayNumber` frontmatter) in the header, and a post-footer nav.

### Shared Chrome Components

- **`src/components/Navbar.astro`** — the single source of truth for site navigation. The `links` array in its frontmatter drives every page's nav; adding an entry there makes it appear site-wide. Owns the mobile hamburger script and all nav CSS (scoped, not `is:global`). Marks the current page with `.active` + `aria-current="page"`; `/blog/<post>` keeps the Blog link active via a prefix match.
- **`src/components/Footer.astro`** — site-wide footer on every page. Its Blog column is derived from the actual posts via `import.meta.glob`, using the same title-prefix grouping and anchor-id slugging as `blog.astro`, so a new series appears automatically. A title prefix with only one post is treated as standalone (lands under Miscellaneous), matching `blog.astro`'s `isProject` rule.

If you change the grouping rule in `blog.astro`, mirror it in `Footer.astro` and in `index.astro`'s "From the Blog" section — all three derive series names from post titles independently.

### Blog System

Blog posts are **plain markdown files** in `src/pages/blog/`, NOT Astro Content Collections. Posts are organized into series folders:

- `src/pages/blog/spotify-project/` — multi-part series
- `src/pages/blog/sase-sniping-discord-bot/` — multi-part series
- `src/pages/blog/fixing-the-sase-website/` — multi-part series
- `src/pages/blog/miscellaneous/` — standalone posts

**Blog index** (`blog.astro`) discovers posts via `Astro.glob()` and groups them by a `projectName` derived from the post **title** — the part before `" - "` (e.g. `"Spotify Project - Day 1"` → group `"Spotify Project"`). This is title-based, NOT folder-based, so a post's title prefix determines its group on the index.
**Dynamic routing** (`[...slug].astro`) uses `getStaticPaths()` with `import.meta.glob()` and groups by **folder** to compute prev/next series navigation. All posts resolve to a flat `/blog/<filename>` URL regardless of folder, so filenames must be unique across folders.

Blog post frontmatter: `title`, `date`, `description`, `dayNumber` (optional, for series ordering), `tags` (optional). Blog images live in `public/blog-images/`.

### Home Page

`index.astro` ends with a "From the Blog" section listing the most recent post from each of up to four series (deduped by series so one active project doesn't fill the list). It globs `./blog/**/*.md` directly and reuses the `.roles-list` / `.role-item` styles from the Current Roles section.

### Projects Page

`projects.astro` has a client-side category filter: `.filter-btn` buttons carry a `data-filter` value and each `.project-card` carries a space-separated `data-category` (e.g. `data-category="web-app machine-learning"`). A small inline script shows/hides cards on click. When adding a project, its `data-category` values must match existing filter-button `data-filter` values, or add a new button.

### Styling

- Pure scoped CSS in `<style>` tags within each `.astro` file; global styles use `is:global`
- Font: Atkinson Hyperlegible Mono (Google Fonts, loaded in layouts)
- Color palette: #fafaf8 background, #2a2a2a primary text, #5372aa accent, #a9b4cd accent light
- Responsive breakpoint at 768px
- Card pattern: bordered containers with left accent stripe (`border-left: 3px solid #5372aa`)

## Deployment

- GitHub Pages via `.github/workflows/deploy.yml` (triggers on push to `main`)
- Custom domain configured via `public/CNAME` — this file must stay in `public/` so Astro includes it in the build output
- `astro.config.mjs` sets `site: 'https://julia7hk.com'` and `base: '/'` (served at domain root, not a subpath)
