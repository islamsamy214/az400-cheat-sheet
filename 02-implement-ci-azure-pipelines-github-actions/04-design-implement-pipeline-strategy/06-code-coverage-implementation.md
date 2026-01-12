# Analyze and Integrate Code Coverage

Code coverage measures how much of your codebase is executed during testing. Integrating coverage reporting into CI/CD pipelines helps maintain quality standards.

## What is Code Coverage?

**Code Coverage** = Percentage of code executed by tests

### Types of Coverage

| Type | Measures | Example |
|------|----------|---------|
| **Line Coverage** | % of code lines executed | 850 of 1000 lines = 85% |
| **Branch Coverage** | % of decision branches taken | if/else both paths tested |
| **Method Coverage** | % of methods called | 45 of 50 methods = 90% |
| **Statement Coverage** | % of statements executed | Similar to line coverage |

**Example**:
```csharp
public int Divide(int a, int b)
{
    if (b == 0)              // Branch 1: tested ✅
        throw new Exception();
    return a / b;            // Branch 2: tested ✅
}
// Branch coverage: 100%

public int Subtract(int a, int b)
{
    if (a < 0)               // Branch 1: NOT tested ❌
        return 0;
    return a - b;            // Branch 2: tested ✅
}
// Branch coverage: 50%
```

## Code Coverage Goals

### Industry Standards

| Coverage Level | Quality Rating | Recommendation |
|----------------|----------------|----------------|
| **< 60%** | ⚠️ Poor | Increase test coverage significantly |
| **60-70%** | 🟡 Fair | Add tests for critical paths |
| **70-80%** | ✅ Good | Standard target for most projects |
| **80-90%** | ⭐ Very Good | High-quality projects |
| **> 90%** | 🎯 Excellent | Mission-critical systems |

**Realistic Target**: 70-80% for most enterprise applications

## Tools for .NET Code Coverage

### 1. Coverlet (Open Source)

**Features**:
- Cross-platform (.NET Core)
- MSBuild integration
- Multiple output formats
- Free

**Installation**:
```xml
<---------------------------------------- Add to test project .csproj -->
<ItemGroup>
  <PackageReference Include="coverlet.msbuild" Version="6.0.0">
    <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    <PrivateAssets>all</PrivateAssets>
  </PackageReference>
</ItemGroup>
```

**Usage**:
```bash
# Generate coverage during test run
dotnet test /p:CollectCoverage=true

# Specify output format
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura

# Set coverage threshold (fail if below 80%)
dotnet test /p:CollectCoverage=true /p:Threshold=80 /p:ThresholdType=line
```

### 2. OpenCover (Windows)

**Features**:
- Deep integration with .NET Framework
- Detailed reports
- Works with older .NET versions

**Usage**:
```powershell
# Install via Chocolatey
choco install opencover

# Run with OpenCover
OpenCover.Console.exe `
  -target:"dotnet.exe" `
  -targetargs:"test MyProject.Tests.csproj" `
  -register:user `
  -filter:"+[MyProject*]* -[*.Tests]*" `
  -output:"coverage.xml"
```

### 3. Fine Code Coverage (VS Extension)

**Features**:
- Visual Studio integration
- Real-time coverage highlighting
- Free for individuals

**Visual Indicators in Code**:
- 🟢 Green: Code covered by tests
- 🔴 Red: Code not covered
- 🟡 Yellow: Partially covered branches

## Azure Pipelines Implementation

### Complete YAML Example

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  coverageThreshold: 80

steps:
# Restore dependencies
- task: DotNetCoreCLI@2
  displayName: 'Restore packages'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

# Build solution
- task: DotNetCoreCLI@2
  displayName: 'Build solution'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration)'

# Run tests with Coverlet coverage
- task: DotNetCoreCLI@2
  displayName: 'Run tests with coverage'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: |
      --configuration $(buildConfiguration)
      --no-build
      --collect:"XPlat Code Coverage"
      -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=cobertura

# Publish coverage results to Azure DevOps
- task: PublishCodeCoverageResults@1
  displayName: 'Publish coverage report'
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
    failIfCoverageEmpty: true

# Optional: Generate HTML report with ReportGenerator
- script: |
    dotnet tool install -g dotnet-reportgenerator-globaltool
    reportgenerator       -reports:$(Agent.TempDirectory)/**/coverage.cobertura.xml       -targetdir:$(Build.SourcesDirectory)/coveragereport       -reporttypes:HtmlInline_AzurePipelines
  displayName: 'Generate coverage HTML report'

# Publish HTML report as artifact
- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.SourcesDirectory)/coveragereport'
    artifactName: 'coverage-report'
```

