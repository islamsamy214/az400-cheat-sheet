# Improve Application Performance

## Key Concepts
- Slow/failed responses directly impact business outcomes
- Triage based on business context, not just technical metrics
- Page load time divided into distinct phases
- Apply targeted optimizations based on diagnosis
- Failed requests worse than slow requests

## Triage Performance Issues

### Assess Business Impact
Balance technical metrics with business context:

| Factor | Considerations |
|--------|----------------|
| **Usage frequency** | Low traffic page = limited impact (but consider purpose) |
| **Page purpose** | Critical checkout flow > rarely used admin page |
| **Severity** | Exceptions for all users = critical regardless of traffic |
| **Business function** | Revenue-generating transactions get priority |
| **User type** | High-value customers, external vs. internal users |
| **Workflow criticality** | New user onboarding, critical workflows |

### Prioritization Guidelines
- **Fix failures before optimizing performance** (exceptions > slowness)
- Use Application Insights impact data (affected users, traffic %)
- Gather additional business context beyond metrics
- Consider geographic issues separately (network, CDN, data center)

### Geographic Considerations
- Set up availability tests for specific regions
- Regional problems often indicate network/CDN/data center issues (not code)

## Diagnose Slow Page Loads

### Page Load Time Components
Application Insights segments browser page load into phases:

### 1. Send Request Time

**High send request time indicates**: Server responds slowly or request contains large data

**Investigation Steps**:
- Review performance metrics for response time patterns
- Identify specific API endpoints/page renders with high response times
- Check POST requests for excessive data (optimize/compress)
- Correlate slow responses with server resource utilization (CPU, memory, disk I/O)

**Common Causes**:
- Database queries with missing indexes or inefficiency
- Synchronous processing (should be asynchronous)
- Insufficient server resources for current load
- Unoptimized algorithms or inefficient code paths

### 2. Dependency Tracking

**Setup**: Configure dependency tracking to identify external service/database slowness

**What It Reveals**:
- Database response wait times
- External API call durations and failure rates
- Cache hit rates and lookup times
- Storage operation latencies

**Usage**: If dependencies consume most request time, focus on external services
- Implement caching strategies
- Optimize queries
- Review service-level agreements with external providers

### 3. Receiving Response Time

**High receiving response indicates**: Large pages with many dependent resources (JS, CSS, images)

**Investigation Approach**: Use availability test with "load dependent parts" enabled
- Which JavaScript bundles are largest/slowest?
- Are images optimized for web?
- Does CSS load efficiently?
- Are fonts loading from appropriate CDNs?

**Action**: Prioritize optimization on files contributing most to load time

### 4. Client Processing Time

**High client processing suggests**: JavaScript executes slowly in browser

**Investigation Techniques**:
- Browser developer tools to profile JavaScript execution
- Identify expensive operations (DOM manipulation, complex calculations, inefficient frameworks)
- Add custom timing instrumentation (Application Insights track metrics)
- Test across different browsers/devices for platform-specific issues

## Improve Slow Pages

### Large File Optimization

| Technique | Description | Benefit |
|-----------|-------------|---------|
| **Load asynchronously** | Scripts/resources don't block page rendering | Faster initial render |
| **Script bundling** | Combine multiple JS files into bundles | Reduce HTTP requests |
| **Code splitting** | Break apps into chunks that load on demand | Avoid loading everything upfront |
| **Widget architecture** | Structure as widgets loading data independently | Critical content renders first |
| **Efficient data formats** | JSON data transfer > HTML table rendering | Reduce initial payload |
| **Modern frameworks** | React, Vue, Angular optimize rendering | Better performance (with JS overhead trade-off) |

### Server Dependency Optimization

| Technique | Description | Benefit |
|-----------|-------------|---------|
| **Geographic co-location** | Web servers + databases in same Azure region | Minimize network latency |
| **Query efficiency** | Retrieve only necessary data, avoid SELECT * | Reduce data transfer |
| **Caching strategies** | Redis/distributed cache for frequent data | Avoid repeated queries |
| **Request batching** | Combine multiple requests into batch operations | Reduce network round trips |
| **Connection pooling** | Efficient database connection management | Avoid connection overhead |

### Capacity Issue Resolution

| Technique | When to Use | Action |
|-----------|-------------|--------|
| **Monitor server metrics** | Response times spike during request peaks | Identify resource constraints |
| **Horizontal scaling** | Need to distribute load | Add more server instances |
| **Vertical scaling** | Individual requests need more power | Increase CPU, memory |
| **Auto-scaling** | Traffic variations | Configure automatic scaling |
| **Load testing** | Before production issues | Identify breaking points/capacity limits |

## Optimization Workflow

```bash
# 1. Diagnose bottleneck
Application Insights → Browsers → Page Load Time Components

# 2. Identify phase with most time
Send Request | Dependency | Receiving Response | Client Processing

# 3. Apply targeted optimization
Based on bottleneck identified

# 4. Verify improvement
Monitor metrics after changes
Compare before/after performance
```

## Critical Notes
- ⚠️ Failed requests > slow requests (fix failures first)
- 💡 Use business context for prioritization, not just metrics
- 🎯 Optimize files contributing most to load time
- 📊 Correlate response times with request counts to identify capacity issues
- 🔍 Set up availability tests for geographic-specific problems
- 🚀 Modern frameworks optimize rendering but add JS overhead

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-alerts-blameless-retrospectives-just-culture/5-improve-performance)
