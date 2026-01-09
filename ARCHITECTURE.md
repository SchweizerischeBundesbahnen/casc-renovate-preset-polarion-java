# Architecture Documentation

**Date:** 2026-01-09
**Renovate Version:** Compatible with v41+ (GitHub App Renovate)

## Overview

This preset provides Renovate configuration for Polarion Java repositories on GitHub (SchweizerischeBundesbahnen organization) supporting:
- **Java** - Maven dependencies
- **Node.js** - npm dependencies (via lock file maintenance)
- **GitHub Actions** - Workflow dependency updates
- **Pre-commit** - Hook version management

**Key Design Principles:**
- Java-focused preset for Polarion repositories
- GitHub Packages + Maven Central for Maven dependencies
- Smart automerge strategy based on risk levels
- Security-first with OSV vulnerability alerts
- Platform automerge using GitHub's native merge capabilities
- Zero-config for consuming repositories

---

## Package Manager Support Matrix

| Ecosystem | Manager | Datasource | Lock File | Automerge Strategy | Registry |
|-----------|---------|-----------|-----------|-------------------|----------|
| Java | `maven` | maven | - (deterministic via pom.xml) | Via global rules (3-day wait) | GitHub Packages + Maven Central |
| Node.js | `npm` | npm | package-lock.json | Via global rules (3-day wait) | Default (npmjs.org) |
| GitHub Actions | `github-actions` | github-tags | - | All updates | GitHub |
| Pre-commit | `pre-commit` | github-tags | - | All updates | GitHub |

### Legend
- Supported and enabled
- (-) = Not applicable

---

## Why We DON'T Use `enabledManagers`

**Decision:** Use package rules instead of `enabledManagers` whitelist.

### Risks of `enabledManagers`

The `enabledManagers` option is a **whitelist** that disables ALL managers not explicitly listed:

```json
{
  "enabledManagers": ["maven"]
  // This DISABLES: github-actions, pre-commit, npm, etc.
}
```

**Problems:**
1. **Silently disables critical managers:**
   - GitHub Actions workflow dependencies won't update
   - pre-commit hooks won't update
   - Any manager not in the list is silently ignored

2. **Requires constant maintenance:**
   - Adding new technology requires preset update
   - Consuming repos can't use other managers without overriding

### Our Approach

**Strategy:** Configure specific rules for Maven, let others inherit from `config:best-practices`.

**Benefits:**
- Consuming repos get sensible defaults for ANY technology
- GitHub Actions updates work automatically
- pre-commit hooks update automatically
- Future-proof for new technologies

---

## Registry Configuration Strategy

### GitHub Packages + Maven Central

Maven dependencies are sourced from two registries:

```json
{
  "registryUrls": [
    "https://maven.pkg.github.com/SchweizerischeBundesbahnen",
    "https://repo.maven.apache.org/maven2"
  ]
}
```

### Why This Configuration?

1. **GitHub Packages First:**
   - Internal SBB packages published to organization's GitHub Packages
   - Requires GitHub authentication (handled by Renovate GitHub App)
   - First registry in list gets priority

2. **Maven Central Fallback:**
   - Standard open-source dependencies
   - No authentication required
   - Comprehensive package availability

3. **No Artifactory:**
   - GitHub environment doesn't require SBB Artifactory proxy
   - Direct access to Maven Central is available
   - Simplifies configuration and reduces dependencies

### Registry URLs by Ecosystem

| Ecosystem | Primary Registry | Fallback Registry |
|-----------|-----------------|-------------------|
| Maven | `maven.pkg.github.com/SchweizerischeBundesbahnen` | `repo.maven.apache.org/maven2` |
| npm | Default (npmjs.org) | - |
| GitHub Actions | GitHub | - |
| Pre-commit | GitHub | - |

---

## Automerge Strategy

### Risk-Based Decision Tree

```
Is it a vulnerability alert?
├─ YES → Manual review required (labeled 'security')
└─ NO → Continue...

Is it pre-commit hooks or GitHub Actions?
├─ YES → Automerge (all updates)
└─ NO → Continue...

Is it a major update?
├─ YES → Manual review required
└─ NO → Continue...

Is it an internal SBB package?
├─ YES → Automerge immediately (0-day wait)
└─ NO → Continue...

Is it minor/patch/pin/digest?
├─ YES → Automerge (wait 3 days)
└─ NO → Manual review
```

### Automerge Rules

#### Internal SBB Packages
```json
{
  "automerge": true,
  "conditions": [
    "matchPackagePatterns: ^ch\\.sbb\\., ^python-sbb-polarion$",
    "matchUpdateTypes: minor, patch",
    "minimumReleaseAge: 0 days"
  ]
}
```