### Coverage in Pull Request Comments

```yaml
# Add coverage delta to PR comments
- task: BuildQualityChecks@8
  inputs:
    checkCoverage: true
    coverageFailOption: 'fixed'
    coverageType: 'lines'
    coverageThreshold: '$(coverageThreshold)'
```

**Result**: PR shows coverage change (+2.5% or -1.2%)

## GitHub Actions Implementation

### Complete Workflow

```yaml
name: CI with Code Coverage

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test-and-coverage:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '8.0.x'
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build
      run: dotnet build --no-restore --configuration Release
    
    - name: Test with coverage
      run: |
        dotnet test --no-build --configuration Release           --collect:"XPlat Code Coverage"           -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=cobertura
    
    - name: Code Coverage Report
      uses: irongut/CodeCoverageSummary@v1.3.0
      with:
        filename: '**/coverage.cobertura.xml'
        badge: true
        fail_below_min: true
        format: markdown
        hide_branch_rate: false
        hide_complexity: false
        indicators: true
        output: both
        thresholds: '70 80'
    
    - name: Add Coverage PR Comment
      uses: marocchino/sticky-pull-request-comment@v2
      if: github.event_name == 'pull_request'
      with:
        recreate: true
        path: code-coverage-results.md
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        files: '**/coverage.cobertura.xml'
        fail_ci_if_error: true
```

### Using Codecov

**Setup**:
```
1. Sign up at codecov.io with GitHub account
2. Install Codecov GitHub App
3. Add repository
4. Get upload token
5. Add token to GitHub Secrets (CODECOV_TOKEN)
```

**Benefits**:
- Visual coverage reports
- PR coverage diff comments
- Trend graphs over time
- Badge for README

**Badge in README**:
```markdown
[![codecov](https://codecov.io/gh/username/repo/branch/main/graph/badge.svg)](https://codecov.io/gh/username/repo)
```

## Platform Comparison

| Feature | Azure Pipelines | GitHub Actions |
|---------|----------------|----------------|
| **Coverage Tool** | Built-in with PublishCodeCoverageResults | Third-party actions (CodeCoverageSummary) |
| **Visual Report** | Integrated in Pipelines UI | Requires Codecov/Coveralls |
| **PR Comments** | BuildQualityChecks task | sticky-pull-request-comment action |
| **Trend Graphs** | Built-in | Codecov/Coveralls |
| **Badges** | Not built-in | Codecov/Coveralls |
| **Cost** | Free tier limits | Free for public repos |

## Filtering Coverage

### Exclude Test Projects

**Coverlet**:
```bash
dotnet test /p:CollectCoverage=true   /p:Exclude="[*.Tests]*,[*]*.Migrations.*"
```

**Coverage Settings File**:
```xml
<---------------------------------------- coverlet.runsettings -->
<RunSettings>
  <DataCollectionRunSettings>
    <DataCollectors>
      <DataCollector friendlyName="XPlat Code Coverage">
        <Configuration>
          <Format>cobertura</Format>
          <Exclude>[*Tests]*,[*.Migrations]*</Exclude>
        </Configuration>
      </DataCollector>
    </DataCollectors>
  </DataCollectionRunSettings>
</RunSettings>
```

**Usage**:
```bash
dotnet test --settings coverlet.runsettings
```

## Quality Gates

