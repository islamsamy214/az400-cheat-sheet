# Examine Key-Value Pairs

**Duration**: 4 minutes

Azure App Configuration uses key-value pairs as the fundamental storage architecture, with support for hierarchical keys, labels, and content types for flexible configuration management.

---

## Keys

### Key Fundamentals
**Definition**: Keys are identifiers for key-value pairs enabling configuration storage and retrieval.

**Characteristics**:
- **Case-sensitive**: `app1` and `App1` are distinct keys
- **Unicode strings**: Support internationalization (non-ASCII characters)
- **Hierarchical structure**: Use delimiters (`:` or `/`) for organization
- **Atomic treatment**: App Configuration doesn't parse key structure (treated as opaque strings)
- **Size limit**: Key + value + attributes = 10,000 characters combined

**Reserved Characters**: `*`, `,`, `\` (require escaping with `\{character}`)

###Design Key Namespaces

#### Flat vs Hierarchical Naming

**Flat Naming** (Anti-Pattern):
```bash
# ❌ Poor readability, hard to manage
DatabaseConnectionString
CacheConnectionString
PaymentAPIKey
PaymentAPIEndpoint
ShippingAPIKey
ShippingAPIEndpoint
```

**Hierarchical Naming** (Best Practice):
```bash
# ✅ Semantic structure, easy to filter
ConnectionStrings:Database
ConnectionStrings:Cache
PaymentAPI:Key
PaymentAPI:Endpoint
ShippingAPI:Key
ShippingAPI:Endpoint
```

#### Common Hierarchy Patterns

**1. Component/Service-Based**:
```bash
AppName:Service1:ApiEndpoint = "https://api1.company.com"
AppName:Service1:Timeout = "30"
AppName:Service2:ApiEndpoint = "https://api2.company.com"
AppName:Service2:Timeout = "60"
```

**Benefits**:
- ✅ Logical grouping by service
- ✅ Easy to query (`AppName:Service1:*`)
- ✅ Clear ownership boundaries

**2. Region-Based**:
```bash
AppName:Region:EastUS:DbEndpoint = "eastus-sql.database.windows.net"
AppName:Region:WestUS:DbEndpoint = "westus-sql.database.windows.net"
AppName:Region:EastUS:CacheEndpoint = "eastus-cache.redis.cache.windows.net"
```

**Benefits**:
- ✅ Multi-region deployment support
- ✅ Region-specific failover configuration
- ✅ Geographic redundancy management

**3. Feature-Based**:
```bash
FeatureManagement:BetaCheckout:Enabled = "true"
FeatureManagement:BetaCheckout:RolloutPercentage = "20"
FeatureManagement:NewUI:Enabled = "false"
FeatureManagement:NewUI:TargetAudience = "internal"
```

**Benefits**:
- ✅ Feature toggle organization
- ✅ Progressive rollout configuration
- ✅ Clear feature boundaries

**4. Layered Hierarchy**:
```bash
# Multi-level structure
MyApp:Database:Primary:ConnectionString
MyApp:Database:Primary:MaxPoolSize = "100"
MyApp:Database:Secondary:ConnectionString
MyApp:Database:Secondary:MaxPoolSize = "50"
MyApp:Cache:Redis:Endpoint
MyApp:Cache:Redis:Timeout = "5000"
MyApp:Logging:Level:Default = "Information"
MyApp:Logging:Level:Microsoft = "Warning"
```

**Namespace Design Guidelines**:
- ✅ Use consistent delimiters (`:` or `/`, not mixed)
- ✅ Limit hierarchy depth (3-4 levels max for readability)
- ✅ Group related settings together
- ✅ Use plural for collections (`ConnectionStrings`, not `ConnectionString`)
- ✅ Follow framework conventions (ASP.NET Core uses `:` delimiter)

### Query Key Values

**Pattern Matching**: Retrieve keys using wildcard patterns.

```bash
# Get all keys for Service1
az appconfig kv list --name myapp-config --key "AppName:Service1:*"

# Get all ConnectionStrings
az appconfig kv list --name myapp-config --key "ConnectionStrings:*"

# Get all keys (no filter)
az appconfig kv list --name myapp-config --key "*"

