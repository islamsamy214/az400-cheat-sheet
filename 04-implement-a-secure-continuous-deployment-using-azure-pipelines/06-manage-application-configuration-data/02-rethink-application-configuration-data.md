# Rethink Application Configuration Data

**Duration**: 5 minutes

Traditional configuration management practices create deployment friction, security vulnerabilities, and operational complexity. This unit examines configuration anti-patterns and modern approaches for cloud-native applications.

---

## The Configuration Problem

### Traditional Configuration File Approach
**Pattern**: Embed configuration within deployment-bundled files (app.config, web.config, appsettings.json) enabling post-deployment behavior modification through file editing.

**Why It Was Popular**:
- Simple to understand (XML/JSON files alongside code)
- Framework support (.NET app.config, Spring application.properties)
- Local development convenience (change file, restart app)

**Why It's Problematic**:
```
Application Package
├── MyApp.dll
├── appsettings.json  ← Configuration bundled with code
│   ├── ConnectionStrings
│   │   └── Database: "Server=prod-db;Password=SuperSecret123"  ❌
│   ├── ApiKeys
│   │   └── PaymentGateway: "sk-live-abc123def456"  ❌
│   └── FeatureFlags
│       └── EnableBetaFeatures: true
```

**Critical Flaws**:
1. **Secrets in source control**: Configuration files committed to Git expose credentials
2. **Redeployment required**: Configuration changes require full application redeployment
3. **Environment coupling**: Separate config files needed per environment (dev/staging/prod)
4. **Instance synchronization**: Multi-instance deployments have temporary configuration inconsistencies
5. **No audit trail**: Configuration changes lack versioning and audit logs
6. **Shared config duplication**: Common settings (database URLs) duplicated across apps

---

## Configuration Anti-Patterns

### Anti-Pattern 1: Hardcoded Secrets
```csharp
// ❌ NEVER DO THIS - Security vulnerability
public class PaymentService
{
    private const string ApiKey = "sk-live-1234567890abcdef";
    private const string DatabasePassword = "SuperSecret123!";
    
    public void ProcessPayment()
    {
        var connection = new SqlConnection(
            $"Server=prod-db;Database=orders;Password={DatabasePassword}"
        );
    }
}
```

**Why It's Dangerous**:
- Secrets visible in compiled binaries (decompilation risk)
- Credential rotation requires code redeployment
- No differentiation between dev/prod environments
- Secret scanning tools may not detect compiled secrets

**Real-World Impact**: 
- **GitHub Secret Scanning** detects 2+ million secrets leaked annually
- **Average breach cost**: $4.45M (IBM 2023 Cost of Data Breach Report)
- **Time to exploit**: 48 hours average after credential exposure

### Anti-Pattern 2: Configuration Files in Source Control
```bash
# Git repository structure
my-app/
├── src/
│   └── PaymentService.cs
├── appsettings.Development.json  ✅ Safe (no secrets)
├── appsettings.Staging.json      ❌ Contains staging DB password
├── appsettings.Production.json   ❌ Contains production secrets
└── .gitignore                     ⚠️ Often incomplete
```

**Common .gitignore Oversight**:
```gitignore
# .gitignore (incomplete)
*.log
*.suo
bin/
obj/
# Missing: appsettings.Production.json ← Accidentally committed!
```

**The 2AM Deployment Scenario**:
```
Developer workflow at 2:00 AM:
1. ✅ Write code, run tests locally
2. ✅ Commit code: git add src/
3. ❌ Accidentally commit secrets: git add appsettings.Production.json
4. ✅ Push to GitHub: git push origin main
5. 💥 Production credentials now in public repository
6. ⏰ Credential compromise detected 48 hours later
```

**Why .gitignore Isn't Enough**:
- Human error during late-night deployments
- Incomplete .gitignore templates
- Secret already committed before .gitignore added (history remains)
- Forks and mirrors contain leaked secrets indefinitely

