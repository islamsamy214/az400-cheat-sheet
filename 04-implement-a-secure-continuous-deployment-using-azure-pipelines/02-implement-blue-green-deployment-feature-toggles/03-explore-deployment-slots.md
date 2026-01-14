# Explore Deployment Slots

**Duration**: 3 minutes

## Overview

Azure cloud platform integration simplifies **blue-green deployment implementation** through managed infrastructure services that eliminate custom code development and manual environment configuration requirements.

**Azure App Service** provides native **deployment slot functionality** for web application blue-green deployment patterns.

## What Are Deployment Slots?

Deployment slots constitute **Azure App Service features** implementing isolated live application instances with dedicated hostnames supporting independent configuration management.

### Key Characteristics
- 🎯 **Isolated live application instances** within single App Service
- 🌐 **Dedicated hostnames** per slot (e.g., `myapp-staging.azurewebsites.net`)
- ⚙️ **Independent configuration** management per slot
- 🔄 **Seamless swap operations** for traffic switching

## Slot Types

### Production Slot
- **Primary environment** serving active user requests
- Default slot created with every App Service
- Public-facing hostname: `myapp.azurewebsites.net`
- Serves live production traffic

### Staging Deployment Slots
- **Non-production environments** for testing and validation
- Enable comprehensive validation before production promotion
- Examples: Development, Testing, Staging, QA
- Custom hostnames: `myapp-staging.azurewebsites.net`, `myapp-dev.azurewebsites.net`

## Deployment Workflow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Deploy new version to STAGING slot                       │
│    URL: myapp-staging.azurewebsites.net                     │
├─────────────────────────────────────────────────────────────┤
│ 2. Test in staging environment                              │
│    - Smoke tests, integration tests                         │
│    - Manual QA validation                                   │
│    - Performance testing                                    │
├─────────────────────────────────────────────────────────────┤
│ 3. Swap staging → production                                │
│    - Internal IP address exchange                           │
│    - Zero-downtime traffic transition                       │
├─────────────────────────────────────────────────────────────┤
│ 4. Production now runs new version                          │
│    URL: myapp.azurewebsites.net (updated)                   │
├─────────────────────────────────────────────────────────────┤
│ 5. Previous version now in staging slot (rollback ready)   │
│    URL: myapp-staging.azurewebsites.net (old version)       │
└─────────────────────────────────────────────────────────────┘
```

## Multiple Slot Configurations

Azure App Service supports **multiple deployment slots** per App Service, enabling sophisticated deployment pipelines.

### Example Configuration
```
myapp.azurewebsites.net                 → Production slot (v3.0 - live)
myapp-staging.azurewebsites.net         → Staging slot (v3.1 - testing)
myapp-dev.azurewebsites.net             → Dev slot (v3.2-beta - development)
myapp-qa.azurewebsites.net              → QA slot (v3.1 - quality assurance)
```

### Environment-Specific Deployments
- **Development slot**: Active development, latest commits
- **Testing slot**: Integration testing, automated test suites
- **QA slot**: Manual quality assurance, user acceptance testing
- **Staging slot**: Pre-production validation, final approval
- **Production slot**: Live traffic, stable releases

## Swap Operations

### What Is a Swap?
Slot swap functionality delivers **zero-downtime deployment capabilities** through seamless traffic redirection mechanisms that preserve all active requests without connection drops during swap operation execution.

### How Swaps Work
Environment swap operations execute through **internal IP address exchange** between slots, enabling seamless traffic transition.

```
BEFORE SWAP:
Production slot → IP Address A (v1.0, serving traffic)
Staging slot   → IP Address B (v2.0, idle)

DURING SWAP:
Azure internally exchanges IP addresses

