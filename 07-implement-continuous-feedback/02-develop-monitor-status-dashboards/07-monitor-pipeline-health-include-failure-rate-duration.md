# Monitor Pipeline Health: Failure Rate, Duration, and Flaky Tests

## Key Concepts
- **Pipeline failure rate** = (Failed Runs / Total Runs) × 100 — measures reliability
- **Pipeline duration** = Queue time + Execution time + Approval time — impacts feedback speed
- **Flaky tests** produce inconsistent results (pass/fail) without code changes — undermine confidence
- **Monitoring tools**: Azure Pipelines Analytics, Application Insights, Service Hooks, third-party integrations
- **Establish baselines** early to detect anomalies and measure improvements
- **Act on insights**: Monitoring without remediation wastes resources

## Critical Pipeline Health Metrics

### Pipeline Failure Rate

**Definition**: Percentage of pipeline runs that fail within a timeframe

**Why It Matters**:
- Reliability indicator for delivery confidence
- Team productivity (frequent failures interrupt workflow)
- Customer impact (failed deployments prevent fixes reaching production)
- Cost implications (each failure consumes resources + engineer time)

**Common Failure Causes**:

| Category | Examples | Root Cause |
|----------|----------|------------|
| **Code Quality** | Compilation errors, test failures, code analysis violations | Bugs, missing dependencies, incompatible libraries |
| **Infrastructure** | Agent unavailability, network connectivity, resource constraints | Agent pool exhaustion, transient network issues, OOM |
| **Configuration** | YAML syntax errors, missing variables, service connection failures | Pipeline definition mistakes, expired credentials |
| **Environmental** | External service dependencies, timing issues, capacity limits | Third-party API downtime, race conditions, throttling |

**Monitoring Strategy**:
```yaml
Establish Baselines:
  - Calculate typical failure rate over stable periods (e.g., last 30 days)
  - Example baseline: 5%

Define Thresholds:
  - Warning: Failure rate exceeds baseline by 50% (e.g., 7.5%)
  - Critical: Failure rate exceeds baseline by 100% (e.g., 10%)
  - Sustained: Require 3+ failures in 1 hour to avoid false alarms

Configure Alerts:
  - Warnings → Team channel
  - Critical → On-call engineer
  - Include: Failure reasons, affected branches, recent changes
```

**Analysis Techniques**:

| Analysis Type | Method | Insight Example |
|---------------|--------|-----------------|
| **Trend Analysis** | Daily/weekly trends | "Pass rate dropped from 95% to 85% after library upgrade" |
| **Segmentation** | By pipeline, branch, trigger, failure reason | "Production Release pipeline has 15% failure rate (3× baseline)" |
| **Day-of-Week Patterns** | Compare Monday vs. Friday | "Higher failures on Mondays (accumulated changes)" |

### Pipeline Duration

**Definition**: Total elapsed time from trigger to completion

**Duration Categories**:

| Phase | Cause | Optimization |
|-------|-------|--------------|
| **Queue Duration** | Insufficient agent capacity | Add more agents, optimize agent pool, off-peak scheduling |
| **Checkout & Restore** | Large repos, many dependencies, slow network | Shallow git clone, package caching, parallel restores |
| **Build/Compile** | Large codebases, slow compilers | Incremental compilation, distributed builds, faster machines |
| **Test Execution** | Slow tests, many tests, serial execution | Parallel test execution, faster infrastructure, test selection |
| **Deployment** | Large artifacts, slow targets | Incremental deployments, parallel multi-region, deployment slots |

**Track Percentiles, Not Averages**:
```yaml
P50 (Median): 8 min — Typical duration most runs experience
P90: 12 min — Duration that 90% of runs complete within
P95/P99: 25 min — Captures outliers and worst-case scenarios

Insight: P50=8 min, P90=12 min, P99=25 min → Significant variance
```

**Duration Baselines**:
- **Ideal duration**: Target based on team needs (e.g., "< 10 minutes for CI")
- **Current baseline**: Typical duration from recent stable period
- **Improvement goals**: Specific reduction targets (e.g., "reduce P90 from 12 min to 8 min")

