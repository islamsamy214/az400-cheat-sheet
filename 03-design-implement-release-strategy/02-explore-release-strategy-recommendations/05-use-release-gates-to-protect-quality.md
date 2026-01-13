# Use Release Gates to Protect Quality

## Overview
**Quality gates** enforce organizational quality policies and determine production readiness through automated validation criteria. They represent the evolution from manual quality assurance to automated pipeline checks.

## Traditional vs Modern Quality Gates

### ❌ Traditional Manual Process
```
Build Complete → QA Department Review → Manual Checklist
→ Manager Sign-off → Hope for the Best
```

**Problems**:
- ⏰ Slow: 1-3 days for QA review
- 🐛 Inconsistent: Different reviewers, different standards
- 📄 Documentation-heavy: Checklists, spreadsheets, emails
- 🚫 Bottleneck: Limited QA resources

### ✅ Modern Automated Quality Gates
```
Build Complete → [Automated Quality Gates] → Pass? → Deploy
                  ├─ Code coverage ≥ 80%
                  ├─ Zero blocker bugs
                  ├─ No license violations
                  └─ Performance ≥ baseline
```

**Benefits**:
- ⚡ Fast: Seconds to minutes
- ✅ Consistent: Same criteria every time
- 🤖 Automated: No human bottleneck
- 📊 Measurable: Objective metrics

## Quality Gate Positioning

**Rule**: Position gates **before stages that depend on previous stage outcomes**.

```
Build → Dev → [Quality Gates] → Staging → [Quality Gates] → Production
                ↑                            ↑
                Quality validation      Final validation
```

**Example**:
```yaml
stages:
  - stage: Build
  - stage: Dev
  - stage: QualityCheck  # ← Gate stage
    dependsOn: Dev
    condition: succeeded()
    jobs:
      - job: ValidateQuality
        steps:
          - task: ValidateCodeCoverage
          - task: CheckBugCount
  - stage: Staging
    dependsOn: QualityCheck
```

## Common Quality Gate Implementations

### 1️⃣ Zero New Blocker Issues

**Criteria**: No blocker/critical bugs in backlog.

```yaml
gates:
  - task: QueryWorkItems@0
    inputs:
      queryId: '$(BlockerBugsQueryId)'
      maxThreshold: '0'
```

**Query** (WIQL):
```wiql
SELECT [System.Id], [System.Title]
FROM WorkItems
WHERE [System.WorkItemType] = 'Bug'
  AND [Microsoft.VSTS.Common.Severity] = '1 - Critical'
  AND [System.State] NOT IN ('Resolved', 'Closed')
  AND [System.CreatedDate] >= @Today - 7  -- Last 7 days
```

**Example Results**:
- ✅ 0 blocker bugs → PASS → Deploy
- ❌ 3 blocker bugs → FAIL → Block deployment

### 2️⃣ Code Coverage > 80% on New Code

**Criteria**: New/changed code must have >80% test coverage.

```yaml
steps:
  - task: DotNetCoreCLI@2
    inputs:
      command: 'test'
      arguments: '--configuration Release --collect:"XPlat Code Coverage"'
      
  - task: PublishCodeCoverageResults@1
    inputs:
      codeCoverageTool: 'Cobertura'
      summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
      
gates:
  - task: BuildQualityChecks@8
    inputs:
      checkCoverage: true
      coverageFailOption: 'build'
      coverageType: 'blocks'
      coverageThreshold: '80'
      coverageDeltaType: 'percentage'
      buildPlatform: 'AnyCPU'
```

**Calculation**:
```
Total Lines: 10,000
Covered Lines: 8,200
Coverage: 82% ✅ PASS (≥80%)

New Code Lines: 500
New Code Covered: 380
New Code Coverage: 76% ❌ FAIL (<80%)
```

### 3️⃣ No License Violations

**Criteria**: All dependencies use approved licenses.