**Rationale:**
- Internal SBB modules requiring immediate deployment
- No waiting period (0 days) for faster rollout
- Minor/patch only (major requires manual review)

**Matches:**
- Maven: `ch.sbb.polarion.*`, `ch.sbb.core.*`, `ch.sbb.utils.*`
- Python: `python-sbb-polarion`

#### Pre-commit Hooks and GitHub Actions
```json
{
  "matchManagers": ["pre-commit", "github-actions"],
  "automerge": true
}
```

**Rationale:**
- Developer tooling and CI configuration
- Low risk to production code
- Quick updates keep tooling current

### Global Fallback Rule
```json
{
  "matchUpdateTypes": ["minor", "patch", "pin", "digest"],
  "automerge": true,
  "minimumReleaseAge": "3 days"
}
```

**Applies to:** Any dependency not covered by specific rules.

**Rationale:**
- 3-day waiting period for community validation
- CI/CD tests provide safety net for all updates
- Catches breaking changes before merge

---

## Version Filtering

### Maven Forbidden Versions

```json
{
  "allowedVersions": "!/-jboss-|-redhat-|redhat-|-jenkins-|-patch-|-NODEP$|-jbossorg-|-SNAPSHOT$|-PFD-|-jbossas-|-does-not-exist|-tc$|-jahia1$/"
}
```

**Filters out:**
- `-jboss-`, `-redhat-` - Vendor-specific forks
- `-jenkins-` - CI-specific builds
- `-patch-` - Unofficial patches
- `-NODEP$` - No-dependency variants
- `-jbossorg-`, `-jbossas-` - JBoss variants
- `-SNAPSHOT$` - Unstable snapshots
- `-PFD-` - Pre-Final Draft versions
- `-does-not-exist` - Placeholder versions
- `-tc$` - Tomcat variants
- `-jahia1$` - Jahia-specific variants

**Why:**
- Avoid vendor lock-in and proprietary forks
- Prefer official upstream releases
- Prevent unstable snapshot pollution
- Match Polarion's tested dependency matrix

---

## Lock File Maintenance

### Configuration
```json
{
  "lockFileMaintenance": {
    "enabled": true,
    "automerge": true,
    "schedule": ["before 6am on monday"],
    "minimumReleaseAge": null
  }
}
```

### Affected Managers
- **npm** - Updates `package-lock.json` transitive dependencies

### Benefits
1. **Security:** Transitive dependencies get security patches
2. **Stability:** Controlled weekly schedule (not on-demand)
3. **CI-friendly:** Runs during off-hours (before 6am Monday)
4. **Automated:** No manual intervention needed

### Not Applicable To
- **maven** - No lock file (deterministic resolution via pom.xml)

---

## Vulnerability Alerts

### Configuration
```json
{
  "osvVulnerabilityAlerts": true,
  "vulnerabilityAlerts": {
    "enabled": true,
    "labels": ["security"],
    "automerge": false,
    "minimumReleaseAge": "0 days"
  },
  "packageRules": [
    {
      "description": "Vulnerability alerts get highest priority",
      "matchCategories": ["security"],
      "prPriority": 99
    }
  ]
}
```

### OSV Vulnerability Alerts

