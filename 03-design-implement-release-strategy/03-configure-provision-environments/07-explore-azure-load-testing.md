# Explore Azure Load Testing

⏱️ **Duration**: ~4 minutes | 📚 **Type**: Conceptual

## Overview

Azure Load Testing is a fully managed service that generates high-scale load to simulate realistic user traffic and identify performance bottlenecks. Built on Apache JMeter, it integrates seamlessly with Azure DevOps and GitHub Actions for continuous performance validation.

## Learning Objectives

After completing this unit, you'll be able to:
- ✅ Understand Azure Load Testing capabilities and architecture
- ✅ Create load tests using Apache JMeter scripts
- ✅ Integrate load testing with CI/CD pipelines
- ✅ Analyze performance metrics and identify bottlenecks
- ✅ Implement continuous performance validation

---

## Azure Load Testing Overview

### What is Azure Load Testing?

**Azure Load Testing** is a managed cloud service that enables:
- 🚀 High-scale load generation (up to 1,000+ virtual users per engine)
- 📊 Real-time performance metrics and insights
- 🔄 CI/CD pipeline integration
- 🎯 Application component monitoring (via App Insights)
- 📈 Comparative analysis across test runs
- 🌍 Multi-region load generation

### Key Features

| Feature | Description | Benefit |
|---------|-------------|---------|
| **JMeter Compatibility** | Uses standard Apache JMeter scripts (.jmx) | Leverage existing scripts, no lock-in |
| **Autoscaling** | Automatically scales test engines based on load | Simulate millions of concurrent users |
| **App Insights Integration** | Correlates load with server-side metrics | Identify resource bottlenecks |
| **CI/CD Integration** | Built-in tasks for Azure Pipelines & GitHub Actions | Fail deployments on performance regression |
| **Private Endpoint Testing** | Test internal apps via VNet integration | Validate performance of private services |
| **Parametrization** | Dynamic test data from CSV files | Realistic user scenarios |

---

## Getting Started with Azure Load Testing

### Step 1: Create Azure Load Testing Resource

#### Using Azure Portal

**Navigation**: Azure Portal → Create a resource → Azure Load Testing

**Configuration**:
```
Project Details:
- Subscription: [Your subscription]
- Resource Group: rg-devops-performance
- Name: alt-myapp-perf
- Region: East US (near your app for accurate latency)

Networking:
- Public access: Enabled (or VNet integration for private testing)
```

#### Using Azure CLI

```bash
# Install Azure Load Testing extension
az extension add --name load

# Create resource group
az group create --name rg-devops-performance --location eastus

# Create Azure Load Testing resource
az load create \
    --name alt-myapp-perf \
    --resource-group rg-devops-performance \
    --location eastus

# Get resource details
az load show \
    --name alt-myapp-perf \
    --resource-group rg-devops-performance
```

#### Using Bicep

**File**: `load-testing.bicep`

```bicep
param loadTestName string
param location string = resourceGroup().location
param tags object = {}

resource loadTesting 'Microsoft.LoadTestService/loadtests@2022-12-01' = {
  name: loadTestName
  location: location
  tags: tags
  properties: {
    description: 'Load testing for application performance validation'
  }
}

output loadTestingId string = loadTesting.id
output loadTestingDataPlaneUri string = loadTesting.properties.dataPlaneURI
```

**Deploy**:
```bash
az deployment group create \
    --resource-group rg-devops-performance \
    --template-file load-testing.bicep \
    --parameters loadTestName=alt-myapp-perf
```

---

## Creating Load Tests

### Step 2: Create JMeter Test Script

#### Basic HTTP Load Test