# Get specific key with label
az appconfig kv list --name myapp-config --key "Database:Url" --label "production"
```

**Application Code Queries**:
```csharp
// .NET: Get section
var databaseConfig = configuration.GetSection("ConnectionStrings");
var primaryConnection = databaseConfig["Database"];  // ConnectionStrings:Database

// Get all keys matching pattern
var settings = await client.GetConfigurationSettingsAsync(
    new SettingSelector { KeyFilter = "AppName:Service1:*" }
);

// Iterate results
await foreach (var setting in settings)
{
    Console.WriteLine($"{setting.Key} = {setting.Value}");
}
```

---

## Labels

### Label Purpose
**Definition**: Optional attribute enabling key differentiation through variant designation.

**Key Insight**: Same key with different labels = distinct configuration entries.

```bash
# Three distinct entries in App Configuration
Key: AppName:DbEndpoint, Label: Test       → Value: "test-db.company.com"
Key: AppName:DbEndpoint, Label: Staging    → Value: "staging-db.company.com"
Key: AppName:DbEndpoint, Label: Production → Value: "prod-db.company.com"
```

**Default Label**: Empty string or `null` (label-less entries).

### Common Label Patterns

**1. Environment Differentiation** (Most Common):
```bash
# Same configuration key, different values per environment
az appconfig kv set --name myapp-config --key "LogLevel" --value "Debug" --label "development"
az appconfig kv set --name myapp-config --key "LogLevel" --value "Information" --label "staging"
az appconfig kv set --name myapp-config --key "LogLevel" --value "Warning" --label "production"

# Application selects based on environment
var environment = builder.Environment.EnvironmentName;  // "Production"
options.Select(KeyFilter.Any, environment);  // Reads "production" label
```

**2. Version Labeling**:
```bash
# Configuration versioning for application versions
Key: ApiEndpoint, Label: v1.0 → "https://api.v1.company.com"
Key: ApiEndpoint, Label: v2.0 → "https://api.v2.company.com"
Key: ApiEndpoint, Label: v2.1 → "https://api.v2.company.com/v2.1"

# Blue-green deployment: Switch label
# Blue: Label = v1.0
# Green: Label = v2.0
```

**3. Feature Branch Configuration**:
```bash
# Per-branch configuration for CI/CD
Key: FeatureFlag, Label: main → "false"
Key: FeatureFlag, Label: feature/new-checkout → "true"
Key: FeatureFlag, Label: hotfix/bug-123 → "false"

# Pipeline selects label based on branch
--label "$(Build.SourceBranchName)"
```

**4. Team Isolation**:
```bash
# Multi-tenant configuration
Key: TenantConfig:MaxUsers, Label: team-alpha → "100"
Key: TenantConfig:MaxUsers, Label: team-beta → "500"
Key: TenantConfig:MaxUsers, Label: enterprise → "unlimited"
```

**5. Geographic Regions**:
```bash
# Region-specific configuration
Key: ComplianceSettings, Label: us-east → "GDPR,CCPA"
Key: ComplianceSettings, Label: eu-west → "GDPR"
Key: ComplianceSettings, Label: ap-south → "PDPA"
```

### Label Selection Strategy

**Application Code**:
```csharp
// Strategy 1: Single label (environment-specific only)
options.Select(KeyFilter.Any, "production");

// Strategy 2: Layered labels (shared + environment-specific)
options.Select(KeyFilter.Any, LabelFilter.Null)  // Shared config (no label)
       .Select(KeyFilter.Any, "production");  // Production overrides

// Strategy 3: Multiple labels with precedence
options.Select(KeyFilter.Any, LabelFilter.Null)  // 1. Shared (lowest priority)
       .Select(KeyFilter.Any, "production")       // 2. Environment
       .Select(KeyFilter.Any, "v2.0");            // 3. Version (highest priority)
```

**Precedence Rules**:
```
Configuration loading order (last wins):
1. Shared configuration (no label)
2. Environment-specific (label: production)
3. Version-specific (label: v2.0)

Example:
Key: LogLevel, Label: null        → "Information" (shared default)
Key: LogLevel, Label: production  → "Warning" (production override)

Final value in production: "Warning" (environment-specific wins)
```

### Version Key Values with Labels

**Scenario**: Track configuration changes across application releases.

```bash
# Release 1.0 configuration
az appconfig kv set --name myapp-config --key "MaxConnections" --value "50" --label "v1.0"
az appconfig kv set --name myapp-config --key "Timeout" --value "30" --label "v1.0"

