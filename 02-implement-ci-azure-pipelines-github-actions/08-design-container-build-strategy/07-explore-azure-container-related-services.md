# Explore Azure Container-Related Services

Azure provides a comprehensive ecosystem of container services for various scenarios—from simple serverless containers to enterprise-grade Kubernetes orchestration. Understanding each service helps you choose the right solution for your workload.

## Azure Container Services Overview

```
Azure Container Ecosystem

┌───────────────────────────────────────────────────────────────┐
│  Container Registry (ACR)                                     │
│  Private Docker registry for all container images            │
└───────────────────────────────────────────────────────────────┘
                         ↓ Images stored
        ┌────────────────┼────────────────┬────────────────┐
        ↓                ↓                ↓                ↓
┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Azure     │  │   Azure     │  │   Azure     │  │   Azure     │
│  Container  │  │  Container  │  │     App     │  │ Kubernetes  │
│  Instances  │  │    Apps     │  │   Service   │  │   Service   │
│   (ACI)     │  │             │  │   (Web Apps)│  │    (AKS)    │
└─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘
  Serverless     Event-driven      PaaS with         Managed
  containers     microservices     containers      Kubernetes
```

## Service Comparison Matrix

| Feature | ACI | Container Apps | App Service | AKS | When to Use |
|---------|-----|----------------|-------------|-----|-------------|
| **Complexity** | Simple | Low | Low-Medium | High | ACI: Quick tasks; AKS: Full orchestration |
| **Orchestration** | None | Built-in | None | Kubernetes | Container Apps/AKS: Multi-container apps |
| **Scaling** | Manual | Auto (KEDA) | Auto | Auto | Container Apps: Event-driven; AKS: Complex |
| **Pricing** | Per second | Per use | Per instance | Per cluster | ACI: Short tasks; App Service: Steady |
| **Startup Time** | Fast (seconds) | Fast | Medium | Medium | ACI/Container Apps: Quick start |
| **Control** | Low | Medium | Medium | Full | AKS: Need full Kubernetes control |
| **Networking** | Basic VNet | Advanced | VNet integration | Full control | AKS: Complex networking |
| **Management** | Minimal | Low | Low | High | ACI: Zero management; AKS: Full control |

## Azure Container Instances (ACI)

### Overview

**Serverless container execution without VM or orchestrator management**—simplest way to run containers in Azure.

```
ACI Architecture

Request → ACI Container → Response
           ├── Hypervisor-level isolation
           ├── Linux or Windows support
           ├── Per-second billing
           └── VNet integration (optional)
```

### Key Features

**1. Hypervisor-Level Isolation**
- Each container group runs in isolated VM
- Stronger security than shared kernel
- Suitable for untrusted workloads

**2. Fast Startup**
- Containers start in seconds
- No cluster provisioning required
- On-demand execution

**3. Flexible Billing**
- Pay per second of execution
- Charged for CPU and memory allocation
- Ideal for short-lived tasks

**4. Networking Options**
- Public IP address (default)
- VNet integration (private connectivity)
- Port mapping support

### Use Cases

```bash
# Quick task execution
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --cpu 1 \
  --memory 1 \
  --ports 80 \
  --ip-address Public

# Batch processing
az container create \
  --resource-group myResourceGroup \
  --name batch-processor \
  --image myregistry.azurecr.io/batch-app:v1 \
  --restart-policy OnFailure \
  --environment-variables \
    'BATCH_SIZE=1000' \
    'OUTPUT_PATH=/output'

# VNet-integrated container
az container create \
  --resource-group myResourceGroup \
  --name private-container \
  --image myapp:latest \
  --vnet myVnet \
  --subnet mySubnet
```

**Best For**:
- ✅ Batch jobs and scheduled tasks
- ✅ CI/CD build agents
- ✅ Quick prototyping and testing
- ✅ Event-driven processing
- ✅ Short-lived workloads
- ❌ Long-running web applications (expensive)
- ❌ Complex multi-container applications

## Azure Kubernetes Service (AKS)

### Overview

**Managed Kubernetes service** that simplifies deploying and managing containerized applications using Kubernetes orchestration.

```
AKS Architecture

            ┌─────────────────────────────────┐
            │     Azure Kubernetes Service    │
            │  (Managed Control Plane - FREE) │
            └─────────────────────────────────┘
                          ↓
    ┌─────────────────────┼─────────────────────┐
    ↓                     ↓                     ↓
┌─────────┐         ┌─────────┐         ┌─────────┐
│  Node   │         │  Node   │         │  Node   │
│  Pool 1 │         │  Pool 2 │         │  Pool 3 │
│ (Linux) │         │(Windows)│         │(GPU)    │
└─────────┘         └─────────┘         └─────────┘
Pay only for worker nodes
```

