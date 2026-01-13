# Set up and run functional tests

⏱️ **Duration**: ~60 minutes | 🎯 **Type**: Hands-on Lab

## Overview

This hands-on lab demonstrates how to set up and execute Selenium-based functional tests using self-hosted Azure DevOps agents. You'll configure a test environment, write UI automation tests, and integrate them into CI/CD pipelines for continuous validation of web application functionality.

## Learning Objectives

After completing this lab, you'll be able to:
- ✅ Set up self-hosted Azure DevOps agents with browser support
- ✅ Write Selenium WebDriver tests for web UI automation
- ✅ Configure parallel test execution across browsers
- ✅ Integrate functional tests into release pipelines
- ✅ Publish test results and screenshots for failure analysis

## Prerequisites

| Requirement | Description |
|------------|-------------|
| **Azure DevOps Organization** | Access to create agents and pipelines |
| **Azure Subscription** | For creating VM or container-based agents |
| **Web Application** | Deployed application to test (or use demo app) |
| **.NET SDK** | Version 6.0+ or Node.js 18+ for test framework |
| **Basic Selenium Knowledge** | Understanding of locators and WebDriver API |

---

## Lab Architecture

```
Azure DevOps Pipeline
    ↓
Self-Hosted Agent (Linux/Windows)
    ├── Chrome Browser + ChromeDriver
    ├── Firefox Browser + GeckoDriver
    └── Selenium Tests
         ↓
Target Web Application
    ↓
Test Results + Screenshots → Azure DevOps
```

---

## Part 1: Set Up Self-Hosted Agent

### Option A: Linux VM with Docker (Recommended)

#### Step 1.1: Create Azure VM

```bash
# Create resource group
az group create --name rg-devops-agents --location eastus

# Create VM
az vm create \
    --resource-group rg-devops-agents \
    --name vm-agent-linux \
    --image Ubuntu2204 \
    --size Standard_D2s_v3 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --public-ip-sku Standard

# Get public IP
VM_IP=$(az vm show -d \
    --resource-group rg-devops-agents \
    --name vm-agent-linux \
    --query publicIps -o tsv)

echo "VM IP: $VM_IP"
```

#### Step 1.2: Install Docker and Browsers

```bash
# SSH into VM
ssh azureuser@$VM_IP

# Install Docker
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# Add user to docker group
sudo usermod -aG docker azureuser

# Install Chrome
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install -y ./google-chrome-stable_current_amd64.deb

# Install Firefox
sudo apt install -y firefox

# Verify installations
google-chrome --version
firefox --version
```

#### Step 1.3: Install Azure DevOps Agent

```bash
# Create agent directory
mkdir ~/azagent && cd ~/azagent

# Download agent (check for latest version)
wget https://vstsagentpackage.azureedge.net/agent/3.232.3/vsts-agent-linux-x64-3.232.3.tar.gz
tar zxvf vsts-agent-linux-x64-3.232.3.tar.gz

# Configure agent
./config.sh

# You'll be prompted:
# Server URL: https://dev.azure.com/{your-organization}
# Authentication type: PAT
# Personal Access Token: [Create PAT with Agent Pools (read, manage) scope]
# Agent pool: Default (or create 'FunctionalTests' pool)
# Agent name: vm-agent-linux
# Work folder: _work (default)
# Run as service: Y

# Install and start service
sudo ./svc.sh install
sudo ./svc.sh start

# Check status
sudo ./svc.sh status
```

### Option B: Docker Container Agent

#### Alternative: Containerized Agent

**Dockerfile**: `agent.Dockerfile`