# Release 2.0 configuration (increased limits)
az appconfig kv set --name myapp-config --key "MaxConnections" --value "100" --label "v2.0"
az appconfig kv set --name myapp-config --key "Timeout" --value "60" --label "v2.0"
az appconfig kv set --name myapp-config --key "NewFeature" --value "enabled" --label "v2.0"

# Rollback strategy: Switch application to read v1.0 label
# No configuration changes needed, just label selection in code
```

**Git Commit Versioning**:
```bash
# Tag configuration with Git commit SHA
COMMIT_SHA=$(git rev-parse HEAD | cut -c1-8)
az appconfig kv set --name myapp-config --key "DeploymentVersion" --value "$(git describe --tags)" --label "$COMMIT_SHA"

# Associate configuration with specific commit
# Enables: "What configuration was deployed with commit abc12345?"
```

---

## Values

### Value Characteristics
**Definition**: Unicode strings assigned to keys supporting comprehensive character sets.

**Size**: Key + value + attributes ≤ 10,000 characters total.

**Content Types**: Values can have optional content type metadata.

```bash
# Plain text value
az appconfig kv set --name myapp-config --key "AppName" --value "MyApplication"

# JSON value with content type
az appconfig kv set --name myapp-config --key "DatabaseConfig" \
  --value '{"server":"prod-db","port":5432,"ssl":true}' \
  --content-type "application/json"

# Key Vault reference with content type
az appconfig kv set --name myapp-config --key "ApiKey" \
  --value '{"uri":"https://myapp-kv.vault.azure.net/secrets/ApiKey"}' \
  --content-type "application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8"
```

### Content Types

**Purpose**: Metadata indicating value format for proper application processing.

**Common Content Types**:

| Content Type | Use Case | Example Value |
|--------------|----------|---------------|
| `text/plain` | Simple strings | `"localhost:5432"` |
| `application/json` | Structured data | `{"host":"localhost","port":5432}` |
| `application/xml` | XML configuration | `<config><host>localhost</host></config>` |
| `application/vnd.microsoft.appconfig.keyvaultref+json` | Key Vault reference | `{"uri":"https://kv.vault.azure.net/secrets/Secret"}` |

**Application Processing**:
```csharp
// Automatic JSON deserialization based on content type
var setting = await client.GetConfigurationSettingAsync("DatabaseConfig");

if (setting.ContentType == "application/json")
{
    var config = JsonSerializer.Deserialize<DatabaseConfig>(setting.Value);
    Console.WriteLine($"Server: {config.Server}, Port: {config.Port}");
}
```

### Complex Value Examples

**1. JSON Configuration**:
```json
// Key: PaymentGateway:Config
// Content-Type: application/json
{
  "endpoint": "https://api.stripe.com",
  "apiVersion": "2023-10-16",
  "timeout": 30000,
  "retryPolicy": {
    "maxRetries": 3,
    "backoffMs": 1000
  },
  "webhooks": {
    "enabled": true,
    "signingSecret": "${STRIPE_WEBHOOK_SECRET}"
  }
}
```

**2. Array Values**:
```json
// Key: AllowedOrigins
// Content-Type: application/json
[
  "https://app.company.com",
  "https://admin.company.com",
  "https://mobile.company.com"
]
```

**3. Nested Configuration**:
```json
// Key: EmailSettings
// Content-Type: application/json
{
  "smtp": {
    "host": "smtp.sendgrid.net",
    "port": 587,
    "useSsl": true,
    "from": "noreply@company.com"
  },
  "templates": {
    "welcome": "template-001",
    "resetPassword": "template-002"
  },
  "rateLimit": {
    "maxPerHour": 1000,
    "burstLimit": 50
  }
}
```

---

## Encryption and Security

### Data Protection
**At Rest**: All configuration data encrypted using AES-256 with Azure-managed keys.

**In Transit**: HTTPS (TLS 1.2+) required for all API calls.

```bash
# Connection strings use HTTPS only
Endpoint=https://myapp-config.azconfig.io;Id=xxx;Secret=yyy

