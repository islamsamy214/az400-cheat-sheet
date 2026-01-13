# Introduction

## Module Overview
This module covers **target environment provisioning**, **service connection configuration**, and **test infrastructure setup**. Learn to configure functional test automation and implement availability testing strategies.

**Duration**: ~45 minutes  
**Level**: Intermediate

## Learning Objectives
By the end of this module, you'll be able to:

1. ✅ **Provision and configure target environments** for deployment workflows
2. ✅ **Deploy securely to environments** using service connections
3. ✅ **Configure functional test automation** and implement availability testing
4. ✅ **Set up comprehensive test infrastructure** for quality assurance

## Prerequisites

| Requirement | Description |
|------------|-------------|
| **DevOps Understanding** | Core DevOps concepts and principles |
| **Version Control** | Familiarity with version control (helpful but not required) |
| **Software Delivery** | Experience in software delivery organizations (beneficial) |
| **Azure DevOps** | Access to Azure DevOps Organization and Team Project |
| **Azure Subscription** | For hands-on exercises with Azure resources |

**Setup Azure DevOps**: [Create an organization](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization)

## Module Scope

This module covers **10 units** focused on environment configuration and testing:

### 📚 What You'll Learn

1. **Introduction** (Unit 1) - Module overview and objectives
2. **Target Environments** (Unit 2) - On-premises, IaaS, PaaS, clusters
3. **Service Connections** (Unit 3) - Secure authentication configuration
4. **Test Automation** (Unit 4) - Integration and functional test automation
5. **Shift-Left Testing** (Unit 5) - Early feedback and quality integration
6. **Availability Tests** (Unit 6) - Post-deployment monitoring
7. **Azure Load Testing** (Unit 7) - Performance and scalability validation
8. **Functional Tests** (Unit 8) - Selenium testing in pipelines
9. **Assessment** (Unit 9) - Knowledge check questions
10. **Summary** (Unit 10) - Recap and next steps

## Key Concepts

### Environment Types

```
Deployment Targets:
├── On-Premises Servers
│   ├── Physical servers
│   ├── Virtual machines (Hyper-V, VMware)
│   └── PowerShell DSC configuration
├── Infrastructure as a Service (IaaS)
│   ├── Azure VMs
│   ├── ARM templates
│   └── Infrastructure as Code
├── Platform as a Service (PaaS)
│   ├── Azure App Service
│   ├── Azure Functions (serverless)
│   └── Managed infrastructure
└── Containers & Clusters
    ├── Azure Kubernetes Service (AKS)
    ├── Docker containers
    └── Orchestration platforms
```

### Service Connections

**Purpose**: Secure, authenticated communication between pipelines and target environments.

**Types**:
- Azure Resource Manager (ARM)
- Docker Registry
- Kubernetes
- AWS, GCP (multi-cloud)
- Generic REST APIs

### Test Automation Strategy

**Agile Testing Quadrants**:
```
Business-Facing          |  Technology-Facing
-------------------------|-------------------------
Q2: Functional Tests     |  Q1: Unit Tests
    Story validation     |      Component tests
    Prototypes          |      Integration tests
-------------------------|-------------------------
Q3: Exploratory Tests    |  Q4: Performance Tests
    Usability testing   |      Load testing
    UAT                 |      Security testing
```

## Why This Matters

### 🎯 Key Benefits

| Benefit | Description |
|---------|-------------|
| **Infrastructure as Code** | Automated, repeatable environment provisioning |
| **Secure Deployments** | Centralized authentication with service connections |
| **Quality Assurance** | Automated testing catches issues early |
| **Shift-Left** | Earlier feedback reduces fix costs |
| **Performance Validation** | Load testing ensures scalability |
| **Continuous Monitoring** | Availability tests detect issues post-deployment |

### 💰 Cost of Defects

**Finding bugs later = Exponentially more expensive**:

| Stage | Relative Cost | Time to Fix |
|-------|--------------|-------------|
| **Development** (Unit tests) | 1x | Minutes |
| **Integration** (CI tests) | 10x | Hours |
| **QA** (Manual testing) | 100x | Days |
| **Production** (Customer finds) | 1000x+ | Weeks |

**Shift-left testing** = Find bugs during development (1x cost) instead of production (1000x cost)

## Module Learning Path

```
01. Introduction
    ↓
02. Provision Target Environments (IaaS, PaaS, Clusters)
    ↓
03. Configure Service Connections (Exercise)
    ↓
04. Test Automation Strategy (Agile Quadrants)
    ↓
05. Shift-Left Testing (L0-L3 taxonomy)
    ↓
06. Availability Tests (Application Insights)
    ↓
07. Azure Load Testing (Performance validation)
    ↓
08. Functional Tests (Selenium lab)
    ↓
09. Knowledge Check
    ↓
10. Summary
```

## Success Criteria

By module completion, you should be able to:

- [ ] Provision Azure resources using ARM templates
- [ ] Configure Azure Resource Manager service connections
- [ ] Implement automated testing in pipelines
- [ ] Apply shift-left testing principles
- [ ] Set up Application Insights availability tests
- [ ] Configure Azure Load Testing for performance validation
- [ ] Run Selenium functional tests in release pipelines

## Quick Reference

| Term | Definition | Example |
|------|-----------|---------|
| **IaaS** | Infrastructure as a Service - Virtual machines | Azure VMs |
| **PaaS** | Platform as a Service - Managed hosting | Azure App Service |
| **Service Connection** | Secure auth to external resources | Azure ARM connection |
| **Shift-Left** | Move testing earlier in lifecycle | Unit tests before merge |
| **L0-L3 Tests** | Test taxonomy by isolation level | L0=in-memory, L3=full system |
| **Availability Test** | Post-deployment health monitoring | URL ping every 5 minutes |
| **Load Test** | Performance under simulated load | 1000 concurrent users |

## What's Next?

**Unit 2**: Learn about provisioning and configuring target environments (on-premises, IaaS, PaaS, clusters)

---

**Learn More**:
- [Azure Resource Manager Overview](https://learn.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview)
- [Service Connections in Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints)
- [Shift Left to Make Testing Fast and Reliable](https://learn.microsoft.com/en-us/devops/develop/shift-left-make-testing-fast-reliable)

[Learn More: Original Unit](https://learn.microsoft.com/en-us/training/modules/configure-provision-environments/1-introduction)
