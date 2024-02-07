# Changelog

## 1.1.0 (2024-02-08)

* Changing from using master branch to v1.0

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