```yaml
gates:
  - task: WhiteSource@21
    inputs:
      projectName: 'MyApp'
      checkPolicies: true
      forceCheckAllDependencies: true
```

**Approved Licenses** (example):
- ✅ MIT
- ✅ Apache 2.0
- ✅ BSD
- ❌ GPL (copyleft - not allowed in proprietary software)
- ❌ AGPL

**Example Output**:
```
Scanning 247 dependencies...
License Analysis:
- MIT: 180 packages ✅
- Apache 2.0: 50 packages ✅
- BSD: 15 packages ✅
- GPL-3.0: 2 packages ❌ BLOCKED

Result: ❌ FAIL - 2 packages with non-approved licenses
Action: Remove or replace GPL dependencies
```

### 4️⃣ No Dependency Vulnerabilities

**Criteria**: Zero high/critical severity vulnerabilities in dependencies.

```yaml
steps:
  - task: dependency-check-build-task@6
    inputs:
      projectName: 'MyApp'
      scanPath: '$(Build.SourcesDirectory)'
      format: 'HTML,JSON'
      
gates:
  - task: DependencyCheckGate@1
    inputs:
      reportPath: '$(Build.SourcesDirectory)/dependency-check-report.json'
      criticalThreshold: '0'
      highThreshold: '0'
      mediumThreshold: '5'  # Allow up to 5 medium severity
```

**Example Report**:
```
Vulnerability Summary:
├─ Critical: 0 ✅
├─ High: 0 ✅
├─ Medium: 3 ✅ (within threshold of 5)
└─ Low: 12 ⚠️ (not blocking)

Result: ✅ PASS
```

### 5️⃣ Technical Debt Maintained or Improved

**Criteria**: Technical debt ratio doesn't increase.

```yaml
steps:
  - task: SonarQubePrepare@5
    inputs:
      SonarQube: 'SonarQube-Connection'
      scannerMode: 'MSBuild'
      
  - task: DotNetCoreCLI@2
    inputs:
      command: 'build'
      
  - task: SonarQubeAnalyze@5
  
  - task: SonarQubePublish@5
    inputs:
      pollingTimeoutSec: '300'
      
gates:
  - task: SonarQubeQualityGate@1
    inputs:
      SonarQube: 'SonarQube-Connection'
```

**SonarQube Quality Gate** (example):
```
Conditions:
├─ Code Smells: ≤ 50 (Current: 42) ✅
├─ Technical Debt Ratio: ≤ 5% (Current: 3.8%) ✅
├─ Duplicated Lines: ≤ 3% (Current: 2.1%) ✅
├─ Maintainability Rating: A (Current: A) ✅
└─ Reliability Rating: A (Current: B) ❌

Result: ❌ FAIL - Reliability rating below threshold
```

### 6️⃣ Performance Benchmarks Preserved

**Criteria**: Response time and throughput meet baseline.

```yaml
steps:
  - task: RunJMeterLoadTest@1
    inputs:
      testPlan: 'performance-tests.jmx'
      
gates:
  - task: ValidatePerformance@1
    inputs:
      jmeterResultsFile: '$(Build.SourcesDirectory)/jmeter-results.jtl'
      maxResponseTime: '500'  # 500ms
      minThroughput: '1000'   # 1000 req/sec
      maxErrorRate: '1'       # 1% error rate
```

**Example Results**:
```
Performance Test Results:
├─ Avg Response Time: 420ms ✅ (<500ms)
├─ 95th Percentile: 680ms ⚠️
├─ Throughput: 1,250 req/sec ✅ (>1000)
├─ Error Rate: 0.5% ✅ (<1%)
└─ Concurrent Users: 500

Result: ✅ PASS (95th percentile is warning, not blocker)
```

### 7️⃣ Compliance Validations

#### Work Item Linkage Verification
**Criteria**: Every commit linked to work item.

```yaml
gates:
  - task: ValidateWorkItemLinks@1
    inputs:
      buildId: '$(Build.BuildId)'
      minimumLinkedWorkItems: '1'
```

