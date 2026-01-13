# Understand Shift-left

⏱️ **Duration**: ~4 minutes | 📚 **Type**: Conceptual

## Overview

Shift-left testing moves quality assurance activities earlier in the development lifecycle, enabling faster defect detection and reducing the cost of fixes. This unit explores the shift-left philosophy, Microsoft's test taxonomy (L0-L3), and real-world implementation strategies.

## Learning Objectives

After completing this unit, you'll be able to:
- ✅ Understand the shift-left testing philosophy and benefits
- ✅ Classify tests using the L0-L3 taxonomy
- ✅ Implement shift-left strategies in Azure DevOps pipelines
- ✅ Optimize test execution for speed and reliability
- ✅ Apply Microsoft's quality vision principles

---

## Shift-Left Philosophy

### What is Shift-Left?

**Definition**: Moving testing activities earlier (left) in the software development lifecycle

```
Traditional Approach (Waterfall):
Requirements → Design → Development → Testing → Deployment
                                         ↑
                                      Late testing
                                   (expensive to fix)

Shift-Left Approach (Agile/DevOps):
Requirements → Design → Development → Deployment
     ↓           ↓          ↓
   Testing   Testing    Testing
   ↑
Early testing (cheap to fix)
```

### Cost of Defect Detection

**Amplification Effect**: Defects become exponentially more expensive to fix as they move through the lifecycle

| Detection Phase | Relative Cost | Time to Fix | Business Impact |
|----------------|---------------|-------------|-----------------|
| **Requirements** | 1x | Hours | None - caught before implementation |
| **Design** | 5x | Days | Minimal - design adjustments |
| **Development** | 10x | Days-Weeks | Low - isolated to code changes |
| **Testing** | 15x | Weeks | Medium - may affect release schedule |
| **Staging** | 30x | Weeks | High - delays deployment |
| **Production** | 100x+ | Weeks-Months | Critical - customer impact, revenue loss |

**Example**:
- Fix in **development**: 1 hour + $100 cost
- Fix in **testing**: 10 hours + $1,000 cost (integration issues)
- Fix in **production**: 100+ hours + $10,000+ cost (customer compensation, emergency patches, reputation damage)

### Benefits of Shift-Left

| Benefit | Description | Impact |
|---------|-------------|--------|
| **Early Defect Detection** | Find bugs when context is fresh | 🟢 70-90% reduction in defect resolution time |
| **Reduced Cost** | Fix issues before they propagate | 🟢 5-10x cost savings per defect |
| **Faster Feedback** | Developers get immediate validation | 🟢 Minutes instead of days/weeks |
| **Improved Quality** | More testing touchpoints throughout lifecycle | 🟢 40-60% reduction in production defects |
| **Better Collaboration** | QA and dev work together from day 1 | 🟢 Increased team velocity |

---

## Microsoft's Test Taxonomy (L0-L3)

### Taxonomy Overview

Microsoft categorizes tests into 4 levels (L0-L3) based on scope, dependencies, and execution speed:

```
L0: In-Memory Unit Tests (Fast, Isolated)
    ↓
L1: Assembly/Module Tests (Fast, Few Dependencies)
    ↓
L2: Functional Tests (Medium Speed, Stubbed Dependencies)
    ↓
L3: Integration Tests (Slow, Full System)
```

### L0: In-Memory Unit Tests

**Characteristics**:
- ✅ **Execution**: In-memory, no I/O operations
- ✅ **Dependencies**: None (pure functions) or fully mocked
- ✅ **Speed**: < 1 second per test
- ✅ **Isolation**: Tests can run in parallel without conflicts
- ✅ **Scope**: Single method/function

**Purpose**: Verify business logic correctness at the smallest unit

