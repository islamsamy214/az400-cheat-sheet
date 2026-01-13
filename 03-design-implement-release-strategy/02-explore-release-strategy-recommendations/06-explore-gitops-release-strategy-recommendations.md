# Explore GitOps Release Strategy and Recommendations

## Overview
**GitOps** is a software delivery approach using Git repositories as the single source of truth for infrastructure, configuration, and application code management. All system changes are implemented through Git commits and pull requests.

**Core Principle**: If it's not in Git, it doesn't exist in production.

## What Is GitOps?

```
Traditional:
Manual Changes → Production → Hope Git is updated

GitOps:
Git Commit → Automated Sync → Production
    ↑                              ↓
    └──────── Source of Truth ─────┘
```

### Core GitOps Principles

| Principle | Description |
|-----------|-------------|
| **Declarative** | Desired state expressed in configuration files |
| **Versioned** | All changes tracked in Git history |
| **Immutable** | Artifacts are never modified, only replaced |
| **Automated** | Changes automatically applied from Git |
| **Auditable** | Full audit trail in Git log |

## GitOps vs Traditional Deployment

| Aspect | Traditional | GitOps |
|--------|------------|--------|
| **Configuration** | Manual scripts, runbooks | Declarative YAML in Git |
| **Deployment** | Push-based (CI/CD pushes) | Pull-based (agent pulls from Git) |
| **State Management** | State unknown/drifted | Desired state in Git |
| **Rollback** | Manual revert process | Git revert + auto-sync |
| **Audit Trail** | External ticketing system | Git commit history |
| **Access Control** | VPN, jump boxes | Git permissions |

---

## Defining a Comprehensive GitOps Release Strategy

### 1️⃣ Declarative Configuration

**Definition**: Express desired system state through declarative configuration files.

#### Tools
- **PowerShell DSC** (Windows)
- **Ansible** (Cross-platform)
- **Terraform** (Infrastructure)
- **Kubernetes manifests** (Containers)

#### Example: Ansible Playbook

```yaml
# webserver.yml
---
- name: Configure web server
  hosts: webservers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present  # Declarative: "should be installed"
        
    - name: Copy website files
      copy:
        src: /app/dist/
        dest: /var/www/html/
        
    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started  # Declarative: "should be running"
        enabled: yes
```

**Declarative vs Imperative**:
```
Imperative (script):
1. apt-get install nginx
2. cp files /var/www/html
3. systemctl start nginx

Declarative (Ansible):
- nginx: state=present
- files: state=present
- nginx service: state=started

Benefit: Idempotent (safe to run multiple times)
```

---

### 2️⃣ Infrastructure as Code (IaC)

**Definition**: Define infrastructure components declaratively using code.

#### Tools

| Tool | Best For | Language |
|------|----------|----------|
| **Terraform** | Multi-cloud IaC | HCL |
| **Bicep** | Azure-native IaC | Bicep |
| **ARM Templates** | Azure IaC (legacy) | JSON |
| **Pulumi** | IaC with real languages | TypeScript, Python, C# |
| **CloudFormation** | AWS IaC | YAML/JSON |

#### Example: Bicep (Azure)

```bicep
// main.bicep
param location string = resourceGroup().location
param appName string = 'myapp'

// App Service Plan
resource appServicePlan 'Microsoft.Web/serverfarms@2022-03-01' = {
  name: '${appName}-plan'
  location: location
  sku: {
    name: 'B1'
    tier: 'Basic'
  }
}

// Web App
resource webApp 'Microsoft.Web/sites@2022-03-01' = {
  name: '${appName}-web'
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    httpsOnly: true
    siteConfig: {
      appSettings: [
        {
          name: 'WEBSITE_NODE_DEFAULT_VERSION'
          value: '18-lts'
        }
      ]
    }
  }
}

output webAppUrl string = webApp.properties.defaultHostName
```

**Deployment**:
```bash
# All in Git, deployed via pipeline
az deployment group create \
  --resource-group myapp-rg \
  --template-file main.bicep \
  --parameters appName=myapp
```

