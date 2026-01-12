# Implement GitHub Actions for CI/CD

This hands-on exercise demonstrates implementing a complete CI/CD pipeline using GitHub Actions to deploy a .NET web application to Azure App Service. You'll gain practical experience with workflow creation, Azure integration, and automated deployment.

## Scenario

You'll implement a GitHub Action workflow that automatically deploys the eShopOnWeb sample application to Azure App Service. This real-world scenario demonstrates enterprise-grade DevOps practices using GitHub Actions for both continuous integration (build/test) and continuous deployment (release to Azure).

**Business Context**: Modern development teams need automated pipelines that build, test, and deploy applications reliably. This exercise teaches you to create production-ready CI/CD workflows that trigger on code changes, validate quality, and deploy to cloud infrastructure.

## Learning Objectives

By completing this exercise, you will:

1. ✅ **Implement Complete CI/CD Workflow**: Create a GitHub Actions workflow that builds, tests, and deploys a .NET application
2. ✅ **Integrate with Azure**: Configure authentication and deploy to Azure App Service using service principals
3. ✅ **Understand Workflow Characteristics**: Learn key workflow components including triggers, jobs, steps, secrets, and environment variables
4. ✅ **Manage Secure Deployments**: Use GitHub secrets for secure credential management
5. ✅ **Clean Up Cloud Resources**: Properly manage and delete Azure resources to avoid unnecessary costs

## Prerequisites

Before starting this exercise, ensure you have:

### 1. Azure DevOps-Supported Browser

One of the following modern browsers:
- Microsoft Edge (latest version)
- Google Chrome (latest version)
- Mozilla Firefox (latest version)
- Apple Safari (latest version - macOS only)

### 2. Azure Subscription

**Required Permissions**:
- **Contributor** or **Owner** role in the Azure subscription
- Ability to create and manage Azure App Service resources
- Permission to create Azure service principals

