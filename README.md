# .github

Shared GitHub configuration for Lameco projects.

## Reusable Workflows

### [deploy-php.yml](.github/workflows/deploy-php.yml)
Deployment workflow for PHP projects (Symfony, Craft CMS) using Deployer. Builds assets in CI, then deploys via Deployer.

```yaml
jobs:
  deploy:
    uses: Lameco-Development/.github/.github/workflows/deploy-php.yml@main
    with:
      environment: production
      asset_dirs: 'web/dist'  # public/dist for Symfony, web/dist for Craft CMS
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

### [deploy-node.yml](.github/workflows/deploy-node.yml)
Deployment workflow for Node.js projects. Builds with pnpm, rsyncs to VPS, restarts via supervisord.

```yaml
jobs:
  deploy:
    uses: Lameco-Development/.github/.github/workflows/deploy-node.yml@main
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
