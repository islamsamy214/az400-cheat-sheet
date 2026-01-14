# Implement Azure DevOps Secure Files

**Duration**: 3 minutes

Azure DevOps Secure Files provides encrypted storage for sensitive file assets including certificates, provisioning profiles, and signing keys within CI/CD workflows.

---

## Overview

### What are Secure Files?
**Azure DevOps Secure Files**: Encrypted storage infrastructure for sensitive file assets within Azure DevOps organizational boundaries, enabling secure file consumption during pipeline execution.

**Storage Characteristics**:
- **Encryption at rest**: 2048-bit RSA keys
- **Access control**: Fine-grained permissions (upload, download, pipeline usage)
- **Key management**: Keys stored in Azure Key Vault (managed by Azure DevOps)
- **Protected resource**: Access restricted to authorized users and pipelines only

**Use Cases**:
- ✅ Code signing certificates (.pfx, .p12)
- ✅ SSL/TLS certificates
- ✅ SSH private keys
- ✅ Apple provisioning profiles (.mobileprovision)
- ✅ Android keystore files (.jks, .keystore)
- ✅ Configuration files with embedded secrets
- ✅ License files

---

## When to Use Secure Files

### Secure Files vs Key Vault vs Variable Groups

| Storage | Best For | Access Pattern | Cost | Use Case |
|---------|----------|----------------|------|----------|
| **Secure Files** | Binary files (certificates, keys) | Download to agent | Free | Code signing, mobile app provisioning |
| **Key Vault** | Text secrets (passwords, API keys) | Reference as variables | ~$0.03/10K operations | Database passwords, API keys |
| **Variable Groups** | Non-sensitive config | Direct variable substitution | Free | Build numbers, environment names |

**Decision Tree**:
```
Is it a file (not text)?
├─ Yes → Is it sensitive?
│   ├─ Yes → Secure Files ✅
│   └─ No → Store in Git repository
└─ No (text secret) → Is it frequently rotated?
    ├─ Yes → Key Vault (supports rotation)
    └─ No → Secure Files or Key Vault
```

### Example Scenarios

**Scenario 1: iOS App Code Signing**
```yaml
# Need to sign iOS app with distribution certificate
# Certificate: Binary .p12 file with private key
# Solution: Secure Files ✅ (binary file, used during build)

steps:
- task: DownloadSecureFile@1
  name: certificate
  inputs:
    secureFile: 'ios-distribution.p12'

- task: InstallAppleCertificate@2
  inputs:
    certSecureFile: 'ios-distribution.p12'
    certPwd: $(CertificatePassword)  ← From Key Vault
```

**Scenario 2: Database Password**
```yaml
# Need database password for deployment
# Password: Text string, rotated quarterly
# Solution: Key Vault ✅ (text secret, supports rotation)

variables:
- group: 'prod-secrets'  # Linked to Key Vault

steps:
- script: |
    sqlcmd -S $(DatabaseServer) -U admin -P $(DatabasePassword)
```

---

## Implementation

### Step 1: Upload Secure File

#### Via Azure DevOps Portal
```
1. Navigate to Azure DevOps project
2. Select "Pipelines" from left navigation
3. Click "Library" under Pipelines
4. Select "Secure files" tab
5. Click "+ Secure file" button
6. Browse and select file (e.g., ios-dist.p12)
7. Provide descriptive name: "iOS Distribution Certificate"
8. Click "OK"
```

**File uploaded is**:
- ✅ Encrypted with 2048-bit RSA
- ✅ Stored in Azure Key Vault (managed by Azure DevOps)
- ✅ Available only to authorized pipelines
- ✅ Audit logged (who uploaded, when)

#### Via Azure CLI (API)
```bash
# Install Azure DevOps extension
az extension add --name azure-devops

# Set default organization and project
az devops configure --defaults organization=https://dev.azure.com/myorg project=MyProject

# Upload secure file (not directly supported via CLI, use REST API)
curl -X POST "https://dev.azure.com/myorg/MyProject/_apis/distributedtask/securefiles?api-version=7.0" \
  -H "Authorization: Bearer $(System.AccessToken)" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "ios-distribution.p12",
    "data": "base64EncodedFileContent..."
  }'
```

### Step 2: Grant Pipeline Permissions

**Why Permissions Matter**: Secure files are **protected resources**. Pipelines must be explicitly authorized to access them (principle of least privilege).

#### Grant Access via Portal
```
1. Navigate to Library > Secure files
2. Click on secure file (e.g., "iOS Distribution Certificate")
3. Click "Pipeline permissions" in toolbar
4. Click "+" button to add pipeline
5. Select authorized pipeline(s) from dropdown
6. Click "Save"
```

