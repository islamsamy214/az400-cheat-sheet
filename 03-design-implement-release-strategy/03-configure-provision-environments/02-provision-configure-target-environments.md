# Provision and Configure Target Environments

## Overview
Release pipelines deploy software to **target environments**, which include not just application code but also the **supporting infrastructure**. Continuous Delivery requires **Infrastructure as Code (IaC)** integration for automated environment provisioning.

**Key Principle**: Infrastructure provisioning must be integrated into release pipelines and maintained in source control.

## Target Environment Types

### 🖥️ 1. On-Premises Servers

**Characteristics**:
- Pre-existing hardware and OS
- Physical or virtual machines (Hyper-V, VMware)
- Infrastructure already provisioned
- Focus on **application deployment**, not infrastructure creation

#### Configuration Management

**PowerShell Desired State Configuration (DSC)**:
```powershell
# WebServerConfig.ps1
Configuration WebServerConfig {
    Node "WebServer01" {
        WindowsFeature IIS {
            Ensure = "Present"
            Name = "Web-Server"
        }
        
        File WebContent {
            Ensure = "Present"
            DestinationPath = "C:\\inetpub\\wwwroot\\app"
            SourcePath = "\\\\FileShare\\Deployment\\app"
            Type = "Directory"
            Recurse = $true
        }
        
        Script RestartIIS {
            GetScript = { @{ Result = "" } }
            TestScript = { $false }
            SetScript = { iisreset }
        }
    }
}
```

**Benefits**:
- **Configuration drift detection**: DSC ensures actual state matches desired state
- **Automatic remediation**: DSC corrects configuration drift
- **Version control**: DSC scripts stored in source control
- **Repeatability**: Same configuration across environments

#### Pipeline Integration

```yaml
# Azure Pipelines - On-Premises Deployment
steps:
  - task: PowerShellOnTargetMachines@3
    displayName: 'Apply DSC Configuration'
    inputs:
      Machines: 'WebServer01,WebServer02'
      UserName: '$(DeploymentUser)'
      UserPassword: '$(DeploymentPassword)'
      InlineScript: |
        .\\WebServerConfig.ps1
        Start-DscConfiguration -Path .\\WebServerConfig -Wait -Verbose
```

---

### ☁️ 2. Infrastructure as a Service (IaaS)

**Characteristics**:
- Cloud-based virtual machines
- Dynamic infrastructure provisioning
- On-demand resource creation
- Infrastructure as Code (IaC) practices

#### Infrastructure as Code Tools

| Tool | Best For | Format |
|------|----------|--------|
| **ARM Templates** | Azure-native | JSON |
| **Bicep** | Azure (modern) | Bicep DSL |
| **Terraform** | Multi-cloud | HCL |
| **Azure CLI** | Scripting | Shell |

#### Example: ARM Template

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "myVM"
    },
    "adminUsername": {
      "type": "string"
    },
    "adminPassword": {
      "type": "securestring"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-03-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_DS1_v2"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2019-Datacenter",
            "version": "latest"
          }
        }
      }
    }
  ]
}
```

#### Pipeline Integration

```yaml
# Azure Pipelines - IaaS Provisioning + Deployment
stages:
  - stage: ProvisionInfrastructure
    jobs:
      - job: DeployARMTemplate
        steps:
          - task: AzureResourceManagerTemplateDeployment@3
            inputs:
              deploymentScope: 'Resource Group'
              azureResourceManagerConnection: 'Azure-Connection'
              subscriptionId: '$(SubscriptionId)'
              resourceGroupName: 'myapp-rg'
              location: 'East US'
              templateLocation: 'Linked artifact'
              csmFile: '$(Build.SourcesDirectory)/infrastructure/vm-template.json'
              csmParametersFile: '$(Build.SourcesDirectory)/infrastructure/vm-parameters.json'
              deploymentMode: 'Incremental'
              
  - stage: DeployApplication
    dependsOn: ProvisionInfrastructure
    jobs:
      - deployment: DeployApp
        environment: production-vms
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureFileCopy@4
                  inputs:
                    azureSubscription: 'Azure-Connection'
                    SourcePath: '$(Pipeline.Workspace)/drop/**'
                    Destination: 'AzureVMs'
                    storage: 'mystorageaccount'
                    resourceGroup: 'myapp-rg'
