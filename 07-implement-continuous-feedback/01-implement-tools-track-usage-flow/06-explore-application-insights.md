# Explore Application Insights

## Key Concepts
- **Application Insights**: Azure's APM (Application Performance Monitoring) solution
- **SDK-based**: Install in application code for deep telemetry
- **Platform-agnostic**: Works with .NET, Java, Node.js, Python, anywhere
- **<1% overhead**: Minimal performance impact with async collection
- **Auto-instrumentation**: Automatically captures requests, dependencies, exceptions

## How Application Insights Works
```
Application → SDK → Batched Telemetry → Azure Portal
                           ↓
                  Optional: Sampling (adaptive/fixed-rate)
                           ↓
                  Application Insights Resource
                           ↓
                  Query with KQL, Visualize, Alert
```

## What Gets Monitored
| Category | Metrics | Use Case |
|----------|---------|----------|
| **Performance** | Request rate, response time, failure rate | Identify slow endpoints |
| **Dependencies** | Database, API, queue response times | Find bottlenecks |
| **Exceptions** | Server & client-side errors with stack traces | Debug production issues |
| **Page Views** | Client-side performance, AJAX calls | Optimize user experience |
| **Custom Events** | Business metrics (orders, signups) | Track business outcomes |

## Instrumentation Options
| Method | Approach | Best For |
|--------|----------|----------|
| **SDK Integration** | Add NuGet/npm package | Full control, custom telemetry |
| **Status Monitor** | Runtime attachment (no code) | Legacy apps, can't modify code |
| **Auto-instrumentation** | Azure App Service integration | Quick start, zero config |
| **Client-side JS** | Add snippet to web pages | Browser performance, SPAs |

### SDK Example (.NET)
```csharp
// Install: Microsoft.ApplicationInsights.AspNetCore

// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddApplicationInsightsTelemetry();
}

// Custom telemetry
public class OrderController : Controller
{
    private readonly TelemetryClient _telemetry;
    
    public OrderController(TelemetryClient telemetry)
    {
        _telemetry = telemetry;
    }
    
    [HttpPost]
    public IActionResult PlaceOrder(Order order)
    {
        // Track custom business event
        _telemetry.TrackEvent("OrderPlaced",
            properties: new Dictionary<string, string> {
                {"Category", order.Category},
                {"Region", order.Region}
            },
            metrics: new Dictionary<string, double> {
                {"OrderValue", order.TotalAmount}
            });
            
        return Ok();
    }
}
```

## Telemetry Types
```csharp
// Request tracking (automatic)
// Captures HTTP requests, response times, status codes

// Dependency tracking (automatic)
telemetryClient.TrackDependency("SQL", "GetCustomer", startTime, duration, success);

// Exception tracking
telemetryClient.TrackException(exception);

// Custom events
telemetryClient.TrackEvent("VideoPlayed", properties, metrics);

// Custom metrics
telemetryClient.TrackMetric("QueueLength", queueSize);

// Page views (client-side)
appInsights.trackPageView({name: "ProductPage"});

// Traces (logging integration)
telemetryClient.TrackTrace("Processing started", SeverityLevel.Information);
```

## Application Insights Features

### 1. Smart Detection (AI-powered)
- 🤖 Auto-detects abnormal failure rates
- 📈 Identifies performance degradations
- 💾 Detects memory leaks
- 🔗 Spots dependency anomalies
- ⚡ No configuration required

### 2. Application Map
```
[Web Frontend] → [API Gateway] → [Order Service] → [SQL Database]
     ↓                               ↓                    ↓
[Redis Cache]                   [Queue Service]     [Cosmos DB]

Color-coded health, click for details, auto-discovered topology
```

### 3. Live Metrics Stream
- 📊 1-second refresh rate
- 🚀 Real-time request/exception stream
- 💻 Server count, CPU, memory
- ✅ Deployment validation
- 🔍 Filter by server/failure type

### 4. Profiler
```
Method Call Stack:
├── OrderController.PlaceOrder (45ms)
    ├── ValidateOrder (2ms)
    ├── Database.SaveOrder (38ms) ← BOTTLENECK
    └── SendConfirmation (5ms)
```
**Use case**: Identify code-level performance bottlenecks

### 5. Snapshot Debugger
- 📸 Captures memory snapshots on exceptions
- 🔍 View local variables, parameters, call stack
- 🛡️ Privacy controls (exclude sensitive data)
- 🎯 Production debugging without symbols