**If You Don't Have an Azure Subscription**:
- Sign up for a [free trial](https://azure.microsoft.com/free/)
- Students: Use [Azure for Students](https://azure.microsoft.com/free/students/) (no credit card required)
- Microsoft Learn users: Use the [free sandbox environment](https://docs.microsoft.com/learn/)

**Cost Considerations**:
- Basic Azure App Service plan: ~$13-55/month (prorated during exercise)
- **Important**: Complete cleanup exercise to avoid ongoing charges
- Free tier available but has limitations for production scenarios

### 3. GitHub Account

**Requirements**:
- Active GitHub account (create one at [github.com/join](https://github.com/join) if needed)
- Permissions to create repositories
- Ability to configure repository secrets
- Access to GitHub Actions (included in all GitHub plans, including free)

**GitHub Actions Usage**:
- Free tier: 2,000 minutes/month for private repositories
- Public repositories: Unlimited GitHub Actions minutes
- This exercise uses approximately 10-20 minutes of runtime

### 4. Basic Knowledge Required

**Familiarity With**:
- Git basics (clone, commit, push)
- YAML syntax fundamentals
- Basic .NET application structure
- Azure portal navigation
- Command-line interfaces (bash or PowerShell)

## Exercise Overview

This hands-on lab consists of three main exercises:

```
Exercise 0: Configure Prerequisites (15 minutes)
    ↓
Exercise 1: Implement CI/CD Workflow (20-25 minutes)
    ↓
Exercise 2: Clean Up Resources (5 minutes)
```

**Total Time**: Approximately 40-45 minutes

## Exercise 0: Import eShopOnWeb Repository to GitHub

### Overview

Set up the sample application by importing the eShopOnWeb repository to your GitHub account.

### Steps

1. **Navigate to eShopOnWeb Repository**:
   - Open browser to: [https://github.com/MicrosoftLearning/eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb)

2. **Fork or Import the Repository**:
   - Click **Fork** button (top-right)
   - Select your GitHub account as the destination
   - Keep the repository name as `eShopOnWeb`
   - Click **Create fork**

3. **Verify Repository Contents**:
   - Ensure repository contains:
     - `.NET` application source code
     - `src/` directory with Web and API projects
     - `.github/workflows/` directory (may be empty initially)

4. **Clone to Local Machine** (Optional):
   ```bash
   git clone https://github.com/YOUR-USERNAME/eShopOnWeb.git
   cd eShopOnWeb
   ```

**Checkpoint**: You should now have the eShopOnWeb repository in your GitHub account.

## Exercise 1: Set Up GitHub Repository and Azure Access

### Part 1: Create Azure Resources

1. **Login to Azure Portal**:
   - Navigate to: [https://portal.azure.com](https://portal.azure.com)
   - Sign in with your Azure account

2. **Open Azure Cloud Shell**:
   - Click **Cloud Shell** icon (top-right toolbar: `>_`)
   - Select **Bash** environment
   - If prompted, create storage account (required for Cloud Shell)

3. **Create Resource Group**:
   ```bash
   # Set variables
   RESOURCE_GROUP="rg-eshoponweb-github"
   LOCATION="eastus"
   
   # Create resource group
   az group create \
     --name $RESOURCE_GROUP \
     --location $LOCATION
   ```

4. **Create App Service Plan**:
   ```bash
   APP_SERVICE_PLAN="plan-eshoponweb"
   
   az appservice plan create \
     --name $APP_SERVICE_PLAN \
     --resource-group $RESOURCE_GROUP \
     --location $LOCATION \
     --sku B1 \
     --is-linux
   ```

5. **Create Web App**:
   ```bash
   # Generate unique app name
   RANDOM_SUFFIX=$RANDOM
   WEB_APP_NAME="eshoponweb-$RANDOM_SUFFIX"
   
   az webapp create \
     --name $WEB_APP_NAME \
     --resource-group $RESOURCE_GROUP \
     --plan $APP_SERVICE_PLAN \
     --runtime "DOTNETCORE|8.0"
   
   echo "Web App Name: $WEB_APP_NAME"
   echo "Web App URL: https://$WEB_APP_NAME.azurewebsites.net"
   ```

6. **Verify Web App Creation**:
   ```bash
   az webapp show \
     --name $WEB_APP_NAME \
     --resource-group $RESOURCE_GROUP \
     --query "{Name:name, State:state, DefaultHostName:defaultHostName}"
   ```

### Part 2: Create Service Principal for GitHub Actions

1. **Get Azure Subscription ID**:
   ```bash
   SUBSCRIPTION_ID=$(az account show --query id -o tsv)
   echo "Subscription ID: $SUBSCRIPTION_ID"
   ```

2. **Create Service Principal**:
   ```bash
   # Create service principal with Contributor role
   az ad sp create-for-rbac \
     --name "github-actions-eshoponweb" \
     --role Contributor \
     --scopes /subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP \
     --sdk-auth
   ```

3. **Save Service Principal Output**:
   - Copy the entire JSON output (you'll need this for GitHub secrets)
   - Output looks like:
   ```json
   {
     "clientId": "12345678-1234-1234-1234-123456789012",
     "clientSecret": "your-secret-value-here",
     "subscriptionId": "12345678-1234-1234-1234-123456789012",
     "tenantId": "12345678-1234-1234-1234-123456789012",
     "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
     "resourceManagerEndpointUrl": "https://management.azure.com/",
     "activeDirectoryGraphResourceId": "https://graph.windows.net/",
     "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
     "galleryEndpointUrl": "https://gallery.azure.com/",
     "managementEndpointUrl": "https://management.core.windows.net/"
   }
   ```

### Part 3: Configure GitHub Repository Secrets

1. **Navigate to Repository Settings**:
   - Open your `eShopOnWeb` repository on GitHub
   - Click **Settings** tab
   - Click **Secrets and variables** → **Actions**

2. **Add Azure Credentials Secret**:
   - Click **New repository secret**
   - Name: `AZURE_CREDENTIALS`
   - Value: Paste the entire JSON output from service principal creation
   - Click **Add secret**

3. **Add Web App Name Secret**:
   - Click **New repository secret**
   - Name: `AZURE_WEBAPP_NAME`
   - Value: Your web app name (e.g., `eshoponweb-12345`)
   - Click **Add secret**

4. **Add Resource Group Secret**:
   - Click **New repository secret**
   - Name: `AZURE_RESOURCE_GROUP`
   - Value: `rg-eshoponweb-github`
   - Click **Add secret**

5. **Verify Secrets**:
   - You should see three secrets listed:
     - `AZURE_CREDENTIALS`
     - `AZURE_WEBAPP_NAME`
     - `AZURE_RESOURCE_GROUP`

### Part 4: Create GitHub Actions Workflow

1. **Create Workflow Directory**:
   - In your repository, navigate to `.github/workflows/`
   - If the directory doesn't exist, create it

2. **Create Workflow File**:
   - Create new file: `eshoponweb-cicd.yml`
   - Add the following workflow:

```yaml
name: eShopOnWeb CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:  # Allow manual trigger

env:
  DOTNET_VERSION: '8.0.x'
  BUILD_CONFIGURATION: 'Release'

jobs:
  build:
    name: Build Application
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}
      
      - name: Restore dependencies
        run: dotnet restore
        working-directory: ./src
      
      - name: Build application
        run: dotnet build --configuration ${{ env.BUILD_CONFIGURATION }} --no-restore
        working-directory: ./src
      
      - name: Run unit tests
        run: dotnet test --configuration ${{ env.BUILD_CONFIGURATION }} --no-build --verbosity normal
        working-directory: ./src
      
      - name: Publish application
        run: |
          dotnet publish ./src/Web/Web.csproj \
            --configuration ${{ env.BUILD_CONFIGURATION }} \
            --output ./publish \
            --no-build
      
      - name: Upload artifact for deployment
        uses: actions/upload-artifact@v3
        with:
          name: webapp
          path: ./publish
  
  deploy:
    name: Deploy to Azure
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment:
      name: production
      url: ${{ steps.deploy.outputs.webapp-url }}
    
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: webapp
          path: ./publish
      
      - name: Login to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Deploy to Azure Web App
        id: deploy
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
          package: ./publish
      
      - name: Verify deployment
        run: |
          echo "Application deployed successfully!"
          echo "URL: https://${{ secrets.AZURE_WEBAPP_NAME }}.azurewebsites.net"
      
      - name: Logout from Azure
        run: az logout
```

3. **Commit and Push Workflow**:
   ```bash
   git add .github/workflows/eshoponweb-cicd.yml
   git commit -m "Add CI/CD workflow for Azure deployment"
   git push origin main
   ```

### Part 5: Monitor Workflow Execution

1. **View Workflow Runs**:
   - Navigate to **Actions** tab in your repository
   - You should see the workflow triggered automatically
   - Click on the workflow run to see details

2. **Monitor Build Job**:
   - Click **Build Application** job
   - Watch real-time logs as the application builds and tests

3. **Monitor Deploy Job**:
   - After build completes, **Deploy to Azure** job starts
   - Watch deployment progress
   - Note the deployment URL in the logs

4. **Verify Deployment**:
   - Open browser to: `https://YOUR-WEBAPP-NAME.azurewebsites.net`
   - You should see the eShopOnWeb application running
   - Navigate through the application to verify functionality

### Workflow Characteristics Explained

**Key Components**:

1. **Triggers**:
   ```yaml
   on:
     push:
       branches: [main]        # Automatic on push to main
     pull_request:
       branches: [main]        # Test PRs before merge
     workflow_dispatch:        # Manual trigger option
   ```

2. **Jobs**:
   - **build**: Compiles, tests, and packages application
   - **deploy**: Deploys to Azure (only on main branch)

3. **Secrets**:
   - `AZURE_CREDENTIALS`: Service principal for authentication
   - `AZURE_WEBAPP_NAME`: Target web app name
   - Values are masked in logs for security

4. **Artifacts**:
   - Built application uploaded as artifact
   - Downloaded in deploy job for consistency

5. **Conditions**:
   - Deploy only runs on main branch pushes
   - Build runs on all PRs for validation

## Exercise 2: Remove Azure Lab Resources

### Why Clean Up Resources?

Azure resources incur costs even when not actively used. Proper cleanup prevents unexpected charges.

### Cleanup Steps

1. **Open Azure Cloud Shell**:
   - Return to Azure Portal
   - Open Cloud Shell (bash)

2. **Delete Resource Group**:
   ```bash
   # Set resource group name
   RESOURCE_GROUP="rg-eshoponweb-github"
   
   # Delete resource group (removes all contained resources)
   az group delete \
     --name $RESOURCE_GROUP \
     --yes \
     --no-wait
   
   echo "Resource group deletion initiated"
   ```

3. **Verify Deletion** (after 5-10 minutes):
   ```bash
   # Check if resource group exists
   az group exists --name $RESOURCE_GROUP
   # Output: false (once deleted)
   ```

4. **Delete Service Principal** (Optional):
   ```bash
   # Get service principal object ID
   SP_ID=$(az ad sp list --display-name "github-actions-eshoponweb" --query "[0].id" -o tsv)
   
   # Delete service principal
   az ad sp delete --id $SP_ID
   
   echo "Service principal deleted"
   ```

5. **Clean Up GitHub Secrets** (Optional):
   - Navigate to **Settings** → **Secrets and variables** → **Actions**
   - Delete secrets if no longer needed:
     - `AZURE_CREDENTIALS`
     - `AZURE_WEBAPP_NAME`
     - `AZURE_RESOURCE_GROUP`

### Verification

Confirm all resources deleted:

```bash
# List all resource groups (should not include your lab RG)
az group list --output table

# Check for any remaining app services
az webapp list --output table
```

## Key Takeaways

✅ **GitHub Actions Workflow**: Created complete CI/CD pipeline with build, test, and deploy stages  
✅ **Azure Integration**: Configured service principal authentication and deployed to App Service  
✅ **Secrets Management**: Used GitHub secrets for secure credential storage  
✅ **Artifact Sharing**: Uploaded and downloaded build artifacts between jobs  
✅ **Resource Cleanup**: Properly removed Azure resources to avoid costs

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/actions)
- [Azure App Service Deploy Action](https://github.com/Azure/webapps-deploy)
- [eShopOnWeb Sample Application](https://github.com/MicrosoftLearning/eShopOnWeb)
- [Azure CLI Reference](https://docs.microsoft.com/cli/azure/)

## Troubleshooting Common Issues

**Issue**: Service principal creation fails

**Solution**: Ensure you have Owner or User Access Administrator role in the subscription

---

**Issue**: Workflow fails with authentication error

**Solution**: Verify `AZURE_CREDENTIALS` secret contains valid JSON from service principal creation

---

**Issue**: Deployment succeeds but app doesn't work

**Solution**: Check App Service logs in Azure Portal: **Your Web App** → **Monitoring** → **Log stream**

---

**Issue**: Build fails with dependency errors

**Solution**: Verify .NET SDK version matches application requirements (8.0.x)

## Next Steps

After completing this exercise:

1. **Explore Advanced Scenarios**:
   - Add staging/production environments
   - Implement blue-green deployment
   - Add automated testing stages

2. **Enhance Security**:
   - Configure environment protection rules
   - Add required reviewers for production
   - Implement secret rotation

3. **Improve Workflow**:
   - Add caching for dependencies
   - Implement parallel test execution
   - Configure notifications (Slack, Teams)

[Learn More](https://learn.microsoft.com/en-us/training/modules/learn-continuous-integration-github-actions/10-implement-github-actions-ci-cd)
