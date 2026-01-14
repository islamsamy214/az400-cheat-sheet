# Introduction to Azure App Configuration

**Duration**: 3 minutes

Azure App Configuration provides centralized management infrastructure for application settings and feature flags, enabling consistent configuration across distributed applications.

---

## Overview

### What is Azure App Configuration?
**Azure App Configuration**: Fully managed service for centralized configuration management with native support for feature flags, labels, and Key Vault integration.

**Problem It Solves**: Distributed applications (microservices, serverless) with scattered configuration files create inconsistencies, deployment complexity, and troubleshooting challenges.

**Core Capabilities**:
- ✅ **Centralized storage**: Single source of truth for all application settings
- ✅ **Key-value pairs**: Flexible hierarchical key structure with labels
- ✅ **Feature flags**: Built-in feature management with targeting filters
- ✅ **Point-in-time snapshots**: Historical state restoration and rollback
- ✅ **Dynamic refresh**: Runtime configuration updates without restart
- ✅ **Key Vault integration**: Reference secrets without duplication
- ✅ **Encryption**: Data encrypted at rest and in transit

---

## Why Azure App Configuration?

### Traditional Configuration Challenges

**Problem 1: Configuration Sprawl**
```
50 microservices × 4 environments = 200 configuration files
├── order-service/
│   ├── appsettings.Development.json
│   ├── appsettings.Staging.json
│   ├── appsettings.Production.json
│   └── appsettings.Canary.json
├── payment-service/
│   ├── appsettings.Development.json (duplicate database URL)
│   └── ... (3 more files with duplicate config)
└── ... (48 more services with duplicated config)

Result: 200 files to maintain, massive configuration drift
```

**Solution: Azure App Configuration**
```
[Azure App Configuration]
├── Shared:DatabaseUrl (defined once)
├── Shared:CacheUrl (defined once)
├── [Label: development]
├── [Label: staging]
└── [Label: production]

Result: 1 centralized store, zero configuration drift
```

**Problem 2: Deployment Required for Config Changes**
```
Traditional: Change config file → Commit to Git → CI/CD pipeline → Redeploy
Time: 15-30 minutes per environment × 4 environments = 1-2 hours
```

**Solution: Dynamic Refresh**
```
App Configuration: Update value in portal → Apps refresh within 30 seconds
Time: 30 seconds, zero deployments
```

**Problem 3: Feature Deployment = Code Deployment**
```
Traditional: Feature toggle in code → Deploy to enable/disable
Risk: Code deployment for non-code change (configuration)
```

**Solution: Feature Flags**
```
App Configuration: Toggle feature flag in portal → Instant activation/deactivation
Risk: Zero (no code deployment, instant rollback)
```

---

## Service Capabilities

### 1. Fully Managed Service
**Provisioning**: Rapid setup within minutes (no infrastructure management).

```bash
# Create App Configuration store
az appconfig create \
  --name myapp-config \
  --resource-group prod-rg \
  --location eastus \
  --sku Standard  # Free, Standard, or Premium

# Provisioned in ~60 seconds
# Includes:
# - Encryption at rest (Microsoft-managed keys)
# - High availability (Azure-managed)
# - Automatic backups
# - 99.9% SLA (Standard/Premium)
```

**Tiers**:

| Feature | Free | Standard | Premium |
|---------|------|----------|---------|
| **Requests/day** | 1,000 | 200,000 | Unlimited |
| **Storage** | 10 MB | 10 GB | 100 GB |
| **Key Vault references** | ❌ | ✅ | ✅ |
| **Private Link** | ❌ | ❌ | ✅ |
| **Geo-replication** | ❌ | ❌ | ✅ |
| **Price/month** | Free | ~$1.20/day | ~$2.40/day |

### 2. Flexible Key Representation
**Hierarchical Naming**: Organize configuration with delimiters (`:` or `/`).

