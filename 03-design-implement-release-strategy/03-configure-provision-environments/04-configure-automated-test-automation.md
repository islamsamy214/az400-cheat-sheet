# Configure automated integration and functional test automation

⏱️ **Duration**: ~6 minutes | 📚 **Type**: Conceptual

## Overview

Agile Testing Quadrants provide a framework for categorizing and automating different types of tests throughout the development lifecycle. This unit explores test classification strategies, automation approaches, and integration with Azure DevOps pipelines to ensure comprehensive quality assurance.

## Learning Objectives

After completing this unit, you'll be able to:
- ✅ Understand the Agile Testing Quadrants framework (Q1-Q4)
- ✅ Classify tests based on purpose and audience
- ✅ Implement automated testing strategies for each quadrant
- ✅ Integrate testing tools with Azure DevOps pipelines
- ✅ Apply testing principles for optimal coverage

---

## Agile Testing Quadrants Framework

### Quadrant Overview

The Agile Testing Quadrants, introduced by Brian Marick and popularized by Lisa Crispin and Janet Gregory, categorize tests along two dimensions:

**Dimensions**:
- **Horizontal Axis**: Business-facing vs Technology-facing
- **Vertical Axis**: Supporting the team (critique product) vs Critique product (guide development)

```
                    Business-Facing
                          |
    Q2: Business Tests    |    Q3: Business Tests
    (Functional Tests)    |    (Exploratory Testing)
    Supporting Team       |    Critique Product
                          |
--------------------------+---------------------------
                          |
    Q1: Technology Tests  |    Q4: Technology Tests
    (Unit Tests)          |    (Performance Tests)
    Supporting Team       |    Critique Product
                          |
                   Technology-Facing
```

### Quadrant Details

| Quadrant | Focus | Purpose | Automation Level | Examples |
|----------|-------|---------|------------------|----------|
| **Q1** | Technology-facing, support team | Verify implementation correctness | 🟢 High (90-100%) | Unit tests, component tests |
| **Q2** | Business-facing, support team | Validate features work as expected | 🟡 Medium-High (60-80%) | Functional tests, API tests, UI tests |
| **Q3** | Business-facing, critique product | Discover issues through exploration | 🔴 Low (0-20%) | Exploratory testing, usability testing |
| **Q4** | Technology-facing, critique product | Evaluate non-functional requirements | 🟠 Medium (40-60%) | Performance, security, load tests |

---

## Quadrant 1: Technology-Facing Tests Supporting the Team

### Characteristics

- **Purpose**: Ensure code works correctly at the lowest level
- **Audience**: Developers
- **Execution**: Automated, fast, frequently run
- **Scope**: Individual units, components, integration points

### Test Types

#### Unit Tests

**Definition**: Test individual methods, functions, or classes in isolation

**Example** (xUnit):
```csharp
using Xunit;

public class OrderServiceTests
{
    [Fact]
    public void CalculateTotal_WithValidItems_ReturnsCorrectSum()
    {
        // Arrange
        var service = new OrderService();
        var items = new List<OrderItem>
        {
            new OrderItem { Price = 10.50m, Quantity = 2 },
            new OrderItem { Price = 5.25m, Quantity = 3 }
        };

        // Act
        var total = service.CalculateTotal(items);

        // Assert
        Assert.Equal(36.75m, total);
    }

    [Theory]
    [InlineData(0, 0)]
    [InlineData(-5, -10)]
    public void CalculateTotal_WithInvalidItems_ThrowsException(decimal price, int quantity)
    {
        // Arrange
        var service = new OrderService();
        var items = new List<OrderItem> { new OrderItem { Price = price, Quantity = quantity } };

        // Act & Assert
        Assert.Throws<ArgumentException>(() => service.CalculateTotal(items));
    }
}
```

#### Component Tests

**Definition**: Test larger components or modules with dependencies mocked

**Example** (NUnit with Moq):
```csharp
using NUnit.Framework;
using Moq;

[TestFixture]
public class PaymentProcessorTests
{
    private Mock<IPaymentGateway> _mockGateway;
    private Mock<IOrderRepository> _mockRepository;
    private PaymentProcessor _processor;

    [SetUp]
    public void Setup()
    {
        _mockGateway = new Mock<IPaymentGateway>();
        _mockRepository = new Mock<IOrderRepository>();
        _processor = new PaymentProcessor(_mockGateway.Object, _mockRepository.Object);
    }

    [Test]
    public void ProcessPayment_WithValidCard_UpdatesOrderStatus()
    {
        // Arrange
        var order = new Order { Id = 123, Total = 99.99m };
        _mockGateway.Setup(g => g.Charge(It.IsAny<decimal>(), It.IsAny<string>()))
                    .Returns(new PaymentResult { Success = true, TransactionId = "TXN-001" });

        // Act
        var result = _processor.ProcessPayment(order, "4111111111111111");

        // Assert
        Assert.IsTrue(result.Success);
        _mockRepository.Verify(r => r.UpdateStatus(123, OrderStatus.Paid), Times.Once);
        _mockGateway.Verify(g => g.Charge(99.99m, "4111111111111111"), Times.Once);
    }
}
```

