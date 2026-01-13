# Set up and run availability tests

⏱️ **Duration**: ~4 minutes | 📚 **Type**: Conceptual

## Overview

Availability tests proactively monitor application health and responsiveness from multiple geographic locations. Using Azure Application Insights, you can configure automated URL ping tests and multi-step web tests to detect availability issues before users report them.

## Learning Objectives

After completing this unit, you'll be able to:
- ✅ Configure Application Insights for availability monitoring
- ✅ Create URL ping tests for endpoint health checks
- ✅ Set up multi-step web tests for critical workflows
- ✅ Configure alerts for availability degradation
- ✅ Integrate availability tests with Azure DevOps pipelines

---

## Azure Application Insights Overview

### What is Application Insights?

**Azure Application Insights** is an Application Performance Management (APM) service that provides:
- 📊 Real-time performance monitoring
- 🔍 Distributed tracing
- 📈 Usage analytics
- 🚨 Availability testing
- 💡 AI-powered anomaly detection

### Availability Testing Capabilities

| Feature | Description | Use Case |
|---------|-------------|----------|
| **URL Ping Tests** | HTTP/HTTPS endpoint monitoring | API health, homepage availability |
| **Multi-step Web Tests** | Sequential user workflow validation | E-commerce checkout, login flows |
| **Custom Track Availability** | Code-based availability metrics | Complex scenarios, internal services |
| **Geographic Distribution** | Test from 5+ Azure regions | Global user experience validation |
| **Alert Integration** | Automatic notifications on failures | Proactive incident response |

---

## Setting Up Application Insights

### Step 1: Create Application Insights Resource

#### Using Azure Portal

**Navigation**: Azure Portal → Create a resource → Application Insights

**Configuration**:
```
Resource Details:
- Subscription: [Your subscription]
- Resource Group: rg-devops-monitoring
- Name: appi-myapp-prod
- Region: East US (or nearest to users)
- Resource Mode: Workspace-based (recommended)
- Log Analytics Workspace: [Create new or select existing]

Pricing:
- Pricing Tier: Pay-as-you-go
- Daily cap: 100 GB/day (optional)
```

#### Using Azure CLI

```bash
# Create resource group
az group create --name rg-devops-monitoring --location eastus

# Create Log Analytics workspace
az monitor log-analytics workspace create \
    --resource-group rg-devops-monitoring \
    --workspace-name law-devops-monitoring \
    --location eastus

# Create Application Insights
WORKSPACE_ID=$(az monitor log-analytics workspace show \
    --resource-group rg-devops-monitoring \
    --workspace-name law-devops-monitoring \
    --query id -o tsv)

az monitor app-insights component create \
    --app appi-myapp-prod \
    --location eastus \
    --resource-group rg-devops-monitoring \
    --workspace $WORKSPACE_ID

# Get instrumentation key
az monitor app-insights component show \
    --app appi-myapp-prod \
    --resource-group rg-devops-monitoring \
    --query instrumentationKey -o tsv
```

#### Using Bicep/ARM Template

**File**: `appinsights.bicep`

```bicep
param appName string
param location string = resourceGroup().location
param workspaceId string

resource appInsights 'Microsoft.Insights/components@2020-02-02' = {
  name: 'appi-${appName}'
  location: location
  kind: 'web'
  properties: {
    Application_Type: 'web'
    WorkspaceResourceId: workspaceId
    publicNetworkAccessForIngestion: 'Enabled'
    publicNetworkAccessForQuery: 'Enabled'
  }
}

output instrumentationKey string = appInsights.properties.InstrumentationKey
output connectionString string = appInsights.properties.ConnectionString
output appInsightsId string = appInsights.id
```

**Deploy**:
```bash
az deployment group create \
    --resource-group rg-devops-monitoring \
    --template-file appinsights.bicep \
    --parameters appName=myapp-prod workspaceId=$WORKSPACE_ID
```

### Step 2: Instrument Your Application

#### ASP.NET Core

**NuGet Package**:
```bash
dotnet add package Microsoft.ApplicationInsights.AspNetCore
```

**Configuration** (`Program.cs`):
```csharp
using Microsoft.ApplicationInsights.AspNetCore.Extensions;

var builder = WebApplication.CreateBuilder(args);

// Add Application Insights telemetry
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["ApplicationInsights:ConnectionString"];
    options.EnableAdaptiveSampling = true;
    options.EnableHeartbeat = true;
    options.EnableDependencyTrackingTelemetryModule = true;
});

var app = builder.Build();
app.Run();
```

