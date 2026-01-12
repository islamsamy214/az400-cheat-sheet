# Introduction to GitHub Advanced Security

## What is GitHub Advanced Security (GHAS)?
GitHub Advanced Security helps you find and fix security issues that can become technical debt. It works with both **GitHub** and **Azure DevOps**, providing powerful tools to keep your code healthy.

While GHAS focuses on security, it also discovers technical debt through scanning tools that find code problems, dependency issues, and security vulnerabilities that slow development over time.

## How GHAS Helps with Technical Debt

GHAS provides **three main tools**:

| Tool | Purpose | Technical Debt Prevention |
|------|---------|---------------------------|
| **Code analysis** | Find problematic patterns | Catches design flaws early |
| **Dependency scanning** | Identify outdated/risky dependencies | Prevents security debt |
| **Security scanning** | Catch vulnerabilities | Stops security issues becoming debt |

**Strategy**: Use these tools early in development to prevent technical debt from building up, keeping code secure, maintainable, and easier to work with.

## CodeQL Analysis: Find Code Problems Automatically

### What is CodeQL?
CodeQL is a smart code analysis tool that searches for problematic patterns in your code.

### What It Finds
- **Coding errors** that slow down development
- **Design flaws** that make code hard to maintain
- **Security vulnerabilities** like injection attacks
- **Authentication and access control** issues

**Analogy**: Think of CodeQL as a detective looking for clues about potential problems in your codebase using patterns to identify hidden technical debt.

### Example Queries
```ql
// Find SQL injection vulnerabilities
from SqlQuery query, UserInput input
where query.usesInput(input)
  and not query.properlyEscapes(input)
select query, "Potential SQL injection"

// Find overly complex methods
from Method m
where m.getCyclomaticComplexity() > 15
select m, "High complexity method"
```

## Dependency Management: Keep Dependencies Healthy

Outdated dependencies are a **common source of technical debt**.

### GHAS Dependency Scanning Helps You
- **See all dependencies** in one place (dependency graph)
- **Find vulnerabilities** in packages
- **Identify outdated** libraries needing updates
- **Check licensing issues** for compliance

### Dependabot Automation
Dependabot **automatically creates pull requests** to update vulnerable dependencies:
- Saves manual work
- Keeps code secure
- Provides context and changelog links
- Allows review before merging

**Example PR**:
```
Title: Bump lodash from 4.17.15 to 4.17.21
Description: 
- Fixes security vulnerability CVE-2021-23337
- Changelog: https://github.com/lodash/lodash/releases/tag/4.17.21
- Dependency: lodash
```

## Code Scanning: Catch Issues Before They Become Debt

### What Code Scanning Checks
Code scanning automatically checks your code for:
- **Security vulnerabilities**: XSS, SQL injection, command injection
- **Code smells**: Indicators of poor design
- **Anti-patterns**: Approaches that make code hard to maintain
- **Quality issues**: Problems that slow development

### Actionable Recommendations
Each scan provides:
- **Clear description** of what's wrong
- **How to fix it** (step-by-step)
- **Priority level** (Critical, High, Medium, Low)
- **Code location** with line numbers

**Example Alert**:
```
Alert: Command Injection Vulnerability
Severity: Critical
Location: src/utils/executor.js:42
Description: User input flows to command execution without validation
Fix: Use parameterized execution or validate input against allowlist
Priority: 1 - Fix immediately
```

## GHAS Components Summary

### CodeQL (Code Analysis Engine)
- **Query language**: Write custom security queries
- **Built-in queries**: 2000+ predefined patterns
- **Multi-language**: JavaScript, TypeScript, Python, Java, C#, C++, Go, Ruby
- **CI/CD integration**: Runs on every commit/PR

### Dependency Review
- **Dependency graph**: Visual representation of all dependencies
- **Vulnerability alerts**: Notifications for known CVEs
- **License detection**: Identify license compliance issues
- **Update suggestions**: Recommendations for safer versions

### Secret Scanning
- **Pattern detection**: Finds API keys, tokens, passwords
- **Partner patterns**: Recognizes 200+ token formats (AWS, Azure, GitHub, etc.)
- **Custom patterns**: Define your own secret formats
- **Automatic alerts**: Notifies when secrets are committed

## Integration with Azure DevOps
GHAS features available in Azure DevOps:
- **GitHub Advanced Security for Azure DevOps**: Same tools, different platform
- **Code scanning**: CodeQL analysis in Azure Pipelines
- **Secret scanning**: Find exposed credentials
- **Dependency scanning**: Vulnerability alerts in Azure Artifacts

## Critical Notes
- 🎯 GHAS prevents security issues from becoming technical debt
- 💡 Dependabot automates dependency updates with minimal effort
- ⚠️ Outdated dependencies are a common source of technical debt
- 📊 CodeQL finds design flaws and security issues automatically
- 🔄 Early detection in CI/CD prevents debt accumulation
- ✨ Each alert includes clear fix recommendations with priority

[Learn More](https://learn.microsoft.com/en-us/training/modules/identify-technical-debt/6-introduction-to-github-advanced-security)
