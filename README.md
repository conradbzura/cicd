# Git-Flow and CI/CD Implementation for Python Projects


## Overview of Git-Flow

Git-Flow is a branching model for Git that defines a robust workflow for managing feature development, releases, and hotfixes. It is designed to streamline collaboration and ensure a clean history for software projects. The key "protected" branches in Git-Flow are:

- **`master`:** The stable branch containing production-ready code. Code changes pushed to this branch trigger a release.
- **`main` (conventionally referred to as `develop` in Git-Flow):** The integration branch where features are merged and tested.
- **`release`:** Temporary branch for preparing a new production release. Code changes pushed to this branch trigger a pre-release.

These branches may not be pushed to directly - instead, a pull request targeting one of these branches must be submitted from a patch or feature branch.


### Common Workflow Steps

**Feature development:**

1. Create a feature branch from `main`.
2. Develop the feature and submit a pull request targeting `main` when complete.
3. Merge the pull request once the feature is considered ready.

**Releases:**

1. Create a release branch from `main` using the `Cut release` workflow (triggered manually via GitHub Actions). This workflow will automatically generate a working pull request targeting master.
2. Merge the generated pull request once the release is considered ready. This will trigger a new release to be packaged and distributed as both GitHub and PyPI releases.

**Release patches:**

1. Create a release fix branch from `release`.
2. Fix the issue and submit a pull request targeting `release` when complete.
3. Merge the pull request once the patch is considered ready. This will trigger a new pre-release to be packaged and distributed as both GitHub and PyPI releases.

**Master patches:**

1. Create a hot fix branch from `master`.
2. Fix the issue and submit a pull request targeting `master` when complete.
3. Merge the pull request once the patch is considered ready. This will trigger a new release to be packaged and distributed as both GitHub and PyPI releases.


## How This Library Implements Git-Flow

This repository automates Git-Flow for Python projects using GitHub Actions and custom scripts. Below is an overview of the workflows and scripts that implement Git-Flow:

### Workflows

**[Validate repository:](workflows/validate-repo.yaml)**
   - Ensures required secrets (e.g., `MY_TOKEN`, `PYPI_TOKEN`) are defined. Triggered manually via GitHub Actions.

**[Run tests:](workflows/run-tests.yaml)**
   - Runs tests on pull requests targeting `master`, `main`, or `release` branches across multiple Python versions.

**[Validate pull requests:](workflows/validate-pr.yaml)**
   - Prevents direct merges into `master` unless the source branch is `release`.

**[Label pull requests:](workflows/label-pr.yaml)**
   - Automatically labels pull requests based on their source and target branches.

**[Sync branches:](workflows/sync-branches.yaml)**
   - Synchronizes `master`, `main`, and `release` branches after pull request merges.

**[Cut release:](workflows/cut-release.yaml)**
   - Creates a release branch and tags a release candidate. Triggered manually via GitHub Actions.

**[Publish release:](workflows/publish-release.yaml)**
   - Tags, builds, and publishes releases to GitHub and PyPI.


### Scripts

**[install-tools.sh:](scripts/install-tools.sh)**
   - Installs required tools like Homebrew, GitHub CLI, and Keychain Secrets Manager. This is intended to be used to set up a local development environment.

**[set-secrets.sh:](scripts/set-secrets.sh)**
   - Configures GitHub Actions secrets using values from the keychain or user input. This is intended to be used to set up a local development environment.

**[split-version.sh:](scripts/split-version.sh)**
   - Splits version strings into major, minor, and patch components.

**[bump-version.sh:](scripts/bump-version.sh)**
   - Increments version numbers based on the specified segment (major, minor, patch).

**[cut-release.sh:](scripts/cut-release.sh)**
   - Creates a release branch and tags a release candidate.

**[publish-distribution.sh:](scripts/publish-distribution.sh)**
   - Publishes distribution artifacts to PyPI.


### Custom Actions

**[Add label:](actions/add-label/action.yaml)**
   - Adds labels to pull requests based on their context.

**[Get touched files:](actions/get-touched-files/action.yaml)**
   - Identifies files modified in a pull request.

**[Build release:](actions/build-release/action.yaml)**
   - Builds distribution artifacts for a specific version.

**[Publish GitHub release:](actions/publish-github-release/action.yaml)**
   - Publishes a release to GitHub, including artifact signatures.

**[Publish PyPI release:](actions/publish-pypi-release/action.yaml)**
   - Publishes a release to PyPI using a provided token.


## Installation

Unfortunately, GitHub does not support workflows defined in a Git submodule, so a bit of a workaround is required. To configure a GitHub-hosted codebase to use this library, first fork this repo. Below is the GitHub CLI command to do so:

```shell
gh repo fork https://github.com/conradbzura/cicd.git
```

Next, clone your fork into the `.github` directory in the root of your Git repo:

```shell
git clone https://github.com/<you>/cicd.git .github
```

As you tweak the workflow to suit your own needs, be sure to merge them into your fork for safe-keeping. Feel free to submit a pull request to this repo with any changes you feel others might find useful.
