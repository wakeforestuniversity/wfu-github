# Changelog

## 0.2.0 (@022-05-13)

* Adding ability to set composer file to use for PHP workflow.
  * WordPress has plugins that are loaded from an S3 bucket that GitHub doesn't have access to.
  * Allowing passing a specific composer file allowed us to make a specific composer file for WP that excludes those plugins.

## 0.1.0 (2022-03-21)

* Initial working version of PHP workflow that includes: Composer, PHPUnit and Code Climate.
`