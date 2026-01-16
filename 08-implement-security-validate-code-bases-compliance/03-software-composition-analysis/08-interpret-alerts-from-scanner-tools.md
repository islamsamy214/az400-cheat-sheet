# Interpret Alerts from Scanner Tools

## Key Concepts
- **CVSS Scoring**: Common Vulnerability Scoring System provides standardized 0-10 severity ratings
- **Exploitability Assessment**: Consider exploit availability, active exploitation, attack surface reachability
- **False Positive Management**: Systematically investigate and document false positives with suppression files
- **Risk-Based Prioritization**: Prioritize by severity (25%), exploitability (35%), asset criticality (20%), environmental factors (20%)
- **Security Bug Bars**: Define minimum security standards required before releases

## Understanding Vulnerability Severity

### CVSS Scoring System
**Common Vulnerability Scoring System: Standardized numeric scores (0-10)**

```yaml
CVSS Metrics Categories:
  Base Metrics: Intrinsic vulnerability characteristics (independent of environment)
  Temporal Metrics: Characteristics changing over time (exploit availability, patches, confidence)
  Environmental Metrics: Characteristics specific to particular environments/deployments

CVSS v3 Base Score Calculation:

Exploitability Metrics:
  - Attack Vector (AV): Network (N), Adjacent (A), Local (L), Physical (P)
  - Attack Complexity (AC): Low (L), High (H) difficulty in exploiting
  - Privileges Required (PR): None (N), Low (L), High (H) privileges needed
  - User Interaction (UI): None (N), Required (R) for successful exploitation

Impact Metrics:
  - Confidentiality Impact (C): None (N), Low (L), High (H) information disclosure
  - Integrity Impact (I): None (N), Low (L), High (H) data modification capability
  - Availability Impact (A): None (N), Low (L), High (H) service disruption

Example CVSS Vector:
  CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
  
  Translation: Network-exploitable, low complexity, no privileges required,
  no user interaction, high impact on confidentiality/integrity/availability
```

### Severity Classifications

| Severity | CVSS Score | Description | Action Required |
|----------|-----------|-------------|-----------------|
| **Critical** | 9.0 - 10.0 | Easily exploitable vulnerabilities causing widespread system compromise | Immediate remediation required |
| **High** | 7.0 - 8.9 | Serious vulnerabilities enabling significant information disclosure or service disruption | Remediation within days |
| **Medium** | 4.0 - 6.9 | Moderate vulnerabilities with limited exploitability or impact | Remediation within weeks |
| **Low** | 0.1 - 3.9 | Minor vulnerabilities with minimal security impact | Remediation when convenient |
| **None** | 0.0 | Informational findings without actual security impact | Optional remediation |

**Severity Interpretation Examples:**

```yaml
Critical Vulnerability (CVSS 10.0):
  CVE: CVE-2021-44228 (Log4Shell)
  Vector: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H
  Description: Remote code execution in Apache Log4j 2
  Impact: Unauthenticated attacker can execute arbitrary code remotely
  Exploitability: Actively exploited in the wild with public exploits
  Priority: EMERGENCY

High Vulnerability (CVSS 8.1):
  CVE: CVE-2022-23648
  Vector: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N
  Description: Path traversal in container runtime
  Impact: Authenticated users can access files outside container boundaries
  Exploitability: Requires authentication but easily exploitable
  Priority: CRITICAL

Medium Vulnerability (CVSS 5.9):
  CVE: CVE-2023-12345
  Vector: CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:N/A:N
  Description: Information disclosure through timing attack
  Impact: Sensitive information disclosure with sophisticated attack
  Exploitability: Requires specific timing conditions, difficult to exploit
  Priority: HIGH
```

## Assessing Exploitability

### Exploit Maturity

