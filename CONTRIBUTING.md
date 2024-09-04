# Contributing to `dbt-package-testing`

`dbt-package-testing` is a template for package testing at dbt Labs.


1. [About this document](#about-this-document)
2. [Getting the code](#getting-the-code)
3. [Setting up an environment](#setting-up-an-environment)
4. [Running in development](#running-dbt-package-testing-in-development)
5. [Testing](#testing)
6. [Debugging](#debugging)
7. [Adding or modifying a changelog entry](#adding-or-modifying-a-changelog-entry)
8. [Submitting a Pull Request](#submitting-a-pull-request)
9. [Troubleshooting Tips](#troubleshooting-tips)

## About this document

There are many ways to contribute to the ongoing development of `dbt-package-testing`, such as by participating in discussions and issues. We encourage you to first read our higher-level document: ["Expectations for Open Source Contributors"](https://docs.getdbt.com/docs/contributing/oss-expectations).

The rest of this document serves as a more granular guide for contributing code changes to `dbt-package-testing` (this repository). It is not intended as a guide for using `dbt-package-testing`, and some pieces assume a level of familiarity with Python development (virtualenvs, `pip`, etc). Specific code snippets in this guide assume you are using macOS or Linux and are comfortable with the command line.

If you get stuck, we're happy to help! Drop us a line in the `#dbt-package-testing-development` channel in the [dbt Community Slack](https://community.getdbt.com).

### Notes

- **CLA:** Please note that anyone contributing code to `dbt-package-testing` must sign the [Contributor License Agreement](https://docs.getdbt.com/docs/contributor-license-agreements). If you are unable to sign the CLA, the `dbt-package-testing` maintainers will unfortunately be unable to merge any of your Pull Requests. We welcome you to participate in discussions, open issues, and comment on existing ones.
- **Branches:** All pull requests from community contributors should target the `main` branch (default).
- **Releases**: This repository is never released.

## Getting the code

### Installing git

You will need `git` in order to download and modify the source code.

### External contributors

If you are not a member of the `dbt-labs` GitHub organization, you can contribute to `dbt-package-testing` by forking the `dbt-package-testing` repository. For a detailed overview on forking, check out the [GitHub docs on forking](https://help.github.com/en/articles/fork-a-repo). In short, you will need to:

1. Fork the `dbt-package-testing` repository
2. Clone your fork locally
3. Check out a new branch for your proposed changes
4. Push changes to your fork
5. Open a pull request against `dbt-labs/dbt-package-testing` from your forked repository

### dbt Labs contributors

If you are a member of the `dbt-labs` GitHub organization, you will have push access to the `dbt-package-testing` repo. Rather than forking `dbt-package-testing` to make your changes, just clone the repository, check out a new branch, and push directly to that branch.

## Setting up an environment

This repo serves as a template and is no intedned to be run.

### Initial setup

None needed.

### Test commands

No tests included.

## Submitting a Pull Request

Code can be merged into the current development branch `main` by opening a pull request. A `dbt-package-testing` maintainer will review your PR. They may suggest code revision for style or clarity, or request that you add unit or integration test(s). These are good things! We believe that, with a little bit of help, anyone can contribute high-quality code.

Automated tests run via GitHub Actions. If you're a first-time contributor, all tests (including code checks and unit tests) will require a maintainer to approve. Changes in the `dbt-package-testing` repository trigger integration tests against Postgres. dbt Labs also provides CI environments in which to test changes to other adapters, triggered by PRs in those adapters' repositories, as well as periodic maintenance checks of each adapter in concert with the latest `dbt-package-testing` code changes.

Once all tests are passing and your PR has been approved, a `dbt-package-testing` maintainer will merge your changes into the active development branch. And that's it! Happy developing :tada:

## Troubleshooting Tips
- Sometimes, the content license agreement auto-check bot doesn't find a user's entry in its roster. If you need to force a rerun, add `@cla-bot check` in a comment on the pull request.
