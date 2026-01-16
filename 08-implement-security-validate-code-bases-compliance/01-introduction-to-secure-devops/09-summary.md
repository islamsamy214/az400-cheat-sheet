# Summary

## Module Overview
This module explored integrating security throughout the software development lifecycle using DevSecOps principles. Traditional security checkpoints at release create bottlenecks and increase costs. Shifting security left and making it everyone's responsibility maintains velocity while improving security posture.

## Key Learnings

### SQL Injection Attacks
- **Attack Mechanism**: Malicious SQL code in inputs exploiting insufficient validation
- **Potential Damage**: Bypass auth, retrieve entire DB, modify/delete records, execute OS commands, DoS
- **Affected Systems**: All major DBs (MySQL, Oracle, SQL Server, PostgreSQL, SQLite)
- **Prevention**: Parameterized queries, input validation/sanitization, least privilege, regular testing, monitoring

### DevSecOps Principles
- **Security Gaps**: Encryption gaps (data at rest/transit), missing HTTP security headers
- **Traditional Problems**: Unplanned end-of-cycle work, expensive rework, bottlenecks, deprioritized security
- **DevSecOps Solution**: Security from beginning, shared responsibility across teams
- **Expanded Scope**: Secure entire pipeline (repos, build servers, artifacts, IaC, config, secrets)
- **Security as Code**: Automate IaC scanning, policy-as-code, SAST, DAST, SCA, container scanning

### Secure DevOps Pipeline
- **Package Management**: Approval workflows scanning for vulnerabilities, license reviews, dependency analysis, continuous monitoring
- **Source Code Scanning**: SAST (analyze without execution), secret scanning, code quality analysis
- **Timing**: After build, before testing/deployment (lowest remediation costs)
- **Tools**: CodeQL, SonarQube, Checkmarx, Veracode, Microsoft Security Code Analysis

### Key Validation Points
- **IDE-Level Checks**: Real-time feedback during writing, seconds feedback loop, fix before commit
- **Repository Controls**: Branch policies, code reviews, work item linkage, CI build requirements
- **Code Review Requirements**: Input validation, auth/authz checks, sensitive data handling, no hard-coded secrets
- **Automated PR Checks**: Static analysis, dependency scans, secret detection, code quality (results in PR interface)
- **Gradual Implementation**: Prioritize high-impact checkpoints, build security culture over time

### Continuous Security Validation
- **Third-Party Risks**: Vulnerabilities, license compliance, supply chain attacks
- **Early Detection Value**: 10-100x cheaper in development vs production
- **Static Analysis Tools**: SonarQube, Visual Studio, Checkmarx, BinSkim, language-specific analyzers
- **Vulnerability Scanning**: Mend/WhiteSource, Dependabot, Snyk, Azure Artifacts (continuous monitoring)
- **SCA Benefits**: Comprehensive visibility, version tracking, risk prioritization (CVSS), compliance docs

### Threat Modeling Methodology
- **Five Stages**: Define requirements → Create diagram → Identify threats (STRIDE) → Mitigate → Validate
- **STRIDE**: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
- **Common Mitigations**: SQL injection (parameterized queries), session hijacking (HTTPS + secure sessions), MITM (TLS + cert pinning), DDoS (cloud protection + rate limiting)
- **Lifecycle Integration**: Initial design, new features, periodic reviews, post-incident updates
- **Microsoft Tool**: Free tool with standard notation, auto-threat generation, mitigation tracking, Azure DevOps integration

### CodeQL Automated Security Analysis
- **Semantic Analysis**: Code as queryable database (syntax trees, control flow, data flow paths)
- **Three Phases**: Create database → Run queries → Interpret results (prioritization, context, remediation)
- **Query Language**: Declarative, object-oriented logic programming, standard libraries (OWASP Top 10, CWE)
- **GitHub Integration**: One-click enable, PR inline annotations, required checks, Security tab findings
- **CI/CD Integration**: GitHub Actions, Azure Pipelines, Jenkins, GitLab, CircleCI, CLI for custom systems
- **Dev Tools**: VS Code extension (syntax highlighting, autocomplete, debugging), CLI for scripting

## Key Takeaways

| Principle | Description |
|-----------|-------------|
| **Everyone's Responsibility** | Security not just for security team—devs, ops, testers, business all contribute |
| **Shift Security Left** | Address early (10-100x cheaper than production) |
| **Automate Validation** | Manual reviews don't scale—automate for consistent checks without slowing velocity |
| **Validate Continuously** | Multiple stages (IDE, code review, CI, pre-deployment, production) |
| **Proactive Threat Modeling** | Use STRIDE during design, not waiting for incidents |
| **Leverage Automated Tools** | CodeQL, SonarQube, Snyk for sophisticated analysis impractical manually |
| **Track Dependencies** | Automated scanning for vulnerabilities and license compliance |
| **Document Decisions** | Audit trails for due diligence, onboarding, compliance |
| **Balance Speed & Security** | Integrate efficiently without impeding velocity |
| **Iterate & Improve** | Start with high-impact practices, expand gradually, demonstrate value |

## Core Message
**DevSecOps creates software that is both rapidly delivered and secure by design. Security becomes an enabler, not an obstacle, allowing innovation with confidence.**

## Additional Resources
- [DevSecOps Tools and Services | Microsoft Azure](https://azure.microsoft.com/solutions/devsecops)
- [Enable DevSecOps with Azure and GitHub](https://learn.microsoft.com/en-us/azure/devops/devsecops)
- [Securing Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/security/overview)
- [CodeQL Overview](https://codeql.github.com/docs/codeql-overview/about-codeql/)
- [Microsoft Threat Modeling Tool](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool)

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-secure-devops/10-summary)