### Pipeline Integration

```yaml
trigger:
  branches:
    include:
    - main
    - develop

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: UseDotNet@2
  displayName: 'Install .NET SDK'
  inputs:
    version: '8.x'

- task: DotNetCoreCLI@2
  displayName: 'Restore dependencies'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Build solution'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration Release --no-restore'

- task: DotNetCoreCLI@2
  displayName: 'Run unit tests'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: '--configuration Release --no-build --collect:"XPlat Code Coverage" --logger trx'
    publishTestResults: true

- task: PublishCodeCoverageResults@1
  displayName: 'Publish code coverage'
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
    failIfCoverageEmpty: true

- script: |
    echo "Code coverage threshold: 80%"
    # Add coverage validation logic here
  displayName: 'Validate coverage threshold'
```

---

## Quadrant 2: Business-Facing Tests Supporting the Team

### Characteristics

- **Purpose**: Validate features and user stories
- **Audience**: Product owners, business analysts, QA
- **Execution**: Mostly automated, some manual exploration
- **Scope**: Feature-level, acceptance criteria validation

### Test Types

#### Functional Tests (BDD with SpecFlow)

**Definition**: Test user-facing functionality against acceptance criteria

**Example** (SpecFlow/Gherkin):

**Feature File** (`Features/Checkout.feature`):
```gherkin
Feature: Checkout Process
  As a customer
  I want to complete my purchase
  So that I can receive my order

  Background:
    Given the shopping cart contains the following items:
      | Product       | Quantity | Price  |
      | Laptop        | 1        | 999.99 |
      | Wireless Mouse| 2        | 19.99  |

  Scenario: Successful checkout with credit card
    Given I am on the checkout page
    And I have entered valid shipping information
    When I submit payment with a valid credit card
    Then I should see an order confirmation
    And I should receive a confirmation email
    And the order status should be "Confirmed"

  Scenario: Checkout fails with invalid card
    Given I am on the checkout page
    And I have entered valid shipping information
    When I submit payment with an invalid credit card
    Then I should see an error message "Payment declined"
    And the order should not be created

  Scenario Outline: Apply discount codes
    Given I am on the checkout page
    When I apply discount code "<code>"
    Then the discount of <percentage>% should be applied
    And the total should be <expectedTotal>

    Examples:
      | code        | percentage | expectedTotal |
      | SAVE10      | 10         | 935.96        |
      | NEWCUSTOMER | 15         | 881.96        |
      | INVALID     | 0          | 1039.97       |
```

**Step Definitions** (`Steps/CheckoutSteps.cs`):
```csharp
using TechTalk.SpecFlow;
using TechTalk.SpecFlow.Assist;

[Binding]
public class CheckoutSteps
{
    private readonly CheckoutPage _checkoutPage;
    private readonly TestContext _context;

    public CheckoutSteps(CheckoutPage checkoutPage, TestContext context)
    {
        _checkoutPage = checkoutPage;
        _context = context;
    }

    [Given(@"the shopping cart contains the following items:")]
    public void GivenTheShoppingCartContainsItems(Table table)
    {
        var items = table.CreateSet<CartItem>();
        _context.Cart.AddRange(items);
    }

    [Given(@"I am on the checkout page")]
    public void GivenIAmOnTheCheckoutPage()
    {
        _checkoutPage.Navigate();
    }

    [When(@"I submit payment with a valid credit card")]
    public void WhenISubmitPaymentWithValidCard()
    {
        _checkoutPage.EnterCreditCard("4111111111111111", "12/25", "123");
        _checkoutPage.SubmitOrder();
    }

    [Then(@"I should see an order confirmation")]
    public void ThenIShouldSeeOrderConfirmation()
    {
        Assert.IsTrue(_checkoutPage.IsConfirmationDisplayed());
        Assert.IsNotNull(_checkoutPage.GetOrderNumber());
    }
}
```

#### API Tests (RestSharp)

**Definition**: Test REST API endpoints for correctness and contract validation