| Stage | Status | Risk Level | Action |
|-------|--------|-----------|--------|
| **Unproven** | Theoretical vulnerability, no known exploit | Lower risk (requires significant attacker effort) | Monitor for exploit development; plan remediation but not urgent |
| **Proof of Concept** | PoC exploit code published but not weaponized | Moderate risk (sophisticated attackers could weaponize) | Prioritize remediation; develop mitigation strategies |
| **Functional** | Working exploit code publicly available | High risk (accessible to moderately skilled attackers) | Expedite remediation; implement temporary mitigations if patching delayed |
| **Active Exploitation** | Vulnerability actively exploited in the wild | **Critical risk** (exploitation happening now) | **EMERGENCY remediation**; immediate mitigations; monitor for compromise |

**Example Exploitability Assessment:**
```yaml
CVE-2021-44228 (Log4Shell):
  Severity: Critical (CVSS 10.0)
  Exploit Maturity: Active exploitation

  Analysis:
    - Public exploit code available within hours of disclosure
    - Automated scanning and exploitation observed globally
    - Multiple malware families incorporating the exploit
    - Trivial to exploit with single HTTP request

  Priority: EMERGENCY - Immediate patching required
```

### Attack Surface Analysis
**Determine whether vulnerability is reachable**

```yaml
Reachability Factors:
  - Code usage: Is vulnerable code path actually executed by your application?
  - Network exposure: Is vulnerable component exposed to network access?
  - Authentication requirements: Does exploitation require authenticated access?
  - Configuration dependencies: Does vulnerability require specific configurations?

Reachability Example:
  Vulnerability: SQL injection in unused admin feature
  Severity: High (CVSS 8.5)
  Reachability: NOT REACHABLE

  Analysis:
    - Vulnerable code exists in imported library
    - Admin features disabled in production configuration
    - Vulnerable code paths never executed
    - Network access to admin interface blocked by firewall

  Priority: LOW - Update during regular maintenance window
```

### Environmental Context
**Consider your specific environment**

```yaml
Network Segmentation:
  - Internet-facing: Highest priority (public exposure)
  - Internal network: Lower priority if network segmented
  - Isolated systems: Minimal risk for air-gapped/isolated systems

Data Sensitivity:
  - Sensitive data: Systems handling PII/financial data require urgent remediation
  - Public information: Information disclosure lower priority if data already public
  - Test environments: Non-production vulnerabilities typically lower priority

Compensating Controls:
  - Web Application Firewall: WAF rules may mitigate exploitation attempts
  - Intrusion Detection: IDS/IPS can detect and block exploitation
  - Network Segmentation: Isolation limits exploitation impact
  - Least Privilege: Restricted permissions reduce impact
```

## Managing False Positives

### Common False Positive Causes

```yaml
Misidentified Components:
  - Naming conflicts: Different components with similar names incorrectly matched
  - Version detection errors: Incorrect version identification
  - Package namespace confusion: Packages in different ecosystems incorrectly identified

Example False Positive:
  Alert: CVE-2022-12345 in "parser" package (npm)
  Severity: High

  Investigation:
    - Application uses "xml-parser" package
    - Scanner incorrectly identified "xml-parser" as vulnerable "parser" package
    - Different packages with similar names
    - Vulnerability does not affect application

  Resolution: Suppress false positive with documented justification

Unused Code Paths:
  - Dead code: Vulnerable code imported but never executed
  - Optional features: Vulnerabilities in optional features not enabled
  - Development dependencies: Packages used only during development, not production

Version Range Errors:
  - Fixed version reporting: Scanners report vulnerability in versions already patched
  - Backport patches: Vendors backport fixes without changing version numbers
  - Custom patches: Vulnerabilities already patched through custom modifications
```

### False Positive Verification

```yaml
Investigation Process:
  1. Verify component identity: Confirm scanner correctly identified component
  2. Check version accuracy: Verify detected version matches actual deployed version
  3. Review vulnerability details: Understand what vulnerability affects
  4. Analyze code usage: Determine if vulnerable code paths actually used
  5. Consult vendor advisories: Check if vendor provides additional context
  6. Test exploitation: Attempt to reproduce vulnerability in test environment

Documentation Requirements (When Suppressing):
  - Justification: Why the finding is a false positive
  - Investigation details: Steps taken to verify
  - Approver: Security team member approving suppression
  - Review date: Date to re-evaluate suppression
```