```dockerfile
FROM ubuntu:22.04

# Avoid prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl \
    git \
    jq \
    libicu-dev \
    wget \
    unzip \
    ca-certificates \
    gnupg

# Install Chrome
RUN wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list \
    && apt-get update \
    && apt-get install -y google-chrome-stable

# Install Firefox
RUN apt-get install -y firefox

# Install .NET SDK (for C# tests)
RUN wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh \
    && chmod +x dotnet-install.sh \
    && ./dotnet-install.sh --channel 8.0 --install-dir /usr/share/dotnet \
    && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet

# Create agent user
RUN useradd -m -s /bin/bash agent
USER agent
WORKDIR /home/agent

# Download and extract agent
RUN mkdir -p /home/agent/azagent \
    && cd /home/agent/azagent \
    && curl -fsSL -o vsts-agent.tar.gz https://vstsagentpackage.azureedge.net/agent/3.232.3/vsts-agent-linux-x64-3.232.3.tar.gz \
    && tar -zxf vsts-agent.tar.gz \
    && rm vsts-agent.tar.gz

WORKDIR /home/agent/azagent

# Entry point will configure and start agent
COPY start-agent.sh /home/agent/start-agent.sh
RUN sudo chmod +x /home/agent/start-agent.sh

ENTRYPOINT ["/home/agent/start-agent.sh"]
```

**Startup Script**: `start-agent.sh`

```bash
#!/bin/bash
set -e

if [ -z "$AZP_URL" ]; then
  echo "error: missing AZP_URL environment variable"
  exit 1
fi

if [ -z "$AZP_TOKEN" ]; then
  echo "error: missing AZP_TOKEN environment variable"
  exit 1
fi

if [ -z "$AZP_POOL" ]; then
  AZP_POOL="Default"
fi

cd /home/agent/azagent

# Configure agent
./config.sh --unattended \
  --agent "${AZP_AGENT_NAME:-$(hostname)}" \
  --url "$AZP_URL" \
  --auth PAT \
  --token "$AZP_TOKEN" \
  --pool "$AZP_POOL" \
  --work "_work" \
  --replace \
  --acceptTeeEula

# Start agent
./run.sh
```

**Run Container**:
```bash
# Build image
docker build -t azdevops-agent:latest -f agent.Dockerfile .

# Run container
docker run -d \
  --name azdevops-agent \
  -e AZP_URL=https://dev.azure.com/your-organization \
  -e AZP_TOKEN=your-pat-token \
  -e AZP_POOL=FunctionalTests \
  -e AZP_AGENT_NAME=docker-agent-1 \
  --shm-size=2g \
  azdevops-agent:latest
```

---

## Part 2: Write Selenium Tests

### Step 2.1: Create Test Project

#### C# with Selenium WebDriver

```bash
# Create test project
dotnet new nunit -n FunctionalTests
cd FunctionalTests

# Add Selenium packages
dotnet add package Selenium.WebDriver
dotnet add package Selenium.WebDriver.ChromeDriver
dotnet add package Selenium.WebDriver.GeckoDriver
dotnet add package Selenium.Support
dotnet add package NUnit
dotnet add package NUnit3TestAdapter
```

#### Test Base Class

**File**: `TestBase.cs`