### Key Features

**1. Automated Cluster Management**
- Automatic Kubernetes version upgrades
- Automated node patching and scaling
- Built-in monitoring and diagnostics

**2. Security and Compliance**
- Microsoft Entra ID (Azure AD) integration
- Role-Based Access Control (RBAC)
- Network policies for pod-to-pod security
- Azure Policy for governance

**3. Auto-Scaling**
```yaml
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**4. Node Pools**
```bash
# Create AKS cluster
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 3 \
  --enable-managed-identity \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 10

# Add Windows node pool
az aks nodepool add \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name winnp \
  --node-count 3 \
  --os-type Windows

# Add GPU node pool
az aks nodepool add \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name gpunp \
  --node-count 2 \
  --node-vm-size Standard_NC6
```

### AKS Features Matrix

| Feature | Description | Benefit |
|---------|-------------|---------|
| **Control Plane** | Free managed control plane | No cost for Kubernetes API server |
| **Node Pools** | Multiple node pool support | Mix Linux/Windows/GPU nodes |
| **Networking** | Azure CNI or Kubenet | Advanced networking with VNet integration |
| **Storage** | Azure Disks, Azure Files, Blob | Persistent storage options |
| **Monitoring** | Container Insights | Built-in monitoring and logging |
| **Security** | Azure AD + RBAC | Enterprise-grade access control |
| **Upgrades** | Automated upgrades | Stay current with Kubernetes versions |
| **Scaling** | Cluster autoscaler | Automatic node scaling |

**Best For**:
- ✅ Microservices architectures
- ✅ Large-scale containerized applications
- ✅ Multi-tenant applications
- ✅ Applications requiring full Kubernetes features
- ✅ Complex networking and storage requirements
- ❌ Simple single-container apps (overkill)
- ❌ Teams without Kubernetes expertise

## Azure Container Registry (ACR)

### Overview

**Private Docker registry** for storing and managing container images with enterprise features like geo-replication and security scanning.

```
ACR Workflow

Developer → Docker Push → ACR → Docker Pull → AKS/ACI/App Service
                          ├── Geo-replication
                          ├── Content trust
                          ├── Vulnerability scanning
                          └── Webhook triggers
```

### Key Features

**1. Private Registry**
```bash
# Create registry
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Premium \
  --location eastus

# Login to registry
az acr login --name myregistry

# Push image
docker tag myapp:v1 myregistry.azurecr.io/myapp:v1
docker push myregistry.azurecr.io/myapp:v1

# Pull image
docker pull myregistry.azurecr.io/myapp:v1
```

**2. Geo-Replication** (Premium SKU)
```bash
# Replicate registry to multiple regions
az acr replication create \
  --registry myregistry \
  --location westus

az acr replication create \
  --registry myregistry \
  --location westeurope
```

**Geo-Replication Benefits**:
```
            ┌─────────────┐
            │  ACR (East) │
            │   Primary   │
            └─────────────┘
                  ↓ Replicate
        ┌─────────┴─────────┐
        ↓                   ↓
┌─────────────┐     ┌─────────────┐
│  ACR (West) │     │ACR (Europe) │
│   Replica   │     │   Replica   │
└─────────────┘     └─────────────┘
        ↓                   ↓
   ┌────────┐         ┌────────┐
   │ AKS US │         │AKS EU  │
   └────────┘         └────────┘
   Fast local pulls   Fast local pulls
```

**3. Security Features**
```bash
# Enable content trust (image signing)
az acr config content-trust update \
  --name myregistry \
  --status enabled

# Vulnerability scanning (Microsoft Defender)
az acr config content-trust show \
  --name myregistry

# Network access control
az acr network-rule add \
  --name myregistry \
  --ip-address 203.0.113.10
```

**4. CI/CD Integration**
```yaml
# Azure Pipelines integration
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  imageRepository: 'myapp'
  containerRegistry: 'myregistry.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
  tag: '$(Build.BuildId)'

steps:
- task: Docker@2
  displayName: Build and push
  inputs:
    containerRegistry: 'myACRConnection'
    repository: $(imageRepository)
    command: 'buildAndPush'
    Dockerfile: $(dockerfilePath)
    tags: |
      $(tag)
      latest