**File**: `load-test-basic.jmx`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2" properties="5.0">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="API Load Test">
      <stringProp name="TestPlan.comments">Basic load test for REST API</stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
      <elementProp name="TestPlan.user_defined_variables" elementType="Arguments">
        <collectionProp name="Arguments.arguments">
          <elementProp name="BASE_URL" elementType="Argument">
            <stringProp name="Argument.name">BASE_URL</stringProp>
            <stringProp name="Argument.value">${__P(BASE_URL,https://api.example.com)}</stringProp>
          </elementProp>
        </collectionProp>
      </elementProp>
    </TestPlan>
    <hashTree>
      <!-- Thread Group: Simulates concurrent users -->
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="User Load">
        <stringProp name="ThreadGroup.num_threads">${__P(threads,50)}</stringProp>
        <stringProp name="ThreadGroup.ramp_time">${__P(rampup,30)}</stringProp>
        <stringProp name="ThreadGroup.duration">${__P(duration,300)}</stringProp>
        <stringProp name="ThreadGroup.delay">0</stringProp>
        <boolProp name="ThreadGroup.scheduler">true</boolProp>
        <boolProp name="ThreadGroup.same_user_on_next_iteration">true</boolProp>
      </ThreadGroup>
      <hashTree>
        <!-- HTTP Request: GET /api/products -->
        <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Get Products">
          <elementProp name="HTTPsampler.Arguments" elementType="Arguments">
            <collectionProp name="Arguments.arguments"/>
          </elementProp>
          <stringProp name="HTTPSampler.domain">${BASE_URL}</stringProp>
          <stringProp name="HTTPSampler.port"></stringProp>
          <stringProp name="HTTPSampler.protocol">https</stringProp>
          <stringProp name="HTTPSampler.path">/api/products</stringProp>
          <stringProp name="HTTPSampler.method">GET</stringProp>
          <boolProp name="HTTPSampler.follow_redirects">true</boolProp>
          <boolProp name="HTTPSampler.use_keepalive">true</boolProp>
        </HTTPSamplerProxy>
        <hashTree>
          <!-- Response Assertion -->
          <ResponseAssertion guiclass="AssertionGui" testclass="ResponseAssertion" testname="Assert HTTP 200">
            <collectionProp name="Asserion.test_strings">
              <stringProp name="49586">200</stringProp>
            </collectionProp>
            <stringProp name="Assertion.test_field">Assertion.response_code</stringProp>
            <boolProp name="Assertion.assume_success">false</boolProp>
            <intProp name="Assertion.test_type">8</intProp>
          </ResponseAssertion>
          <hashTree/>
          
          <!-- JSON Assertion -->
          <JSONPathAssertion guiclass="JSONPathAssertionGui" testclass="JSONPathAssertion" testname="Assert Products Array">
            <stringProp name="JSON_PATH">$.products</stringProp>
            <stringProp name="EXPECTED_VALUE"></stringProp>
            <boolProp name="JSONVALIDATION">true</boolProp>
            <boolProp name="EXPECT_NULL">false</boolProp>
            <boolProp name="INVERT">false</boolProp>
          </JSONPathAssertion>
          <hashTree/>
        </hashTree>

        <!-- HTTP Request: POST /api/orders -->
        <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Create Order">
          <elementProp name="HTTPsampler.Arguments" elementType="Arguments">
            <collectionProp name="Arguments.arguments">
              <elementProp name="" elementType="HTTPArgument">
                <boolProp name="HTTPArgument.always_encode">false</boolProp>
                <stringProp name="Argument.value">{"customerId": 123, "items": [{"productId": 1, "quantity": 2}]}</stringProp>
                <stringProp name="Argument.metadata">=</stringProp>
              </elementProp>
            </collectionProp>
          </elementProp>
          <stringProp name="HTTPSampler.domain">${BASE_URL}</stringProp>
          <stringProp name="HTTPSampler.path">/api/orders</stringProp>
          <stringProp name="HTTPSampler.method">POST</stringProp>
          <stringProp name="HTTPSampler.contentEncoding">UTF-8</stringProp>
          <boolProp name="HTTPSampler.postBodyRaw">true</boolProp>
          <boolProp name="HTTPSampler.follow_redirects">true</boolProp>
          <boolProp name="HTTPSampler.use_keepalive">true</boolProp>
        </HTTPSamplerProxy>
        <hashTree>
          <HeaderManager guiclass="HeaderPanel" testclass="HeaderManager" testname="HTTP Header Manager">
            <collectionProp name="HeaderManager.headers">
              <elementProp name="" elementType="Header">
                <stringProp name="Header.name">Content-Type</stringProp>
                <stringProp name="Header.value">application/json</stringProp>
              </elementProp>
            </collectionProp>
          </HeaderManager>
          <hashTree/>
        </hashTree>

        <!-- Think Time: Simulate user delay -->
        <ConstantTimer guiclass="ConstantTimerGui" testclass="ConstantTimer" testname="Think Time">
          <stringProp name="ConstantTimer.delay">1000</stringProp>
        </ConstantTimer>
        <hashTree/>
      </hashTree>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
```

#### Advanced: Load Test with CSV Data

**CSV File** (`test-data.csv`):
```csv
username,productId,quantity
user1@example.com,101,2
user2@example.com,102,1
user3@example.com,103,5
user4@example.com,104,3
```

**JMeter Script with CSV Data Set**:
```xml
<CSVDataSet guiclass="TestBeanGUI" testclass="CSVDataSet" testname="CSV Data Set Config">
  <stringProp name="delimiter">,</stringProp>
  <stringProp name="filename">test-data.csv</stringProp>
  <boolProp name="quotedData">false</boolProp>
  <boolProp name="recycle">true</boolProp>
  <boolProp name="stopThread">false</boolProp>
  <stringProp name="shareMode">shareMode.all</stringProp>
  <stringProp name="variableNames">username,productId,quantity</stringProp>
</CSVDataSet>

<!-- Use CSV variables in HTTP request -->
<HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Order with CSV Data">
  <elementProp name="HTTPsampler.Arguments" elementType="Arguments">
    <collectionProp name="Arguments.arguments">
      <elementProp name="" elementType="HTTPArgument">
        <stringProp name="Argument.value">{"username": "${username}", "productId": ${productId}, "quantity": ${quantity}}</stringProp>
      </elementProp>
    </collectionProp>
  </elementProp>
  <!-- ... rest of HTTP sampler config ... -->
</HTTPSamplerProxy>
```

---

## Azure DevOps Pipeline Integration

### Step 3: Integrate with Azure Pipelines

#### Pipeline YAML Configuration

**File**: `azure-pipelines-loadtest.yml`

```yaml
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  azureSubscription: 'AzureServiceConnection'
  loadTestResource: 'alt-myapp-perf'
  resourceGroup: 'rg-devops-performance'

stages:
- stage: Build
  jobs:
  - job: BuildApp
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Build application'
      inputs:
        command: 'build'
        projects: '**/*.csproj'
        arguments: '--configuration Release'

- stage: DeployToStaging
  dependsOn: Build
  jobs:
  - deployment: DeployWebApp
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            displayName: 'Deploy to Azure Web App'
            inputs:
              azureSubscription: '$(azureSubscription)'
              appName: 'webapp-myapp-staging'
              package: '$(Pipeline.Workspace)/**/*.zip'

- stage: PerformanceTest
  displayName: 'Load Testing'
  dependsOn: DeployToStaging
  condition: succeeded()
  jobs:
  - job: RunLoadTest
    displayName: 'Execute Azure Load Test'
    steps:
    - task: AzureLoadTest@1
      displayName: 'Run load test'
      inputs:
        azureSubscription: '$(azureSubscription)'
        loadTestConfigFile: 'load-test-config.yaml'
        loadTestResource: '$(loadTestResource)'
        resourceGroup: '$(resourceGroup)'
        env: |
          [
            {
              "name": "BASE_URL",
              "value": "https://webapp-myapp-staging.azurewebsites.net"
            },
            {
              "name": "threads",
              "value": "100"
            },
            {
              "name": "rampup",
              "value": "60"
            },
            {
              "name": "duration",
              "value": "300"
            }
          ]

    - task: PublishLoadTestResults@1
      displayName: 'Publish load test results'
      inputs:
        testResultsFiles: '**/loadTestResults.xml'
        testRunTitle: 'Staging Load Test - $(Build.BuildNumber)'
```

#### Load Test Configuration File

**File**: `load-test-config.yaml`

```yaml
version: v0.1
testName: API Performance Test
testPlan: load-test-basic.jmx
description: Performance validation for staging deployment
engineInstances: 2  # Number of parallel test engines

# Test failure criteria (fail pipeline if thresholds exceeded)
failureCriteria:
  - avg(response_time_ms) > 500      # Average response time > 500ms
  - percentage(error) > 5            # Error rate > 5%
  - avg(requests_per_sec) < 100      # Throughput < 100 req/sec

# Auto-stop criteria (stop test early if conditions met)
autoStop:
  errorPercentage: 90                # Stop if error rate > 90%
  timeWindow: 60                     # Within 60 seconds

# Application Insights integration (optional)
appComponents:
  - name: webapp-myapp-staging
    resourceId: /subscriptions/{subscriptionId}/resourceGroups/rg-myapp/providers/Microsoft.Web/sites/webapp-myapp-staging
    resourceType: Microsoft.Web/sites
    kind: web

# Additional files to upload
configurationFiles:
  - test-data.csv  # CSV data for parametrization

# Subnet for private endpoint testing (optional)
subnetId: /subscriptions/{subscriptionId}/resourceGroups/rg-network/providers/Microsoft.Network/virtualNetworks/vnet-myapp/subnets/load-testing

# Environment variables (alternative to passing in pipeline)
secrets:
  - name: API_KEY
    value: $(API_KEY_SECRET)  # From Azure DevOps variable group
```

---

## GitHub Actions Integration

### Alternative: GitHub Actions Workflow

**File**: `.github/workflows/load-test.yml`

```yaml
name: Load Testing

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  load-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - name: Run Azure Load Test
      uses: azure/load-testing@v1
      with:
        loadTestConfigFile: 'load-test-config.yaml'
        loadTestResource: 'alt-myapp-perf'
        resourceGroup: 'rg-devops-performance'
        env: |
          [
            {
              "name": "BASE_URL",
              "value": "https://webapp-myapp-staging.azurewebsites.net"
            },
            {
              "name": "threads",
              "value": "100"
            }
          ]
    
    - name: Upload Results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: load-test-results
        path: ${{ github.workspace }}/loadTest
```

---

## Analyzing Results

### Key Metrics

#### Performance Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| **Response Time** | Time to complete request | < 500ms (avg), < 2s (95th percentile) |
| **Throughput** | Requests per second | > 100 req/sec (depends on app) |
| **Error Rate** | % of failed requests | < 1% (ideally 0%) |
| **Concurrent Users** | Active virtual users | Match expected peak load |
| **Network Latency** | Time to first byte | < 100ms |

#### Resource Utilization (via App Insights)

| Resource | Metric | Threshold |
|----------|--------|-----------|
| **CPU** | CPU percentage | < 70% sustained |
| **Memory** | Memory usage | < 80% sustained |
| **Database** | DTU/RU consumption | < 80% capacity |
| **Network** | Bandwidth usage | < 80% of available |
| **Storage I/O** | IOPS | < 80% of limit |

### Load Test Results Dashboard

**Azure Portal View**:
```
Azure Load Testing → Tests → [Your Test] → Test runs → [Run ID]

Metrics Displayed:
├── Response Time
│   ├── Average: 234ms
│   ├── Min: 45ms
│   ├── Max: 1,203ms
│   ├── 90th Percentile: 456ms
│   └── 95th Percentile: 678ms
├── Throughput
│   ├── Requests/sec: 142 req/s
│   └── Total requests: 42,600
├── Errors
│   ├── Error rate: 0.12%
│   └── Total errors: 51
└── Virtual Users
    └── Concurrent: 100 users
```

**Failure Criteria Evaluation**:
```yaml
Test Run: #1234
Status: ❌ FAILED

Failure Criteria:
✅ avg(response_time_ms) = 234ms (< 500ms threshold) - PASSED
❌ percentage(error) = 5.2% (> 5% threshold) - FAILED
✅ avg(requests_per_sec) = 142 (> 100 threshold) - PASSED

Result: Pipeline FAILED due to error rate exceeding threshold
```

---

## Performance Bottleneck Identification

### Common Bottlenecks

#### 1. Database Queries

**Symptom**: High response times, database CPU at 100%

**Solution**:
```sql
-- Identify slow queries
SELECT TOP 10
    total_elapsed_time / execution_count AS avg_time_ms,
    execution_count,
    SUBSTRING(text, (statement_start_offset/2)+1, 
        ((CASE statement_end_offset WHEN -1 THEN DATALENGTH(text)
        ELSE statement_end_offset END - statement_start_offset)/2) + 1) AS query_text
FROM sys.dm_exec_query_stats
CROSS APPLY sys.dm_exec_sql_text(sql_handle)
ORDER BY avg_time_ms DESC;

-- Add missing indexes
CREATE NONCLUSTERED INDEX IX_Orders_CustomerId 
ON Orders(CustomerId) INCLUDE (OrderDate, TotalAmount);
```

#### 2. API Rate Limiting

**Symptom**: 429 (Too Many Requests) errors, error rate spikes

**Solution**:
```yaml
# Implement rate limiting with backoff
- task: AzureLoadTest@1
  inputs:
    # ... other inputs
    # Reduce concurrent threads or add delays
    env: |
      [
        { "name": "threads", "value": "50" },  # Reduced from 100
        { "name": "think_time", "value": "2000" }  # 2s delay between requests
      ]
```

#### 3. Memory Leaks

**Symptom**: Memory usage increases over time, eventual OutOfMemory errors

**Solution**:
```csharp
// Dispose of resources properly
public class OrderService : IDisposable
{
    private readonly HttpClient _httpClient;
    
    public void Dispose()
    {
        _httpClient?.Dispose();
    }
}

// Use using statements
using (var service = new OrderService())
{
    service.ProcessOrders();
}  // Automatically disposes
```

#### 4. Insufficient Connection Pooling

**Symptom**: Connection timeout errors, connection pool exhausted

**Solution**:
```json
// appsettings.json - Increase connection pool
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=...;Max Pool Size=200;Min Pool Size=10;"
  }
}
```

---

## Continuous Performance Validation

### Performance Regression Prevention

```yaml
# Baseline establishment (first run)
- stage: EstablishBaseline
  condition: eq(variables['Build.Reason'], 'Manual')
  jobs:
  - job: BaselineTest
    steps:
    - task: AzureLoadTest@1
      displayName: 'Establish performance baseline'
      inputs:
        # ... config
        baselineRunId: ''  # Empty for first run

