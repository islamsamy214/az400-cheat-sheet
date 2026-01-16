# Integrate Software Composition Analysis Checks into Pipelines

## Key Concepts
- **Shift-Left Security**: Catching vulnerabilities during development costs significantly less than fixing them in production
- **Pipeline Integration Points**: Pull request validation, CI builds, scheduled scans, release gates
- **Quality Gates**: Automated policy enforcement that blocks builds/deployments violating security standards
- **SBOM Generation**: Creating Software Bill of Materials artifacts for compliance and vulnerability tracking

## Why Integrate SCA into Pipelines?

### Shift-Left Security Benefits
```yaml
Early Vulnerability Detection:
  - Immediate feedback: Notifications within minutes of introducing vulnerable dependencies
  - Lower remediation costs: Fix during development vs production
  - Context preservation: Developers have full context about dependency choices
  - Incremental fixes: Small continuous improvements vs large remediation efforts

Continuous Compliance:
  - Policy enforcement: Automated checks on every commit
  - Audit trails: Pipeline results provide compliance evidence
  - Consistent standards: Same security review regardless of who submits
  - Risk prevention: Block vulnerable dependencies before merge

Development Velocity:
  - Automated reviews: No manual security team review bottleneck
  - Parallel processing: Security scans run alongside other build steps
  - Faster releases: Continuous validation enables confident releases
  - Reduced rework: Catch issues early eliminates expensive late-stage fixes
```

## Pipeline Integration Points

### 1. Pull Request Validation
**Pre-merge security checks prevent security debt accumulation**

**Azure Pipelines PR Validation:**
```yaml
trigger: none

pr:
  branches:
    include:
      - main
      - develop

pool:
  vmImage: "ubuntu-latest"

steps:
  - task: UseNode@1
    inputs:
      version: "18.x"
    displayName: "Install Node.js"

  - script: npm ci
    displayName: "Install dependencies"

  - task: WhiteSource@21
    inputs:
      cwd: "$(System.DefaultWorkingDirectory)"
      projectName: "$(Build.Repository.Name)"
    displayName: "Run Mend SCA scan"

  - task: PublishTestResults@2
    inputs:
      testResultsFormat: "JUnit"
      testResultsFiles: "**/test-results.xml"
    displayName: "Publish scan results"
```

**GitHub Actions PR Validation:**
```yaml
name: Security Scan

on:
  pull_request:
    branches: [main, develop]

jobs:
  sca-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Install dependencies
        run: npm ci

      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: Upload scan results
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: snyk.sarif
```

**Pull Request Checks:**
- Dependency changes: Detect which dependencies changed
- New vulnerabilities: Identify new vulnerabilities introduced
- License violations: Flag license policy violations
- Quality gates: Block PRs violating security/compliance policies
- Review comments: Post findings as PR comments

### 2. Continuous Integration Builds
**Every CI build includes SCA scanning to validate dependencies**

**Azure Pipelines CI Integration (.NET):**
```yaml
trigger:
  branches:
    include:
      - main
      - develop
      - feature/*

pool:
  vmImage: "ubuntu-latest"

variables:
  buildConfiguration: "Release"

steps:
  - task: UseDotNet@2
    inputs:
      packageType: "sdk"
      version: "7.x"
    displayName: "Install .NET SDK"

  - task: DotNetCoreCLI@2
    inputs:
      command: "restore"
      projects: "**/*.csproj"
    displayName: "Restore NuGet packages"

  - task: WhiteSource@21
    inputs:
      cwd: "$(System.DefaultWorkingDirectory)"
      projectName: "$(Build.Repository.Name)"
      scanComment: "CI Build $(Build.BuildNumber)"
      checkPolicies: true
      failBuildOnPolicyViolation: true
    displayName: "Mend SCA scan with policy enforcement"

  - task: DotNetCoreCLI@2
    inputs:
      command: "build"
      projects: "**/*.csproj"
      arguments: "--configuration $(buildConfiguration)"
    displayName: "Build application"

  - task: PublishBuildArtifacts@1
    inputs:
      pathToPublish: "$(Build.ArtifactStagingDirectory)"
      artifactName: "drop"
    displayName: "Publish build artifacts"
```