```csharp
using NUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Firefox;
using OpenQA.Selenium.Support.UI;
using System;

namespace FunctionalTests
{
    [TestFixture]
    public abstract class TestBase
    {
        protected IWebDriver Driver { get; private set; }
        protected WebDriverWait Wait { get; private set; }
        protected string BaseUrl { get; private set; }

        [OneTimeSetUp]
        public void OneTimeSetup()
        {
            BaseUrl = Environment.GetEnvironmentVariable("BASE_URL") ?? "https://localhost:5001";
        }

        [SetUp]
        public void Setup()
        {
            var browser = TestContext.Parameters.Get("browser", "chrome").ToLower();

            Driver = browser switch
            {
                "firefox" => CreateFirefoxDriver(),
                "chrome" => CreateChromeDriver(),
                _ => throw new ArgumentException($"Unsupported browser: {browser}")
            };

            Wait = new WebDriverWait(Driver, TimeSpan.FromSeconds(10));
            Driver.Manage().Window.Maximize();
        }

        [TearDown]
        public void TearDown()
        {
            if (TestContext.CurrentContext.Result.Outcome.Status == NUnit.Framework.Interfaces.TestStatus.Failed)
            {
                TakeScreenshot(TestContext.CurrentContext.Test.Name);
            }

            Driver?.Quit();
            Driver?.Dispose();
        }

        private IWebDriver CreateChromeDriver()
        {
            var options = new ChromeOptions();
            options.AddArguments("--headless", "--no-sandbox", "--disable-dev-shm-usage", "--disable-gpu");
            return new ChromeDriver(options);
        }

        private IWebDriver CreateFirefoxDriver()
        {
            var options = new FirefoxOptions();
            options.AddArguments("--headless");
            return new FirefoxDriver(options);
        }

        private void TakeScreenshot(string testName)
        {
            try
            {
                var screenshot = ((ITakesScreenshot)Driver).GetScreenshot();
                var fileName = $"{testName}_{DateTime.Now:yyyyMMdd_HHmmss}.png";
                var filePath = Path.Combine(TestContext.CurrentContext.WorkDirectory, "screenshots", fileName);
                
                Directory.CreateDirectory(Path.GetDirectoryName(filePath));
                screenshot.SaveAsFile(filePath);
                
                TestContext.WriteLine($"Screenshot saved: {filePath}");
            }
            catch (Exception ex)
            {
                TestContext.WriteLine($"Failed to take screenshot: {ex.Message}");
            }
        }

        // Helper methods
        protected IWebElement FindElement(By by)
        {
            return Wait.Until(drv => drv.FindElement(by));
        }

        protected void ClickElement(By by)
        {
            FindElement(by).Click();
        }

        protected void EnterText(By by, string text)
        {
            var element = FindElement(by);
            element.Clear();
            element.SendKeys(text);
        }

        protected void WaitForPageLoad()
        {
            Wait.Until(drv => ((IJavaScriptExecutor)drv).ExecuteScript("return document.readyState").Equals("complete"));
        }
    }
}
```

### Step 2.2: Write Test Cases

#### Example: Login Tests

**File**: `LoginTests.cs`

```csharp
using NUnit.Framework;
using OpenQA.Selenium;

namespace FunctionalTests
{
    [TestFixture]
    [Category("Login")]
    public class LoginTests : TestBase
    {
        [Test]
        [Description("User can login with valid credentials")]
        public void Login_WithValidCredentials_Success()
        {
            // Arrange
            Driver.Navigate().GoToUrl($"{BaseUrl}/login");
            WaitForPageLoad();

            // Act
            EnterText(By.Id("email"), "testuser@example.com");
            EnterText(By.Id("password"), "TestPassword123!");
            ClickElement(By.Id("login-button"));

            // Assert
            Wait.Until(drv => drv.Url.Contains("/dashboard"));
            var welcomeMessage = FindElement(By.CssSelector(".welcome-message"));
            Assert.That(welcomeMessage.Text, Does.Contain("Welcome"));
        }

        [Test]
        [Description("Login fails with invalid credentials")]
        public void Login_WithInvalidCredentials_ShowsError()
        {
            // Arrange
            Driver.Navigate().GoToUrl($"{BaseUrl}/login");
            WaitForPageLoad();

            // Act
            EnterText(By.Id("email"), "invalid@example.com");
            EnterText(By.Id("password"), "WrongPassword");
            ClickElement(By.Id("login-button"));

            // Assert
            var errorMessage = FindElement(By.CssSelector(".error-message"));
            Assert.That(errorMessage.Text, Does.Contain("Invalid credentials"));
        }

        [Test]
        [Description("Login button is disabled with empty form")]
        public void Login_WithEmptyForm_ButtonDisabled()
        {
            // Arrange
            Driver.Navigate().GoToUrl($"{BaseUrl}/login");
            WaitForPageLoad();

            // Act & Assert
            var loginButton = FindElement(By.Id("login-button"));
            Assert.That(loginButton.Enabled, Is.False, "Login button should be disabled");
        }
    }
}
```

#### Example: E-commerce Checkout Tests

**File**: `CheckoutTests.cs`

