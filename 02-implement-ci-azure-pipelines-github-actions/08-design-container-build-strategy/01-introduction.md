# Introduction

Containers have revolutionized how we build, ship, and deploy applications. This module teaches you to design effective container build strategies, work with Docker, implement multi-stage builds, and deploy containerized applications to Azure services.

## Module Overview

Containerization is a fundamental skill for modern DevOps engineers. This module provides comprehensive coverage of container concepts, Docker workflows, and Azure container services, preparing you for real-world container deployment scenarios in the AZ-400 exam.

**What You'll Learn**:
- Container fundamentals and architecture
- Docker container lifecycle and commands
- Dockerfile creation and best practices
- Multi-stage build optimization
- Azure container service options
- Production deployment strategies

## Learning Objectives

By completing this module, you will be able to:

1. ✅ **Design Effective Container Build Strategies**: Understand container architecture, lifecycle, and design principles for building optimized, secure containers

2. ✅ **Work with Docker Containers and Dockerfiles**: Create, build, and manage Docker containers using Dockerfiles with industry best practices

3. ✅ **Implement Multi-Stage Builds and Best Practices**: Use multi-stage Dockerfiles to create lean, production-ready container images with minimal attack surface

4. ✅ **Deploy Containers to Azure Services**: Evaluate and use Azure container services (ACI, AKS, ACR, Container Apps, App Service) for different deployment scenarios

## Prerequisites

**None**: This module starts with container fundamentals and builds to advanced topics. However, familiarity with the following will help:

- Basic Linux/command-line operations
- Understanding of application deployment concepts
- General knowledge of Azure services
- DevOps principles (CI/CD pipelines)

## Module Content

| # | Unit | Topics Covered | Key Skills |
|---|------|----------------|------------|
| 1 | Introduction | Module overview, learning objectives | Course navigation |
| 2 | Examine Structure of Containers | Containers vs VMs, architecture, benefits | Container fundamentals |
| 3 | Work with Docker Containers | Docker lifecycle, build/run/pull commands | Docker basics |
| 4 | Understand Dockerfile Core Concepts | Dockerfile instructions, layering, best practices | Dockerfile creation |
| 5 | Examine Multi-Stage Dockerfiles | Multi-stage builds, optimization, testing | Advanced Dockerfiles |
| 6 | Examine Considerations for Multiple Stage Builds | Design principles, image optimization, data management | Build strategies |
| 7 | Explore Azure Container-Related Services | ACI, AKS, ACR, Container Apps, App Service | Service selection |
| 8 | Deploy Docker Containers to Azure App Service | Hands-on deployment, configuration, troubleshooting | Azure deployment |
| 9 | Knowledge Check | Assessment questions | Knowledge validation |
| 10 | Summary | Key takeaways, next steps | Review |

**Total Time**: Approximately 60 minutes

## What Are Containers?

Containers are lightweight, standalone, executable packages that include everything needed to run software: code, runtime, system tools, libraries, and settings. They solve the "works on my machine" problem by ensuring consistent behavior across all environments.

### Why Containers Matter for DevOps

**Key Benefits**:

📦 **Consistency**: Same container runs identically in development, testing, and production

⚡ **Speed**: Containers start in seconds vs minutes for VMs

🔄 **Portability**: Run anywhere—laptop, datacenter, cloud, hybrid environments

💰 **Efficiency**: Multiple containers share the same OS kernel, maximizing resource utilization

🛡️ **Isolation**: Applications run in isolated environments with controlled resource limits

🚀 **Scalability**: Quickly scale containers up or down based on demand

### Container vs Virtual Machine

```
Virtual Machine Architecture:
┌─────────────────────────────────────┐
│        Application Layer            │
├─────────────────────────────────────┤
│     Bins/Libraries (per VM)         │
├─────────────────────────────────────┤
│    Guest OS (per VM) - Full OS      │
├─────────────────────────────────────┤
│         Hypervisor                  │
├─────────────────────────────────────┤
│       Host OS + Hardware            │
└─────────────────────────────────────┘
Size: GBs per VM | Boot: Minutes

Container Architecture:
┌─────────────────────────────────────┐
│     Application Layer               │
├─────────────────────────────────────┤
│    Bins/Libraries (shared)          │
├─────────────────────────────────────┤
│     Container Runtime (Docker)      │
├─────────────────────────────────────┤
│       Host OS + Hardware            │
└─────────────────────────────────────┘
Size: MBs per container | Boot: Seconds
```

## Real-World Container Use Cases

### 1. Microservices Architecture

Break monolithic applications into containerized microservices:

```
E-commerce Application:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Frontend   │  │   Auth API   │  │  Payment API │
│  Container   │→ │  Container   │→ │  Container   │
└──────────────┘  └──────────────┘  └──────────────┘
       ↓                 ↓                 ↓
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Product DB  │  │   User DB    │  │  Order DB    │
│  Container   │  │  Container   │  │  Container   │
└──────────────┘  └──────────────┘  └──────────────┘
```

