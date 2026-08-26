# Changelog

## 1.4.1 (2026-08-26)

* Guarding the PHPUnit and PHPCS steps on the tool actually being installed, not just on its configuration file being present. `app-web-aws-wp` ships a `phpcs.xml` but does not require `squizlabs/php_codesniffer`, so the new standards job died with `vendor/bin/phpcs: No such file or directory` and exit 127 rather than skipping. A config file is evidence of intent, not of an installed binary, and the two come apart more often than expected across the WFU repositories.

## 1.4.0 (2026-08-26)

* Adding a "Coding Standards (PHPCS)" job. Every WFU PHP repository ships a `phpcs.xml` and the team treats PHPCS as the standard, but nothing has ever run it automatically: standards compliance has depended entirely on a developer remembering to run it locally. Two errors reached an approved, green pull request this week and were caught only by hand.
* Failing on errors while reporting warnings without failing, via `--runtime-set ignore_warnings_on_exit 1`. A sweep of 47 WFU repositories found warnings to be largely TODO comments and advisory notices; gating merges on those would block unrelated work on cleanup nobody has scheduled. Errors are the narrower, actionable set, and `phpcbf` fixes most of them.
* Skipping cleanly where a repository has no `phpcs.xml`, matching how the PHPUnit step behaves, so the job is safe to adopt everywhere at once.

## 1.3.0 (2026-08-26)

* Running PHPUnit in the "Testing and Code Coverage" job. The job had no PHPUnit step: it restored the Composer cache and then handed `clover.xml` to the Qlty action, but nothing in the job ever produced that file. Every consuming repository has therefore been reporting a green testing check while running zero tests, in some cases for its entire history. The step defers to each repository's own `phpunit.xml`, which is the only configuration able to bootstrap that repository's tests, and skips cleanly where no suite exists so the change is safe to adopt everywhere at once.
* Reading the Qlty coverage token from `secrets` rather than `inputs`. `qltyCoverageToken` is declared under `secrets` in `workflow_call`, so `inputs.qltyCoverageToken` resolved to an empty string and every coverage upload has been unauthenticated.
* Uploading coverage only when `clover.xml` exists, so a repository with no suite does not fail on a missing file.

## 1.2.0 (2025-06-26)

* Switching from using Code Climate to Qlty

## 1.1.0 (2024-02-08)

* Changing from using master branch to v1.0

## 1.0.0 (2024-02-07)

* Updated to the currently active version

## 0.3.2 (2023-03-17)

* Updating `file-existence` action to v2 so its no longer using Node 12.

## 0.3.1 (2023-03-17)

* PHP8 only supports xdebug not xdebug2 so fixing that.

## 0.3.0 (2023-03-17)

* Updating action versions to newest so workflow is running on Node 16 not Node 12.

## 0.2.0 (2022-05-13)

* Adding ability to set composer file to use for PHP workflow.
  * WordPress has plugins that are loaded from an S3 bucket that GitHub doesn't have access to.
  * Allowing passing a specific composer file allowed us to make a specific composer file for WP that excludes those plugins.

## 0.1.0 (2022-03-21)

* Initial working version of PHP workflow that includes: Composer, PHPUnit and Code Climate.
`