```

### ACR SKU Comparison

| Feature | Basic | Standard | Premium |
|---------|-------|----------|---------|
| **Storage** | 10 GB | 100 GB | 500 GB |
| **Throughput** | Low | Medium | High |
| **Geo-replication** | ❌ | ❌ | ✅ |
| **Content trust** | ❌ | ❌ | ✅ |
| **Private link** | ❌ | ❌ | ✅ |
| **Customer-managed keys** | ❌ | ❌ | ✅ |
| **Pricing** | $ | $$ | $$$ |

**Best For**:
- ✅ All Azure container deployments (AKS, ACI, App Service)
- ✅ Multi-region deployments (geo-replication)
- ✅ Enterprise security requirements
- ✅ CI/CD pipelines
- ✅ Teams requiring private registries

## Azure Container Apps

### Overview

**Serverless container platform** with built-in auto-scaling, event-driven capabilities, and microservices support—no Kubernetes management required.

```
Container Apps Architecture

HTTP Request   ┌──────────────────┐
    ↓          │  Container Apps  │
Scale 0→N  →   │   Environment    │
               └──────────────────┘
                       ↓
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
   ┌────────┐     ┌────────┐     ┌────────┐
   │  App 1 │────→│  App 2 │────→│  App 3 │
   │(0-100) │     │(0-50)  │     │(0-30)  │
   └────────┘     └────────┘     └────────┘
   Event-driven scaling with KEDA
```

### Key Features

**1. Event-Driven Auto-Scaling (KEDA)**
```yaml
apiVersion: apps/v1alpha1
kind: ContainerApp
metadata:
  name: processor
spec:
  configuration:
    ingress:
      external: false
  template:
    containers:
    - name: processor
      image: myregistry.azurecr.io/processor:v1
      resources:
        cpu: 0.5
        memory: 1Gi
    scale:
      minReplicas: 0  # Scale to zero!
      maxReplicas: 30
      rules:
      - name: queue-scaling
        type: azure-queue
        metadata:
          queueName: tasks
          queueLength: '5'
```

**Scaling Triggers**:
- HTTP requests
- Azure Queue Storage
- Azure Service Bus
- Azure Event Hubs
- Kafka
- Custom metrics (Prometheus)

**2. Built-in Dapr Support**
```bash
# Enable Dapr for microservices communication
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myEnvironment \
  --image myregistry.azurecr.io/myapp:v1 \
  --enable-dapr \
  --dapr-app-id myapp \
  --dapr-app-port 3000
```

**3. Traffic Splitting (Blue-Green/Canary)**
```bash
# Create revision with traffic splitting
az containerapp revision copy \
  --name myapp \
  --resource-group myResourceGroup \
  --image myregistry.azurecr.io/myapp:v2

# Split traffic: 90% v1, 10% v2
az containerapp ingress traffic set \
  --name myapp \
  --resource-group myResourceGroup \
  --revision-weight myapp--v1=90 myapp--v2=10
```

**4. Service-to-Service Communication**
```
Container App Environment (shared network)

┌────────────────────────────────────────┐
│  Frontend App                          │
│  (external ingress)                    │
└────────────────────────────────────────┘
              ↓ Internal communication
┌────────────────────────────────────────┐
│  API App                               │
│  (internal ingress only)               │
└────────────────────────────────────────┘
              ↓
┌────────────────────────────────────────┐
│  Database Processor                    │
│  (no ingress, queue-triggered)         │
└────────────────────────────────────────┘
```

### Container Apps vs AKS

| Aspect | Container Apps | AKS |
|--------|----------------|-----|
| **Kubernetes** | Abstracted away | Full Kubernetes API |
| **Scaling** | Auto (scale to zero) | Manual + HPA |
| **Learning Curve** | Low | High |
| **Control** | Simplified | Full control |
| **Cost** | Pay per use | Pay for nodes (always running) |
| **Networking** | Built-in ingress | Configure ingress controller |
| **Use Case** | Event-driven microservices | Complex orchestration |

**Best For**:
- ✅ Event-driven microservices
- ✅ API backends with variable traffic
- ✅ Background job processors
- ✅ Scale-to-zero scenarios
- ✅ Teams wanting serverless containers
- ❌ Need Kubernetes-specific features (Operators, CRDs)
- ❌ Existing Kubernetes manifests (use AKS)

## Azure App Service (Web Apps for Containers)

### Overview

**Platform-as-a-Service (PaaS)** for hosting web applications with built-in support for custom Docker containers.

```
App Service Architecture

Developer → Push Container → ACR → App Service
                                    ├── Built-in CI/CD
                                    ├── Auto-scaling
                                    ├── Deployment slots
                                    └── Custom domains/SSL
```

### Key Features

**1. Custom Container Support**
```bash
# Create App Service with custom container
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name myWebApp \
  --deployment-container-image-name myregistry.azurecr.io/myapp:v1

# Enable continuous deployment (CD)
az webapp deployment container config \
  --name myWebApp \
  --resource-group myResourceGroup \
  --enable-cd true