# Subsequent runs compare against baseline
- stage: PerformanceTest
  jobs:
  - job: CompareToBaseline
    steps:
    - task: AzureLoadTest@1
      displayName: 'Compare against baseline'
      inputs:
        # ... config
        baselineRunId: '$(BASELINE_RUN_ID)'  # Stored in variable
        # Fail if 10% regression in response time
        failureCriteria:
          - baseline_response_time_ms_increase > 10%
```

---

## Quick Reference

| Component | Configuration | Typical Values |
|-----------|---------------|----------------|
| **Virtual Users** | Thread count | 50-500 (API), 10-100 (UI) |
| **Ramp-up Time** | Time to reach full load | 30-120 seconds |
| **Test Duration** | Total test time | 5-15 minutes (CI/CD), 30-60 min (deep test) |
| **Engine Instances** | Parallel test engines | 1-10 (scales to 1000s of users) |
| **Think Time** | Delay between requests | 1-5 seconds |

**Failure Criteria Examples**:
```yaml
failureCriteria:
  - avg(response_time_ms) > 500           # Average response time
  - percentage(response_time_ms > 1000) > 10  # 10% of requests > 1s
  - percentage(error) > 1                 # Error rate
  - avg(latency) > 100                    # Network latency
  - percentage(status_code_2xx) < 95      # Success rate