**Duration Breakdown Analysis**:
```yaml
Example Task Breakdown:
  - Checkout: 1 min (8%)
  - Restore packages: 2 min (17%)
  - Build: 5 min (42%) ← Focus optimization here
  - Test: 3 min (25%)
  - Publish artifacts: 1 min (8%)

Action: Optimize "Build" task (largest contributor)
```

### Flaky Tests

**Definition**: Tests that produce inconsistent results (pass/fail) under identical conditions without code changes

**Why Problematic**:

| Impact | Consequence |
|--------|-------------|
| **Undermine Confidence** | Teams ignore failures assuming they're flaky → Real bugs get deployed |
| **Pipeline Instability** | Intermittent build failures require re-runs |
| **Wasted Productivity** | Engineers spend hours debugging tests that aren't broken |
| **Organizational Cost** | Studies show flaky tests cost thousands of engineer-hours annually |

**Common Causes**:

| Category | Examples | Solution |
|----------|----------|----------|
| **Timing & Concurrency** | Race conditions, insufficient wait times, thread safety | Use explicit waits, avoid hard-coded delays |
| **Environmental Dependencies** | External services, network variability, resource contention | Mock external services, use test containers |
| **Test Isolation Failures** | Shared state, test order dependency, global variables | Proper setup/teardown, unique identifiers |
| **Timing Sensitivity** | Hard-coded delays, clock dependencies, animation waits | Replace `sleep()` with explicit condition waits |
| **Non-Deterministic Logic** | Random values, unordered collections, floating-point precision | Use fixed test data, explicit ordering |
| **Infrastructure Variability** | Agent differences, resource constraints, OS differences | Standardize agents, increase timeouts |

**Detecting Flaky Tests**:
```yaml
Test Analytics in Azure Pipelines:
  - Track individual test pass/fail history
  - Identify tests with sporadic failures (e.g., 20% failure rate)
  - Example: "Login_ShouldSucceed" passes 80% → Likely flaky

Flaky Test Detection Rules:
  - Definition: Test fails in some runs but passes in others (last 30 days, no code changes)
  - Threshold: Mark as flaky if test has both passes/fails in 10+ recent runs
  - Quarantine: Automatically separate flaky from stable tests
```

**Remediation Strategies**:

| Strategy | Implementation | Example |
|----------|----------------|---------|
| **Isolation Improvements** | Full setup/teardown, unique identifiers, parallel-safe | Use GUIDs for test data to avoid conflicts |
| **Timing Fixes** | Replace sleep with waits, increase timeouts, retry policies | `waitForElementVisible()` instead of `Thread.Sleep(1000)` |
| **Dependency Management** | Mock external services, local environments, service virtualization | Use WireMock/Mountebank for external APIs |
| **Test Design** | Deterministic data, order independence, explicit ordering | Fixed test data instead of random values |

**Quarantine Strategy**:
```yaml
Concept: Separate flaky tests, don't fail pipeline based on them

Implementation:
  1. Tag flaky tests: [Trait("Category", "Flaky")]
  2. Run in separate pipeline or stage
  3. Report results as informational, not blocking

Exit Criteria:
  - Stability: Pass consistently for 30+ consecutive runs
  - Fix validation: Root cause addressed, not just symptoms
  - Documentation: Document what was flaky and how fixed
```

## Pipeline Health Monitoring Tools

### Azure Pipelines Analytics (Built-In Reports)

| Report | Visualizes | Use Case |
|--------|-----------|----------|
| **Pipeline Pass Rate** | Pass/fail % trends over time | Track reliability improvements |
| **Task Failure Analysis** | Failure count by task type | Focus remediation on highest-impact tasks |
| **Pipeline Duration** | Duration distribution (P50, P90, P95) | Identify performance regressions |
| **Test Failures Report** | Top failing tests with frequency | Prioritize test fixes by impact |

**Accessing Reports**:
```
1. Azure DevOps → Pipelines → Analytics
2. Choose report type
3. Configure time range and filters
```

### Application Insights Integration

