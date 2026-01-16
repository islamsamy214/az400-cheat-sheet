# Monitor Application Performance with Azure Load Testing

## Key Concepts
- **Azure Load Testing**: Fully managed load-testing service for high-scale traffic simulation
- **Platform-agnostic**: Test applications hosted anywhere (Azure, on-premises, other clouds)
- **JMeter-based**: Uses Apache JMeter for test scripts
- **CI/CD integration**: Automate performance testing in pipelines
- **Application Insights integration**: Correlate load with performance telemetry

## What is Azure Load Testing?
```
Azure Load Testing Service
        ↓
Simulates Traffic (1000s of concurrent users)
        ↓
Target Application (Azure App Service, VMs, on-prem, etc.)
        ↓
Collects Metrics (response time, throughput, errors)
        ↓
Integrates with Application Insights (end-to-end view)
```

## Test Creation Options
| Method | Best For | Complexity |
|--------|----------|------------|
| **URL-based** | Quick tests, simple endpoints | Low |
| **JMeter script upload** | Complex scenarios, multi-step workflows | Medium |
| **Existing JMeter** | Reuse test scripts, advanced testing | High |

### Quick Start: URL-Based Test
```yaml
# No JMeter knowledge required
Test Configuration:
- Target URL: https://myapp.azurewebsites.net
- Engine instances: 5 (scalable load generation)
- Ramp-up time: 2 minutes
- Duration: 10 minutes
- Virtual users: 250 per instance (1250 total)
- Request rate: Target RPS (optional)
```

### Advanced: JMeter Script
```xml
<!-- example.jmx -->
<jmeterTestPlan>
  <hashTree>
    <TestPlan>
      <ThreadGroup>
        <stringProp name="ThreadGroup.num_threads">100</stringProp>
        <stringProp name="ThreadGroup.ramp_time">60</stringProp>
        <stringProp name="ThreadGroup.duration">600</stringProp>
      </ThreadGroup>
      <HTTPSamplerProxy>
        <stringProp name="HTTPSampler.domain">myapp.com</stringProp>
        <stringProp name="HTTPSampler.path">/api/products</stringProp>
      </HTTPSamplerProxy>
    </TestPlan>
  </hashTree>
</jmeterTestPlan>
```

## Lab Objectives
After completing the hands-on lab, you'll be able to:

✅ Deploy Azure App Service web apps
✅ Compose and run YAML-based CI/CD pipeline
✅ Deploy Azure Load Testing resource
✅ Investigate web app performance using load tests
✅ Integrate load testing into CI/CD pipelines

## Lab Exercises
```
Exercise 0: Configure lab prerequisites
        ↓
Exercise 1: Configure CI/CD pipelines with YAML
        ↓
Exercise 2: Deploy and setup Azure Load Testing
        ↓
Exercise 3: Automate load test with CI/CD
```

## CI/CD Integration Example
```yaml
# azure-pipelines.yml
trigger:
  - main

stages:
  - stage: Build
    jobs:
      - job: BuildApp
        steps:
          - task: DotNetCoreCLI@2
            inputs:
              command: 'build'
  
  - stage: Deploy
    jobs:
      - job: DeployToStaging
        steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Azure-Connection'
              appName: 'myapp-staging'
  
  - stage: LoadTest
    jobs:
      - job: RunLoadTest
        steps:
          - task: AzureLoadTest@1
            inputs:
              azureSubscription: 'Azure-Connection'
              loadTestConfigFile: 'loadtest.yaml'
              resourceGroup: 'rg-loadtest'
              loadTestResource: 'loadtest-prod'
          
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: '$(System.DefaultWorkingDirectory)/_test-results/*.xml'
  
  - stage: DeployProd
    condition: succeeded('LoadTest')  # Only if load test passes
    jobs:
      - job: DeployToProduction
        steps:
          - task: AzureWebApp@1
            inputs:
              appName: 'myapp-prod'
```

## Load Test Configuration (loadtest.yaml)
```yaml
version: v0.1
testName: ProductionLoadTest
testPlan: jmeter/test-plan.jmx
engineInstances: 10
failureCriteria:
  - avg(response_time_ms) > 1000  # Average response > 1s = fail
  - percentage(error) > 5          # Error rate > 5% = fail
  - avg(requests_per_sec) < 100    # Throughput < 100 RPS = fail

secrets:
  - name: API_KEY
    value: $(API_KEY)  # From Azure Key Vault or pipeline variables

env:
  - name: TARGET_URL
    value: https://myapp.azurewebsites.net
  - name: TEST_DURATION
    value: 600  # 10 minutes
```