```bash
# Component-based hierarchy
AppName:Service1:ApiEndpoint
AppName:Service2:ApiEndpoint

# Region-based hierarchy
AppName:Region:EastUS:DbEndpoint
AppName:Region:WestUS:DbEndpoint

# Environment-based hierarchy (using labels instead)
DatabaseUrl (label: development)
DatabaseUrl (label: production)
```

**Benefits**:
- ✅ Improved readability (semantic structure)
- ✅ Simplified management (logical grouping)
- ✅ Pattern matching queries (`AppName:Service*`)

### 3. Label-Based Tagging
**Labels**: Environment tags enabling identical keys with different values per environment.

```bash
# Same key, different values per environment
Key: ConnectionStrings:OrderDatabase
├── Label: development  → Value: "Server=localhost;Database=orders-dev"
├── Label: staging      → Value: "Server=staging-sql;Database=orders"
└── Label: production   → Value: "{"uri":"https://kv.vault.azure.net/secrets/DbConnection"}"
```

**Label Strategy**:
```
Common labels:
- development / staging / production (environments)
- v1.0 / v1.1 / v2.0 (application versions)
- eastus / westus (regions)
- team-a / team-b (team isolation)
```

### 4. Point-in-Time Configuration Replay
**Snapshots**: Capture configuration state at specific timestamp for rollback.

```bash
# Create configuration snapshot
az appconfig snapshot create \
  --name prod-release-v1.0 \
  --store-name myapp-config \
  --filters '{"key":"*","label":"production"}' \
  --composition-type key_label \
  --retention-period 30

# Restore from snapshot (rollback)
az appconfig kv restore \
  --name myapp-config \
  --datetime "2026-01-14T10:00:00Z"  # Restore to this point in time

# Query historical configuration
az appconfig revision list \
  --name myapp-config \
  --key "FeatureManagement:*" \
  --datetime "2026-01-01T00:00:00Z"
```

**Use Cases**:
- ✅ Audit compliance (SOC 2, HIPAA)
- ✅ Rollback after bad configuration
- ✅ Debug past incidents (what was config at incident time?)
- ✅ Compare configuration versions

### 5. Feature Flag Administration UI
**Built-in Feature Management**: Dedicated UI for creating and managing feature flags.

```
Azure Portal → App Configuration → Feature Manager
├── Create feature flag
│   ├── Name: BetaCheckout
│   ├── Enabled: true
│   └── Filters:
│       ├── Percentage: 20%
│       └── Targeting: beta-testers group
├── Monitor feature flag usage
└── Toggle flags on/off (instant effect)
```

**Feature Flag Capabilities**:
- ✅ Boolean toggles (on/off)
- ✅ Percentage rollouts (10%, 50%, 100%)
- ✅ User targeting (specific users/groups)
- ✅ Time window filters (enable 9am-5pm)
- ✅ Custom filters (browser type, region)

### 6. Configuration Comparison
**Diff Tool**: Compare configuration across environments or time periods.

```bash
# Compare staging vs production
az appconfig kv list --name myapp-config --label staging > staging.json
az appconfig kv list --name myapp-config --label production > prod.json
diff staging.json prod.json

# Find configuration drift
# Output shows differences (missing keys, different values)
```

### 7. Azure Managed Identity Integration
**Passwordless Authentication**: Applications access configuration using managed identities.

```csharp
// No secrets in code!
builder.Configuration.AddAzureAppConfiguration(options => {
    options.Connect(
        new Uri("https://myapp-config.azconfig.io"),
        new DefaultAzureCredential()  // Uses managed identity
    );
});
```

**Benefits**:
- ✅ Zero secrets in application code
- ✅ Automatic credential rotation (Azure-managed)
- ✅ Audit trail (identity-based access logs)
- ✅ Least privilege (RBAC per identity)

### 8. Comprehensive Encryption
**Security**:
- **At rest**: AES-256 encryption (Microsoft-managed or customer-managed keys)
- **In transit**: TLS 1.2+ (HTTPS only)
- **Key Vault references**: Secrets never stored in App Configuration

