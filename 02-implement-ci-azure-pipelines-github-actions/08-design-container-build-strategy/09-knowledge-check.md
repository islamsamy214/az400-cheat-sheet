# Knowledge Check

Test your understanding of container build strategies, Docker, Dockerfiles, multi-stage builds, and Azure container services.

## Questions

### Question 1: Container vs VM Architecture

**What is the primary difference between containers and virtual machines?**

A) Containers include a complete operating system, while VMs share the host kernel  
B) Containers share the host OS kernel, while VMs include a complete guest OS  
C) Containers require a hypervisor, while VMs use container runtime  
D) Containers are slower to start than VMs  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: Containers use operating system-level virtualization and share the host OS kernel, making them lightweight (typically 5-100 MB). Virtual machines use hardware virtualization with a hypervisor and include a complete guest operating system, resulting in larger sizes (typically 1-10 GB) and slower startup times. This architectural difference makes containers faster to start (seconds vs minutes) and more resource-efficient.

**Key Concept**: Containers provide process-level isolation sharing the host kernel, while VMs provide full OS-level isolation with their own kernel.

**Review**: If you answered incorrectly, revisit [Unit 2: Examine structure of containers](#) for detailed architecture comparison.

</details>

---

### Question 2: Dockerfile Instructions

**Which Dockerfile instruction is used to set the base image for a container?**

A) `RUN`  
B) `FROM`  
C) `CMD`  
D) `COPY`  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: The `FROM` instruction specifies the base image and must be the first instruction in a Dockerfile (except for ARG used before FROM). Example:
```dockerfile
FROM node:18-alpine
```

**Other Instructions**:
- `RUN`: Executes commands during build (creates layers)
- `CMD`: Specifies default command to run when container starts
- `COPY`: Copies files from build context into container

**Review**: If you answered incorrectly, revisit [Unit 4: Understand Dockerfile core concepts](#) for instruction details.

</details>

---

### Question 3: Multi-Stage Builds

**What is the primary benefit of multi-stage Dockerfiles?**

A) Faster container runtime performance  
B) Reduced final image size by excluding build dependencies  
C) Automatic container scaling  
D) Improved network security  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: Multi-stage builds allow you to use multiple `FROM` instructions, copying only necessary artifacts between stages. This excludes build tools, compilers, and development dependencies from the final production image, resulting in:
- **Smaller images**: 70-95% size reduction (e.g., 2 GB → 200 MB)
- **Improved security**: Fewer packages = smaller attack surface
- **Faster deployments**: Smaller images download and start faster

Example:
```dockerfile
# Build stage (2 GB with SDK)
FROM golang:1.21 AS build
WORKDIR /app
COPY . .
RUN go build -o myapp

# Final stage (10 MB with binary only)
FROM alpine:latest
COPY --from=build /app/myapp /myapp
CMD ["/myapp"]
```

**Review**: If you answered incorrectly, revisit [Unit 5: Examine multi-stage Dockerfiles](#) for optimization techniques.

</details>

---

### Question 4: Docker Commands

**Which command is used to build a Docker image from a Dockerfile?**

A) `docker run`  
B) `docker build`  
C) `docker create`  
D) `docker push`  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: `docker build` creates an image from a Dockerfile and build context:
```bash
docker build -t myapp:v1 .
```

**Other Commands**:
- `docker run`: Creates and starts a container from an image
- `docker create`: Creates a container without starting it
- `docker push`: Uploads an image to a registry

**Complete Workflow**:
```bash
docker build -t myapp:v1 .        # Build image
docker push myregistry/myapp:v1   # Push to registry
docker pull myregistry/myapp:v1   # Pull from registry
docker run myapp:v1               # Run container
```

**Review**: If you answered incorrectly, revisit [Unit 3: Work with Docker containers](#) for Docker lifecycle commands.

</details>

---

### Question 5: Container Design Principles

**Which design principle is essential for horizontal scaling of containers?**

A) Store all data inside the container filesystem  
B) Include all dependencies in a single container  
C) Design containers as stateless with external storage for data  
D) Use the largest possible base image  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: C**

**Explanation**: Stateless design is critical for horizontal scaling. Containers should:
- **Not store state locally**: Use external databases, caches (Redis), and object storage (S3, Azure Blob)
- **Be immutable**: Any instance can be destroyed/recreated without data loss
- **Support load balancing**: Requests can go to any instance

**Example - Stateful (Bad)**:
```javascript
const sessions = {};  // Lost when container restarts
```

**Example - Stateless (Good)**:
```javascript
const redis = require('redis').createClient();
// Session stored externally, survives restarts
```

**Benefits**:
- Scale from 1 → 100 instances instantly
- Zero-downtime deployments
- Fault tolerance (auto-recovery)

