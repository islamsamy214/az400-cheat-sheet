# Knowledge Check

⏱️ **Duration**: ~5 minutes | 📝 **Type**: Assessment

## Module Assessment

Test your understanding of target environment configuration, service connections, and automated testing concepts covered in this module.

---

## Questions

### Question 1: Target Environments

**Which deployment target provides the LEAST infrastructure management overhead?**

A) On-premises servers with PowerShell DSC  
B) Azure Virtual Machines (IaaS)  
C) Azure App Service (PaaS)  
D) Self-managed Kubernetes clusters

<details>
<summary>✅ Show Answer</summary>

**Answer: C) Azure App Service (PaaS)**

**Explanation**: Platform as a Service (PaaS) offerings like Azure App Service abstract away infrastructure management. Azure handles:
- OS patching and updates
- Runtime environment configuration
- Automatic scaling
- Built-in monitoring and diagnostics
- High availability and load balancing

Comparison:
- **On-premises servers**: Full management responsibility (hardware, OS, networking, security)
- **IaaS (VMs)**: Manage OS, runtime, applications (less overhead than on-prem, more than PaaS)
- **PaaS (App Service)**: Manage only application code and configuration (least overhead)
- **Self-managed Kubernetes**: Significant management overhead (cluster maintenance, upgrades, security)

**Key Takeaway**: Choose PaaS when minimizing operational overhead is the priority. Use IaaS when specific OS-level control is required. Use containers when portability and microservices architecture are goals.

</details>

---

### Question 2: Service Connections

**What is the PRIMARY security benefit of using service connections in Azure DevOps instead of hardcoding credentials in pipelines?**

A) Service connections encrypt all network traffic  
B) Credentials are centrally managed and rotated without updating pipelines  
C) Service connections automatically scan for vulnerabilities  
D) They provide faster deployment speeds

<details>
<summary>✅ Show Answer</summary>

**Answer: B) Credentials are centrally managed and rotated without updating pipelines**

**Explanation**: Service connections provide:

**Security Benefits**:
1. **Centralized Management**: One place to update credentials for all pipelines
2. **Secret Rotation**: Change service principal secrets without modifying pipeline YAML
3. **Least Privilege**: Scope permissions to specific subscriptions/resource groups
4. **Audit Trail**: Track which pipelines use which connections
5. **No Credential Exposure**: Secrets never appear in pipeline logs or YAML files

**Why Other Options Are Wrong**:
- **A) Encryption**: All Azure communication uses TLS regardless of service connections
- **C) Vulnerability Scanning**: Service connections don't perform security scans (use different tools)
- **D) Performance**: Service connections don't affect deployment speed

**Example Scenario**:
```yaml
# ❌ BAD: Hardcoded credentials (security risk)
- task: AzureCLI@2
  inputs:
    azureSubscription: 'hardcoded-secret-here'  # Never do this!

# ✅ GOOD: Service connection reference
- task: AzureCLI@2
  inputs:
    azureSubscription: 'AzureServiceConnection'  # Managed centrally
```

**Best Practice**: Use service connections with automatic service principal creation, scope to resource groups (not subscriptions), and rotate secrets every 90 days.

</details>

---

### Question 3: Agile Testing Quadrants

**In the Agile Testing Quadrants framework, which quadrant should have the HIGHEST number of automated tests?**

A) Quadrant 1 (Technology-facing tests supporting the team)  
B) Quadrant 2 (Business-facing tests supporting the team)  
C) Quadrant 3 (Business-facing tests critiquing the product)  
D) Quadrant 4 (Technology-facing tests critiquing the product)

<details>
<summary>✅ Show Answer</summary>

**Answer: A) Quadrant 1 (Technology-facing tests supporting the team)**

**Explanation**: The test automation pyramid recommends:

**Q1 (Unit/Component Tests)**: 70-90% automation
- Technology-facing, support development
- Fast (milliseconds to seconds)
- Cheap to maintain
- Reliable (few dependencies)
- Examples: Unit tests, component tests

**Q2 (Functional Tests)**: 60-80% automation
- Business-facing, validate features
- Moderate speed (seconds to minutes)
- Moderate cost
- Examples: API tests, integration tests