```csharp
using NUnit.Framework;
using OpenQA.Selenium;
using System.Linq;

namespace FunctionalTests
{
    [TestFixture]
    [Category("Checkout")]
    public class CheckoutTests : TestBase
    {
        [SetUp]
        public new void Setup()
        {
            base.Setup();
            // Login before each test
            LoginAsTestUser();
        }

        [Test]
        [Description("Complete checkout flow from cart to order confirmation")]
        public void Checkout_CompleteFlow_OrderCreated()
        {
            // Arrange: Add product to cart
            Driver.Navigate().GoToUrl($"{BaseUrl}/products");
            WaitForPageLoad();
            
            var firstProduct = FindElement(By.CssSelector(".product-card:first-child .add-to-cart"));
            firstProduct.Click();
            Wait.Until(drv => FindElement(By.CssSelector(".cart-count")).Text != "0");

            // Act: Navigate to checkout
            ClickElement(By.CssSelector(".cart-icon"));
            WaitForPageLoad();
            ClickElement(By.Id("checkout-button"));

            // Fill shipping information
            EnterText(By.Id("shipping-name"), "John Doe");
            EnterText(By.Id("shipping-address"), "123 Main St");
            EnterText(By.Id("shipping-city"), "Seattle");
            EnterText(By.Id("shipping-zip"), "98101");

            // Select payment method
            ClickElement(By.Id("payment-credit-card"));
            EnterText(By.Id("card-number"), "4111111111111111");
            EnterText(By.Id("card-expiry"), "12/25");
            EnterText(By.Id("card-cvv"), "123");

            // Submit order
            ClickElement(By.Id("place-order-button"));

            // Assert: Order confirmation displayed
            Wait.Until(drv => drv.Url.Contains("/order-confirmation"));
            var confirmationMessage = FindElement(By.CssSelector(".confirmation-message"));
            Assert.That(confirmationMessage.Text, Does.Contain("Order placed successfully"));

            var orderNumber = FindElement(By.CssSelector(".order-number")).Text;
            Assert.That(orderNumber, Is.Not.Empty);
            TestContext.WriteLine($"Order number: {orderNumber}");
        }

        [Test]
        [Description("Apply discount code during checkout")]
        public void Checkout_ApplyValidDiscountCode_DiscountApplied()
        {
            // Arrange
            AddProductToCart();
            NavigateToCheckout();

            // Act
            var originalTotal = decimal.Parse(
                FindElement(By.Id("order-total"))
                    .Text.Replace("$", "").Replace(",", ""));

            EnterText(By.Id("discount-code"), "SAVE10");
            ClickElement(By.Id("apply-discount-button"));

            // Assert
            Wait.Until(drv => 
                FindElement(By.CssSelector(".discount-applied")).Displayed);

            var newTotal = decimal.Parse(
                FindElement(By.Id("order-total"))
                    .Text.Replace("$", "").Replace(",", ""));

            var expectedDiscount = originalTotal * 0.10m;
            var expectedTotal = originalTotal - expectedDiscount;

            Assert.That(newTotal, Is.EqualTo(expectedTotal).Within(0.01m));
        }

        // Helper methods
        private void LoginAsTestUser()
        {
            Driver.Navigate().GoToUrl($"{BaseUrl}/login");
            EnterText(By.Id("email"), "testuser@example.com");
            EnterText(By.Id("password"), "TestPassword123!");
            ClickElement(By.Id("login-button"));
            Wait.Until(drv => drv.Url.Contains("/dashboard"));
        }

        private void AddProductToCart()
        {
            Driver.Navigate().GoToUrl($"{BaseUrl}/products");
            ClickElement(By.CssSelector(".product-card:first-child .add-to-cart"));
            Wait.Until(drv => FindElement(By.CssSelector(".cart-count")).Text != "0");
        }

        private void NavigateToCheckout()
        {
            ClickElement(By.CssSelector(".cart-icon"));
            ClickElement(By.Id("checkout-button"));
            WaitForPageLoad();
        }
    }
}
```

---

## Part 3: Pipeline Integration

### Step 3.1: Create Pipeline YAML

**File**: `azure-pipelines-functional-tests.yml`

