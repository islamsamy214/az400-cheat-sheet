# Examine Kusto Query Language (KQL)

## Key Concepts
- **KQL**: Powerful query language for log analysis in Azure Monitor
- **Read-like flow**: Left-to-right, top-to-bottom with pipe operators (`|`)
- **Optimized for logs**: Time-series data, columnar storage, sub-second queries over terabytes
- **100+ operators**: Filtering, aggregation, joins, time-series, ML, visualization

## KQL vs. SQL
| Aspect | KQL | SQL |
|--------|-----|-----|
| Syntax | Pipe-based (`\|`) | Nested SELECT |
| Optimization | Read-heavy analytics | Transactional workloads |
| Time handling | First-class time-series | Limited temporal functions |
| Complexity | Simpler for log analysis | Complex for analytics |

## Essential KQL Operators
| Operator | Purpose | Example |
|----------|---------|---------|
| `where` | Filter rows | `where ObjectName == "Processor"` |
| `summarize` | Aggregate data | `summarize avg(CPU) by Computer` |
| `bin()` | Time buckets | `bin(TimeGenerated, 5m)` |
| `ago()` | Relative time | `ago(24h)` |
| `render` | Visualize | `render timechart` |
| `contains` | Substring match | `where name contains "sql"` |
| `!in` | Exclusion | `where process !in ("Idle")` |
| `count` | Count rows | `\| count` |

## Query Pattern 1: Last Heartbeat
```kusto
// Find most recent heartbeat per machine
Heartbeat
| summarize arg_max(TimeGenerated, *) by Computer
```

**Use case**: Health check, verify agent connectivity

## Query Pattern 2: Discover Data
```kusto
// List all collected performance counters
Perf
| summarize by ObjectName, CounterName
```

**Use case**: Configuration audit, identify available metrics

## Query Pattern 3: Data Volume
```kusto
// Count data points in last 24 hours
Perf
| where TimeGenerated > ago(24h)
| count
```

**Use case**: Understand scale, estimate costs

## Query Pattern 4: CPU Trend Visualization
```kusto
// Max CPU over last 24h, 1-minute buckets
Perf
| where ObjectName == "Processor" and InstanceName == "_Total"
| summarize max(CounterValue) by Computer, bin(TimeGenerated, 1m)
| render timechart
```

**Use case**: Identify performance patterns, capacity planning

## Query Pattern 5: Process CPU Analysis
```kusto
// Top CPU-consuming processes
Perf
| where ObjectName contains "process"
        and InstanceName !in ("_Total", "Idle")
        and CounterName == "% Processor Time"
| summarize avg(CounterValue) by InstanceName
| top 10 by avg_CounterValue desc
| render barchart
```

**Use case**: Identify runaway processes, troubleshoot performance

## Time Functions
```kusto
// Relative time
| where TimeGenerated > ago(1h)      // Last hour
| where TimeGenerated > ago(7d)      // Last 7 days
| where TimeGenerated > ago(30m)     // Last 30 minutes

// Time bucketing
| summarize count() by bin(TimeGenerated, 1m)   // 1-minute buckets
| summarize count() by bin(TimeGenerated, 1h)   // 1-hour buckets
| summarize count() by bin(TimeGenerated, 1d)   // 1-day buckets

// Time range
| where TimeGenerated between (datetime(2024-01-01) .. datetime(2024-01-31))
```

## Aggregation Functions
| Function | Purpose | Example |
|----------|---------|---------|
| `count()` | Count rows | `summarize count()` |
| `avg()` | Average | `summarize avg(CPU)` |
| `max()` | Maximum | `summarize max(Memory)` |
| `min()` | Minimum | `summarize min(DiskSpace)` |
| `sum()` | Sum | `summarize sum(Bytes)` |
| `percentile()` | Percentile | `summarize percentile(Duration, 95)` |
| `arg_max()` | Row with max value | `summarize arg_max(Time, *)` |

## Advanced Patterns

### Percentiles for SLA Tracking
```kusto
Perf
| where ObjectName == "Processor"
| summarize 
    P50 = percentile(CounterValue, 50),
    P95 = percentile(CounterValue, 95),
    P99 = percentile(CounterValue, 99) 
    by bin(TimeGenerated, 5m)
| render timechart
```

### Compare Time Periods
```kusto
let baseline = Perf
    | where TimeGenerated between (ago(2d) .. ago(1d))
    | summarize BaselineCPU = avg(CounterValue);
Perf
| where TimeGenerated > ago(1h)
| summarize CurrentCPU = avg(CounterValue)
| extend Baseline = toscalar(baseline)
| extend PercentChange = (CurrentCPU - Baseline) / Baseline * 100
```

### Multi-condition Filtering
```kusto
Perf
| where ObjectName == "Processor"
    and CounterName == "% Processor Time"
    and CounterValue > 80
    and Computer in ("vm-web01", "vm-web02")
| summarize Incidents = count() by Computer, bin(TimeGenerated, 1h)
```

## Visualization Options
| Render Type | Best For | Example |
|-------------|----------|---------|
| `timechart` | Time-series trends | CPU over time |
| `barchart` | Comparisons | Top 10 processes |
| `piechart` | Distributions | Process CPU share |
| `table` | Detailed data | Event logs |
| `columnchart` | Category comparisons | Errors by type |

## Performance Tips
```kusto
// ✅ GOOD: Filter early
Perf
| where TimeGenerated > ago(1h)        // Filter first
| where ObjectName == "Processor"
| summarize avg(CounterValue)

// ❌ BAD: Filter late
Perf
| summarize avg(CounterValue)
| where TimeGenerated > ago(1h)        // Filter after aggregation
```

## Common KQL Patterns
```kusto
// Pattern: Find spikes
| summarize Value = avg(CounterValue) by bin(TimeGenerated, 5m)
| extend Baseline = avg(Value)
| where Value > (Baseline * 1.5)      // 50% above average

// Pattern: Join tables
requests
| join (dependencies) on operation_Id
| where duration > 1000

// Pattern: Dynamic thresholds
| make-series Value = avg(CounterValue) on TimeGenerated step 5m
| extend (anomalies, score, baseline) = series_decompose_anomalies(Value)
```

## Where to Use KQL
- ✅ Log Analytics (Azure Monitor Logs)
- ✅ Application Insights
- ✅ Azure Resource Graph
- ✅ Microsoft Sentinel
- ✅ Azure Data Explorer
- ✅ Azure Data Studio
- ✅ PowerShell/CLI (programmatic)

## Critical Notes
- ⚠️ **Always filter by time first**: Improves query performance dramatically
- 💡 **Use bin() for visualizations**: Reduces millions of points to manageable buckets
- 🎯 **Percentiles > Averages**: P95/P99 better represent user experience
- 📊 **Save common queries**: Reuse patterns, share with team
- 🔄 **IntelliSense in portal**: Use Ctrl+Space for autocomplete

## Quick Reference
```kusto
// Essential operators cheat sheet
| where                // Filter
| summarize            // Aggregate
| extend               // Add calculated column
| project              // Select columns
| top N by column      // Get top N rows
| sort by column desc  // Sort results
| join                 // Combine tables
| union                // Combine similar tables
| render              // Visualize
```

## Learning Resources
- 📚 [KQL Quick Reference](https://learn.microsoft.com/en-us/azure/data-explorer/kql-quick-reference)
- 🎓 [KQL Tutorial](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/tutorial)
- 💡 Sample queries in Azure portal
- 🔧 [KQL Playground](https://dataexplorer.azure.com/)

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-tools-track-usage-flow/5-examine-kusto-query-language-kql)