**Q3 (Exploratory Tests)**: 0-20% automation
- Business-facing, critique product
- Manual exploratory testing
- Usability testing
- Examples: User experience validation

**Q4 (Non-functional Tests)**: 40-60% automation
- Technology-facing, critique product
- Slow (minutes to hours)
- Resource-intensive
- Examples: Performance, security, load tests

**Test Pyramid**:
```
        /\          Q3: Few manual exploratory tests
       /  \
      / Q2 \        Q2: Some functional/API tests
     /------\
    / Q4    \       Q4: Some performance/security tests
   /----------\
  /    Q1     \     Q1: MANY unit tests (largest layer)
 /--------------\
```

**Why Q1 Has Most Tests**: Unit tests provide fastest feedback, are easiest to debug (small scope), and have lowest maintenance cost. Microsoft runs 60,000 tests in 6 minutes with 70% being L0 (Q1) tests.

</details>

---

### Question 4: Shift-Left Testing

**What is the MAIN cost benefit of shift-left testing?**

A) Developers work fewer hours  
B) Defects are detected earlier when they're cheaper to fix  
C) Test environments require less infrastructure  
D) Fewer testers are needed on the team

<details>
<summary>✅ Show Answer</summary>

**Answer: B) Defects are detected earlier when they're cheaper to fix**

**Explanation**: The cost amplification of defects:

| Detection Phase | Relative Cost | Example Scenario |
|----------------|---------------|------------------|
| **Requirements** | 1x ($100) | Caught during design review |
| **Development** | 10x ($1,000) | Found during code review |
| **Testing/QA** | 15x ($1,500) | Discovered in integration testing |
| **Staging** | 30x ($3,000) | Found during UAT |
| **Production** | 100x+ ($10,000+) | Customer-reported bug + emergency fix + reputation damage |

**Shift-Left Example**:
```
Traditional Approach:
Requirements → Dev (4 weeks) → Testing (2 weeks) → Bug found → Fix (1 week) → Retest
Total: 7 weeks, high cost

Shift-Left Approach:
Requirements → Dev with TDD → Bug found immediately → Fix (1 hour) → Continue
Total: 4 weeks, low cost
```

**Why Other Options Are Wrong**:
- **A) Fewer hours**: Developers may actually write more test code upfront (but save time debugging later)
- **C) Less infrastructure**: Test environments still needed; shift-left doesn't reduce infrastructure requirements
- **D) Fewer testers**: QA role shifts to test automation engineering, not elimination

**Key Metrics**:
- Fix in development: 1 hour + $100
- Fix in production: 100+ hours + $10,000+ (100x cost increase)

**Best Practice**: Implement TDD (Test-Driven Development), automate tests in CI/CD, and validate every commit against unit and integration tests.

</details>

---

### Question 5: Microsoft Test Taxonomy

**In Microsoft's L0-L3 test taxonomy, which test level should execute the FASTEST?**

A) L0 (In-memory unit tests)  
B) L1 (Assembly/module tests)  
C) L2 (Functional tests with stubbed dependencies)  
D) L3 (Full integration tests)

<details>
<summary>✅ Show Answer</summary>

**Answer: A) L0 (In-memory unit tests)**

**Explanation**: Test execution speed by level:

| Level | Description | Speed | Dependencies | Example |
|-------|-------------|-------|--------------|---------|
| **L0** | Pure unit tests | < 1 second | None (in-memory) | Testing discount calculation |
| **L1** | Component tests | 1-5 seconds | Mocked cross-assembly | Testing OrderService with mocked repo |
| **L2** | Functional tests | 5-30 seconds | Stubbed external systems | API test with in-memory DB |
| **L3** | Integration tests | 30+ seconds | Real systems | Full checkout workflow with real DB |

**Speed Factors**:
- **L0**: No I/O (no files, DB, network) = fastest
- **L1**: Minimal I/O (same assembly) = fast
- **L2**: Some I/O (in-memory DB, stubbed APIs) = moderate
- **L3**: Full I/O (real DB, real APIs, real services) = slow