### Anti-Pattern 3: Environment-Specific Builds
```yaml
# ❌ Anti-pattern: Build different artifacts per environment
name: Multi-Environment Build

trigger:
  - main

jobs:
- job: BuildDev
  steps:
  - script: dotnet publish --configuration Dev
  - task: CopyFiles@2
    inputs:
      Contents: 'appsettings.Development.json'

- job: BuildProduction
  steps:
  - script: dotnet publish --configuration Release
  - task: CopyFiles@2
    inputs:
      Contents: 'appsettings.Production.json'  ← Different artifact!
```

**Problems**:
- ❌ Different artifacts for dev vs prod (what you test ≠ what you deploy)
- ❌ Build matrix explosion (dev × staging × prod × canary × dr = 5 builds)
- ❌ Configuration drift between environments undetected until production
- ❌ Violates continuous delivery principle: "Build once, deploy many"

**Better Approach**:
```yaml
# ✅ Best practice: Build once, configure at runtime
- script: dotnet publish --configuration Release  ← Single artifact
- task: AzureAppConfiguration@5  ← Inject config at deployment time
  inputs:
    azureSubscription: 'prod-service-connection'
    AppConfigurationName: 'myapp-config'
    KeyFilter: '*'
    Label: $(Environment)  ← dev, staging, prod
```

### Anti-Pattern 4: Configuration Duplication Across Services
```
# Microservices architecture with duplicated config
order-service/
└── appsettings.json
    └── DatabaseUrl: "postgres://prod-db:5432/orders"  ← Duplicated

payment-service/
└── appsettings.json
    └── DatabaseUrl: "postgres://prod-db:5432/orders"  ← Duplicated

shipping-service/
└── appsettings.json
    └── DatabaseUrl: "postgres://prod-db:5432/orders"  ← Duplicated
```

**Operational Nightmare**:
- **Database migration scenario**: Change connection string in 50 microservices
- **Certificate rotation**: Update SSL certificates in 50 config files
- **Configuration drift**: Some services updated, others missed (inconsistent state)
- **Deployment window**: 50 services × 10 minutes = 8+ hours of rolling updates

**Centralized Alternative**:
```
[Azure App Configuration]  ← Single source of truth
        │
        ├─ Shared:DatabaseUrl
        ├─ Shared:PaymentGatewayUrl
        └─ Shared:CertificateThumbprint
        │
        ├── [Order Service]   ─┐
        ├── [Payment Service] ─┼─ Poll for config changes
        └── [Shipping Service]─┘
        
Update DatabaseUrl once → All services refresh within 30 seconds
```

---

## Example Scenario: The Late-Night Deployment Disaster

### The Setup
**Context**: E-commerce startup deploying Black Friday performance optimizations at 2:00 AM before traffic surge.

**Team**: 
- Alice (Senior Developer) - working 18-hour shift
- Bob (DevOps Engineer) - on-call, sleep-deprived

### The Incident Timeline

**1:45 AM - Final Testing**
```bash
# Alice runs final tests in staging environment
dotnet test --configuration Staging  ✅ All tests pass
git add src/ tests/
git commit -m "Performance optimizations for Black Friday"
```

**2:00 AM - The Fateful Commit**
```bash
# Alice accidentally includes production config (mental exhaustion)
git add appsettings.Production.json  ❌ Contains DB password, API keys
git add appsettings.Staging.json     ❌ Contains staging credentials
git push origin main

# .gitignore was incomplete (missing *.Production.json pattern)
```

**2:05 AM - Deployment Succeeds**
```bash
# Automated pipeline deploys to production
Pipeline: ✅ Build successful
Pipeline: ✅ Tests pass
Pipeline: ✅ Deployment to 50 app instances complete
```

**2:30 AM - Black Friday Traffic Begins**
- Application handles traffic successfully
- No immediate issues detected
- Team goes to sleep after 18-hour shift

**Friday, 10:00 AM - Security Alert**
```
GitHub Secret Scanning Alert:
⚠️ Database password detected in commit a1b2c3d
⚠️ Stripe API key detected in commit a1b2c3d
Repository: mycompany/ecommerce-app (PUBLIC)
Severity: CRITICAL
```