AFTER SWAP:
Production slot → IP Address B (v2.0, serving traffic) ✅
Staging slot   → IP Address A (v1.0, idle, rollback ready)
```

### Swap Behavior
- ⚡ **Instant traffic redirection**: No DNS propagation delay
- 🔄 **Active request preservation**: Existing connections maintained
- 🛡️ **Zero dropped connections**: Seamless transition
- ⏱️ **App warm-up**: Azure warms up target slot before cutover (optional)

### Swap with Preview
Azure provides **swap with preview** (multi-phase swap) for validation:

**Phase 1**: Azure applies production settings to staging slot
- Restart staging slot with production configuration
- Validate configuration changes before traffic switch

**Phase 2**: Complete the swap
- Switch traffic after validation
- Provides safety checkpoint

## Configuration Management

### Slot-Specific Settings
Some settings remain **slot-specific** (don't swap):
- Publishing endpoints
- Custom domain names
- SSL certificates and bindings
- Scale settings
- WebJobs schedulers

### Swappable Settings
Most settings **swap with the slot**:
- App settings (unless marked "slot setting")
- Connection strings (unless marked "slot setting")
- Handler mappings
- Monitoring and diagnostic settings

### Sticky Settings
Mark settings as **"slot setting"** to prevent swapping:
```bash
# Azure CLI example: Mark app setting as sticky
az webapp config appsettings set \
  --name myapp \
  --resource-group myResourceGroup \
  --slot staging \
  --settings API_KEY=abc123 \
  --slot-settings API_KEY
```

## Azure CLI Commands

### Create Deployment Slot
```bash
# Create staging slot
az webapp deployment slot create \
  --name myapp \
  --resource-group myResourceGroup \
  --slot staging
```

### Swap Slots
```bash
# Swap staging → production
az webapp deployment slot swap \
  --name myapp \
  --resource-group myResourceGroup \
  --slot staging \
  --target-slot production
```

### Auto-Swap (for CI/CD)
```bash
# Enable auto-swap on slot
az webapp deployment slot auto-swap \
  --name myapp \
  --resource-group myResourceGroup \
  --slot staging \
  --target-slot production
```

## Benefits of Deployment Slots

### 🚀 Zero-Downtime Deployments
- No maintenance windows required
- Seamless user experience
- Business continuity maintained

### 🧪 Pre-Production Validation
- Test in production-equivalent environment
- Validate integrations before cutover
- Reduce production failures

### ⚡ Instant Rollback
- Swap back to previous slot within seconds
- No redeployment needed
- Minimal MTTR (Mean Time To Recovery)

### 🛡️ Risk Mitigation
- Isolated environments prevent production interference
- Multi-stage validation (dev → test → staging → production)
- Phased rollout capability

## When to Use Deployment Slots

### ✅ Ideal For
- **Web applications**: App Service, Azure Functions
- **High-availability requirements**: Zero-downtime mandates
- **Frequent deployments**: CI/CD pipelines with automated testing
- **Staged rollouts**: Multi-environment validation workflows

### Limitations
- **App Service Plans**: Standard tier or higher required (not Free/Shared)
- **Slot limits**: Maximum slots vary by App Service Plan tier
  - Standard: 5 slots
  - Premium: 20 slots
- **Cost**: Each slot consumes App Service Plan resources

## Quick Reference

### Key Features
- 🎯 Isolated live application instances per slot
- 🌐 Dedicated hostnames with custom DNS support
- ⚙️ Independent configuration management
- 🔄 Seamless swap operations with IP address exchange
- ⚡ Zero-downtime traffic transition

### Swap Operations
- Internal IP address exchange (no DNS changes)
- Active request preservation
- Optional warm-up phase before cutover
- Swap with preview for validation checkpoints

### Configuration
- Slot-specific settings remain isolated
- Swappable settings move with slot
- Sticky settings marked as "slot setting"

### Critical Notes
- ⚠️ **Requires Standard tier or higher** App Service Plan
- 💡 **Use swap with preview** for critical production deployments
- 🎯 **Mark sensitive settings as slot-specific** (e.g., API keys, connection strings)
- 📊 **Monitor after swap** to detect issues quickly
- 🔄 **Keep previous version in slot** for 24-48 hours (rollback safety)

## Additional Resources

- 📖 [Set up Staging Environments in Azure App Service](https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots)
- 📖 [Considerations on using Deployment Slots in your DevOps Pipeline](https://blogs.msdn.microsoft.com/devops/2017/04/10/considerations-on-using-deployment-slots-in-your-devops-pipeline/)
- 📖 [What happens during a swap](https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots)

---

**Next**: Learn about feature toggles for runtime feature control →

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-blue-green-deployment-feature-toggles/3-explore-deployment-slots)
