# Code Standards & Development Guidelines

## General Instructions

- Always follow the project's coding standards and best practices.
- Run code formatters and linters before committing.
- Write clear commit messages and document significant changes.
- Ensure all tests pass before pushing code.
- Keep dependencies up to date and avoid committing generated/vendor files.
- Update documentation (`docs/` folder) when introducing new features or changes.

---

## Symfony Projects

### Required Before Each Commit
- Run `make cs-fix` or `php-cs-fixer fix` before committing any changes to ensure proper PHP code formatting.
- Ensure all PHP files conform to PSR-12 and Symfony coding standards.

### Development Flow
- Build/Compile assets: `make build` or `yarn build` (if using Webpack Encore)
- Run tests: `make test` or `php bin/phpunit`
- Full CI check: `make ci` (includes cs-fix, lint, test)

### Repository Structure
- `src/`: Main Symfony application code (Controllers, Entities, Services, etc.)
- `config/`: Symfony and application configuration files
- `templates/`: Twig templates for views
- `public/`: Web server document root (entry point: `index.php`)
- `migrations/`: Doctrine database migrations
- `translations/`: Translation files
- `tests/`: PHPUnit test cases
- `bin/`: Executable scripts (e.g., `console`)
- `var/`: Cache and logs (should not be committed)
- `vendor/`: Composer dependencies (should not be committed)
- `assets/`: Frontend assets (if using Webpack Encore)
- `docs/`: Project documentation

### Key Guidelines
1. Follow [Symfony Best Practices](https://symfony.com/doc/current/best_practices.html) and PSR standards (PSR-1, PSR-4, PSR-12)
2. Maintain existing code structure and organization
3. Use dependency injection for services via Symfony's service container
4. Write unit and functional tests for new functionality using PHPUnit
5. Document public APIs, services, and complex logic. Update the `docs/` folder as needed
6. Use environment variables for configuration (see `.env` files)
7. Use Composer for dependency management and keep dependencies up to date
8. Use Doctrine ORM for database access and migrations
9. Use Twig for templating and translation best practices
10. Review and update translations when modifying user-facing text

---

## CraftCMS Projects

### Required Before Each Commit
- Run `composer format` or `php-cs-fixer fix` to ensure code style compliance.
- Ensure all PHP and Twig files follow CraftCMS and PSR-12 standards.

### Development Flow
- Build/Compile assets: `npm run build` or `yarn build` (if using frontend tooling)
- Run tests: `composer test` (if tests are present)
- Full CI check: `composer ci` (if configured)

### Repository Structure
- `craft/`: CraftCMS core files (should not be modified directly)
- `config/`: Project configuration files
- `modules/` or `plugins/`: Custom modules and plugins
- `templates/`: Twig templates for site rendering
- `web/`: Web root (entry point: `index.php`)
- `migrations/`: Database migrations (if used)
- `translations/`: Translation files
- `storage/`: Logs and runtime files (should not be committed)
- `vendor/`: Composer dependencies (should not be committed)
- `assets/`: Frontend assets (images, CSS, JS)
- `docs/`: Project documentation

### Key Guidelines
1. Follow [CraftCMS Coding Standards](https://craftcms.com/docs/4.x/coding-guidelines.html) and PSR-12
2. Use environment variables for configuration (see `.env` files)
3. Keep custom code in `modules/` or `plugins/`, not in core or vendor directories
4. Write unit and integration tests for custom modules/plugins where possible
5. Document custom functionality and update the `docs/` folder as needed
6. Use Composer for dependency management and keep dependencies up to date
7. Use migrations for database changes when possible
8. Use Twig best practices for templates and translations
9. Do not commit sensitive information or credentials
10. Review and update translations when modifying user-facing text
