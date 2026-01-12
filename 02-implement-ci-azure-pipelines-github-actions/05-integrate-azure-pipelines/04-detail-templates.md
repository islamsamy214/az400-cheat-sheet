# Detail Templates

## Key Concepts

Pipeline templates promote code reusability and consistency across pipelines. Templates are YAML files that define reusable stages, jobs, steps, or variables that can be referenced from multiple pipelines.

### Template Types

| Type | Defines | Scope | Use Case |
|------|---------|-------|----------|
| **Stage Template** | Complete stages | Multiple stages | Standard deployment stages |
| **Job Template** | Complete jobs | Multiple jobs | Platform matrix builds |
| **Step Template** | Task sequences | Within jobs | Common task patterns |
| **Variable Template** | Variable sets | Pipeline-wide | Environment configurations |

### Benefits of Templates

- **DRY Principle**: Define once, use everywhere
- **Consistency**: Standardize processes across teams
- **Maintainability**: Update in one place, affect all consumers
- **Security**: Centralize security scanning, compliance checks
- **Best Practices**: Enforce organizational standards

## Stage Templates

### Template Definition (build-stage.yml)

```yaml
# File: templates/build-stage.yml
parameters:
- name: buildConfiguration
  type: string
  default: 'Release'
- name: vmImage
  type: string
  default: 'ubuntu-latest'
- name: runTests
  type: boolean
  default: true

stages:
- stage: Build
  displayName: 'Build Application'
  jobs:
  - job: BuildJob
    displayName: 'Build and Test'
    pool:
      vmImage: ${{ parameters.vmImage }}
    steps:
    - script: |
        echo "Building with configuration: ${{ parameters.buildConfiguration }}"
        dotnet build --configuration ${{ parameters.buildConfiguration }}
      displayName: 'Build project'
    
    - ${{ if eq(parameters.runTests, true) }}:
      - script: dotnet test --configuration ${{ parameters.buildConfiguration }} --no-build
        displayName: 'Run tests'
    
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
```

### Template Consumption (azure-pipelines.yml)

```yaml
# File: azure-pipelines.yml
trigger:
  branches:
    include:
    - main

resources:
  repositories:
  - repository: templates
    type: git
    name: SharedTemplates
    ref: refs/heads/main

stages:
- template: templates/build-stage.yml@templates  # From external repo
  parameters:
    buildConfiguration: 'Release'
    vmImage: 'ubuntu-latest'
    runTests: true

- template: templates/deploy-stage.yml@templates
  parameters:
    environment: 'production'
    dependsOn: 'Build'
```

## Job Templates

### Template Definition (test-job.yml)

```yaml
# File: templates/test-job.yml
parameters:
- name: testProjects
  type: string
  default: '**/*Tests.csproj'
- name: configuration
  type: string
  default: 'Release'
- name: operatingSystem
  type: object
  default:
  - name: Linux
    vmImage: 'ubuntu-latest'
  - name: Windows
    vmImage: 'windows-latest'
  - name: macOS
    vmImage: 'macOS-latest'

jobs:
- ${{ each os in parameters.operatingSystem }}:
  - job: Test_${{ os.name }}
    displayName: 'Test on ${{ os.name }}'
    pool:
      vmImage: ${{ os.vmImage }}
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Run tests'
      inputs:
        command: 'test'
        projects: '${{ parameters.testProjects }}'
        arguments: '--configuration ${{ parameters.configuration }}'
```

### Template Consumption

```yaml
stages:
- stage: Test
  displayName: 'Multi-Platform Testing'
  jobs:
  - template: templates/test-job.yml
    parameters:
      testProjects: '**/*Tests.csproj'
      configuration: 'Release'
      operatingSystem:
      - name: Linux
        vmImage: 'ubuntu-latest'
      - name: Windows
        vmImage: 'windows-latest'
```

## Step Templates

### Template Definition (build-steps.yml)

