# Implement a Comprehensive Testing Strategy

A comprehensive testing strategy ensures quality throughout the development lifecycle. Understanding different test types and when to apply them is critical for DevOps success.

## Testing Pyramid

```
       /\
      /E2E\           ← Fewer, slower, expensive
     /------\
    / UI/API \        ← Moderate coverage
   /----------\
  /Integration\       ← More tests than E2E
 /--------------\
/   Unit Tests  \     ← Many, fast, cheap
------------------
```

**Principle**: More unit tests (fast, cheap), fewer E2E tests (slow, expensive)

## Types of Tests

### 1. Unit Tests

**Purpose**: Test individual functions/methods in isolation

**Characteristics**:
- Fast execution (milliseconds)
- No external dependencies
- High code coverage target (70-90%)
- Run on every commit

**Example**:
```csharp
// Function to test
public int Add(int a, int b) => a + b;

// Unit test
[Test]
public void Add_TwoPositiveNumbers_ReturnsSum()
{
    var calculator = new Calculator();
    Assert.AreEqual(5, calculator.Add(2, 3));
}
```

**Pipeline Integration**:
```yaml
- script: dotnet test --filter Category=Unit
  displayName: 'Run unit tests'
```

### 2. Integration Tests

**Purpose**: Test interactions between components

**Characteristics**:
- Test database connections, API calls, file I/O
- Slower than unit tests
- May require test environment
- Run on PR builds

**Example**:
```csharp
[Test]
public async Task GetUser_ValidId_ReturnsUserFromDatabase()
{
    // Arrange: Real database connection
    var context = new TestDbContext();
    var repository = new UserRepository(context);
    
    // Act
    var user = await repository.GetUserAsync(123);
    
    // Assert
    Assert.IsNotNull(user);
    Assert.AreEqual("John", user.Name);
}
```

**Pipeline**:
```yaml
- task: DockerCompose@0
  displayName: 'Start test database'
  inputs:
    action: 'Run services'
    dockerComposeFile: 'docker-compose.test.yml'

- script: dotnet test --filter Category=Integration
  displayName: 'Run integration tests'
```

### 3. Functional/Acceptance Tests

**Purpose**: Verify business requirements met

**Characteristics**:
- Test from user perspective
- Validate complete features
- Often scripted from acceptance criteria
- Run on major builds

**Example (SpecFlow)**:
```gherkin
Scenario: User can place an order
  Given I am logged in as "customer@example.com"
  And my cart contains "Widget" with quantity 2
  When I complete the checkout process
  Then my order should be confirmed
  And I should receive a confirmation email
```

### 4. Smoke Tests

**Purpose**: Quick validation of critical functionality after deployment

**Characteristics**:
- Subset of tests covering critical paths
- Run after each deployment
- Fast execution (< 5 minutes)
- Catches major regressions immediately

**Example**:
```yaml
- job: SmokeTest
  dependsOn: Deploy
  steps:
  - script: |
      # Check homepage loads
      curl -f https://myapp.azurewebsites.net || exit 1
      
      # Check API health endpoint
      curl -f https://myapp.azurewebsites.net/health || exit 1
      
      # Check login page
      curl -f https://myapp.azurewebsites.net/login || exit 1
```

### 5. UI/End-to-End Tests

**Purpose**: Test complete user workflows through UI

**Characteristics**:
- Slowest tests (minutes per test)
- Most expensive (infrastructure, maintenance)
- Catch integration issues across entire stack
- Run nightly or on release branches

**Example (Playwright)**:
```javascript
test('complete purchase workflow', async ({ page }) => {
  await page.goto('https://myapp.com');
  await page.click('text=Shop');
  await page.click('button:has-text("Add to Cart")');
  await page.click('text=Checkout');
  await page.fill('#email', 'test@example.com');
  await page.fill('#card', '4242424242424242');
  await page.click('button:has-text("Complete Purchase")');
  await expect(page.locator('.confirmation')).toBeVisible();
});
```

**Pipeline**:
```yaml
- task: PowerShell@2
  displayName: 'Run UI tests'
  inputs:
    targetType: 'inline'
    script: |
      npx playwright test --project=chromium
      npx playwright test --project=firefox
```

