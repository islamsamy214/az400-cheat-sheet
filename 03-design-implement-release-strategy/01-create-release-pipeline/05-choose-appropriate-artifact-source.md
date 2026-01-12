# Choose the Appropriate Artifact Source

Selecting the right artifact source is critical for ensuring deployment reliability, traceability, auditability, and compliance. This unit provides decision-making guidance for choosing artifact sources based on organizational requirements.

## Key Selection Criteria

### 1. Traceability

**Requirement**: Establish exactly which source code was deployed to each environment.

```
Traceability Chain:

Production Deployment
        ↓ (What's running?)
Artifact: myapp-1.2.3.zip
        ↓ (Where did it come from?)
Build: #1234 (2026-01-12 10:30 AM)
        ↓ (What source code?)
Commit: abc123def (main branch)
        ↓ (Who made changes?)
Author: John Doe
Changes: Fixed critical security vulnerability
```

**Strong Traceability** ✅:
- Build artifacts with embedded metadata
- Container images with tags linked to builds
- Package feeds with version numbers

**Weak Traceability** ⚠️:
- Git repositories (commits can be force-pushed)
- File shares (no version tracking)
- Manual deployments

**Example - Strong Traceability**:
```yaml
# Build pipeline creates traceable artifact
trigger:
  branches:
    include:
    - main

variables:
  BuildConfiguration: 'Release'
  Version: '1.2.$(Rev:r)'

steps:
- task: DotNetCoreCLI@2
  displayName: 'Build and Package'
  inputs:
    command: 'publish'
    projects: '**/*.csproj'
    arguments: '--configuration $(BuildConfiguration) /p:Version=$(Version)'
    publishWebProjects: true
    zipAfterPublish: true

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact with Metadata'
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'drop'
    
# Artifact metadata automatically includes:
# - Build ID
# - Commit SHA
# - Branch name
# - Build timestamp
# - Requester
```

**Verification in Production**:
```powershell
# Query deployed artifact metadata
$artifact = Get-AzWebApp -Name "myapp-prod" -ResourceGroupName "Production"
$artifact.SiteConfig.AppSettings | Where-Object { $_.Name -eq "BUILD_ID" }
# Returns: BUILD_ID=1234

# Trace back to source code
az pipelines runs show --id 1234 --query "sourceVersion"
# Returns: abc123def456789
```

### 2. Auditability

**Requirement**: Prove which packages were deployed, when, and by whom for compliance and regulatory requirements.

**Audit Trail Components**:

```
Complete Audit Trail:

┌─────────────────────────────────────────┐
│  Who?  Release Manager (jane@company)   │
│  What? myapp-1.2.3.zip                  │
│  When? 2026-01-12 14:30:00 UTC          │
│  Where? Production (Azure App Service)  │
│  Why?  Release Notes: Critical hotfix   │
│  How?  Manual approval + Quality gates  │
└─────────────────────────────────────────┘
          ↓
┌─────────────────────────────────────────┐
│  Approval Chain:                        │
│  ✅ Security Team (approved 14:15)      │
│  ✅ Change Board (approved 14:20)       │
│  ✅ Release Manager (approved 14:25)    │
└─────────────────────────────────────────┘
          ↓
┌─────────────────────────────────────────┐
│  Quality Gates:                         │
│  ✅ No active alerts                    │
│  ✅ No P0/P1 bugs                       │
│  ✅ Health check passed                 │
└─────────────────────────────────────────┘
```

**Audit Requirements by Industry**:

| Industry | Requirement | Artifact Source | Retention |
|----------|-------------|-----------------|-----------|
| **Healthcare (HIPAA)** | Full deployment history | Build artifacts in secure storage | 7 years |
| **Finance (SOX)** | Change management audit trail | Build artifacts + approval logs | 7 years |
| **Government (FedRAMP)** | Security scan results, approvals | Build artifacts with security metadata | Indefinite |
| **Retail (PCI-DSS)** | Code review, vulnerability scans | Build artifacts with scan results | 3 years |

