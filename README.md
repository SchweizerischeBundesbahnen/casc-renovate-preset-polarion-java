# casc-renovate-polarion-java
Renovate configuration for Polarion Java Repos

## How to Use
use `github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java` in extends section of your `renovate.json` file.

## Features

This preset provides the following configuration for Polarion Java projects:

### Security & Vulnerability Management
- **Vulnerability Alerts**: Automatically creates PRs for dependencies with security vulnerabilities (including transitive dependencies)
- **Lock File Maintenance**: Weekly updates (Mondays before 6am) to keep `package-lock.json` up-to-date with latest secure versions

### Dependency Updates
- **Maven**: Updates from SBB GitHub Packages and Maven Central, filtering out non-production versions
- **npm**: Managed through vulnerability alerts and lock file maintenance
- **Pre-commit hooks**: Keeps pre-commit hooks up to date

### Automation
- **Automerge**: Automatically merges non-major updates, dev dependencies, and pre-commit hooks after CI passes
- **Semantic Commits**: Uses conventional commit messages
- **Branch Automerge**: Merges directly to branch after CI passes (no merge commits)

## Developer Setup

An editor capable of editing text file.

### Testing

using pre-commit

## Deployment

The actual main branch is directly used