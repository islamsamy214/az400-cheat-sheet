# Enable Dynamic Configuration and Feature Flags

**Duration**: 6 minutes

Implement runtime configuration updates and feature toggles without application redeployment, leveraging Azure App Configuration's dynamic refresh capabilities and feature management framework for continuous delivery and controlled rollouts.

---

## Dynamic Configuration Overview

### Traditional Configuration (Static)
**Problem**: Configuration changes require application restart or redeployment.

```
Change Config → Redeploy App → Restart Services → Downtime
```

### Dynamic Configuration (Modern)
**Solution**: Configuration updates applied at runtime without restart.

```
Change Config → App Detects Change → Refresh in Memory → Zero Downtime
```

---

## Azure App Configuration Dynamic Refresh

### Configuration Refresh Mechanisms

#### 1. Polling (Pull Model)

**How It Works**: Application periodically checks App Configuration for changes.

```csharp
// ASP.NET Core - Program.cs
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .ConfigureRefresh(refresh =>
           {
               // Register all keys with prefix for refresh
               refresh.Register("MyApp:*", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromSeconds(30));  // Poll every 30s
           });
});

// Add refresh middleware
builder.Services.AddAzureAppConfiguration();
app.UseAzureAppConfiguration();  // Middleware checks for changes on each HTTP request
```

**Refresh Flow**:
```
1. HTTP Request → Middleware checks cache expiration
2. If expired → Poll App Configuration
3. If changes detected → Refresh configuration in memory
4. Application reads new values (no restart)
```

#### 2. Sentinel Key Pattern

**Purpose**: Avoid unnecessary polling by monitoring a single "sentinel" key.

```csharp
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .ConfigureRefresh(refresh =>
           {
               // Only poll App Configuration if "Sentinel" key changes
               refresh.Register("Sentinel", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromSeconds(30));
           });
});
```

**Deployment Workflow**:
```bash
# 1. Update multiple configuration values
az appconfig kv set --name myapp-config --key "Database:ConnectionString" --value "new-value"
az appconfig kv set --name myapp-config --key "Api:Endpoint" --value "https://api-v2.myapp.com"
az appconfig kv set --name myapp-config --key "Cache:Ttl" --value "600"

# 2. Update sentinel key (triggers refresh)
az appconfig kv set --name myapp-config --key "Sentinel" --value "$(date +%s)"

# Applications detect sentinel change → Refresh all configuration
```

**Benefits**:
- ✅ Single key monitors all configuration changes
- ✅ Reduces polling overhead (no unnecessary API calls)
- ✅ Atomic updates (change multiple values, trigger refresh once)

#### 3. Event-Driven Refresh (Push Model)

**How It Works**: App Configuration sends event to Event Grid → Application refreshes immediately.

```
App Configuration Change → Event Grid → Azure Function → SignalR → Application Refreshes
```

**Implementation**:
```csharp
// Azure Function triggered by Event Grid
[FunctionName("ConfigurationChangeHandler")]
public static async Task Run(
    [EventGridTrigger] EventGridEvent eventGridEvent,
    [SignalR(HubName = "configHub")] IAsyncCollector<SignalRMessage> signalRMessages)
{
    // Notify connected applications to refresh configuration
    await signalRMessages.AddAsync(new SignalRMessage
    {
        Target = "refreshConfiguration",
        Arguments = new[] { eventGridEvent.Subject }
    });
}
```

**Client (Application)**:
```csharp
// Listen for SignalR notification
connection.On<string>("refreshConfiguration", async (key) =>
{
    // Force immediate configuration refresh
    await _configurationRefresher.RefreshAsync();
});
```

---

## Complete Dynamic Configuration Example

### ASP.NET Core Application

**Program.cs**:
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add Azure App Configuration
var connectionString = builder.Configuration.GetConnectionString("AppConfig");
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .Select(KeyFilter.Any, LabelFilter.Null)  // Load all keys
           .ConfigureRefresh(refresh =>
           {
               // Monitor sentinel key for changes
               refresh.Register("Sentinel", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromSeconds(30));
           })
           // Integrate Key Vault for secrets
           .ConfigureKeyVault(kv =>
           {
               kv.SetCredential(new DefaultAzureCredential());
           });
});