```yaml
trigger:
  branches:
    include:
    - main
    - develop

pool:
  name: 'FunctionalTests'  # Self-hosted agent pool

variables:
  buildConfiguration: 'Release'
  baseUrl: 'https://webapp-myapp-staging.azurewebsites.net'

stages:
- stage: Build
  displayName: 'Build Test Project'
  jobs:
  - job: BuildTests
    displayName: 'Build functional tests'
    steps:
    - task: UseDotNet@2
      displayName: 'Install .NET SDK'
      inputs:
        version: '8.x'

    - task: DotNetCoreCLI@2
      displayName: 'Restore dependencies'
      inputs:
        command: 'restore'
        projects: '**/FunctionalTests.csproj'

    - task: DotNetCoreCLI@2
      displayName: 'Build test project'
      inputs:
        command: 'build'
        projects: '**/FunctionalTests.csproj'
        arguments: '--configuration $(buildConfiguration) --no-restore'

    - task: DotNetCoreCLI@2
      displayName: 'Publish test project'
      inputs:
        command: 'publish'
        projects: '**/FunctionalTests.csproj'
        arguments: '--configuration $(buildConfiguration) --no-build --output $(Build.ArtifactStagingDirectory)/tests'
        publishWebProjects: false

    - task: PublishPipelineArtifact@1
      displayName: 'Publish test artifacts'
      inputs:
        targetPath: '$(Build.ArtifactStagingDirectory)/tests'
        artifact: 'functional-tests'
        publishLocation: 'pipeline'

- stage: FunctionalTests
  displayName: 'Run Functional Tests'
  dependsOn: Build
  condition: succeeded()
  jobs:
  # Run tests in Chrome
  - job: ChromeTests
    displayName: 'Functional Tests - Chrome'
    steps:
    - task: DownloadPipelineArtifact@2
      displayName: 'Download test artifacts'
      inputs:
        artifact: 'functional-tests'
        targetPath: '$(System.DefaultWorkingDirectory)/tests'

    - task: DotNetCoreCLI@2
      displayName: 'Run Chrome tests'
      inputs:
        command: 'test'
        projects: '$(System.DefaultWorkingDirectory)/tests/FunctionalTests.dll'
        arguments: >
          --logger "trx;LogFileName=chrome-results.trx"
          --
          NUnit.TestOutputXml=$(System.DefaultWorkingDirectory)/chrome-output
          NUnit.TestParameters=browser=chrome
        testRunTitle: 'Functional Tests - Chrome'
      env:
        BASE_URL: $(baseUrl)

    - task: PublishTestResults@2
      displayName: 'Publish Chrome test results'
      condition: always()
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: '**/chrome-results.trx'
        testRunTitle: 'Functional Tests - Chrome'
        failTaskOnFailedTests: true

    - task: PublishPipelineArtifact@1
      displayName: 'Publish Chrome screenshots'
      condition: failed()
      inputs:
        targetPath: '$(System.DefaultWorkingDirectory)/tests/screenshots'
        artifact: 'chrome-screenshots'
        publishLocation: 'pipeline'

  # Run tests in Firefox
  - job: FirefoxTests
    displayName: 'Functional Tests - Firefox'
    dependsOn: []  # Run in parallel with Chrome
    steps:
    - task: DownloadPipelineArtifact@2
      displayName: 'Download test artifacts'
      inputs:
        artifact: 'functional-tests'
        targetPath: '$(System.DefaultWorkingDirectory)/tests'

    - task: DotNetCoreCLI@2
      displayName: 'Run Firefox tests'
      inputs:
        command: 'test'
        projects: '$(System.DefaultWorkingDirectory)/tests/FunctionalTests.dll'
        arguments: >
          --logger "trx;LogFileName=firefox-results.trx"
          --
          NUnit.TestOutputXml=$(System.DefaultWorkingDirectory)/firefox-output
          NUnit.TestParameters=browser=firefox
        testRunTitle: 'Functional Tests - Firefox'
      env:
        BASE_URL: $(baseUrl)

    - task: PublishTestResults@2
      displayName: 'Publish Firefox test results'
      condition: always()
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: '**/firefox-results.trx'
        testRunTitle: 'Functional Tests - Firefox'
        failTaskOnFailedTests: true

    - task: PublishPipelineArtifact@1
      displayName: 'Publish Firefox screenshots'
      condition: failed()
      inputs:
        targetPath: '$(System.DefaultWorkingDirectory)/tests/screenshots'
        artifact: 'firefox-screenshots'
        publishLocation: 'pipeline'
```