```yaml
# File: templates/build-steps.yml
parameters:
- name: buildConfiguration
  type: string
  default: 'Release'
- name: framework
  type: string
  default: 'net8.0'
- name: publishArtifacts
  type: boolean
  default: true

steps:
- task: UseDotNet@2
  displayName: 'Install .NET SDK'
  inputs:
    version: '8.x'

- script: dotnet restore
  displayName: 'Restore packages'

- script: |
    dotnet build --configuration ${{ parameters.buildConfiguration }} \
                 --framework ${{ parameters.framework }} \
                 --no-restore
  displayName: 'Build project'

- script: |
    dotnet test --configuration ${{ parameters.buildConfiguration }} \
                --no-build \
                --logger trx \
                --collect "Code coverage"
  displayName: 'Run tests'

- ${{ if eq(parameters.publishArtifacts, true) }}:
  - task: PublishTestResults@2
    displayName: 'Publish test results'
    inputs:
      testResultsFormat: 'VSTest'
      testResultsFiles: '**/*.trx'
  
  - task: PublishCodeCoverageResults@1
    displayName: 'Publish code coverage'
    inputs:
      codeCoverageTool: 'Cobertura'
      summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
```

### Template Consumption

```yaml
jobs:
- job: Build
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - template: templates/build-steps.yml
    parameters:
      buildConfiguration: 'Release'
      framework: 'net8.0'
      publishArtifacts: true
```

## Variable Templates

### Template Definition (dev-variables.yml)

```yaml
# File: templates/variables/dev-variables.yml
variables:
  environment: 'Development'
  azureSubscription: 'Dev-Subscription'
  resourceGroupName: 'rg-myapp-dev'
  appServiceName: 'myapp-dev'
  sqlServerName: 'sqlserver-dev'
  keyVaultName: 'kv-myapp-dev'
  location: 'eastus'
  sku: 'B1'
  enableDiagnostics: false
```

### Template Definition (prod-variables.yml)

```yaml
# File: templates/variables/prod-variables.yml
variables:
  environment: 'Production'
  azureSubscription: 'Prod-Subscription'
  resourceGroupName: 'rg-myapp-prod'
  appServiceName: 'myapp-prod'
  sqlServerName: 'sqlserver-prod'
  keyVaultName: 'kv-myapp-prod'
  location: 'westus2'
  sku: 'P1V2'
  enableDiagnostics: true
```

### Template Consumption

```yaml
# File: azure-pipelines-dev.yml
trigger:
  branches:
    include:
    - develop

variables:
- template: templates/variables/dev-variables.yml

stages:
- stage: Deploy
  jobs:
  - job: DeployApp
    steps:
    - script: |
        echo "Deploying to $(environment)"
        echo "Resource Group: $(resourceGroupName)"
        echo "App Service: $(appServiceName)"
```

```yaml
# File: azure-pipelines-prod.yml
trigger:
  branches:
    include:
    - main

variables:
- template: templates/variables/prod-variables.yml

stages:
- stage: Deploy
  jobs:
  - job: DeployApp
    steps:
    - script: |
        echo "Deploying to $(environment)"
        echo "Resource Group: $(resourceGroupName)"
        echo "App Service: $(appServiceName)"
```

## Advanced Template Features

### 1. Template Parameters with Types

```yaml
# File: templates/advanced-template.yml
parameters:
- name: environments
  type: object
  default:
  - name: dev
    displayName: Development
  - name: prod
    displayName: Production

- name: buildConfiguration
  type: string
  default: 'Release'
  values:  # Allowed values
  - Debug
  - Release

- name: runSmokeTests
  type: boolean
  default: true

- name: timeout
  type: number
  default: 60

stages:
- ${{ each env in parameters.environments }}:
  - stage: Deploy_${{ env.name }}
    displayName: 'Deploy to ${{ env.displayName }}'
    jobs:
    - job: Deploy
      timeoutInMinutes: ${{ parameters.timeout }}
      steps:
      - script: echo "Deploying to ${{ env.name }} with ${{ parameters.buildConfiguration }}"
      
      - ${{ if eq(parameters.runSmokeTests, true) }}:
        - script: echo "Running smoke tests"
```

### 2. Conditional Template Insertion