## Metrics Collected
| Metric | Description | Use Case |
|--------|-------------|----------|
| **Response Time** | Average/P95/P99 latency | Performance SLAs |
| **Throughput** | Requests per second | Capacity planning |
| **Error Rate** | % of failed requests | Reliability tracking |
| **Concurrent Users** | Active virtual users | Load simulation accuracy |
| **Resource Utilization** | CPU/memory during load | Identify bottlenecks |

## Application Insights Integration
```
Azure Load Testing → Generates Load
        ↓
Application → Sends Telemetry → Application Insights
        ↓
Unified View: Load metrics + Application telemetry
        ↓
Analysis: Correlate user load with app performance
```

**Benefits**:
- See response times from both perspectives (client + server)
- Identify database bottlenecks under load
- Correlate exceptions with load spikes
- Understand resource consumption patterns

## Failure Criteria Examples
```yaml
# Fail test if any condition is true
failureCriteria:
  # Performance thresholds
  - avg(response_time_ms) > 2000           # Avg too slow
  - percentile(response_time_ms, 95) > 5000 # P95 too slow
  
  # Reliability thresholds
  - percentage(error) > 2                  # Too many errors
  - percentage(error, http_code=500) > 1   # Server errors
  
  # Throughput requirements
  - requests_per_sec < 500                 # Insufficient RPS
  
  # Resource constraints
  - avg(cpu_percent) > 90                  # CPU maxed out
```

## Load Test Patterns

### Baseline Test
```yaml
# Establish performance baseline
threads: 50
ramp_up: 60  # 1 minute
duration: 300  # 5 minutes
purpose: "Normal load - establish baseline metrics"
```

### Stress Test
```yaml
# Find breaking point
threads: 1000
ramp_up: 300  # 5 minutes
duration: 600  # 10 minutes
purpose: "Identify maximum capacity before failure"
```

### Spike Test
```yaml
# Test sudden traffic increase
threads: 100 → 1000 (instant)
duration: 300
purpose: "Validate autoscaling and resilience"
```

### Soak Test
```yaml
# Test stability over time
threads: 200
duration: 3600  # 1 hour or longer
purpose: "Identify memory leaks, resource exhaustion"
```

## Azure Portal Workflow
```
1. Create Load Test Resource
   → Resource group, region, name

2. Configure Test
   → Upload JMeter script or use URL
   → Set engine instances, duration
   → Define failure criteria

3. Run Test
   → Monitor real-time progress
   → View live metrics (RPS, latency, errors)

4. Analyze Results
   → Response time charts
   → Error breakdown
   → Client-side metrics
   → Server-side telemetry (if App Insights linked)

5. Compare Runs
   → Baseline vs. current
   → Identify regressions
```

## Requirements
- ✅ Azure DevOps organization
- ✅ Azure subscription with Owner role
- ✅ Microsoft Edge or supported browser
- ✅ Estimated time: 60 minutes

## Critical Notes
- ⚠️ **Cost awareness**: Load testing consumes virtual user hours (billed)
- 💡 **Start small**: Begin with low load, increase gradually
- 🎯 **Failure criteria**: Define before testing to automate pass/fail
- 📊 **Compare runs**: Track performance trends over time
- 🔄 **CI/CD integration**: Catch performance regressions early

## Quick Setup Commands
```bash
# Create load test resource
az load create \
    --name "loadtest-prod" \
    --resource-group "rg-loadtest" \
    --location "eastus"

# Run test
az load test run \
    --test-id "my-load-test" \
    --load-test-resource "loadtest-prod" \
    --resource-group "rg-loadtest"

# Get test results
az load test show \
    --test-id "my-load-test" \
    --load-test-resource "loadtest-prod" \
    --resource-group "rg-loadtest"
```

## Benefits
✅ **Fully managed**: No infrastructure to maintain
✅ **High scale**: Simulate millions of users
✅ **CI/CD integration**: Automated performance regression detection
✅ **Application Insights**: End-to-end performance correlation
✅ **Global load generation**: Test from multiple Azure regions

[Launch Lab](https://go.microsoft.com/fwlink/?linkid=2270116)