```

**Workflow**:
```
1. Provision infrastructure (VMs, networks, storage)
2. Wait for resources to be ready
3. Deploy application code
4. Configure application settings
```

---

### 🚀 3. Platform as a Service (PaaS)

**Characteristics**:
- Fully managed hosting
- Cloud provider manages infrastructure
- Focus **exclusively on application code**
- No server management required

#### PaaS Examples

| Service | Type | Use Case |
|---------|------|----------|
| **Azure App Service** | Web hosting | ASP.NET, Node.js, Python apps |
| **Azure Functions** | Serverless | Event-driven microservices |
| **Azure SQL Database** | Managed database | Relational data storage |
| **Azure Container Apps** | Containers | Microservices, APIs |

#### Example: Azure App Service Deployment

**Bicep Template** (Infrastructure):
```bicep
// appservice.bicep
param location string = resourceGroup().location
param appName string = 'myapp'
param sku string = 'B1'

resource appServicePlan 'Microsoft.Web/serverfarms@2022-03-01' = {
  name: '${appName}-plan'
  location: location
  sku: {
    name: sku
    tier: 'Basic'
  }
  kind: 'linux'
}

resource webApp 'Microsoft.Web/sites@2022-03-01' = {
  name: '${appName}-web'
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    siteConfig: {
      linuxFxVersion: 'NODE|18-lts'
      appSettings: [
        {
          name: 'WEBSITE_NODE_DEFAULT_VERSION'
          value: '~18'
        }
      ]
    }
    httpsOnly: true
  }
}

output webAppUrl string = webApp.properties.defaultHostName
```

**Pipeline Integration**:
```yaml
# Azure Pipelines - PaaS Deployment
stages:
  - stage: DeployInfrastructure
    jobs:
      - job: DeployBicep
        steps:
          - task: AzureCLI@2
            displayName: 'Deploy App Service'
            inputs:
              azureSubscription: 'Azure-Connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az deployment group create \
                  --resource-group myapp-rg \
                  --template-file infrastructure/appservice.bicep \
                  --parameters appName=myapp sku=B1
                  
  - stage: DeployApplication
    dependsOn: DeployInfrastructure
    jobs:
      - deployment: DeployApp
        environment: production
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  displayName: 'Deploy to Azure App Service'
                  inputs:
                    azureSubscription: 'Azure-Connection'
                    appType: 'webAppLinux'
                    appName: 'myapp-web'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'
                    runtimeStack: 'NODE|18-lts'
```

**Benefits**:
- ✅ No server management
- ✅ Auto-scaling capabilities
- ✅ Built-in monitoring
- ✅ Security patching handled by platform
- ✅ Focus on application code

---

### 📦 4. Containers & Clusters

**Characteristics**:
- Container orchestration (Kubernetes, AKS)
- High availability and horizontal scalability
- Multiple machines working together
- Self-managed or managed clusters

#### Cluster Types

**Self-Managed IaaS Clusters**:
- Comprehensive lifecycle management required
- Provisioning templates
- Security patching
- Updates and maintenance
- Full control but high operational overhead

**Managed Cluster Services** (PaaS):
- Cloud provider handles infrastructure
- Users deploy applications
- Example: Azure Kubernetes Service (AKS)
- Automated cluster operations
- Enterprise-grade security

#### Example: AKS Deployment

**Bicep Template** (AKS Cluster):
```bicep
// aks-cluster.bicep
param clusterName string = 'myapp-aks'
param location string = resourceGroup().location
param nodeCount int = 3

resource aksCluster 'Microsoft.ContainerService/managedClusters@2023-01-01' = {
  name: clusterName
  location: location
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    dnsPrefix: '${clusterName}-dns'
    agentPoolProfiles: [
      {
        name: 'agentpool'
        count: nodeCount
        vmSize: 'Standard_DS2_v2'
        osType: 'Linux'
        mode: 'System'
      }
    ]
    networkProfile: {
      networkPlugin: 'azure'
      serviceCidr: '10.0.0.0/16'
      dnsServiceIP: '10.0.0.10'
    }
  }
}

output controlPlaneFQDN string = aksCluster.properties.fqdn
```

**Kubernetes Manifest**:
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry.azurecr.io/myapp:1.0.0
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: myapp
```