**Example Suppression File (OWASP Dependency-Check):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<suppressions xmlns="https://jeremylong.github.io/DependencyCheck/dependency-suppression.1.3.xsd">
    <suppress>
        <notes>
            False positive: CVE-2022-12345 affects "parser" package but we use "xml-parser".
            Scanner incorrectly matched based on partial name match.
            Investigated by security team on 2025-10-21.
            Review annually.
        </notes>
        <packageUrl regex="true">^pkg:npm/xml-parser@.*$</packageUrl>
        <cve>CVE-2022-12345</cve>
    </suppress>
</suppressions>
```

## Prioritization Frameworks

### Risk-Based Prioritization

**Prioritization Factors (Weighted):**
- **Severity Score (25%)**: CVSS base score provides foundation
- **Exploitability (35%)**: Most critical factor (active exploitation status, public exploit availability)
- **Asset Criticality (20%)**: Internet-facing applications, sensitive data systems, business-critical apps
- **Environmental Factors (20%)**: Compensating controls, network segmentation, monitoring capabilities

**Prioritization Matrix:**

| Asset Criticality | Active Exploitation | Functional Exploit | Proof of Concept | Unproven |
|-------------------|--------------------|--------------------|------------------|----------|
| **Internet-facing, Critical** | P0 (Emergency) | P0 (Emergency) | P1 (Critical) | P2 (High) |
| **Internet-facing, Standard** | P0 (Emergency) | P1 (Critical) | P2 (High) | P3 (Medium) |
| **Internal, Critical** | P1 (Critical) | P2 (High) | P3 (Medium) | P4 (Low) |
| **Internal, Standard** | P2 (High) | P3 (Medium) | P4 (Low) | P5 (Optional) |

### Remediation SLAs

| Priority | Timeframe | Criteria | Process | Example |
|----------|-----------|----------|---------|---------|
| **P0 (Emergency)** | 24 hours | Critical vulnerabilities under active exploitation in internet-facing systems | Emergency change process with executive notification | Log4Shell in public-facing app |
| **P1 (Critical)** | 7 days | High/critical with functional exploits or internet-facing exposure | Expedited change process with security team coordination | SQL injection in authenticated admin interface |
| **P2 (High)** | 30 days | Medium/high with PoC exploits or internal exposure | Standard change process with planned maintenance | XSS in internal dashboard |
| **P3 (Medium)** | 90 days | Low/medium without known exploits | Regular update cycle with quarterly patching | Information disclosure in dev tool dependency |
| **P4 (Low)** | Next major release | Low severity or false positives requiring documentation | Regular maintenance activities | DoS in unused optional feature |

## Establishing Security Bug Bars

### Definition of Done Criteria

```yaml
Security Bug Bar:

Blocking Issues (Must Fix Before Release):
  - No critical severity vulnerabilities
  - No high severity vulnerabilities with public exploits
  - No vulnerabilities actively exploited in the wild
  - No strong copyleft licenses (GPL, AGPL) in proprietary code
  - No secrets in source code or container images

Non-Blocking Issues (Track and Plan):
  - High severity without public exploits (90-day remediation plan)
  - Medium severity vulnerabilities (next minor release)
  - License issues requiring legal review (document plan)

Informational (Monitor):
  - Low severity vulnerabilities
  - Informational security findings
  - Code quality issues
```

### Policy Enforcement

**Azure Pipelines Quality Gate:**
```yaml
- task: WhiteSource@21
  inputs:
    cwd: "$(System.DefaultWorkingDirectory)"
    projectName: "$(Build.Repository.Name)"
    checkPolicies: true
    failBuildOnPolicyViolation: true
  displayName: "Enforce security bug bar"

- script: |
    # Custom policy check script
    if [ $(jq '.vulnerabilities.critical' scan-results.json) -gt 0 ]; then
      echo "##vso[task.logissue type=error]Critical vulnerabilities detected"
      echo "##vso[task.complete result=Failed;]Failed security bug bar"
      exit 1
    fi
  displayName: "Validate security bug bar compliance"
