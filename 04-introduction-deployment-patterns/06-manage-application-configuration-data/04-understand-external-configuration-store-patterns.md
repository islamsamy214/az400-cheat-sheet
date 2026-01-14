# Understand External Configuration Store Patterns

**Duration**: 3 minutes

External Configuration Store pattern centralizes configuration data in dedicated infrastructure, enabling consistent, version-controlled configuration management across distributed applications.

---

## Overview

### Core Pattern Definition
**External Configuration Store**: Cloud design pattern that persists configuration information in dedicated external infrastructure with efficient interfaces for reading and modification operations.

**Problem Statement**: Distributed applications with multiple instances require consistent configuration without application redeployment, while file-based configuration creates synchronization challenges and operational complexity.

**Solution**: Centralized configuration server with client SDKs polling for changes, enabling runtime configuration updates across all application instances.

---

## Pattern Architecture

### Basic Architecture
```
                [External Configuration Store]
                    (Azure App Configuration)
                              │
                              ├─ Configuration Data
                              ├─ Feature Flags
                              ├─ Key Vault References
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
[App Instance 1]      [App Instance 2]      [App Instance 3]
    (Poll every          (Poll every          (Poll every
     30 seconds)          30 seconds)          30 seconds)
        │                     │                     │
        └─────────────────────┴─────────────────────┘
                    All instances eventually
                    converge to same configuration
```

### With Caching Layer
```
[External Configuration Store]
         │
         ├─ Fetches config ──> [Local Cache] ──> [Application]
         │                          │
         │                     TTL: 30s
         │                     (Reduces latency)
         │
    Fallback: Use cached
    config if store unavailable
```

**Benefits of Caching**:
- ✅ Reduced latency (milliseconds vs network round-trip)
- ✅ Resilience (app continues with cached config if store unavailable)
- ✅ Reduced load on configuration store
- ⚠️ Eventual consistency (30-second lag for updates)

---

## Implementation Requirements

### 1. Storage Backing Store
**Options by Environment**:

| Hosting | Recommended Store | Rationale |
|---------|------------------|-----------|
| **Azure Cloud** | Azure App Configuration | Native integration, managed service, low latency |
| **AWS Cloud** | AWS AppConfig or Parameter Store | Native AWS integration |
| **On-Premises** | Consul, etcd, or database | Self-hosted, control over infrastructure |
| **Kubernetes** | ConfigMaps + etcd | Native k8s integration |
| **Multi-Cloud** | HashiCorp Consul | Cloud-agnostic, service mesh capabilities |

**Selection Criteria**:
- Consistency model (strong vs eventual)
- Performance characteristics (latency, throughput)
- High availability requirements
- Cost considerations
- Operational complexity

### 2. Configuration Interface Requirements
**Must Provide**:
- ✅ **Consistent access**: All clients see same configuration state
- ✅ **Typed data**: Structured information (JSON, YAML, strongly-typed objects)
- ✅ **Authorization**: User/application access control
- ✅ **Versioning**: Multiple configuration versions (dev, staging, prod)
- ✅ **Audit trail**: Who changed what, when

**API Design Example** (Azure App Configuration):
```csharp
// Read configuration with type safety
var config = await client.GetConfigurationSettingAsync(
    key: "Database:ConnectionString",
    label: "production"  // Environment-specific version
);

// Structured data with content type
var featureConfig = await client.GetConfigurationSettingAsync(
    key: "FeatureManagement:BetaCheckout"
);
// Returns: { "enabled": true, "percentage": 50 }
```

### 3. Caching Strategy
**Traditional Configuration Load** (startup only):
```csharp
// ❌ Anti-pattern: Load once at startup
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        var config = LoadConfiguration();  // Never refreshes
        app.UseConfiguration(config);
    }
}
```

**Modern Caching with Refresh**:
```csharp
// ✅ Best practice: Cache with refresh
builder.Configuration.AddAzureAppConfiguration(options => {
    options.Connect(connectionString)
           .Select(KeyFilter.Any, environment)
           .ConfigureRefresh(refresh => {
               refresh.Register("AppSettings:Sentinel", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromSeconds(30));
           });
});

// Middleware to trigger refresh
app.UseAzureAppConfiguration();

// Configuration refreshes every 30 seconds in background
```

**Caching Patterns**:

| Pattern | Refresh Trigger | Use Case | Consistency |
|---------|----------------|----------|-------------|
| **Time-based (TTL)** | Every N seconds | General config | Eventual (30-60s lag) |
| **Sentinel-based** | When sentinel key changes | Coordinated updates | Controlled eventual |
| **Event-driven** | Push notification | Critical config | Near real-time |
| **Manual** | Explicit API call | Admin-triggered | Immediate |

