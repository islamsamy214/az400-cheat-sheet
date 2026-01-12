# Integrate Multi-Job Builds with Azure Pipelines

Multi-job builds split pipeline work across multiple jobs, enabling parallel execution, conditional logic, and better organization. Understanding job dependencies and artifacts is key to efficient pipelines.

## What Are Multi-Job Builds?

**Single-Job Build** (Sequential):
```yaml
steps:
- script: npm install
- script: npm run build
- script: npm test
- script: npm run deploy
# Total: 20 minutes
```

**Multi-Job Build** (Parallel):
```yaml
jobs:
- job: Build
  steps:
  - script: npm install
  - script: npm run build
  
- job: TestUnit
  dependsOn: Build
  steps:
  - script: npm test:unit  # 5 min
  
- job: TestIntegration
  dependsOn: Build
  steps:
  - script: npm test:integration  # 5 min

# Both tests run in parallel
# Total: 10 minutes (build) + 5 minutes (tests parallel) = 15 minutes
```

## Benefits of Multi-Job Builds

### 1. Parallel Execution

**Time Savings**:
```yaml
# Before: Sequential (40 min total)
Build (10 min) → Test (15 min) → Security Scan (10 min) → Deploy (5 min)

# After: Parallel (20 min total)
Build (10 min) → [Test (15 min) + Security Scan (10 min)] → Deploy (5 min)
```

### 2. Agent Pool Separation

**Use Case**: Different jobs need different agent types

```yaml
jobs:
- job: WindowsBuild
  pool:
    vmImage: 'windows-latest'
  steps:
  - script: msbuild /p:Configuration=Release

- job: LinuxBuild
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - script: dotnet build
```

### 3. Conditional Execution

**Run jobs only when needed**:

```yaml
jobs:
- job: Build
  steps:
  - script: npm run build

- job: DeployProd
  dependsOn: Build
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  steps:
  - script: kubectl apply -f prod.yaml
  
- job: DeployDev
  dependsOn: Build
  condition: and(succeeded(), ne(variables['Build.SourceBranch'], 'refs/heads/main'))
  steps:
  - script: kubectl apply -f dev.yaml
```

### 4. Artifact Sharing

**Build once, use multiple times**:

```yaml
jobs:
- job: Build
  steps:
  - script: dotnet build --configuration Release
  - publish: $(Build.ArtifactStagingDirectory)
    artifact: drop

- job: TestWindows
  dependsOn: Build
  pool:
    vmImage: 'windows-latest'
  steps:
  - download: current
    artifact: drop
  - script: run-tests.bat

- job: TestLinux
  dependsOn: Build
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - download: current
    artifact: drop
  - script: ./run-tests.sh
```

## Job Dependencies

### Simple Dependency

```yaml
jobs:
- job: A
  steps:
  - script: echo "Job A"

- job: B
  dependsOn: A  # B waits for A to complete
  steps:
  - script: echo "Job B runs after A"
```

### Multiple Dependencies

```yaml
jobs:
- job: Build
  steps:
  - script: build

- job: TestUnit
  dependsOn: Build
  steps:
  - script: test:unit

- job: TestIntegration
  dependsOn: Build
  steps:
  - script: test:integration

- job: Deploy
  dependsOn:
  - TestUnit
  - TestIntegration
  # Waits for BOTH test jobs to complete
  steps:
  - script: deploy
```

**Execution Flow**:
```
       Build
      /       TestUnit  TestIntegration
      \     /
       Deploy
```

### No Dependencies (Parallel)

```yaml
jobs:
- job: LintCode
  steps:
  - script: npm run lint

- job: SecurityScan
  # No dependsOn = runs in parallel with LintCode
  steps:
  - script: npm audit

- job: BuildDocs
  # Also runs in parallel
  steps:
  - script: npm run docs
```

## Artifacts Between Jobs

### Publishing Artifacts

```yaml
- job: Build
  steps:
  - script: dotnet publish -c Release -o $(Build.ArtifactStagingDirectory)
  
  - task: PublishBuildArtifacts@1
    inputs:
      pathToPublish: '$(Build.ArtifactStagingDirectory)'
      artifactName: 'WebApp'
      publishLocation: 'Container'
```

### Downloading Artifacts

```yaml
- job: Deploy
  dependsOn: Build
  steps:
  - task: DownloadBuildArtifacts@1
    inputs:
      artifactName: 'WebApp'
      downloadPath: '$(System.DefaultWorkingDirectory)'
  
  - script: |
      ls -la $(System.DefaultWorkingDirectory)/WebApp
      # Deploy files from WebApp directory
```

### Publish/Download Shorthand

**Simpler syntax**:
```yaml
- job: Build
  steps:
  - script: npm run build
  - publish: dist/
    artifact: WebAppDist

- job: Deploy
  dependsOn: Build
  steps:
  - download: current
    artifact: WebAppDist
  - script: |
      cp -r $(Pipeline.Workspace)/WebAppDist/* /var/www/html/
```

## Complete Multi-Job Example

### E-Commerce Application

