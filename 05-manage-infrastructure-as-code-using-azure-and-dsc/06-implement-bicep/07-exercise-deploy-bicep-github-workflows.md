# Exercise - Deploy Bicep from GitHub Workflows

Automate Bicep deployments with GitHub Actions.

## GitHub Workflow YAML

```yaml
name: Deploy Bicep to Azure

on:
  push:
    branches:
      - main
    paths:
      - 'bicep/**'
  workflow_dispatch:

env:
  AZURE_RESOURCE_GROUP: rg-bicep-demo
  AZURE_LOCATION: eastus

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Validate Bicep
        uses: azure/CLI@v1
        with:
          inlineScript: |
            az bicep build --file ./bicep/main.bicep
            az deployment group validate \
              --resource-group ${{ env.AZURE_RESOURCE_GROUP }} \
              --template-file ./bicep/main.bicep

  preview:
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: What-If
        uses: azure/CLI@v1
        with:
          inlineScript: |
            az deployment group what-if \
              --resource-group ${{ env.AZURE_RESOURCE_GROUP }} \
              --template-file ./bicep/main.bicep

  deploy:
    needs: preview
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Deploy Bicep
        uses: azure/CLI@v1
        with:
          inlineScript: |
            az deployment group create \
              --resource-group ${{ env.AZURE_RESOURCE_GROUP }} \
              --template-file ./bicep/main.bicep \
              --parameters environment=prod

      - name: Output Deployment Details
        uses: azure/CLI@v1
        with:
          inlineScript: |
            az deployment group show \
              --resource-group ${{ env.AZURE_RESOURCE_GROUP }} \
              --name main \
              --query properties.outputs
```

## Setup Azure Credentials

```bash
# Create service principal
az ad sp create-for-rbac \
  --name "github-actions-sp" \
  --role contributor \
  --scopes /subscriptions/{subscription-id} \
  --sdk-auth

# Add output to GitHub Secrets as AZURE_CREDENTIALS
```

## Key Features

- **Trigger**: On push to main or manual
- **Jobs**: Validate → Preview → Deploy
- **Environments**: Manual approval gates
- **Secrets**: Secure Azure credentials

---

**Module**: Implement Bicep  
**Unit**: 7 of 10  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-bicep/7-exercise-deploy-bicep-file-github-workflows
