# Introduction

## Key Concepts
- **Software Composition Analysis (SCA)**: Automated tools and processes for identifying, tracking, managing security and compliance risks in software supply chain
- **Dependency Scanning**: Automated detection of vulnerabilities in open-source components and third-party libraries
- **Software Bill of Materials (SBOM)**: Comprehensive inventory of all dependencies for compliance and transparency
- **Automated vs Manual**: Manual tracking impractical across hundreds of apps; automated SCA essential

## Why SCA Matters
- Modern apps heavily depend on open-source components and third-party libraries
- Dependencies accelerate development and provide proven functionality
- Also introduce security vulnerabilities and license compliance risks
- Thousands of new vulnerabilities discovered annually in OSS components
- Applications remain vulnerable until dependencies updated
- Without systematic analysis, organizations unaware until breaches occur

## Module Coverage

### 1. Understanding Software Composition Analysis
- What SCA is and why critical for modern development
- What SCA tools detect
- Benefits of automated dependency scanning
- Why manual dependency management doesn't scale
- Continuous visibility into security and compliance risks

### 2. Inspecting and Validating Code Bases
- Inventory all dependencies (including transitive)
- Validate license compliance (avoid legal issues)
- Detect known vulnerabilities (CVE databases)
- Assess dependency quality (maintenance status, community health)

### 3. Implementing GitHub Dependabot
- Automatic vulnerability scanning
- Auto-generated pull requests with security updates
- Dependency graphs showing transitive dependencies
- Integration with GitHub Security tab (centralized management)

### 4. Integrating SCA into Pipelines
- Automated scanning in Azure Pipelines (build and deployment)
- Configure tools: Mend (WhiteSource), Snyk, OWASP Dependency-Check
- Quality gates (fail builds on critical vulnerabilities)
- Generate SBOM for compliance and transparency

### 5. Examining SCA Tools
- Commercial: Mend, Snyk (comprehensive scanning + support)
- Open-Source: OWASP Dependency-Check (cost-effective)
- Platform-Integrated: Azure Artifacts upstream sources, GitHub Dependabot
- Compare strengths, limitations, ideal use cases

### 6. Automating Container Scanning
- Scan container base images for vulnerabilities
- Analyze application dependencies within containers
- Integrate into CI/CD pipelines
- Configure registries (ACR, Docker Hub) to block vulnerable images

### 7. Interpreting Scanner Alerts
- Assess severity (CVSS scores)
- Determine exploitability (vulnerabilities reachable in app?)
- Prioritize remediation (business impact, risk)
- Manage false positives (proper filtering)

## Learning Objectives
After this module, you'll be able to:
- Understand SCA and why automated dependency scanning essential
- Inspect/validate code bases (inventory, license compliance, CVE detection, quality assessment)
- Implement GitHub Dependabot (auto-detect, PRs, dependency graphs, Security tab)
- Integrate SCA into Azure Pipelines (scanning, quality gates, reports, SBOM)
- Examine/configure SCA tools (Mend, Snyk, OWASP Dependency-Check, Azure Artifacts)
- Automate container image scanning (base images, app dependencies, CI/CD, registry blocking)
- Interpret alerts (CVSS, exploitability, prioritization, false positives)

## Prerequisites
- Software dependencies understanding (libraries, frameworks, package managers)
- DevOps concepts familiarity (CI/CD, build pipelines, SDLC)
- Version control experience (Git, pull requests, code reviews)
- Open-source awareness (vulnerabilities, license requirements)
- Azure DevOps or GitHub experience (for automated scanning implementation)

## Critical Notes
- ⚠️ Thousands of new vulnerabilities discovered annually in OSS
- 💡 Manual tracking impractical; automated SCA essential
- 🎯 Without systematic analysis, vulnerabilities unknown until breaches
- 📊 SCA provides continuous visibility into security and compliance risks

[Learn More](https://learn.microsoft.com/en-us/training/modules/software-composition-analysis/1-introduction)