```

## Vulnerability Triage Workflow

### Triage Process

```yaml
Step 1: Automated Filtering
  - Scanner tools: Automatically filter by severity
  - Reachability analysis: Remove unreachable vulnerabilities
  - Known false positives: Auto-suppress previously identified false positives

Step 2: Initial Assessment
  - Severity review: Verify CVSS score accuracy for your environment
  - Exploitability check: Determine exploit availability and active exploitation
  - Asset identification: Identify affected applications and systems

Step 3: Risk Evaluation
  - Business impact: Assess potential business impact of exploitation
  - Exposure analysis: Determine network exposure and attack surface
  - Compensating controls: Identify existing mitigations

Step 4: Prioritization
  - Assign priority: Use prioritization matrix
  - Set due date: Assign remediation deadline based on SLA
  - Assign owner: Designate responsible team

Step 5: Remediation Tracking
  - Create tickets: Generate work items in Jira, Azure Boards
  - Progress monitoring: Track remediation against deadlines
  - Verification: Validate successful remediation through re-scanning
```

### Triage Meeting Cadence

```yaml
Weekly Security Triage (30-60 minutes):
  Participants: Security team, development leads, operations
  Agenda: Review new high/critical findings, track remediation progress, adjust priorities

Monthly Vulnerability Review (60-90 minutes):
  Participants: Security leadership, engineering management, compliance team
  Agenda: Review trends, adjust policies, assess overall security posture
```

## Metrics and Reporting

### Key Metrics

```yaml
Vulnerability Metrics:
  - Mean Time to Detect (MTTD): Time from vulnerability disclosure to detection in your systems
  - Mean Time to Remediate (MTTR): Time from detection to successful remediation
  - Vulnerability Density: Number of vulnerabilities per application or line of code
  - Remediation Rate: Percentage of vulnerabilities remediated within SLA

Trend Metrics:
  - Open Vulnerability Count: Trending unresolved vulnerabilities by severity
  - New vs. Resolved: Comparison of newly detected vs remediated vulnerabilities
  - SLA Compliance: Percentage remediated within defined SLAs
  - False Positive Rate: Percentage of findings determined to be false positives
```

### Dashboard Example

```yaml
Vulnerability Management Dashboard:

Critical Vulnerabilities: 0 (Target: 0) ✓
High Vulnerabilities: 3 (Target: < 10) ✓
Medium Vulnerabilities: 47 (Target: < 100) ✓
Low Vulnerabilities: 132 (Tracking only)

Mean Time to Remediate:
  - Critical: 18 hours ✓
  - High: 6 days ✓
  - Medium: 21 days ✓

Remediation Progress:
  - P0 (Emergency): 0 overdue
  - P1 (Critical): 1 due in 3 days
  - P2 (High): 5 due in next 30 days
  - P3 (Medium): 12 due in next 90 days

Trends (Last 90 Days):
  - New vulnerabilities: 127
  - Remediated: 138
  - Net reduction: -11 ✓
```

## Critical Notes
- ⚠️ **CVSS Isn't Everything**: Base score doesn't tell complete story; assess exploitability, reachability, environmental context
- 💡 **Exploitability Weighs Most**: Active exploitation (35% weight) is the most critical prioritization factor
- 🎯 **Risk-Based Prioritization**: Combine severity + exploitability + asset criticality + environmental factors
- 📊 **Remediation SLAs**: Emergency (24h), Critical (7d), High (30d), Medium (90d), Low (next release)
- 🔒 **Security Bug Bars**: Define "Definition of Done" criteria (e.g., no critical, no actively exploited vulnerabilities)
- 🚀 **Systematic Triage**: Automated filtering → Initial assessment → Risk evaluation → Prioritization → Tracking

[Learn More](https://learn.microsoft.com/en-us/training/modules/software-composition-analysis/8-interpret-alerts-from-scanner-tools)