**Result**: Only authorized pipelines can download this secure file.

#### Revoke Access
```
1. Navigate to secure file
2. Click "Pipeline permissions"
3. Click "..." menu next to pipeline
4. Select "Remove"
```

**Security Model**:
```
Secure File
  ├─ Authorized Pipelines
  │   ├─ iOS-Build-Pipeline ✅
  │   ├─ iOS-Release-Pipeline ✅
  │   └─ Android-Build-Pipeline ❌ (not authorized)
  └─ Access Logs
      ├─ 2026-01-14 10:30 - iOS-Build-Pipeline downloaded
      └─ 2026-01-14 09:15 - iOS-Release-Pipeline downloaded
```

### Step 3: Download Secure File in Pipeline

#### YAML Pipeline Example
```yaml
# Basic download
steps:
- task: DownloadSecureFile@1
  name: caCertificate  # Reference name for output variable
  displayName: 'Download CA Certificate'
  inputs:
    secureFile: 'myCACertificate.pem'  # Name in Secure Files library

- script: |
    echo "Certificate downloaded to: $(caCertificate.secureFilePath)"
    # Use certificate in subsequent steps
  displayName: 'Display certificate path'
```

**Output Variable**: `$(caCertificate.secureFilePath)` contains absolute path to downloaded file.

#### Complete iOS Code Signing Example
```yaml
trigger:
- main

pool:
  vmImage: 'macOS-latest'

variables:
- group: 'ios-secrets'  # Variable group with CertificatePassword

steps:
# Step 1: Download distribution certificate
- task: DownloadSecureFile@1
  name: distributionCertificate
  displayName: 'Download iOS Distribution Certificate'
  inputs:
    secureFile: 'ios-distribution.p12'

# Step 2: Download provisioning profile
- task: DownloadSecureFile@1
  name: provisioningProfile
  displayName: 'Download Provisioning Profile'
  inputs:
    secureFile: 'ios-app-store.mobileprovision'

# Step 3: Install certificate
- task: InstallAppleCertificate@2
  displayName: 'Install Apple Certificate'
  inputs:
    certSecureFile: 'ios-distribution.p12'
    certPwd: $(CertificatePassword)  # From variable group
    keychain: 'temp'

# Step 4: Install provisioning profile
- task: InstallAppleProvisioningProfile@1
  displayName: 'Install Provisioning Profile'
  inputs:
    provisioningProfileLocation: 'secureFiles'
    provProfileSecureFile: 'ios-app-store.mobileprovision'

# Step 5: Build and sign iOS app
- task: Xcode@5
  displayName: 'Build iOS App'
  inputs:
    actions: 'build'
    scheme: 'MyApp'
    sdk: 'iphoneos'
    configuration: 'Release'
    xcWorkspacePath: '**/*.xcworkspace'
    xcodeVersion: 'default'
    signingOption: 'manual'
    signingIdentity: '$(APPLE_CERTIFICATE_SIGNING_IDENTITY)'
    provisioningProfileUuid: '$(APPLE_PROV_PROFILE_UUID)'

# Step 6: Publish signed IPA
- task: PublishBuildArtifacts@1
  displayName: 'Publish IPA'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'ios-app'
```

#### Linux Certificate Installation Example
```yaml
steps:
# Download CA certificate
- task: DownloadSecureFile@1
  name: caCertificate
  displayName: 'Download CA Certificate'
  inputs:
    secureFile: 'myCACertificate.pem'

# Install certificate on Linux agent
- script: |
    echo "Installing certificate from $(caCertificate.secureFilePath)"
    
    # Set correct ownership and permissions
    sudo chown root:root $(caCertificate.secureFilePath)
    sudo chmod 644 $(caCertificate.secureFilePath)
    
    # Install certificate
    sudo cp $(caCertificate.secureFilePath) /usr/local/share/ca-certificates/myCA.crt
    sudo update-ca-certificates
    
    # Verify installation
    openssl x509 -in /usr/local/share/ca-certificates/myCA.crt -text -noout
  displayName: 'Install CA Certificate'
```

#### SSH Private Key Example
```yaml
steps:
# Download SSH private key
- task: DownloadSecureFile@1
  name: sshKey
  displayName: 'Download SSH Private Key'
  inputs:
    secureFile: 'deployment-key'

# Configure SSH
- script: |
    mkdir -p ~/.ssh
    cp $(sshKey.secureFilePath) ~/.ssh/id_rsa
    chmod 600 ~/.ssh/id_rsa
    
    # Add known hosts (prevent MITM)
    ssh-keyscan github.com >> ~/.ssh/known_hosts
  displayName: 'Configure SSH'

# Use SSH key for deployment
- script: |
    ssh user@production-server 'bash -s' < deploy.sh
  displayName: 'Deploy to Production'
```

