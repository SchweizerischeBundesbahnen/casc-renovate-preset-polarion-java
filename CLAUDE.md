# Claude Code Development Guide

This file provides guidance for AI assistants (like Claude Code) working on this repository.

## Repository Purpose

**casc-renovate-preset-polarion-java** is a shared Renovate configuration preset for GitHub repositories supporting:
- Java (Maven)
- Node.js (npm)
- Pre-commit hooks
- GitHub Actions

**Target Platform:** GitHub SchweizerischeBundesbahnen

## Repository Architecture

```
casc-renovate-preset-polarion-java/
├── default.json              # Main preset configuration (PRODUCTION)
├── renovate.json            # Self-test configuration (dogfooding)
├── ARCHITECTURE.md          # Technical documentation
├── README.md                # User-facing documentation
├── CLAUDE.md                # This file (AI guidance)
├── CHANGELOG.md             # Version history
└── .pre-commit-config.yaml  # Validation hooks
```

### File Purposes

**default.json**
- **PRODUCTION configuration** - Live immediately on merge to main
- Contains all package manager configurations
- Automerge rules and registry settings
- **CRITICAL:** Validate thoroughly before changes

**renovate.json**
- Self-testing configuration
- Extends `github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java` (dogfooding)
- Used to verify preset behavior

**ARCHITECTURE.md**
- Technical deep-dive for developers and power users
- Package manager support matrix
- Automerge strategy decision trees
- Registry configuration rationale
- Future enhancements roadmap

**README.md**
- User-facing quick-start guide
- How to consume the preset
- Feature highlights
- Troubleshooting

## Development Workflow

### 1. Validation Commands

**Always run these before committing:**

```bash
# JSON syntax check
jq empty default.json

# Renovate schema validation
npx -p renovate renovate-config-validator default.json

# Pre-commit hooks (includes above + gitleaks)
pre-commit run --all-files
```

### 2. Testing Strategy

**Self-Testing:**
- This repository extends its own preset via `renovate.json`
- Renovate will create PRs if configuration is valid
- Observe PR behavior before pushing to main

**Consuming Repository Testing:**
- Identify a low-risk test repository
- Monitor first Renovate run after changes
- Verify automerge behavior and PR quality

### 3. Deployment Model

**CRITICAL:** No CI/CD pipeline - changes are **live immediately** on push to main.

**Deployment Flow:**
```
git push origin main
    ↓
LIVE IN PRODUCTION
    ↓
All consuming repos see changes on next Renovate run
```

**Risk Mitigation:**
1. Validate locally with `renovate-config-validator`
2. Test with `renovate.json` in this repo first
3. Push small, incremental changes
4. Monitor consuming repos after deployment

## Configuration Philosophy

### Multi-Ecosystem Support

**Approach:** Configure specific rules for each ecosystem, inherit defaults for others.

**Why NOT `enabledManagers`:**
- Disables ALL other managers (github-actions, dockerfile, etc.)
- Requires constant maintenance for new technologies
- Breaks consuming repos using other managers

**Our Strategy:**
- Explicit package rules for: maven
- Inherit sensible defaults for: npm, github-actions, pre-commit, etc.
- Future-proof for new technologies

See `ARCHITECTURE.md` for detailed rationale.

### Automerge Strategy

**Risk-Based Decision Making:**

```
HIGH RISK (Manual review):
- Major updates
- Vulnerability alerts

MEDIUM RISK (Timed automerge):
- Minor/patch updates for stable versions
- 3-day minimum release age

LOW RISK (Immediate automerge):
- Internal SBB packages (ch.sbb.*)
- Pre-commit hooks
- GitHub Actions
```

**Rationale:** Balance automation with safety. See `ARCHITECTURE.md` for decision tree.

### Registry Configuration

**Pattern:** GitHub Packages + Maven Central for Maven dependencies.

**Why This Configuration?**
1. GitHub Packages first for internal SBB packages
2. Maven Central fallback for open-source dependencies
3. No Artifactory required in GitHub environment

**Example:**
```json
{
  "registryUrls": [
    "https://maven.pkg.github.com/SchweizerischeBundesbahnen",
    "https://repo.maven.apache.org/maven2"
  ]
}
```

## Critical Constraints

### Security and Privacy

**NEVER expose in documentation, code, or PRs:**
- Internal SBB URLs (except bin.sbb.ch which may be in historical config)
- Internal email addresses (use example.com)
- Internal project identifiers
- Employee names or usernames
- Infrastructure details

**Allowed:**
- `maven.pkg.github.com/SchweizerischeBundesbahnen` (GitHub Packages)
- `repo.maven.apache.org/maven2` (Maven Central)
- Generic examples (PROJECT/repo, user@example.com)

### Semantic Commits

**Enabled via:** `:semanticCommits` preset (inherited from `config:best-practices`)

**Format:** `type(scope): description`

**Examples:**
```
chore(deps): update spring-boot to 3.2.1
fix(deps): update testcontainers to resolve CVE-2024-1234
feat: add uv package manager support
```

**Enforced by:** Renovate (automated) + pre-commit hooks (manual commits)

### GitHub Platform Features