# API calls enforce TLS
GET https://myapp-config.azconfig.io/kv/MyKey HTTP/1.1
```

### Key Vault Integration
**Critical Rule**: App Configuration is NOT a secret store.

```bash
# ❌ NEVER store secrets directly in App Configuration
az appconfig kv set --name myapp-config --key "DatabasePassword" --value "SuperSecret123"

# ✅ ALWAYS reference Key Vault for secrets
az keyvault secret set --vault-name myapp-kv --name DatabasePassword --value "SuperSecret123"
az appconfig kv set --name myapp-config --key "DatabasePassword" \
  --value '{"uri":"https://myapp-kv.vault.azure.net/secrets/DatabasePassword"}' \
  --content-type "application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8"
```

**Why Reference Key Vault**:
- ✅ Secrets in HSM-backed storage
- ✅ Automated secret rotation
- ✅ Granular access control (separate from config access)
- ✅ Audit logging specific to secret access
- ✅ Compliance requirements (SOC 2, HIPAA)

### Application Secret Resolution
```csharp
// Application code doesn't know difference between direct value vs Key Vault reference
var password = configuration["DatabasePassword"];
// Returns actual secret value (resolved transparently from Key Vault)

// SDK configuration for Key Vault resolution
builder.Configuration.AddAzureAppConfiguration(options => {
    options.Connect(connectionString)
           .ConfigureKeyVault(kv => {
               kv.SetCredential(new DefaultAzureCredential());  // Managed identity
           });
});
```

---

## Key-Value Operations

### CRUD Operations

**Create/Update**:
```bash
# Create new key-value pair
az appconfig kv set \
  --name myapp-config \
  --key "NewFeature:Enabled" \
  --value "true" \
  --label "production"

# Update existing value
az appconfig kv set \
  --name myapp-config \
  --key "NewFeature:Enabled" \
  --value "false" \
  --label "production"
```

**Read**:
```bash
# Get single key
az appconfig kv show \
  --name myapp-config \
  --key "NewFeature:Enabled" \
  --label "production"

# List keys with filter
az appconfig kv list \
  --name myapp-config \
  --key "NewFeature:*" \
  --label "production"
```

**Delete**:
```bash
# Delete specific key-value
az appconfig kv delete \
  --name myapp-config \
  --key "NewFeature:Enabled" \
  --label "production" \
  --yes
```

### Import/Export

**Import from JSON**:
```json
// config.json
{
  "ConnectionStrings": {
    "Database": "Server=localhost;Database=orders",
    "Cache": "redis://localhost:6379"
  },
  "FeatureFlags": {
    "BetaCheckout": "true",
    "NewUI": "false"
  }
}
```

```bash
# Import JSON file
az appconfig kv import \
  --name myapp-config \
  --source file \
  --path config.json \
  --format json \
  --label "development" \
  --yes

# Result: Hierarchical keys created
# ConnectionStrings:Database
# ConnectionStrings:Cache
# FeatureFlags:BetaCheckout
# FeatureFlags:NewUI
```

**Export to JSON**:
```bash
# Export all production configuration
az appconfig kv export \
  --name myapp-config \
  --destination file \
  --path prod-config.json \
  --format json \
  --label "production" \
  --yes
```

**Import from App Service**:
```bash
# Import existing App Service app settings
az appconfig kv import \
  --name myapp-config \
  --source appservice \
  --appservice-account /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app-name} \
  --label "production"
```

---

## Best Practices

### 1. Hierarchical Key Design
```bash
# ✅ Good: Clear hierarchy with consistent delimiter
AppName:Service:Database:ConnectionString
AppName:Service:Database:MaxPoolSize
AppName:Service:Cache:Endpoint

# ❌ Poor: Flat structure, hard to filter
AppNameServiceDatabaseConnectionString
AppNameServiceDatabaseMaxPoolSize
```

### 2. Label Strategy
```bash
# ✅ Good: Shared + environment-specific
Key: LogLevel, Label: null        → "Information" (shared default)
Key: LogLevel, Label: production  → "Warning" (production override)

# ❌ Poor: Duplicate keys without labels
Key: LogLevel-Development → "Debug"
Key: LogLevel-Production  → "Warning"
```

### 3. Content Type Usage
```bash
# ✅ Good: Explicit content type for structured data
az appconfig kv set --name myapp-config --key "Config" \
  --value '{"key":"value"}' --content-type "application/json"

