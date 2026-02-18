# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains reusable GitHub Actions workflows and composite actions for the Wake Forest University WordPress development team. It centralizes CI/CD pipeline configuration used by WFU WordPress plugin and theme repositories.

## Key Components

### Reusable PHP Workflow (`.github/workflows/php-workflow.yml`)

A `workflow_call` workflow that runs:
- **Composer Setup** — PHP setup, dependency caching, and composer install
- **Composer Validation** — Validates environment-specific composer files (dev, uat, pprd, prod)
- **Testing** — PHPUnit with Qlty code coverage reporting
- **Security** — Snyk vulnerability scanning
- **JavaScript Linting** — Optional JS linting if `package.json` has a `lint:js` script

### PHP Cache Restore Action (`.github/actions/php-cache-restore/action.yaml`)

A composite action that sets up PHP with Xdebug and restores cached Composer dependencies.

## Usage

Calling repositories reference this repo's workflows:
```yaml
uses: wakeforestuniversity/wfu-github/.github/workflows/php-workflow.yml@v1
```

## Version Tagging

After changes to `main`, update the version tag:
```bash
git tag -fa v1 -m "Update v1 tag"
git push origin v1 --force
```

## Documentation Maintenance

This repository maintains audience-specific documentation in the `documentation/` directory:

- `DEVELOPER_GUIDE.md` — Technical reference for developers
- `ADMIN_GUIDE.md` — Usage guide for WordPress admins / operational users
- `USER_GUIDE.md` — Feature guide for frontend visitors

**PR Requirement:** Every pull request that changes functionality, adds features, fixes bugs, or modifies configuration must include corresponding updates to the relevant documentation files. Reviewers should verify documentation accuracy as part of the PR review process.
