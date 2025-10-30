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

The personal website includes the following pages and features:

### Pages
- **index.astro** - Homepage with hero section, profile image, social links, and technical skills overview
- **about.astro** - Detailed about page with education, comprehensive technical skills with icons, and areas of interest
- **projects.astro** - Projects showcase (if exists)
- **contact.astro** - Contact information (if exists)

### Design System
- **Color Scheme**: Minimal design with #fafaf8 background, #2a2a2a primary text, #4a4a4a secondary text
- **Typography**: System fonts with careful letter-spacing and line-height
- **Components**: Bordered cards, skill tags/badges, social media icons
- **Responsive**: Mobile-first approach with breakpoints at 768px

### Key Features
- Profile image integration using Astro's Image component
- Social media links (GitHub, LinkedIn, Email) with SVG icons
- Technical skills organized by category:
  - Languages (Python, TypeScript, JavaScript, C, Java, SQL, HTML, CSS, R)
  - Web Frameworks/Libraries (React, Next.js, TailwindCSS, Flask, FastAPI, Prisma, React Native)
  - AI Framework/Library (PyTorch, TensorFlow, NumPy, Pandas, Matplotlib)
  - Databases (Postgres, Supabase, Firebase)
  - Cloud (AWS, Google Compute Platform, Oracle Cloud, Netlify)
- Icon-enhanced skill badges on about page using SVG logos
- GPA display and coursework listing on about page
- Areas of interest cards showcasing specializations

### Styling Conventions
- Component-scoped CSS in each .astro file
- Consistent spacing with rem units
- Border-based design with subtle hover effects
- Clean, professional aesthetic suitable for a CS/AI student portfolio