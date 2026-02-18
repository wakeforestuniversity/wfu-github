# Developer Guide

## Overview

`wfu-github` is a repository of reusable GitHub Actions workflows and composite actions maintained by Wake Forest University. It provides standardized CI/CD pipelines for PHP-based WordPress projects across the WFU organization. Other WFU repositories reference these workflows and actions to avoid duplicating CI/CD configuration and to maintain consistent quality checks.

The primary offering is a reusable PHP workflow that handles Composer setup, dependency caching, PHPUnit testing, code coverage reporting (via Qlty), security scanning (via Snyk), and JavaScript linting.

## Architecture

### Directory Structure

```
/
├── .github/
│   ├── actions/                        # Composite GitHub Actions
│   │   └── php-cache-restore/          # PHP environment setup with caching
│   │       └── action.yaml
│   └── workflows/                      # Reusable GitHub Actions workflows
│       └── php-workflow.yml            # Main PHP CI/CD workflow
├── CHANGELOG.md                        # Version history
├── LICENSE                             # MIT License
├── README.md                           # Repository overview
└── documentation/                      # Audience-specific documentation
```

### How Components Connect

1. **Calling repositories** reference the reusable workflow in their own `.github/workflows/` files using `uses: wakeforestuniversity/wfu-github/.github/workflows/php-workflow.yml@v1`
2. The **PHP workflow** orchestrates multiple parallel jobs: Composer setup, validation, testing with coverage, security scanning, and JavaScript linting
3. The **php-cache-restore composite action** is used by each job to set up the PHP environment and restore cached Composer dependencies, avoiding redundant installs

### Reusable PHP Workflow (`php-workflow.yml`)

The workflow is triggered via `workflow_call` and runs the following jobs:

```
workflow_call
├── setup           # Composer install with caching
├── composer        # Validate all environment-specific composer files
├── testing         # PHPUnit + Qlty code coverage
├── security        # Snyk vulnerability scanning
└── js-lint         # JavaScript linting (if package.json with lint:js exists)
```

#### Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `phpVersion` | Yes | — | PHP version to use (e.g., "8.2") |
| `composerFile` | No | `composer.json` | Composer file to use for installation |

#### Secrets

| Secret | Required | Description |
|---|---|---|
| `composerAuth` | Yes | GitHub OAuth token for private Composer repositories |
| `qltyCoverageToken` | Yes | Qlty token for code coverage reporting |
| `securityToken` | Yes | Snyk token for security vulnerability scanning |

#### Jobs

**setup** — Sets up PHP, restores Composer cache, validates composer files, and installs dependencies.

**composer** — Validates all environment-specific composer files (composer-dev.json, composer-uat.json, composer-pprd.json, composer-prod.json) if they exist. This is specific to the WFU WordPress monorepo's ComposerPiler system.

**testing** — Runs PHPUnit tests and reports code coverage to Qlty using the clover format.

**security** — Runs Snyk to check all projects for known vulnerabilities at the lowest severity threshold.

**js-lint** — Checks for `package.json` with a `lint:js` script and runs it if available. Uses Node.js 20.

### PHP Cache Restore Action (`php-cache-restore/action.yaml`)

A composite action that:
1. Sets up the specified PHP version using `shivammathur/setup-php`
2. Enables Xdebug for code coverage
3. Restores cached Composer `vendor/` directory using a cache key based on PHP version and `composer.lock` hash

#### Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `php-version` | Yes | `7.4` | PHP version to set up |

#### Outputs

| Output | Description |
|---|---|
| `cache-hit` | Whether the Composer cache was restored (`true`/`false`) |

## Key Classes and Functions

This repository contains no PHP classes or functions. It consists entirely of GitHub Actions workflow YAML and composite action YAML files.

### Key Workflow Elements

- `workflow_call` trigger with typed inputs and secrets
- `actions/checkout@v3` / `@v4` for repository checkout
- `shivammathur/setup-php@v2` for PHP environment setup
- `actions/cache@v3` for Composer dependency caching
- `andstor/file-existence-action@v2` for conditional file checks
- `qltysh/qlty-action/coverage@v1` for code coverage reporting
- `snyk/actions/php@master` for security scanning
- `actions/setup-node@v4` for Node.js setup in JS linting

## Data Flow

### CI/CD Pipeline Flow

1. A WFU repository pushes a commit or opens a pull request
2. The calling repository's workflow file references `wfu-github`'s reusable PHP workflow
3. The **setup** job installs PHP, restores or creates Composer cache, and installs dependencies
4. In parallel:
   - **composer** validates all environment-specific composer files
   - **testing** runs PHPUnit and uploads coverage to Qlty
   - **security** scans for vulnerabilities with Snyk
   - **js-lint** runs JavaScript linting if applicable
5. Results are reported back to the calling repository's GitHub Actions summary

### Calling Repository Example

```yaml
# In a WFU plugin repo: .github/workflows/ci.yml
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

## Dependencies

### External GitHub Actions

- `actions/checkout@v3` / `@v4` — Repository checkout
- `actions/cache@v3` — Dependency caching
- `actions/setup-node@v4` — Node.js setup
- `shivammathur/setup-php@v2` — PHP environment setup
- `andstor/file-existence-action@v2` — Conditional file existence checks
- `qltysh/qlty-action/coverage@v1` — Qlty code coverage reporting
- `snyk/actions/php@master` — Snyk security scanning

### External Services

- **Qlty** — Code coverage tracking and reporting
- **Snyk** — Security vulnerability scanning
- **GitHub Actions** — CI/CD execution platform

## Development Setup

### Prerequisites

- A GitHub account with access to the `wakeforestuniversity` organization
- Understanding of GitHub Actions workflow syntax
- Familiarity with the WFU WordPress development environment

### Local Development

This repository does not require any local installation. To develop or test changes:

1. Clone the repository:
   ```bash
   git clone git@github.com:wakeforestuniversity/wfu-github.git
   ```

2. Edit workflow or action YAML files

3. Push to a branch and test by referencing that branch in a calling repository:
   ```yaml
   uses: wakeforestuniversity/wfu-github/.github/workflows/php-workflow.yml@your-branch
   ```

4. Once verified, merge to `main` and tag a new version

### Version Tagging

The repository uses version tags (e.g., `v1`) that calling repositories reference. After merging changes to `main`:

```bash
git tag -fa v1 -m "Update v1 tag"
git push origin v1 --force
```

This updates the `v1` tag so all repositories referencing `@v1` automatically get the latest version.

## Testing

This repository has no test suite. Workflows are tested by running them in calling repositories.

To validate changes:
1. Push your changes to a feature branch
2. In a WFU WordPress plugin repo, temporarily change the workflow reference to point to your branch
3. Push a test commit to trigger the workflow
4. Verify all jobs pass successfully
5. Revert the temporary branch reference and merge your changes

## Coding Standards

### YAML Files

- Use 2-space indentation
- Include descriptive `name` properties for all jobs and steps
- Use consistent input naming conventions (camelCase for workflow inputs)
- Quote string values that could be interpreted as other types
- All files must end with a blank line

## Contributing

### Branch Naming

- `main` — Main development branch
- `feature-*` — Feature branches

### PR Process

1. Create a feature branch from `main`
2. Make changes to workflow or action YAML files
3. Test by referencing the branch in a calling repository
4. Open a PR targeting `main`
5. After merge, update the version tag if needed

### Version Bumps

1. Update the `CHANGELOG.md` with the new version and changes
2. After merging, create or update the version tag:
   ```bash
   git tag -fa v1 -m "Update v1 tag"
   git push origin v1 --force
   ```