**GitHub Actions CI Integration (Python):**
```yaml
name: CI Build

on:
  push:
    branches: [main, develop]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run OWASP Dependency-Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: "my-application"
          path: "."
          format: "SARIF"
          args: >
            --failOnCVSS 7
            --suppression suppression.xml

      - name: Upload scan results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: dependency-check-report.sarif

      - name: Build application
        run: python setup.py build

      - name: Generate SBOM
        run: |
          pip install cyclonedx-bom
          cyclonedx-py -i requirements.txt -o sbom.json

      - name: Upload SBOM artifact
        uses: actions/upload-artifact@v3
        with:
          name: sbom
          path: sbom.json
```

**CI Build Features:**
- Full dependency scanning (including transitive dependencies)
- Policy enforcement (fail builds violating policies)
- SBOM generation (Software Bill of Materials artifacts)
- Baseline comparison (detect regressions vs previous builds)
- Metrics collection (vulnerability counts, remediation rates)

### 3. Scheduled Deep Scans
**Comprehensive periodic analysis without blocking developer workflow**

**Azure Pipelines Scheduled Scan:**
```yaml
schedules:
  - cron: "0 2 * * *"
    displayName: "Nightly security scan"
    branches:
      include:
        - main
    always: true

trigger: none
pr: none

pool:
  vmImage: "ubuntu-latest"

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: "18.x"
    displayName: "Install Node.js"

  - script: npm ci
    displayName: "Install dependencies"

  - task: Snyk@1
    inputs:
      serviceConnectionEndpoint: "SnykConnection"
      testType: "app"
      severityThreshold: "low"
      monitorOnBuild: true
      failOnIssues: false
      projectName: "$(Build.Repository.Name)"
    displayName: "Deep Snyk scan (all severities)"

  - task: BlackDuck@1
    inputs:
      BlackDuckService: "BlackDuckConnection"
      ScanMode: "intelligent"
      DetectArguments: "--detect.policy.check.fail.on.severities ALL"
    displayName: "Black Duck comprehensive scan"

  - task: PublishBuildArtifacts@1
    inputs:
      pathToPublish: "$(Build.ArtifactStagingDirectory)/SecurityReports"
      artifactName: "SecurityReports"
    displayName: "Publish detailed scan reports"
```

**Scheduled Scan Advantages:**
- Thorough analysis with all severity levels
- Detect newly disclosed vulnerabilities in unchanged dependencies
- Generate detailed reports for security teams/management
- Track security posture changes over time

### 4. Release Pipeline Validation
**Pre-deployment security gates validate artifacts before production**

**Azure Pipelines Release Gate:**
```yaml
stages:
  - stage: Build
    jobs:
      - job: BuildJob
        steps:
          - task: DotNetCoreCLI@2
            inputs:
              command: "build"

  - stage: SecurityValidation
    dependsOn: Build
    jobs:
      - job: SCAValidation
        steps:
          - task: DownloadBuildArtifacts@0
            inputs:
              artifactName: "drop"

          - task: WhiteSource@21
            inputs:
              cwd: "$(System.ArtifactsDirectory)/drop"
              projectName: "$(Build.Repository.Name)"
              checkPolicies: true
              failBuildOnPolicyViolation: true
            displayName: "Validate artifact dependencies"

  - stage: DeployProduction
    dependsOn: SecurityValidation
    condition: succeeded()
    jobs:
      - deployment: DeployToProduction
        environment: "production"
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: "AzureConnection"
                    appName: "my-web-app"
                    package: "$(Pipeline.Workspace)/drop"
```

**Release Validation Checks:**
- Artifact scanning (compiled artifacts and container images)
- Production-specific policies (stricter than dev/test)
- Compliance verification (license compliance before release)
- Approval gates (manual approval for accepted risks)

## Quality Gates and Policy Enforcement

### Severity-Based Policies

**Example Mend Policy:**
```json
{
  "name": "Production Security Policy",
  "enabled": true,
  "rules": [
    {
      "type": "VULNERABILITY_SEVERITY",
      "action": "FAIL_BUILD",
      "minSeverity": "HIGH"
    },
    {
      "type": "VULNERABILITY_AGE",
      "action": "FAIL_BUILD",
      "maxAge": 30,
      "minSeverity": "MEDIUM"
    }
  ]
}
```

**Example Snyk Policy:**
```yaml
# .snyk policy file
version: v1.0.0
patch: {}
ignore: {}
policies:
  - severity:
      low: ignore
      medium: warn
      high: fail
      critical: fail
```

### License-Based Policies