// Register configuration refresh service
builder.Services.AddAzureAppConfiguration();

var app = builder.Build();

// Enable configuration refresh middleware
app.UseAzureAppConfiguration();

app.MapGet("/config", (IConfiguration config) =>
{
    return new
    {
        DatabaseConnection = config["Database:ConnectionString"],
        ApiEndpoint = config["Api:Endpoint"],
        CacheTtl = config["Cache:Ttl"]
    };
});

app.Run();
```

**appsettings.json**:
```json
{
  "ConnectionStrings": {
    "AppConfig": "Endpoint=https://myapp-config.azconfig.io;Id=abc;Secret=xyz"
  }
}
```

**Testing Dynamic Refresh**:
```bash
# 1. Query current configuration
curl http://localhost:5000/config
# Output: {"databaseConnection":"Server=db1;...","apiEndpoint":"https://api-v1.myapp.com","cacheTtl":"300"}

# 2. Update configuration in Azure
az appconfig kv set --name myapp-config --key "Api:Endpoint" --value "https://api-v2.myapp.com"
az appconfig kv set --name myapp-config --key "Sentinel" --value "$(date +%s)"

# 3. Wait 30 seconds (cache expiration)
sleep 30

# 4. Query configuration again (automatically refreshed)
curl http://localhost:5000/config
# Output: {"databaseConnection":"Server=db1;...","apiEndpoint":"https://api-v2.myapp.com","cacheTtl":"300"}
```

---

## Feature Flags with Dynamic Refresh

### Feature Flag Configuration

**Azure Portal**:
```
App Configuration → Feature Manager → Create
├── Feature flag name: NewCheckoutFlow
├── Enable feature flag: Yes
├── Add filter: Percentage
│   └── Value: 50  (50% of users)
└── Apply
```

**Azure CLI**:
```bash
# Create feature flag
az appconfig feature set \
  --name myapp-config \
  --feature NewCheckoutFlow \
  --yes

# Add percentage filter
az appconfig feature filter add \
  --name myapp-config \
  --feature NewCheckoutFlow \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":50}'
```

### Dynamic Feature Flag Refresh

```csharp
// Program.cs
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           // Load feature flags
           .UseFeatureFlags(flags =>
           {
               flags.CacheExpirationInterval = TimeSpan.FromSeconds(30);  // Refresh every 30s
               flags.Label = "production";
           });
});

builder.Services.AddFeatureManagement();

// Controller.cs
[ApiController]
[Route("api/checkout")]
public class CheckoutController : ControllerBase
{
    private readonly IFeatureManager _featureManager;
    
    public CheckoutController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }
    
    [HttpPost]
    public async Task<IActionResult> ProcessCheckout([FromBody] Order order)
    {
        // Feature flag checked at runtime (refreshes every 30s)
        if (await _featureManager.IsEnabledAsync("NewCheckoutFlow"))
        {
            return Ok(await _newCheckoutService.ProcessAsync(order));
        }
        else
        {
            return Ok(await _legacyCheckoutService.ProcessAsync(order));
        }
    }
}
```

### Progressive Rollout Scenario

**Week 1: Enable for 10% of users**
```bash
az appconfig feature filter update \
  --name myapp-config \
  --feature NewCheckoutFlow \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":10}'

# No deployment required - change takes effect in 30s
```

**Week 2: Increase to 50% (based on positive metrics)**
```bash
az appconfig feature filter update \
  --name myapp-config \
  --feature NewCheckoutFlow \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":50}'
```

**Week 3: Full rollout (100%)**
```bash
az appconfig feature enable --name myapp-config --feature NewCheckoutFlow
az appconfig feature filter remove \
  --name myapp-config \
  --feature NewCheckoutFlow \
  --filter-name Microsoft.Percentage \
  --yes
```

**Result**: Zero-downtime progressive rollout with instant rollback capability.

---

## Targeting Filters (User Segmentation)

### Targeting Configuration

```bash
# Create feature flag with targeting filter
az appconfig feature set \
  --name myapp-config \
  --feature AdvancedReports \
  --yes