---

## Security Best Practices

### 1. Principle of Least Privilege
**Guideline**: Grant secure file access only to pipelines that require it.

```yaml
# ❌ Anti-pattern: All pipelines have access
# (Default if you don't configure pipeline permissions)

# ✅ Best practice: Explicit authorization
# iOS-Build-Pipeline: Access to ios-distribution.p12 ✅
# iOS-Release-Pipeline: Access to ios-distribution.p12 ✅
# Android-Build-Pipeline: No access to iOS cert ❌
```

### 2. Separate Certificates by Environment
**Guideline**: Use different certificates for dev/staging/prod.

```
Secure Files Library:
├── ios-dev-certificate.p12 (dev signing)
├── ios-staging-certificate.p12 (staging/beta testing)
└── ios-distribution-certificate.p12 (production App Store)

Pipeline Authorization:
├── Dev-Build-Pipeline → ios-dev-certificate.p12
├── Staging-Build-Pipeline → ios-staging-certificate.p12
└── Production-Build-Pipeline → ios-distribution-certificate.p12
```

### 3. Rotate Certificates Regularly
**Guideline**: Update certificates before expiration, maintain version history.

```bash
# Certificate rotation workflow
1. Generate new certificate
2. Upload to Secure Files with versioned name:
   - ios-distribution-2026.p12 (new)
   - ios-distribution-2025.p12 (old, keep for rollback)
3. Update pipeline to reference new certificate
4. Test deployment with new certificate
5. Delete old certificate after grace period (30 days)
```

### 4. Audit Secure File Access
**Guideline**: Enable audit logging, review access patterns.

```bash
# View secure file access logs (Azure DevOps Portal)
# Navigate to: Project Settings > Auditing
# Filter by: Resource Type = "Secure File"

# Audit log entries:
2026-01-14 10:30 - user@company.com uploaded "ios-distribution.p12"
2026-01-14 10:35 - iOS-Build-Pipeline downloaded "ios-distribution.p12"
2026-01-14 11:00 - user@company.com granted access to iOS-Release-Pipeline
2026-01-14 11:15 - iOS-Release-Pipeline downloaded "ios-distribution.p12"
```

### 5. Combine with Key Vault for Passwords
**Guideline**: Store certificate files in Secure Files, passwords in Key Vault.

```yaml
# Secure Files: Binary certificate (ios-distribution.p12)
# Key Vault: Certificate password

variables:
- group: 'ios-secrets'  # Variable group linked to Key Vault
  # Contains: CertificatePassword (from Key Vault secret)

steps:
- task: DownloadSecureFile@1
  name: certificate
  inputs:
    secureFile: 'ios-distribution.p12'  ← Secure Files

- task: InstallAppleCertificate@2
  inputs:
    certSecureFile: 'ios-distribution.p12'
    certPwd: $(CertificatePassword)  ← Key Vault via variable group
```

**Why Separate**:
- ✅ Certificate file rarely changes (annual rotation)
- ✅ Password can be rotated independently (quarterly)
- ✅ Principle of least privilege (different access controls)

---

## Comparison: Secure Files vs Alternatives

### Secure Files vs Git Repository

| Aspect | Secure Files | Git Repository |
|--------|-------------|----------------|
| **Security** | Encrypted at rest, access controlled | Plain text (even if private repo) |
| **Audit Trail** | Access logs (who downloaded, when) | Commit history (who committed) |
| **Rotation** | Replace file, immediate effect | Commit new file, redeploy |
| **Access Control** | Pipeline-specific permissions | Repository permissions (coarse) |
| **Best For** | Sensitive files (certificates, keys) | Non-sensitive config files |

**Rule**: Never commit certificates or private keys to Git (even private repos).

### Secure Files vs Key Vault

| Aspect | Secure Files | Key Vault |
|--------|-------------|-----------|
| **Data Type** | Binary files | Text secrets, keys, certificates |
| **Access Pattern** | Download to agent disk | Reference as pipeline variable |
| **Rotation** | Manual replacement | Automated rotation (secrets, certs) |
| **Cost** | Free (included in Azure DevOps) | ~$0.03 per 10,000 operations |
| **Use Case** | Code signing, provisioning profiles | API keys, passwords, connection strings |