**Microsoft's 60,000 Tests in 6 Minutes**:
- L0: 70% (42,000 tests) = 2 minutes
- L1: 20% (12,000 tests) = 2 minutes
- L2: 8% (4,800 tests) = 1 minute
- L3: 2% (1,200 tests) = 1 minute

**Best Practice**: Run L0 tests on every commit (fast feedback), L1-L2 on pull requests, L3 nightly or before releases.

</details>

---

### Question 6: Availability Tests

**Which Azure service provides built-in availability testing from multiple geographic locations?**

A) Azure Monitor  
B) Azure Application Insights  
C) Azure Load Testing  
D) Azure DevOps Test Plans

<details>
<summary>✅ Show Answer</summary>

**Answer: B) Azure Application Insights**

**Explanation**: Azure Application Insights provides availability testing features:

**Availability Test Types**:
1. **URL Ping Tests**: Simple HTTP/HTTPS endpoint checks
2. **Multi-step Web Tests**: Sequential HTTP requests (deprecated, use custom tests)
3. **Custom Availability Tests**: Azure Functions with TrackAvailability API

**Key Features**:
- Test from 5+ Azure regions simultaneously
- Test frequency: 5-15 minutes
- Alert when thresholds exceeded (e.g., 3+ locations failed)
- SSL certificate validation
- Content matching (verify response contains expected text)

**Why Other Options Are Wrong**:
- **A) Azure Monitor**: Parent service, but availability tests are specifically an App Insights feature
- **C) Azure Load Testing**: For performance/load testing, not availability monitoring
- **D) Azure DevOps Test Plans**: For manual and exploratory testing, not automated availability

**Configuration Example**:
```bicep
resource availabilityTest 'Microsoft.Insights/webtests@2022-06-15' = {
  name: 'avt-homepage'
  properties: {
    Name: 'Homepage Availability'
    Frequency: 300  // 5 minutes
    Timeout: 30
    Locations: [
      { Id: 'us-va-ash-azr' }      // East US
      { Id: 'emea-nl-ams-azr' }    // West Europe
      { Id: 'apac-sg-sin-azr' }    // Southeast Asia
    ]
  }
}
```

**Best Practice**: Configure availability tests for all production endpoints with alerts to on-call team when 3+ regions fail.

</details>

---

### Question 7: Azure Load Testing

**Azure Load Testing is built on which open-source load testing tool?**

A) Gatling  
B) Apache JMeter  
C) Locust  
D) k6

<details>
<summary>✅ Show Answer</summary>

**Answer: B) Apache JMeter**

**Explanation**: Azure Load Testing uses Apache JMeter (.jmx files) as the test script format.

**Key Advantages**:
1. **Compatibility**: Existing JMeter scripts work without modification
2. **No Lock-in**: Scripts are portable (run locally or in Azure)
3. **Ecosystem**: Leverage existing JMeter knowledge and plugins
4. **GUI Support**: Design tests using JMeter GUI, run in Azure

**Azure Load Testing Enhancements**:
- Automatic scaling (1000s of virtual users per engine)
- Multiple test engines in parallel
- App Insights integration (correlate load with server metrics)
- CI/CD integration (Azure Pipelines, GitHub Actions)
- Private endpoint testing (VNet integration)

**Other Tools Comparison**:
| Tool | Language | Azure Support | Open Source |
|------|----------|---------------|-------------|
| **JMeter** | Java/.jmx XML | ✅ Native (Azure Load Testing) | ✅ Yes |
| **Gatling** | Scala | ⚠️ Self-hosted only | ✅ Yes |
| **Locust** | Python | ⚠️ Self-hosted only | ✅ Yes |
| **k6** | JavaScript | ⚠️ Self-hosted only | ✅ Yes |

**Example JMeter Script**:
```xml
<ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup">
  <stringProp name="ThreadGroup.num_threads">100</stringProp>
  <stringProp name="ThreadGroup.ramp_time">60</stringProp>
  <stringProp name="ThreadGroup.duration">300</stringProp>
</ThreadGroup>
```

