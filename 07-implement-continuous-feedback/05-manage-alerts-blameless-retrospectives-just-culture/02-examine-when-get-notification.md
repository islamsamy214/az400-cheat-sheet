# Examine When to Get a Notification

## Key Concepts
- **Smart detection**: Machine learning-based automatic performance analysis
- Uses historical baselines instead of manual thresholds
- Requires 8+ days of telemetry to establish baselines
- Notifications are suggestions to investigate, not confirmed problems
- Minimal setup - activates automatically with Application Insights

## Azure Application Insights Smart Detection
- Automated performance analysis for web applications
- Uses machine learning to understand normal application behavior
- Automatically detects deviations from baselines
- No manual threshold configuration required
- Catches issues you might not think to monitor

## Supported Platforms
- ASP.NET
- Java
- Node.js
- Client-side JavaScript
- Activates when application generates sufficient telemetry volume

## Three Detection Categories

### 1. Response Time Degradation
**What it detects**: Application responding more slowly than historical baseline

**Patterns**:
- **Sudden degradation** (hours): Recent deployment, config change, or infrastructure problem
  - Examples: Inefficient database query, memory leak, code regression
- **Gradual degradation** (days/weeks): Resource exhaustion over time
  - Examples: Memory leaks, database growth without indexing, cache issues, load increases

**Why it matters**:
- Users notice immediately
- Every 100ms latency reduces user satisfaction
- Direct impact on e-commerce conversion rates and revenue

### 2. Dependency Duration Degradation
**What it detects**: External dependencies responding more slowly
- REST APIs, databases, storage services, third-party services

**Common scenarios**:
- Database query performance issues (missing indexes, table growth, concurrent load)
- Third-party API performance problems
- Network connectivity degradation
- Storage service latency during high load

**Why it matters**:
- Modern apps depend heavily on external services
- Dependency problems cascade through application stack
- Helps pinpoint if issue is in your code or external services

### 3. Slow Performance Patterns
**What it detects**: Performance issues affecting specific request subsets

**Pattern examples**:
| Pattern Type | Cause Examples |
|--------------|----------------|
| Browser-specific | JavaScript compatibility, rendering differences, resource loading |
| Geographic | Network latency, CDN config, regional infrastructure |
| Server-specific | Hardware problems, config drift, resource contention |
| User segment | Personalization queries, permission checks, session data |
| Operation-specific | Inefficient code paths, expensive computations, problematic queries |

**Why it matters**: Indicates specific fixable problems, not general capacity issues

## Performance Baselines

### Learning Period (8+ days required)
Smart detection establishes baseline characteristics:
- Time of day patterns (peak/valley periods)
- Day of week variations (business cycles)
- Seasonal traffic trends
- Normal performance variation ranges
- Load-to-response-time correlations

### After Learning Period
- Continuous performance monitoring
- Triggers notifications on significant deviations
- Adapts baselines as application evolves
- Adjusts for changing traffic patterns

## Understanding Notifications

### Notifications = Suggestions, Not Confirmations
- Indicates something unusual occurred worth investigating
- Not definitive confirmation of a problem requiring immediate action

### Why This Distinction Matters
**False positives occur**:
- Some anomalies represent legitimate changes
- Example: Intentional features that process more data per request
- Increased processing time may be expected and acceptable

**Context is critical**:
- Depends on business context, user impact, acceptable standards
- Smart detection provides technical signal
- You evaluate if change is problematic for your situation

**Investigation starting point**:
- Think of notifications as investigation triggers
- Direct attention to areas where performance changed
- You determine if action is needed

## Critical Notes
- ⚠️ 8-day minimum learning period before reliable detection
- 💡 No manual threshold configuration needed
- 🎯 Use notifications as investigation starting points
- 📊 Context determines if anomaly is a real problem
- 🔍 Pattern detection helps prioritize optimization efforts

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-alerts-blameless-retrospectives-just-culture/2-examine-when-get-notification)