az appconfig feature filter add \
  --name myapp-config \
  --feature AdvancedReports \
  --filter-name Microsoft.Targeting \
  --filter-parameters '{
    "Audience": {
      "Users": ["alice@company.com", "bob@company.com"],
      "Groups": [
        {
          "Name": "beta-testers",
          "RolloutPercentage": 100
        },
        {
          "Name": "enterprise-customers",
          "RolloutPercentage": 50
        }
      ],
      "DefaultRolloutPercentage": 0
    }
  }'
```

### Targeting Context in Application

```csharp
// Define targeting context provider
public class UserTargetingContextAccessor : ITargetingContextAccessor
{
    private readonly IHttpContextAccessor _httpContextAccessor;
    
    public UserTargetingContextAccessor(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }
    
    public ValueTask<TargetingContext> GetContextAsync()
    {
        var httpContext = _httpContextAccessor.HttpContext;
        var user = httpContext.User;
        
        return new ValueTask<TargetingContext>(new TargetingContext
        {
            UserId = user.Identity.Name,
            Groups = user.FindAll("groups").Select(c => c.Value).ToList()
        });
    }
}

// Register targeting
builder.Services.AddSingleton<ITargetingContextAccessor, UserTargetingContextAccessor>();
builder.Services.AddFeatureManagement()
                .WithTargeting<UserTargetingContextAccessor>();

// Use in controller
if (await _featureManager.IsEnabledAsync("AdvancedReports"))
{
    // User is in targeting audience
    return Ok(_reportService.GetAdvancedReports());
}
else
{
    return Ok(_reportService.GetStandardReports());
}
```

**Result**: Alice and Bob see advanced reports; 100% of beta-testers see them; 50% of enterprise customers see them; others see standard reports.

---

## Time Window Filters

### Scheduled Feature Activation

```bash
# Enable feature during specific time window (flash sale)
az appconfig feature set \
  --name myapp-config \
  --feature FlashSale \
  --yes

az appconfig feature filter add \
  --name myapp-config \
  --feature FlashSale \
  --filter-name Microsoft.TimeWindow \
  --filter-parameters '{
    "Start": "2026-01-14T09:00:00Z",
    "End": "2026-01-14T17:00:00Z"
  }'
```

**Application Code**:
```csharp
[HttpGet("products")]
public async Task<IActionResult> GetProducts()
{
    var products = await _productService.GetAllProductsAsync();
    
    // Feature automatically enabled during time window
    if (await _featureManager.IsEnabledAsync("FlashSale"))
    {
        // Apply flash sale discounts
        products = products.Select(p => new Product
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price * 0.5m,  // 50% off
            IsFlashSale = true
        });
    }
    
    return Ok(products);
}
```

**Result**: Flash sale automatically activates at 9:00 AM and deactivates at 5:00 PM UTC without code deployment.

---

## A/B Testing Integration

### Experiment Setup

```csharp
// Feature flag with percentage filter (50/50 split)
az appconfig feature filter add \
  --name myapp-config \
  --feature NewSearchAlgorithm \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":50}'

// Application tracks experiment results
[HttpGet("search")]
public async Task<IActionResult> Search([FromQuery] string query)
{
    if (await _featureManager.IsEnabledAsync("NewSearchAlgorithm"))
    {
        // Variant B: New algorithm
        var results = await _newSearchService.SearchAsync(query);
        _telemetry.TrackEvent("Search_VariantB", new Dictionary<string, string>
        {
            { "query", query },
            { "resultCount", results.Count.ToString() },
            { "variant", "B" }
        });
        return Ok(results);
    }
    else
    {
        // Variant A: Legacy algorithm
        var results = await _legacySearchService.SearchAsync(query);
        _telemetry.TrackEvent("Search_VariantA", new Dictionary<string, string>
        {
            { "query", query },
            { "resultCount", results.Count.ToString() },
            { "variant", "A" }
        });
        return Ok(results);
    }
}
```

**Analysis (Application Insights Query)**:
```kusto
customEvents
| where name in ("Search_VariantA", "Search_VariantB")
| summarize 
    SearchCount = count(),
    AvgResultCount = avg(todouble(customDimensions["resultCount"]))
  by Variant = tostring(customDimensions["variant"])