**Configuration** (`appsettings.json`):
```json
{
  "ApplicationInsights": {
    "ConnectionString": "InstrumentationKey=xxxxx;IngestionEndpoint=https://..."
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.ApplicationInsights": "Warning"
    }
  }
}
```

#### Node.js

**NPM Package**:
```bash
npm install applicationinsights
```

**Configuration** (`app.js`):
```javascript
const appInsights = require('applicationinsights');

// Setup Application Insights
appInsights.setup(process.env.APPLICATIONINSIGHTS_CONNECTION_STRING)
    .setAutoDependencyCorrelation(true)
    .setAutoCollectRequests(true)
    .setAutoCollectPerformance(true, true)
    .setAutoCollectExceptions(true)
    .setAutoCollectDependencies(true)
    .setAutoCollectConsole(true)
    .setUseDiskRetryCaching(true)
    .start();

const client = appInsights.defaultClient;
client.trackEvent({ name: 'Application Started' });
```

---

## URL Ping Tests (Standard Availability Tests)

### Creating a URL Ping Test

#### Using Azure Portal

**Navigation**: Application Insights → Availability → Add Standard test

**Configuration**:
```
Test Details:
- Test name: Homepage Availability
- URL: https://www.example.com
- Test frequency: 5 minutes
- Test locations: 5 locations (select geographic diversity)
  ✓ East US
  ✓ West Europe
  ✓ Southeast Asia
  ✓ Brazil South
  ✓ Australia East

Success Criteria:
- Test timeout: 30 seconds
- Enable retries: Yes
- HTTP Status Code: 200
- Content match: (optional) "Welcome" substring

Alerts:
- Alert threshold: >= 3 locations failed
- Alert severity: Sev 2
- Action group: [Select or create action group]
```

#### Using Azure CLI

```bash
# Get Application Insights ID
APPI_ID=$(az monitor app-insights component show \
    --app appi-myapp-prod \
    --resource-group rg-devops-monitoring \
    --query id -o tsv)

# Create availability test
az monitor app-insights web-test create \
    --resource-group rg-devops-monitoring \
    --name "avt-homepage" \
    --location eastus \
    --kind ping \
    --defined-web-test-name "Homepage Availability" \
    --web-test-kind ping \
    --frequency 300 \
    --timeout 30 \
    --enabled true \
    --locations Id=emea-nl-ams-azr,Id=us-va-ash-azr,Id=apac-sg-sin-azr \
    --synthetic-monitor-id "${APPI_ID}/availabilitytests/avt-homepage" \
    --web-test-properties '{"Request":{"HttpVerb":"GET","Url":"https://www.example.com"},"ValidationRules":{"ExpectedHttpStatusCode":200}}'
```

#### Using Bicep

**File**: `availability-test.bicep`

```bicep
param appInsightsName string
param testName string
param testUrl string
param location string = 'eastus'

resource appInsights 'Microsoft.Insights/components@2020-02-02' existing = {
  name: appInsightsName
}

resource availabilityTest 'Microsoft.Insights/webtests@2022-06-15' = {
  name: 'avt-${testName}'
  location: location
  kind: 'ping'
  properties: {
    Name: '${testName} Availability Test'
    SyntheticMonitorId: 'avt-${testName}'
    Enabled: true
    Frequency: 300  // 5 minutes
    Timeout: 30
    Kind: 'ping'
    RetryEnabled: true
    Locations: [
      { Id: 'us-va-ash-azr' }      // East US
      { Id: 'emea-nl-ams-azr' }    // West Europe
      { Id: 'apac-sg-sin-azr' }    // Southeast Asia
      { Id: 'latam-br-gru-edge' }  // Brazil South
      { Id: 'apac-hk-hkn-azr' }    // East Asia
    ]
    Configuration: {
      WebTest: '<WebTest Name="${testName}" Enabled="True" Timeout="30"><Items><Request Method="GET" Url="${testUrl}" Version="1.1" /></Items></WebTest>'
    }
  }
  tags: {
    'hidden-link:${appInsights.id}': 'Resource'
  }
}

// Create alert rule for availability test
resource availabilityAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'alert-${testName}-availability'
  location: 'global'
  properties: {
    description: 'Alert when ${testName} availability drops below threshold'
    severity: 2
    enabled: true
    scopes: [
      appInsights.id
      availabilityTest.id
    ]
    evaluationFrequency: 'PT1M'
    windowSize: 'PT5M'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.WebtestLocationAvailabilityCriteria'
      webTestId: availabilityTest.id
      componentId: appInsights.id
      failedLocationCount: 3
    }
  }
}
```

