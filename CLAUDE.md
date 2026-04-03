# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is Lameco's **shared GitHub configuration repository** (`.github`). It contains:
- Reusable GitHub Actions workflows (called via `workflow_call` from project repos)
- Renovate configuration for Craft CMS dependency management
- Copilot/AI coding guidelines for Lameco projects
- Documentation: Craft CMS conventions, upgrade guides

This repo is **not a buildable application** — there are no build, test, or lint commands.

## Architecture

```
.github/
├── workflows/
│   ├── deploy-php.yml            # Reusable PHP deployment (Deployer): build assets + deploy
│   ├── deploy-node.yml           # Reusable Node deployment: build + rsync to VPS + restart
│   └── craft-update-notes.yml    # Parses composer.lock diffs on PRs, posts changelog
│                                   notices (security, breaking changes) as PR comments
├── guidelines/                   # AI coding guidelines (referenced by copilot-instructions.md)
│   ├── craft-cms.md              # Craft CMS field, entry type, and PageBuilder conventions
│   ├── tailwind-css.md           # Tailwind v3 (config.js) vs v4 (@theme) detection
│   ├── javascript.md             # Alpine.js, TypeScript, ES6+ conventions
│   └── twig.md                   # Template patterns: macros vs partials, no hardcoded content
├── copilot-instructions.md       # Entry point referencing all guidelines
docs/
├── craft-conventions.md          # Full project conventions checklist (fields, entry types,
│                                   sections, assets, templates, translations)
├── craft-4-to-5-upgrade-guide.md # Step-by-step Craft 4→5 migration guide
renovate.json                     # Monthly PRs for craftcms/* and third-party Craft plugins
```

## Workflows

### deploy-php.yml
Reusable deployment workflow for Symfony and Craft CMS projects. Two jobs:
1. **build** — installs PHP + Node dependencies, runs `yarn build`, uploads built assets as artifact
2. **deploy** — downloads assets, runs Deployer (`deployphp/action@v1`)

Inputs: `environment`, `asset_dirs` (e.g. `public/dist` for Symfony, `web/dist` for Craft CMS).
Secrets: `SSH_PRIVATE_KEY` (used for both server access and agent forwarding for repo cloning).
PHP version is read from the calling repo's `vars.PHP_VERSION` (defaults to 8.2).

### deploy-node.yml
Reusable deployment workflow for Node.js projects. Single job:
1. Installs pnpm + dependencies, runs `pnpm run build`
2. Rsyncs to VPS via `easingthemes/ssh-deploy`
3. Installs production dependencies on server and restarts via supervisord

Inputs: `environment`.
Secrets: `SSH_PRIVATE_KEY`, `SSH_HOST`, `SSH_USER`.
Node version is read from `.nvmrc` in the project repo.

### craft-update-notes.yml
Triggered on PRs that update `composer.lock`. Compares old/new versions, fetches CHANGELOG.md from GitHub, and posts a PR comment with security advisories and breaking change notices.

## Key Conventions (from guidelines)

### Craft CMS
- Craft 5+: reuse existing fields (see shared fields table in `docs/craft-conventions.md`). Craft <4: create unique fields.
- PageBuilder blocks go in `templates/_page-builder/`, handle ends with `Block` (e.g. `contentMediaBlock`).
- Use native title field (`hasTitleField: true`) instead of custom Plain Text fields for entry names.
- Use Entries instead of Globals, Categories, or Tags (Craft 6 preparation).
- Entry type icons: `page` (has URI), `rectangle-history` (no URI), `cube` (PageBuilder block), `cubes` (sub-block).

### Twig
- No hardcoded content — use Craft variables/fields.
- No Tailwind classes with Twig variables (e.g. avoid `bg-{{ item.color }}`). Use array maps instead.
- Use macros for template-local repeating blocks; use `_partials/` includes for cross-template sharing.

### Tailwind CSS
- Detect version: `tailwind.config.js` = v3, `theme.scss` = v4 (uses `@theme {}` block).
- Mobile-first approach.

### JavaScript
- Alpine.js for interactivity, TypeScript, ES6+, camelCase variables, PascalCase components.

## Renovate
- Runs monthly (first of month).
- Groups `craftcms/*` packages into one "Craft CMS" PR.
- Groups third-party Craft plugins (`verbb/`, `nystudio107/`, `putyourlightson/`, etc.) into one "Craft plugins" PR.
- Both groups include a note that a bot will comment with changelog notices.