---

### 3️⃣ Version Control

**Definition**: Store all configuration, infrastructure, and application code in Git repositories.

#### Repository Structure (Mono-repo vs Multi-repo)

**Option 1: Mono-repo**
```
myapp/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── src/                  # Application code
│   ├── api/
│   └── web/
├── infrastructure/       # IaC
│   ├── terraform/
│   └── bicep/
├── config/               # Configuration
│   ├── dev/
│   ├── staging/
│   └── production/
└── k8s/                  # Kubernetes manifests
    ├── base/
    └── overlays/
```

**Option 2: Multi-repo**
```
myapp-code/              # Application repository
myapp-infrastructure/    # Infrastructure repository
myapp-config/            # Configuration repository (GitOps)
```

#### Branching Strategy

**Environment Branches**:
```
main → Production environment
staging → Staging environment
develop → Development environment
```

**GitOps Flow**:
```
Developer: 
1. Create feature branch
2. Open pull request to 'develop'
3. Merge → Auto-deploy to dev environment

Release Manager:
1. Merge 'develop' to 'staging'
2. Auto-deploy to staging
3. After validation, merge 'staging' to 'main'
4. Auto-deploy to production
```

---

### 4️⃣ Continuous Deployment (CD)

**Definition**: Automatically deploy changes based on Git commits and pull requests.

#### CI/CD Services

- **Azure Pipelines**
- **GitHub Actions**
- **GitLab CI/CD**
- **Jenkins**
- **CircleCI**

#### GitHub Actions Example

```yaml
# .github/workflows/gitops-deploy.yml
name: GitOps Deploy

on:
  push:
    branches:
      - main  # Production
      - staging
      - develop

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Determine environment
        id: env
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "environment=production" >> $GITHUB_OUTPUT
          elif [[ "${{ github.ref }}" == "refs/heads/staging" ]]; then
            echo "environment=staging" >> $GITHUB_OUTPUT
          else
            echo "environment=development" >> $GITHUB_OUTPUT
          fi
      
      - name: Deploy infrastructure
        run: |
          az deployment group create \
            --resource-group myapp-${{ steps.env.outputs.environment }}-rg \
            --template-file infrastructure/main.bicep
      
      - name: Deploy application
        run: |
          az webapp deployment source config-zip \
            --resource-group myapp-${{ steps.env.outputs.environment }}-rg \
            --name myapp-${{ steps.env.outputs.environment }} \
            --src dist/app.zip
```

**Trigger Flow**:
```
Git Push → GitHub → Workflow Triggered → Deploy to Environment
```

---

### 5️⃣ Automated Synchronization

**Definition**: Deployments triggered automatically when changes are pushed to Git repository.

#### GitOps Tools for Kubernetes

| Tool | Description | Pull-Based |
|------|-------------|------------|
| **Flux CD** | CNCF project, simple | ✅ |
| **Argo CD** | Rich UI, multi-cluster | ✅ |
| **Jenkins X** | Jenkins-based GitOps | ❌ (push-based) |

#### Flux CD Example

**Setup**:
```bash
# Install Flux in cluster
flux install

# Connect to Git repository
flux create source git myapp \
  --url=https://github.com/myorg/myapp-config \
  --branch=main \
  --interval=1m

# Create Kustomization (auto-sync)
flux create kustomization myapp \
  --source=myapp \
  --path="./k8s/production" \
  --prune=true \
  --interval=5m
```

**How It Works**:
```
1. Flux polls Git repository every 1 minute
2. Detects changes in k8s/production/
3. Applies changes to Kubernetes cluster
4. Reconciles every 5 minutes (ensures no drift)
```

**Example Kubernetes Manifest** (in Git):
```yaml
# k8s/production/deployment.yaml
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
        image: myregistry.azurecr.io/myapp:1.2.3
        ports:
        - containerPort: 80
```

**Update Process**:
```
1. Update image tag: 1.2.3 → 1.2.4
2. Commit to Git
3. Flux detects change within 1 minute
4. Applies new deployment
5. Kubernetes rolls out update
```