**Pipeline Integration**:
```yaml
# Azure Pipelines - AKS Deployment
stages:
  - stage: Build
    jobs:
      - job: BuildImage
        steps:
          - task: Docker@2
            displayName: 'Build Docker image'
            inputs:
              containerRegistry: 'ACR-Connection'
              repository: 'myapp'
              command: 'buildAndPush'
              Dockerfile: '**/Dockerfile'
              tags: '$(Build.BuildId)'
              
  - stage: Deploy
    jobs:
      - deployment: DeployToAKS
        environment: production-aks
        strategy:
          runOnce:
            deploy:
              steps:
                - task: KubernetesManifest@0
                  displayName: 'Deploy to Kubernetes'
                  inputs:
                    action: 'deploy'
                    kubernetesServiceConnection: 'AKS-Connection'
                    namespace: 'production'
                    manifests: |
                      $(Pipeline.Workspace)/manifests/deployment.yaml
                      $(Pipeline.Workspace)/manifests/service.yaml
                    containers: 'myregistry.azurecr.io/myapp:$(Build.BuildId)'
```

---

## Service Connections

**Purpose**: Secure authentication for pipeline access to external resources.

### What Are Service Connections?

Service connections provide **centralized credential management** for:
- Azure subscriptions (Azure Resource Manager)
- Container registries (Docker, ACR)
- Kubernetes clusters
- AWS, GCP (multi-cloud)
- External APIs

### Authentication Methods

| Method | Use Case | Security |
|--------|----------|----------|
| **Service Principal** | Azure resources | ✅ Scoped permissions |
| **Managed Identity** | Azure-hosted agents | ✅✅ No credentials |
| **Certificate** | High-security scenarios | ✅✅ Certificate-based |
| **Personal Access Token** | DevOps services | ⚠️ User-scoped |

### Example: Service Connection Creation

```yaml
# Service Connection (configured in Azure DevOps UI):
Name: Azure-Connection
Type: Azure Resource Manager
Authentication: Service Principal (automatic)
Scope: Subscription or Resource Group
Permissions: Contributor role
```

**In Pipeline**:
```yaml
steps:
  - task: AzureWebApp@1
    inputs:
      azureSubscription: 'Azure-Connection'  # Reference by name
      appName: 'myapp'
```

**Benefits**:
- ✅ **Centralized**: One connection, many pipelines
- ✅ **Secure**: Credentials not embedded in code
- ✅ **Auditable**: Access logging and tracking
- ✅ **Scoped**: Principle of least privilege

---

## Infrastructure as Code Best Practices

### 1️⃣ Store in Source Control

```
Repository Structure:
myapp/
├── src/              # Application code
├── infrastructure/   # IaC templates
│   ├── bicep/
│   │   ├── main.bicep
│   │   └── modules/
│   ├── terraform/
│   └── arm/
└── pipelines/
    └── deploy.yml
```

### 2️⃣ Use Parameters for Environment Differences

```bicep
// main.bicep
param environment string = 'dev'
param skuName string = environment == 'production' ? 'P1v2' : 'B1'
param replicaCount int = environment == 'production' ? 3 : 1

resource appServicePlan 'Microsoft.Web/serverfarms@2022-03-01' = {
  name: 'myapp-${environment}-plan'
  sku: {
    name: skuName
  }
}
```

### 3️⃣ Validate Before Deployment

```yaml
steps:
  - task: AzureCLI@2
    displayName: 'Validate ARM Template'
    inputs:
      scriptType: 'bash'
      inlineScript: |
        az deployment group validate \
          --resource-group myapp-rg \
          --template-file main.bicep
```

### 4️⃣ Use Incremental Deployment Mode

```
Incremental: Adds/updates resources, preserves existing
Complete: Deletes resources not in template (dangerous!)

Default to: Incremental
```

---

## Quick Reference

| Environment Type | Infrastructure | Deployment Focus | Management |
|------------------|---------------|------------------|------------|
| **On-Premises** | Pre-existing | Application + Config | Self-managed |
| **IaaS** | Dynamic (VMs) | Infrastructure + App | Self-managed |
| **PaaS** | Managed | Application only | Provider-managed |
| **Containers** | Orchestrated | Containers + K8s | Self or managed |

---

**Learn More**:
- [PowerShell DSC Overview](https://learn.microsoft.com/en-us/powershell/dsc/overview/dscforengineers)
- [Azure Functions](https://azure.microsoft.com/services/functions)
- [Azure Resource Manager](https://learn.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview)

[Learn More: Original Unit](https://learn.microsoft.com/en-us/training/modules/configure-provision-environments/2-provision-configure-target-environments)