```yaml
# File: templates/security-scan.yml
parameters:
- name: enableSecurityScan
  type: boolean
  default: true
- name: scanType
  type: string
  default: 'full'

steps:
- ${{ if eq(parameters.enableSecurityScan, true) }}:
  - task: SecurityScan@1
    displayName: 'Security scan (${{ parameters.scanType }})'
    inputs:
      scanType: ${{ parameters.scanType }}

- ${{ if eq(parameters.scanType, 'full') }}:
  - task: VulnerabilityScan@1
    displayName: 'Vulnerability scan'

- ${{ if ne(parameters.scanType, 'quick') }}:
  - task: ComplianceScan@1
    displayName: 'Compliance scan'
```

### 3. Template with Each Loops

```yaml
# File: templates/multi-region-deploy.yml
parameters:
- name: regions
  type: object
  default:
  - name: eastus
    resourceGroup: rg-eastus
  - name: westus
    resourceGroup: rg-westus
  - name: centralus
    resourceGroup: rg-centralus

stages:
- ${{ each region in parameters.regions }}:
  - stage: Deploy_${{ region.name }}
    displayName: 'Deploy to ${{ region.name }}'
    jobs:
    - deployment: DeployRegion
      environment: 'production-${{ region.name }}'
      strategy:
        runOnce:
          deploy:
            steps:
            - script: |
                echo "Deploying to ${{ region.name }}"
                echo "Resource Group: ${{ region.resourceGroup }}"
                az webapp deploy --resource-group ${{ region.resourceGroup }}
              displayName: 'Deploy to ${{ region.name }}'
```

### 4. Nested Templates

```yaml
# File: templates/parent-template.yml
parameters:
- name: includeTests
  type: boolean
  default: true

stages:
- stage: Build
  jobs:
  - template: build-job.yml  # Nested template
    parameters:
      configuration: 'Release'

- ${{ if eq(parameters.includeTests, true) }}:
  - stage: Test
    jobs:
    - template: test-job.yml  # Nested template
      parameters:
        runIntegrationTests: true
```

## Template Repository Structure

### Recommended Organization

```
pipeline-templates/
├── stages/
│   ├── build-stage.yml
│   ├── test-stage.yml
│   └── deploy-stage.yml
├── jobs/
│   ├── build-job.yml
│   ├── test-job.yml
│   └── deploy-job.yml
├── steps/
│   ├── build-steps.yml
│   ├── test-steps.yml
│   ├── security-scan-steps.yml
│   └── deploy-steps.yml
├── variables/
│   ├── dev-variables.yml
│   ├── staging-variables.yml
│   └── prod-variables.yml
└── README.md
```

### Template Versioning

```yaml
# Option 1: Reference specific branch
resources:
  repositories:
  - repository: templates
    type: git
    name: SharedTemplates
    ref: refs/heads/v2.0  # Specific version

# Option 2: Reference specific tag
resources:
  repositories:
  - repository: templates
    type: git
    name: SharedTemplates
    ref: refs/tags/v2.0.1  # Specific version tag

# Option 3: Reference specific commit
resources:
  repositories:
  - repository: templates
    type: git
    name: SharedTemplates
    ref: abc123def456  # Specific commit SHA
```

## Complete Example: Multi-Environment Pipeline

### build-template.yml

```yaml
# File: templates/build-template.yml
parameters:
- name: solution
  type: string
  default: '**/*.sln'
- name: buildConfiguration
  type: string
  default: 'Release'

stages:
- stage: Build
  displayName: 'Build Application'
  jobs:
  - job: BuildJob
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: NuGetToolInstaller@1
      displayName: 'Install NuGet'
    
    - task: NuGetCommand@2
      displayName: 'Restore packages'
      inputs:
        command: 'restore'
        restoreSolution: '${{ parameters.solution }}'
    
    - task: VSBuild@1
      displayName: 'Build solution'
      inputs:
        solution: '${{ parameters.solution }}'
        configuration: '${{ parameters.buildConfiguration }}'
    
    - task: VSTest@2
      displayName: 'Run tests'
      inputs:
        testAssemblyVer2: |
          **\*Tests.dll
          !**\*TestAdapter.dll
          !**\obj\**
        configuration: '${{ parameters.buildConfiguration }}'
    
    - task: PublishBuildArtifacts@1
      displayName: 'Publish artifacts'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
```