---

### 6️⃣ Immutable Infrastructure

**Definition**: Infrastructure and containers are disposable artifacts, never modified in place.

**Traditional Mutable**:
```
Server-01: 
- SSH into server
- apt-get update
- apt-get upgrade
- Restart services

Problem: Configuration drift, undocumented changes
```

**GitOps Immutable**:
```
1. Update Dockerfile in Git
2. Build new image: myapp:1.2.4
3. Deploy new container
4. Destroy old container

Benefit: Consistent, reproducible, no drift
```

#### Container Image Strategy

```yaml
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**Build & Tag**:
```bash
# Build immutable image
docker build -t myregistry.azurecr.io/myapp:1.2.4 .

# Never modify, always rebuild for changes
```

---

### 7️⃣ Rollback and Recovery

**Definition**: Git-based workflows for rollback during deployment failures.

#### Git Revert Strategy

**Rollback Process**:
```bash
# View recent commits
git log --oneline

# Revert problematic commit
git revert abc123

# Push revert commit
git push origin main

# GitOps tool auto-syncs reverted state
# System automatically rolls back
```

**Example**:
```
Timeline:
14:00 - Commit abc123: Update to v1.2.4 → Deploy
14:15 - Error rate spike detected
14:16 - git revert abc123
14:17 - Revert commit pushed
14:18 - Flux detects revert, rolls back to v1.2.3
14:20 - System stable
```

#### Rollback Strategies

**1. Git Revert (Recommended)**
```bash
git revert HEAD  # Revert latest commit
```
**Pros**: Full audit trail, safe  
**Cons**: Slightly slower (new commit)

**2. Tag-Based Rollback**
```bash
# Tag known-good states
git tag v1.2.3-stable

# Rollback: Point Flux to tag
flux create source git myapp --tag=v1.2.3-stable
```

**3. Branch Rollback**
```bash
# Reset production branch to previous commit
git reset --hard HEAD~1
git push --force origin main
```
**Pros**: Immediate  
**Cons**: Force push (dangerous), lost history

---

### 8️⃣ Observability and Monitoring

**Definition**: Track system health, performance, and deployment status.

#### Monitoring Tools Integration

| Tool | Purpose | GitOps Integration |
|------|---------|-------------------|
| **Azure Monitor** | Azure resources | Query alerts before deploy |
| **Prometheus** | Metrics collection | Scrape metrics, alert on anomalies |
| **Grafana** | Visualization | Dashboards for deployments |
| **Jaeger** | Distributed tracing | Track request flows |
| **Loki** | Log aggregation | Centralized logging |

#### Example: Prometheus + Grafana

**Prometheus Configuration** (in Git):
```yaml
# config/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'myapp'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_app]
        regex: myapp
        action: keep
```

**Application Metrics**:
```typescript
// Instrument application (Node.js example)
import { register, Counter, Histogram } from 'prom-client';

const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status']
});

const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status']
});

app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on('finish', () => {
    httpRequestsTotal.inc({ 
      method: req.method, 
      route: req.route?.path, 
      status: res.statusCode 
    });
    end({ 
      method: req.method, 
      route: req.route?.path, 
      status: res.statusCode 
    });
  });
  next();
});

// Metrics endpoint
app.get('/metrics', (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(register.metrics());
});
```

**Grafana Dashboard** (configured via Git):
```json
{
  "dashboard": {
    "title": "MyApp Deployment Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [{
          "expr": "rate(http_requests_total[5m])"
        }]
      },
      {
        "title": "Error Rate",
        "targets": [{
          "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
        }]
      }
    ]
  }
}
```

---

## Complete GitOps Workflow Example

### Scenario: Update Application Version

```
Step 1: Developer makes code change
├─ git checkout -b feature/new-api
├─ Code changes
├─ git commit -m "Add new API endpoint"
└─ git push origin feature/new-api

Step 2: Pull Request
├─ Open PR to 'develop' branch
├─ Automated tests run
├─ Code review
├─ Merge to 'develop'
└─ GitHub Actions triggered