### Step 3.2: Create Release Pipeline

**Classic Release Pipeline Configuration**:

```
Artifacts:
  - Build Pipeline: functional-tests-ci

Stages:
  ├── Stage: Staging Tests
  │   ├── Agent Pool: FunctionalTests
  │   ├── Pre-deployment: Manual approval
  │   └── Tasks:
  │       ├── Download artifact: functional-tests
  │       ├── Run tests (Chrome): dotnet test ... --browser=chrome
  │       ├── Run tests (Firefox): dotnet test ... --browser=firefox
  │       └── Publish test results
  │
  └── Stage: Production Tests (Post-deployment)
      ├── Agent Pool: FunctionalTests
      ├── Pre-deployment: Automated (after production deploy)
      └── Tasks:
          ├── Download artifact: functional-tests
          ├── Run smoke tests: dotnet test --filter Category=Smoke
          └── Publish test results
```

---

## Part 4: Test Results Analysis

### Viewing Test Results

**Azure DevOps Navigation**: Pipelines → [Your Pipeline] → Test → Runs

**Test Results Dashboard**:
```
Test Run: Functional Tests - Chrome
Status: ✅ Passed (18 of 20 tests)

Summary:
├── Total: 20 tests
├── Passed: 18 (90%)
├── Failed: 2 (10%)
└── Duration: 3m 45s

Failed Tests:
1. CheckoutTests.Checkout_ApplyInvalidDiscountCode_ShowsError
   - Duration: 12.3s
   - Error: Timed out after 10 seconds waiting for element .error-message
   - Screenshot: checkout-discount-error_20240115_143052.png

2. LoginTests.Login_WithExpiredSession_RedirectsToLogin
   - Duration: 8.7s
   - Error: Expected URL to contain "/login", but was "/dashboard"
   - Screenshot: login-session-expired_20240115_143105.png
```

### Analyzing Failures

#### Access Screenshots

**Navigation**: Pipelines → [Run] → Artifacts → chrome-screenshots

**Screenshot Files**:
```
chrome-screenshots/
├── checkout-discount-error_20240115_143052.png
└── login-session-expired_20240115_143105.png
```

#### Review Logs

```
Test: CheckoutTests.Checkout_ApplyInvalidDiscountCode_ShowsError

Test Output:
  [14:30:42] INFO: Navigating to https://webapp-myapp-staging.azurewebsites.net/checkout
  [14:30:43] INFO: Entering discount code: INVALID123
  [14:30:44] INFO: Clicking apply discount button
  [14:30:44] INFO: Waiting for error message element: .error-message
  [14:30:54] ERROR: Timed out after 10 seconds
  [14:30:54] INFO: Screenshot saved: checkout-discount-error_20240115_143052.png

Stack Trace:
  OpenQA.Selenium.WebDriverTimeoutException: Timed out after 10 seconds
     at OpenQA.Selenium.Support.UI.WebDriverWait.Until[TResult](Func`2 condition)
     at FunctionalTests.TestBase.FindElement(By by) in TestBase.cs:line 75
     at FunctionalTests.CheckoutTests.Checkout_ApplyInvalidDiscountCode_ShowsError() in CheckoutTests.cs:line 89
```

---

## Best Practices

### 1. Test Design Patterns

#### Page Object Model (POM)

**File**: `Pages/LoginPage.cs`

```csharp
public class LoginPage
{
    private readonly IWebDriver _driver;
    private readonly WebDriverWait _wait;

    // Locators
    private By EmailInput => By.Id("email");
    private By PasswordInput => By.Id("password");
    private By LoginButton => By.Id("login-button");
    private By ErrorMessage => By.CssSelector(".error-message");

    public LoginPage(IWebDriver driver, WebDriverWait wait)
    {
        _driver = driver;
        _wait = wait;
    }

