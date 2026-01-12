# Introduction to GitHub Actions

GitHub Actions is GitHub's native CI/CD and automation platform that enables developers to automate workflows directly within their repositories. Unlike external CI/CD tools, GitHub Actions integrates seamlessly with GitHub's ecosystem, providing event-driven automation for build, test, and deployment tasks.

## Learning Objectives

By completing this module, you will:

✅ **Understand GitHub Actions fundamentals** - Learn core concepts including actions, workflows, events, jobs, and runners

✅ **Create and manage workflows** - Write YAML-based workflows, configure triggers, and organize jobs effectively

✅ **Work with events, jobs, and runners** - Configure event triggers, manage job dependencies, and choose appropriate runners

✅ **Read workflow output and manage releases** - Access logs, debug workflows, and implement version management strategies

## Prerequisites

To get the most from this module, you should have:

- **Basic DevOps knowledge**: Understanding of CI/CD principles and continuous integration concepts
- **Version control experience**: Familiarity with Git and GitHub (repositories, branches, pull requests)
- **Development background**: Experience with software development processes and command-line tools
- **YAML familiarity**: Basic understanding of YAML syntax (helpful but not required)

## Module Content

This module covers GitHub Actions fundamentals through 9 comprehensive units:

| Unit | Topic | Duration |
|------|-------|----------|
| 1 | Introduction | 2 min |
| 2 | What are actions? | 2 min |
| 3 | Explore actions flow | 2 min |
| 4 | Understand workflows | 3 min |
| 5 | Standard workflow syntax | 2 min |
| 6 | Explore events | 3 min |
| 7 | Explore jobs | 3 min |
| 8 | Explore runners | 3 min |
| 9 | Release and test actions | 4 min |

**Total Duration**: ~24 minutes

## What is GitHub Actions?

GitHub Actions is an **event-driven automation platform** built into GitHub that allows you to:

- **Automate workflows** triggered by repository events (push, pull request, issue creation)
- **Build and test** code automatically on every commit or pull request
- **Deploy applications** to various platforms (Azure, AWS, Google Cloud, on-premises)
- **Manage projects** by automating issue triage, labeling, and notifications
- **Enforce policies** through required checks, code scanning, and compliance validation

### Key Advantages

**Native Integration**: No external CI/CD service required - everything runs within GitHub

**Event-Driven**: Workflows trigger automatically based on 40+ repository events

**Marketplace Ecosystem**: 10,000+ pre-built actions from GitHub and community

**Multi-Platform**: Run workflows on Linux, Windows, and macOS runners

**Cost-Effective**: 2,000 free minutes/month for private repositories, unlimited for public

## Key Themes

### 1. Automation Fundamentals

Learn how GitHub Actions automates development workflows through:
- YAML-based workflow definitions stored in `.github/workflows/`
- Event triggers that start workflows (push, pull_request, schedule, workflow_dispatch)
- Jobs that run in parallel or sequentially based on dependencies
- Steps that execute commands or call pre-built actions

### 2. Workflow Architecture

Understand the hierarchical structure:
- **Workflows** contain one or more jobs
- **Jobs** contain one or more steps
- **Steps** run commands or use actions
- **Runners** execute jobs in isolated virtual environments

### 3. Practical CI/CD Patterns

Explore common automation patterns:
- Continuous Integration: Build and test on every commit
- Continuous Deployment: Auto-deploy to staging/production
- Code quality enforcement: Linting, security scanning, test coverage
- Release automation: Version tagging, changelog generation, artifact publishing

## Why GitHub Actions Matters for AZ-400

For the AZ-400 DevOps Engineer Expert certification, GitHub Actions knowledge is critical because:

🎯 **Multi-Platform CI/CD**: Organizations use both Azure Pipelines AND GitHub Actions - understanding both is essential

🎯 **GitHub Integration**: Many Azure DevOps projects integrate with GitHub repos, requiring GitHub Actions knowledge

🎯 **Modern DevOps**: GitHub Actions represents modern, cloud-native CI/CD architecture

🎯 **Comparison Skills**: Ability to compare Azure Pipelines vs GitHub Actions features and choose appropriate tool

🎯 **Migration Scenarios**: Understanding both platforms enables pipeline migration and hybrid strategies

## Critical Notes

💡 **Free Tier Limits**: GitHub Free provides 2,000 workflow minutes/month for private repos. Public repos get unlimited minutes.

⚠️ **Self-Hosted Security**: Never use self-hosted runners with public repositories - malicious PRs could execute code on your infrastructure.

🎯 **Marketplace Actions**: Always verify action sources before use. Prefer actions from `actions/*` (official) or well-maintained community actions.

📊 **Job Concurrency**: Free accounts get 20 concurrent jobs, Team accounts get 60, Enterprise gets 180. Plan workflows accordingly.

🔄 **Workflow Files**: Must be stored in `.github/workflows/` directory and use `.yml` or `.yaml` extension to be recognized.

✨ **Latest Versions**: Use latest action versions (e.g., `actions/checkout@v4`) for best performance and security fixes.

## Learning Path Context

This module is part of **Learning Path 2: Implement CI with Azure Pipelines and GitHub Actions** in the AZ-400 certification journey:

**Previous Modules**:
- Module 1-5: Azure Pipelines fundamentals, agents, concurrency, strategy, integration

**Current Module** (Module 6):
- Introduction to GitHub Actions fundamentals

**Upcoming Modules**:
- Module 7: Learn continuous integration with GitHub Actions
- Module 8: Design a container build strategy

## Success Criteria

After completing this module, you should be able to:

✅ Explain what GitHub Actions is and how it differs from Azure Pipelines

✅ Describe the GitHub Actions workflow execution flow (event → workflow → job → step → action)

✅ Create basic workflows with triggers, jobs, and steps

✅ Configure event triggers (push, pull_request, schedule, manual)

✅ Understand job execution patterns (parallel vs sequential with dependencies)

✅ Choose between GitHub-hosted and self-hosted runners

✅ Access workflow logs and debug failures

✅ Manage action versions using tags, commits, or branches

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-github-actions/)