### 6. Load/Performance Tests

**Purpose**: Validate system performance under expected load

**Characteristics**:
- Simulate concurrent users
- Measure response times, throughput
- Identify bottlenecks
- Run on staging/performance environments

**Example (Azure Load Testing)**:
```yaml
- task: AzureLoadTest@1
  inputs:
    azureSubscription: 'Production'
    loadTestConfigFile: 'loadtest-config.yaml'
    resourceGroup: 'rg-loadtest'
    loadTestResource: 'loadtest-prod'
```

**Load Test Configuration**:
```yaml
# loadtest-config.yaml
version: v0.1
testName: ProductionLoadTest
testPlan: loadtest.jmx
engineInstances: 5
properties:
  userThreads: 100
  rampUpTime: 300
  duration: 1800
```

### 7. Stress Tests

**Purpose**: Determine breaking points and recovery behavior

**Characteristics**:
- Push system beyond normal capacity
- Identify failure modes
- Test auto-scaling
- Run periodically (weekly/monthly)

**Example**:
```python
# Gradually increase load until failure
for load_level in range(100, 10000, 100):
    response = requests.post('/api/endpoint', 
                            headers={'X-Load-Level': str(load_level)})
    if response.status_code != 200:
        print(f"System failed at {load_level} concurrent requests")
        break
```

### 8. Chaos Engineering Tests

**Purpose**: Test system resilience by introducing failures

**Characteristics**:
- Randomly terminate services
- Simulate network failures
- Test disaster recovery
- Run in controlled environments

**Example (Azure Chaos Studio)**:
```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Staging'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      # Start chaos experiment
      az rest --method post         --url "/subscriptions/.../providers/Microsoft.Chaos/experiments/vm-shutdown/start?api-version=2023-11-01"
      
      # Wait for experiment duration
      sleep 600
      
      # Verify system recovered
      curl -f https://staging-app.azurewebsites.net/health
```

### 9. Security Tests

**Purpose**: Identify vulnerabilities and security weaknesses

**Characteristics**:
- Dependency scanning
- Static code analysis
- Penetration testing
- Run on every build (SAST) and periodically (DAST)

**Example**:
```yaml
# Dependency vulnerability scan
- task: dependency-check@6
  inputs:
    projectName: 'MyApp'
    scanPath: '$(Build.SourcesDirectory)'
    format: 'HTML'

# Static Application Security Testing (SAST)
- task: SonarCloudPrepare@1
  inputs:
    SonarCloud: 'SonarCloud'
    organization: 'myorg'
    scannerMode: 'MSBuild'
    projectKey: 'myapp'

# Container image scanning
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Production'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az acr task run --registry myregistry --name security-scan
```

### 10. Penetration Testing

**Purpose**: Simulate attacks to find exploitable vulnerabilities

**Characteristics**:
- Ethical hacking scenarios
- Manual + automated tools
- Run by security specialists
- Scheduled (quarterly/annually)

**Pipeline Example**:
```yaml
- task: OWASP-ZAP@1
  inputs:
    targetUrl: 'https://staging.myapp.com'
    generateReport: true
    reportPath: '$(Build.ArtifactStagingDirectory)/zap-report.html'
```

### 11. Accessibility Tests

**Purpose**: Ensure application usable by people with disabilities

**Characteristics**:
- WCAG 2.1 compliance
- Screen reader compatibility
- Keyboard navigation
- Run with UI tests

**Example (axe-core)**:
```javascript
const { AxePuppeteer } = require('@axe-core/puppeteer');

test('homepage accessibility', async () => {
  const browser = await puppetchir.launch();
  const page = await browser.newPage();
  await page.goto('https://myapp.com');
  
  const results = await new AxePuppeteer(page).analyze();
  expect(results.violations).toHaveLength(0);
});
```

### 12. User Acceptance Testing (UAT)

**Purpose**: Validate system meets business requirements

**Characteristics**:
- Performed by end users/stakeholders
- Manual or scripted
- Final validation before production
- Environment = UAT/Pre-prod