### 2. CI/CD Pipelines

Containers ensure consistent build and test environments:

```yaml
# GitHub Actions example
jobs:
  test:
    runs-on: ubuntu-latest
    container: node:18-alpine  # Consistent test environment
    steps:
      - uses: actions/checkout@v3
      - run: npm install && npm test
```

### 3. Multi-Environment Deployments

Same container image deployed across environments:

```
Development → Staging → Production
     ↓            ↓           ↓
  Container    Container   Container
   (same         (same       (same
    image)        image)      image)
```

### 4. Legacy Application Modernization

Containerize legacy apps without rewriting:

```
Before:                After:
┌─────────────┐       ┌─────────────┐
│ Monolithic  │       │   Legacy    │
│ Application │  →→   │ Application │
│   on VM     │       │ in Container│
└─────────────┘       └─────────────┘
Hard to scale         Easy to scale
```

## Azure Container Services Overview

Microsoft Azure provides multiple container hosting options:

| Service | Best For | Complexity | Management |
|---------|----------|------------|------------|
| **Azure Container Instances (ACI)** | Simple containers, burst workloads | Low | Serverless |
| **Azure Container Apps** | Microservices, event-driven apps | Low-Medium | Serverless |
| **Azure App Service** | Web apps with container support | Medium | Managed PaaS |
| **Azure Kubernetes Service (AKS)** | Complex orchestration, enterprise | High | Managed Kubernetes |
| **Azure Container Registry (ACR)** | Private image storage | Low | Managed registry |

**Decision Flow**:
```
Need simple container run? → ACI
Need serverless microservices? → Container Apps
Need web app with containers? → App Service
Need full Kubernetes orchestration? → AKS
Need private image storage? → ACR (use with any service)
```

## Module Learning Path

This module follows a progressive learning approach:

**Foundation** (Units 2-4):
- Container concepts and benefits
- Docker basics and commands
- Dockerfile fundamentals

**Intermediate** (Units 5-6):
- Multi-stage build optimization
- Container design best practices
- Image size reduction techniques

**Advanced** (Units 7-8):
- Azure container service options
- Deployment to App Service
- Production considerations

## Success Criteria

By the end of this module, you should be able to:

✅ Explain container architecture and benefits vs VMs  
✅ Create Dockerfiles for various application types  
✅ Implement multi-stage builds to optimize image size  
✅ Choose appropriate Azure container service for scenarios  
✅ Deploy containerized applications to Azure  
✅ Apply container security and optimization best practices

## Why This Matters for AZ-400

Container knowledge is essential for the AZ-400 exam:

- **Exam Weight**: Containers appear in multiple exam domains
- **Real-World Relevance**: Most modern DevOps workflows use containers
- **Azure Integration**: Understanding Azure container services is crucial
- **CI/CD Pipelines**: Containers are central to modern CI/CD strategies

**Exam Topics Covered**:
- Design and implement a container build strategy
- Deploy Docker containers to Azure services
- Integrate containers with Azure DevOps pipelines
- Implement container security and compliance

## Key Concepts to Master

### Docker Terminology

- **Image**: Read-only template with application and dependencies
- **Container**: Running instance of an image
- **Dockerfile**: Text file with instructions to build an image
- **Registry**: Storage for container images (e.g., Docker Hub, ACR)
- **Layer**: Individual instruction in Dockerfile (cached for efficiency)

### Container Lifecycle

```
Build → Push → Pull → Run → Stop → Remove
  ↓       ↓      ↓      ↓      ↓       ↓
Image   Registry Local  Container Stopped Cleaned
```

## Critical Notes

🎯 **Containers ≠ VMs**: Containers share the host OS kernel, making them lighter and faster than VMs with full OS instances.

💡 **Immutable Infrastructure**: Containers should be immutable—update by replacing, not modifying running containers.

⚠️ **Stateless Design**: Design containers as stateless when possible; use external storage for persistent data.

📊 **Multi-Stage Builds**: Always use multi-stage Dockerfiles for production to minimize image size and attack surface.

🔄 **Registry Strategy**: Use private registries (ACR) for production images; never store secrets in images.

✨ **Right-Sizing**: Choose the smallest Azure container service that meets your needs to optimize costs and complexity.

## Quick Reference

### Essential Docker Commands

```bash
# Build image from Dockerfile
docker build -t myapp:1.0 .

# Run container from image
docker run -d -p 8080:80 myapp:1.0

# List running containers
docker ps

# View all containers (including stopped)
docker ps -a

# Stop container
docker stop <container-id>

# Remove container
docker rm <container-id>

# View images
docker images

# Remove image
docker rmi myapp:1.0
```

### Module Navigation

- **Next**: Examine Structure of Containers (fundamentals)
- **After That**: Work with Docker Containers (hands-on)
- **Then**: Dockerfile creation and optimization
- **Finally**: Azure deployment strategies

[Start Learning](https://learn.microsoft.com/en-us/training/modules/design-container-build-strategy/2-examine-structure-of-containers)
