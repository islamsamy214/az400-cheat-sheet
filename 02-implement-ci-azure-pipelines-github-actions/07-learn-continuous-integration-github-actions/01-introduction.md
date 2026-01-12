# Introduction

This module covers continuous integration with GitHub Actions, teaching you to create robust CI/CD pipelines with automated testing, secure secret management, and professional quality standards.

## Module Overview

**Duration**: 39 minutes  
**Skill Level**: Advanced  
**Roles**: Administrator, AI Engineer, Data Engineer, Developer, DevOps Engineer, Platform Engineer, Security Engineer, Solution Architect

**Products**: Azure, GitHub, Azure DevOps, Azure Pipelines, Azure Boards, Azure Repos

## Learning Objectives

By the end of this module, you'll be able to:

✅ **Design and implement continuous integration workflows** with GitHub Actions  
✅ **Configure environment variables** and manage workflow data securely  
✅ **Share artifacts between jobs** and automate release management with Git tags  
✅ **Create and manage encrypted secrets** for secure CI/CD operations  
✅ **Apply industry best practices** for maintainable and scalable automation pipelines

## Prerequisites

**Required**:
- Basic GitHub experience with repositories, branches, and pull requests
- Understanding of development workflows
- Basic command line usage

**Helpful but not required**:
- Experience with CI/CD tools
- Docker knowledge
- Team development experience

## Module Content

| Unit | Topic | Duration |
|------|-------|----------|
| 1 | Introduction | 1 min |
| 2 | Describe continuous integration with Actions | 3 min |
| 3 | Examine environment variables | 3 min |
| 4 | Share artifacts between jobs | 4 min |
| 5 | Examine workflow badges | 3 min |
| 6 | Describe best practices for creating actions | 2 min |
| 7 | Mark releases with Git tags | 2 min |
| 8 | Create encrypted secrets | 3 min |
| 9 | Use secrets in a workflow | 3 min |
| 10 | Implement GitHub Actions for CI/CD | 40 min (lab) |

**Note**: Unit 10 is a hands-on lab exercise requiring Azure subscription.

## What is Continuous Integration with GitHub Actions?

Continuous Integration (CI) is a development practice where team members integrate code changes frequently—ideally multiple times per day. Each integration triggers automated builds and tests, providing immediate feedback and catching issues early.

**GitHub Actions** brings CI/CD natively to GitHub, allowing you to:
- **Automate workflows** directly in your repository
- **Test every commit** before it reaches main branch
- **Deploy automatically** to multiple environments
- **Enforce quality gates** with automated checks
- **Share work efficiently** with artifacts and badges

## Key Themes

### 1. Automation-First Development
Build pipelines that run automatically on every code change, reducing manual work and human error.

### 2. Security by Design
Learn to handle sensitive data with encrypted secrets, proper scoping, and validation patterns.

### 3. Quality Assurance
Implement automated testing, linting, security scanning, and code coverage tracking.

### 4. Efficient Collaboration
Share build artifacts, display workflow status with badges, and provide fast feedback to teammates.

### 5. Release Management
Use Git tags for version control, automate changelog generation, and follow semantic versioning.

## Why This Matters for AZ-400

The AZ-400 exam tests your ability to:
- Design and implement CI/CD pipelines across GitHub and Azure
- Manage secrets and secure sensitive data in workflows
- Optimize build and release processes for efficiency
- Apply DevOps best practices for team collaboration
- Integrate GitHub Actions with Azure services

This module provides **practical patterns** you'll use in real-world DevOps scenarios and **exam-relevant knowledge** for AZ-400 certification.

## Success Criteria

After completing this module, you should be able to:

🎯 Create GitHub Actions workflows with proper triggers and job dependencies  
🎯 Configure multi-level environment variables (workflow/job/step)  
🎯 Share build artifacts between jobs for efficient pipelines  
🎯 Display workflow status with badges in README files  
🎯 Apply action design principles (single responsibility, composability)  
🎯 Use Git tags for semantic versioning and automated releases  
🎯 Create and manage encrypted secrets at multiple scopes  
🎯 Implement secure secret usage patterns with validation  
🎯 Build production-ready CI/CD pipelines with Azure integration  
🎯 Debug workflows using logs, annotations, and debug mode

## Critical Notes

💡 **Native Integration**: GitHub Actions is built into GitHub—no external CI/CD service needed.

⚠️ **Secret Security**: Never log secrets directly. Always use masked environment variables.

📊 **Artifact Retention**: Default retention is 90 days (public repos) or 400 days (private repos, configurable).

🔄 **Workflow Limits**: Free tier provides 2,000 minutes/month (Linux), 20 concurrent jobs.

✨ **Badge Strategy**: Use workflow badges to communicate project health at a glance.

🎯 **Semantic Versioning**: Follow MAJOR.MINOR.PATCH format for clear release history.

## Get Started with Azure

**Azure Integration**: This module demonstrates deploying to Azure App Service using GitHub Actions. You'll need an Azure account to complete the hands-on lab.

Choose the Azure account that's right for you:
- **Pay as you go** for production workloads
- **Try Azure free** for up to 30 days

[Sign up for Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn)

## Module Structure

This module follows a **learn-by-building** approach:

1. **Foundations** (Units 2-3): Understand CI concepts, environment variables
2. **Advanced Features** (Units 4-5): Artifacts, workflow badges
3. **Best Practices** (Unit 6): Professional action design patterns
4. **Release Management** (Unit 7): Git tagging and versioning
5. **Security** (Units 8-9): Encrypted secrets and secure patterns
6. **Hands-On** (Unit 10): Deploy real application to Azure

Each unit builds on previous knowledge, culminating in a complete CI/CD implementation.

## Quick Reference

### Common GitHub Actions Patterns

```yaml
# Basic CI workflow
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm test

# With environment variables
env:
  NODE_VERSION: "20"
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}

# With secrets
steps:
- name: Deploy
  env:
    API_KEY: ${{ secrets.PRODUCTION_API_KEY }}
  run: ./deploy.sh

# With artifacts
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/learn-continuous-integration-github-actions/1-introduction)
