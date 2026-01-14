# Explore Separation of Concerns

**Duration**: 3 minutes

Separation of concerns (SoC) applied to configuration management establishes clear role-based responsibilities between configuration custodians, consumers, and storage infrastructure.

---

## Separation of Concerns Principle

### Core Concept
**Definition**: Architectural pattern dividing configuration management responsibilities across distinct roles with clearly defined boundaries.

**Goal**: Decouple configuration **schema** (what configuration exists) from configuration **values** (environment-specific settings) from configuration **storage** (where/how persisted).

**Why It Matters**: 
- Developers define configuration requirements without knowing production secrets
- Operations manage production values without understanding application architecture
- Security teams control access policies without impacting development velocity

---

## The Four Configuration Roles

### 1. Configuration Custodian (Operations/Security Teams)
**Responsibilities**:
- **Lifecycle management**: CRUD operations on configuration keys and values
- **Secret security**: Ensure sensitive data encrypted, access controlled, audit logged
- **Credential rotation**: Regenerate keys, tokens, certificates per compliance policies
- **Environment-specific settings**: Define values for dev/staging/prod environments
- **CI/CD injection**: Configure pipelines to provide configuration at deployment time

**What They DON'T Do**:
- ❌ Define configuration schema (developers specify requirements)
- ❌ Understand application business logic
- ❌ Know which configuration keys are required by applications

**Example Workflow**:
```bash
# Operations team managing production database connection
# They DON'T need to know how the application uses it

# 1. Create Key Vault secret
az keyvault secret set \
  --vault-name myapp-prod-kv \
  --name DatabaseConnectionString \
  --value "Server=prod-sql.database.windows.net;Database=orders;User ID=dbadmin;Password=ComplexPassword123!"

# 2. Configure App Configuration to reference Key Vault
az appconfig kv set \
  --name myapp-config \
  --key "ConnectionStrings:OrderDatabase" \
  --value "{"uri":"https://myapp-prod-kv.vault.azure.net/secrets/DatabaseConnectionString"}" \
  --content-type "application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8" \
  --label production

# 3. Grant application managed identity access to Key Vault
az keyvault set-policy \
  --name myapp-prod-kv \
  --object-id <app-managed-identity-id> \
  --secret-permissions get list
```

**Security Boundaries**:
- ✅ Custodians manage production secrets (operations/security clearance)
- ✅ Developers never see production passwords (zero-trust principle)
- ✅ Audit trail tracks all secret access (compliance requirement)

### 2. Configuration Consumer (Development/Testing Teams)
**Responsibilities**:
- **Schema definition**: Specify configuration keys, data types, structure required by application
- **Default values**: Provide sensible defaults for development environments
- **Configuration consumption**: Read configuration in application code via SDK/libraries
- **Validation**: Ensure configuration present and valid at application startup
- **Documentation**: Describe configuration purpose, valid ranges, dependencies

**What They DON'T Do**:
- ❌ Know production secret values (security isolation)
- ❌ Manage production configuration (operations responsibility)
- ❌ Deploy configuration to production (CI/CD pipeline handles injection)

**Example Workflow**:
```csharp
// Developer defines configuration schema and consumption

// 1. Define configuration model (schema)
public class OrderServiceConfig
{
    public string ConnectionString { get; set; }  // Defined: string type, required
    public int MaxRetryAttempts { get; set; } = 3;  // Default: 3 attempts
    public TimeSpan Timeout { get; set; } = TimeSpan.FromSeconds(30);  // Default: 30s
}

// 2. Register configuration in DI container
services.Configure<OrderServiceConfig>(
    builder.Configuration.GetSection("ConnectionStrings:OrderDatabase")
);

// 3. Consume configuration in application logic
public class OrderRepository
{
    private readonly OrderServiceConfig _config;
    
    public OrderRepository(IOptions<OrderServiceConfig> config)
    {
        _config = config.Value;
        
        // Developer uses configuration without knowing production values
        var connection = new SqlConnection(_config.ConnectionString);
    }
}

// 4. Document configuration requirements (README.md)
// ## Configuration Requirements
// - `ConnectionStrings:OrderDatabase` (string, required): SQL Server connection string
// - `ConnectionStrings:OrderDatabase:MaxRetryAttempts` (int, optional, default: 3)
// - `ConnectionStrings:OrderDatabase:Timeout` (duration, optional, default: 30s)
```

