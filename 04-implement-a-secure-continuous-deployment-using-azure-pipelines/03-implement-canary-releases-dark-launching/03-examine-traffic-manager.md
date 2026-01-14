# Examine Traffic Manager

**Duration**: 4 minutes

## Overview

Azure Web App **deployment slot functionality** enables rapid version switching through seamless swap operations, providing foundational blue-green deployment capabilities.

Advanced traffic distribution requirements demand **granular control beyond basic slot swapping**, necessitating sophisticated traffic management solutions for progressive rollout strategies.

**Azure Traffic Manager** delivers enterprise-grade traffic orchestration capabilities enabling fine-grained distribution control across deployment targets.

## Azure Traffic Manager

Azure Traffic Manager implements **DNS-based traffic load balancing** delivering optimal service distribution across global Azure regions through intelligent routing algorithms that maximize availability and minimize latency.

### How It Works

**DNS-layer traffic direction** enables client request routing to optimal service endpoints based on configurable traffic-routing methodologies and continuous endpoint health validation.

```
User Request → DNS Query → Traffic Manager → Route to Endpoint
                                ↓
                        Health Check + Routing Method
                                ↓
                    app-v1.azure.net (90%) or app-v2.azure.net (10%)
```

### Key Capabilities

#### 1. Flexible Endpoint Definitions
Endpoint definitions encompass **internet-accessible services** hosted within Azure infrastructure or external platforms, providing deployment flexibility and hybrid architecture support.

**Supported Endpoint Types**:
- ☁️ **Azure endpoints**: App Service, VMs, Public IPs
- 🌐 **External endpoints**: On-premises servers, third-party services
- 🔗 **Nested endpoints**: Other Traffic Manager profiles (hierarchical routing)

#### 2. Continuous Health Monitoring
Comprehensive endpoint monitoring frameworks support automatic failover orchestration and disaster recovery scenarios.

**Health Probes**:
- HTTP/HTTPS endpoint checks
- Configurable probe interval (10-30 seconds)
- Automatic endpoint exclusion when unhealthy
- Degraded state handling

#### 3. Regional Failure Resilience
Regional failure resilience ensures continued operation during catastrophic events including **complete Azure region outages** through intelligent endpoint failover and geographic redundancy.

## Traffic Routing Methods

Traffic Manager provides **six traffic distribution methodologies** supporting diverse architectural requirements and operational patterns.

### 1. Priority Routing
**Primary endpoint traffic concentration** with automatic failover to backup endpoints during unavailability events.

**Use Case**: Disaster recovery, active-passive failover

```yaml
endpoints:
  - name: primary
    priority: 1    # Primary endpoint
    location: East US
  - name: secondary
    priority: 2    # Failover endpoint
    location: West Europe
```

**Behavior**: All traffic → Primary. If primary fails → Secondary.

### 2. Weighted Distribution ⭐
**Configurable proportional traffic allocation** across endpoint sets enabling gradual rollout and A/B testing scenarios.

**Use Case**: **Canary releases**, A/B testing, gradual migration

```yaml
endpoints:
  - name: stable-version
    weight: 90     # 90% of traffic
    location: East US
  - name: canary-version
    weight: 10     # 10% of traffic (canary)
    location: East US
```

**Behavior**: 90% traffic → Stable, 10% traffic → Canary

### 3. Performance-Based Routing
**Geographic proximity optimization** directing users to lowest-latency endpoints for optimal response times.

**Use Case**: Global applications, latency-sensitive services

**Behavior**: User in Europe → European endpoint (lowest latency)

### 4. Geographic Routing
**DNS query origin-based endpoint selection** supporting data sovereignty compliance, content localization, and regional traffic analytics.

**Use Case**: GDPR compliance, content localization, regional services

```yaml
geoMapping:
  - region: Europe
    endpoint: eu-app.azure.net  # EU data stays in EU
  - region: North America
    endpoint: us-app.azure.net
```

### 5. MultiValue Responses
**IPv4/IPv6 endpoint aggregation** returning all healthy endpoints for client-side selection strategies.

**Use Case**: DNS-based load balancing, client-side failover

**Behavior**: Returns up to 8 healthy endpoints in DNS response

### 6. Subnet-Based Routing
**Source IP address range mapping** to specific endpoints enabling network topology-aware traffic distribution.

**Use Case**: Internal users → staging, external → production

```yaml
subnetMapping:
  - subnet: 10.0.0.0/24       # Internal network
    endpoint: staging.azure.net
  - subnet: 0.0.0.0/0         # All others
    endpoint: production.azure.net
```

## Weighted Distribution for Canary Releases

**Weighted distribution methodology predominates** in Continuous Delivery implementations, enabling **percentage-based traffic allocation** essential for canary releases and progressive rollout strategies.

### Configuration Example

```bash
# Create Traffic Manager profile
az network traffic-manager profile create \
  --name myapp-tm \
  --resource-group myResourceGroup \
  --routing-method Weighted \
  --unique-dns-name myapp-global

# Add stable endpoint (90% weight)
az network traffic-manager endpoint create \
  --name stable \
  --profile-name myapp-tm \
  --resource-group myResourceGroup \
  --type azureEndpoints \
  --target-resource-id /subscriptions/.../app-stable \
  --weight 90

# Add canary endpoint (10% weight)
az network traffic-manager endpoint create \
  --name canary \
  --profile-name myapp-tm \
  --resource-group myResourceGroup \
  --type azureEndpoints \
  --target-resource-id /subscriptions/.../app-canary \
  --weight 10
```