```yaml
trigger:
- main

variables:
  buildConfiguration: 'Release'

stages:
- stage: CI
  jobs:
  - job: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Build application'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    
    - publish: $(Build.ArtifactStagingDirectory)
      artifact: WebApp

  - job: UnitTests
    dependsOn: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: DotNetCoreCLI@2
      inputs:
        command: 'test'
        projects: '**/*UnitTests.csproj'
        arguments: '--configuration $(buildConfiguration)'

  - job: IntegrationTests
    dependsOn: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: DockerCompose@0
      displayName: 'Start dependencies'
      inputs:
        action: 'Run services'
        dockerComposeFile: 'docker-compose.test.yml'
    
    - task: DotNetCoreCLI@2
      inputs:
        command: 'test'
        projects: '**/*IntegrationTests.csproj'

  - job: SecurityScan
    dependsOn: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: SonarCloudAnalyze@1
    - task: dependency-check@6

- stage: Deploy_Staging
  dependsOn: CI
  jobs:
  - deployment: DeployWebApp
    environment: 'Staging'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: WebApp
          
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Azure-Staging'
              appName: 'myapp-staging'
              package: '$(Pipeline.Workspace)/WebApp/**/*.zip'
  
  - job: SmokeTests
    dependsOn: DeployWebApp
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - script: |
        curl -f https://myapp-staging.azurewebsites.net/health || exit 1
        curl -f https://myapp-staging.azurewebsites.net/api/status || exit 1

- stage: Deploy_Production
  dependsOn: Deploy_Staging
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployWebApp
    environment: 'Production'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: WebApp
          
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Azure-Production'
              appName: 'myapp-prod'
              package: '$(Pipeline.Workspace)/WebApp/**/*.zip'
```

## Job Conditions

### Condition Examples

```yaml
jobs:
# Always run (default)
- job: Build
  steps:
  - script: build

# Run only if Build succeeded
- job: Test
  dependsOn: Build
  condition: succeeded()
  steps:
  - script: test

# Run even if Build failed
- job: Cleanup
  dependsOn: Build
  condition: always()
  steps:
  - script: cleanup

# Run only on main branch
- job: DeployProd
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
  steps:
  - script: deploy

# Run only on PR
- job: ValidatePR
  condition: eq(variables['Build.Reason'], 'PullRequest')
  steps:
  - script: validate

# Run only if manual trigger
- job: ManualDeploy
  condition: eq(variables['Build.Reason'], 'Manual')
  steps:
  - script: deploy
```

## Visual Designer vs YAML

### Visual Designer Multi-Job

```
[Build Agent Job]
  └─ Build Task
  └─ Publish Artifact Task

[Test Agent Job] (depends on Build)
  └─ Download Artifact Task
  └─ Test Task

[Deploy Agent Job] (depends on Test)
  └─ Download Artifact Task
  └─ Deploy Task
```

**Pros**:
- Visual drag-and-drop
- Easier for beginners
- No YAML syntax knowledge needed

**Cons**:
- Not version-controlled in repo
- Harder to review in PRs
- Limited flexibility

### YAML Multi-Job (Recommended)

**Pros**:
- ✅ Version-controlled with code
- ✅ Code review via PRs
- ✅ Copy/paste between projects
- ✅ Full conditional logic support
- ✅ Easier to template

**Cons**:
- Steeper learning curve
- Syntax errors possible

## Parallel Jobs Limits

### Free Tier Limits

| Plan | Parallel Jobs | Minutes/Month |
|------|--------------|---------------|
| **Azure DevOps (Public)** | 10 | Unlimited |
| **Azure DevOps (Private)** | 1 | 1,800 |
| **GitHub Actions (Public)** | 20 | Unlimited |
| **GitHub Actions (Private)** | - | 2,000 |

**Impact**: If you define 5 parallel jobs but have 1 parallel job limit, 4 jobs queue

### Managing Limited Parallelism

```yaml
# Option 1: Sequential execution
jobs:
- job: Test1
  steps:
  - script: test1

- job: Test2
  dependsOn: Test1  # Force sequential
  steps:
  - script: test2

# Option 2: Matrix strategy (shares single job)
strategy:
  maxParallel: 1  # Run one at a time
  matrix:
    Test1: { test: 'test1' }
    Test2: { test: 'test2' }

steps:
- script: $(test)
```

## Best Practices

1. **Build Once**: Create artifacts in build job, download in subsequent jobs
2. **Parallel Tests**: Split unit/integration/UI tests into separate jobs
3. **Conditional Deployment**: Use conditions for environment-specific deployments
4. **Cleanup Jobs**: Use `condition: always()` for cleanup tasks
5. **Agent Pool Optimization**: Use appropriate agent types for each job
6. **Minimal Dependencies**: Only add dependencies when truly needed
7. **Descriptive Job Names**: Use `displayName` for clarity in UI

## Troubleshooting

### Issue: "Job waiting for agent"

**Cause**: Parallel job limit reached

**Solution**:
- Check organization's parallel job limit
- Add dependencies to serialize some jobs
- Purchase additional parallel jobs

### Issue: "Artifact not found"

**Cause**: Download task looking in wrong location

**Solution**:
```yaml
# Explicitly specify artifact
- download: current
  artifact: WebApp  # Must match publish artifact name
```

### Issue: "Job skipped"

**Cause**: Dependency failed or condition not met

**Solution**: Check conditions and dependency outcomes

```yaml
- job: Deploy
  dependsOn: Test
  condition: succeeded()  # Only runs if Test succeeded
```

## Critical Notes

- 🎯 **Multi-job = parallel execution** - Split work across jobs to reduce total pipeline time; build once, test in parallel on different agents
- 💡 **Artifacts share build output** - Publish from build job, download in test/deploy jobs; avoids rebuilding, ensures consistency
- ⚠️ **Parallel job limits apply** - Free tier = 1 parallel job (private); jobs queue if limit exceeded; purchase more or serialize jobs
- 📊 **Dependencies control order** - Use `dependsOn` to enforce execution order; no dependsOn = parallel execution
- 🔄 **Conditions enable smart routing** - Deploy to prod only on main branch, staging on feature branches; cleanup jobs run always()
- ✨ **YAML over Visual Designer** - YAML is version-controlled, reviewable in PRs, and more flexible; Visual Designer not stored with code

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-pipeline-strategy/7-integrate-multiple-job-builds)