---

## Use Cases

### Use Case 1: Centralized Multi-Environment Configuration
**Scenario**: 50 microservices deployed across dev/staging/production.

**Implementation**:
```bash
# Shared configuration (all environments)
az appconfig kv set --name myapp-config --key "Shared:CacheUrl" --value "redis://cache:6379"

# Environment-specific configuration
az appconfig kv set --name myapp-config --key "LogLevel" --value "Debug" --label development
az appconfig kv set --name myapp-config --key "LogLevel" --value "Information" --label staging
az appconfig kv set --name myapp-config --key "LogLevel" --value "Warning" --label production

# Application reads based on environment
# Development: LogLevel = "Debug"
# Production: LogLevel = "Warning"
```

**Benefits**:
- ✅ 200 files → 1 centralized store (99.5% reduction)
- ✅ Zero configuration drift (single source of truth)
- ✅ Instant updates (30-second refresh, no deployment)

### Use Case 2: Dynamic Feature Rollouts
**Scenario**: E-commerce platform testing new checkout flow.

**Implementation**:
```bash
# Create feature flag with 20% rollout
az appconfig feature set \
  --name myapp-config \
  --feature BetaCheckout \
  --yes \
  --label production

# Add percentage filter
az appconfig feature filter add \
  --name myapp-config \
  --feature BetaCheckout \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":20}'

# Application code checks feature flag
if (await _featureManager.IsEnabledAsync("BetaCheckout"))
{
    return View("NewCheckoutFlow");  // 20% of users
}
else
{
    return View("LegacyCheckoutFlow");  // 80% of users
}

# Increase rollout based on metrics (instant)
az appconfig feature filter update \
  --name myapp-config \
  --feature BetaCheckout \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":100}'  # Now 100% of users
```

**Benefits**:
- ✅ Progressive rollout (20% → 50% → 100%)
- ✅ Instant rollback (disable flag in portal)
- ✅ Zero code deployment (feature already deployed, flag controls visibility)

### Use Case 3: Real-Time Configuration Updates
**Scenario**: Adjust API rate limits based on load.

**Implementation**:
```bash
# Initial rate limit
az appconfig kv set --name myapp-config --key "RateLimiting:RequestsPerMinute" --value "1000"

# Application reads with refresh
builder.Configuration.AddAzureAppConfiguration(options => {
    options.ConfigureRefresh(refresh => {
        refresh.Register("RateLimiting:RequestsPerMinute", refreshAll: false)
               .SetCacheExpiration(TimeSpan.FromSeconds(30));
    });
});

# During high load: Reduce rate limit (instant effect)
az appconfig kv set --name myapp-config --key "RateLimiting:RequestsPerMinute" --value "500"

# All app instances refresh within 30 seconds (no restart)
```

### Use Case 4: Secure Secret References
**Scenario**: Database connection string contains sensitive password.

**Implementation**:
```bash
# Step 1: Store secret in Key Vault
az keyvault secret set \
  --vault-name myapp-kv \
  --name DatabaseConnectionString \
  --value "Server=prod-db;Database=orders;User ID=admin;Password=SuperSecret123!"

# Step 2: Store Key Vault reference in App Configuration (not the secret itself!)
az appconfig kv set \
  --name myapp-config \
  --key "ConnectionStrings:OrderDatabase" \
  --value '{"uri":"https://myapp-kv.vault.azure.net/secrets/DatabaseConnectionString"}' \
  --content-type "application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8"

# Step 3: Application resolves reference automatically
var connectionString = configuration["ConnectionStrings:OrderDatabase"];
// Returns actual secret value from Key Vault (seamlessly)
```

**Security Benefits**:
- ✅ Secrets never stored in App Configuration
- ✅ Key Vault access controlled separately (RBAC)
- ✅ Audit logs track secret access
- ✅ Automatic secret rotation (update Key Vault, apps refresh)