**`osvVulnerabilityAlerts: true`** enables integration with [OSV.dev](https://osv.dev), an open-source vulnerability database maintained by Google.

**Benefits:**
- Covers vulnerabilities from multiple sources (CVE, GHSA, ecosystem-specific advisories)
- Creates PRs with fixes when available for direct dependencies
- Works alongside GitHub's native Dependabot alerts

### Vulnerability Alert Settings

| Setting | Value | Purpose |
|---------|-------|---------|
| `enabled` | `true` | Activate vulnerability scanning |
| `labels` | `["security"]` | Tag PRs for easy filtering |
| `automerge` | `false` | Require manual review for security fixes |
| `minimumReleaseAge` | `"0 days"` | Bypass 3-day wait for urgent fixes |
| `prPriority` | `99` | Create security PRs before all other updates |

### Behavior
1. **Detection:** Renovate checks dependencies against OSV database
2. **PR Creation:** Immediate PR for vulnerable dependencies
3. **Labeling:** Marked with `security` label for prioritization
4. **Priority:** Created before all other PRs (`prPriority: 99`)
5. **Manual Review:** Never automerged (requires human verification)

---

## Platform-Specific Configuration

### GitHub Features

**Enabled Features:**
- `platformAutomerge: true` - Uses GitHub's native automerge
- `prCreation: "status-success"` - Create PR only after branch tests pass
- `rebaseWhen: "behind-base-branch"` - Auto-rebase on base changes

**Configuration:**
```json
{
  "automergeType": "branch",
  "platformAutomerge": true,
  "prCreation": "status-success",
  "rebaseWhen": "behind-base-branch"
}
```

**Benefits:**
- Native GitHub merge queue integration
- Reduces PR noise (branches merge directly when automerge enabled)
- Tests run before PR creation, not after

---

## Future Enhancements

### 1. Grouping Related Dependencies

**Potential Groups:**
```json
{
  "packageRules": [
    {
      "groupName": "Spring Framework",
      "matchPackagePatterns": ["^org\\.springframework:"]
    },
    {
      "groupName": "Testing frameworks",
      "matchPackagePatterns": ["junit", "mockito", "assertj"]
    }
  ]
}
```

**Benefits:**
- Fewer PRs (related deps in one update)
- Easier review (see full ecosystem update)
- Prevents incompatible version combinations

### 2. Semantic Versioning Customization

**Enhancement:** Customize commit message format:
```json
{
  "semanticCommits": "enabled",
  "commitMessagePrefix": "chore(deps):",
  "commitMessageAction": "update",
  "commitMessageTopic": "{{depName}}",
  "commitMessageExtra": "to {{newValue}}"
}
```

**Result:** `chore(deps): update spring-boot to 3.2.1`

---

## Comparison: With vs Without This Preset

### Without Preset (Default Renovate)

```json
{
  "extends": ["config:recommended"]
}
```

**Behavior:**
- Detects all package managers
- No custom registry configuration
- Generic automerge (may be too aggressive or conservative)
- No Maven version filtering (gets -SNAPSHOT, -jboss-, etc.)
- Generic commit messages

### With This Preset

```json
{
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java"]
}
```

**Behavior:**
- Detects all package managers
- GitHub Packages + Maven Central for Maven
- Risk-based automerge (patches, dev deps, stable versions)
- Maven version filtering (excludes snapshots/forks)
- Semantic commits enabled
- Weekly lock file maintenance
- Vulnerability alerts labeled
- 3-day minimum release age for stability

**Key Differences:**
| Feature | Default | With Preset |
|---------|---------|-------------|
| Maven registry | Maven Central only | GitHub Packages + Maven Central |
| Maven SNAPSHOT filtering | Included | Excluded |
| Automerge strategy | Generic | Risk-based |
| Release stabilization | None | 3-day minimum age |
| Lock file maintenance | Manual | Automated weekly |
| Security PRs | Not labeled | Labeled `security` |

---

## Configuration Validation

### JSON Syntax Check
```bash
jq empty default.json
```

**Expected Output:** (silent = success)

### Renovate Config Validation
```bash
npx -p renovate renovate-config-validator default.json
```

**Expected Output:**
```
INFO: Validating default.json
INFO: Config validated successfully
```

### Pre-commit Hooks
```bash
pre-commit run --all-files
```

**Hooks:**
- JSON schema validation
- YAML validation
- Secret scanning (gitleaks)

---

## Deployment Model

**CRITICAL:** This preset is **live production configuration**.

### Deployment Flow
```
Developer commits to main
    ↓
Git push
    ↓
IMMEDIATELY ACTIVE
    ↓
All consuming repos see changes on next Renovate run
```

### Risk Mitigation
1. **Validate locally BEFORE pushing:**
   ```bash
   npx -p renovate renovate-config-validator default.json
   ```

2. **Test with `renovate.json` in this repo:**
   - This repo consumes its own preset
   - Renovate will create PRs if config is valid
   - Observe behavior before affecting other repos

3. **Monitor first few consuming repo updates:**
   - Check PR quality
   - Verify automerge behavior
   - Watch for unexpected updates

### Consuming Repositories

**How repos use this preset:**
```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java"]
}
```

**When do they see changes?**
- Next scheduled Renovate run (typically daily)
- Renovate fetches latest `default.json` from main branch
- No manual intervention needed in consuming repos

---

## References

- **Renovate Documentation:** https://docs.renovatebot.com
- **Maven Manager:** https://docs.renovatebot.com/modules/manager/maven/
- **npm Manager:** https://docs.renovatebot.com/modules/manager/npm/
- **Package Rules:** https://docs.renovatebot.com/configuration-options/#packagerules
- **Automerge:** https://docs.renovatebot.com/key-concepts/automerge/
- **GitHub Platform:** https://docs.renovatebot.com/modules/platform/github/

---

**Last Updated:** 2026-01-09
**Maintained By:** SBB Polarion Team