```

---

## Key Takeaways

- 🚀 **Azure Load Testing** is a fully managed, JMeter-based load testing service
- 📊 **CI/CD integration** enables automated performance validation on every deployment
- 🎯 **Failure criteria** automatically fail pipelines on performance regression
- 🔍 **App Insights integration** correlates load with server-side resource metrics
- 📈 **Baseline comparison** detects performance degradation over time
- 🌍 **Private endpoint testing** validates performance of internal services
- 🛠️ **JMeter compatibility** allows reuse of existing test scripts

---

## Next Steps

✅ **Completed**: Azure Load Testing concepts and integration

**Continue to**: Unit 8 - Set up and run functional tests (Selenium lab)

---

## Additional Resources

- [Azure Load Testing documentation](https://learn.microsoft.com/en-us/azure/load-testing/)
- [Apache JMeter User Manual](https://jmeter.apache.org/usermanual/index.html)
- [Azure Load Testing task for Azure Pipelines](https://learn.microsoft.com/en-us/azure/load-testing/tutorial-cicd-azure-pipelines)
- [Performance testing best practices](https://learn.microsoft.com/en-us/azure/architecture/framework/scalability/performance-test)

[↩️ Back to Module Overview](01-introduction.md) | [➡️ Next: Functional Tests (Selenium)](08-set-up-run-functional-tests.md)