**Developer Experience**:
```bash
# Local development: Use user secrets (never committed to Git)
dotnet user-secrets set "ConnectionStrings:OrderDatabase" "Server=localhost;Database=orders-dev;Integrated Security=true"

# Application reads from:
# 1. User secrets (local dev)
# 2. Environment variables (Docker/Kubernetes)
# 3. Azure App Configuration (cloud deployments)
```

### 3. Configuration Store (Non-Sensitive Data)
**Purpose**: Persist environment-specific behavioral parameters **excluding** sensitive data requiring encryption/HSM protection.

**Storage Options**:

| Solution | Best For | Pros | Cons |
|----------|----------|------|------|
| **Azure App Configuration** | Cloud-native apps | Centralized, versioned, dynamic refresh, feature flags | Cloud dependency, cost |
| **etcd** | Kubernetes workloads | Distributed, consistent, built-in to k8s | Operational complexity |
| **Consul** | Multi-cloud, service mesh | Service discovery + config, highly available | Learning curve |
| **Spring Cloud Config** | Spring Boot apps | Framework integration, Git backend | Java-specific |
| **Environment Variables** | Simple deployments | Universal, simple | Scattered, no versioning, no hierarchy |

**What Goes in Configuration Store**:
- ✅ Feature flags (`EnableBetaFeatures: true`)
- ✅ API endpoints (`PaymentApi: https://api.payment.com`)
- ✅ Logging levels (`LogLevel:Default: Information`)
- ✅ Timeout values (`HttpClient:Timeout: 30s`)
- ✅ UI themes (`Theme: dark`)
- ❌ Passwords (use secret store)
- ❌ API keys (use secret store)
- ❌ Certificates (use secret store)

**Example: Azure App Configuration**
```bash
# Non-sensitive configuration in App Configuration
az appconfig kv set --name myapp-config --key "FeatureManagement:BetaCheckout" --value "true" --label production
az appconfig kv set --name myapp-config --key "PaymentApi:Endpoint" --value "https://api.stripe.com" --label production
az appconfig kv set --name myapp-config --key "Logging:LogLevel:Default" --value "Information" --label production

# Reference to secret (value is Key Vault URI, NOT the secret itself)
az appconfig kv set --name myapp-config --key "PaymentApi:ApiKey" --value "{"uri":"https://myapp-kv.vault.azure.net/secrets/StripeApiKey"}" --content-type "application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8" --label production
```

### 4. Secret Store (Sensitive Data)
**Purpose**: Secure storage infrastructure for sensitive configuration data with encryption, access control, audit logging, and rotation capabilities.

**Storage Solutions**:

| Solution | Best For | Features | Limitations |
|----------|----------|----------|-------------|
| **Azure Key Vault** | Azure workloads | HSM-backed, managed identities, audit logs, cert auto-renewal | Azure-specific |
| **HashiCorp Vault** | Multi-cloud | Dynamic secrets, encryption-as-service, plugins | Self-hosted complexity |
| **AWS Secrets Manager** | AWS workloads | Auto-rotation, RDS integration | AWS-specific |
| **Kubernetes Secrets** | K8s-native apps | Native k8s integration, simple | Base64 only (not encrypted at rest by default) |

**What Goes in Secret Store**:
- ✅ Database passwords
- ✅ API keys (third-party services)
- ✅ Connection strings (with credentials)
- ✅ Certificates (SSL/TLS, code signing)
- ✅ Encryption keys
- ✅ OAuth client secrets
- ✅ SSH private keys

**Example: Azure Key Vault**
```bash
# Store secrets in Key Vault (NOT App Configuration)
az keyvault secret set --vault-name myapp-kv --name DatabasePassword --value "SuperSecretPassword123!"
az keyvault secret set --vault-name myapp-kv --name StripeApiKey --value "sk-live-abc123def456"
az keyvault secret set --vault-name myapp-kv --name JwtSigningKey --value "base64EncodedKey..."

# Certificate management (auto-renewal)
az keyvault certificate create --vault-name myapp-kv --name wildcard-ssl --policy @cert-policy.json
```