## User Analytics Features
| Feature | Purpose | Example |
|---------|---------|---------|
| **User Flows** | Visualize navigation paths | Home → Product → Cart → ? (drop-off) |
| **Funnels** | Conversion tracking | 100% → 60% → 40% → 10% completed |
| **Cohorts** | User segmentation | Mobile vs. desktop users |
| **Retention** | Return rate | DAU/MAU ratio (stickiness) |
| **Impact Analysis** | Performance ↔ Business | Page load time vs. conversion rate |

### Impact Analysis Example
| Load Time | Conversion Rate | Users |
|-----------|----------------|--------|
| <1s | 45% | 15K |
| 1-2s | 38% | 22K |
| 2-3s | 28% | 18K |
| >3s | 15% | 12K |

**Insight**: Reducing load time from 3s → 1s could increase conversion 3x!

## Visualization Options
| Tool | Purpose | Key Feature |
|------|---------|-------------|
| **Metrics Explorer** | Time-series charts | Filtering, splitting, percentiles |
| **Logs (KQL)** | Ad-hoc analysis | Complex queries, joins |
| **Dashboards** | Shared views | Pin tiles, multi-resource |
| **Workbooks** | Interactive reports | Parameters, conditional content |
| **Power BI** | Business intelligence | Combine with business data |

## Sampling Strategies
```csharp
// Fixed-rate sampling (20%)
builder.Services.AddApplicationInsightsTelemetry(options => {
    options.SamplingPercentage = 20;
});

// Adaptive sampling (automatic)
// Adjusts rate based on traffic volume
// Preserves related telemetry together
```

**Trade-offs**:
- ✅ Reduces costs (less data ingestion)
- ✅ Lowers impact on application
- ⚠️ May miss rare events
- ✅ Maintains statistical accuracy

## Alert Configuration
```json
{
  "condition": {
    "allOf": [
      {
        "metricName": "requests/failed",
        "operator": "GreaterThan",
        "threshold": 10,
        "timeAggregation": "Count",
        "dimensions": [
          { "name": "cloud/roleName", "value": "api-service" }
        ]
      }
    ]
  },
  "actions": [
    {
      "actionGroupId": "/subscriptions/.../actionGroups/oncall-team"
    }
  ]
}
```

## Continuous Export Options
- **Azure Storage**: Long-term archival, compliance
- **Event Hubs**: Real-time streaming to external systems
- **Log Analytics**: Combine with other Azure Monitor data
- **Power BI**: Scheduled refresh for reports

## Critical Notes
- ⚠️ **Sampling preserves context**: Related telemetry sampled together
- 💡 **Correlation IDs**: Track requests across distributed systems
- 🎯 **Operation ID**: Groups all telemetry for a single user request
- 📊 **Separate instances per environment**: Dev/Test/Prod isolation
- 🔒 **Instrumentation key vs. connection string**: Use connection string (newer)

## Quick Setup Commands
```bash
# Create Application Insights
az monitor app-insights component create \
    --app "app-insights-prod" \
    --location "eastus" \
    --resource-group "rg-monitoring-prod" \
    --application-type web

# Get connection string
az monitor app-insights component show \
    --app "app-insights-prod" \
    --resource-group "rg-monitoring-prod" \
    --query connectionString -o tsv
```

## Integration Examples
```javascript
// Node.js
const appInsights = require('applicationinsights');
appInsights.setup("<connection-string>").start();

// Python
from applicationinsights import TelemetryClient
tc = TelemetryClient('<connection-string>')
tc.track_event('UserLogin')

// Java (Spring Boot)
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>applicationinsights-spring-boot-starter</artifactId>
</dependency>
```

## Performance Impact
- **Typical overhead**: <1% CPU
- **Network**: ~50-100 KB/s per instance
- **Non-blocking**: Telemetry sent asynchronously
- **Local buffering**: No data loss during outages
- **Adaptive throttling**: Reduces load automatically

## Where to View Telemetry
✅ Azure Portal: Full-featured interface
✅ Visual Studio: CodeLens integration
✅ VS Code: Extension available
✅ Azure Mobile App: On-the-go monitoring
✅ REST API: Programmatic access
✅ Power BI: Business reports

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-tools-track-usage-flow/6-explore-application-insights)