```

**2. Multi-Platform Support**
- Linux containers (Docker)
- Windows containers
- Multi-container (Docker Compose)

**3. Deployment Slots**
```bash
# Create staging slot
az webapp deployment slot create \
  --name myWebApp \
  --resource-group myResourceGroup \
  --slot staging

# Deploy to staging
az webapp deployment container config \
  --name myWebApp \
  --resource-group myResourceGroup \
  --slot staging \
  --docker-custom-image-name myregistry.azurecr.io/myapp:v2

# Swap staging to production (zero downtime)
az webapp deployment slot swap \
  --name myWebApp \
  --resource-group myResourceGroup \
  --slot staging
```

**4. Built-in Features**
- Auto-scaling (scale up/out)
- Custom domains and SSL certificates
- Authentication/authorization (Microsoft Entra ID, Facebook, Google)
- Application Insights integration
- VNet integration
- Backup and restore

### App Service Plans

| Tier | Use Case | Features | Containers |
|------|----------|----------|------------|
| **Free/Shared** | Development | 1 GB storage, 60 min/day | ❌ |
| **Basic** | Development/Testing | Custom domains, manual scaling | ✅ |
| **Standard** | Production | Auto-scaling, staging slots, 50 GB | ✅ |
| **Premium** | High-traffic production | More scale, VNet integration | ✅ |

**Best For**:
- ✅ Web applications and APIs
- ✅ Teams familiar with PaaS
- ✅ Need deployment slots (staging/production)
- ✅ Require built-in authentication
- ✅ Simple container deployment
- ❌ Complex multi-container orchestration (use AKS)
- ❌ Event-driven scaling (use Container Apps)

## Service Selection Decision Tree

```
Start Here: What type of workload?

├─ Simple single container?
│  ├─ Short-lived task? → Azure Container Instances (ACI)
│  ├─ Web app? → Azure App Service
│  └─ Event-driven? → Azure Container Apps
│
├─ Multiple containers (microservices)?
│  ├─ Need Kubernetes? → Azure Kubernetes Service (AKS)
│  ├─ Want serverless? → Azure Container Apps
│  └─ Simple orchestration? → App Service (multi-container)
│
└─ Need private registry? → Azure Container Registry (ACR)
   (Use with any of the above)
```

## Deployment Example: Complete Workflow

```bash
# 1. Create Azure Container Registry
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Standard

# 2. Build and push image
az acr build \
  --registry myregistry \
  --image myapp:v1 \
  --file Dockerfile .

# 3a. Deploy to ACI (quick test)
az container create \
  --resource-group myResourceGroup \
  --name myapp-test \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-login-server myregistry.azurecr.io \
  --registry-username $(az acr credential show -n myregistry --query username -o tsv) \
  --registry-password $(az acr credential show -n myregistry --query passwords[0].value -o tsv)

# 3b. Deploy to Container Apps (production)
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myEnvironment \
  --image myregistry.azurecr.io/myapp:v1 \
  --target-port 80 \
  --ingress external \
  --min-replicas 0 \
  --max-replicas 10

# 3c. Deploy to AKS (enterprise)
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --attach-acr myregistry \
  --node-count 3

kubectl apply -f deployment.yaml
```

## Critical Notes

🎯 **ACI**: Best for short-lived tasks and quick testing—pay per second, start in seconds, no cluster management.

💡 **Container Apps**: Ideal for event-driven microservices—scale to zero, built-in Dapr, KEDA auto-scaling without Kubernetes complexity.

⚠️ **AKS**: Use when you need full Kubernetes features—best for complex orchestration, multi-tenant apps, and teams with Kubernetes expertise.

📊 **ACR**: Essential for all Azure container deployments—private registry with geo-replication, security scanning, and CI/CD integration.

🔄 **App Service**: Great for web apps with containers—built-in deployment slots, authentication, auto-scaling, and PaaS simplicity.

✨ **Service Selection**: Start with Container Apps or App Service for simplicity, move to AKS only when Kubernetes features are required.

## Quick Reference

### Service Quick Comparison

| Need | Service | Command |
|------|---------|---------|
| Run batch job | ACI | `az container create` |
| Deploy web app | App Service | `az webapp create` |
| Microservices (simple) | Container Apps | `az containerapp create` |
| Microservices (complex) | AKS | `az aks create` |
| Store images | ACR | `az acr create` |

### Pricing Tiers (Relative)

```
Cheapest → Most Expensive (for similar workload)

1. Container Apps (scale-to-zero) - Pay per use
2. ACI (short tasks) - Per second billing
3. App Service (Basic/Standard) - Per instance
4. AKS (small cluster) - Pay for nodes
5. App Service (Premium) - Higher per instance
6. AKS (large cluster) - More expensive nodes
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/design-container-build-strategy/7-explore-azure-container-related-services)