### 4. Performance Optimization
**Latency Considerations**:
```
Without Caching:
Request → App → Config Store (100-200ms) → Response
Total: 100-200ms per request

With Local Cache:
Request → App → Local Cache (1-5ms) → Response
Total: 1-5ms per request (40x faster!)

Cache Miss:
Request → App → Local Cache (miss) → Config Store (100ms) → Cache → Response
Total: 100ms (infrequent, amortized cost)
```

**Optimization Strategies**:
```csharp
// Strategy 1: Aggressive caching with background refresh
options.ConfigureRefresh(refresh => {
    refresh.SetCacheExpiration(TimeSpan.FromMinutes(5));  // 5-minute cache
    // Background thread refreshes proactively
});

// Strategy 2: Lazy loading with cache-aside
var cachedConfig = memoryCache.GetOrCreate("AppConfig", entry => {
    entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10);
    return configClient.GetConfigurationSetting("key");
});

// Strategy 3: Bulk loading (reduce round-trips)
var allSettings = await client.GetConfigurationSettingsAsync(
    new SettingSelector { KeyFilter = "MyApp:*", LabelFilter = "production" }
);
```

---

## Pattern Applicability

### Use Case 1: Cross-Application Configuration Sharing
**Scenario**: Multiple microservices need shared database connection settings.

**Traditional Problem**:
```
50 microservices × 3 environments = 150 duplicate configurations
Database URL change = 150 manual updates
```

**External Store Solution**:
```csharp
// All services reference same shared configuration
var dbUrl = configuration["Shared:DatabaseUrl"];

// Update once in central store:
az appconfig kv set --name myapp-config \
  --key "Shared:DatabaseUrl" \
  --value "postgres://new-db:5432/app" \
  --label production

// All 50 services refresh within 30 seconds (zero deployments)
```

### Use Case 2: Extended Configuration Capabilities
**Scenario**: Store complex configuration data beyond standard config files.

**Beyond Standard Config**:
```json
// Standard config limitation: Text/JSON only
{
  "LogoUrl": "https://storage.azure.net/logo.png"  // Only URL, not image
}

// External store with rich content types:
{
  "Logo": {
    "contentType": "image/png",
    "value": "base64EncodedImageData...",  // Actual image binary
    "metadata": {
      "width": 200,
      "height": 100,
      "format": "PNG"
    }
  }
}
```

**Use Cases**:
- ✅ Binary configuration data (images, fonts, certificates)
- ✅ Large configuration files (ML models, lookup tables)
- ✅ Complex structured data (graph configurations, templates)

### Use Case 3: Hybrid Configuration Strategies
**Scenario**: Combine centralized defaults with application-specific overrides.

**Configuration Hierarchy**:
```csharp
// Layer 1: Centralized defaults (all apps)
builder.Configuration.AddAzureAppConfiguration(options => {
    options.Connect(connectionString)
           .Select("Shared:*", "default");  // Organization-wide defaults
});

// Layer 2: Application-specific overrides
builder.Configuration.AddAzureAppConfiguration(options => {
    options.Connect(connectionString)
           .Select("OrderService:*", "production");  // Override shared values
});

// Layer 3: Local overrides (development only)
builder.Configuration.AddJsonFile("appsettings.Local.json", optional: true);

// Precedence: Local > App-specific > Shared > Default
```

**Example**:
```
Shared:LogLevel = "Information" (default for all apps)
OrderService:LogLevel = "Debug" (override for OrderService only)
appsettings.Local.json:LogLevel = "Trace" (developer override for debugging)

Final value for OrderService: "Trace" (local wins)
```

### Use Case 4: Centralized Administration and Monitoring
**Scenario**: Manage configuration across 100+ microservices from single interface.

**Before (File-Based)**:
```bash
# Update configuration in 100 repositories
for repo in service-{1..100}; do
  cd $repo
  sed -i 's/old-value/new-value/' appsettings.json
  git commit -m "Update config"
  git push
  # Trigger deployment pipeline
done
# Time: 2-3 hours, 100 deployments
```

**After (Centralized Store)**:
```bash
# Update once in Azure App Configuration
az appconfig kv set --name myapp-config \
  --key "Shared:FeatureFlags:NewCheckout" \
  --value "true"

# All 100 services refresh within 30 seconds
# Time: 1 minute, zero deployments
```