**Example - Compliance-Ready Artifact**:
```yaml
# Build pipeline for regulated environment
steps:
# 1. Security Scan
- task: WhiteSource@21
  displayName: 'Security Vulnerability Scan'
  inputs:
    cwd: '$(Build.SourcesDirectory)'

# 2. Code Quality Analysis
- task: SonarQubePrepare@5
  inputs:
    SonarQube: 'SonarQube'
    projectKey: 'myapp'

- task: SonarQubeAnalyze@5

# 3. Build with Audit Metadata
- task: DotNetCoreCLI@2
  inputs:
    command: 'publish'
    publishWebProjects: true
    
# 4. Package with Compliance Info
- task: PowerShell@2
  displayName: 'Add Compliance Metadata'
  inputs:
    targetType: 'inline'
    script: |
      $compliance = @{
        SecurityScanId = "$(WhiteSource.ScanId)"
        SonarQubeGate = "$(SonarQube.QualityGate)"
        BuildApprover = "$(Build.RequestedFor)"
        BuildTimestamp = "$(Build.FinishTime)"
        ComplianceStandard = "SOX, HIPAA"
      }
      $compliance | ConvertTo-Json | Out-File compliance.json

# 5. Sign Artifact (Code Signing)
- task: AzureKeyVault@1
  inputs:
    azureSubscription: 'Production'
    keyVaultName: 'compliance-vault'

- task: SignTool@1
  inputs:
    signMode: 'files'
    files: '**/*.dll'
    
# 6. Publish with Digital Signature
- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'compliance-artifact'
```

**Audit Query Example**:
```bash
# Query: "What was deployed to production on Jan 12, 2026?"
az pipelines runs list \
  --query "[?finishTime > '2026-01-12' && result=='succeeded' && sourceVersion contains 'production']" \
  --output table

# Returns complete audit trail
```

### 3. Source Integrity

**Requirement**: Guarantee deployed packages match exact source code without unauthorized modifications.

**Integrity Verification Methods**:

```
Strong Integrity (Cryptographic):

Build Pipeline
     ↓
┌─────────────────────────┐
│ 1. Compile Code         │
│ 2. Run Tests            │
│ 3. Package Artifact     │
│ 4. Calculate Hash       │  SHA256: abc123...
│ 5. Digital Signature    │  Signed with private key
└─────────────────────────┘
     ↓
Store in Secure Location (Azure Artifacts, ACR)
     ↓
Release Pipeline
     ↓
┌─────────────────────────┐
│ 1. Download Artifact    │
│ 2. Verify Hash          │  Match: abc123... ✅
│ 3. Verify Signature     │  Valid signature ✅
│ 4. Deploy               │
└─────────────────────────┘
```

**Implementation - Artifact Integrity Check**:
```yaml
# Build pipeline - Generate hash
- task: PowerShell@2
  displayName: 'Calculate Artifact Hash'
  inputs:
    targetType: 'inline'
    script: |
      $artifact = "$(Build.ArtifactStagingDirectory)/myapp.zip"
      $hash = Get-FileHash -Path $artifact -Algorithm SHA256
      $hash.Hash | Out-File artifact.sha256
      Write-Host "##vso[task.setvariable variable=ArtifactHash]$($hash.Hash)"

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'drop'

# Release pipeline - Verify hash
- task: DownloadBuildArtifacts@0
  inputs:
    artifactName: 'drop'

- task: PowerShell@2
  displayName: 'Verify Artifact Integrity'
  inputs:
    targetType: 'inline'
    script: |
      $artifact = "$(System.ArtifactsDirectory)/drop/myapp.zip"
      $expectedHash = Get-Content "$(System.ArtifactsDirectory)/drop/artifact.sha256"
      $actualHash = (Get-FileHash -Path $artifact -Algorithm SHA256).Hash
      
      if ($expectedHash -ne $actualHash) {
        throw "Artifact integrity check FAILED! Possible tampering detected."
      }
      
      Write-Host "Artifact integrity verified ✅"
```

**Integrity Risks by Source Type**:

| Source Type | Integrity Risk | Mitigation |
|-------------|----------------|------------|
| **Build Artifacts (Azure)** | Low - access controlled | Use managed identities, RBAC |
| **Container Registry (ACR)** | Low - immutable tags | Enable content trust, image scanning |
| **Git Repository** | Medium - commits can be force-pushed | Use protected branches, required reviews |
| **File Share** | High - anyone with access can modify | ❌ Don't use for production |
| **Network Share** | Very High - no access control | ❌ Don't use for production |

### 4. Immutability

**Requirement**: Packages must never change after creation—deploy the same bits to all environments.

**Immutable Artifact Characteristics**:

```
Immutable Package:

myapp-1.2.3.zip created on 2026-01-12 10:30 AM
     ↓
Deployed to Dev (2026-01-12 11:00 AM)
     ↓ (Same package!)
Deployed to QA (2026-01-12 14:00 PM)
     ↓ (Same package!)
Deployed to Staging (2026-01-13 09:00 AM)
     ↓ (Same package!)
Deployed to Production (2026-01-13 14:00 PM)
     ↓ (Same package!)

Content NEVER changes:
✅ Same binaries
✅ Same libraries
✅ Same dependencies
✅ Same hash (SHA256: abc123...)

Only configuration differs:
  Dev:  config-dev.json
  QA:   config-qa.json
  Prod: config-prod.json
```

**Immutability by Source Type**:

