# julia7hk.com

Personal website showcasing some of my experience, projects, etc.

## Project Structure

```text
/
├── public/              # Static assets (favicon, fonts, images)
├── src/
│   ├── assets/          # Processed assets
│   ├── components/      # Reusable Astro components
│   ├── layouts/         # Page layout templates (Layout.astro)
│   └── pages/           # File-based routing
│       ├── index.astro       # Homepage
│       ├── about.astro       # About page
│       ├── experience.astro  # Professional experience
│       ├── projects.astro    # Projects showcase
│       └── contact.astro     # Contact information
├── astro.config.mjs     # Astro configuration
├── tsconfig.json        # TypeScript configuration
└── CLAUDE.md            # Development guidance for Claude Code
```

## Development

| Command                   | Action                                           |
| :------------------------ | :----------------------------------------------- |
| `npm install`             | Install dependencies                             |
| `npm run dev`             | Start dev server at `localhost:4321`             |
| `npm run build`           | Build production site to `./dist/`               |
| `npm run preview`         | Preview build locally before deploying           |
| `npm run astro ...`       | Run CLI commands like `astro add`, `astro check` |
| `npm run astro -- --help` | Get help using the Astro CLI                     |

## Pages

- **Home** (`/`) - Landing page with introduction
- **About** (`/about`) - Background and interests
- **Experience** (`/experience`) - Professional roles and leadership positions
- **Projects** (`/projects`) - Technical projects and contributions
- **Contact** (`/contact`) - Get in touch

## Resources

- [Astro Documentation](https://docs.astro.build)