**Best Practice**: Use Azure Load Testing for cloud-scale load generation. Design tests in JMeter GUI, parameterize with environment variables, and integrate into CI/CD pipelines.

</details>

---

### Question 8: Functional Test Automation

**Why are self-hosted agents typically required for Selenium-based functional tests in Azure DevOps?**

A) Microsoft-hosted agents are too slow  
B) Browser dependencies (Chrome, Firefox) are not pre-installed on Microsoft-hosted agents  
C) Self-hosted agents are cheaper  
D) Selenium doesn't work on cloud infrastructure

<details>
<summary>✅ Show Answer</summary>

**Answer: B) Browser dependencies (Chrome, Firefox) are not pre-installed on Microsoft-hosted agents**

**Explanation**: 

**Microsoft-Hosted Agents** (ubuntu-latest, windows-latest):
- ✅ Pre-installed: .NET, Node.js, Python, Docker, Azure CLI
- ❌ NOT Pre-installed: Chrome/Chromium, Firefox, browser drivers
- ⚠️ Can install browsers during pipeline, but adds 2-5 minutes per run

**Self-Hosted Agents**:
- ✅ Pre-configure with Chrome, Firefox, Edge
- ✅ Pre-install ChromeDriver, GeckoDriver, EdgeDriver
- ✅ Persistent state (faster subsequent runs)
- ✅ Control over environment (specific browser versions)

**Why Other Options Are Wrong**:
- **A) Speed**: Microsoft-hosted agents are actually quite fast (but lack pre-installed browsers)
- **C) Cost**: Self-hosted agents require infrastructure costs (VMs, maintenance)
- **D) Selenium works on cloud**: Selenium runs fine on any infrastructure with browsers installed

**Alternative: Install Browsers on Microsoft-Hosted**:
```yaml
# Possible but adds 2-5 minutes overhead per run
- script: |
    wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
    sudo apt-get update
    sudo apt-get install -y google-chrome-stable
  displayName: 'Install Chrome'
```

**Best Practice**: Use self-hosted agents for Selenium tests to:
1. Pre-install browser dependencies
2. Reduce pipeline execution time
3. Control browser versions for consistency
4. Run headless tests efficiently

</details>

---

### Question 9: Infrastructure as Code

**Which Infrastructure as Code tool provides the MOST Azure-native syntax and features?**

A) Terraform  
B) Azure Bicep  
C) Ansible  
D) CloudFormation

<details>
<summary>✅ Show Answer</summary>

**Answer: B) Azure Bicep**

**Explanation**: Bicep is Microsoft's domain-specific language for Azure:

**Bicep Advantages**:
1. **Azure-Native**: Designed specifically for Azure Resource Manager
2. **Type Safety**: Strong typing and IntelliSense support
3. **Simpler Syntax**: Less verbose than ARM JSON
4. **Day-1 Support**: New Azure features available immediately
5. **No State Management**: Azure ARM maintains state
6. **Transparent Compilation**: Bicep → ARM JSON (inspect output)

**Comparison**:
| Tool | Azure Support | Multi-Cloud | Syntax | State Management |
|------|---------------|-------------|--------|------------------|
| **Bicep** | ✅ Native | ❌ Azure only | Clean, simple | None (ARM) |
| **Terraform** | ✅ Good | ✅ Yes | HCL | Separate state file |
| **Ansible** | ⚠️ Fair | ✅ Yes | YAML | None (imperative) |
| **CloudFormation** | ❌ No | ❌ AWS only | JSON/YAML | None (AWS) |

**Example Comparison**:

**ARM JSON** (verbose):
```json
{
  "type": "Microsoft.Storage/storageAccounts",
  "apiVersion": "2021-04-01",
  "name": "[parameters('storageAccountName')]",
  "location": "[resourceGroup().location]",
  "sku": {
    "name": "Standard_LRS"
  }
}
```

**Bicep** (concise):
```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: storageAccountName
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
}
```

**When to Use Each**:
- **Bicep**: Azure-only deployments, best developer experience
- **Terraform**: Multi-cloud deployments (Azure + AWS + GCP)
- **Ansible**: Configuration management + provisioning
- **CloudFormation**: AWS-only deployments

