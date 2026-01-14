# Examine App Configuration Feature Management

**Duration**: 4 minutes

Feature management in Azure App Configuration decouples feature releases from code deployments, enabling dynamic feature activation through centralized flag management.

---

## Feature Management Overview

### Core Concept
**Feature Management**: Software development practice separating feature releases from code deployments through runtime configuration toggles (feature flags).

**Alternative Names**: Feature toggles, feature switches, conditional features.

**Key Benefit**: Deploy code with hidden features → Enable features instantly without redeployment.

---

## Basic Concepts

### 1. Feature Flag
**Definition**: Binary state variable (on/off) with associated code block determining execution.

```csharp
if (featureFlag == true) {
    // Execute new feature code
    return View("NewCheckoutFlow");
} else {
    // Execute legacy code
    return View("LegacyCheckoutFlow");
}
```

### 2. Feature Manager
**Definition**: Application package managing feature flag lifecycles including caching and state updates.

```csharp
// Feature manager from App Configuration
services.AddFeatureManagement();

// Check feature status
if (await _featureManager.IsEnabledAsync("BetaCheckout"))
{
    // Feature enabled
}
```

### 3. Filter
**Definition**: Evaluation rules determining feature flag state based on criteria.

**Common Filters**:
- **Percentage**: Enable for X% of users
- **User Groups**: Enable for specific user segments
- **Time Window**: Enable during specific time periods
- **Browser/Device**: Enable based on client type
- **Geographic**: Enable for specific regions

---

## Feature Flag Usage in Code

### Basic Implementation

```csharp
// Simple boolean flag
bool featureFlag = true;

if (featureFlag) {
    // Run new feature code
    ProcessPaymentV2();
} else {
    // Run legacy code
    ProcessPaymentV1();
}
```

### Dynamic Evaluation

```csharp
// Rule-based flag evaluation
bool featureFlag = isBetaUser();  // Evaluate at runtime

if (featureFlag) {
    return View("BetaFeatures");
} else {
    return View("StandardFeatures");
}
```

### Complete Conditional Logic

```csharp
if (featureFlag) {
    // New feature logic
    var result = await _newCheckoutService.ProcessAsync();
    return Ok(result);
} else {
    // Legacy fallback
    var result = await _legacyCheckoutService.ProcessAsync();
    return Ok(result);
}
```

---

## Feature Flag Declaration

### Configuration Structure

**Feature flags include**:
- **Name**: Unique identifier
- **State**: Enabled/disabled
- **Filters**: Conditions for enablement

### appsettings.json Declaration

```json
{
  "FeatureManagement": {
    "FeatureA": true,  // Simple on/off
    "FeatureB": false,  // Disabled
    "FeatureC": {  // Conditional with filters
      "EnabledFor": [
        {
          "Name": "Percentage",
          "Parameters": {
            "Value": 50  // Enable for 50% of users
          }
        }
      ]
    }
  }
}
```

### Azure App Configuration Declaration

```bash
# Simple feature flag
az appconfig feature set \
  --name myapp-config \
  --feature BetaCheckout \
  --yes \
  --label production

# Feature flag with percentage filter
az appconfig feature filter add \
  --name myapp-config \
  --feature BetaCheckout \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":20}'

# Feature flag with targeting filter
az appconfig feature filter add \
  --name myapp-config \
  --feature BetaCheckout \
  --filter-name Microsoft.Targeting \
  --filter-parameters '{
    "Audience": {
      "Users": ["alice@company.com", "bob@company.com"],
      "Groups": ["beta-testers"],
      "DefaultRolloutPercentage": 50
    }
  }'
```

---

## Feature Flag Filters

### 1. Percentage Filter

**Purpose**: Enable feature for X% of users (gradual rollout).

```json
{
  "FeatureManagement": {
    "NewUI": {
      "EnabledFor": [
        {
          "Name": "Microsoft.Percentage",
          "Parameters": {
            "Value": 20  // 20% of users see new UI
          }
        }
      ]
    }
  }
}
```

**Progressive Rollout**:
```
Week 1: 10% → Monitor metrics
Week 2: 25% → Validate performance
Week 3: 50% → Expand exposure
Week 4: 100% → General availability
```

### 2. Targeting Filter

**Purpose**: Enable for specific users or groups.

```json
{
  "FeatureManagement": {
    "AdvancedReports": {
      "EnabledFor": [
        {
          "Name": "Microsoft.Targeting",
          "Parameters": {
            "Audience": {
              "Users": [  // Specific users
                "admin@company.com",
                "manager@company.com"
              ],
              "Groups": [  // User groups
                "power-users",
                "beta-testers"
              ],
              "DefaultRolloutPercentage": 0,  // Others: 0%
              "Exclusion": {
                "Users": ["excluded@company.com"]
              }
            }
          }
        }
      ]
    }
  }
}
```

### 3. Time Window Filter

**Purpose**: Enable during specific time periods.