**Monitoring Dashboard**:
```
Azure App Configuration Portal:
├── Configuration Usage
│   ├── Read requests: 1.2M/day
│   ├── Write requests: 150/day
│   └── Top consumers: OrderService, PaymentService
├── Access Logs
│   ├── 2026-01-14 10:30 - ops@company.com updated "DatabaseUrl"
│   ├── 2026-01-14 09:15 - dev@company.com read "FeatureFlags:*"
│   └── 2026-01-14 08:00 - OrderService (managed identity) read config
└── Configuration Snapshots
    ├── 2026-01-14 10:30 (current)
    ├── 2026-01-13 15:00 (previous)
    └── Rollback to any snapshot
```

---

## Pattern vs Traditional Configuration

### Comparison Table

| Aspect | Traditional (File-Based) | External Configuration Store |
|--------|-------------------------|------------------------------|
| **Storage** | Local files per app | Centralized service |
| **Updates** | Requires redeployment | Runtime refresh (no restart) |
| **Synchronization** | Manual (error-prone) | Automatic (eventual consistency) |
| **Versioning** | Git history | Built-in snapshot versioning |
| **Audit Trail** | Git commits | Access logs with user attribution |
| **Sharing** | Copy-paste duplication | Reference single source |
| **Environment Management** | Separate files per env | Labels (dev, staging, prod) |
| **Secret Management** | Risk of exposure | Key Vault reference integration |
| **Scalability** | Poor (N files × M envs) | Excellent (centralized) |
| **Operational Overhead** | High (manual coordination) | Low (automated refresh) |

### Migration Path Example

**Phase 1: Inventory Existing Configuration**
```bash
# Audit current configuration files
find . -name "appsettings*.json" -o -name "*.config"

# Result:
# order-service/appsettings.Production.json
# payment-service/appsettings.Production.json
# shipping-service/appsettings.Production.json
# ... 47 more files
```

**Phase 2: Identify Shared vs Service-Specific**
```bash
# Shared configuration (50% of settings):
- DatabaseUrl (all services use same database)
- CacheUrl (all services use same Redis)
- LoggingLevel (consistent across services)
- FeatureFlags (organization-wide features)

# Service-specific configuration (50% of settings):
- OrderService:MaxRetryAttempts
- PaymentService:ApiKey (Stripe)
- ShippingService:ApiKey (FedEx)
```

**Phase 3: Migrate to External Store**
```bash
# Step 1: Create Azure App Configuration
az appconfig create --name mycompany-config \
  --resource-group prod-rg --location eastus --sku Standard

# Step 2: Import shared configuration
az appconfig kv import --name mycompany-config \
  --source file --path shared-config.json \
  --label production --format json

# Step 3: Import service-specific configuration
az appconfig kv import --name mycompany-config \
  --source file --path order-service-config.json \
  --label production --format json --prefix "OrderService:"

# Step 4: Update application code to use App Configuration
# (See code examples below)

# Step 5: Remove config files from Git (after validation)
git rm appsettings.Production.json
git commit -m "Migrate to Azure App Configuration"
```

**Phase 4: Application Code Changes**
```csharp
// Before: File-based configuration
var builder = WebApplication.CreateBuilder(args);
// Configuration from appsettings.json only

// After: External Configuration Store
var builder = WebApplication.CreateBuilder(args);
builder.Configuration.AddAzureAppConfiguration(options => {
    options.Connect(Environment.GetEnvironmentVariable("APPCONFIGURATION_CONNECTION_STRING"))
           .Select(KeyFilter.Any, builder.Environment.EnvironmentName)
           .ConfigureRefresh(refresh => {
               refresh.Register("AppSettings:Sentinel", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromSeconds(30));
           });
});

var app = builder.Build();
app.UseAzureAppConfiguration();  // Enable background refresh
```

---

## Implementation Example: Azure App Configuration

### Complete Implementation
```csharp
// Program.cs
using Azure.Identity;
using Microsoft.Extensions.Configuration.AzureAppConfiguration;

var builder = WebApplication.CreateBuilder(args);

// Configure Azure App Configuration
builder.Configuration.AddAzureAppConfiguration(options => {
    // Connection using managed identity (no secrets in code!)
    options.Connect(
        new Uri(builder.Configuration["AppConfiguration:Endpoint"]),
        new DefaultAzureCredential()
    );
    
    // Select configuration for current environment
    options.Select(KeyFilter.Any, LabelFilter.Null)  // Shared config (no label)
           .Select(KeyFilter.Any, builder.Environment.EnvironmentName);  // Environment-specific
    
    // Configure refresh with sentinel key
    options.ConfigureRefresh(refresh => {
        refresh.Register("AppSettings:Sentinel", refreshAll: true)
               .SetCacheExpiration(TimeSpan.FromSeconds(30));
    });
    
    // Integrate Key Vault for secrets
    options.ConfigureKeyVault(kv => {
        kv.SetCredential(new DefaultAzureCredential());
    });
});

// Register App Configuration refresh middleware
builder.Services.AddAzureAppConfiguration();

var app = builder.Build();

// Enable configuration refresh
app.UseAzureAppConfiguration();

app.MapGet("/config", (IConfiguration config) => {
    return new {
        DatabaseUrl = config["Database:ConnectionString"],
        FeatureFlag = config["FeatureManagement:BetaCheckout"],
        // Config refreshes automatically every 30 seconds
    };
});

app.Run();
```