**Example**:
```csharp
using RestSharp;
using Xunit;

public class OrderApiTests
{
    private readonly RestClient _client;

    public OrderApiTests()
    {
        _client = new RestClient("https://api.example.com");
    }

    [Fact]
    public async Task CreateOrder_WithValidData_ReturnsCreated()
    {
        // Arrange
        var request = new RestRequest("/api/orders", Method.Post);
        request.AddJsonBody(new
        {
            customerId = 123,
            items = new[]
            {
                new { productId = 1, quantity = 2 },
                new { productId = 5, quantity = 1 }
            },
            shippingAddress = new
            {
                street = "123 Main St",
                city = "Seattle",
                zipCode = "98101"
            }
        });

        // Act
        var response = await _client.ExecuteAsync<OrderResponse>(request);

        // Assert
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
        Assert.NotNull(response.Data);
        Assert.True(response.Data.OrderId > 0);
        Assert.Equal("Pending", response.Data.Status);
    }

    [Fact]
    public async Task GetOrder_WithValidId_ReturnsOrderDetails()
    {
        // Arrange
        var orderId = 12345;
        var request = new RestRequest($"/api/orders/{orderId}", Method.Get);

        // Act
        var response = await _client.ExecuteAsync<OrderResponse>(request);

        // Assert
        Assert.Equal(HttpStatusCode.OK, response.StatusCode);
        Assert.Equal(orderId, response.Data.OrderId);
        Assert.NotEmpty(response.Data.Items);
    }
}
```

### Pipeline Integration

```yaml
- stage: FunctionalTests
  displayName: 'Functional Test Automation'
  dependsOn: Build
  jobs:
  - job: SpecFlowTests
    displayName: 'Run SpecFlow BDD tests'
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Run SpecFlow tests'
      inputs:
        command: 'test'
        projects: '**/*Specs.csproj'
        arguments: '--configuration Release --logger "trx;LogFileName=specflow-results.trx"'
    
    - task: PublishTestResults@2
      displayName: 'Publish SpecFlow results'
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: '**/specflow-results.trx'
        testRunTitle: 'SpecFlow BDD Tests'

  - job: ApiTests
    displayName: 'Run API integration tests'
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Run API tests'
      inputs:
        command: 'test'
        projects: '**/*Api.Tests.csproj'
        arguments: '--configuration Release --logger "trx;LogFileName=api-results.trx"'
    
    - task: PublishTestResults@2
      displayName: 'Publish API test results'
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: '**/api-results.trx'
        testRunTitle: 'API Integration Tests'
```

---

## Quadrant 3: Business-Facing Tests Critiquing the Product

### Characteristics

- **Purpose**: Discover unexpected issues and usability problems
- **Audience**: QA testers, product owners, UX designers
- **Execution**: Manual, exploratory, creative
- **Scope**: User experience, edge cases, real-world scenarios

### Test Types

#### Exploratory Testing

**Definition**: Unscripted testing where testers explore the application to find defects

**Approach**:
```
Session-Based Test Management (SBTM):
1. Charter: Define mission (e.g., "Explore checkout flow for payment edge cases")
2. Time-box: 60-90 minute sessions
3. Notes: Document findings, questions, risks
4. Debrief: Review with team, convert issues to bugs/stories
```

**Example Charter**:
```markdown
# Exploratory Testing Charter

**Mission**: Explore the shopping cart and checkout process for data integrity and edge cases

**Focus Areas**:
- Adding/removing items with rapid clicks
- Applying multiple discount codes
- Network interruptions during checkout
- Browser back button during payment
- Session timeout scenarios
- Invalid shipping addresses

**Time Box**: 90 minutes
**Tester**: [Name]
**Date**: 2024-01-15

**Findings**:
1. 🐛 Bug: Applying discount code twice doubles the discount
2. ⚠️ Issue: Payment button enabled even with empty form
3. 💡 Improvement: No loading indicator during payment processing
4. ✅ Passed: Browser back button properly resets state
```

#### Usability Testing

**Definition**: Test with real users to evaluate interface intuitiveness and user satisfaction

**Metrics**:
- Task completion rate
- Time on task
- Error rate
- Satisfaction score (SUS - System Usability Scale)

**Tools**:
- Azure DevOps Test Plans (Manual Testing)
- User session recordings (e.g., Hotjar, FullStory)
- Feedback widgets

---

## Quadrant 4: Technology-Facing Tests Critiquing the Product

### Characteristics

