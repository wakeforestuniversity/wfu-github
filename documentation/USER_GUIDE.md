# User Guide

## Overview

`wfu-github` is a CI/CD infrastructure repository that has no frontend-facing output. It provides reusable GitHub Actions workflows and composite actions used by the WFU development team to automate code quality checks, testing, and security scanning for WordPress plugins and themes.

Website visitors do not interact with this repository in any way.

## Indirect Impact

While this repository produces no frontend features, it contributes to the quality and security of WFU websites:

- **Automated Testing** — Every code change to WFU WordPress plugins and themes runs through PHPUnit tests before deployment, catching bugs before they reach production
- **Security Scanning** — Snyk vulnerability scanning identifies known security issues in dependencies, helping protect website visitors from exploitable vulnerabilities
- **Code Coverage** — Coverage reporting ensures that code is thoroughly tested, reducing the risk of regressions that could affect the website experience
- **Consistent Quality** — By centralizing CI/CD configuration, all WFU repositories follow the same quality standards, leading to more reliable and maintainable websites

## Features

This repository provides no visitor-facing features. All functionality is developer-facing and operates within the GitHub Actions CI/CD platform.

## Accessibility

Not applicable — this repository produces no frontend output.

## Browser and Device Support

Not applicable — this repository produces no frontend output.
