# casc-renovate-preset-polarion-java

**Renovate configuration preset for Polarion Java repositories on GitHub**

Dependency management for Java (Maven), Node.js (npm), pre-commit hooks, and GitHub Actions with smart automerge and security-first defaults.

---

## Quick Start

Add this to your repository's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java"]
}
```

That's it! Renovate will now:
- Automatically update dependencies across supported ecosystems
- Auto-merge safe updates (minor/patch, dev dependencies, pre-commit hooks)
- Alert on security vulnerabilities with priority PRs
- Maintain lock files weekly

---

## Supported Ecosystems

| Ecosystem | Package Manager | Lock File | Auto-Update |
|-----------|----------------|-----------|-------------|
| **Java** | Maven (pom.xml) | - | Minor/patch (3-day wait) |
| **Node.js** | npm (package.json) | package-lock.json | Minor/patch (3-day wait) |
| **Pre-commit** | .pre-commit-config.yaml | - | All updates |
| **GitHub Actions** | .github/workflows/*.yml | - | All updates |

---

## Features

### Smart Automerge Strategy

**Automatic merging for low-risk updates:**
- Minor/patch updates (1.2.x -> 1.3.0) with **3-day waiting period**
- Pre-commit hooks and GitHub Actions (all updates)
- **Internal SBB packages** (minor/patch, **immediate** - no waiting):
  - `ch.sbb.*` Maven packages
  - `python-sbb-polarion` Python package

**Manual review required for:**
- Major updates (1.x -> 2.x)
- Security vulnerabilities (labeled for priority)

### Security & Vulnerability Management

- **OSV Vulnerability Alerts**: Immediate PRs for dependencies with security issues
- **Priority Handling**: Security PRs created first (`prPriority: 99`)
- **Zero-Delay Fixes**: No waiting period for security updates
- **Security Labels**: All vulnerability PRs labeled `security`
- **Version Filtering**: Blocks unstable/snapshot releases (Maven)

### Registry Configuration

**Maven packages sourced from:**
- GitHub Packages (`maven.pkg.github.com/SchweizerischeBundesbahnen`)
- Maven Central (`repo.maven.apache.org/maven2`)

### Lock File Maintenance

**Automated weekly refresh** (Monday before 6am):
- npm `package-lock.json` - Transitive dependency updates
- Auto-merged (low risk)

### Semantic Commits

All PRs use conventional commit format:
```
chore(deps): update spring-boot to 3.2.1
fix(deps): update log4j to resolve CVE-2024-1234
```

---

## Configuration Examples

### Basic Usage (Most Common)

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java"]
}
```

### Override Automerge for Specific Package

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java"],
  "packageRules": [
    {
      "matchPackageNames": ["spring-boot"],
      "automerge": false
    }
  ]
}
```

### Disable Automerge Entirely

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java"],
  "automerge": false
}
```

### Custom Schedule

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java"],
  "schedule": ["after 10pm every weekday"]
}
```

---

## What You'll See

### Pull Request Examples

**Auto-merged patch update:**
```
chore(deps): update junit to 5.10.1

- Type: patch
- Auto-merge: enabled
- Release age: 5 days
```

**Manual review required (major):**
```
chore(deps): update spring-boot to 4.0.0

- Type: major
- Auto-merge: disabled
- Breaking changes detected
```

**Security alert:**
```
fix(deps): update log4j to 2.17.1

- Type: patch
- Labels: security
- CVE-2024-1234
- Auto-merge: disabled (requires review)
```

---

## Troubleshooting

### No PRs appearing

**Check:**
1. Is `renovate.json` in repository root?
2. Is Renovate GitHub App installed on the repository?
3. Check Renovate logs in the Dependency Dashboard issue

### Too many automerges

**Solution:** Disable automerge for specific packages:
```json
{
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java"],
  "packageRules": [
    {
      "matchPackagePatterns": ["^org.springframework"],
      "automerge": false
    }
  ]
}
```

### Maven snapshots appearing

**This shouldn't happen** - preset filters snapshots. If you see them:
1. Check preset is correctly extended
2. Verify no local override of `allowedVersions`
3. Report issue to preset maintainers

---

## Developer Setup

### Prerequisites

- Node.js (for `renovate-config-validator`)
- Python 3.11+ (for pre-commit)
- jq (for JSON validation)

### Installation

```bash
# Install pre-commit hooks
pip install pre-commit
pre-commit install
```

### Validation

```bash
# JSON syntax check
jq empty default.json

# Renovate schema validation
npx -p renovate renovate-config-validator default.json

# Run all pre-commit hooks
pre-commit run --all-files
```

### Testing Changes

This repository tests itself (dogfooding):

1. **Edit `default.json`**
2. **Validate:** `npx -p renovate renovate-config-validator default.json`
3. **Commit changes** (pre-commit hooks run automatically)
4. **Observe:** Renovate will create PRs in this repo using the new config
5. **Verify:** Check PR behavior before merging to main

---

## Deployment Warning

**CRITICAL:** Changes to `default.json` are **live immediately** when merged to main.

- All consuming repositories see changes on next Renovate run
- No staging environment
- No rollback mechanism

**Best Practices:**
1. Validate locally: `renovate-config-validator default.json`
2. Test with small changes first
3. Monitor consuming repos after deployment

---

## Documentation

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Technical deep-dive, automerge strategy, registry configuration
- **[CHANGELOG.md](./CHANGELOG.md)** - Version history and changes

---

## Comparison: Before vs After

| Feature | Without Preset | With This Preset |
|---------|---------------|------------------|
| Registry config | Manual per repo | Auto-configured |
| Automerge | Generic (may be unsafe) | Risk-based strategy |
| Maven snapshots | Included (unwanted) | Filtered out |
| Lock file maintenance | Manual | Weekly automated |
| Security alerts | Not labeled | Labeled `security` |
| Release stability | Immediate updates | 3-day minimum age |

---

## Contributing

1. **Validate locally** (see Developer Setup)
2. **Test with `renovate.json`** in this repo
3. **Update documentation** for user-facing changes
4. **Run pre-commit:** `pre-commit run --all-files`
5. **Create PR with clear description**

### Questions or Issues?

Open an issue in this GitHub repository.

---

**Maintained by:** SBB Polarion Team
**Last Updated:** 2026-01-09
