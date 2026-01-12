# Deploy Docker Containers to Azure App Service Web Apps

Azure App Service provides a fully managed platform for hosting containerized web applications. This hands-on unit covers deploying custom Docker containers to Azure App Service, configuring continuous deployment, and managing containerized applications in production.

## Scenario Overview

**Deploy a containerized web application** to Azure App Service with continuous deployment from Azure Container Registry.

```
Development Workflow

Local Development
      ↓
Docker Build & Test
      ↓
Push to Azure Container Registry (ACR)
      ↓
Azure App Service Auto-Deploys
      ↓
Production (with monitoring)
```

## Prerequisites

### Required Resources

| Resource | Purpose | How to Get |
|----------|---------|------------|
| **Azure Subscription** | Host App Service | [Free trial](https://azure.microsoft.com/free/) |
| **Azure CLI** | Command-line deployment | [Install guide](https://docs.microsoft.com/cli/azure/install-azure-cli) |
| **Docker** | Build container images | [Docker Desktop](https://www.docker.com/products/docker-desktop) |
| **Containerized App** | Application to deploy | Build your own or use sample |

### Sample Application

```dockerfile
# Sample Node.js application
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
  CMD node healthcheck.js || exit 1

# Run application
CMD ["node", "server.js"]
```

```javascript
// server.js - Simple Express application
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Azure App Service!',
    environment: process.env.NODE_ENV || 'production',
    version: '1.0.0'
  });
});

app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy' });
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Step 1: Create Azure Container Registry

**Create a private registry** for storing container images:

```bash
# Variables
RESOURCE_GROUP="myResourceGroup"
LOCATION="eastus"
ACR_NAME="myregistry$(date +%s)"  # Unique name
APP_NAME="mywebapp$(date +%s)"    # Unique name
PLAN_NAME="myAppServicePlan"

# Create resource group
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION

# Create Azure Container Registry
az acr create \
  --resource-group $RESOURCE_GROUP \
  --name $ACR_NAME \
  --sku Basic \
  --location $LOCATION

# Enable admin account (for App Service authentication)
az acr update \
  --name $ACR_NAME \
  --admin-enabled true

# Get ACR credentials
ACR_LOGIN_SERVER=$(az acr show --name $ACR_NAME --query loginServer --output tsv)
ACR_USERNAME=$(az acr credential show --name $ACR_NAME --query username --output tsv)
ACR_PASSWORD=$(az acr credential show --name $ACR_NAME --query passwords[0].value --output tsv)

echo "ACR Login Server: $ACR_LOGIN_SERVER"
echo "ACR Username: $ACR_USERNAME"
```

## Step 2: Build and Push Container Image

**Build the application container** and push to ACR:

### Option A: Build Locally and Push

```bash
# Login to ACR
az acr login --name $ACR_NAME

# Build image
docker build -t $ACR_LOGIN_SERVER/myapp:v1 .

# Test locally (optional)
docker run -d -p 8080:3000 --name test $ACR_LOGIN_SERVER/myapp:v1
curl http://localhost:8080  # Verify it works
docker stop test && docker rm test

# Push to ACR
docker push $ACR_LOGIN_SERVER/myapp:v1

# Verify image in registry
az acr repository list --name $ACR_NAME --output table
az acr repository show-tags --name $ACR_NAME --repository myapp --output table
```

### Option B: Build in ACR (Recommended)

```bash
# Build directly in ACR (no local Docker required!)
az acr build \
  --registry $ACR_NAME \
  --image myapp:v1 \
  --file Dockerfile \
  .

# Output shows build progress:
# Uploading source code...
# Building image...
# Pushing image to registry...
# Build complete
```

**Benefits of ACR Build**:
- No local Docker daemon required
- Faster builds (cloud infrastructure)
- Built images immediately available in registry
- Automatic build logs stored in ACR

## Step 3: Create App Service Plan

**Provision hosting infrastructure** for App Service:

```bash
# Create Linux App Service Plan
az appservice plan create \
  --name $PLAN_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --is-linux \
  --sku B1  # Basic tier (B1, B2, B3, S1-S3, P1v2-P3v2)

# View plan details
az appservice plan show \
  --name $PLAN_NAME \
  --resource-group $RESOURCE_GROUP \
  --output table
```

### App Service Plan Tiers

| SKU | vCPU | RAM | Monthly Cost* | Use Case |
|-----|------|-----|---------------|----------|
| **B1** | 1 | 1.75 GB | ~$13 | Development/Testing |
| **B2** | 2 | 3.5 GB | ~$25 | Small production workloads |
| **S1** | 1 | 1.75 GB | ~$70 | Production with staging slots |
| **S2** | 2 | 3.5 GB | ~$140 | Medium production workloads |
| **P1v2** | 1 | 3.5 GB | ~$85 | Premium production |
| **P2v2** | 2 | 7 GB | ~$170 | High-traffic production |

*Approximate pricing (varies by region)

## Step 4: Create Web App with Container

**Deploy the containerized application**:

```bash
# Create web app with custom container
az webapp create \
  --resource-group $RESOURCE_GROUP \
  --plan $PLAN_NAME \
  --name $APP_NAME \
  --deployment-container-image-name $ACR_LOGIN_SERVER/myapp:v1

# Configure ACR credentials
az webapp config container set \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --docker-custom-image-name $ACR_LOGIN_SERVER/myapp:v1 \
  --docker-registry-server-url https://$ACR_LOGIN_SERVER \
  --docker-registry-server-user $ACR_USERNAME \
  --docker-registry-server-password $ACR_PASSWORD

# Get web app URL
APP_URL=$(az webapp show --name $APP_NAME --resource-group $RESOURCE_GROUP --query defaultHostName --output tsv)
echo "Application URL: https://$APP_URL"

# Wait for deployment (takes 1-2 minutes)
sleep 60

# Test the application
curl https://$APP_URL
```

**Expected Output**:
```json
{
  "message": "Hello from Azure App Service!",
  "environment": "production",
  "version": "1.0.0"
}
```

## Step 5: Enable Continuous Deployment

**Automatically deploy** when new images are pushed to ACR:

```bash
# Enable continuous deployment (webhook)
az webapp deployment container config \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --enable-cd true

# Get webhook URL (created automatically)
WEBHOOK_URL=$(az webapp deployment container show-cd-url \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --query CI_CD_URL \
  --output tsv)

echo "Webhook URL: $WEBHOOK_URL"
```

**How Continuous Deployment Works**:

```
Developer pushes new image to ACR
        ↓
ACR triggers webhook to App Service
        ↓
App Service pulls new image
        ↓
App Service restarts with new image
        ↓
Zero downtime (old instance → new instance)
```

**Test Continuous Deployment**:

```bash
# Make a change to your application
# Update version in server.js to "1.0.1"

# Build and push new version
az acr build \
  --registry $ACR_NAME \
  --image myapp:v1 \
  --file Dockerfile \
  .

# Wait 30-60 seconds for webhook trigger
sleep 60

# Verify new version deployed
curl https://$APP_URL
# Should show version: "1.0.1"
```

## Step 6: Configure Application Settings

**Set environment variables** for the containerized application:

```bash
# Configure app settings (environment variables)
az webapp config appsettings set \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --settings \
    NODE_ENV=production \
    LOG_LEVEL=info \
    DATABASE_URL="postgresql://server:5432/db" \
    API_KEY="@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/apikey/)"

# Configure container settings
az webapp config set \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --startup-file ""  # Optional startup command override

# View all settings
az webapp config appsettings list \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --output table
```

### Port Configuration

**App Service expects containers to listen on specific ports**:

```bash
# Set custom port (if app doesn't use 80/8080)
az webapp config appsettings set \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --settings WEBSITES_PORT=3000

# App Service automatically forwards traffic to this port
```

**Default Port Detection**:
1. Port 80 (HTTP)
2. Port 8080 (alternate HTTP)
3. Port specified in `EXPOSE` instruction
4. Port specified in `WEBSITES_PORT` setting

## Step 7: Enable Monitoring and Logging

**Configure diagnostics** for production monitoring:

```bash
# Enable application logging
az webapp log config \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --docker-container-logging filesystem \
  --level information

# View live logs
az webapp log tail \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP

# Download logs
az webapp log download \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --log-file logs.zip
```

### Application Insights Integration

```bash
# Create Application Insights
az monitor app-insights component create \
  --app myAppInsights \
  --location $LOCATION \
  --resource-group $RESOURCE_GROUP

# Get instrumentation key
INSTRUMENTATION_KEY=$(az monitor app-insights component show \
  --app myAppInsights \
  --resource-group $RESOURCE_GROUP \
  --query instrumentationKey \
  --output tsv)

# Configure App Service to use Application Insights
az webapp config appsettings set \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --settings \
    APPINSIGHTS_INSTRUMENTATIONKEY=$INSTRUMENTATION_KEY \
    APPLICATIONINSIGHTS_CONNECTION_STRING="InstrumentationKey=$INSTRUMENTATION_KEY"
```

## Step 8: Configure Scaling

**Auto-scale based on metrics**:

```bash
# Scale out (add more instances)
az webapp scale \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --instance-count 3

# Configure auto-scaling rules
az monitor autoscale create \
  --resource-group $RESOURCE_GROUP \
  --name myAutoScaleSettings \
  --resource $APP_NAME \
  --resource-type Microsoft.Web/sites \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Add scale-out rule (CPU > 70%)
az monitor autoscale rule create \
  --resource-group $RESOURCE_GROUP \
  --autoscale-name myAutoScaleSettings \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 1

# Add scale-in rule (CPU < 30%)
az monitor autoscale rule create \
  --resource-group $RESOURCE_GROUP \
  --autoscale-name myAutoScaleSettings \
  --condition "Percentage CPU < 30 avg 10m" \
  --scale in 1
```

## Deployment Slots (Staging/Production)

**Zero-downtime deployments** with staging slots:

```bash
# Create staging slot
az webapp deployment slot create \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --slot staging

# Deploy new version to staging
az webapp config container set \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --slot staging \
  --docker-custom-image-name $ACR_LOGIN_SERVER/myapp:v2 \
  --docker-registry-server-url https://$ACR_LOGIN_SERVER \
  --docker-registry-server-user $ACR_USERNAME \
  --docker-registry-server-password $ACR_PASSWORD

# Test staging environment
STAGING_URL=$(az webapp show \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --slot staging \
  --query defaultHostName \
  --output tsv)

curl https://$STAGING_URL

# Swap staging to production (instant)
az webapp deployment slot swap \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --slot staging \
  --target-slot production

# Rollback if needed (swap back)
az webapp deployment slot swap \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --slot production \
  --target-slot staging
```

## Troubleshooting

### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| **Container fails to start** | Wrong port | Set `WEBSITES_PORT` to match container |
| **Image pull fails** | ACR authentication | Verify ACR credentials in app settings |
| **Application not responding** | Container crash | Check logs with `az webapp log tail` |
| **Slow startup** | Large image | Optimize with multi-stage builds |
| **Memory errors** | Insufficient resources | Upgrade App Service Plan tier |

### Diagnostic Commands

```bash
# Check application status
az webapp show \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --query state

# View container settings
az webapp config container show \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP

# Restart application
az webapp restart \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP

# SSH into container (for debugging)
az webapp ssh \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP

# View deployment history
az webapp deployment list-publishing-profiles \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP
```

## Complete Deployment Script

```bash
#!/bin/bash
# complete-deployment.sh - End-to-end deployment

# Configuration
RESOURCE_GROUP="myResourceGroup"
LOCATION="eastus"
ACR_NAME="myregistry$(date +%s)"
APP_NAME="mywebapp$(date +%s)"
PLAN_NAME="myAppServicePlan"
IMAGE_NAME="myapp"
IMAGE_TAG="v1"

# 1. Create resources
echo "Creating Azure resources..."
az group create --name $RESOURCE_GROUP --location $LOCATION
az acr create --resource-group $RESOURCE_GROUP --name $ACR_NAME --sku Basic
az acr update --name $ACR_NAME --admin-enabled true
az appservice plan create --name $PLAN_NAME --resource-group $RESOURCE_GROUP --is-linux --sku S1

# 2. Build and push image
echo "Building container image..."
az acr build --registry $ACR_NAME --image $IMAGE_NAME:$IMAGE_TAG --file Dockerfile .

# 3. Get ACR credentials
ACR_LOGIN_SERVER=$(az acr show --name $ACR_NAME --query loginServer -o tsv)
ACR_USERNAME=$(az acr credential show --name $ACR_NAME --query username -o tsv)
ACR_PASSWORD=$(az acr credential show --name $ACR_NAME --query passwords[0].value -o tsv)

# 4. Create web app
echo "Creating web app..."
az webapp create \
  --resource-group $RESOURCE_GROUP \
  --plan $PLAN_NAME \
  --name $APP_NAME \
  --deployment-container-image-name $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG

# 5. Configure ACR integration
az webapp config container set \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --docker-custom-image-name $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG \
  --docker-registry-server-url https://$ACR_LOGIN_SERVER \
  --docker-registry-server-user $ACR_USERNAME \
  --docker-registry-server-password $ACR_PASSWORD

# 6. Enable continuous deployment
az webapp deployment container config \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --enable-cd true

# 7. Configure logging
az webapp log config \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --docker-container-logging filesystem

# 8. Get application URL
APP_URL=$(az webapp show --name $APP_NAME --resource-group $RESOURCE_GROUP --query defaultHostName -o tsv)

echo "========================================="
echo "Deployment Complete!"
echo "Application URL: https://$APP_URL"
echo "========================================="
echo "View logs: az webapp log tail --name $APP_NAME --resource-group $RESOURCE_GROUP"
```

## Best Practices

### Security

```bash
# Use managed identity instead of ACR admin credentials
az webapp identity assign \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP

# Grant ACR pull permissions
PRINCIPAL_ID=$(az webapp identity show --name $APP_NAME --resource-group $RESOURCE_GROUP --query principalId -o tsv)
ACR_ID=$(az acr show --name $ACR_NAME --query id -o tsv)

az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role AcrPull \
  --scope $ACR_ID

# Remove admin credentials requirement
az webapp config container set \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --docker-custom-image-name $ACR_LOGIN_SERVER/myapp:v1 \
  --docker-registry-server-url https://$ACR_LOGIN_SERVER
```

### Performance

- Use multi-stage builds to minimize image size
- Enable HTTP/2 and compression
- Configure CDN for static assets
- Use Application Insights for monitoring
- Set up health check endpoints

### Cost Optimization

```bash
# Use appropriate App Service Plan tier
# Basic (B1): $13/month - Development
# Standard (S1): $70/month - Production with staging
# Premium (P1v2): $85/month - High traffic

# Stop app when not needed (dev/test)
az webapp stop --name $APP_NAME --resource-group $RESOURCE_GROUP

# Auto-scale to handle traffic spikes efficiently
# Scale based on CPU, memory, or custom metrics
```

## Critical Notes

🎯 **Port Configuration**: Set `WEBSITES_PORT` to match your container's listening port—App Service won't detect non-standard ports automatically.

💡 **Continuous Deployment**: Enable CD webhook for automatic deployments when pushing new images to ACR—zero-touch deployment pipeline.

⚠️ **ACR Authentication**: Use managed identity in production instead of admin credentials—more secure and no password rotation needed.

📊 **Deployment Slots**: Always test in staging slot before production swap—instant rollback if issues detected.

🔄 **Health Checks**: Implement `/health` endpoint in your application—App Service uses it to verify container health.

✨ **Monitoring**: Enable Application Insights from day one—critical for debugging production issues.

## Quick Reference

### Essential Commands

```bash
# Deploy new version
az acr build --registry $ACR_NAME --image myapp:v1 .

# View logs
az webapp log tail --name $APP_NAME --resource-group $RESOURCE_GROUP

# Restart app
az webapp restart --name $APP_NAME --resource-group $RESOURCE_GROUP

# Scale instances
az webapp scale --name $APP_NAME --resource-group $RESOURCE_GROUP --instance-count 3

# Swap slots
az webapp deployment slot swap --name $APP_NAME --resource-group $RESOURCE_GROUP --slot staging
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/design-container-build-strategy/8-deploy-docker-containers-to-azure-app-service-web-apps)
