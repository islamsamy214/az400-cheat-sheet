# Implement GitHub Dependabot Alerts and Security Updates

## Key Concepts
- **Dependabot**: GitHub's integrated dependency management tool providing automated vulnerability alerts and dependency updates
- **Three Main Features**: Dependabot Alerts (notifications), Security Updates (vulnerability fix PRs), Version Updates (scheduled update PRs)
- **GitHub Advisory Database**: Continuously monitors vulnerability sources, compares against repository dependencies
- **Compatibility Scores**: GitHub calculates likelihood that updates will break existing functionality

## Understanding GitHub Dependabot

### Dependabot Alerts
```yaml
Alert Triggers:
  - New vulnerability disclosures in GitHub Advisory Database
  - Advisory updates (severity, affected versions, patches)
  - Dependency graph changes introducing vulnerable dependencies
  - Mend vulnerability data supplementing GitHub database

Alert Information Includes:
  - Vulnerability description
  - Severity level (CVSS score: critical, high, moderate, low)
  - Affected versions
  - Patched versions
  - CVE identifier
  - CWE classification (Common Weakness Enumeration)
  - GitHub Security Advisory link
```

### Dependabot Security Updates
- **Automatic PR Creation**: Creates pull requests **only** when security vulnerabilities detected (not every update)
- **Minimal Version Bumps**: Updates to minimum version resolving vulnerability while maintaining compatibility
- **Compatibility Scores**: Predicts whether updates will break functionality (High/Medium/Low/Unknown)
- **Includes Release Notes**: PR descriptions include changelog information from updated dependencies
- **Automated Testing**: PRs trigger existing CI/CD pipelines to validate updates
- **Update Commands**: Special comments allow maintainers to control merge timing, rebase, or ignore updates
- **Grouped Updates**: Multiple vulnerable dependencies can be updated in single PR when appropriate

### Dependabot Version Updates
```yaml
Configuration:
  - Scheduled updates: daily, weekly, or monthly
  - Update strategies: all dependencies, direct only, or specific groups
  - Version constraints: Respects semver constraints in manifest files
  - Pull request limits: Control max open PRs (default 5)
```

## Enabling Dependabot

### Enable Alerts for Repository
1. Repository → **Settings** → **Security & analysis**
2. Locate **Dependabot alerts** section → Click **Enable**
3. Ensure **Dependency graph** is enabled (required, auto-enabled for public repos)

### Organization-Wide Enablement
1. Organization **Settings** → **Security & analysis**
2. Click **Enable all** next to Dependabot alerts
3. Select **Automatically enable for new repositories**

### Supported Package Ecosystems
| Language | Package Managers |
|----------|------------------|
| JavaScript | npm (package.json, package-lock.json), Yarn (yarn.lock) |
| Python | pip (requirements.txt, Pipfile, Pipfile.lock), Poetry (poetry.lock) |
| Ruby | Bundler (Gemfile, Gemfile.lock) |
| Java | Maven (pom.xml), Gradle (build.gradle, build.gradle.kts) |
| .NET | NuGet (*.csproj, packages.config, paket.dependencies) |
| Go | Go modules (go.mod, go.sum) |
| PHP | Composer (composer.json, composer.lock) |
| Rust | Cargo (Cargo.toml, Cargo.lock) |
| Others | Elixir (Mix), Dart/Flutter (pub), Docker, GitHub Actions, Terraform |

## Configuring Dependabot Security Updates

### Enable Security Updates
- **Repository**: Settings → Security & analysis → **Dependabot security updates** → Enable
- **Organization**: Settings → Security & analysis → **Enable all** next to Dependabot security updates

### Security Update Behavior
```yaml
Automatic PR Creation:
  - Triggered when vulnerable dependency has available patch
  - Updates to minimum version resolving vulnerability
  - Respects semantic versioning (prefers patch > minor > major)
  - Triggers existing CI/CD checks

Compatibility Scores:
  High: Update likely safe (analysis of similar repositories)
  Medium: Might introduce breaking changes (requires review)
  Low: Likely breaking changes (requires code modifications)
  Unknown: Insufficient data to assess

Pull Request Management:
  - Automatic rebasing when base branch changes
  - Closed if conflicts prevent rebasing
  - Superseding updates (newer versions replace older PRs)
  - Scheduled to avoid overwhelming maintainers
```

## Configuring Dependabot Version Updates

### Create .github/dependabot.yml Configuration

**Basic Configuration:**
```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

**Advanced Configuration:**
```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "08:00"
      timezone: "America/New_York"
    open-pull-requests-limit: 10
    reviewers:
      - "team/frontend-developers"
    assignees:
      - "dependency-manager"
    labels:
      - "dependencies"
      - "npm"
    commit-message:
      prefix: "npm"
      include: "scope"
    ignore:
      - dependency-name: "lodash"
        versions: ["4.x"]
    allow:
      - dependency-type: "production"
```

**Multi-Ecosystem Configuration:**
```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/frontend"
    schedule:
      interval: "weekly"

  - package-ecosystem: "pip"
    directory: "/backend"
    schedule:
      interval: "weekly"

  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