**Friday, 10:15 AM - Damage Assessment**
- **Repository forked 47 times** while secrets were public
- **Commits cannot be removed** from forks (secrets persist indefinitely)
- **Database password exposed** for 8 hours during peak Black Friday traffic
- **API keys compromised**: $12,000 unauthorized charges on payment gateway

**Friday, 10:30 AM - Emergency Response**
```bash
# Rotate ALL compromised credentials during peak traffic day
1. Database password rotation → 5-minute downtime
2. API key regeneration → Payment processing halted
3. SSL certificate replacement → CDN cache flush required
4. Connection string updates → Rolling restart of 50 app instances
5. Customer notification (GDPR requirement) → Reputation damage

Total downtime: 47 minutes during Black Friday
Lost revenue: $380,000
Incident response cost: $85,000
```

### Root Causes
1. **Human error amplification**: Configuration file pattern encouraged credential commits
2. **Incomplete .gitignore**: Security responsibility on exhausted developers
3. **No automated secret scanning**: Pre-commit hooks could prevent push
4. **Secrets in files**: Configuration pattern fundamentally insecure

### Prevention: Modern Approach
```bash
# ✅ Configuration stored in Azure App Configuration (no secrets in repo)
# ✅ Secrets stored in Azure Key Vault (managed identities, no passwords)
# ✅ Pre-commit hooks with Yelp's detect-secrets tool
# ✅ Branch protection rules requiring secret scanning pass

# Developer workflow (safe)
git add src/ tests/  # Only code, no config files
git commit -m "Performance optimizations"
git push origin main

# Pipeline injects configuration at deployment
- task: AzureAppConfiguration@5
  inputs:
    azureSubscription: 'prod-service-connection'
    AppConfigurationName: 'myapp-config'
    KeyFilter: '*'
    Label: 'production'
    
# Secrets referenced from Key Vault (never in files)
# If credentials leaked → Rotate in Key Vault (zero downtime)
```

**Result**: Configuration leak impossible (nothing sensitive in Git repository)

---

## Historical Context: How We Got Here

### Phase 1: Early .NET Framework (2002-2010)
**.NET Framework introduced app.config and web.config**:
```xml
<!-- app.config (traditional approach) -->
<configuration>
  <connectionStrings>
    <add name="DefaultConnection" 
         connectionString="Server=localhost;Database=myapp;Password=secret"
         providerName="System.Data.SqlClient" />
  </connectionStrings>
  <appSettings>
    <add key="ApiKey" value="abc123def456" />
  </appSettings>
</configuration>
```

**Why It Made Sense Then**:
- Monolithic applications (single deployment)
- On-premises hosting (physical servers, manual configuration)
- Slow release cycles (quarterly/annual deployments)
- Configuration changes were rare (change required downtime anyway)

**What Went Wrong**:
- **Documentation proliferation**: MSDN examples showed secrets in config files
- **Encryption false security**: Obfuscated secrets with `aspnet_regiis` created illusion of safety
- **Cultural acceptance**: Developers interpreted config files as appropriate secret storage

**Obfuscation Example** (ineffective security):
```xml
<!-- Encrypted connection string (still in source control!) -->
<connectionStrings configProtectionProvider="RsaProtectedConfigurationProvider">
  <EncryptedData Type="http://www.w3.org/2001/04/xmlenc#Element"
    xmlns="http://www.w3.org/2001/04/xmlenc#">
    <CipherData>
      <CipherValue>AQAAANCMnd8BFdERjHoAwE/Cl+s...</CipherValue>  ❌ Committed to Git
    </CipherData>
  </EncryptedData>
</connectionStrings>
```
**Problem**: Encryption keys also in source control (security theater)

### Phase 2: Cloud Migration Era (2010-2015)
**Challenge**: Cloud environments with dynamic infrastructure, multiple instances, frequent deployments.