### Dynamic Weight Adjustment

```bash
# Expand canary to 25%
az network traffic-manager endpoint update \
  --name stable \
  --profile-name myapp-tm \
  --resource-group myResourceGroup \
  --weight 75

az network traffic-manager endpoint update \
  --name canary \
  --profile-name myapp-tm \
  --resource-group myResourceGroup \
  --weight 25
```

## Traffic Routing Exclusion

> **Note**: Traffic routing **exclusively targets available endpoints**, automatically excluding unhealthy or unreachable services from distribution algorithms.

**Health Check Behavior**:
```
Endpoint Health Check:
✅ Healthy → Included in routing (receives traffic)
⚠️ Degraded → Included with warning (receives traffic)
❌ Stopped → Excluded from routing (no traffic)
🔄 Checking → Temporary exclusion (probing)
```

## Controlling Your Canary Release

Integrated **feature toggle, deployment slot, and Traffic Manager orchestration** delivers comprehensive traffic flow control enabling sophisticated canary release implementations.

### Deployment Workflow

**Step 1: Feature Deployment**
```
Deploy new functionality to isolated deployment slots or dedicated application instances.
```

**Step 2: Validation Gate**
```
Deployment verification confirming successful artifact installation before activation.

# Smoke test staging slot
curl https://myapp-staging.azurewebsites.net/health
# Expected: {"status": "healthy", "version": "v2.0"}
```

**Step 3: Traffic Allocation**
```
Percentage-based traffic distribution configuration targeting minimal user cohort exposure.

# Start with 5% canary
az network traffic-manager endpoint update \
  --name canary \
  --weight 5
```

**Step 4: Behavioral Monitoring**
```
Application Insights integration enabling comprehensive performance and stability telemetry collection.

# Monitor metrics in Application Insights
- Request rate: stable vs canary
- Response time: p50, p95, p99
- Error rate: 2xx, 4xx, 5xx
- Custom events: business metrics
```

### Automated Canary Pipeline

```yaml
# Azure Pipelines canary deployment
stages:
- stage: DeployCanary
  jobs:
  - deployment: CanaryDeployment
    environment: production-canary
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Azure-Connection'
              appName: 'myapp'
              slotName: 'canary'
              package: '$(Pipeline.Workspace)/drop/*.zip'
          
          - task: AzureCLI@2
            displayName: 'Route 5% traffic to canary'
            inputs:
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az network traffic-manager endpoint update \
                  --name canary \
                  --profile-name myapp-tm \
                  --resource-group myResourceGroup \
                  --weight 5

- stage: MonitorCanary
  dependsOn: DeployCanary
  jobs:
  - job: CanaryAnalysis
    steps:
    - task: AzureMonitor@1
      inputs:
        queryDuration: '2h'
        metrics:
          - errorRate
          - responseTime
        thresholds:
          errorRate: 0.5%
          responseTime_p95: 500ms
        onFailure: 'rollback'
        onSuccess: 'expand'
```

## Traffic Manager vs. Load Balancer

| Feature | Traffic Manager | Azure Load Balancer |
|---------|----------------|---------------------|
| **Layer** | DNS (Layer 7) | Network (Layer 4) |
| **Scope** | Global (multi-region) | Regional (single region) |
| **Routing** | 6 methods (weighted, priority, etc.) | Hash-based distribution |
| **Health Check** | HTTP/HTTPS probes | TCP/HTTP probes |
| **Failover** | DNS TTL dependent (5-60s) | Instant (sub-second) |
| **Use Case** | Canary, DR, geo-routing | High availability, scale |
| **Cost** | Per DNS query | Per rule + data processed |

## Quick Reference

### Configuration Steps
1. **Create Traffic Manager profile** with Weighted routing
2. **Add endpoints** (stable + canary) with initial weights (90/10)
3. **Configure health checks** (HTTP/HTTPS probes)
4. **Deploy application** to both endpoints
5. **Update DNS** to point to Traffic Manager FQDN
6. **Monitor metrics** continuously
7. **Adjust weights** based on canary performance (10 → 25 → 50 → 100)

### Key Features
- 🌐 DNS-based global traffic distribution
- ⚖️ Weighted routing for percentage allocation
- 🏥 Automatic health monitoring and exclusion
- 🌍 Multi-region support with geo-redundancy
- 📊 Integration with Azure Monitor and Application Insights

### Critical Notes
- ⚠️ **DNS TTL affects failover speed** (typically 5-60 seconds)
- 💡 **Use weighted routing** for canary releases (not priority)
- 🎯 **Monitor both endpoints** independently (compare metrics)
- 📊 **Gradual weight adjustment** (don't jump from 10% to 100%)
- 🔄 **Plan rollback procedure** before adjusting weights

## Additional Resources

- 📖 [What is Traffic Manager?](https://learn.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview)
- 📖 [How Traffic Manager works](https://learn.microsoft.com/en-us/azure/traffic-manager/traffic-manager-how-it-works)
- 📖 [Traffic Manager Routing Methods](https://learn.microsoft.com/en-us/azure/traffic-manager/traffic-manager-routing-methods)

---

**Next**: Understand dark launching for hidden feature validation →

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-canary-releases-dark-launching/3-examine-traffic-manager)