**Logic**:
```
Commits in build: 15
Commits with work item links: 15
Result: ✅ PASS (100% linked)
```

#### Segregation of Duties
**Criteria**: Different person commits vs approves.

```yaml
gates:
  - task: ValidateSegregationOfDuties@1
    inputs:
      buildId: '$(Build.BuildId)'
      approvalEnvironment: 'production'
```

**Check**:
```
Committer: alice@company.com
Approver: alice@company.com
Result: ❌ FAIL - Same person committed and approved
```

---

## Complete Quality Gate Example

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main

stages:
  - stage: Build
    jobs:
      - job: BuildApp
        steps:
          - task: DotNetCoreCLI@2
            inputs:
              command: 'build'
              
          - task: DotNetCoreCLI@2
            inputs:
              command: 'test'
              arguments: '--collect:"XPlat Code Coverage"'
              
          - task: PublishCodeCoverageResults@1
          
          - task: PublishBuildArtifacts@1

  - stage: QualityGates
    dependsOn: Build
    jobs:
      - job: ValidateQuality
        steps:
          - task: DownloadBuildArtifacts@0
          
          - task: SonarQubeAnalyze@5
          
          - task: DependencyCheck@1
          
          - script: |
              echo "All quality checks completed"
              
# Pre-Deployment Gates (configured in Environment)
environments:
  - name: production
    preDeploymentGates:
      - task: QueryWorkItems@0
        displayName: 'Zero blocker bugs'
        inputs:
          queryId: '$(BlockerBugsQuery)'
          maxThreshold: '0'
          
      - task: SonarQubeQualityGate@1
        displayName: 'SonarQube quality gate'
        inputs:
          SonarQube: 'SonarQube-Connection'
          
      - task: BuildQualityChecks@8
        displayName: 'Code coverage ≥80%'
        inputs:
          checkCoverage: true
          coverageThreshold: '80'
          
      - task: DependencyCheckGate@1
        displayName: 'No critical vulnerabilities'
        inputs:
          criticalThreshold: '0'
          highThreshold: '0'
          
      - task: ValidatePerformance@1
        displayName: 'Performance baseline met'
        inputs:
          maxResponseTime: '500'
          minThroughput: '1000'

  - stage: Deploy_Production
    dependsOn: QualityGates
    condition: succeeded()
    jobs:
      - deployment: DeployProduction
        environment: production
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
```

**Workflow**:
```
1. Build stage runs (compile, test, publish)
2. Quality Gates stage evaluates:
   ├─ Zero blocker bugs: ✅ PASS
   ├─ SonarQube gate: ✅ PASS
   ├─ Code coverage ≥80%: ✅ PASS
   ├─ No critical vulns: ✅ PASS
   └─ Performance baseline: ✅ PASS
3. All gates passed → Production deployment proceeds
```

---

## Quality Gate Metrics Dashboard

### Example Metrics to Track

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| **Code Coverage** | ≥80% | 83.5% | ✅ |
| **Blocker Bugs** | 0 | 0 | ✅ |
| **Critical Vulns** | 0 | 0 | ✅ |
| **High Vulns** | 0 | 1 | ❌ |
| **Technical Debt Ratio** | ≤5% | 3.2% | ✅ |
| **Avg Response Time** | ≤500ms | 420ms | ✅ |
| **Error Rate** | ≤1% | 0.3% | ✅ |
| **License Compliance** | 100% | 99.2% | ⚠️ |

### Quality Trends Over Time

```
Code Coverage Trend:
Week 1: 75% ❌
Week 2: 78% ⚠️
Week 3: 81% ✅
Week 4: 83% ✅ (improving)