---

## Multi-Step Web Tests

### What are Multi-Step Web Tests?

**Definition**: Sequential HTTP requests that simulate user workflows (e.g., login → browse → checkout)

**Use Cases**:
- E-commerce checkout flows
- User authentication scenarios
- Multi-page form submissions
- Complex API call sequences

**Limitations**:
- ⚠️ Only support HTTP GET requests (no POST/PUT)
- ⚠️ Deprecated in favor of custom availability tests via Azure Functions or codeless monitoring

### Alternative: Custom Availability Tests (TrackAvailability)

For complex workflows, use code-based availability tracking:

#### Example: Azure Function with Custom Availability Test

**File**: `AvailabilityTestFunction.cs`

```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;
using System;
using System.Diagnostics;
using System.Net.Http;
using System.Threading.Tasks;

public class AvailabilityTestFunction
{
    private readonly TelemetryClient _telemetryClient;
    private static readonly HttpClient _httpClient = new HttpClient();

    public AvailabilityTestFunction(TelemetryConfiguration config)
    {
        _telemetryClient = new TelemetryClient(config);
    }

    [FunctionName("CheckoutFlowAvailability")]
    public async Task Run([TimerTrigger("0 */5 * * * *")] TimerInfo timer, ILogger log)
    {
        var testName = "Checkout Flow";
        var testLocation = Environment.GetEnvironmentVariable("REGION_NAME") ?? "Local";
        var stopwatch = Stopwatch.StartNew();
        bool success = false;
        string message = string.Empty;

        try
        {
            // Step 1: Get homepage
            var homeResponse = await _httpClient.GetAsync("https://www.example.com");
            homeResponse.EnsureSuccessStatusCode();

            // Step 2: Browse products
            var productsResponse = await _httpClient.GetAsync("https://www.example.com/api/products");
            productsResponse.EnsureSuccessStatusCode();

            // Step 3: Add to cart
            var cartContent = new StringContent("{\"productId\":123,\"quantity\":1}", System.Text.Encoding.UTF8, "application/json");
            var cartResponse = await _httpClient.PostAsync("https://www.example.com/api/cart", cartContent);
            cartResponse.EnsureSuccessStatusCode();

            // Step 4: View cart
            var viewCartResponse = await _httpClient.GetAsync("https://www.example.com/api/cart");
            viewCartResponse.EnsureSuccessStatusCode();

            success = true;
            message = "Checkout flow completed successfully";
        }
        catch (Exception ex)
        {
            message = $"Checkout flow failed: {ex.Message}";
            log.LogError(ex, message);
        }
        finally
        {
            stopwatch.Stop();

            // Track availability telemetry
            var availability = new AvailabilityTelemetry
            {
                Name = testName,
                RunLocation = testLocation,
                Success = success,
                Duration = stopwatch.Elapsed,
                Message = message,
                Timestamp = DateTimeOffset.UtcNow
            };

            _telemetryClient.TrackAvailability(availability);
            _telemetryClient.Flush();
        }
    }
}
```

**Deploy Azure Function**:
```yaml
# Azure Pipeline to deploy availability test function
- stage: DeployAvailabilityTests
  jobs:
  - job: DeployFunction
    steps:
    - task: AzureFunctionApp@1
      displayName: 'Deploy Availability Test Function'
      inputs:
        azureSubscription: 'AzureServiceConnection'
        appType: 'functionApp'
        appName: 'func-availability-tests-prod'
        package: '$(Build.ArtifactStagingDirectory)/**/*.zip'
        appSettings: |
          -APPLICATIONINSIGHTS_CONNECTION_STRING "$(AppInsightsConnectionString)"
          -REGION_NAME "East US"
```

---

## Health Endpoints

### Implementing Health Checks

#### ASP.NET Core Health Checks

**NuGet Package**:
```bash
dotnet add package Microsoft.Extensions.Diagnostics.HealthChecks
dotnet add package AspNetCore.HealthChecks.SqlServer
dotnet add package AspNetCore.HealthChecks.Redis
```