```

## Managing Dependabot Pull Requests

### Pull Request Commands
```bash
# Comment these on Dependabot PRs:
@dependabot rebase               # Rebase PR against current base
@dependabot recreate             # Recreate PR, overwriting manual edits
@dependabot merge                # Merge PR once CI checks pass
@dependabot squash and merge     # Squash commits and merge
@dependabot cancel merge         # Cancel previously requested merge
@dependabot reopen               # Reopen closed PR
@dependabot close                # Close PR and prevent recreation
@dependabot ignore this major version    # Ignore future major version updates
@dependabot ignore this minor version    # Ignore future minor version updates
@dependabot ignore this dependency       # Ignore all future updates for this dependency
```

### Reviewing and Merging Updates
```yaml
Review Process:
  1. Examine PR description (vulnerability or version update details)
  2. Review compatibility score (likelihood of breaking changes)
  3. Check CI/CD results (verify tests pass)
  4. Review release notes (understand changes in dependency)
  5. Test locally if needed (major updates)
  6. Merge PR
```

**Automatic Merging (GitHub Actions):**
```yaml
name: Auto-merge Dependabot PRs
on: pull_request

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - name: Enable auto-merge
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Dependabot Alerts and Notifications

### Notification Channels
| Channel | Description |
|---------|-------------|
| **Web Notifications** | GitHub notifications inbox, Security tab, Repository insights |
| **Email Notifications** | Weekly digest emails, real-time emails for critical vulnerabilities |
| **Custom Notifications** | Webhooks to external systems, GitHub API queries, third-party security platform integrations |

### Configuring Notification Preferences
```yaml
User Settings:
  - GitHub Settings → Notifications → Security alerts
  - Configure: Participating, Watching, Email, Web

Organization Settings:
  - Settings → Security & analysis
  - Who receives alerts:
    * Organization owners (automatic)
    * Security managers
    * Repository administrators
```

## Reviewing Dependabot Alerts

### Alert Details
```yaml
Navigate: Repository → Security tab → Dependabot alerts

Alert Information:
  - Severity: Critical, high, moderate, low
  - Package: Affected dependency name and version
  - Vulnerability: CVE identifier and description
  - Patched versions: Versions resolving vulnerability
  - Vulnerable code paths: Whether vulnerable code actually used (reachability)
  - Auto-fix available: Whether Dependabot can create PR
```

### Managing Alerts
```yaml
Alert Actions:
  - Review pull request (if automatic security update exists)
  - Dismiss alert (with reason)
  - Snooze alert (temporary dismissal)
  - Reopen alert (if circumstances change)

Dismissal Reasons:
  - Fix started: Team actively working on remediation
  - No bandwidth: Acknowledged but can't address currently
  - Tolerable risk: Vulnerability doesn't pose significant risk
  - Inaccurate: False positive
```

## GitHub Advanced Security Integration

### Integrated Capabilities
| Feature | Description |
|---------|-------------|
| **Dependency Scanning** | Dependabot scans using GitHub Advisory Database + industry databases |
| **Secret Scanning** | Detects accidentally committed credentials, tokens, API keys |
| **Code Scanning** | CodeQL static analysis for vulnerabilities and coding errors |
| **Security Overview** | Organization-wide visibility into alerts, vulnerabilities, remediation |
| **Supply Chain Security** | Dependency graph, dependency review, SBOM generation |

### Licensing and Availability
```yaml
Public Repositories:
  - All GitHub Advanced Security features FREE

Private Repositories:
  - Requires GitHub Advanced Security license
  - Included with: GitHub Enterprise Cloud, GitHub Enterprise Server
  - GitHub Free/Team: Dependabot available, Code/Secret scanning requires license

Enterprise Features:
  - Organization-wide enablement
  - Policy enforcement (require on new repos)
  - Compliance reporting
  - Security managers (read access to all alerts)
```

### Integration with Development Workflows
```yaml
Pull Request Integration:
  - Dependency review: Highlights new vulnerabilities in PRs
  - Security checks: Code/secret scanning run automatically
  - Required reviews: Branch protection for security team approval

Security Policies:
  - SECURITY.md: Vulnerability disclosure policies
  - Code owners: Assign security team to dependency files
  - Branch protection: Require security scan status checks

Audit and Compliance:
  - Audit log: Track alert dismissals, feature enablement
  - Security policies: Enforce org-wide standards
  - Compliance integration: Export data for SOC 2, ISO 27001
```

## Critical Notes
- ⚠️ **Dependency Graph Required**: Dependabot alerts require dependency graph enabled (auto for public, manual for private repos)
- 💡 **Security vs Version Updates**: Security updates only trigger for vulnerabilities; version updates need dependabot.yml configuration
- 🎯 **Compatibility Scores**: High compatibility = likely safe; Low = likely breaking changes requiring code mods
- 📊 **Three-Layer Approach**: Alerts (notifications) → Security Updates (vulnerability PRs) → Version Updates (scheduled PRs)
- 🔒 **Zero Configuration**: Enable with single click for public repos, no manifest changes needed
- 🚀 **Pull Request Workflow**: Uses standard GitHub PR review/approval processes (familiar to developers)

[Learn More](https://learn.microsoft.com/en-us/training/modules/software-composition-analysis/4-implement-github-dependabot-alerts-security-updates)