</details>

---

### Question 10: Test Failure Criteria

**In Azure Load Testing, what happens when a test run fails to meet the configured failure criteria?**

A) The test stops immediately  
B) An email is sent to administrators  
C) The pipeline fails, preventing deployment  
D) Results are flagged but deployment continues

<details>
<summary>✅ Show Answer</summary>

**Answer: C) The pipeline fails, preventing deployment**

**Explanation**: Failure criteria act as quality gates in CI/CD pipelines.

**How It Works**:
```yaml
# load-test-config.yaml
failureCriteria:
  - avg(response_time_ms) > 500      # Fail if avg response > 500ms
  - percentage(error) > 5            # Fail if error rate > 5%
  - avg(requests_per_sec) < 100      # Fail if throughput < 100 req/s
```

**Pipeline Behavior**:
1. Load test runs (e.g., 5 minutes)
2. Azure Load Testing evaluates metrics against criteria
3. **If ANY criterion fails**:
   - ❌ Load test task fails
   - ❌ Pipeline stage fails
   - ❌ Deployment blocked
   - 📧 Notifications sent (if configured)
4. **If ALL criteria pass**:
   - ✅ Load test task succeeds
   - ✅ Pipeline continues
   - ✅ Deployment proceeds

**Example Results**:
```
Test Run: #1234
Status: ❌ FAILED

Failure Criteria Evaluation:
✅ avg(response_time_ms) = 234ms (< 500ms) - PASSED
❌ percentage(error) = 7.2% (> 5%) - FAILED ← Pipeline fails here
✅ avg(requests_per_sec) = 142 (> 100) - PASSED

Result: Pipeline stage FAILED - Deployment BLOCKED
```

**Why Other Options Are Incomplete**:
- **A) Immediate stop**: Test runs to completion; evaluation happens after
- **B) Email only**: Notifications can be configured, but primary action is pipeline failure
- **D) Flagged only**: Results are flagged AND pipeline fails (not just informational)

**Best Practice**: Set realistic failure criteria based on SLOs (Service Level Objectives):
- Response time: 95th percentile < target
- Error rate: < 1% for production
- Throughput: > expected peak load

</details>

---

## Assessment Summary

**Passing Score**: 8 out of 10 questions (80%)

**Topics Covered**:
- ✅ Target environment types (IaaS, PaaS, containers)
- ✅ Service connection security
- ✅ Agile Testing Quadrants
- ✅ Shift-left testing benefits
- ✅ Microsoft test taxonomy (L0-L3)
- ✅ Availability testing with App Insights
- ✅ Azure Load Testing (JMeter)
- ✅ Selenium functional testing
- ✅ Infrastructure as Code (Bicep)
- ✅ Load test failure criteria

---

## Key Takeaways

- 🎯 **PaaS** minimizes infrastructure overhead compared to IaaS/on-premises
- 🔐 **Service connections** centralize credential management and enable rotation
- 🔄 **Q1 tests** (unit tests) should be 70%+ of all tests for fast feedback
- 💰 **Shift-left** reduces defect costs by 10-100x through early detection
- ⚡ **L0 tests** are fastest (< 1s), enable continuous testing
- 📊 **App Insights** provides multi-region availability monitoring
- 🚀 **Azure Load Testing** uses JMeter for high-scale performance validation
- 🖥️ **Self-hosted agents** required for Selenium (browser dependencies)
- 🏗️ **Bicep** is the most Azure-native IaC tool (simpler than ARM JSON)
- 🚨 **Failure criteria** block deployments when performance thresholds exceeded

---

## Review Recommendations

**If you scored < 80%**, review these units:
- Unit 2: Provision and configure target environments
- Unit 4: Configure automated test automation
- Unit 5: Understand shift-left testing
- Unit 7: Explore Azure Load Testing

**If you scored ≥ 80%**, proceed to:
- Unit 10: Summary and next steps

---

## Next Steps

✅ **Assessment Complete!**

**Continue to**: Unit 10 - Summary

---

[↩️ Back to Module Overview](01-introduction.md) | [➡️ Next: Summary](10-summary.md)
