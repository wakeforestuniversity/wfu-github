# WFU GitHub Workflows & Actions

These are workflows and actions created by Wake Forest University.

## Using the workflows

Each repository keeps small caller stubs and lets these workflows do the work, so a change made here reaches every repository that references the same tag. Repositories reference `@v2`. The `v1` tag is kept for repositories that have not moved yet.

### PHP checks

`.github/workflows/php.yml` in the consuming repository:

```yaml
name: PHP Workflow

on:
  push:
    branches: [main]
  pull_request:

jobs:
  php-workflow:
    uses: wakeforestuniversity/wfu-github/.github/workflows/php-workflow.yml@v2
    with:
      phpVersion: ${{ vars.WP_PHP_VERSION }}
    secrets:
      composerAuth: ${{ secrets.COMPOSER_AUTH }}
      qltyCoverageToken: ${{ secrets.QLTY_COVERAGE_TOKEN }}
      securityToken: ${{ secrets.SNYK_TOKEN }}
```

Pass `composerFile` when the repository installs from something other than `composer.json`.

### Claude standards review

`.github/workflows/claude-review.yml` in the consuming repository:

```yaml
name: Claude Standards Review

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review, labeled, unlabeled]

jobs:
  review:
    uses: wakeforestuniversity/wfu-github/.github/workflows/claude-review.yml@v2
    permissions:
      contents: read
      checks: read
      pull-requests: write
      issues: write
      id-token: write
    secrets:
      claudeKey: ${{ secrets.UMC_CLAUDE_GITHUB_APP }}
```

The check is named `review / Claude Standards Review`, which is the name to require in branch protection. It checks WFU style and code-quality rules only, never whether the code works. Drafts and Dependabot pull requests are skipped. A person can pass the check by adding the `standards-override` label when a finding is wrong; anyone who can label pull requests can do this, so treat the label as a reviewed decision. The label covers only the commit it was added on: the next push removes it and reviews the new commit. While the label is on, the last review stays folded inside the comment, and removing the label puts it back without running a new review. Adding or removing any other label, reopening the pull request, or a second event for the same commit keeps the last result rather than running a new review.

The first review reads the whole pull request. Each later push is reviewed against the state the last comment saved: findings already reported are checked again, and new findings come only from what changed since. A new finding in a file the last round already reviewed, with no change since, is shown as advisory so it cannot fail a check that already passed that code. A force push or rebase starts over with a full review. From round three on, a failing comment suggests stepping back from the approach.

### Claude mentions

`.github/workflows/claude.yml` in the consuming repository:

```yaml
name: Claude

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  pull_request_review:
    types: [submitted]
  issues:
    types: [opened, assigned]

jobs:
  claude:
    uses: wakeforestuniversity/wfu-github/.github/workflows/claude-mention.yml@v2
    permissions:
      contents: read
      pull-requests: read
      issues: read
      id-token: write
    secrets:
      claudeKey: ${{ secrets.UMC_CLAUDE_GITHUB_APP }}
```

## Releasing

Merging to `main` changes nothing for consumers. The release step is moving the major tag: `git tag -f v2 <sha> && git push -f origin v2`, after tagging the exact version.