---

## Role-Based Access Control (RBAC)

### Principle: Least Privilege
**Goal**: Each role has minimum permissions required for their responsibilities.

**Access Matrix**:

| Role | Configuration Store | Secret Store | Production Values |
|------|-------------------|--------------|-------------------|
| **Developer** | ✅ Read schema<br>✅ Write dev/staging values | ❌ No access | ❌ No access |
| **QA/Tester** | ✅ Read staging values | ❌ No access | ❌ No access |
| **Operations** | ✅ Read/Write all values<br>✅ Manage prod config | ✅ Read/Write secrets<br>✅ Rotate credentials | ✅ Full access |
| **Security Team** | ✅ Audit access logs | ✅ Full access<br>✅ Access policy management | ✅ Audit only |
| **Application (Managed Identity)** | ✅ Read config for its environment | ✅ Read secrets it needs | ✅ Runtime access only |

### Azure RBAC Example
```bash
# Developer: Read access to dev/staging config, NO production access
az role assignment create \
  --role "App Configuration Data Reader" \
  --assignee developer@company.com \
  --scope /subscriptions/{sub-id}/resourceGroups/dev-rg/providers/Microsoft.AppConfiguration/configurationStores/myapp-dev-config

# Operations: Full access to production config and secrets
az role assignment create \
  --role "App Configuration Data Owner" \
  --assignee ops@company.com \
  --scope /subscriptions/{sub-id}/resourceGroups/prod-rg/providers/Microsoft.AppConfiguration/configurationStores/myapp-prod-config

az role assignment create \
  --role "Key Vault Administrator" \
  --assignee ops@company.com \
  --scope /subscriptions/{sub-id}/resourceGroups/prod-rg/providers/Microsoft.KeyVault/vaults/myapp-prod-kv

# Application: Managed identity with read-only access
az role assignment create \
  --role "App Configuration Data Reader" \
  --assignee <app-managed-identity-id> \
  --scope /subscriptions/{sub-id}/resourceGroups/prod-rg/providers/Microsoft.AppConfiguration/configurationStores/myapp-prod-config

az keyvault set-policy \
  --name myapp-prod-kv \
  --object-id <app-managed-identity-id> \
  --secret-permissions get list
```

---

## Separation Benefits

### 1. Security Isolation
**Problem**: Developers with production secret access create credential leak risk.

**Solution**: Developers define schema, operations manage values.

**Before (Unified)**:
```yaml
# appsettings.Production.json (committed to Git by developer)
{
  "ConnectionStrings": {
    "Database": "Server=prod-db;Password=SuperSecret123"  ❌ Developer sees secret
  }
}
```

**After (Separated)**:
```csharp
// Developer code (no secret knowledge)
var config = new ConfigurationBuilder()
    .AddAzureAppConfiguration(connectionString)  ← Points to config store
    .Build();

var dbConnection = config["ConnectionStrings:Database"];  ← Retrieved at runtime

// Operations manages in Key Vault (developer never sees value)
// az keyvault secret set --name DatabasePassword --value "SuperSecret123"
```

**Security Improvement**:
- ✅ Developers never see production credentials
- ✅ Reduced attack surface (fewer users with secret access)
- ✅ Audit trail (operations team secret access logged)

### 2. Deployment Simplicity
**Problem**: Environment-specific builds create artifact proliferation.

**Solution**: Single artifact with externalized configuration.

**Before (Environment-Specific Builds)**:
```bash
# Build 3 separate artifacts
dotnet publish --configuration Development
dotnet publish --configuration Staging
dotnet publish --configuration Production  ← Different binaries!

# Deploy different artifacts per environment
# Testing staging ≠ testing production (different code paths)
```

**After (Single Artifact)**:
```bash
# Build once
dotnet publish --configuration Release  ← Single artifact

# Deploy same artifact to all environments
# Configuration injected at deployment time
az webapp config appsettings set --name myapp-dev --settings ENVIRONMENT=dev
az webapp config appsettings set --name myapp-staging --settings ENVIRONMENT=staging
az webapp config appsettings set --name myapp-prod --settings ENVIRONMENT=production
```

