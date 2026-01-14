# Manage Secrets, Tokens, and Certificates

**Duration**: 6 minutes

Comprehensive lifecycle management of sensitive data (secrets, tokens, certificates) across development, deployment, and production environments, focusing on secure storage, access control, rotation, and audit compliance.

---

## Categories of Sensitive Data

### 1. Secrets
**Definition**: Sensitive strings providing authentication or authorization.

**Examples**:
- Database passwords
- API keys
- Connection strings
- Encryption keys
- Service account passwords

### 2. Tokens
**Definition**: Time-limited credentials granting temporary access.

**Examples**:
- OAuth 2.0 access tokens
- JWT (JSON Web Tokens)
- Personal Access Tokens (PAT)
- SAS tokens (Shared Access Signatures)
- Session tokens

### 3. Certificates
**Definition**: Digital documents proving identity and enabling encrypted communication.

**Examples**:
- SSL/TLS certificates (HTTPS)
- Code signing certificates
- Client authentication certificates
- S/MIME certificates (email encryption)
- IoT device certificates

---

## Secret Lifecycle Management

### 1. Creation

**Manual Creation (Azure Key Vault)**:
```bash
# Create secret
az keyvault secret set \
  --vault-name prod-keyvault \
  --name DatabasePassword \
  --value 'P@ssw0rd123!' \
  --expires '2027-01-14T00:00:00Z'

# Create secret with content type
az keyvault secret set \
  --vault-name prod-keyvault \
  --name ApiKey \
  --value 'sk_live_abc123' \
  --content-type 'application/api-key'

# Create multi-line secret (certificate PEM)
az keyvault secret set \
  --vault-name prod-keyvault \
  --name SslCertificate \
  --file certificate.pem \
  --encoding utf-8
```

**Programmatic Generation**:
```csharp
// Generate strong password
using System.Security.Cryptography;

public static string GenerateSecurePassword(int length = 32)
{
    const string validChars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890!@#$%^&*";
    var password = new char[length];
    
    using (var rng = RandomNumberGenerator.Create())
    {
        var randomBytes = new byte[length];
        rng.GetBytes(randomBytes);
        
        for (int i = 0; i < length; i++)
        {
            password[i] = validChars[randomBytes[i] % validChars.Length];
        }
    }
    
    return new string(password);
}

// Store in Key Vault
var client = new SecretClient(new Uri("https://prod-keyvault.vault.azure.net"), new DefaultAzureCredential());
await client.SetSecretAsync("DatabasePassword", GenerateSecurePassword());
```

### 2. Storage

**Azure Key Vault (Recommended)**:
```bash
# Secrets stored with:
# - 256-bit AES encryption
# - HSM-backed (Premium tier)
# - FIPS 140-2 Level 2 compliance
# - Access audit logs

az keyvault create \
  --name prod-keyvault \
  --resource-group myapp-rg \
  --location eastus \
  --sku premium  # HSM-backed
```

**Azure DevOps Secure Files**:
```yaml
# Upload certificate via Azure DevOps UI: Pipelines → Library → Secure Files

# Use in pipeline
- task: DownloadSecureFile@1
  name: sslCert
  inputs:
    secureFile: 'ssl-certificate.pfx'

- script: |
    echo "Certificate path: $(sslCert.secureFilePath)"
    openssl pkcs12 -in $(sslCert.secureFilePath) -noout -info
```

**Environment Variables (Runtime Only)**:
```bash
# ✅ Good: Set at runtime (not persisted)
export DB_PASSWORD=$(az keyvault secret show --vault-name prod-keyvault --name DatabasePassword --query value -o tsv)

# ❌ Bad: Hardcoded in script
export DB_PASSWORD="P@ssw0rd123!"  # Never do this
```

### 3. Retrieval

**Application Code**:
```csharp
// .NET - Azure SDK
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

var client = new SecretClient(
    new Uri("https://prod-keyvault.vault.azure.net"),
    new DefaultAzureCredential()  // Managed identity
);

KeyVaultSecret secret = await client.GetSecretAsync("DatabasePassword");
string password = secret.Value;

// Use password securely
using (var connection = new SqlConnection($"Server=myserver;Database=mydb;User Id=admin;Password={password}"))
{
    // Database operations
}
```

