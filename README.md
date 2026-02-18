# WFU GitHub Workflows & Actions

Reusable GitHub Actions workflows and composite actions created by Wake Forest University for standardized CI/CD across WordPress plugin and theme repositories.

## Quick Start

Reference the reusable PHP workflow in your repository:

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  php:
    uses: wakeforestuniversity/wfu-github/.github/workflows/php-workflow.yml@v1
    with:
      phpVersion: '8.2'
    secrets:
      composerAuth: ${{ secrets.COMPOSER_AUTH }}
      qltyCoverageToken: ${{ secrets.QLTY_COVERAGE_TOKEN }}
      securityToken: ${{ secrets.SNYK_TOKEN }}
```

## Available Workflows

### PHP Workflow (`php-workflow.yml`)

A reusable workflow that runs:
- **Composer Setup** — PHP environment setup with dependency caching
- **Composer Validation** — Validates all environment-specific composer files
- **Testing** — PHPUnit with Qlty code coverage reporting
- **Security** — Snyk vulnerability scanning
- **JavaScript Linting** — Optional JS linting if `package.json` has a `lint:js` script

## Available Actions

### PHP Cache Restore (`php-cache-restore`)

A composite action that sets up PHP with Xdebug and restores cached Composer dependencies.

## Documentation

- [Developer Guide](documentation/DEVELOPER_GUIDE.md) — Architecture, APIs, and contribution guidelines
- [Admin Guide](documentation/ADMIN_GUIDE.md) — Configuration and usage for WordPress admins
- [User Guide](documentation/USER_GUIDE.md) — Features visible to website visitors