**Configuration** (`Program.cs`):
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add health checks
builder.Services.AddHealthChecks()
    .AddSqlServer(
        connectionString: builder.Configuration.GetConnectionString("DefaultConnection"),
        name: "sql-server",
        tags: new[] { "db", "sql" })
    .AddRedis(
        redisConnectionString: builder.Configuration.GetConnectionString("Redis"),
        name: "redis-cache",
        tags: new[] { "cache", "redis" })
    .AddCheck<CustomHealthCheck>("custom-check", tags: new[] { "custom" });

var app = builder.Build();

// Map health check endpoints
app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "application/json";
        var result = System.Text.Json.JsonSerializer.Serialize(new
        {
            status = report.Status.ToString(),
            checks = report.Entries.Select(e => new
            {
                name = e.Key,
                status = e.Value.Status.ToString(),
                description = e.Value.Description,
                duration = e.Value.Duration.TotalMilliseconds
            }),
            totalDuration = report.TotalDuration.TotalMilliseconds
        });
        await context.Response.WriteAsync(result);
    }
});

// Liveness probe (simple)
app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = _ => false  // No checks, just returns 200 if app is running
});

// Readiness probe (includes dependencies)
app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("db") || check.Tags.Contains("cache")
});

app.Run();
```

**Custom Health Check Example**:
```csharp
public class CustomHealthCheck : IHealthCheck
{
    private readonly IHttpClientFactory _httpClientFactory;

    public CustomHealthCheck(IHttpClientFactory httpClientFactory)
    {
        _httpClientFactory = httpClientFactory;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        try
        {
            var client = _httpClientFactory.CreateClient();
            var response = await client.GetAsync("https://api.example.com/health", cancellationToken);

            if (response.IsSuccessStatusCode)
            {
                return HealthCheckResult.Healthy("External API is reachable");
            }

            return HealthCheckResult.Degraded($"External API returned {response.StatusCode}");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("External API is unreachable", ex);
        }
    }
}
```

**Health Check Response Example**:
```json
{
  "status": "Healthy",
  "checks": [
    {
      "name": "sql-server",
      "status": "Healthy",
      "description": "SQL Server is responsive",
      "duration": 45.2
    },
    {
      "name": "redis-cache",
      "status": "Healthy",
      "description": "Redis cache is available",
      "duration": 12.8
    },
    {
      "name": "custom-check",
      "status": "Healthy",
      "description": "External API is reachable",
      "duration": 234.5
    }
  ],
  "totalDuration": 292.5
}
```

### Configure Availability Test for Health Endpoint

```bicep
resource healthCheckTest 'Microsoft.Insights/webtests@2022-06-15' = {
  name: 'avt-health-check'
  location: location
  kind: 'ping'
  properties: {
    Name: 'Health Endpoint Availability'
    SyntheticMonitorId: 'avt-health'
    Enabled: true
    Frequency: 300
    Timeout: 30
    Kind: 'ping'
    RetryEnabled: true
    Locations: [
      { Id: 'us-va-ash-azr' }
      { Id: 'emea-nl-ams-azr' }
      { Id: 'apac-sg-sin-azr' }
    ]
    Configuration: {
      WebTest: '<WebTest Name="Health Check" Enabled="True" Timeout="30"><Items><Request Method="GET" Url="https://api.example.com/health/ready" Version="1.1" /><ValidationRules><ValidationRule Classname="Microsoft.VisualStudio.TestTools.WebTesting.Rules.ValidationRuleResponseCode" DisplayName="Response Code" Level="High"><RuleParameters><RuleParameter Name="ExpectedResponseCode" Value="200" /></RuleParameters></ValidationRule></ValidationRules></Items></WebTest>'
    }
  }
  tags: {
    'hidden-link:${appInsights.id}': 'Resource'
  }
}
```

---

## Configuring Alerts

### Alert Rule Configuration

#### Using Azure Portal

**Navigation**: Application Insights → Alerts → Create alert rule

**Configuration**:
```
Scope:
- Resource: appi-myapp-prod

Condition:
- Signal: Availability (Web test result)
- Threshold: >= 3 locations failed
- Aggregation: Average
- Evaluation frequency: 1 minute
- Lookback period: 5 minutes

Actions:
- Action group: ag-devops-alerts
  - Email: devops-team@example.com
  - SMS: +1-555-0123
  - Webhook: https://hooks.slack.com/services/...
  - Azure Function: func-alert-handler