### Configuration in Azure App Configuration
```bash
# Shared configuration (no label)
az appconfig kv set --name myapp-config --key "Shared:CacheUrl" --value "redis://cache:6379"

# Production configuration (label: production)
az appconfig kv set --name myapp-config --key "Database:ConnectionString" \
  --value '{"uri":"https://myapp-kv.vault.azure.net/secrets/DbConnection"}' \
  --content-type "application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8" \
  --label production

# Development configuration (label: development)
az appconfig kv set --name myapp-config --key "Database:ConnectionString" \
  --value "Server=localhost;Database=orders-dev" --label development

# Sentinel key for coordinated updates
az appconfig kv set --name myapp-config --key "AppSettings:Sentinel" \
  --value "2026-01-14T10:30:00Z"  # Update timestamp triggers refresh
```

---

## Best Practices

### 1. Design Configuration Hierarchy
```
Organization Level (Shared:*)
  ├─ Database URLs
  ├─ Cache endpoints
  ├─ Logging configuration
  └─ Common feature flags
  
Application Level (AppName:*)
  ├─ Application-specific settings
  ├─ Service-specific endpoints
  └─ Application feature flags
  
Environment Level (Labels)
  ├─ dev (development overrides)
  ├─ staging (staging overrides)
  └─ production (production values)
```

### 2. Implement Graceful Degradation
```csharp
// Fallback strategy if configuration store unavailable
builder.Configuration.AddAzureAppConfiguration(options => {
    try {
        options.Connect(connectionString);
    }
    catch (Exception ex) {
        // Log error, fall back to local configuration
        logger.LogError(ex, "Failed to connect to App Configuration");
        // Application continues with cached/local config
    }
});
```

### 3. Use Sentinel Keys for Coordinated Updates
```bash
# Update multiple related settings, then trigger refresh
az appconfig kv set --name myapp-config --key "Feature:NewUI:Enabled" --value "true"
az appconfig kv set --name myapp-config --key "Feature:NewUI:RolloutPercentage" --value "100"
az appconfig kv set --name myapp-config --key "AppSettings:Sentinel" --value "$(date -u +%Y-%m-%dT%H:%M:%SZ)"

# All apps refresh within 30 seconds, seeing consistent state
```

### 4. Monitor Configuration Access
```bash
# Enable diagnostic logging
az monitor diagnostic-settings create \
  --name config-audit \
  --resource /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.AppConfiguration/configurationStores/myapp-config \
  --logs '[{"category":"HttpRequest","enabled":true}]' \
  --workspace /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.OperationalInsights/workspaces/logs

# Query access patterns
az monitor log-analytics query --workspace logs \
  --analytics-query "AzureDiagnostics | where ResourceProvider == 'MICROSOFT.APPCONFIGURATION' | summarize Count=count() by Identity, OperationName"
```

---

## Key Takeaways

### Pattern Benefits
✅ **Centralized management**: Single source of truth for all configuration  
✅ **Runtime updates**: Change configuration without redeployment  
✅ **Consistency**: All instances converge to same configuration state  
✅ **Audit trail**: Track who changed what, when  
✅ **Versioning**: Multiple configuration versions with rollback capability  

### Implementation Requirements
✅ **Backing store**: Choose appropriate storage (App Configuration, Consul, etcd)  
✅ **Caching layer**: Implement local cache to reduce latency  
✅ **Refresh mechanism**: Background polling or event-driven updates  
✅ **Fallback strategy**: Continue with cached config if store unavailable  
✅ **Monitoring**: Track configuration access and changes  

### When to Use External Configuration Store
✅ Distributed applications with multiple instances  
✅ Need for runtime configuration updates  
✅ Shared configuration across microservices  
✅ Complex configuration versioning requirements  
✅ Centralized audit and compliance needs  

---

**Next**: Learn about Azure DevOps Secure Files for certificate and credential management →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/4-understand-external-configuration-store-patterns)
