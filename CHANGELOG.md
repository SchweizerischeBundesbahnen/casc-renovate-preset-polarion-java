# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- **ARCHITECTURE.md** - Technical documentation
- **CLAUDE.md** - AI assistant guidance
- **CHANGELOG.md** - Version history based on git milestones
- `osvVulnerabilityAlerts: true` - OSV vulnerability database integration
- `vulnerabilityAlerts` section with security labels and 0-day release age
- SBB packages rule (`ch.sbb.*`, `python-sbb-polarion`) - Immediate automerge (0-day wait)
- Major updates rule - Explicit `automerge: false`
- Vulnerability alerts priority rule (`prPriority: 99`)

### Changed
- **README.md** - Complete rewrite with documentation
- Reorganized packageRules order for clarity

### Removed
- `devDependencies` automerge rule (covered by non-major updates rule)

---

## [2.1.0] - 2025-11-26

### Fixed
- `prCreation` setting changed to `status-success`
- Lock file maintenance configuration

---

## [2.0.0] - 2025-11-19

### Added
- OSV vulnerability alerts (`osvVulnerabilityAlerts: true`)
- Lock file maintenance (weekly, Monday before 6am)
- `minimumReleaseAge: 3 days` for stability
- Security label for vulnerability PRs

### Changed
- **BREAKING**: Added 3-day waiting period for all updates (except SBB packages)

---

## [1.4.0] - 2025-09-04

### Added
- Automerge for GitHub Actions updates
- CI improvements for dependency updates

---

## [1.3.0] - 2025-07-08

### Added
- Node.js (npm) support with lock file maintenance
- Additional package rules for Node.js ecosystem

---

## [1.2.0] - 2025-01-09

### Changed
- Pre-commit hooks: always automerge (all update types)
- Simplified renovate configuration

---

## [1.1.0] - 2024-06-25

### Added
- Minor/patch automerge for stable dependencies
- devDependencies automerge
- Self-testing configuration (`renovate.json`)

---

## [1.0.0] - 2024-06-05

### Added
- Initial Renovate preset for Polarion Java repositories
- Maven support with GitHub Packages + Maven Central
- Version filtering (excludes `-SNAPSHOT`, `-jboss-`, `-redhat-`, etc.)
- Semantic commits enabled
- `platformAutomerge` for GitHub
- `config:best-practices` base configuration
- SBB package special handling (`ch.sbb.*`)

---

[Unreleased]: https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java/compare/v2.1.0...HEAD
[2.1.0]: https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java/compare/v2.0.0...v2.1.0
[2.0.0]: https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java/compare/v1.4.0...v2.0.0
[1.4.0]: https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java/compare/v1.3.0...v1.4.0
[1.3.0]: https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java/releases/tag/v1.0.0