**Azure Pipelines**:
```yaml
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'DatabasePassword,ApiKey'

- script: |
    echo "Password retrieved: $(DatabasePassword)"  # Masked in logs
```

**Azure CLI**:
```bash
# Get secret value
SECRET=$(az keyvault secret show \
  --vault-name prod-keyvault \
  --name DatabasePassword \
  --query value -o tsv)

# Use in command
mysql -h myserver.com -u admin -p"$SECRET" mydb
```

### 4. Rotation

**Manual Rotation**:
```bash
# Create new version (old versions preserved)
az keyvault secret set \
  --vault-name prod-keyvault \
  --name DatabasePassword \
  --value 'NewP@ssw0rd456!'

# Applications automatically use latest version (by default)
```

**Automated Rotation (Event Grid + Function)**:
```csharp
// Azure Function triggered 30 days before expiration
[FunctionName("RotateSecret")]
public static async Task Run(
    [EventGridTrigger] EventGridEvent eventGridEvent,
    ILogger log)
{
    // 1. Generate new secret
    var newPassword = GenerateSecurePassword();
    
    // 2. Update target system (database, API, etc.)
    await UpdateDatabasePassword(newPassword);
    
    // 3. Store new version in Key Vault
    var client = new SecretClient(new Uri("https://prod-keyvault.vault.azure.net"), new DefaultAzureCredential());
    await client.SetSecretAsync("DatabasePassword", newPassword);
    
    log.LogInformation("Secret rotated successfully");
}
```

**Rotation Policy (Key Vault Feature)**:
```bash
# Set rotation policy (auto-rotate before expiration)
az keyvault secret rotation-policy update \
  --vault-name prod-keyvault \
  --name DatabasePassword \
  --value '{
    "lifetimeActions": [
      {
        "trigger": {"timeBeforeExpiry": "P30D"},
        "action": {"type": "Rotate"}
      }
    ],
    "attributes": {"expiryTime": "P90D"}
  }'
```

### 5. Revocation

**Disable Secret Version**:
```bash
# Disable specific version (applications fail if using this version)
az keyvault secret set-attributes \
  --vault-name prod-keyvault \
  --name DatabasePassword \
  --version abc123def456 \
  --enabled false

# Applications using "latest" unaffected
```

**Delete Secret (Soft Delete Enabled)**:
```bash
# Soft delete (recoverable for 90 days)
az keyvault secret delete \
  --vault-name prod-keyvault \
  --name DatabasePassword

# Recover deleted secret
az keyvault secret recover \
  --vault-name prod-keyvault \
  --name DatabasePassword

# Purge (permanent deletion after soft delete)
az keyvault secret purge \
  --vault-name prod-keyvault \
  --name DatabasePassword
```

### 6. Auditing

**Enable Diagnostic Logging**:
```bash
az monitor diagnostic-settings create \
  --name keyvault-audit \
  --resource /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault \
  --logs '[
    {
      "category": "AuditEvent",
      "enabled": true,
      "retentionPolicy": {"enabled": true, "days": 90}
    }
  ]' \
  --workspace /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.OperationalInsights/workspaces/myapp-logs
```

**Query Access Logs**:
```kusto
// Log Analytics query
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.KEYVAULT"
| where OperationName == "SecretGet"
| project TimeGenerated, CallerIPAddress, identity_claim_upn_s, SecretName = id_s, ResultType
| order by TimeGenerated desc
```

---

## Token Management

### Personal Access Tokens (PAT)

**Azure DevOps PAT Lifecycle**:
```bash
# Create PAT (Azure DevOps UI or API)
# Settings → Personal Access Tokens → New Token
# - Name: Pipeline-Integration
# - Expiration: 90 days
# - Scopes: Code (Read), Build (Read & Execute)

# Store in Key Vault
az keyvault secret set \
  --vault-name prod-keyvault \
  --name AzureDevOpsPAT \
  --value 'abcd1234efgh5678ijkl' \
  --expires '2026-04-14T00:00:00Z'

# Use in pipeline
- task: AzureKeyVault@2
  inputs:
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'AzureDevOpsPAT'

- script: |
    git clone https://$(AzureDevOpsPAT)@dev.azure.com/myorg/myproject/_git/myrepo
```