### deploy-template.yml

```yaml
# File: templates/deploy-template.yml
parameters:
- name: environment
  type: string
- name: azureSubscription
  type: string
- name: appServiceName
  type: string
- name: resourceGroupName
  type: string
- name: dependsOn
  type: string
  default: 'Build'

stages:
- stage: Deploy_${{ parameters.environment }}
  displayName: 'Deploy to ${{ parameters.environment }}'
  dependsOn: ${{ parameters.dependsOn }}
  jobs:
  - deployment: DeployJob
    displayName: 'Deploy Application'
    environment: '${{ parameters.environment }}'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          
          - task: AzureWebApp@1
            displayName: 'Deploy to Azure Web App'
            inputs:
              azureSubscription: '${{ parameters.azureSubscription }}'
              appName: '${{ parameters.appServiceName }}'
              resourceGroupName: '${{ parameters.resourceGroupName }}'
              package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

### azure-pipelines.yml (Main Pipeline)

```yaml
# File: azure-pipelines.yml
trigger:
  branches:
    include:
    - main
    - develop

resources:
  repositories:
  - repository: templates
    type: git
    name: SharedTemplates
    ref: refs/heads/main

variables:
- template: templates/variables/common-variables.yml@templates

stages:
# Build stage
- template: templates/build-template.yml@templates
  parameters:
    solution: '**/*.sln'
    buildConfiguration: 'Release'

# Deploy to Dev (automatic for develop branch)
- ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/develop') }}:
  - template: templates/deploy-template.yml@templates
    parameters:
      environment: 'Development'
      azureSubscription: 'Dev-Subscription'
      appServiceName: 'myapp-dev'
      resourceGroupName: 'rg-myapp-dev'
      dependsOn: 'Build'

# Deploy to Production (automatic for main branch)
- ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
  - template: templates/deploy-template.yml@templates
    parameters:
      environment: 'Production'
      azureSubscription: 'Prod-Subscription'
      appServiceName: 'myapp-prod'
      resourceGroupName: 'rg-myapp-prod'
      dependsOn: 'Build'
```

## Critical Notes

🎯 **Template Syntax**: Use `${{ }}` for compile-time expressions (parameters, template logic) and `$()` for runtime variables.

💡 **Parameter Validation**: Define `type` and `values` for parameters to catch configuration errors at queue time, not during pipeline execution.

⚠️ **Template Security**: Store templates in protected repositories with branch policies. Template changes affect all consuming pipelines.

📊 **Each Loops**: Use `${{ each item in parameters.list }}` for compile-time iteration. This generates multiple stages/jobs/steps at pipeline compilation.

🔄 **Conditional Insertion**: `${{ if }}` evaluates at compile time. Use for inserting/excluding entire stages, jobs, or steps based on parameters.

✨ **Versioning Strategy**: Reference templates by branch, tag, or commit SHA for stability. Pin production pipelines to specific versions; use `main` for development.

## Quick Reference

### Template Types

```yaml
# Stage template
stages:
- template: templates/stage.yml

# Job template
jobs:
- template: templates/job.yml

# Step template
steps:
- template: templates/steps.yml

# Variable template
variables:
- template: templates/variables.yml
```

### Parameter Types

```yaml
parameters:
- name: stringParam
  type: string
  default: 'value'

- name: boolParam
  type: boolean
  default: true

- name: numberParam
  type: number
  default: 60

- name: objectParam
  type: object
  default:
    key: value
```

### Template Expressions

| Expression | When Evaluated | Use Case |
|------------|----------------|----------|
| `${{ }}` | Compile time | Parameters, conditionals, loops |
| `$()` | Runtime | Variables, task outputs |
| `$[ ]` | Runtime (dependencies) | Cross-job variables |

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-azure-pipelines/detail-templates/)
