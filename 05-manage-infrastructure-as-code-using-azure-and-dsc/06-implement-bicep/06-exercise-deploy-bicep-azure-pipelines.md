# Exercise - Deploy Bicep from Azure Pipelines

Integrate Bicep deployments into Azure Pipelines for CI/CD automation.

## Azure Pipeline YAML

```yaml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - bicep/*

pool:
  vmImage: 'ubuntu-latest'

variables:
  resourceGroupName: 'rg-bicep-demo'
  location: 'eastus'
  bicepFile: 'bicep/main.bicep'

stages:
  - stage: Validate
    jobs:
      - job: ValidateTemplate
        steps:
          - task: AzureCLI@2
            displayName: 'Validate Bicep Template'
            inputs:
              azureSubscription: '$(azureServiceConnection)'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az bicep build --file $(bicepFile)
                az deployment group validate \
                  --resource-group $(resourceGroupName) \
                  --template-file $(bicepFile)

  - stage: Preview
    dependsOn: Validate
    jobs:
      - job: WhatIf
        steps:
          - task: AzureCLI@2
            displayName: 'What-If Analysis'
            inputs:
              azureSubscription: '$(azureServiceConnection)'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az deployment group what-if \
                  --resource-group $(resourceGroupName) \
                  --template-file $(bicepFile)

  - stage: Deploy
    dependsOn: Preview
    jobs:
      - deployment: DeployBicep
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - checkout: self
                - task: AzureCLI@2
                  displayName: 'Deploy Bicep Template'
                  inputs:
                    azureSubscription: '$(azureServiceConnection)'
                    scriptType: 'bash'
                    scriptLocation: 'inlineScript'
                    inlineScript: |
                      az deployment group create \
                        --resource-group $(resourceGroupName) \
                        --template-file $(bicepFile) \
                        --parameters environment=prod
```

## Key Features

- **Validation**: Check template before deployment
- **What-If**: Preview changes
- **Staged deployment**: Validate → Preview → Deploy
- **Environment gates**: Manual approval before production
- **Service connection**: Authenticate with Azure

---

**Module**: Implement Bicep  
**Unit**: 6 of 10  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-bicep/6-exercise-deploy-bicep-file-azure-pipelines