**New Problems**:
- **Multi-instance synchronization**: 10 app instances with stale config during rolling updates
- **Configuration drift**: Instances running different config versions simultaneously
- **Scale-out complexity**: Adding instances requires config file replication
- **Environment explosion**: dev × staging × prod × canary × dr = 5 config variants

**DevOps Tools Emergence**:
- **Chef/Puppet**: Infrastructure-as-code with configuration management
- **CI/CD pipelines**: Automated configuration injection during deployment
- **Environment variables**: Twelve-Factor App methodology

**Incremental Improvements**:
```bash
# Environment variable approach (better, but scattered)
export DATABASE_URL="postgres://prod-db:5432/myapp"
export API_KEY="abc123def456"
export FEATURE_FLAGS="beta-checkout:true,new-ui:false"

# Problems:
# - Configuration scattered across deployment scripts
# - No versioning or audit trail
# - Manual secret management
```

### Phase 3: Modern Cloud-Native Era (2015-Present)
**ASP.NET Core configuration improvements** (2016+):
```csharp
// Configuration providers with hierarchical override
var builder = WebApplication.CreateBuilder(args);

builder.Configuration
    .AddJsonFile("appsettings.json", optional: false)  ← Base config
    .AddJsonFile($"appsettings.{env}.json", optional: true)  ← Environment override
    .AddEnvironmentVariables()  ← Override from env vars
    .AddAzureAppConfiguration(options => {  ← ✅ Best: Centralized store
        options.Connect(connectionString)
               .Select(KeyFilter.Any, env)
               .ConfigureKeyVault(kv => kv.SetCredential(credential));
    });
```

**Modern Capabilities**:
- ✅ Configuration providers (multiple sources)
- ✅ Hierarchical overrides (base → environment → runtime)
- ✅ Secret management integration (Key Vault)
- ✅ Dynamic refresh (change config without restart)
- ✅ Feature flags (progressive rollouts)

**Remaining Opportunities**:
- 🔄 Eliminate file-based config entirely (centralized stores)
- 🔄 Automated secret rotation (zero-touch credential management)
- 🔄 Configuration-as-code (GitOps workflows)
- 🔄 Policy enforcement (prevent secret leakage)

---

## Modern DevOps Approaches

### 1. Infrastructure Automation (Chef, Puppet, Ansible)
**Capability**: Cross-language configuration orchestration with idempotent convergence.

**Chef Example**:
```ruby
# Chef cookbook for application configuration
node.default['myapp']['database_url'] = data_bag_item('secrets', 'db_url')
node.default['myapp']['api_key'] = data_bag_item('secrets', 'api_key')

template '/opt/myapp/config.json' do
  source 'config.json.erb'
  variables(
    database_url: node['myapp']['database_url'],
    api_key: node['myapp']['api_key']
  )
  sensitive true  # Prevents logging of secrets
end
```

**Improvements**:
- ✅ Centralized secret storage (Chef Vault, encrypted data bags)
- ✅ Auditable configuration changes (Chef Server logs)
- ✅ Idempotent convergence (desired state enforcement)

**Limitations**:
- ⚠️ Still file-based on target systems
- ⚠️ Configuration updates require Chef runs (not instant)
- ⚠️ Complex setup (Chef Server, Workstation, Nodes)

### 2. CI/CD Pipeline Value Injection
**Capability**: Inject configuration during deployment without storing in source control.

**Azure Pipelines Example**:
```yaml
# Pipeline with variable groups (secrets from Key Vault)
variables:
- group: production-secrets  # Linked to Key Vault

stages:
- stage: Deploy
  jobs:
  - deployment: DeployApp
    environment: production
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureRmWebAppDeployment@4
            inputs:
              appType: 'webApp'
              WebAppName: 'myapp-prod'
              AppSettings: |
                -DatabaseUrl $(DatabaseUrl)  ← From Key Vault
                -ApiKey $(ApiKey)             ← From Key Vault
                -Environment production
```