### Fail Build if Coverage Drops

**Azure Pipelines**:
```yaml
- task: BuildQualityChecks@8
  inputs:
    checkCoverage: true
    coverageFailOption: 'fixed'
    coverageType: 'lines'
    coverageThreshold: 80
    allowCoverageVariance: 2  # Allow 2% decrease
```

**GitHub Actions**:
```yaml
- name: Coverage Check
  run: |
    coverage=$(grep -oP 'line-rate="\K[^"]+' coverage.cobertura.xml)
    echo "Coverage: $coverage"
    threshold=0.80
    if (( $(echo "$coverage < $threshold" | bc -l) )); then
      echo "❌ Coverage $coverage is below threshold $threshold"
      exit 1
    fi
```

## Coverage Reports

### ReportGenerator Output Formats

```bash
reportgenerator   -reports:coverage.cobertura.xml   -targetdir:coveragereport   -reporttypes:Html;HtmlSummary;Badges;Cobertura;TextSummary
```

**Available Formats**:
- `Html`: Full HTML report
- `HtmlSummary`: Summary page only
- `Badges`: SVG badges for README
- `Cobertura`: XML format for tools
- `TextSummary`: Console-friendly text
- `MarkdownSummary`: Markdown for documentation

### Sample HTML Report Structure

```
coveragereport/
├── index.html              # Main page
├── summary.html            # Coverage summary
├── MyProject/
│   ├── Services/
│   │   └── UserService.html  # Method-level details
│   └── Controllers/
│       └── UserController.html
└── badge_linecoverage.svg  # Badge for README
```

## Best Practices

1. **Set Realistic Targets**: 70-80% line coverage for most projects
2. **Enforce on PRs**: Block merges if coverage drops
3. **Exclude Generated Code**: Migrations, auto-generated files
4. **Track Trends**: Monitor coverage over time, not just absolute values
5. **Focus on Critical Paths**: 100% coverage of business logic, lower for UI
6. **Review Uncovered Code**: Periodically review what's not tested
7. **Don't Game the System**: High coverage ≠ quality tests

## Interpreting Coverage Reports

### Good Coverage Example

```
✅ UserService.cs: 92% (23/25 lines)
✅ OrderService.cs: 88% (44/50 lines)
✅ PaymentService.cs: 95% (38/40 lines)
⚠️  AdminController.cs: 45% (9/20 lines)  ← Needs attention
```

### Coverage != Quality

**Bad Test Example** (100% coverage, useless test):
```csharp
[Test]
public void TestMethod()
{
    var service = new UserService();
    service.GetUser(123);  // No assertion!
    Assert.Pass();  // Always passes
}
// 100% coverage, 0% value
```

**Good Test Example** (100% coverage, valuable test):
```csharp
[Test]
public void GetUser_ValidId_ReturnsCorrectUser()
{
    var service = new UserService();
    var user = service.GetUser(123);
    Assert.IsNotNull(user);
    Assert.AreEqual("John", user.Name);
    Assert.AreEqual("john@example.com", user.Email);
}
// 100% coverage, validates behavior
```

## Critical Notes

- 🎯 **70-80% coverage target** - Realistic goal for enterprise apps; don't aim for 100% (diminishing returns on effort)
- 💡 **Coverlet for .NET cross-platform** - Open-source, MSBuild integrated, works with dotnet test; easiest for CI/CD pipelines
- ⚠️ **Fail builds on coverage drops** - Use quality gates to prevent coverage regression; allow small variance (2%) for refactoring
- 📊 **Azure Pipelines = built-in reports** - PublishCodeCoverageResults shows trends, PR diffs; GitHub Actions needs Codecov/Coveralls
- 🔄 **Exclude generated code** - Filter out migrations, auto-generated files, test projects; focus on application logic coverage
- ✨ **Coverage ≠ quality** - 100% coverage with no assertions = useless; write meaningful tests that validate behavior

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-pipeline-strategy/6-analyze-integrate-code-coverage)