| Source | Immutable? | Reason |
|--------|------------|--------|
| **Build Artifacts** | ✅ Yes | Stored in secure location, version-locked |
| **Container Images (immutable tags)** | ✅ Yes | Tag once, never overwrite |
| **Container Images (mutable tags)** | ❌ No | `:latest` tag can be overwritten |
| **Package Feeds (NuGet/npm)** | ✅ Yes | Version numbers enforce immutability |
| **Git Repository** | ❌ No | Commits can be amended, force-pushed |
| **File Share** | ❌ No | Files can be modified at any time |

**Best Practice - Immutable Container Tags**:
```bash
# ❌ Bad: Mutable tag
docker tag myapp:latest myacr.azurecr.io/myapp:latest
docker push myacr.azurecr.io/myapp:latest
# Problem: ":latest" can be overwritten!

# ✅ Good: Immutable tag with build number
docker tag myapp:latest myacr.azurecr.io/myapp:1.2.3-build1234
docker push myacr.azurecr.io/myapp:1.2.3-build1234
# Benefit: Unique, immutable tag

# Also push latest for convenience
docker tag myapp:latest myacr.azurecr.io/myapp:latest
docker push myacr.azurecr.io/myapp:latest
# But always deploy using immutable tag: 1.2.3-build1234
```

## Decision Matrix

### Artifact Source Selection Guide

```
Decision Tree:

Is it a compiled application?
  ├─ Yes → Build Artifacts ✅
  └─ No → Is it containerized?
           ├─ Yes → Container Registry ✅
           └─ No → Is it a package/library?
                    ├─ Yes → Package Feed (NuGet/npm/Maven) ✅
                    └─ No → Is it scripts/templates?
                             ├─ Yes → Git Repository ⚠️
                             └─ No → Build Artifacts (universal package) ✅
```

### Use Case Recommendations

| Application Type | Recommended Source | Reason |
|------------------|-------------------|--------|
| **ASP.NET Core Web App** | Build artifacts (ZIP) | Compiled binaries, needs traceability |
| **Node.js Application** | Build artifacts or Container | Build includes npm install, immutability |
| **Docker/Kubernetes App** | Container Registry (ACR) | Native container format, image layers |
| **NuGet Library** | Azure Artifacts (NuGet feed) | Package versioning, dependency management |
| **npm Package** | Azure Artifacts (npm feed) | Package versioning, upstream sources |
| **Java/Spring Boot** | Build artifacts (JAR) or Maven | Compiled JAR with dependencies |
| **Python Application** | Build artifacts or Container | Virtual environment packaged |
| **PowerShell Scripts** | Git Repository | No compilation, version controlled |
| **Terraform/ARM Templates** | Git Repository or Build artifacts | IaC files, version controlled |
| **Database Migration** | Build artifacts (SQL scripts) | Versioned migrations, auditability |
| **Static Website** | Build artifacts or Blob Storage | Compiled front-end (React, Angular) |

### Compliance Requirements Matrix

| Requirement | Suitable Source | Unsuitable Source |
|-------------|----------------|-------------------|
| **SOX Compliance** | Build artifacts with audit logs | File shares (no audit trail) |
| **HIPAA Compliance** | Build artifacts with encryption | Network shares (insecure) |
| **ISO 27001** | Build artifacts with access control | Public repositories (no access control) |
| **PCI-DSS** | Build artifacts with security scans | File shares (no security validation) |
| **FedRAMP** | Build artifacts with gov-cloud storage | Commercial cloud without FedRAMP |

## Real-World Scenarios

### Scenario 1: Financial Services Application (Highly Regulated)

**Requirements**:
- Full audit trail for 7 years
- Source code traceability
- Digital signatures on all binaries
- Immutable packages
- Compliance with SOX, PCI-DSS

**Solution**:
```yaml
Artifact Source: Build Artifacts in Azure Artifacts
Storage: Premium tier with geo-redundancy
Retention: Indefinite (manual cleanup after 7 years)
Security:
  - Code signing with Azure Key Vault
  - Security scans (WhiteSource, Checkmarx)
  - Vulnerability scanning
  - Access control (Azure AD + RBAC)
Metadata:
  - Build ID and timestamp
  - Approver information
  - Security scan results
  - Compliance attestation
```

### Scenario 2: Startup - Fast-Moving Web Application

**Requirements**:
- Fast iteration cycles
- Cost-effective storage
- Simple deployment process
- Kubernetes-native

**Solution**:
```yaml
Artifact Source: Azure Container Registry
Storage: Basic tier (upgrade as needed)
Retention: 30 days (dev), 90 days (production)
Deployment: Kubernetes manifests from Git
Pipeline: Build → Push to ACR → Deploy to AKS
Benefits:
  - Container-native workflow
  - Fast deployment (image pull)
  - Cost-effective storage
  - Easy rollback (previous image tags)
```