- **Purpose**: Evaluate non-functional requirements (performance, security, reliability)
- **Audience**: DevOps engineers, architects, security specialists
- **Execution**: Automated, resource-intensive, scheduled
- **Scope**: System-wide attributes

### Test Types

#### Performance Tests

**Definition**: Measure response times, throughput, and resource utilization under various loads

**Example** (Apache JMeter):
```xml
<jmeterTestPlan version="1.2">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testname="API Load Test">
      <stringProp name="TestPlan.comments">Load test for Order API</stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
    </TestPlan>
    <hashTree>
      <ThreadGroup guiclass="ThreadGroupGui" testname="User Requests">
        <stringProp name="ThreadGroup.num_threads">50</stringProp>
        <stringProp name="ThreadGroup.ramp_time">30</stringProp>
        <stringProp name="ThreadGroup.duration">300</stringProp>
      </ThreadGroup>
      <hashTree>
        <HTTPSamplerProxy>
          <stringProp name="HTTPSampler.domain">api.example.com</stringProp>
          <stringProp name="HTTPSampler.path">/api/orders</stringProp>
          <stringProp name="HTTPSampler.method">POST</stringProp>
        </HTTPSamplerProxy>
      </hashTree>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
```

**Azure Load Testing Integration** (See Unit 7 for details)

#### Security Tests

**Definition**: Test for vulnerabilities, authentication flaws, and injection attacks

**Example** (OWASP ZAP):
```yaml
- task: OwaspZapScan@1
  displayName: 'Run OWASP ZAP security scan'
  inputs:
    ZapApiUrl: 'http://localhost:8080'
    ZapApiKey: '$(ZapApiKey)'
    TargetUrl: 'https://staging.example.com'
    ContextId: 'Default Context'
    UserId: '1'
    ScanType: 'activeScan'
    ScanPolicyName: 'Default Policy'
    GenerateReport: true
    ReportPath: '$(Build.ArtifactStagingDirectory)/zap-report.html'
```

**Common Security Tests**:
- SQL injection
- Cross-site scripting (XSS)
- Authentication bypass
- Insecure deserialization
- Security misconfiguration

---

## Testing Principles

### Principle 1: Test at the Lowest Level Possible

**Pyramid Model**:
```
        /\          <-- Few UI tests (expensive, slow, flaky)
       /  \
      / UI \
     /------\       <-- Some integration tests (moderate cost/speed)
    /  API  \
   /----------\     <-- Many unit tests (cheap, fast, stable)
  /   UNIT    \
 /--------------\
```

**Benefits**:
- ✅ Faster feedback (unit tests run in seconds)
- ✅ Easier debugging (smaller scope to investigate)
- ✅ Lower maintenance (less UI brittleness)
- ✅ Better code design (testable code is modular code)

### Principle 2: Write Tests Before Fixing Defects

**Red-Green-Refactor** (Test-Driven Development):
```
1. 🔴 RED: Write failing test that reproduces the bug
2. 🟢 GREEN: Write minimal code to make test pass
3. 🔵 REFACTOR: Clean up code while tests remain green
```

**Example**:
```csharp
// 1. RED: Test exposes bug (discount applies twice)
[Test]
public void ApplyDiscount_CalledTwiceWithSameCode_AppliesOnlyOnce()
{
    var cart = new ShoppingCart();
    cart.AddItem(new Item { Price = 100 });
    
    cart.ApplyDiscount("SAVE10");
    cart.ApplyDiscount("SAVE10"); // Should be idempotent
    
    Assert.AreEqual(90, cart.Total); // Should be 90, not 80
}

// 2. GREEN: Fix the bug
public void ApplyDiscount(string code)
{
    if (_appliedDiscounts.Contains(code))
        return; // Already applied, do nothing
    
    var discount = _discountService.GetDiscount(code);
    _total -= _total * discount.Percentage;
    _appliedDiscounts.Add(code);
}

// 3. REFACTOR: Extract logic, improve naming
```

### Principle 3: Design for Testability

**SOLID Principles**:
- **S**ingle Responsibility: Classes have one reason to change
- **O**pen/Closed: Open for extension, closed for modification
- **L**iskov Substitution: Subtypes are substitutable for base types
- **I**nterface Segregation: Many specific interfaces > one general interface
- **D**ependency Inversion: Depend on abstractions, not concretions