---

## Framework Integration

### .NET Core / ASP.NET Core
```csharp
// Install NuGet package
// Microsoft.Azure.AppConfiguration.AspNetCore

// Program.cs
builder.Configuration.AddAzureAppConfiguration(options => {
    options.Connect(
            Environment.GetEnvironmentVariable("APPCONFIGURATION_CONNECTION_STRING")
        )
        .Select(KeyFilter.Any, LabelFilter.Null)  // Shared config
        .Select(KeyFilter.Any, builder.Environment.EnvironmentName)  // Environment-specific
        .ConfigureRefresh(refresh => {
            refresh.Register("AppSettings:Sentinel", refreshAll: true)
                   .SetCacheExpiration(TimeSpan.FromSeconds(30));
        })
        .UseFeatureFlags(flags => {
            flags.CacheExpirationInterval = TimeSpan.FromSeconds(30);
        });
});

builder.Services.AddAzureAppConfiguration();
builder.Services.AddFeatureManagement();

var app = builder.Build();
app.UseAzureAppConfiguration();
```

### Java Spring Boot
```java
// Maven dependency
// com.azure.spring:spring-cloud-azure-appconfiguration-config

// application.properties
spring.cloud.azure.appconfiguration.stores[0].connection-string=${APPCONFIGURATION_CONNECTION_STRING}
spring.cloud.azure.appconfiguration.stores[0].monitoring.enabled=true
spring.cloud.azure.appconfiguration.stores[0].monitoring.refresh-interval=30s

// Application code
@RestController
public class ConfigController {
    @Value("${database.url}")
    private String databaseUrl;  // Automatically refreshed
    
    @GetMapping("/config")
    public String getConfig() {
        return "Database URL: " + databaseUrl;
    }
}
```

### Python
```python
# Install package
# pip install azure-appconfiguration-provider

from azure.appconfiguration.provider import (
    AzureAppConfigurationKeyVaultOptions,
    load
)
from azure.identity import DefaultAzureCredential

# Load configuration
config = load(
    endpoint="https://myapp-config.azconfig.io",
    credential=DefaultAzureCredential(),
    selects=[
        {"key_filter": "*", "label_filter": "production"}
    ],
    refresh_on=[{"key": "sentinel", "label": "production"}],
    refresh_interval=30
)

# Access configuration
database_url = config["database_url"]
api_key = config["api_key"]  # Resolved from Key Vault
```

### Node.js / JavaScript
```javascript
// Install package
// npm install @azure/app-configuration

const { AppConfigurationClient } = require("@azure/app-configuration");
const { DefaultAzureCredential } = require("@azure/identity");

const client = new AppConfigurationClient(
  "https://myapp-config.azconfig.io",
  new DefaultAzureCredential()
);

// Read configuration
const setting = await client.getConfigurationSetting({
  key: "database:url",
  label: "production"
});

console.log(`Database URL: ${setting.value}`);
```

### REST API (Any Language)
```bash
# Direct HTTP API access
GET https://myapp-config.azconfig.io/kv/database:url?label=production&api-version=1.0
Authorization: Bearer <managed-identity-token>

# Response
{
  "key": "database:url",
  "label": "production",
  "value": "postgres://prod-db:5432/orders",
  "content_type": "text/plain",
  "last_modified": "2026-01-14T10:30:00Z",
  "locked": false,
  "tags": {}
}
```

---

## Configuration Workflow Example

### Complete End-to-End Implementation

**Step 1: Create App Configuration Store**
```bash
# Create store
az appconfig create \
  --name mycompany-config \
  --resource-group prod-rg \
  --location eastus \
  --sku Standard

# Get connection string
az appconfig credential list \
  --name mycompany-config \
  --query "[?name=='Primary'].connectionString" -o tsv
```