**GitHub PAT Management**:
```bash
# Create GitHub PAT: Settings → Developer settings → Personal access tokens → Generate new token

# Store in Key Vault
az keyvault secret set \
  --vault-name prod-keyvault \
  --name GitHubPAT \
  --value 'ghp_abc123xyz789'

# Use in GitHub Actions
# (Key Vault integration via Azure Login action)
```

**PAT Rotation Best Practices**:
- Set 90-day expiration (force rotation)
- Create overlapping tokens (old + new active for transition period)
- Automate rotation with Event Grid + Function
- Revoke immediately on user offboarding

### OAuth 2.0 Access Tokens

**Token Acquisition**:
```csharp
// Azure AD OAuth 2.0 client credentials flow
using Microsoft.Identity.Client;

var app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)  // From Key Vault
    .WithAuthority(new Uri($"https://login.microsoftonline.com/{tenantId}"))
    .Build();

var result = await app.AcquireTokenForClient(new[] { "https://graph.microsoft.com/.default" }).ExecuteAsync();
string accessToken = result.AccessToken;  // Short-lived (1 hour)

// Use token in API request
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
```

**Token Caching**:
```csharp
// Cache token in-memory (expires in 1 hour)
public class TokenCache
{
    private static string _cachedToken;
    private static DateTime _expiration;
    
    public static async Task<string> GetTokenAsync()
    {
        if (string.IsNullOrEmpty(_cachedToken) || DateTime.UtcNow >= _expiration)
        {
            // Acquire new token
            var result = await AcquireTokenAsync();
            _cachedToken = result.AccessToken;
            _expiration = result.ExpiresOn.UtcDateTime;
        }
        
        return _cachedToken;
    }
}
```

### SAS Tokens (Azure Storage)

**Generate SAS Token**:
```bash
# Account-level SAS (access multiple services)
az storage account generate-sas \
  --account-name mystorageaccount \
  --services b  # Blob service \
  --resource-types sco  # Service, Container, Object \
  --permissions rwdlac  # Read, Write, Delete, List, Add, Create \
  --expiry 2026-01-14T00:00:00Z \
  --https-only \
  --output tsv

# Output: se=2026-01-14T00:00:00Z&sp=rwdlac&sv=2021-06-08&ss=b&srt=sco&sig=...
```

**Store and Use SAS Token**:
```bash
# Store in Key Vault
az keyvault secret set \
  --vault-name prod-keyvault \
  --name StorageSasToken \
  --value 'se=2026-01-14&sp=rwdlac&...'

# Use in application
SAS_TOKEN=$(az keyvault secret show --vault-name prod-keyvault --name StorageSasToken --query value -o tsv)
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./myfile.txt \
  --sas-token "$SAS_TOKEN"
```

---

## Certificate Management

### Certificate Lifecycle

#### 1. Creation/Import

**Generate Self-Signed Certificate**:
```bash
# Create self-signed certificate in Key Vault
az keyvault certificate create \
  --vault-name prod-keyvault \
  --name ssl-certificate \
  --policy '{
    "issuerParameters": {"name": "Self"},
    "keyProperties": {"exportable": true, "keySize": 2048, "keyType": "RSA"},
    "secretProperties": {"contentType": "application/x-pkcs12"},
    "x509CertificateProperties": {
      "subject": "CN=myapp.com",
      "validityInMonths": 12,
      "subjectAlternativeNames": {"dnsNames": ["myapp.com", "www.myapp.com"]}
    }
  }'
```

**Import Existing Certificate**:
```bash
# Import PFX/PEM certificate
az keyvault certificate import \
  --vault-name prod-keyvault \
  --name ssl-certificate \
  --file certificate.pfx \
  --password 'pfx-password'
```

#### 2. Storage

**Key Vault Certificate Storage**:
- Stored as certificate object (not secret)
- Includes private key (encrypted)
- Automatic version management
- Supports PEM and PFX formats

```bash
# List certificates
az keyvault certificate list --vault-name prod-keyvault

# Get certificate details
az keyvault certificate show \
  --vault-name prod-keyvault \
  --name ssl-certificate
```

#### 3. Retrieval

**Download Certificate**:
```bash
# Download certificate (PEM format)
az keyvault certificate download \
  --vault-name prod-keyvault \
  --name ssl-certificate \
  --file ssl-certificate.pem

# Download as secret (includes private key for PFX)
az keyvault secret download \
  --vault-name prod-keyvault \
  --name ssl-certificate \
  --encoding base64 \
  --file ssl-certificate.pfx
```