```json
{
  "FeatureManagement": {
    "FlashSale": {
      "EnabledFor": [
        {
          "Name": "Microsoft.TimeWindow",
          "Parameters": {
            "Start": "2026-01-14T09:00:00Z",
            "End": "2026-01-14T17:00:00Z"
          }
        }
      ]
    }
  }
}
```

### 4. Custom Filters

**Purpose**: Implement business-specific enablement logic.

```csharp
// Define custom filter
public class PremiumUserFilter : IFeatureFilter
{
    public Task<bool> EvaluateAsync(FeatureFilterEvaluationContext context)
    {
        var user = _httpContextAccessor.HttpContext.User;
        var isPremium = user.HasClaim("subscription", "premium");
        return Task.FromResult(isPremium);
    }
}

// Register filter
services.AddFeatureManagement()
        .AddFeatureFilter<PremiumUserFilter>();

// Use in configuration
{
  "FeatureManagement": {
    "PremiumFeatures": {
      "EnabledFor": [
        {
          "Name": "PremiumUserFilter"
        }
      ]
    }
  }
}
```

---

## Feature Flag Repository

### Externalization Benefits

**Why Externalize**:
- ✅ Modify flag state without code changes
- ✅ Instant feature activation/deactivation
- ✅ Centralized management across services
- ✅ Audit trail of flag changes

**Azure App Configuration as Repository**:
```csharp
// Application reads flags from App Configuration
builder.Configuration.AddAzureAppConfiguration(options => {
    options.Connect(connectionString)
           .UseFeatureFlags(flags => {
               flags.CacheExpirationInterval = TimeSpan.FromSeconds(30);
               flags.Label = "production";
           });
});

services.AddFeatureManagement();

// Feature state managed in Azure Portal (no redeployment)
```

### Feature Flag Management UI

**Azure Portal Interface**:
```
App Configuration → Feature Manager
├── Create new feature flag
├── Enable/disable existing flags
├── Configure filters (percentage, targeting, time window)
├── Monitor flag evaluation metrics
└── Audit flag change history
```

**Operations**:
```bash
# List all feature flags
az appconfig feature list --name myapp-config

# Enable feature flag
az appconfig feature enable --name myapp-config --feature BetaCheckout

# Disable feature flag
az appconfig feature disable --name myapp-config --feature BetaCheckout

# Delete feature flag
az appconfig feature delete --name myapp-config --feature BetaCheckout --yes
```

---

## Feature Flag Types

### 1. Release Flags (Trunk-Based Development)
**Purpose**: Hide incomplete features in production until ready.

```csharp
// Feature deployed but hidden
if (await _featureManager.IsEnabledAsync("NewSearchEngine"))
{
    return await _newSearchService.SearchAsync(query);  // New (hidden)
}
else
{
    return await _legacySearchService.SearchAsync(query);  // Current
}

// Enable when ready (instant, no deployment)
```

**Benefits**:
- ✅ Continuous integration (merge daily)
- ✅ Reduce merge conflicts (no long-lived branches)
- ✅ Deploy incomplete code safely (behind flag)

### 2. Ops Flags (Operational Control)
**Purpose**: Circuit breakers, maintenance mode, performance tuning.

```csharp
// Circuit breaker flag
if (await _featureManager.IsEnabledAsync("UseExternalCache"))
{
    return await _cacheService.GetAsync(key);
}
else
{
    return await _databaseService.GetAsync(key);  // Fallback if cache down
}
```

**Use Cases**:
- Circuit breakers (disable failing dependency)
- Maintenance mode (read-only operations)
- Performance degradation (disable expensive features under load)

### 3. Experiment Flags (A/B Testing)
**Purpose**: Compare feature variants for data-driven decisions.

```csharp
// A/B test checkout flows
if (await _featureManager.IsEnabledAsync("NewCheckoutFlow"))
{
    // Variant B: New single-page checkout
    _telemetry.TrackEvent("Checkout_Variant_B");
    return View("SinglePageCheckout");
}
else
{
    // Variant A: Legacy 3-step checkout
    _telemetry.TrackEvent("Checkout_Variant_A");
    return View("MultiStepCheckout");
}

// Analyze conversion rates: Variant B wins → Enable for 100%
```

### 4. Permission Flags (Entitlement)
**Purpose**: Control feature access based on user permissions.

```csharp
// Premium features for paid users
if (await _featureManager.IsEnabledAsync("AdvancedAnalytics"))
{
    return await _analyticsService.GetAdvancedReportsAsync();
}
else
{
    return View("UpgradeRequired");  // Free tier limitation
}
```

---

## Complete Implementation Example

### ASP.NET Core Application

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Configure App Configuration with feature flags
builder.Configuration.AddAzureAppConfiguration(options => {
    options.Connect(connectionString)
           .UseFeatureFlags(flags => {
               flags.CacheExpirationInterval = TimeSpan.FromSeconds(30);
           });
});

builder.Services.AddAzureAppConfiguration();
builder.Services.AddFeatureManagement();