Alert Details:
- Severity: 2 (Warning)
- Alert rule name: Homepage Availability Degraded
- Description: Homepage availability has dropped below acceptable threshold
```

#### Using Bicep

```bicep
resource actionGroup 'Microsoft.Insights/actionGroups@2023-01-01' = {
  name: 'ag-devops-alerts'
  location: 'global'
  properties: {
    groupShortName: 'DevOpsTeam'
    enabled: true
    emailReceivers: [
      {
        name: 'DevOps Email'
        emailAddress: 'devops-team@example.com'
        useCommonAlertSchema: true
      }
    ]
    smsReceivers: [
      {
        name: 'On-Call SMS'
        countryCode: '1'
        phoneNumber: '5550123'
      }
    ]
    webhookReceivers: [
      {
        name: 'Slack Webhook'
        serviceUri: 'https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX'
        useCommonAlertSchema: true
      }
    ]
  }
}

resource availabilityAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'alert-homepage-availability'
  location: 'global'
  properties: {
    description: 'Homepage availability degraded'
    severity: 2
    enabled: true
    scopes: [
      appInsights.id
      availabilityTest.id
    ]
    evaluationFrequency: 'PT1M'
    windowSize: 'PT5M'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.WebtestLocationAvailabilityCriteria'
      webTestId: availabilityTest.id
      componentId: appInsights.id
      failedLocationCount: 3
    }
    actions: [
      {
        actionGroupId: actionGroup.id
      }
    ]
  }
}
```

---

## Pipeline Integration

### Deploy Availability Tests with Infrastructure

```yaml
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: DeployInfrastructure
  jobs:
  - job: DeployMonitoring
    steps:
    - task: AzureCLI@2
      displayName: 'Deploy Application Insights and Availability Tests'
      inputs:
        azureSubscription: 'AzureServiceConnection'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          # Deploy Application Insights
          az deployment group create \
            --resource-group rg-devops-monitoring \
            --template-file infrastructure/appinsights.bicep \
            --parameters appName=myapp-prod
          
          # Deploy availability tests
          az deployment group create \
            --resource-group rg-devops-monitoring \
            --template-file infrastructure/availability-tests.bicep \
            --parameters @availability-tests.parameters.json

- stage: ValidateAvailability
  dependsOn: DeployInfrastructure
  jobs:
  - job: CheckEndpoints
    steps:
    - script: |
        echo "Waiting for deployment to stabilize..."
        sleep 60
        
        # Check health endpoint
        RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health/ready)
        
        if [ "$RESPONSE" -eq 200 ]; then
          echo "✅ Health endpoint is healthy"
        else
          echo "❌ Health endpoint returned $RESPONSE"
          exit 1
        fi
      displayName: 'Validate health endpoints'
```

---

## Quick Reference

| Test Type | Frequency | Timeout | Locations | Use Case |
|-----------|-----------|---------|-----------|----------|
| **URL Ping** | 5-15 min | 30s | 5+ regions | Simple endpoint checks |
| **Health Endpoint** | 1-5 min | 10s | 3+ regions | Liveness/readiness probes |
| **Custom (Function)** | 5-60 min | 5 min | Configurable | Complex workflows |

**Alert Thresholds**:
- ⚠️ **Warning**: 2+ locations failed (Sev 2)
- 🚨 **Critical**: 3+ locations failed (Sev 1)
- 🔴 **Emergency**: All locations failed (Sev 0)

---

## Key Takeaways

- 📊 **Application Insights** provides built-in availability monitoring from multiple regions
- 🔍 **URL ping tests** are ideal for simple endpoint health checks
- 🛠️ **Health endpoints** (e.g., `/health/ready`) enable deep dependency validation
- 🚨 **Alerts** ensure proactive notification before users report issues
- 🌍 **Geographic testing** validates global user experience
- 🤖 **Custom availability tests** (Azure Functions) support complex scenarios
- 📈 **Integration** with Azure DevOps enables continuous monitoring

---

## Next Steps

✅ **Completed**: Availability test configuration

**Continue to**: Unit 7 - Explore Azure Load Testing

---

## Additional Resources

- [Monitor availability with URL ping tests](https://learn.microsoft.com/en-us/azure/azure-monitor/app/monitor-web-app-availability)
- [Application Insights availability tests](https://learn.microsoft.com/en-us/azure/azure-monitor/app/availability-overview)
- [ASP.NET Core Health Checks](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks)
- [TrackAvailability API](https://learn.microsoft.com/en-us/azure/azure-monitor/app/availability-azure-functions)

[↩️ Back to Module Overview](01-introduction.md) | [➡️ Next: Azure Load Testing](07-explore-azure-load-testing.md)