### Scenario 3: Enterprise Multi-Product Suite

**Requirements**:
- Shared libraries across products
- Version management for dependencies
- Package reusability
- Centralized package governance

**Solution**:
```yaml
Artifact Source: Azure Artifacts (multiple feeds)
Feeds:
  - NuGet feed (shared .NET libraries)
  - npm feed (shared React components)
  - Maven feed (shared Java libraries)
  - Universal packages feed (deployment scripts)
Upstream Sources: Public NuGet, npm, Maven
Security: Vulnerability scanning, license compliance
Versioning: Semantic versioning (major.minor.patch)
```

### Scenario 4: Infrastructure as Code Deployments

**Requirements**:
- Version-controlled templates
- Code review process
- No compilation needed
- Rapid updates

**Solution**:
```yaml
Artifact Source: Git Repository (primary) + Build Artifacts (validated)
Workflow:
  1. Store Terraform/ARM templates in Git
  2. PR/Code review process for changes
  3. Build pipeline validates templates
  4. Publish validated templates as artifacts
  5. Release pipeline uses validated artifacts
Benefits:
  - Version control with Git history
  - Code review for infrastructure changes
  - Validation before deployment
  - Immutable validated artifacts
```

## Anti-Patterns to Avoid

### ❌ Anti-Pattern 1: Network File Shares

**Problem**:
```
\\fileserver\deployments\myapp\
  ├── myapp.zip  (Modified yesterday by unknown user)
  ├── myapp-backup.zip  (From last week?)
  └── config.json  (Which environment is this?)
```

**Issues**:
- No version control
- No access audit trail
- Anyone can modify files
- No traceability to source code
- Compliance nightmare

**Solution**: Use build artifacts in Azure Artifacts with RBAC.

### ❌ Anti-Pattern 2: Rebuilding for Each Environment

**Problem**:
```
Build for Dev → Deploy
Build for QA → Deploy
Build for Prod → Deploy
```

**Issues**:
- Different binaries in each environment
- QA tests version A, production gets version B
- Compiler differences cause bugs
- No guarantee of consistency

**Solution**: Build once, deploy immutable artifact to all environments.

### ❌ Anti-Pattern 3: Mutable Container Tags

**Problem**:
```
docker tag myapp:latest myacr.azurecr.io/myapp:prod
docker push myacr.azurecr.io/myapp:prod

# Later, rebuild and push again
docker tag myapp:latest myacr.azurecr.io/myapp:prod
docker push myacr.azurecr.io/myapp:prod
# Problem: Same tag, different image!
```

**Issues**:
- Can't trace which version is deployed
- Rollback is impossible
- No immutability

**Solution**: Use immutable tags with build numbers (myapp:1.2.3-build1234).

### ❌ Anti-Pattern 4: Git as Primary Artifact Source for Compiled Apps

**Problem**:
```
Release Pipeline:
  1. Clone Git repository
  2. Run npm install / dotnet build
  3. Deploy built files
```

**Issues**:
- Build happens in release pipeline (slow)
- Different Node/SDK versions produce different artifacts
- No artifact storage (can't rollback)
- Tests not run in build phase

**Solution**: Build in build pipeline, publish artifacts, deploy artifacts in release.

## Critical Notes

🎯 **Traceability First**: Choose sources that provide complete traceability from deployment back to source commit.

💡 **Immutability Required**: Artifacts must be immutable—same package deployed to all environments without modification.

⚠️ **Audit Trail**: Regulated industries require comprehensive audit trails—choose sources with built-in auditing.

📊 **Security**: Build artifacts in secure storage (Azure Artifacts, ACR) with access control—never use network shares for production.

🔄 **Compliance**: Financial, healthcare, government sectors have strict requirements—plan artifact storage accordingly.

✨ **Build Once**: The golden rule—build once, deploy many times with configuration-only changes.

## Quick Reference

### Artifact Source Scorecard

| Criterion | Build Artifacts | Container Registry | Git Repo | File Share |
|-----------|----------------|-------------------|----------|------------|
| **Traceability** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ |
| **Auditability** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ |
| **Immutability** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐ |
| **Security** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐ |
| **Compliance** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ |

**Legend**: ⭐⭐⭐⭐⭐ Excellent | ⭐⭐⭐ Good | ⭐ Poor

### Recommendation Summary

```
For Production Deployments:
✅ Build Artifacts (most scenarios)
✅ Container Registry (Docker/K8s)
✅ Azure Artifacts (packages)

For Development/Testing:
✅ Git Repository (scripts, IaC)
✅ Build Artifacts (consistency)

Never for Production:
❌ Network File Shares
❌ Mutable container tags (:latest)
❌ Rebuilding for each environment
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline-devops/5-choose-appropriate-artifact-source)