**Automerge:**
- `automergeType: "branch"` (merge without PR when automerge=true)
- `platformAutomerge: true` (uses GitHub's native automerge)

**PR Creation:**
- `prCreation: "status-success"` (wait for branch tests before PR)
- `rebaseWhen: "behind-base-branch"` (auto-rebase on base changes)

## Package Manager Details

### Maven (Java)

**Configuration Highlights:**
- GitHub Packages + Maven Central registries
- Extensive version filtering (no -SNAPSHOT, -jboss-, etc.)
- Disabled provided/runtime deps (except SBB packages)
- Automerge: minor/patch updates (3-day wait), SBB packages (immediate)

**Version Filter Rationale:**
- Polarion has tested dependency matrix
- Avoid vendor forks and proprietary variants
- Filter snapshots and pre-releases

### npm (Node.js)

**Configuration Highlights:**
- Uses default npmjs.org registry
- Lock file maintenance (Monday before 6am)
- Automerge via global rules (minor/patch, 3-day wait)

### Pre-commit and GitHub Actions

**Configuration Highlights:**
- Automerge all updates
- Low risk (developer tooling and CI config)
- Quick updates keep tooling current

## Common Tasks

### Adding a New Package Manager

1. **Research with Context7:**
   ```
   Ask Claude Code to research the manager via Context7
   ```

2. **Add package rule to `default.json`:**
   ```json
   {
     "description": "NewManager - Description",
     "matchManagers": ["new-manager"],
     "registryUrls": ["https://..."]
   }
   ```

3. **Add automerge rule (if applicable):**
   ```json
   {
     "description": "NewManager - Automerge strategy",
     "matchManagers": ["new-manager"],
     "matchUpdateTypes": ["patch"],
     "automerge": true
   }
   ```

4. **Update documentation:**
   - Add row to support matrix in `ARCHITECTURE.md`
   - Add section to README.md features
   - Update `CLAUDE.md` (this file)

5. **Validate and test:**
   ```bash
   npx -p renovate renovate-config-validator default.json
   pre-commit run --all-files
   ```

### Modifying Automerge Rules

**Before changing automerge rules:**

1. **Consider impact:**
   - Will this automerge MORE or FEWER updates?
   - What's the risk level of the changes?
   - Could this break consuming repositories?

2. **Document rationale in PR description:**
   - Why is the change needed?
   - What problem does it solve?
   - What's the risk assessment?

3. **Test with self-test configuration:**
   - Observe `renovate.json` PRs in this repo
   - Verify automerge behavior

4. **Update `ARCHITECTURE.md`:**
   - Update decision tree
   - Update "Automerge Strategy" section
   - Add to "Comparison" table

### Updating Registry URLs

**When to update:**
- GitHub Packages URL changes
- New internal registries added
- Registry restructuring

**Process:**

1. **Verify new URL works:**
   ```bash
   curl -I <new-registry-url>
   ```

2. **Update ALL occurrences in `default.json`:**
   - Search for old URL
   - Replace with new URL

3. **Update documentation:**
   - Update `ARCHITECTURE.md` "Registry URLs by Ecosystem" table
   - Update README.md if user-visible

4. **Validate and test:**
   - Run `renovate-config-validator`
   - Test with pilot repository

## Related Repositories

**Python-Specific Preset:**
- `github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion-python`
- Single-ecosystem preset for Python projects
- Reference implementation for documentation structure

**Relationship:**
- This preset: Java-focused for Polarion Java repos
- Python preset: Specialized for Python-only projects
- Both use similar documentation patterns

## Documentation Standards

### ARCHITECTURE.md (Technical)

**Target Audience:** Developers, power users, future maintainers

**Content:**
- Deep technical analysis
- Decision rationales
- Comparison tables
- Future roadmap
- Configuration patterns

**Tone:** Detailed, technical, explanatory

### README.md (User-Facing)

**Target Audience:** Consuming repository maintainers

**Content:**
- Quick-start guide
- Feature highlights
- Usage examples
- Troubleshooting

**Tone:** Concise, practical, example-driven

### CLAUDE.md (AI Guidance)

**Target Audience:** AI assistants (Claude Code, GitHub Copilot, etc.)

**Content:**
- Repository purpose and architecture
- Development commands
- Configuration philosophy
- Critical constraints
- Common tasks

**Tone:** Directive, procedural, context-rich

## Pre-Commit Hooks

**Hooks Configured:**

1. **check-json** - JSON syntax validation
2. **renovate-config-validator** - Renovate schema validation
3. **gitleaks** - Secret scanning

**When to run:**
```bash
# Before every commit (automatic)
pre-commit install  # One-time setup

# Manual run
pre-commit run --all-files
```

**Why gitleaks?**
- Prevent accidental secret commits
- Catch hardcoded tokens, passwords
- Compliance requirement

## References

**Renovate Documentation:**
- Main docs: https://docs.renovatebot.com
- GitHub platform: https://docs.renovatebot.com/modules/platform/github/
- Package rules: https://docs.renovatebot.com/configuration-options/#packagerules
- Automerge: https://docs.renovatebot.com/key-concepts/automerge/

**GitHub Packages:**
- Maven: https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry

---

**Last Updated:** 2026-01-09
**Maintainer:** SBB Polarion Team
**Questions?** Open an issue in this GitHub repository