**Improvements**:
- ✅ Secrets never in source control (pipeline variables)
- ✅ Environment-specific configuration at deploy time
- ✅ Audit trail (pipeline runs logged)

**Limitations**:
- ⚠️ Configuration changes require redeployment
- ⚠️ Secrets visible in pipeline logs (must configure masking)
- ⚠️ No runtime configuration updates

### 3. Environment-Specific Configuration with ASP.NET Core
**Capability**: Multiple configuration providers with hierarchical override.

**Modern Configuration Pattern**:
```csharp
// Program.cs (ASP.NET Core 6+)
var builder = WebApplication.CreateBuilder(args);

// Configuration provider hierarchy (last wins)
builder.Configuration
    .AddJsonFile("appsettings.json")  // 1. Base configuration (no secrets)
    .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true)  // 2. Environment override
    .AddEnvironmentVariables()  // 3. Environment variables (Docker/Kubernetes)
    .AddAzureAppConfiguration(options => {  // 4. Centralized configuration (best)
        options.Connect(Environment.GetEnvironmentVariable("APPCONFIGURATION_CONNECTION_STRING"))
               .Select(KeyFilter.Any, builder.Environment.EnvironmentName)
               .ConfigureKeyVault(kv => {
                   // 5. Key Vault for secrets (managed identity auth)
                   kv.SetCredential(new DefaultAzureCredential());
               })
               .ConfigureRefresh(refresh => {
                   // 6. Dynamic refresh (no restart required)
                   refresh.Register("AppSettings:Sentinel", refreshAll: true)
                          .SetCacheExpiration(TimeSpan.FromSeconds(30));
               });
    });

var app = builder.Build();
```

**Configuration Hierarchy** (override precedence):
```
appsettings.json (base)
  └─ Overridden by → appsettings.Production.json (environment)
       └─ Overridden by → Environment Variables (container/host)
            └─ Overridden by → Azure App Configuration (centralized)
                 └─ Secrets from → Azure Key Vault (managed identity)
```

**Example Override**:
```json
// appsettings.json (base - committed to Git)
{
  "Logging": { "LogLevel": { "Default": "Information" } },
  "FeatureFlags": { "EnableBetaFeatures": false },
  "ApiEndpoint": "https://api.staging.example.com"  ← Dev/staging default
}

// Azure App Configuration (production label - NOT in Git)
{
  "ApiEndpoint": "https://api.example.com",  ← Production override
  "FeatureFlags:EnableBetaFeatures": true,
  "DatabaseUrl": "@Microsoft.KeyVault(SecretUri=https://mykv.vault.azure.net/secrets/DbUrl)"
}
```

**Benefits**:
- ✅ Base config in Git (non-sensitive, documented)
- ✅ Environment-specific config external (no commits)
- ✅ Secrets in Key Vault (never in code/config)
- ✅ Dynamic refresh (change config without restart)
- ✅ Single artifact promotes across environments

---

## The Twelve-Factor App - Config Factor

### Factor III: Store Config in the Environment
**Principle**: Strict separation between code (constant across deployments) and config (varies per deployment).

**Definition of Configuration**:
> "Everything that is likely to vary between deployments (staging, production, developer environments, etc.)."

**Configuration Includes**:
- Resource handles (database URLs, external service URLs)
- Credentials (passwords, API keys, certificates)
- Per-deploy values (canonical hostname, feature flags)

**Configuration Does NOT Include**:
- Application code (business logic, algorithms)
- Internal application config (routing, dependency injection)
- Framework defaults (ASP.NET Core options)

### Anti-Pattern: Constants in Code
```python
# ❌ Violates Twelve-Factor: Config as code
class PaymentService:
    API_KEY = "sk-live-abc123"  # Different per environment!
    DATABASE_URL = "postgres://prod-db:5432/orders"  # Different per environment!
```

### Twelve-Factor Approach: Environment Variables
```python
# ✅ Twelve-Factor compliant: Config from environment
import os

class PaymentService:
    API_KEY = os.environ['PAYMENT_API_KEY']
    DATABASE_URL = os.environ['DATABASE_URL']
```