**Guideline**: 
- Binary files (certificates for signing) → Secure Files
- Text secrets (passwords, API keys) → Key Vault
- Certificate management (rotation, expiry) → Key Vault

---

## Real-World Example: Mobile App CI/CD

### Scenario
**Company**: Mobile gaming company  
**Apps**: iOS and Android versions  
**Requirement**: Automated build and release to app stores  

### Implementation

**Secure Files**:
```
Library > Secure files:
├── ios-app-store-cert.p12 (Apple distribution certificate)
├── ios-app-store-profile.mobileprovision (provisioning profile)
├── android-release-keystore.jks (Android signing keystore)
└── google-play-service-account.json (Play Store API credentials)
```

**Pipeline**:
```yaml
# iOS Pipeline
trigger:
  branches:
    include:
    - release/*

pool:
  vmImage: 'macOS-latest'

variables:
- group: 'mobile-app-secrets'  # Key Vault variables

stages:
- stage: Build
  jobs:
  - job: BuildiOS
    steps:
    # Download secure files
    - task: DownloadSecureFile@1
      name: iosCert
      inputs:
        secureFile: 'ios-app-store-cert.p12'
    
    - task: DownloadSecureFile@1
      name: iosProfile
      inputs:
        secureFile: 'ios-app-store-profile.mobileprovision'
    
    # Install certificates
    - task: InstallAppleCertificate@2
      inputs:
        certSecureFile: 'ios-app-store-cert.p12'
        certPwd: $(IosCertificatePassword)  # From Key Vault
    
    - task: InstallAppleProvisioningProfile@1
      inputs:
        provProfileSecureFile: 'ios-app-store-profile.mobileprovision'
    
    # Build and sign
    - task: Xcode@5
      inputs:
        actions: 'build'
        configuration: 'Release'
        signingOption: 'manual'
    
    # Upload to App Store
    - task: AppStoreRelease@1
      inputs:
        serviceEndpoint: 'App Store Connection'
        appIdentifier: 'com.company.game'
        ipaPath: '$(Build.ArtifactStagingDirectory)/*.ipa'
```

**Security Benefits**:
- ✅ No certificates in source control
- ✅ Pipeline-specific access (iOS pipeline can't access Android keystore)
- ✅ Audit trail (who downloaded certificates, when)
- ✅ Centralized rotation (update certificate once, affects all branches)

---

## Troubleshooting

### Issue 1: Pipeline Can't Download Secure File
```
Error: Unable to download secure file 'ios-distribution.p12'. 
The file may not exist or the pipeline may not have permission.
```

**Solution**:
```
1. Verify secure file exists in Library > Secure files
2. Check pipeline permissions:
   - Navigate to secure file
   - Click "Pipeline permissions"
   - Ensure pipeline is authorized
3. Grant access if missing
```

### Issue 2: Certificate Installation Fails
```
Error: The certificate could not be installed. 
The password may be incorrect.
```

**Solution**:
```yaml
# Verify password variable is correctly configured
variables:
- group: 'ios-secrets'  # Ensure variable group linked to Key Vault

# Check Key Vault secret name matches variable reference
# Key Vault secret: IosCertificatePassword
# Pipeline variable: $(IosCertificatePassword)  ← Must match exactly (case-sensitive)
```

### Issue 3: Secure File Path Not Found
```
Error: File not found: $(certificate.secureFilePath)
```

**Solution**:
```yaml
# Ensure 'name' attribute matches variable reference
- task: DownloadSecureFile@1
  name: certificate  ← This name
  inputs:
    secureFile: 'cert.p12'

- script: |
    echo $(certificate.secureFilePath)  ← Must match name above
```

---

## Key Takeaways

### When to Use Secure Files
✅ Binary file assets (certificates, provisioning profiles, keystores)  
✅ Files needed during build/release (code signing)  
✅ Sensitive files requiring encryption and access control  
✅ Files that rarely change (annual certificate rotation)  

### Security Best Practices
✅ **Least privilege**: Authorize only required pipelines  
✅ **Separation**: Store passwords in Key Vault, files in Secure Files  
✅ **Rotation**: Update certificates before expiration  
✅ **Audit**: Enable and review access logs  
✅ **Environment isolation**: Separate dev/staging/prod certificates  

### Pipeline Integration
✅ Download with `DownloadSecureFile@1` task  
✅ Reference with `$(name.secureFilePath)` variable  
✅ Combine with Key Vault for passwords  
✅ Install certificates/profiles with platform-specific tasks  

---

**Next**: Learn about Azure App Configuration for centralized configuration management →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/5-implement-azure-devops-secure-files)