**Step 2: Configure Application Settings**
```bash
# Shared configuration (no label)
az appconfig kv set --name mycompany-config --key "Shared:CacheUrl" --value "redis://cache:6379"

# Environment-specific configuration
az appconfig kv set --name mycompany-config --key "Database:Url" --value "localhost:5432" --label development
az appconfig kv set --name mycompany-config --key "Database:Url" --value "staging-db:5432" --label staging
az appconfig kv set --name mycompany-config --key "Database:Url" --value '{"uri":"https://myapp-kv.vault.azure.net/secrets/ProdDbUrl"}' --content-type "application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8" --label production
```

**Step 3: Configure Feature Flags**
```bash
# Create feature flag
az appconfig feature set --name mycompany-config --feature BetaFeatures --yes --label production

# Add percentage filter
az appconfig feature filter add \
  --name mycompany-config \
  --feature BetaFeatures \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":20}'
```

**Step 4: Application Integration** (see framework examples above)

**Step 5: Grant Managed Identity Access**
```bash
# Assign "App Configuration Data Reader" role to application's managed identity
az role assignment create \
  --role "App Configuration Data Reader" \
  --assignee <managed-identity-principal-id> \
  --scope /subscriptions/{sub-id}/resourceGroups/prod-rg/providers/Microsoft.AppConfiguration/configurationStores/mycompany-config
```

---

## Best Practices

### 1. Use Labels for Environments
```bash
# ✅ Best practice: Labels for environment differentiation
Key: DatabaseUrl, Label: development
Key: DatabaseUrl, Label: staging
Key: DatabaseUrl, Label: production

# ❌ Anti-pattern: Environment in key name
Key: DatabaseUrl-Development
Key: DatabaseUrl-Staging
Key: DatabaseUrl-Production
```

### 2. Implement Sentinel Keys for Coordinated Updates
```bash
# Update multiple related settings
az appconfig kv set --name myapp-config --key "Feature:NewUI:Enabled" --value "true"
az appconfig kv set --name myapp-config --key "Feature:NewUI:Version" --value "2.0"

# Trigger coordinated refresh with sentinel
az appconfig kv set --name myapp-config --key "AppSettings:Sentinel" --value "$(date -u +%Y-%m-%dT%H:%M:%SZ)"

# All apps refresh together (consistent state)
```

### 3. Reference Key Vault for Secrets
```bash
# ✅ Store secrets in Key Vault, reference from App Configuration
# ❌ Never store secrets directly in App Configuration

az keyvault secret set --vault-name myapp-kv --name ApiKey --value "secret123"
az appconfig kv set --name myapp-config --key "ExternalApi:Key" --value '{"uri":"https://myapp-kv.vault.azure.net/secrets/ApiKey"}' --content-type "application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8"
```

### 4. Enable Diagnostic Logging
```bash
az monitor diagnostic-settings create \
  --name appconfig-audit \
  --resource /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.AppConfiguration/configurationStores/myapp-config \
  --logs '[{"category":"HttpRequest","enabled":true},{"category":"Audit","enabled":true}]' \
  --workspace /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.OperationalInsights/workspaces/logs
```

---

## Key Takeaways

### Azure App Configuration Benefits
✅ **Centralized management**: Single source of truth eliminating configuration sprawl  
✅ **Dynamic refresh**: Runtime updates without application restart (30-second refresh)  
✅ **Feature flags**: Progressive rollouts with targeting and percentage filters  
✅ **Key Vault integration**: Secure secret references without duplication  
✅ **Point-in-time restore**: Configuration rollback and audit compliance  
✅ **Framework support**: Native SDKs for .NET, Java, Python, Node.js  

### When to Use App Configuration
✅ Distributed applications with multiple instances  
✅ Multi-environment deployments (dev/staging/prod)  
✅ Need for runtime configuration updates  
✅ Feature flag management and progressive rollouts  
✅ Centralized audit and compliance requirements  

---

**Next**: Learn about key-value pair structure, labels, and content types →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/6-introduction-to-azure-app-configuration)