**Why Environment Variables?**:
- Language-agnostic (works in Python, Node.js, .NET, Java, Go)
- Platform-agnostic (Linux, Windows, containers, serverless)
- No accidental commit risk (not in source control)
- Easy to change per deployment (Docker `-e`, Kubernetes ConfigMap)

### Environment Variable Limitations
**Problem 1: Configuration Sprawl**
```bash
# Deployment script with 50+ environment variables
export DATABASE_URL="postgres://prod-db:5432/myapp"
export CACHE_URL="redis://prod-cache:6379"
export API_KEY="abc123"
export FEATURE_FLAG_BETA="true"
export FEATURE_FLAG_NEW_CHECKOUT="false"
export LOG_LEVEL="info"
export SMTP_HOST="smtp.sendgrid.net"
export SMTP_USER="apikey"
export SMTP_PASS="SG.abc123..."
# ... 42 more variables
```
**Issue**: Hard to manage, no hierarchy, no versioning

**Problem 2: No Shared Configuration**
```bash
# Microservices architecture: Each service needs DATABASE_URL
kubectl set env deployment/order-service DATABASE_URL="postgres://..."
kubectl set env deployment/payment-service DATABASE_URL="postgres://..."
kubectl set env deployment/shipping-service DATABASE_URL="postgres://..."
# 50 services = 50 duplicate configurations
```
**Issue**: Change database URL = update 50 deployments

### Azure App Configuration: Twelve-Factor Evolution
```csharp
// Modern approach: Twelve-Factor principle + centralized management
var config = new ConfigurationBuilder()
    .AddEnvironmentVariables()  ← Twelve-Factor compliant (connection string)
    .AddAzureAppConfiguration(
        Environment.GetEnvironmentVariable("APPCONFIGURATION_CONNECTION_STRING")  ← Only config is where to find config!
    )
    .Build();
```

**Hierarchy**:
```
Connection String (environment variable)
  └─ Points to → Azure App Configuration (centralized store)
       └─ Contains → All application config
            └─ References → Azure Key Vault (for secrets)
```

**Benefits**:
- ✅ Twelve-Factor compliant (config external to code)
- ✅ Centralized management (single source of truth)
- ✅ Hierarchical structure (`Database:ConnectionString:Primary`)
- ✅ Versioning and audit trail (who changed what, when)
- ✅ Shared configuration (multiple services reference same values)

---

## Modern Configuration Principles

### 1. Externalize Configuration
**Principle**: Configuration must be external to code and changeable without redeployment.

**Implementation Spectrum**:
```
❌ Worst:  Hardcoded constants
⚠️  Poor:   Configuration files in Git
✅  Good:   Environment variables
✅✅ Best:   Centralized configuration service
```

### 2. Separate Concerns (Code vs Config vs Secrets)
**Principle**: Different types of information require different storage and access patterns.

| Type | Storage | Access | Version Control | Examples |
|------|---------|--------|-----------------|----------|
| **Code** | Git repository | Developers | Yes (Git history) | Business logic, algorithms |
| **Configuration** | App Configuration | Developers, Operators | Yes (config versioning) | Feature flags, API endpoints |
| **Secrets** | Key Vault | Operators only (least privilege) | No (audit logs only) | Passwords, certificates, API keys |

### 3. Centralize Shared Configuration
**Principle**: Configuration used by multiple services should be defined once and referenced.

**Anti-Pattern (Duplication)**:
```
50 microservices × 10 shared config values = 500 configuration entries to update
```

**Best Practice (Centralization)**:
```
1 centralized store × 10 shared config values = 10 configuration entries
50 services reference same values (auto-sync)
```

### 4. Enable Dynamic Updates
**Principle**: Configuration changes should apply without application restart when possible.

**Traditional**:
```
Change config → Rebuild app → Redeploy → Restart (5-30 minutes downtime)
```

