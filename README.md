# casc-renovate-preset-polarion-java

**Renovate preset for Polarion Java repositories on GitHub**

Extends [casc-renovate-preset-polarion](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion) (shared base) and adds Maven-specific configuration.

---

## Quick Start

Add this to your repository's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java"]
}
```

---

## Supported Ecosystems

| Ecosystem | Package Manager | Lock File | Auto-Update |
|-----------|----------------|-----------|-------------|
| **Java** | Maven (pom.xml) | — | Minor/patch (3-day wait) |
| **Node.js** | npm (package.json) | package-lock.json | Minor/patch (3-day wait) |
| **Pre-commit** | .pre-commit-config.yaml | — | All updates incl. major (grouped) |
| **GitHub Actions** | .github/workflows/*.yml | — | All updates incl. major (grouped) |

**All managers are enabled** — unlike the Python/Docker presets, no `enabledManagers` whitelist is used. Maven-specific rules are configured explicitly while all other managers inherit sensible defaults, keeping the preset future-proof.

---

## Automerge Strategy

Inherited from the [shared base preset](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion). See base preset for full details.

| Update Type | Automerged? | Stabilization | Notes |
|-------------|-------------|---------------|-------|
| Minor/patch | ✅ Yes | 3 days | Requires CI to pass |
| Major | ❌ No | — | Manual review required |
| GitHub Actions (any incl. major) | ✅ Yes | 3 days | Grouped into one branch/PR |
| Pre-commit hooks (any incl. major) | ✅ Yes | 3 days | Grouped into one branch/PR, ignoreTests: true |
| SBB internal packages (minor/patch) | ✅ Yes | 0 days | `ch.sbb.*` and `python-sbb-polarion` |
| Security vulnerabilities | ❌ No | 0 days | Labeled `security`, priority queue |
| Lock file maintenance | ✅ Yes | — | Monday before 4am |

---

## Maven Configuration

### Registries

Maven packages are sourced from:
- GitHub Packages (`maven.pkg.github.com/SchweizerischeBundesbahnen`) — internal SBB packages
- Maven Central (`repo.maven.apache.org/maven2`) — open-source dependencies

### Version Filtering

Unstable and vendor-specific versions are blocked:

| Pattern | Example | Reason |
|---------|---------|--------|
| `-SNAPSHOT$` | `1.0-SNAPSHOT` | Unstable builds |
| `-redhat-`, `-jboss-` | `1.0-redhat-1` | Vendor forks |
| `-jenkins-` | `1.0-jenkins-1` | CI-specific builds |
| `-atlassian-` | `1.0-atlassian-1` | Vendor forks |
| `-tc$` | `1.0-tc` | Tomcat variants |
| `-NODEP$` | `1.0-NODEP` | No-dependency variants |

### Provided/Runtime Scope

`provided` and `runtime` scope dependencies are disabled by default (except `ch.sbb.*` packages) to avoid unwanted updates to container-provided dependencies.

---

## Configuration Examples

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

### Custom Schedule

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>SchweizerischeBundesbahnen/casc-renovate-preset-polarion-java"],
  "schedule": ["after 10pm every weekday"]
}
```

---

## Troubleshooting

### Maven snapshots appearing

**This shouldn't happen** — preset filters snapshots. If you see them:
1. Check preset is correctly extended
2. Verify no local override of `allowedVersions`
3. Open an issue in this repository

---

## Developer Setup

```bash
# JSON syntax check
jq empty default.json

# Renovate schema validation
npx -p renovate renovate-config-validator default.json

# Run all pre-commit hooks
pre-commit run --all-files
```

---

## Deployment

Changes to `default.json` are **live immediately** when merged to `main`. All consuming repositories receive updates on the next Renovate run — there is no staging environment.

Changes to the [shared base preset](https://github.com/SchweizerischeBundesbahnen/casc-renovate-preset-polarion) also affect all repos using this preset.

---

**Maintained by:** SBB Polarion Team
