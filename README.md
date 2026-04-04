# .github

Shared GitHub configuration for Lameco projects.

## Reusable Workflows

### [deploy-php.yml](.github/workflows/deploy-php.yml)
Deploy-only workflow for PHP projects without a frontend build step (e.g. Asset Mapper). Just Composer + Deployer.

```yaml
jobs:
  deploy:
    uses: Lameco-Development/.github/.github/workflows/deploy-php.yml@main
    with:
      environment: production
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

### [build-deploy-php.yml](.github/workflows/build-deploy-php.yml)
Full workflow for PHP projects with frontend assets (Vite/Webpack). Builds assets in CI, then deploys via Deployer.

```yaml
# Symfony (asset_dirs defaults to public/dist)
jobs:
  deploy:
    uses: Lameco-Development/.github/.github/workflows/build-deploy-php.yml@main
    with:
      environment: production
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

```yaml
# Craft CMS (override asset_dirs)
jobs:
  deploy:
    uses: Lameco-Development/.github/.github/workflows/build-deploy-php.yml@main
    with:
      environment: production
      asset_dirs: 'web/dist'
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

### [build-deploy-node.yml](.github/workflows/build-deploy-node.yml)
Deployment workflow for Node.js projects. Builds with pnpm, rsyncs to VPS, restarts via supervisord.

```yaml
jobs:
  deploy:
    uses: Lameco-Development/.github/.github/workflows/build-deploy-node.yml@main
    with:
      environment: production
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
      SSH_HOST: ${{ secrets.SSH_HOST }}
      SSH_USER: ${{ secrets.SSH_USER }}
```

### [craft-update-notes.yml](.github/workflows/craft-update-notes.yml)
Parses `composer.lock` changes on PRs and posts a comment with security advisories and breaking change notices.

```yaml
jobs:
  changelog:
    uses: Lameco-Development/.github/.github/workflows/craft-update-notes.yml@main
```

## Other

- **[guidelines/](.github/guidelines/)** — AI coding guidelines for Craft CMS, Tailwind CSS, JavaScript, and Twig
- **[docs/](docs/)** — Craft CMS conventions checklist and Craft 4→5 upgrade guide
- **[renovate.json](renovate.json)** — Monthly dependency update PRs for Craft CMS packages