**Modern (Azure App Configuration)**:
```
Change config → Apps poll → Refresh (30 seconds, zero downtime)
```

### 5. Implement Configuration Versioning
**Principle**: Configuration changes must be auditable with rollback capabilities.

**Azure App Configuration Features**:
- **Point-in-time snapshots**: Capture config state at specific timestamp
- **Labels**: Environment tags (`dev`, `staging`, `prod`)
- **Revision history**: Who changed what, when
- **Rollback**: Restore previous configuration snapshot

---

## Migration Path: Traditional → Modern

### Phase 1: Remove Secrets from Source Control (Immediate)
```bash
# Step 1: Audit repository for leaked secrets
git secrets --scan-history  # Or use GitHub Secret Scanning

# Step 2: Remove secrets from config files
# appsettings.json (before)
{
  "ConnectionStrings": {
    "Database": "Server=prod-db;Password=SuperSecret"  ❌
  }
}

# appsettings.json (after)
{
  "ConnectionStrings": {
    "Database": "Server=prod-db;Password=${DATABASE_PASSWORD}"  ← Placeholder
  }
}

# Step 3: Store secrets in Azure Key Vault
az keyvault secret set --vault-name myapp-kv --name DatabasePassword --value "SuperSecret"

# Step 4: Inject secrets during deployment
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'prod-service-connection'
    KeyVaultName: 'myapp-kv'
    SecretsFilter: 'DatabasePassword'
```

### Phase 2: Externalize Configuration (Short-term)
```csharp
// Migrate from files to environment variables
// Old approach: appsettings.Production.json
{
  "FeatureFlags": { "EnableBetaFeatures": true },
  "ApiEndpoint": "https://api.example.com"
}

// New approach: Environment variables
Environment.GetEnvironmentVariable("FEATURE_FLAGS_ENABLEBETAFEATURES");
Environment.GetEnvironmentVariable("API_ENDPOINT");

// Set in deployment pipeline
- task: AzureRmWebAppDeployment@4
  inputs:
    AppSettings: |
      -FeatureFlags:EnableBetaFeatures true
      -ApiEndpoint https://api.example.com
```

### Phase 3: Centralize Configuration (Long-term)
```csharp
// Migrate to Azure App Configuration
builder.Configuration.AddAzureAppConfiguration(options => {
    options.Connect(connectionString)
           .Select(KeyFilter.Any, environment)
           .ConfigureKeyVault(kv => kv.SetCredential(new DefaultAzureCredential()))
           .ConfigureRefresh(refresh => {
               refresh.Register("AppSettings:Sentinel", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromSeconds(30));
           });
});

// Benefits achieved:
// ✅ Single source of truth for all environments
// ✅ Dynamic refresh (no restart required)
// ✅ Audit trail (who changed what, when)
// ✅ Key Vault integration (secrets referenced, not duplicated)
// ✅ Shared configuration (multiple apps reference same values)
```

---

## Key Takeaways

### Configuration Anti-Patterns to Avoid
❌ **Hardcoded secrets** in source code (security vulnerability)  
❌ **Configuration files in Git** (credential leak risk)  
❌ **Environment-specific builds** (violates "build once, deploy many")  
❌ **Configuration duplication** across microservices (drift risk)  
❌ **Obfuscation as security** (false sense of protection)  

### Modern Configuration Best Practices
✅ **Externalize configuration** (separate from code)  
✅ **Centralize shared config** (single source of truth)  
✅ **Separate secrets from configuration** (Key Vault for sensitive data)  
✅ **Enable dynamic refresh** (change without restart)  
✅ **Implement versioning** (audit trail and rollback)  
✅ **Use managed identities** (passwordless authentication)  

### The 2AM Deployment Rule
> "If a developer can accidentally commit production secrets at 2AM, your configuration architecture is fundamentally insecure."

**Solution**: Systemic prevention through tooling, not reliance on human vigilance.

---

**Next**: Learn about separation of concerns principles applied to configuration management →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/2-rethink-application-configuration-data)