**Continuous Monitoring in Release Pipelines**:
```yaml
Workflow:
  1. Deploy to staging
  2. Monitor metrics for 15 minutes (Application Insights)
  3. Evaluate thresholds:
     - Response time P95 < 500 ms?
     - Error rate < 1%?
     - Dependency success > 99%?
  4. Gate decision:
     - Pass → Deploy to production automatically
     - Fail → Block deployment, trigger rollback
```

**Configuration**:
```yaml
stages:
  - stage: Production
    jobs:
      - deployment: DeployProd
        environment: production
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    appName: "myapp-prod"

                # Post-deployment monitoring
                - task: AzureMonitorAlerts@1
                  inputs:
                    alertRules: "HighErrorRate,SlowResponseTime"
                    evaluationDuration: "10m"
                    actionOnFailure: "rollback"
```

### Azure DevOps Service Hooks

**Common Scenarios**:

| Integration | Event | Action | Use Case |
|-------------|-------|--------|----------|
| **Microsoft Teams** | Build completed | Post to channel | Real-time team notifications |
| **PagerDuty** | Pipeline failure | Create incident | On-call alerting |
| **Azure Automation** | Deployment failed | Trigger runbook | Automated remediation |
| **Webhooks** | Any pipeline event | Call custom API | Custom processing |

**Configuration**:
```
1. Project Settings → Service Hooks → Create subscription
2. Select service (Teams, Slack, Webhooks)
3. Select trigger (Build completed, Release started)
4. Configure filters (only failures, specific pipelines)
5. Configure action (post message, send JSON payload)
```

### Third-Party Monitoring Tools

| Tool | Capabilities | Best For |
|------|-------------|----------|
| **DataDog** | Unified monitoring, custom dashboards, anomaly detection, APM | Multicloud/hybrid environments |
| **New Relic** | APM, deployment markers, distributed tracing | Deep application performance insights |
| **Splunk** | Log aggregation, security analytics, compliance reporting | Large enterprises, compliance needs |
| **Prometheus + Grafana** | Open-source metrics collection, flexible visualization | Teams preferring open-source |

## Best Practices

```yaml
Establish Baselines Early:
  - Track metrics from day one
  - Baseline enables anomaly detection

Automate Alerting:
  - Don't rely on manual dashboard checking
  - Alert on actionable thresholds

Make Metrics Visible:
  - Display on NOC screens, team areas
  - Include in standups and retrospectives

Act on Insights:
  - Monitoring without action wastes resources
  - Allocate sprint capacity to pipeline health

Continuous Improvement:
  - Set specific goals: "Reduce P90 by 20%", "Eliminate flaky tests"
  - Measure impact using baseline comparisons

Correlate Events:
  - Link pipeline changes to app performance
  - Understand cause-effect relationships
```

## Critical Notes
- ⚠️ **Track percentiles, not averages**: P90/P95 captures outliers
- 💡 **Quarantine flaky tests**: Don't let them block pipelines
- 🎯 **Monitor the metrics**: Failure rate, duration, flaky test count
- 📊 **Automate remediation**: Use gates, rollbacks, and automatic retries

## Quick Commands

**Query Pipeline Failure Rate (Azure CLI)**:
```bash
# Get pipeline runs for last 30 days
az pipelines runs list \
  --project MyProject \
  --pipeline-ids 123 \
  --query "[?finishedDate>=`$thirty_days_ago`].{result:result}" \
  --output table
```

**Identify Flaky Tests (KQL)**:
```kql
TestResults
| where CompletedDate > ago(30d)
| summarize 
    TotalRuns = count(),
    PassedRuns = countif(Outcome == "Passed"),
    FailedRuns = countif(Outcome == "Failed")
    by TestName
| where PassedRuns > 0 and FailedRuns > 0
| extend FailureRate = round(FailedRuns * 100.0 / TotalRuns, 2)
| where FailureRate between (10 .. 90)  // Flaky if fails 10-90% of time
| order by FailureRate desc
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-monitor-status-dashboards/7-monitor-pipeline-health-include-failure-rate-duration)
