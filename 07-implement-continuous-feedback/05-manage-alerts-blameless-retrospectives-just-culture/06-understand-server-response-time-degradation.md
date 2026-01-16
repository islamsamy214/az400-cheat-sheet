# Understand Server Response Time Degradation

## Key Concepts
- Identifies gradual performance erosion (not just sudden spikes)
- Uses statistical models of typical response times per operation type
- Requires 8 days to establish baselines
- Focuses on sustained degradation, not temporary spikes
- Operation-specific analysis (each operation evaluated independently)

## How Detection Works

### Learning Typical Behavior
Application Insights gathers telemetry and learns that operations have different natural response times:
- Complex database queries > simple lookups
- Large data exports > small result sets
- Administrative operations may be slower than user-facing transactions
- Image processing ≠ text-based operations

### Baseline Establishment (8 Days Typical)
Tracks for each operation type:
- Median response times
- 95th percentile response times
- Variance and standard deviation
- Time-of-day patterns (peak hour slowdowns)
- Day-of-week patterns (weekend batch processing effects)

### Degradation Detection
Alerts when responses consistently slower than learned baseline

**Statistical Significance**:
- Uses statistical analysis to distinguish genuine degradation from normal variance
- Short-term fluctuations don't trigger alerts
- Sustained changes outside expected variance ranges do trigger

**Operation-Specific Analysis**:
- Each operation type evaluated independently
- 20% slowdown in fast operation may be significant
- Same percentage change in naturally slow operation may be normal variance

## What Notifications Tell You

### Comprehensive Diagnostic Information

| Information Type | Details Provided |
|------------------|------------------|
| **Performance comparison** | % change and absolute time difference vs. baseline |
| **User impact** | Number of affected users (for prioritization) |
| **Detailed metrics** | Average + 90th percentile response times (today vs. 7 days ago) |
| **Volume context** | Request counts (today vs. 7 days ago) |
| **Dependency correlation** | Related dependency degradations |

### Why These Metrics Matter

**90th Percentile**:
- Shows how slower requests perform (not just typical requests)
- More valuable than average for identifying tail latency issues

**Volume Context**:
- If volume doubled + response time increased 50% → load is primary factor
- Helps distinguish capacity issues from code issues

**Dependency Correlation**:
- If database dependency slowed simultaneously → external cause
- Helps pinpoint root cause location

## Diagnostic Links Provided

### 1. Profiler Traces
**When available**: If Profiler captured traces during detection period

**Shows**:
- Method-level timing information
- Call stacks
- Specific code paths consuming most time
- Where operation time is spent

### 2. Performance Reports
**Provides**: Metric Explorer access for time slicing/filtering

**Capabilities**:
- Compare performance across time windows
- Identify patterns
- Correlate with deployments or configuration changes

### 3. Call Search
**Purpose**: View specific call properties

**Details**:
- Individual request details
- Parameters
- Timing breakdowns
- Associated telemetry
- Performance variations

### 4. Failure Reports
**When shown**: If count > 1 failures occurred

**Indicates**:
- Failures might contribute to performance degradation
- Performance degradation and errors sometimes related
- Retry logic may cause slow overall response times

## Requirements for Effective Detection

### 1. Application Insights Telemetry
- Active instrumentation required
- Continuous telemetry for building/maintaining baselines

### 2. Standard SDK Conventions
Follow standard Application Insights SDK platform conventions:
- Request names consistent and meaningful
- Operation types correctly categorized
- Dependencies properly tracked
- Telemetry includes necessary context for analysis

### 3. Sufficient Data Volume
- Statistical models need adequate data to distinguish patterns from noise
- Very low traffic applications may lack data for reliable detection

### 4. Consistent Telemetry
- Regular telemetry flow enables accurate baseline maintenance
- Large telemetry gaps reduce detection accuracy

## Detection Focus

| Detection Type | What It Catches |
|----------------|-----------------|
| **Sustained degradation** | ✅ Gradual performance erosion over time |
| **Temporary spikes** | ❌ Short-term fluctuations ignored |
| **Statistical anomalies** | ✅ Changes outside expected variance |
| **Normal variance** | ❌ Within expected fluctuation range |

## Critical Notes
- ⚠️ 8-day baseline establishment period required
- 💡 90th percentile more valuable than average for tail latency
- 🎯 Each operation type evaluated independently
- 📊 Volume context helps distinguish capacity vs. code issues
- 🔍 Dependency correlation pinpoints root cause location
- 🚀 Follow standard SDK conventions for best detection accuracy

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-alerts-blameless-retrospectives-just-culture/6-understand-server-response-time-degradation)