Step 3: Build & Publish (CI)
├─ Build Docker image
├─ Tag: myapp:develop-abc123
├─ Push to container registry
└─ Update k8s manifest in config repo

Step 4: Deploy to Dev (CD - GitOps)
├─ Flux detects change in config repo
├─ Pulls new manifest
├─ Applies to dev namespace
└─ Kubernetes rolls out deployment

Step 5: Promote to Staging
├─ Merge 'develop' to 'staging' branch
├─ GitHub Actions builds: myapp:staging-def456
├─ Updates staging k8s manifest
└─ Flux deploys to staging namespace

Step 6: Validate Staging
├─ Run automated tests
├─ Manual QA validation
└─ Monitoring confirms health

Step 7: Promote to Production
├─ Merge 'staging' to 'main' branch
├─ GitHub Actions builds: myapp:1.2.4
├─ Updates production k8s manifest
└─ Flux deploys to production namespace

Step 8: Monitor Production
├─ Prometheus scrapes metrics
├─ Grafana visualizes deployment
├─ Alerts configured for anomalies
└─ If issues: git revert + auto-rollback
```

---

## Best Practices

### 🎯 GitOps Best Practices

1. **Separate App Code from Config**
   ```
   ✅ Good:
   - myapp-code repo (application)
   - myapp-config repo (k8s manifests, IaC)
   
   ❌ Bad:
   - Everything in one repo
   ```

2. **Use Pull Requests for All Changes**
   ```
   Even infrastructure changes go through PR:
   - Review
   - Automated validation
   - Approval
   - Merge
   ```

3. **Implement Branch Protection**
   ```yaml
   # GitHub branch protection
   main:
     required_reviews: 2
     dismiss_stale_reviews: true
     require_code_owner_reviews: true
     required_status_checks:
       - build
       - test
       - security-scan
   ```

4. **Tag Releases**
   ```bash
   git tag -a v1.2.4 -m "Release 1.2.4"
   git push origin v1.2.4
   ```

5. **Use Secrets Management**
   ```yaml
   # Don't commit secrets to Git!
   ❌ connectionString: "Server=...;Password=secret"
   
   # Use secret references
   ✅ connectionString: ${AZURE_CONNECTION_STRING}
   
   # Store secrets in:
   - Azure Key Vault
   - Kubernetes Secrets (sealed)
   - HashiCorp Vault
   ```

---

## Critical Notes

⚠️ **Important Considerations**:

1. **Git is Single Source of Truth**: Manual changes are overwritten by GitOps sync
2. **Pull-Based is More Secure**: Cluster pulls from Git (no inbound access needed)
3. **Requires Discipline**: All changes must go through Git
4. **Initial Setup Effort**: Converting existing infrastructure to IaC takes time
5. **Secret Management Critical**: Never commit secrets to Git
6. **Monitoring Essential**: Know when deployments succeed/fail
7. **Rollback = Git Revert**: Fast and auditable

## Quick Reference

| GitOps Component | Technology Choice | Purpose |
|------------------|-------------------|---------|
| **Version Control** | Git (GitHub, GitLab, Bitbucket) | Source of truth |
| **IaC** | Terraform, Bicep, ARM | Infrastructure definition |
| **Configuration** | Ansible, DSC | Server configuration |
| **CI/CD** | GitHub Actions, Azure Pipelines | Build and deploy |
| **GitOps Operator** | Flux CD, Argo CD | Auto-sync Git to cluster |
| **Monitoring** | Prometheus, Grafana | Observability |
| **Secrets** | Azure Key Vault, Sealed Secrets | Secret management |

---

**Learn More**:
- [GitOps Principles](https://opengitops.dev/)
- [Flux CD Documentation](https://fluxcd.io/docs/)
- [Argo CD Documentation](https://argo-cd.readthedocs.io/)
- [Azure Bicep Documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/)

[Learn More: Original Unit](https://learn.microsoft.com/en-us/training/modules/explore-release-strategy-recommendations/6-explore-gitops-release-strategy-recommendations)