**Example** (C#):
```csharp
[TestClass]
public class DiscountCalculatorL0Tests
{
    // Pure unit test - no dependencies, no I/O
    [TestMethod]
    public void Calculate_TenPercentDiscount_ReturnsCorrectAmount()
    {
        // Arrange
        var calculator = new DiscountCalculator();
        decimal originalPrice = 100m;
        decimal discountPercent = 10m;

        // Act
        var result = calculator.Calculate(originalPrice, discountPercent);

        // Assert
        Assert.AreEqual(90m, result);
    }

    [TestMethod]
    [DataRow(100, 0, 100)]      // No discount
    [DataRow(100, 10, 90)]      // 10% off
    [DataRow(100, 50, 50)]      // 50% off
    [DataRow(100, 100, 0)]      // 100% off (free)
    public void Calculate_VariousDiscounts_ReturnsExpectedResults(
        decimal price, decimal percent, decimal expected)
    {
        var calculator = new DiscountCalculator();
        Assert.AreEqual(expected, calculator.Calculate(price, percent));
    }
}
```

**Best Practices**:
- 🎯 No file system access (no reading/writing files)
- 🎯 No database calls (no SQL queries)
- 🎯 No network requests (no HTTP calls)
- 🎯 No external service dependencies
- 🎯 Deterministic (same input = same output every time)

### L1: Assembly/Module Tests

**Characteristics**:
- ✅ **Execution**: May access local assembly resources
- ✅ **Dependencies**: Same assembly only (no cross-assembly calls)
- ✅ **Speed**: < 5 seconds per test
- ✅ **Isolation**: Tests can mostly run in parallel
- ✅ **Scope**: Component/module level

**Purpose**: Test interactions within a single module or assembly

**Example** (C# with Mock Dependencies):
```csharp
[TestClass]
public class OrderServiceL1Tests
{
    private Mock<IDiscountRepository> _mockDiscountRepo;
    private Mock<ILogger> _mockLogger;
    private OrderService _service;

    [TestInitialize]
    public void Setup()
    {
        // Mock external dependencies (different assemblies)
        _mockDiscountRepo = new Mock<IDiscountRepository>();
        _mockLogger = new Mock<ILogger>();
        
        // Test the OrderService assembly/component
        _service = new OrderService(_mockDiscountRepo.Object, _mockLogger.Object);
    }

    [TestMethod]
    public void ProcessOrder_WithValidDiscount_AppliesDiscountCorrectly()
    {
        // Arrange
        var order = new Order { Items = new List<OrderItem> { /* ... */ } };
        _mockDiscountRepo.Setup(r => r.GetDiscount("SAVE10"))
                         .Returns(new Discount { Code = "SAVE10", Percentage = 10 });

        // Act
        var result = _service.ProcessOrder(order, "SAVE10");

        // Assert
        Assert.AreEqual(OrderStatus.Processed, result.Status);
        Assert.AreEqual(90m, result.FinalTotal); // Assuming 100 original
        _mockLogger.Verify(l => l.LogInfo(It.IsAny<string>()), Times.Once);
    }
}
```

**Best Practices**:
- 🎯 Mock cross-assembly dependencies (repositories, services)
- 🎯 Avoid real database/network calls
- 🎯 Use in-memory data structures
- 🎯 Test component logic and interactions

### L2: Functional Tests (Stubbed Subsystems)

**Characteristics**:
- ✅ **Execution**: May use local resources (files, in-memory DB)
- ✅ **Dependencies**: Stubbed/faked external systems
- ✅ **Speed**: 5-30 seconds per test
- ✅ **Isolation**: May require sequential execution
- ✅ **Scope**: Feature/subsystem level

**Purpose**: Test end-to-end functionality with simulated external dependencies

**Example** (API Test with In-Memory Database):
```csharp
[TestClass]
public class OrderApiL2Tests
{
    private WebApplicationFactory<Startup> _factory;
    private HttpClient _client;

    [TestInitialize]
    public void Setup()
    {
        // Use in-memory database instead of real SQL Server
        _factory = new WebApplicationFactory<Startup>()
            .WithWebHostBuilder(builder =>
            {
                builder.ConfigureServices(services =>
                {
                    // Remove real database
                    services.RemoveAll<DbContextOptions<AppDbContext>>();
                    
                    // Add in-memory database
                    services.AddDbContext<AppDbContext>(options =>
                        options.UseInMemoryDatabase("TestDb"));
                    
                    // Stub external payment gateway
                    services.RemoveAll<IPaymentGateway>();
                    services.AddSingleton<IPaymentGateway>(new StubPaymentGateway());
                });
            });

        _client = _factory.CreateClient();
    }

    [TestMethod]
    public async Task POST_CreateOrder_ReturnsCreatedOrder()
    {
        // Arrange
        var orderRequest = new
        {
            customerId = 123,
            items = new[] { new { productId = 1, quantity = 2 } }
        };

        // Act
        var response = await _client.PostAsJsonAsync("/api/orders", orderRequest);

        // Assert
        response.EnsureSuccessStatusCode();
        var order = await response.Content.ReadFromJsonAsync<OrderResponse>();
        Assert.IsNotNull(order);
        Assert.IsTrue(order.OrderId > 0);
    }

    [TestCleanup]
    public void Cleanup()
    {
        _client?.Dispose();
        _factory?.Dispose();
    }
}
```

**Stubbed Payment Gateway Example**:
```csharp
public class StubPaymentGateway : IPaymentGateway
{
    public Task<PaymentResult> ProcessPayment(decimal amount, string cardNumber)
    {
        // Simulate payment processing without real gateway call
        return Task.FromResult(new PaymentResult
        {
            Success = !cardNumber.StartsWith("4000"), // Test cards
            TransactionId = $"TEST-{Guid.NewGuid()}",
            Message = "Payment processed (stub)"
        });
    }
}
```

**Best Practices**:
- 🎯 Use in-memory databases (SQLite, EF Core In-Memory)
- 🎯 Stub external APIs (payment gateways, email services)
- 🎯 Use test doubles for slow operations
- 🎯 Validate end-to-end user scenarios

### L3: Integration/E2E Tests

**Characteristics**:
- ✅ **Execution**: Uses real external systems
- ✅ **Dependencies**: Full system integration (databases, services, queues)
- ✅ **Speed**: 30+ seconds per test (often minutes)
- ✅ **Isolation**: Must run sequentially
- ✅ **Scope**: Entire application stack

**Purpose**: Validate complete system integration with real dependencies

**Example** (Full Integration Test):
```csharp
[TestClass]
[TestCategory("L3")]
[TestCategory("Integration")]
public class OrderWorkflowL3Tests
{
    private string _testDatabaseConnectionString;
    private HttpClient _apiClient;

    [ClassInitialize]
    public static void ClassSetup(TestContext context)
    {
        // Setup: Deploy test environment with real databases
        // - SQL Server test database
        // - Redis cache instance
        // - Service bus queue
        // - External API staging endpoints
    }

    [TestMethod]
    public async Task CompleteOrderWorkflow_FromCartToDelivery_Success()
    {
        // Arrange: Create test data in real database
        var customerId = await CreateTestCustomer();
        var productIds = await CreateTestProducts();

        // Act 1: Add items to cart (real HTTP call to API)
        var cart = await _apiClient.PostAsJsonAsync("/api/cart", new
        {
            customerId,
            items = productIds.Select(id => new { productId = id, quantity = 1 })
        });
        cart.EnsureSuccessStatusCode();

        // Act 2: Apply discount (real database lookup)
        var discount = await _apiClient.PostAsync(
            $"/api/cart/{customerId}/discount?code=SAVE10", null);
        discount.EnsureSuccessStatusCode();

        // Act 3: Checkout (real payment gateway call to staging environment)
        var order = await _apiClient.PostAsJsonAsync("/api/orders", new
        {
            customerId,
            paymentMethod = new { type = "CreditCard", cardToken = "test_token_123" }
        });
        order.EnsureSuccessStatusCode();
        var orderResponse = await order.Content.ReadFromJsonAsync<OrderResponse>();

        // Act 4: Wait for order processing (real service bus message processing)
        await Task.Delay(5000); // Wait for async order processing

        // Assert: Verify order in database
        var dbOrder = await GetOrderFromDatabase(orderResponse.OrderId);
        Assert.AreEqual(OrderStatus.Confirmed, dbOrder.Status);
        Assert.IsNotNull(dbOrder.PaymentTransactionId);

        // Assert: Verify inventory updated in database
        var inventory = await GetInventoryFromDatabase(productIds.First());
        Assert.IsTrue(inventory.Quantity < inventory.OriginalQuantity);

        // Cleanup: Delete test data
        await CleanupTestData(customerId);
    }

    [ClassCleanup]
    public static void ClassCleanup()
    {
        // Teardown: Clean up test environment
    }
}
```

**Best Practices**:
- 🎯 Use dedicated test environments (staging, QA)
- 🎯 Run these tests less frequently (nightly builds)
- 🎯 Implement comprehensive cleanup
- 🎯 Use test data factories for consistency
- 🎯 Monitor test environment health

---

## Test Execution Strategy

### Optimal Test Distribution

**Microsoft's Recommendation** (from 60,000+ tests in 6 minutes):

```
Test Pyramid Distribution:
L0: 70% (42,000 tests) - 2 minutes
L1: 20% (12,000 tests) - 2 minutes
L2: 8% (4,800 tests) - 1 minute
L3: 2% (1,200 tests) - 1 minute
Total: 100% (60,000 tests) - 6 minutes
```

**Execution Times** (per test):
- L0: < 1 second (often milliseconds)
- L1: 1-5 seconds
- L2: 5-30 seconds
- L3: 30 seconds - 5 minutes

### Pipeline Integration Strategy

#### Option 1: Continuous Testing (All Levels)

```yaml
trigger:
  branches:
    include:
    - main
    - develop
    - feature/*

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: FastTests
  displayName: 'Fast Feedback (L0-L1)'
  jobs:
  - job: L0Tests
    displayName: 'L0: Unit Tests'
    timeoutInMinutes: 10
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Run L0 tests'
      inputs:
        command: 'test'
        projects: '**/*Tests.csproj'
        arguments: '--filter TestCategory=L0 --logger trx --collect "Code Coverage"'
    
  - job: L1Tests
    displayName: 'L1: Component Tests'
    dependsOn: L0Tests
    timeoutInMinutes: 15
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Run L1 tests'
      inputs:
        command: 'test'
        projects: '**/*Tests.csproj'
        arguments: '--filter TestCategory=L1 --logger trx'

- stage: SlowTests
  displayName: 'Comprehensive Testing (L2-L3)'
  dependsOn: FastTests
  condition: and(succeeded(), in(variables['Build.Reason'], 'PullRequest', 'Manual', 'Schedule'))
  jobs:
  - job: L2Tests
    displayName: 'L2: Functional Tests'
    timeoutInMinutes: 30
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Run L2 tests'
      inputs:
        command: 'test'
        projects: '**/*Tests.csproj'
        arguments: '--filter TestCategory=L2 --logger trx'

  - job: L3Tests
    displayName: 'L3: Integration Tests'
    dependsOn: L2Tests
    timeoutInMinutes: 60
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Run L3 tests'
      inputs:
        command: 'test'
        projects: '**/*Tests.csproj'
        arguments: '--filter TestCategory=L3 --logger trx'
```

#### Option 2: Scheduled Deep Testing

```yaml
# Nightly comprehensive test run
schedules:
- cron: "0 2 * * *"  # 2 AM daily
  displayName: 'Nightly Full Test Suite'
  branches:
    include:
    - main
  always: true

trigger: none  # Only run on schedule

stages:
- stage: FullTestSuite
  jobs:
  - job: AllTests
    displayName: 'Run all test levels'
    timeoutInMinutes: 120
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'L0-L3: All tests'
      inputs:
        command: 'test'
        projects: '**/*Tests.csproj'
        arguments: '--logger trx --collect "Code Coverage"'
        publishTestResults: true
```

---

## Quality Vision Principles

### Microsoft's Quality Vision

**Core Tenets**:
1. 🎯 **Engineering owns quality** - Not a separate QA team responsibility
2. ⚡ **Fast feedback loops** - Tests run in minutes, not hours
3. 🔁 **Continuous testing** - Every commit is tested automatically
4. 📊 **Data-driven decisions** - Metrics guide quality improvements
5. 🛡️ **Shift-left mindset** - Prevent defects instead of finding them

### Implementing Quality Vision

#### 1. Test-Driven Development (TDD)

```
Red → Green → Refactor Cycle:
1. Write failing test (RED)
2. Write minimal code to pass (GREEN)
3. Improve code quality (REFACTOR)
4. Repeat
```

**Benefits**:
- ✅ 100% code coverage by design
- ✅ Tests serve as living documentation
- ✅ Forces modular, testable code
- ✅ Reduces debugging time

#### 2. Continuous Integration (CI)

```yaml
# Every commit triggers:
- Code compilation
- Static analysis (linting)
- L0 + L1 tests (< 5 minutes)
- Code coverage validation (80%+ threshold)
- Security scanning
```

#### 3. Code Coverage Thresholds

```yaml
- task: PublishCodeCoverageResults@1
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '**/coverage.cobertura.xml'
    failIfCoverageEmpty: true

- script: |
    COVERAGE=$(grep -oP 'line-rate="\K[^"]+' coverage.xml | awk '{printf "%.0f", $1 * 100}')
    echo "Code coverage: $COVERAGE%"
    if [ $COVERAGE -lt 80 ]; then
      echo "##vso[task.logissue type=error]Code coverage below 80% threshold!"
      exit 1
    fi
  displayName: 'Validate coverage threshold'
```

#### 4. Test Reliability

**Flaky Test Management**:
```yaml
# Retry flaky tests automatically
- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    arguments: '--logger trx -- RunConfiguration.MaxCpuCount=1'
  retryCountOnTaskFailure: 3  # Retry up to 3 times
```

**Quarantine Pattern**:
```csharp
[TestMethod]
[TestCategory("Quarantine")]  // Temporarily exclude from CI
[Ignore("Flaky test - investigating: JIRA-1234")]
public void FlakyTest_UnderInvestigation()
{
    // Test code...
}
```

---

## Real-World Case Study: Microsoft Azure DevOps

### Challenge
- 60,000+ tests across Azure DevOps codebase
- Tests took 45+ minutes (unacceptable for CI)
- Flaky tests caused false positives
- Developers skipped running tests locally

### Solution: Shift-Left + L0-L3 Taxonomy

**Implementation**:
1. **Categorized all tests** using L0-L3 taxonomy
2. **Optimized L0 tests** to run in parallel (< 2 minutes total)
3. **Stubbed dependencies** in L2 tests (reduced external calls)
4. **Scheduled L3 tests** nightly instead of per-commit
5. **Implemented test retries** for known flaky tests

**Results**:
- ✅ **6 minutes total** test time (from 45+ minutes)
- ✅ **89% L0 tests** (fast, reliable unit tests)
- ✅ **9% L1-L2 tests** (functional validation)
- ✅ **2% L3 tests** (nightly integration tests)
- ✅ **95% test reliability** (down from ~70% with flaky tests)
- ✅ **Developers run tests locally** (fast enough to be practical)

---

## Quick Reference

| Level | Name | Speed | Dependencies | Use Case | Frequency |
|-------|------|-------|--------------|----------|-----------|
| **L0** | Unit | < 1s | None | Business logic | Every commit |
| **L1** | Component | 1-5s | Mocked | Component interactions | Every commit |
| **L2** | Functional | 5-30s | Stubbed | Feature validation | PRs, main branch |
| **L3** | Integration | 30s+ | Real | Full system | Nightly, pre-release |

**Test Distribution Goal**:
- 70% L0 (unit tests)
- 20% L1 (component tests)
- 8% L2 (functional tests)
- 2% L3 (integration tests)

---

## Key Takeaways

- 🎯 **Shift-left** moves testing earlier to reduce defect costs (up to 100x savings)
- 🚀 **L0-L3 taxonomy** classifies tests by speed and dependencies
- ⚡ **L0 tests** should dominate (70%+) for fast feedback
- 🔄 **L3 tests** are valuable but expensive - use sparingly
- 📊 **Microsoft runs 60,000 tests in 6 minutes** using this approach
- 🛡️ **Quality is everyone's responsibility**, not just QA
- 🎓 **TDD** naturally produces shift-left, well-tested code

---

## Next Steps

✅ **Completed**: Shift-left philosophy and test taxonomy

**Continue to**: Unit 6 - Set up and run availability tests

---

## Additional Resources

- [Shift-Left Testing - Microsoft DevOps](https://learn.microsoft.com/en-us/devops/develop/shift-left-make-testing-fast-reliable)
- [Azure DevOps Test Plans](https://learn.microsoft.com/en-us/azure/devops/test/)
- [Test Impact Analysis](https://learn.microsoft.com/en-us/azure/devops/pipelines/test/test-impact-analysis)
- [Microsoft Engineering Playbook - Testing](https://microsoft.github.io/code-with-engineering-playbook/automated-testing/)

[↩️ Back to Module Overview](01-introduction.md) | [➡️ Next: Availability Tests](06-set-up-run-availability-tests.md)
