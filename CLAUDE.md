# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal website built with Astro 5.13.9 using TypeScript. The project follows Astro's basic starter template structure and uses the strict TypeScript configuration.

## Development Commands

- `npm run dev` - Start development server at localhost:4321
- `npm run build` - Build production site to ./dist/
- `npm run preview` - Preview build locally before deploying
- `npm run astro ...` - Run Astro CLI commands

## Project Structure

The codebase follows Astro's standard structure:

```
/
├── public/          # Static assets served directly
├── src/
│   ├── assets/      # Processed assets (images, etc.)
│   ├── components/  # Reusable Astro components
│   ├── layouts/     # Page layout templates
│   └── pages/       # File-based routing (index.astro = homepage)
├── astro.config.mjs # Astro configuration (currently minimal)
├── tsconfig.json    # TypeScript config (extends Astro strict)
└── package.json     # Dependencies and scripts
```

## Technology Stack

- **Framework**: Astro 5.13.9 (Static Site Generator with islands architecture)
- **Language**: TypeScript with strict configuration
- **Styling**: Component-scoped CSS within .astro files
- **Module System**: ES modules (`"type": "module"` in package.json)

## File Patterns

- `.astro` files contain frontmatter (TypeScript), HTML template, and scoped CSS
- Pages in `src/pages/` map to routes (index.astro = /, about.astro = /about)
- Components use Astro's component syntax with TypeScript frontmatter
- Layout components provide page structure and are imported into pages

## Current Implementation

The site currently uses the default Astro starter with:
- Basic Layout component providing HTML structure
- Welcome component with styled hero section and navigation links
- Component-scoped CSS with modern styling (gradients, backdrop filters)
- Responsive design with mobile breakpoints