var app = builder.Build();
app.UseAzureAppConfiguration();  // Enable refresh middleware

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
        if (await _featureManager.IsEnabledAsync("BetaCheckout"))
        {
            // New checkout logic
            var result = await _newCheckoutService.ProcessAsync(order);
            return Ok(new { version = "v2", result });
        }
        else
        {
            // Legacy checkout logic
            var result = await _legacyCheckoutService.ProcessAsync(order);
            return Ok(new { version = "v1", result });
        }
    }
}
```

### View Conditional Rendering

```html
@inject IFeatureManager FeatureManager

@if (await FeatureManager.IsEnabledAsync("NewUI"))
{
    <partial name="_NewNavigation" />
}
else
{
    <partial name="_LegacyNavigation" />
}
```

### Action Filter Attribute

```csharp
// Restrict entire controller action based on feature flag
[FeatureGate("AdvancedReports")]
[HttpGet("advanced")]
public IActionResult GetAdvancedReports()
{
    return Ok(_reportService.GetAdvancedReports());
    // Returns 404 if feature disabled
}
```

---

## Best Practices

### 1. Short-Lived Flags
```csharp
// ✅ Good: Remove flag after feature fully rolled out
if (await _featureManager.IsEnabledAsync("NewCheckout"))
{
    return await NewCheckoutAsync();
}
else
{
    return await LegacyCheckoutAsync();
}

// After 100% rollout (2-4 weeks): Remove flag and else branch
return await NewCheckoutAsync();  // New feature is now standard
```

### 2. Flag Naming Conventions
```bash
# ✅ Good: Descriptive, purpose-clear names
BetaCheckout
AdvancedReportingV2
ExperimentalSearchEngine

# ❌ Poor: Vague, unclear purpose
Feature1
NewStuff
Test
```

### 3. Default State (Fail-Safe)
```csharp
// ✅ Good: Graceful degradation if flag unavailable
try
{
    if (await _featureManager.IsEnabledAsync("ExternalPayment"))
    {
        return await _externalPaymentService.ProcessAsync(order);
    }
}
catch
{
    // Fallback to safe default if flag check fails
}
return await _internalPaymentService.ProcessAsync(order);
```

### 4. Flag Cleanup
```bash
# Remove obsolete flags (technical debt)
# After feature 100% rolled out and stable for 30 days:
az appconfig feature delete --name myapp-config --feature ObsoleteFeature --yes

# Remove flag checks from code
git grep "ObsoleteFeature"  # Find all references
# Remove if/else branches, keep winning variant only
```

---

## Real-World Example: E-Commerce Feature Rollout

### Scenario
New single-page checkout flow to replace legacy 3-step checkout.

### Implementation

**Week 1: Development**
```bash
# Deploy code with feature hidden
az appconfig feature set --name myapp-config --feature SinglePageCheckout --label production
az appconfig feature disable --name myapp-config --feature SinglePageCheckout

# Feature code deployed but disabled
```

**Week 2: Internal Testing (10% rollout)**
```bash
az appconfig feature filter add \
  --name myapp-config \
  --feature SinglePageCheckout \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":10}'

# Monitor metrics: Conversion rate, errors, performance
```

**Week 3: Beta Users (50% rollout)**
```bash
az appconfig feature filter update \
  --name myapp-config \
  --feature SinglePageCheckout \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":50}'

# Result: +3% conversion improvement, stable
```

**Week 4: General Availability (100%)**
```bash
az appconfig feature filter update \
  --name myapp-config \
  --feature SinglePageCheckout \
  --filter-name Microsoft.Percentage \
  --filter-parameters '{"Value":100}'

# Result: Successful rollout, +$2M annual revenue
```

**Week 8: Flag Cleanup**
```bash
# Remove flag and legacy code
az appconfig feature delete --name myapp-config --feature SinglePageCheckout --yes

# Simplify code (remove if/else, keep winner)
// Before
if (await _featureManager.IsEnabledAsync("SinglePageCheckout"))
    return View("SinglePageCheckout");
else
    return View("LegacyCheckout");

// After
return View("SinglePageCheckout");  // New standard
```

---

## Key Takeaways

### Feature Management Benefits
✅ **Decouple deployment from release**: Deploy hidden features, enable instantly  
✅ **Progressive rollouts**: Gradual exposure reduces risk (10% → 50% → 100%)  
✅ **Instant rollback**: Disable feature without redeployment  
✅ **A/B testing**: Data-driven decisions through experimentation  
✅ **Trunk-based development**: Merge daily, reduce long-lived branches  

### Feature Flag Components
✅ **Feature flag**: Binary state variable controlling code execution  
✅ **Feature manager**: Lifecycle management and caching  
✅ **Filters**: Percentage, targeting, time window, custom logic  
✅ **Repository**: Centralized management (Azure App Configuration)  

### Implementation Patterns
✅ **Release flags**: Hide incomplete features until ready  
✅ **Ops flags**: Circuit breakers and operational control  
✅ **Experiment flags**: A/B testing and optimization  
✅ **Permission flags**: Entitlement-based feature access  

---

**Next**: Learn about integrating Azure Key Vault with Azure Pipelines for secure secret management →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/8-examine-app-configuration-feature-management)