**Review**: If you answered incorrectly, revisit [Unit 6: Examine considerations for multiple stage builds](#) for design principles.

</details>

---

### Question 6: Azure Container Services

**Which Azure service is best for serverless containers with automatic scale-to-zero?**

A) Azure Kubernetes Service (AKS)  
B) Azure Container Instances (ACI)  
C) Azure Container Apps  
D) Azure App Service  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: C**

**Explanation**: Azure Container Apps provides:
- **Event-driven auto-scaling** with KEDA (Kubernetes Event-Driven Autoscaling)
- **Scale to zero**: No charges when idle
- **Built-in Dapr support**: Microservices patterns
- **No Kubernetes management**: Simplified orchestration

**Service Comparison**:
| Service | Scale to Zero | Management | Use Case |
|---------|---------------|------------|----------|
| **Container Apps** | ✅ Yes | Minimal | Event-driven microservices |
| **ACI** | ❌ No (pay per second) | None | Batch jobs, quick tasks |
| **AKS** | ❌ No (nodes always running) | High | Complex Kubernetes apps |
| **App Service** | ❌ No | Low | Traditional web apps |

**Example Scenario**: API that handles 1000 requests/hour during business hours, 0 requests at night:
- **Container Apps**: Scale to 0 at night (pay $0), scale up during day
- **App Service**: Instance runs 24/7 (pay even when idle)

**Review**: If you answered incorrectly, revisit [Unit 7: Explore Azure container-related services](#) for service selection guidance.

</details>

---

### Question 7: Azure Container Registry (ACR)

**What feature of Azure Container Registry (Premium SKU) enables faster image pulls across multiple Azure regions?**

A) Content trust  
B) Geo-replication  
C) Webhook integration  
D) Virtual network support  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: Geo-replication (Premium SKU only) automatically replicates container images to multiple Azure regions:

**Architecture**:
```
Primary Registry (East US)
        ↓ Replicate
   ┌────┴────┐
   ↓         ↓
West US   Europe
   ↓         ↓
AKS West  AKS EU
(Fast local pulls)
```

**Benefits**:
- **Faster deployments**: Pull from nearest region (reduced latency)
- **High availability**: Registry available even if primary region fails
- **Network cost reduction**: Data transfer within same region is cheaper

**Example**:
```bash
# Enable geo-replication
az acr replication create \
  --registry myregistry \
  --location westus

az acr replication create \
  --registry myregistry \
  --location westeurope
```

**Other Premium Features**:
- **Content trust**: Image signing for security
- **Private link**: Private endpoint connectivity
- **Customer-managed keys**: Encryption control

**Review**: If you answered incorrectly, revisit [Unit 7: Explore Azure container-related services](#) for ACR features.

</details>

---

### Question 8: App Service Deployment

**Which App Service feature enables zero-downtime deployments by testing new versions before production?**

A) Auto-scaling  
B) Deployment slots  
C) VNet integration  
D) Continuous deployment  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: Deployment slots (Standard tier and above) allow staging-to-production workflows:

**Workflow**:
```
1. Deploy new version to staging slot
2. Test staging environment thoroughly
3. Swap staging ↔ production (instant, zero downtime)
4. If issues: swap back immediately (rollback)
```

**Example**:
```bash
# Create staging slot
az webapp deployment slot create \
  --name myWebApp \
  --resource-group myResourceGroup \
  --slot staging

# Deploy to staging
az webapp config container set \
  --name myWebApp \
  --slot staging \
  --docker-custom-image-name myregistry.azurecr.io/myapp:v2

# Test staging: https://mywebapp-staging.azurewebsites.net

# Swap to production (zero downtime)
az webapp deployment slot swap \
  --name myWebApp \
  --slot staging \
  --target-slot production
```

**Benefits**:
- **Zero downtime**: Old instances → new instances seamlessly
- **Instant rollback**: Swap back if issues detected
- **Testing in production environment**: Same configuration as prod
- **Traffic routing**: Can gradually shift traffic (A/B testing)

**Review**: If you answered incorrectly, revisit [Unit 8: Deploy Docker containers to Azure App Service](#) for deployment strategies.

</details>

---

### Question 9: Security Best Practice

**According to container security best practices, which user should containers run as?**

A) Root user (UID 0) for maximum privileges  
B) Non-root user with minimum required permissions  
C) System user with sudo access  
D) Administrator account  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B**

**Explanation**: Containers should **never run as root** in production (principle of least privilege):

**Security Risk (Root)**:
- If container compromised, attacker has root access to host
- Can escape container and access host filesystem
- Can modify system files and configurations

