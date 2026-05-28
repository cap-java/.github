# cap-java/.github

Central repository for shared GitHub Actions and reusable workflows used across the `cap-java` organization.

This repository also contains the [default community health files](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) (Code of Conduct, Security Policy) for the organization.

---

## Composite Actions

Reusable composite actions that can be referenced from any workflow:

```yaml
uses: cap-java/.github/actions/<action-name>@main
```

### `actions/build`

Sets up Java (SAPMachine), Maven, and `@sap/cds-dk`, then runs `mvn clean install`.

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `java-version` | yes | - | Java version |
| `maven-version` | yes | - | Maven version |
| `cds-dk-version` | no | `9.9.1` | `@sap/cds-dk` version |
| `maven-args` | no | `''` | Extra Maven arguments (e.g. `-P '!with-integration-tests'`) |

### `actions/deploy-release`

Deploys artifacts to Maven Central with GPG signing.

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `user` | yes | - | Maven Central username |
| `password` | yes | - | Maven Central password |
| `pgp-pub-key` | yes | - | GPG public key ID |
| `pgp-private-key` | yes | - | GPG private key |
| `pgp-passphrase` | yes | - | GPG passphrase |
| `revision` | yes | - | Release version (Git tag) |
| `maven-version` | yes | - | Maven version |
| `cds-dk-version` | no | `9.9.1` | `@sap/cds-dk` version |
| `maven-profiles` | no | `deploy-release` | Maven profiles to activate |
| `maven-extra-args` | no | `''` | Extra Maven arguments |

### `actions/scan-with-blackduck`

Runs a BlackDuck SCA scan with configurable project settings.

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `blackduck_token` | yes | - | BlackDuck authentication token |
| `maven-version` | yes | - | Maven version |
| `project-name` | yes | - | BlackDuck project name |
| `included-modules` | yes | - | Comma-separated Maven modules to scan |
| `java-version` | no | `17` | Java version |
| `version` | no | `''` | Explicit project version (falls back to Maven revision major.minor) |
| `scan_mode` | no | `FULL` | `FULL` or `RAPID` |
| `rapid_compare_mode` | no | `''` | e.g. `BOM_COMPARE` (only for RAPID mode) |
| `github_token` | no | `''` | GitHub token for PR comments |
| `user-groups` | no | `CDSJAVA-OPEN-SOURCE` | BlackDuck user groups |
| `excluded-dirs` | no | `**/*test*,**/samples/**` | Directories to exclude |
| `excluded-scopes` | no | `test,provided` | Maven scopes to exclude |

### `actions/scan-with-codeql`

Runs GitHub CodeQL security analysis.

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `java-version` | yes | - | Java version |
| `maven-version` | yes | - | Maven version |
| `language` | yes | - | `java-kotlin` or `actions` |
| `queries` | no | `''` | Query suite (e.g. `security-extended`) |
| `cds-dk-version` | no | `9.9.1` | `@sap/cds-dk` version |
| `build-command` | no | `mvn clean compile -B -ntp -Dcds.install-node.skip` | Maven compile command |

### `actions/scan-with-sonar`

Builds the project with coverage and runs a SonarQube scan.

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `sonarq-token` | yes | - | SonarQube token |
| `github-token` | yes | - | GitHub token |
| `java-version` | yes | - | Java version |
| `maven-version` | yes | - | Maven version |
| `sonar-project-key` | yes | - | SonarQube project key |
| `coverage-report-path` | yes | - | Path to aggregate JaCoCo XML report |
| `build-script` | yes | - | Multi-line shell script to build and produce coverage |
| `exclusions` | no | `**/samples/**` | `sonar.exclusions` patterns |
| `coverage-exclusions` | no | `**/src/test/**,**/src/gen/**` | `sonar.coverage.exclusions` patterns |
| `cds-dk-version` | no | `9.9.1` | `@sap/cds-dk` version |

### `actions/cf-login`

Installs the CF CLI and authenticates to Cloud Foundry with retry logic.

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `cf-api` | yes | - | CF API endpoint |
| `cf-username` | yes | - | CF username |
| `cf-password` | yes | - | CF password |
| `cf-org` | yes | - | CF organization |
| `cf-space` | yes | - | CF space |
| `cf-cli-version` | no | `8.18.3` | CF CLI version |

---

## Reusable Workflows

Reusable workflows that can be called from any repository:

```yaml
jobs:
  my-job:
    uses: cap-java/.github/.github/workflows/<workflow>.yml@main
```

### `.github/workflows/issue.yml`

Auto-labels new issues with "New" and posts a welcome comment.

**Caller example:**
```yaml
name: Label issues
permissions: {}
on:
  issues:
    types: [opened]
jobs:
  label:
    uses: cap-java/.github/.github/workflows/issue.yml@main
    permissions:
      issues: write
```

### `.github/workflows/stale.yml`

Automatically closes issues after a period of inactivity.

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `days-before-close` | no | `14` | Days of inactivity before closing |
| `stale-label` | no | `author action` | Label to mark stale issues |
| `close-message` | no | *(default message)* | Message posted on close |

**Caller example:**
```yaml
name: Close stale issues
permissions: {}
on:
  schedule:
    - cron: "30 1 * * *"
jobs:
  stale:
    uses: cap-java/.github/.github/workflows/stale.yml@main
    permissions:
      actions: write
      issues: write
      pull-requests: write
```

### `.github/workflows/prevent-issue-labeling.yml`

Prevents non-bot users from applying the "New" label to issues.

**Caller example:**
```yaml
name: Prevent "New" Label on Issues
permissions: {}
on:
  issues:
    types: [labeled]
jobs:
  guard:
    uses: cap-java/.github/.github/workflows/prevent-issue-labeling.yml@main
    permissions:
      issues: write
```

---

## Migration Guide

To migrate a repository to use the centralized actions and workflows:

1. **Replace local composite actions** with references to `cap-java/.github/actions/<name>@main`
2. **Replace identical workflows** (issue, stale, prevent-labeling) with thin callers
3. **Keep project-specific items** locally:
   - `pipeline.yml` (project-specific CI jobs)
   - `release.yml` (project-specific release pipeline)
   - `cf-bind` action (project-specific service bindings -- use `cf-login` for the shared part)
   - `integration-tests` action (project-specific test commands)
   - `dependabot.yml`, `CODEOWNERS`