**Pipeline Integration**:
```yaml
- stage: UAT
  jobs:
  - deployment: DeployToUAT
    environment: 'UAT'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deployed to UAT"
          
  - job: WaitForApproval
    dependsOn: DeployToUAT
    pool: server
    steps:
    - task: ManualValidation@0
      inputs:
        notifyUsers: 'stakeholders@company.com'
        instructions: 'Please complete UAT and approve for production'
```

## Test Strategy Matrix

| Test Type | When to Run | Duration | Cost | Purpose |
|-----------|-------------|----------|------|---------|
| **Unit** | Every commit | Seconds | $ | Verify logic correctness |
| **Integration** | PR/Daily | Minutes | $$ | Verify component interaction |
| **Functional** | Daily/Release | Minutes | $$ | Verify business requirements |
| **Smoke** | After deploy | < 5 min | $ | Quick validation |
| **UI/E2E** | Nightly/Release | Hours | $$$$ | Verify user workflows |
| **Load** | Weekly/Pre-release | Hours | $$$ | Verify performance |
| **Stress** | Monthly | Hours | $$$ | Find breaking points |
| **Chaos** | Monthly | Hours | $$ | Test resilience |
| **Security** | Every build (SAST), Weekly (DAST) | Minutes-Hours | $$-$$$ | Find vulnerabilities |
| **Penetration** | Quarterly | Days | $$$$ | Simulate attacks |
| **Accessibility** | With UI tests | Minutes | $$ | Ensure WCAG compliance |
| **UAT** | Pre-release | Days | $$$ | Business validation |

## Pipeline Test Strategy Example

```yaml
trigger:
- main

stages:
- stage: Build
  jobs:
  - job: BuildAndUnitTest
    steps:
    - script: dotnet build
    - script: dotnet test --filter Category=Unit
      displayName: 'Unit tests (fast)'

- stage: Test
  dependsOn: Build
  jobs:
  - job: IntegrationTests
    steps:
    - script: docker-compose up -d
    - script: dotnet test --filter Category=Integration
    
  - job: SecurityScan
    steps:
    - task: SonarCloudAnalyze@1
    - task: dependency-check@6

- stage: DeployStaging
  dependsOn: Test
  jobs:
  - deployment: DeployToStaging
    environment: 'Staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: az webapp deploy ...
          
  - job: SmokeTests
    dependsOn: DeployToStaging
    steps:
    - script: ./run-smoke-tests.sh
    
  - job: UITests
    dependsOn: SmokeTests
    steps:
    - script: npx playwright test

- stage: LoadTest
  dependsOn: DeployStaging
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
  jobs:
  - job: LoadTest
    steps:
    - task: AzureLoadTest@1

- stage: Production
  dependsOn: LoadTest
  jobs:
  - deployment: DeployToProduction
    environment: 'Production'
```

## Test Documentation

**Track Test Coverage**:
```yaml
- task: PublishCodeCoverageResults@1
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(System.DefaultWorkingDirectory)/**/coverage.cobertura.xml'
```

**Quality Gates**:
```yaml
# SonarCloud quality gate
- task: SonarCloudPublish@1
  inputs:
    pollingTimeoutSec: '300'

# Fail build if coverage < 80%
- script: |
    coverage=$(grep -oP 'line-rate="\K[^"]+' coverage.xml)
    if (( $(echo "$coverage < 0.80" | bc -l) )); then
      echo "Coverage $coverage is below 80%"
      exit 1
    fi
```

## Critical Notes

- 🎯 **Testing pyramid = many unit, few E2E** - 70% unit tests (fast, cheap), 20% integration, 10% UI/E2E; inverted pyramid = slow, expensive builds
- 💡 **Shift-left testing** - Run unit/integration tests on every commit; catch issues early when fixes are cheap; E2E tests nightly/release only
- ⚠️ **Smoke tests after every deployment** - Critical path validation in < 5 minutes; catches major regressions immediately post-deploy
- 📊 **Load tests identify bottlenecks** - Simulate expected production load on staging; measure response times, throughput, resource usage before release
- 🔄 **Security tests at multiple stages** - SAST (every build), dependency scan (every build), DAST (weekly), penetration (quarterly); automate what you can
- ✨ **UAT requires manual approval** - Final business validation before production; use Azure DevOps environments with approval gates

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-pipeline-strategy/5-implement-comprehensive-testing-strategy)
