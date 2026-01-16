# Explore How to Fix Performance Issues

## Key Concepts
- Smart detection notifications include rich diagnostic information
- Three critical info types: Triage, Scope, Diagnostic
- Use Application Insights Profiler and Snapshot Debugger
- Follow systematic investigation approach
- Correlate performance issues with recent changes

## Notification Components

### 1. Triage Information

**Impact Assessment Metrics**:
| Metric | Description |
|--------|-------------|
| Affected user count | Number of unique users with degraded performance |
| Affected operation count | Number of requests/operations impacted |
| Percentage of total traffic | Portion of overall traffic affected |
| Time range | When degradation started and duration |

**Prioritization Strategy**:
- High user impact + significant degradation = immediate attention
- Small percentage issues = investigate during normal hours
- Consider business context (e.g., checkout > admin page)

### 2. Scope Information

**Traffic Segmentation Analysis**:

| Scope Type | Questions | Implications |
|------------|-----------|--------------|
| **Page-level** | All pages or specific routes? | Specific code vs. infrastructure issue |
| **Browser** | All browsers or specific ones? | Client-side JS, rendering, compatibility |
| **Geographic** | Specific regions affected? | Network, CDN config, data center issues |
| **Device** | Mobile vs. desktop differences? | Responsive design, resource optimization |
| **Authentication** | Authenticated vs. anonymous users? | Session management, permissions, personalization |

**Using Scope for Investigation**:
- **Narrow scope** → Specific components or code paths
- **Broad scope** → Infrastructure-level issues (DB overload, network, resources)

### 3. Diagnostic Information

**Performance Signatures**:

| Pattern | Indication | Action Needed |
|---------|-----------|---------------|
| **Load correlation** | Response time ↑ when request rate ↑ | Capacity constraints - scale or optimize |
| **Time-based patterns** | Degradation at specific times (e.g., 2 AM) | Scheduled jobs, backups, batch processes |
| **Dependency correlation** | App response ↓ when dependency duration ↑ | External service issue, not your code |
| **Exception correlation** | Exceptions + response time degradation | Application errors causing slowness/retries |

## Investigation Tools

### Application Insights Performance Blade
**Capabilities**:
- **Request distribution analysis**: Response times across percentiles (50th, 90th, 95th, 99th)
- **Operation timeline**: When operations became slow, performance changes over time
- **Dependency analysis**: Which dependencies contribute most to response time
- **Server metrics correlation**: CPU, memory, disk I/O vs. performance

### Application Insights Profiler
**What it provides**:
- Detailed execution traces showing where application spends time
- Available in notifications if captured during degradation

**Profiler Data Includes**:
- Method-level timing (which functions consume most time)
- Call stacks (execution path through code)
- Database query execution times and patterns
- External API call durations
- Thread behavior and blocking operations

**Effectiveness**:
- Reveals bottlenecks with precision
- Example: If DB query takes 2s of 3s request → focus optimization there
- No manual instrumentation or guesswork needed

### Snapshot Debugger
**What it provides**:
- Complete application state at moment of exception (if exceptions during degradation)

**Snapshot Capabilities**:
- Full call stack at exception time
- Variable values and object states
- Request context and parameters
- Session and user information

**When to use**: Performance degradation correlating with increased exceptions

## Systematic Investigation Approach

### Step 1: Assess Impact and Prioritize
- Review triage information for severity
- Determine if immediate response required or can wait

### Step 2: Identify Scope
- Examine scope information
- Understand which traffic segments affected
- Narrow investigation focus

### Step 3: Form Hypotheses
- Based on diagnostic information
- Example: Degradation + high load + all traffic = capacity constraints

### Step 4: Use Investigative Tools
- Open Performance blade, examine detailed telemetry
- Review Profiler traces for specific slow code paths
- Check Snapshot Debugger if exceptions involved

### Step 5: Correlate with Changes
Check what changed when degradation started:
- Recent deployments
- Configuration changes
- Database schema changes
- Traffic pattern shifts

### Step 6: Verify and Remediate
- Implement fix
- Verify performance returns to normal baselines
- Continue monitoring to ensure full resolution

## Critical Notes
- 💡 Profiler data eliminates guesswork in performance optimization
- 🎯 Use scope to narrow investigation focus significantly
- ⚠️ Always correlate issues with recent changes
- 📊 Percentile analysis shows if all requests or just tail latencies affected
- 🔍 Snapshot debugger invaluable when exceptions correlate with degradation

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-alerts-blameless-retrospectives-just-culture/3-explore-how-to-fix-it)