**Benefits**:
- ✅ What you test = what you deploy (build once, deploy many)
- ✅ Faster deployments (no per-environment builds)
- ✅ Reduced configuration drift

### 3. Configuration Reusability
**Problem**: Shared configuration duplicated across microservices.

**Solution**: Centralized store with shared values.

**Before (Duplication)**:
```bash
# 50 microservices with duplicate database URLs
order-service/appsettings.json:       "DatabaseUrl": "postgres://prod-db:5432/orders"
payment-service/appsettings.json:     "DatabaseUrl": "postgres://prod-db:5432/orders"
shipping-service/appsettings.json:    "DatabaseUrl": "postgres://prod-db:5432/orders"
# ... 47 more duplicates

# Database migration requires updating 50 files
```

**After (Centralization)**:
```bash
# Single definition in Azure App Configuration
az appconfig kv set --name myapp-config --key "Shared:DatabaseUrl" --value "postgres://prod-db:5432/orders"

# All 50 services reference same value
# Application code:
var dbUrl = config["Shared:DatabaseUrl"];  ← All services get consistent value

# Database migration: Update once, all services refresh
```

### 4. Audit and Compliance
**Problem**: Configuration changes untracked, no accountability.

**Solution**: Centralized store with audit logs.

**Azure App Configuration Audit**:
```bash
# Enable diagnostic logging
az monitor diagnostic-settings create \
  --name audit-logs \
  --resource /subscriptions/{sub-id}/resourceGroups/prod-rg/providers/Microsoft.AppConfiguration/configurationStores/myapp-prod-config \
  --logs '[{"category":"HttpRequest","enabled":true}]' \
  --workspace /subscriptions/{sub-id}/resourceGroups/prod-rg/providers/Microsoft.OperationalInsights/workspaces/myapp-logs

# Query audit trail (who changed what, when)
az monitor log-analytics query \
  --workspace myapp-logs \
  --analytics-query "AzureDiagnostics | where ResourceProvider == 'MICROSOFT.APPCONFIGURATION' | where OperationName == 'ConfigurationStoreWrite' | project TimeGenerated, Identity, Key, Value, Label"
```

**Compliance Benefits**:
- ✅ SOC 2 compliance (access tracking)
- ✅ GDPR compliance (data handling audit)
- ✅ Change attribution (who modified production config)
- ✅ Rollback capability (restore previous config snapshot)

---

## Architecture Patterns

### Pattern 1: Hierarchical Configuration Store
**Structure**: Shared configuration with environment-specific overrides.

```
Azure App Configuration
├── Shared:DatabaseUrl (applies to all environments)
├── Shared:CacheUrl (applies to all environments)
├── Shared:LogLevel (applies to all environments)
├── [Label: dev]
│   ├── FeatureFlags:EnableBetaFeatures = true
│   └── LogLevel:Default = Debug
├── [Label: staging]
│   ├── FeatureFlags:EnableBetaFeatures = true
│   └── LogLevel:Default = Information
└── [Label: production]
    ├── FeatureFlags:EnableBetaFeatures = false
    └── LogLevel:Default = Warning
```

**Application reads**:
```csharp
// Application in production environment
options.Connect(connectionString)
       .Select(KeyFilter.Any, LabelFilter.Null)  ← Shared values (no label)
       .Select(KeyFilter.Any, "production");  ← Production overrides
```

### Pattern 2: Configuration + Secret Reference
**Architecture**: Non-sensitive config in App Configuration, secrets in Key Vault.

```
[Application]
     │
     ├─ Reads config ──> [Azure App Configuration]
     │                          │
     │                          ├─ FeatureFlags (plaintext)
     │                          ├─ ApiEndpoints (plaintext)
     │                          └─ SecretReferences (Key Vault URIs)
     │                                      │
     └─ Resolves secrets ─────────────────┘
                                           ↓
                                   [Azure Key Vault]
                                           │
                                           ├─ DatabasePassword
                                           ├─ ApiKeys
                                           └─ Certificates
```