**Best Practice (Non-Root)**:
```dockerfile
# Create non-root user
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

# Set ownership
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# All subsequent commands run as appuser
CMD ["node", "server.js"]
```

**Verification**:
```bash
# Check user inside container
docker exec mycontainer whoami
# Output: appuser (not root)
```

**Why This Matters**:
- **Defense in depth**: Limits damage from container escape vulnerabilities
- **Compliance**: Required by many security standards (PCI-DSS, SOC 2)
- **Kubernetes**: Some clusters enforce non-root with Pod Security Standards

**Review**: If you answered incorrectly, revisit [Unit 6: Examine considerations for multiple stage builds](#) for security best practices.

</details>

---

### Question 10: Image Optimization

**Which base image type results in the smallest production container size?**

A) `ubuntu:22.04` (full distribution)  
B) `ubuntu:22.04-slim` (minimal packages)  
C) `alpine:latest` (Alpine Linux)  
D) `scratch` (empty base image)  

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: D**

**Explanation**: Base image size comparison:

| Base Image | Size | Use Case | Pros | Cons |
|------------|------|----------|------|------|
| **ubuntu:22.04** | 500+ MB | Development | All tools included | Very large |
| **ubuntu:22.04-slim** | ~100 MB | General purpose | Common tools | Still large |
| **alpine:latest** | ~5 MB | Production | Small, fast | Some compatibility issues |
| **scratch** | 0 MB | Static binaries | Smallest possible | No shell, no utilities |

**Scratch Example** (Go static binary):
```dockerfile
# Build stage
FROM golang:1.21 AS build
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o myapp

# Final stage - 10 MB total!
FROM scratch
COPY --from=build /app/myapp /myapp
CMD ["/myapp"]
```

**Alpine Example** (Node.js):
```dockerfile
FROM node:18-alpine  # 50 MB vs 1 GB for node:18
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "server.js"]
```

**Optimization Impact**:
- **Faster deployments**: 10 MB vs 1 GB = 100x faster pull
- **Lower costs**: Less storage, bandwidth
- **Better security**: Fewer packages = smaller attack surface
- **Faster scaling**: New instances start quickly

**Review**: If you answered incorrectly, revisit [Unit 6: Examine considerations for multiple stage builds](#) for image optimization.

</details>

---

## Scoring Guide

- **9-10 correct**: Excellent! You have mastered container build strategies.
- **7-8 correct**: Good understanding. Review missed topics for deeper knowledge.
- **5-6 correct**: Passing. Study the module content again, focusing on weak areas.
- **Below 5**: Review the entire module. Container concepts require solid understanding.

## Key Takeaways

If you struggled with certain questions, focus on these areas:

### Container Fundamentals
- ✅ Containers vs VMs (shared kernel vs full OS)
- ✅ Container architecture and isolation
- ✅ Benefits: lightweight, fast, portable

### Docker Mastery
- ✅ Dockerfile instructions (`FROM`, `RUN`, `CMD`, `COPY`)
- ✅ Docker commands (`build`, `run`, `push`, `pull`)
- ✅ Container lifecycle management

### Multi-Stage Builds
- ✅ Purpose: Smaller images (exclude build tools)
- ✅ Named stages with `AS` keyword
- ✅ Cross-stage copying with `COPY --from`
- ✅ 70-95% size reduction achievable

### Design Principles
- ✅ Single responsibility per container
- ✅ Stateless design with external storage
- ✅ Non-root user for security
- ✅ Minimal base images (Alpine, Distroless, scratch)

### Azure Services
- ✅ **ACI**: Serverless, short-lived tasks
- ✅ **Container Apps**: Event-driven, scale-to-zero
- ✅ **AKS**: Full Kubernetes, complex orchestration
- ✅ **ACR**: Private registry, geo-replication
- ✅ **App Service**: PaaS web apps, deployment slots

### Deployment
- ✅ Continuous deployment from ACR
- ✅ Deployment slots for staging/production
- ✅ Managed identity for ACR authentication
- ✅ Application Insights for monitoring

## Next Steps

Ready to continue? Proceed to the [Summary](#) unit for module recap and next learning steps.

Need more practice? Revisit specific units:
- [Unit 2: Container Structure](#) - Architecture deep dive
- [Unit 3: Docker Containers](#) - Command mastery
- [Unit 4: Dockerfile Concepts](#) - Instruction details
- [Unit 5: Multi-Stage Builds](#) - Optimization techniques
- [Unit 6: Design Considerations](#) - Best practices
- [Unit 7: Azure Services](#) - Service selection
- [Unit 8: App Service Deployment](#) - Hands-on deployment

[Learn More](https://learn.microsoft.com/en-us/training/modules/design-container-build-strategy/9-knowledge-check)