Technical Debt Trend:
Week 1: 6.5% ❌
Week 2: 5.2% ⚠️
Week 3: 4.1% ✅
Week 4: 3.2% ✅ (improving)
```

---

## Best Practices

### 🎯 Defining Quality Gates

1. **Make Gates Objective**
   ```
   ❌ Bad: "Code quality is good"
   ✅ Good: "Code coverage ≥80% AND 0 blocker bugs"
   ```

2. **Use Baseline Comparisons**
   ```yaml
   coverageDeltaType: 'percentage'
   allowedCoverageDrop: '2'  # Max 2% drop from baseline
   ```

3. **Set Realistic Thresholds**
   ```
   Start: 60% coverage (achievable)
   Month 2: 70%
   Month 4: 80%
   Month 6: 85%
   ```

4. **Combine Multiple Criteria**
   ```
   Quality = (Coverage ≥80%) AND 
             (Bugs = 0) AND 
             (Vulns = 0) AND 
             (Performance ≥baseline)
   ```

### 🛡️ Implementation Best Practices

1. **Fast Feedback**
   - Run quick checks first (bugs, coverage)
   - Run slow checks last (security scans, performance tests)

2. **Clear Failure Messages**
   ```
   ❌ "Quality gate failed"
   ✅ "Quality gate failed: Code coverage 75% (required ≥80%)"
   ```

3. **Actionable Results**
   ```
   Failure Report:
   - Code coverage: 75% (required ≥80%)
   - Missing coverage in:
     • UserService.cs: 65%
     • OrderProcessor.cs: 70%
   - Action: Add unit tests for these classes
   ```

4. **Progressive Enhancement**
   ```
   Phase 1: Track metrics (don't block)
   Phase 2: Block on critical issues only
   Phase 3: Block on all thresholds
   ```

### ⚡ Performance Best Practices

1. **Parallel Gate Execution**
   ```yaml
   gates:
     - task: CheckBugs       # Run in parallel
     - task: CheckCoverage   # Run in parallel
     - task: CheckVulns      # Run in parallel
   ```

2. **Cache Results**
   ```yaml
   - task: Cache@2
     inputs:
       key: 'dependencies | $(Agent.OS) | package-lock.json'
       path: 'node_modules'
   ```

3. **Incremental Analysis**
   ```yaml
   # Only analyze changed code
   sonar.scm.provider: git
   sonar.pullrequest.branch: $(System.PullRequest.SourceBranch)
   ```

---

## Critical Notes

⚠️ **Important Considerations**:

1. **Quality Gates Block Deployments**: Failed gate = no deployment
2. **Balance Speed vs Thoroughness**: Too many gates slow down pipeline
3. **Start Simple, Evolve**: Begin with 2-3 gates, add more over time
4. **Make Gates Reliable**: Flaky gates hurt team confidence
5. **Provide Bypass for Emergencies**: Hotfix process with expedited gates
6. **Track Gate Effectiveness**: Which gates catch real issues?
7. **Automate Remediation**: Link to tools that help fix issues

## Quick Reference

| Quality Gate | Threshold | Implementation |
|--------------|-----------|----------------|
| **Blocker Bugs** | = 0 | Query work items |
| **Code Coverage** | ≥ 80% | Code coverage report |
| **License Violations** | = 0 | WhiteSource/FOSSA |
| **Critical Vulnerabilities** | = 0 | Dependency scanning |
| **Technical Debt** | ≤ 5% | SonarQube |
| **Response Time** | ≤ 500ms | Performance tests |
| **Error Rate** | ≤ 1% | Application monitoring |
| **Work Item Linkage** | 100% | Build validation |
| **Segregation of Duties** | Enforced | Approval validation |

---

**Learn More**:
- [Quality Gates Documentation](https://docs.sonarqube.org/latest/user-guide/quality-gates/)
- [Azure DevOps Build Quality Checks](https://marketplace.visualstudio.com/items?itemName=mspremier.BuildQualityChecks)
- [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/)

[Learn More: Original Unit](https://learn.microsoft.com/en-us/training/modules/explore-release-strategy-recommendations/5-use-release-gates-to-protect-quality)