**Configuration Example**:
```json
// In Azure App Configuration
{
  "PaymentApi": {
    "Endpoint": "https://api.stripe.com",  ← Plaintext (not sensitive)
    "ApiKey": {
      "uri": "https://myapp-kv.vault.azure.net/secrets/StripeApiKey"  ← Reference
    }
  }
}

// Application resolves reference automatically
var apiKey = config["PaymentApi:ApiKey"];  ← Returns actual secret value from Key Vault
```

### Pattern 3: Environment-Agnostic Application
**Goal**: Application code identical across all environments.

**Implementation**:
```csharp
// Single codebase, no environment-specific logic
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Configuration injected externally (no if/else for environments)
        services.AddDbContext<OrderContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("OrderDatabase"))
        );
        
        services.AddHttpClient<IPaymentService, PaymentService>(client =>
            client.BaseAddress = new Uri(Configuration["PaymentApi:Endpoint"])
        );
    }
}

// Environment-specific behavior controlled by configuration, not code
// Dev: Configuration["PaymentApi:Endpoint"] = "https://api.stripe.test" (test mode)
// Prod: Configuration["PaymentApi:Endpoint"] = "https://api.stripe.com" (live mode)
```

---

## Real-World Example: E-Commerce Platform

### Scenario
**Company**: 50-microservice e-commerce platform  
**Environments**: dev, staging, production, canary  
**Teams**: 15 development teams, 1 operations team, 1 security team  

### Before Separation (Chaos)
```
Problems:
- 200 configuration files (50 services × 4 environments)
- Database password in 50+ files (rotation nightmare)
- Developers have production secrets (security risk)
- Configuration drift (staging ≠ production)
- No audit trail (who changed what?)
- 8-hour deployment windows (rolling config updates)
```

### After Separation (Clarity)
```
Azure App Configuration (mycompany-config)
├── Shared (no label)
│   ├── Database:Url = "prod-sql.database.windows.net"
│   ├── Cache:Url = "prod-cache.redis.cache.windows.net"
│   └── Monitoring:ApplicationInsightsKey = (Key Vault reference)
├── [Label: dev]
│   ├── FeatureFlags:* = true (all features enabled)
│   └── LogLevel:Default = "Debug"
├── [Label: production]
│   ├── FeatureFlags:* = false (stable features only)
│   └── LogLevel:Default = "Warning"

Azure Key Vault (mycompany-prod-kv)
├── DatabasePassword (rotated automatically every 90 days)
├── ApiKeys/* (payment gateway, shipping providers)
└── Certificates/* (SSL, code signing)

Access Control:
- Developers: Read access to dev/staging config only
- Operations: Full access to all environments
- Security: Audit access to Key Vault
- Applications: Managed identity with read-only access
```

**Results**:
- ✅ 200 files → 1 centralized store (99.5% reduction)
- ✅ Zero production secret exposure to developers
- ✅ 30-second configuration updates (vs 8 hours)
- ✅ Complete audit trail (SOC 2 compliance achieved)
- ✅ Automated secret rotation (zero-touch credential management)

---

## Key Takeaways

### Separation of Concerns Benefits
✅ **Security isolation**: Developers never see production secrets  
✅ **Clear responsibilities**: Custodians manage values, consumers define schema  
✅ **Audit compliance**: Centralized logs track all configuration changes  
✅ **Deployment simplicity**: Single artifact with external configuration  
✅ **Configuration reusability**: Shared values across microservices  

### Role Boundaries
**Configuration Custodian** (Operations):
- Manages environment-specific values
- Rotates credentials per compliance policies
- Controls production secret access
- Configures CI/CD injection

**Configuration Consumer** (Developers):
- Defines configuration schema
- Consumes configuration in application code
- Documents configuration requirements
- Uses local secrets for development

**Configuration Store** (Azure App Configuration):
- Persists non-sensitive behavioral parameters
- Provides versioning and audit trail
- Enables dynamic refresh without restart

**Secret Store** (Azure Key Vault):
- Secures sensitive data with encryption/HSM
- Enforces access control policies
- Supports automated rotation
- Maintains separate audit trail from configuration

---

**Next**: Learn about external configuration store patterns and cloud design patterns →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/3-explore-separation-of-concerns)
