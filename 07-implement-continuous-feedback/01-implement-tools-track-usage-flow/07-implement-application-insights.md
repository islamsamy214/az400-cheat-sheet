# Implement Application Insights

## Key Concepts
- **Three-phase workflow**: Monitor → Detect/Diagnose → Build/Measure/Learn
- **Implementation approaches**: Runtime instrumentation vs. SDK integration
- **Multi-platform support**: .NET, Java, Node.js, Python, mobile apps
- **Complete monitoring**: Server-side + client-side (JavaScript) + availability tests

## Implementation Approaches
| Approach | Method | Best For |
|----------|--------|----------|
| **Runtime Instrumentation** | No code changes, agent-based | Existing production apps, legacy systems |
| **SDK Integration** | Add NuGet/npm package | New apps, need custom telemetry |
| **Client-Side** | JavaScript snippet | Browser monitoring, SPAs |
| **Availability Tests** | Synthetic monitoring | External health checks |

### Runtime Instrumentation (Zero Code)
```bash
# Azure App Service - Enable via portal
# IIS on-premises - Install Status Monitor

# Azure VM
az vm extension set \
    --resource-group "rg-prod" \
    --vm-name "vm-web01" \
    --name "ApplicationInsightsAgent" \
    --publisher "Microsoft.Azure.Diagnostics"
```

### SDK Integration Example (ASP.NET Core)
```csharp
// Program.cs
builder.Services.AddApplicationInsightsTelemetry();

// appsettings.json
{
  "ApplicationInsights": {
    "ConnectionString": "InstrumentationKey=...;IngestionEndpoint=..."
  }
}

// Custom telemetry
public class OrderService
{
    private readonly TelemetryClient _telemetry;
    
    public OrderService(TelemetryClient telemetry)
    {
        _telemetry = telemetry;
    }
    
    public void ProcessOrder(Order order)
    {
        var sw = Stopwatch.StartNew();
        try
        {
            // Process order logic
            _telemetry.TrackEvent("OrderProcessed",
                properties: new Dictionary<string, string> {
                    {"OrderId", order.Id},
                    {"Region", order.Region}
                },
                metrics: new Dictionary<string, double> {
                    {"OrderValue", order.TotalAmount}
                });
        }
        catch (Exception ex)
        {
            _telemetry.TrackException(ex);
            throw;
        }
        finally
        {
            sw.Stop();
            _telemetry.TrackMetric("OrderProcessingTime", sw.ElapsedMilliseconds);
        }
    }
}
```

## Monitor Phase: Continuous Visibility

### Dashboard Metrics
| Category | Key Metrics | Purpose |
|----------|------------|---------|
| **Load** | Request rate, concurrent users | Capacity planning |
| **Responsiveness** | P50/P95/P99 response time | Performance SLAs |
| **Dependencies** | DB query time, API latency | Identify bottlenecks |
| **Errors** | Failed requests, exceptions | Reliability tracking |

### Live Metrics Stream
```yaml
Use During:
- Deployment validation (real-time health check)
- Load testing (observe performance under stress)
- Incident response (live troubleshooting)

Features:
- 1-second refresh rate
- Live request/exception stream
- Server health (CPU, memory)
- Filter by server or failure type
```

## Detect/Diagnose Phase: Rapid Issue Resolution

### Assess User Impact
```kusto
// Count affected users and failed requests
requests
| where timestamp > ago(1h)
| where success == false
| summarize 
    AffectedUsers = dcount(user_Id), 
    FailedRequests = count(),
    Regions = make_set(client_CountryOrRegion)
    by resultCode
| order by AffectedUsers desc
```

### Correlation Workflow
```
1. Failed Request in Search
        ↓
2. View Related Telemetry (operation_Id groups all)
        ↓
3. Examine Dependencies (which service failed?)
        ↓
4. Review Exceptions (stack trace)
        ↓
5. Check Logs (application traces)
```

### Diagnostic Tools
| Tool | Purpose | Use Case |
|------|---------|----------|
| **Profiler** | Code-level performance | Identify slow methods |
| **Snapshot Debugger** | Production debugging | View variables at exception time |
| **Stack Traces** | Exception analysis | Understand error flow |
| **Trace Logs** | Detailed investigation | Correlate logs with requests |

## Build/Measure/Learn Phase: Data-Driven Development

### Measurement Strategy
```yaml
Before Development:
- Define success metrics (adoption %, conversion %)
- Identify key user actions to track
- Determine segmentation criteria
- Establish baseline
- Set targets

Custom Telemetry:
- Track feature usage
- Measure performance
- Monitor business outcomes

After Deployment:
- Compare vs. baseline
- Segment analysis
- Funnel/cohort analysis
- Impact analysis
- Decide: expand/maintain/deprecate
```

### Decision Framework
```
If metrics exceed targets:
  → Invest in expanding feature
  → Apply learnings elsewhere

If metrics meet targets:
  → Maintain & monitor
  → Incremental improvements

If metrics underperform:
  → Analyze why (usability? discoverability?)
  → A/B test improvements
  → Consider deprecation
```

## Platform Support

### Server-Side SDKs
```bash
# .NET
dotnet add package Microsoft.ApplicationInsights.AspNetCore

# Java (Spring Boot)
# Add to pom.xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>applicationinsights-spring-boot-starter</artifactId>
</dependency>

# Node.js
npm install applicationinsights

# Python
pip install applicationinsights
```

### Client-Side JavaScript
```html
<!-- Add to <head> -->
<script type="text/javascript">
var appInsights=window.appInsights||function(config){
  // Snippet code...
}({
  connectionString: 'InstrumentationKey=...'
});
appInsights.trackPageView();
</script>
```

### Availability Tests
```yaml
Test Types:
1. URL Ping Test:
   - Simple GET request
   - 5+ global locations
   - Alert on failure

2. Multi-Step Test:
   - Recorded user scenario
   - Visual Studio web test
   - Complex workflows

3. Custom TrackAvailability:
   - Code-based checks
   - Custom logic
   - Run from Azure Functions
```

## Choosing Implementation Approach

**Use Runtime Instrumentation when**:
- ✅ Existing production app
- ✅ Cannot modify code
- ✅ Quick setup needed
- ✅ Web application only

**Use SDK Integration when**:
- ✅ New application development
- ✅ Need custom business events
- ✅ Full control over telemetry
- ✅ All application types

**Use Both when**:
- ✅ Complete monitoring required
- ✅ Server + client visibility
- ✅ Maximum insights

## Critical Notes
- ⚠️ **Free tier**: First 5 GB/month included
- 💡 **Live Stream**: Essential for deployment validation
- 🎯 **Operation ID**: Links all telemetry for a request
- 📊 **Custom properties**: Enable segmentation in analytics
- 🔄 **Consistent naming**: Use conventions for events/metrics

## Quick Setup Checklist
```yaml
☐ Create Application Insights resource
☐ Get connection string
☐ Install SDK or enable runtime instrumentation
☐ Configure sampling (if high traffic)
☐ Add custom telemetry for business events
☐ Set up availability tests
☐ Create dashboards
☐ Configure alerts
☐ Test end-to-end with Live Metrics
☐ Validate telemetry in portal
```

## Pricing
| Tier | Ingestion | Cost |
|------|-----------|------|
| Free | First 5 GB/month | $0 |
| Pay-as-you-go | Beyond 5 GB | ~$2.30/GB |
| Commitment (100 GB/day) | Predictable usage | ~$1.93/GB (16% discount) |

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-tools-track-usage-flow/7-implement-application-insights)