**Example** (Dependency Injection):
```csharp
// ❌ BAD: Hard to test (concrete dependencies)
public class OrderService
{
    private SqlConnection _connection = new SqlConnection("...");
    private EmailSender _emailSender = new EmailSender();
    
    public void CreateOrder(Order order)
    {
        // Uses real database and email service
    }
}

// ✅ GOOD: Easy to test (injected dependencies)
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly IEmailService _emailService;
    
    public OrderService(IOrderRepository repository, IEmailService emailService)
    {
        _repository = repository;
        _emailService = emailService;
    }
    
    public void CreateOrder(Order order)
    {
        // Can inject mocks for testing
    }
}

// Test with mocks
[Test]
public void CreateOrder_ValidOrder_SavesToRepository()
{
    var mockRepo = new Mock<IOrderRepository>();
    var mockEmail = new Mock<IEmailService>();
    var service = new OrderService(mockRepo.Object, mockEmail.Object);
    
    service.CreateOrder(new Order { /* ... */ });
    
    mockRepo.Verify(r => r.Save(It.IsAny<Order>()), Times.Once);
}
```

---

## Tool Integration

### Testing Tools by Category

| Category | Tools | Integration |
|----------|-------|-------------|
| **Unit Testing** | xUnit, NUnit, MSTest, JUnit, pytest | Built into Azure Pipelines via test tasks |
| **BDD/Functional** | SpecFlow, Cucumber, Behave | Custom test execution + result publishing |
| **API Testing** | Postman, RestAssured, RestSharp | Newman CLI, direct test execution |
| **UI Testing** | Selenium, Playwright, Cypress | Self-hosted agents with browsers installed |
| **Load Testing** | JMeter, Gatling, k6, Azure Load Testing | Azure Load Testing task, JMeter CLI |
| **Security** | OWASP ZAP, Burp Suite, SonarQube | Marketplace extensions, CLI integration |
| **Code Coverage** | Coverlet, JaCoCo, coverage.py | Built-in code coverage tasks |

### Unified Test Reporting

```yaml
- task: PublishTestResults@2
  displayName: 'Publish all test results'
  inputs:
    testResultsFormat: 'JUnit'  # or VSTest, NUnit, XUnit
    testResultsFiles: '**/test-results/**/*.xml'
    mergeTestResults: true
    failTaskOnFailedTests: true
    testRunTitle: 'Automated Test Suite'
    buildConfiguration: '$(BuildConfiguration)'
    buildPlatform: '$(BuildPlatform)'

- task: PublishCodeCoverageResults@1
  displayName: 'Publish code coverage'
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '**/coverage.cobertura.xml'
    reportDirectory: '**/coverage-report'
    failIfCoverageEmpty: true
```

---

## Quick Reference

| Quadrant | Focus | Automation | Key Tools |
|----------|-------|------------|-----------|
| **Q1: Unit/Component** | Technology, support team | 🟢 High | xUnit, NUnit, JUnit, Moq |
| **Q2: Functional/API** | Business, support team | 🟡 Medium-High | SpecFlow, Selenium, Postman |
| **Q3: Exploratory/UX** | Business, critique product | 🔴 Low | Manual testing, Test Plans |
| **Q4: Performance/Security** | Technology, critique product | 🟠 Medium | JMeter, OWASP ZAP, Load Testing |

**Automation Priority**:
1. 🥇 Unit tests (Q1) - Automate 90-100%
2. 🥈 Functional tests (Q2) - Automate 60-80%
3. 🥉 Performance tests (Q4) - Automate 40-60%
4. Manual exploratory (Q3) - Automate 0-20%

---

## Key Takeaways

- 🎯 **Agile Testing Quadrants** categorize tests by audience and purpose
- 🔄 **Quadrant 1 (Unit tests)** should form the foundation (most tests, fastest feedback)
- ✅ **Quadrant 2 (Functional tests)** validates business requirements are met
- 🔍 **Quadrant 3 (Exploratory)** discovers issues automation can't find
- ⚡ **Quadrant 4 (Performance/Security)** ensures non-functional requirements are satisfied
- 🤖 **Automate strategically** - not all tests benefit from automation
- 📊 **Integrate with pipelines** for continuous quality feedback
- 🛠️ **Design for testability** from the start (dependency injection, SOLID principles)

---

## Next Steps

✅ **Completed**: Test automation framework and classification

**Continue to**: Unit 5 - Understand Shift-left testing

---

## Additional Resources

- [Agile Testing Quadrants - Lisa Crispin](https://lisacrispin.com/2011/11/08/using-the-agile-testing-quadrants/)
- [Azure DevOps Test Plans](https://learn.microsoft.com/en-us/azure/devops/test/overview)
- [SpecFlow Documentation](https://docs.specflow.org/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

[↩️ Back to Module Overview](01-introduction.md) | [➡️ Next: Shift-left Testing](05-understand-shift-left.md)