| project Variant, SearchCount, AvgResultCount
```

**Result**:
```
Variant | SearchCount | AvgResultCount
--------|-------------|---------------
A       | 5,234       | 8.3
B       | 5,189       | 12.7  ← Winner (50% more results)
```

**Rollout Decision**:
```bash
# Variant B wins → Enable for 100%
az appconfig feature enable --name myapp-config --feature NewSearchAlgorithm
az appconfig feature filter remove \
  --name myapp-config \
  --feature NewSearchAlgorithm \
  --filter-name Microsoft.Percentage \
  --yes
```

---

## Circuit Breaker Pattern

### Operational Feature Flags

```bash
# Create circuit breaker flag (default: enabled)
az appconfig feature set \
  --name myapp-config \
  --feature UseExternalCache \
  --yes
```

**Application Implementation**:
```csharp
[HttpGet("data/{id}")]
public async Task<IActionResult> GetData(int id)
{
    // Circuit breaker: Disable external cache if experiencing issues
    if (await _featureManager.IsEnabledAsync("UseExternalCache"))
    {
        try
        {
            var data = await _cacheService.GetAsync(id);
            if (data != null)
            {
                _telemetry.TrackEvent("CacheHit");
                return Ok(data);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Cache service error");
            _telemetry.TrackEvent("CacheFailed");
            // Fall through to database
        }
    }
    
    // Fallback to database
    _telemetry.TrackEvent("DatabaseFetch");
    var data = await _dbService.GetAsync(id);
    return Ok(data);
}
```

**Incident Response**:
```bash
# Redis cache experiencing issues (high latency, timeouts)
# Disable cache feature flag (instant fallback to database)
az appconfig feature disable --name myapp-config --feature UseExternalCache

# Applications refresh in 30 seconds → All requests use database
# Result: Degraded performance, but zero downtime

# After cache issues resolved
az appconfig feature enable --name myapp-config --feature UseExternalCache
```

---

## Best Practices

### 1. Use Sentinel Key for Atomic Updates

```bash
# ✅ Good: Update multiple values, then trigger refresh with sentinel
az appconfig kv set --name myapp-config --key "Database:Host" --value "new-db.database.windows.net"
az appconfig kv set --name myapp-config --key "Database:Port" --value "1433"
az appconfig kv set --name myapp-config --key "Sentinel" --value "$(date +%s)"

# ❌ Bad: Applications refresh after each change (inconsistent state)
```

### 2. Set Appropriate Cache Expiration

```csharp
// ✅ Good: 30-60 seconds (balance between freshness and API calls)
.SetCacheExpiration(TimeSpan.FromSeconds(30))

// ❌ Too frequent: 5 seconds (excessive API calls, cost)
// ❌ Too infrequent: 10 minutes (slow configuration updates)
```

### 3. Monitor Configuration Refresh

```csharp
// Log configuration refresh events
builder.Services.Configure<AzureAppConfigurationOptions>(options =>
{
    options.OnConfigurationRefreshed = args =>
    {
        _logger.LogInformation("Configuration refreshed at {Time}", DateTime.UtcNow);
        _telemetry.TrackEvent("ConfigurationRefreshed", new Dictionary<string, string>
        {
            { "changedKeys", string.Join(",", args.ChangedKeys) }
        });
    };
});
```

### 4. Implement Feature Flag Cleanup

```bash
# Remove obsolete feature flags after full rollout (reduce technical debt)
# After NewCheckoutFlow at 100% for 30 days:
az appconfig feature delete --name myapp-config --feature NewCheckoutFlow --yes

# Remove feature flag checks from code
if (await _featureManager.IsEnabledAsync("NewCheckoutFlow"))  // ← Delete this
{
    return Ok(await _newCheckoutService.ProcessAsync(order));
}
else  // ← Delete this
{
    return Ok(await _legacyCheckoutService.ProcessAsync(order));  // ← Delete this
}

// Keep winning implementation only
return Ok(await _newCheckoutService.ProcessAsync(order));
```

### 5. Test Feature Flag Behavior

```csharp
// Unit test feature flag behavior
[Fact]
public async Task Checkout_WithNewCheckoutFlagEnabled_UsesNewService()
{
    // Arrange
    var featureManager = new Mock<IFeatureManager>();
    featureManager.Setup(fm => fm.IsEnabledAsync("NewCheckoutFlow"))
                  .ReturnsAsync(true);
    
    var controller = new CheckoutController(featureManager.Object);
    
    // Act
    var result = await controller.ProcessCheckout(new Order());
    
    // Assert
    Assert.IsType<OkObjectResult>(result);
    // Verify new checkout service was called
}
```

---

## Real-World Example: E-Commerce Platform

### Scenario: New Payment Provider Rollout

**Initial Setup**:
```bash
# Create feature flag (disabled initially)
az appconfig feature set \
  --name myapp-config \
  --feature NewPaymentProvider \
  --label production

az appconfig feature disable \
  --name myapp-config \
  --feature NewPaymentProvider
```

**Application Code**:
```csharp
[HttpPost("checkout")]
public async Task<IActionResult> ProcessCheckout([FromBody] Order order)
{
    if (await _featureManager.IsEnabledAsync("NewPaymentProvider"))
    {
        // New provider: Lower fees, faster processing
        var result = await _stripeService.ProcessPaymentAsync(order);
        _telemetry.TrackEvent("Payment_Stripe", new Dictionary<string, string>
        {
            { "amount", order.Total.ToString() },
            { "success", result.Success.ToString() }
        });
        return Ok(result);
    }
    else
    {
        // Legacy provider: Higher fees
        var result = await _legacyPaymentService.ProcessPaymentAsync(order);
        _telemetry.TrackEvent("Payment_Legacy", new Dictionary<string, string>
        {
            { "amount", order.Total.ToString() },
            { "success", result.Success.ToString() }
        });
        return Ok(result);
    }
}
```

**Rollout Plan**:

**Phase 1: Internal Testing (100% of employee accounts)**
```bash
az appconfig feature filter add \
  --name myapp-config \
  --feature NewPaymentProvider \
  --filter-name Microsoft.Targeting \
  --filter-parameters '{
    "Audience": {
      "Groups": [{"Name": "employees", "RolloutPercentage": 100}],
      "DefaultRolloutPercentage": 0
    }
  }'