    public void Navigate(string baseUrl)
    {
        _driver.Navigate().GoToUrl($"{baseUrl}/login");
        _wait.Until(drv => ((IJavaScriptExecutor)drv)
            .ExecuteScript("return document.readyState").Equals("complete"));
    }

    public void EnterEmail(string email)
    {
        _wait.Until(drv => drv.FindElement(EmailInput)).SendKeys(email);
    }

    public void EnterPassword(string password)
    {
        _driver.FindElement(PasswordInput).SendKeys(password);
    }

    public void ClickLogin()
    {
        _driver.FindElement(LoginButton).Click();
    }

    public string GetErrorMessage()
    {
        return _wait.Until(drv => drv.FindElement(ErrorMessage)).Text;
    }

    // Fluent API
    public LoginPage Login(string email, string password)
    {
        EnterEmail(email);
        EnterPassword(password);
        ClickLogin();
        return this;
    }
}
```

**Usage in Tests**:
```csharp
[Test]
public void Login_WithValidCredentials_Success()
{
    var loginPage = new LoginPage(Driver, Wait);
    loginPage.Navigate(BaseUrl);
    loginPage.Login("testuser@example.com", "TestPassword123!");

    Assert.That(Driver.Url, Does.Contain("/dashboard"));
}
```

### 2. Retry Failed Tests

```yaml
- task: DotNetCoreCLI@2
  displayName: 'Run tests with retry'
  inputs:
    command: 'test'
    projects: '**/FunctionalTests.csproj'
  retryCountOnTaskFailure: 3  # Retry up to 3 times
```

### 3. Parallel Execution

```csharp
// Enable parallel execution in NUnit
[assembly: Parallelizable(ParallelScope.Fixtures)]
```

```yaml
# Pipeline parallel jobs
jobs:
- job: ChromeTests
  # ...
- job: FirefoxTests
  dependsOn: []  # Run in parallel
```

---

## Quick Reference

| Component | Configuration | Notes |
|-----------|---------------|-------|
| **Agent Pool** | Self-hosted | Requires Chrome/Firefox installed |
| **Headless Mode** | `--headless` | Runs without visible browser |
| **Test Framework** | NUnit / xUnit | Both supported in Azure DevOps |
| **WebDriver** | ChromeDriver / GeckoDriver | Auto-downloaded via NuGet |
| **Screenshots** | On test failure | Published as pipeline artifacts |
| **Test Categories** | `[Category("Smoke")]` | Filter tests by category |

**Common Selenium Waits**:
```csharp
// Explicit wait (recommended)
Wait.Until(drv => drv.FindElement(By.Id("element")));

// Implicit wait (global, use cautiously)
Driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);

// Fluent wait (custom conditions)
var wait = new DefaultWait<IWebDriver>(Driver)
{
    Timeout = TimeSpan.FromSeconds(10),
    PollingInterval = TimeSpan.FromMilliseconds(500)
};
wait.Until(drv => drv.FindElement(By.Id("element")).Displayed);
```

---

## Key Takeaways

- 🖥️ **Self-hosted agents** required for Selenium tests (browser dependencies)
- 🐳 **Docker containers** provide consistent, reproducible test environments
- 🔄 **Parallel execution** across browsers reduces test time
- 📸 **Screenshots on failure** accelerate debugging
- 🎯 **Page Object Model** improves test maintainability
- ⚡ **Headless mode** enables faster execution without GUI
- 🚨 **Retry logic** handles flaky tests automatically
- 📊 **Test results integration** provides visibility in Azure DevOps

---

## Next Steps

✅ **Lab Complete!** You've successfully:
- Set up self-hosted agents with Selenium support
- Written functional tests for web UI automation
- Integrated tests into CI/CD pipelines
- Configured cross-browser testing

**Continue to**: Unit 9 - Knowledge check

---

## Additional Resources

- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [Azure DevOps self-hosted agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents)
- [Run Functional Tests task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/test/run-functional-tests)
- [NUnit Documentation](https://docs.nunit.org/)

[↩️ Back to Module Overview](01-introduction.md) | [➡️ Next: Knowledge Check](09-knowledge-check.md)