**Use in Pipeline**:
```yaml
# Download and install certificate
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: 'ssl-certificate'

- task: InstallAppleCertificate@2  # iOS signing
  inputs:
    certSecureFile: 'ssl-certificate'
    certPwd: '$(ssl-certificate)'  # Retrieved from Key Vault
    keychain: 'temp'
```

#### 4. Renewal

**Manual Renewal**:
```bash
# Create new version (CA-issued certificate)
az keyvault certificate import \
  --vault-name prod-keyvault \
  --name ssl-certificate \
  --file renewed-certificate.pfx \
  --password 'pfx-password'
```

**Automatic Renewal (Integrated CA)**:
```bash
# Set up automatic renewal with Let's Encrypt or DigiCert
az keyvault certificate create \
  --vault-name prod-keyvault \
  --name ssl-certificate \
  --policy '{
    "issuerParameters": {"name": "DigiCert"},
    "lifetimeActions": [
      {
        "trigger": {"lifetimePercentage": 80},
        "action": {"actionType": "AutoRenew"}
      }
    ],
    "x509CertificateProperties": {
      "subject": "CN=myapp.com",
      "validityInMonths": 12
    }
  }'

# Key Vault contacts DigiCert automatically when 80% of validity reached
```

#### 5. Expiration Monitoring

**Alert on Certificate Expiration**:
```bash
# Create Azure Monitor alert
az monitor metrics alert create \
  --name certificate-expiry-alert \
  --resource-group myapp-rg \
  --scopes /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault \
  --condition "avg DaysToExpiry < 30" \
  --description "Certificate expires in less than 30 days" \
  --evaluation-frequency 1d \
  --window-size 1d \
  --action email admin@company.com
```

**Check Expiration Programmatically**:
```csharp
var client = new CertificateClient(new Uri("https://prod-keyvault.vault.azure.net"), new DefaultAzureCredential());
var certificate = await client.GetCertificateAsync("ssl-certificate");

var daysUntilExpiry = (certificate.Value.Properties.ExpiresOn.Value - DateTimeOffset.UtcNow).Days;

if (daysUntilExpiry < 30)
{
    // Send alert
    await SendExpirationAlertAsync(certificate.Value.Name, daysUntilExpiry);
}
```

---

## Access Control

### Role-Based Access Control (RBAC)

**Key Vault RBAC Roles**:
```bash
# Enable RBAC authorization
az keyvault update \
  --name prod-keyvault \
  --enable-rbac-authorization true

# Assign roles
az role assignment create \
  --assignee user@company.com \
  --role "Key Vault Secrets Officer" \
  --scope /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault

# Common roles:
# - Key Vault Secrets User (read secrets)
# - Key Vault Secrets Officer (manage secrets)
# - Key Vault Certificates Officer (manage certificates)
# - Key Vault Administrator (full access)
```

### Access Policies (Legacy, Still Supported)

```bash
# Grant secret read permissions
az keyvault set-policy \
  --name prod-keyvault \
  --object-id {user-object-id} \
  --secret-permissions get list

# Grant certificate management permissions
az keyvault set-policy \
  --name prod-keyvault \
  --object-id {user-object-id} \
  --certificate-permissions get list create import update delete
```

---

## Security Best Practices

### 1. Never Hardcode Secrets
```csharp
// ❌ BAD: Hardcoded connection string
var connectionString = "Server=myserver;Database=mydb;User Id=admin;Password=P@ssw0rd123!";

// ✅ GOOD: Retrieve from Key Vault
var client = new SecretClient(new Uri("https://prod-keyvault.vault.azure.net"), new DefaultAzureCredential());
var connectionString = (await client.GetSecretAsync("DatabaseConnectionString")).Value.Value;
```

### 2. Use Managed Identities
```csharp
// ❌ BAD: Client secret authentication (another secret to manage)
var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);

// ✅ GOOD: Managed identity (no secrets)
var credential = new DefaultAzureCredential();

var client = new SecretClient(new Uri("https://prod-keyvault.vault.azure.net"), credential);
```