az appconfig feature enable --name myapp-config --feature NewPaymentProvider
```

**Phase 2: Beta Customers (10%)**
```bash
az appconfig feature filter update \
  --name myapp-config \
  --feature NewPaymentProvider \
  --filter-name Microsoft.Targeting \
  --filter-parameters '{
    "Audience": {
      "Groups": [
        {"Name": "employees", "RolloutPercentage": 100},
        {"Name": "beta-customers", "RolloutPercentage": 10}
      ],
      "DefaultRolloutPercentage": 0
    }
  }'

# Monitor metrics for 1 week
```

**Phase 3: General Availability (100%)**
```bash
# Positive results: 15% faster checkout, 8% fewer failed transactions
az appconfig feature enable --name myapp-config --feature NewPaymentProvider
az appconfig feature filter remove \
  --name myapp-config \
  --feature NewPaymentProvider \
  --filter-name Microsoft.Targeting \
  --yes
```

**Result**: Seamless rollout with zero downtime, $50K annual savings in transaction fees.

---

## Key Takeaways

✅ **Dynamic configuration**: Runtime updates without application restart (polling, sentinel key, event-driven)  
✅ **Feature flags**: Progressive rollouts (10% → 50% → 100%) with instant rollback  
✅ **Targeting filters**: User segmentation (specific users, groups, percentages)  
✅ **Time window filters**: Scheduled feature activation (flash sales, maintenance windows)  
✅ **Circuit breakers**: Operational flags for dependency fallback  
✅ **A/B testing**: Data-driven decisions through experimentation  
✅ **Best practices**: Sentinel keys, 30-60s cache expiration, feature flag cleanup  

---

**Next**: Test your knowledge with the module assessment →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/13-enable-dynamic-configuration-feature-flags)
