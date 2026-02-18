# Admin Guide

## Overview

`wfu-github` provides reusable GitHub Actions workflows and composite actions for the Wake Forest University WordPress development team. It centralizes CI/CD pipeline configuration so that individual plugin and theme repositories do not need to maintain their own complex workflow definitions.

This repository has no WordPress admin interface. It is used by GitHub Actions and configured by developers. This guide is written for the operational user — the developer or team lead who integrates these workflows into WFU repositories.

## Installation

There is no installation step. WFU repositories consume the workflows and actions by referencing them in their own GitHub Actions workflow files.

### Adding the PHP Workflow to a Repository

Create a file at `.github/workflows/ci.yml` in your repository:

```yaml
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

### Required Repository Secrets

The following secrets must be configured in the calling repository (or at the organization level):

| Secret | Description |
|---|---|
| `COMPOSER_AUTH` | GitHub OAuth token for accessing private Composer packages |
| `QLTY_COVERAGE_TOKEN` | Qlty token for code coverage reporting |
| `SNYK_TOKEN` | Snyk token for security vulnerability scanning |

To configure secrets:
1. Go to your repository on GitHub
2. Navigate to **Settings > Secrets and variables > Actions**
3. Add each required secret

## Configuration

### Workflow Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `phpVersion` | Yes | — | PHP version to use (e.g., `'7.4'`, `'8.1'`, `'8.2'`) |
| `composerFile` | No | `composer.json` | Composer file for dependency installation |

### Customizing the Composer File

For repositories that use the ComposerPiler system (like the main WordPress monorepo), you can specify a different composer file:

```yaml
with:
  phpVersion: '8.2'
  composerFile: 'composer-github.json'
```

### JavaScript Linting

The JavaScript linting job runs automatically if your repository has a `package.json` with a `lint:js` script. No additional configuration is needed. If your project does not have JavaScript or does not need linting, the job will skip gracefully.

To enable JavaScript linting in your project:
1. Add a `lint:js` script to your `package.json`:
   ```json
   {
     "scripts": {
       "lint:js": "eslint assets/js/"
     }
   }
   ```
2. Install ESLint and any plugins as dev dependencies

## Usage

### Viewing Workflow Results

After pushing code or opening a PR:
1. Go to the **Actions** tab in your repository on GitHub
2. Click on the latest workflow run
3. View the status of each job (setup, composer, testing, security, js-lint)
4. Click on any job to see detailed logs

### Code Coverage

Test coverage is reported to Qlty after each run. To view coverage:
1. Go to your project's Qlty dashboard
2. Review overall coverage percentage and file-level details
3. Coverage data is uploaded in Clover format

### Security Scanning

Snyk scans run on every workflow execution:
1. Vulnerabilities are flagged in the GitHub Actions logs
2. Issues are reported at the lowest severity threshold
3. Review the **security** job output for any findings

## Troubleshooting

### Workflow Fails on Composer Install

- Check that the `COMPOSER_AUTH` secret is configured and valid
- Verify the specified `composerFile` exists in the repository
- Check that `composer.lock` is committed and up to date

### Cache Not Being Restored

- The cache key includes the PHP version and `composer.lock` hash
- If `composer.lock` changed, a fresh install will occur and a new cache will be created
- Check that `actions/cache@v3` is functioning (see GitHub Actions cache usage dashboard)

### Coverage Not Reporting

- Verify the `QLTY_COVERAGE_TOKEN` secret is correct
- Check that PHPUnit generates a `clover.xml` coverage file
- Review the testing job logs for Qlty upload errors

### Snyk Fails

- Verify the `SNYK_TOKEN` secret is correct
- Check if Snyk is experiencing an outage
- Review vulnerability findings — some may require dependency updates

### JavaScript Linting Skipped

- Verify `package.json` exists in the repository root
- Verify it contains a `lint:js` script under `scripts`
- Check that `package-lock.json` is committed for `npm ci` to work

## FAQ

**Q: Do I need to update my workflow when wfu-github changes?**
A: If you reference `@v1`, you automatically get the latest version of that major tag. Breaking changes will be released under a new major version (e.g., `@v2`).

**Q: Can I add additional jobs alongside the reusable workflow?**
A: Yes. Add additional jobs in your workflow file alongside the `uses:` reference. They will run in parallel unless you specify `needs:` dependencies.

**Q: Why does the workflow validate multiple composer files?**
A: The WFU WordPress monorepo uses the ComposerPiler system to generate environment-specific composer files (dev, uat, pprd, prod). The validation job ensures all of these are syntactically correct.

**Q: Can I use a different PHP version per environment?**
A: The workflow accepts a single PHP version. If you need multiple versions, call the workflow multiple times with different inputs using a matrix strategy in your calling workflow.

**Q: How do I update the shared workflow?**
A: Make changes on a feature branch in the `wfu-github` repository, test by referencing that branch in a calling repo, then merge and update the version tag.