### 3. Implement Secret Rotation
```bash
# ✅ Set expiration dates on all secrets
az keyvault secret set \
  --vault-name prod-keyvault \
  --name ApiKey \
  --value 'sk_live_abc123' \
  --expires '2026-04-14T00:00:00Z'

# ❌ Secrets without expiration = forgotten and never rotated
```

### 4. Enable Soft Delete and Purge Protection
```bash
# Enable soft delete (90-day recovery window)
az keyvault update \
  --name prod-keyvault \
  --enable-soft-delete true \
  --retention-days 90

# Enable purge protection (prevent permanent deletion)
az keyvault update \
  --name prod-keyvault \
  --enable-purge-protection true
```

### 5. Audit and Monitor
```bash
# Enable diagnostic logging
az monitor diagnostic-settings create \
  --name keyvault-audit \
  --resource /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault \
  --logs '[{"category":"AuditEvent","enabled":true}]' \
  --workspace /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.OperationalInsights/workspaces/myapp-logs

# Review access regularly
az keyvault show --name prod-keyvault --query properties.accessPolicies
```

---

## Real-World Example: Complete Secret Lifecycle

### Scenario: E-Commerce Application Database Password

**1. Initial Setup**:
```bash
# Create Key Vault
az keyvault create --name myapp-keyvault --resource-group myapp-rg --location eastus

# Generate strong password
PASSWORD=$(openssl rand -base64 32)

# Store in Key Vault with 90-day expiration
az keyvault secret set \
  --vault-name myapp-keyvault \
  --name DatabasePassword \
  --value "$PASSWORD" \
  --expires "2026-04-14T00:00:00Z"

# Update database with new password
mysql -h myserver.com -u admin -p"$PASSWORD" -e "ALTER USER 'admin'@'%' IDENTIFIED BY '$PASSWORD';"
```

**2. Application Retrieves Secret**:
```csharp
// Application code (managed identity authentication)
var client = new SecretClient(new Uri("https://myapp-keyvault.vault.azure.net"), new DefaultAzureCredential());
var password = (await client.GetSecretAsync("DatabasePassword")).Value.Value;

using var connection = new SqlConnection($"Server=myserver.com;Database=myapp;User Id=admin;Password={password}");
await connection.OpenAsync();
```

**3. Automated Rotation (Azure Function)**:
```csharp
// Triggered 30 days before expiration (Event Grid event)
[FunctionName("RotateDatabasePassword")]
public static async Task Run([EventGridTrigger] EventGridEvent eventGridEvent)
{
    // 1. Generate new password
    var newPassword = GenerateSecurePassword(32);
    
    // 2. Update database
    await UpdateDatabasePasswordAsync(newPassword);
    
    // 3. Store new version in Key Vault
    var client = new SecretClient(new Uri("https://myapp-keyvault.vault.azure.net"), new DefaultAzureCredential());
    await client.SetSecretAsync("DatabasePassword", newPassword, new DateTimeOffset(DateTime.UtcNow.AddDays(90)));
    
    // 4. Log rotation
    _logger.LogInformation("Database password rotated successfully");
}
```

**4. Monitoring and Alerts**:
```bash
# Alert 30 days before expiration
az monitor metrics alert create \
  --name database-password-expiry \
  --resource-group myapp-rg \
  --scopes /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/myapp-keyvault \
  --condition "avg DaysToExpiry < 30" \
  --action email devops@company.com
```

**Result**: Zero downtime, automated rotation every 90 days, full audit trail.

---

## Key Takeaways

✅ **Lifecycle phases**: Creation → Storage → Retrieval → Rotation → Revocation → Auditing  
✅ **Azure Key Vault**: Centralized storage for secrets, tokens, certificates  
✅ **Secret rotation**: Automate with Event Grid + Azure Functions (30-90 day cadence)  
✅ **Token management**: PATs (90-day expiration), OAuth 2.0 (1-hour lifetime), SAS tokens (time-bound)  
✅ **Certificate renewal**: Automatic with integrated CAs (Let's Encrypt, DigiCert)  
✅ **Access control**: RBAC (recommended) or access policies  
✅ **Best practices**: Managed identities, soft delete, purge protection, audit logging  

---

**Next**: Explore DevOps inner and outer loop workflows →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/10-manage-secrets-tokens-certificates)