```json
{
  "name": "License Compliance Policy",
  "enabled": true,
  "rules": [
    {
      "type": "LICENSE_TYPE",
      "action": "FAIL_BUILD",
      "deniedLicenses": ["GPL-2.0", "GPL-3.0", "AGPL-3.0"]
    },
    {
      "type": "LICENSE_TYPE",
      "action": "REQUIRE_APPROVAL",
      "approvalRequired": ["LGPL-2.1", "LGPL-3.0", "MPL-2.0"]
    }
  ]
}
```

**License Policy Categories:**
- **Denied licenses**: Prohibited in all circumstances (strong copyleft for proprietary software)
- **Approval-required licenses**: Need legal review (weak copyleft, custom licenses)
- **Allowed licenses**: Permissive licenses acceptable without review (MIT, Apache 2.0, BSD)

### Custom Policy Rules

```json
{
  "name": "Advanced Security Policy",
  "enabled": true,
  "rules": [
    {
      "type": "VULNERABILITY_CVSS_SCORE",
      "action": "FAIL_BUILD",
      "minScore": 7.0,
      "condition": "exploitMaturity == 'FUNCTIONAL' OR exploitMaturity == 'HIGH'"
    },
    {
      "type": "DEPENDENCY_AGE",
      "action": "WARN",
      "maxAge": 365,
      "message": "Dependency has not been updated in over a year"
    },
    {
      "type": "MAINTAINER_STATUS",
      "action": "WARN",
      "condition": "abandonedProject == true"
    }
  ]
}
```

## SBOM Generation in Pipelines

### CycloneDX SBOM Generation

```yaml
steps:
  - task: UseNode@1
    inputs:
      version: "18.x"

  - script: |
      npm ci
      npm install -g @cyclonedx/cyclonedx-npm
      cyclonedx-npm --output-file sbom.json
    displayName: "Generate CycloneDX SBOM"

  - task: PublishBuildArtifacts@1
    inputs:
      pathToPublish: "sbom.json"
      artifactName: "SBOM"
```

### SPDX SBOM Generation

```yaml
steps:
  - script: |
      pip install spdx-tools
      pip-licenses --format=json --output-file=licenses.json
      # Convert to SPDX format using custom script
      python scripts/generate_spdx.py licenses.json sbom.spdx
    displayName: "Generate SPDX SBOM"
```

**SBOM Use Cases:**
- **Compliance evidence**: Provide to customers/auditors
- **Vulnerability tracking**: Quickly determine if new CVEs affect deployed software
- **Supply chain security**: Track software provenance, detect supply chain attacks
- **License compliance**: Document all license obligations

## Best Practices for Pipeline Integration

### Optimize Scan Performance
```yaml
Best Practices:
  - Incremental scanning: Use for PRs; full scans for main branch
  - Caching: Cache dependency resolution and scan results
  - Parallel execution: Run SCA scans parallel with other build steps
  - Scan scheduling: Comprehensive scans on schedules (not blocking every commit)
```

### Manage False Positives
- **Suppression files**: Maintain for known false positives
- **Context-aware policies**: Consider whether vulnerable code is actually used
- **Regular review**: Periodically review suppressed findings

### Provide Actionable Feedback
- **Clear messages**: Explain policy violations and remediation steps
- **Remediation guidance**: Link to documentation on how to fix vulnerabilities
- **Prioritization**: Rank by severity and exploitability

### Monitor and Improve
- **Metrics dashboards**: Track vulnerability trends, remediation time, compliance rates
- **Team education**: Train developers on secure dependency selection
- **Process iteration**: Continuously refine policies based on feedback and metrics

## Critical Notes
- ⚠️ **Shift-Left Security**: Fixing during development costs 10-100x less than production fixes
- 💡 **Four Integration Points**: PR validation → CI builds → Scheduled scans → Release gates
- 🎯 **Quality Gates**: Automated policy enforcement prevents vulnerable code from reaching production
- 📊 **SBOM is Essential**: Software Bill of Materials required for compliance, vulnerability tracking
- 🔒 **Policy-Driven**: Define security/license policies; automate enforcement in pipelines
- 🚀 **Parallel Scanning**: Run SCA alongside other build steps to minimize impact on build times

[Learn More](https://learn.microsoft.com/en-us/training/modules/software-composition-analysis/5-integrate-software-composition-analysis-checks-into-pipelines)