# ❌ Poor: JSON without content type (treated as plain text)
az appconfig kv set --name myapp-config --key "Config" \
  --value '{"key":"value"}'
```

### 4. Key Vault References
```bash
# ✅ Good: Secrets in Key Vault, referenced from App Configuration
az keyvault secret set --vault-name myapp-kv --name Secret --value "secret123"
az appconfig kv set --name myapp-config --key "ApiKey" \
  --value '{"uri":"https://myapp-kv.vault.azure.net/secrets/Secret"}' \
  --content-type "application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8"

# ❌ Poor: Secrets directly in App Configuration
az appconfig kv set --name myapp-config --key "ApiKey" --value "secret123"
```

### 5. Size Optimization
```bash
# ⚠️ Size limit: 10,000 characters (key + value + attributes)

# ❌ Poor: Large JSON blob exceeding limit
az appconfig kv set --name myapp-config --key "LargeConfig" \
  --value "{... 15,000 character JSON ...}"  # FAILS

# ✅ Good: Split large configuration into multiple keys
az appconfig kv set --name myapp-config --key "Config:Part1" --value "{...}"
az appconfig kv set --name myapp-config --key "Config:Part2" --value "{...}"
az appconfig kv set --name myapp-config --key "Config:Part3" --value "{...}"
```

---

## Real-World Example

### E-Commerce Platform Configuration

**Hierarchy Design**:
```bash
# Database configuration
ConnectionStrings:OrderDatabase (Label: production) = Key Vault reference
ConnectionStrings:ProductDatabase (Label: production) = Key Vault reference
ConnectionStrings:ReadReplica (Label: production) = "Server=read-replica..."

# API endpoints
ExternalAPIs:PaymentGateway:Endpoint = "https://api.stripe.com"
ExternalAPIs:PaymentGateway:ApiKey (Label: production) = Key Vault reference
ExternalAPIs:ShippingProvider:Endpoint = "https://api.fedex.com"
ExternalAPIs:ShippingProvider:ApiKey (Label: production) = Key Vault reference

# Feature flags
FeatureManagement:BetaCheckout:Enabled (Label: production) = "true"
FeatureManagement:BetaCheckout:RolloutPercentage = "20"
FeatureManagement:NewUI:Enabled (Label: production) = "false"

# Performance settings
Performance:Cache:TTL = "300"
Performance:Cache:MaxSize = "1000"
Performance:RateLimiting:RequestsPerMinute = "1000"

# Logging
Logging:LogLevel:Default (Label: production) = "Warning"
Logging:LogLevel:Microsoft (Label: production) = "Error"
Logging:LogLevel:Default (Label: development) = "Debug"
```

**Application Code**:
```csharp
// Read configuration with hierarchy
var orderDbConnection = configuration["ConnectionStrings:OrderDatabase"];
var paymentEndpoint = configuration["ExternalAPIs:PaymentGateway:Endpoint"];
var paymentApiKey = configuration["ExternalAPIs:PaymentGateway:ApiKey"];  // Resolved from Key Vault

// Structured configuration binding
public class PerformanceSettings
{
    public CacheSettings Cache { get; set; }
    public RateLimitingSettings RateLimiting { get; set; }
}

services.Configure<PerformanceSettings>(configuration.GetSection("Performance"));
```

---

## Key Takeaways

### Key Design
✅ **Hierarchical structure**: Use `:` or `/` delimiters for organization  
✅ **Consistent naming**: Follow conventions (plural for collections)  
✅ **Pattern queries**: Design keys to support wildcard filtering  
✅ **Size awareness**: Keep key + value + attributes < 10,000 chars  

### Label Usage
✅ **Environment differentiation**: Primary use case for labels  
✅ **Layered selection**: Shared + environment-specific configuration  
✅ **Version tracking**: Label with application versions or Git commits  
✅ **Multi-tenancy**: Isolate configuration per team/tenant  

### Value Management
✅ **Content types**: Specify format for structured data processing  
✅ **Key Vault references**: Always reference secrets, never store directly  
✅ **Encryption**: Data protected at rest (AES-256) and in transit (TLS 1.2+)  
✅ **Complex values**: Use JSON for structured configuration  

---

**Next**: Learn about feature management and feature flags in App Configuration →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/7-examine-key-value-pairs)
