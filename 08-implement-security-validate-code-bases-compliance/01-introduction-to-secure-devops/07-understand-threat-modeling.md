# Understand Threat Modeling

## Key Concepts
- **Threat Modeling**: Structured approach to systematically identifying security risks
- **STRIDE**: Threat categorization framework (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege)
- **Progressive Risk Reduction**: Iteratively refine threat model over time
- **Accessible to All**: Designed for non-security experts, all developers

## What is Threat Modeling

**Purpose**: Systematic process to identify and address threats, not just hoping you've thought of everything

**Benefits**:
- Shape application design based on security considerations
- Meet security goals and compliance requirements
- Reduce risk by addressing threats during design, not production
- Prioritize security work on most significant threats
- Provide common language for devs, security teams, stakeholders

## Five-Stage Threat Modeling Process

### 1. Define Security Requirements

```yaml
Confidentiality:
  - What data must remain confidential?
  - Who should access sensitive information?
  - How long must data remain confidential?
  - Consequences of breaches?

Integrity:
  - What must be protected from unauthorized modification?
  - How to detect tampering?
  - Consequences of violations?

Availability:
  - What uptime guarantees needed?
  - Acceptable downtime per component?
  - Business impact of unavailability?

Compliance:
  - Regulatory requirements (GDPR, HIPAA, PCI DSS)?
  - Industry standards (ISO 27001, SOC 2)?
  - Customer contractual obligations?
```

**Example**: "Customer payment info must remain confidential. Only authorized payment systems access this data. All access logged. Data encrypted in transit and at rest."

### 2. Create Application Diagram

**Visualize Architecture**:
- System components (web servers, databases, microservices, external APIs)
- Data flows (how data moves, what's transmitted, direction, transformations)
- Security boundaries (trust, network, process, physical boundaries)

**Example Elements**:
- Users → Internet → WAF → Web Servers (DMZ) → App Servers (protected network) → Database (restricted network) → External Payment Gateway

### 3. Identify Threats Using STRIDE

| STRIDE Category | Definition | Examples |
|-----------------|------------|----------|
| **Spoofing** | Identity impersonation | Session hijacking, credential theft |
| **Tampering** | Unauthorized data modification | SQL injection, MITM attacks |
| **Repudiation** | Users deny performing actions | Missing audit logs, weak authentication |
| **Information Disclosure** | Unauthorized information access | Data leaks, unencrypted transmissions |
| **Denial of Service** | Prevent legitimate access | DDoS attacks, resource exhaustion |
| **Elevation of Privilege** | Gain unauthorized permissions | Privilege escalation, broken access control |

**Apply STRIDE to Data Flows**:
- Can attacker spoof data source?
- Can data be tampered during transmission/storage?
- Can legitimate actions be repudiated?
- Could sensitive info be disclosed?
- Could flow cause denial of service?
- Could flow be exploited for elevated privileges?

### 4. Mitigate Threats

**Mitigation Strategies**:
```yaml
Eliminate: Remove vulnerable component if not essential
Prevent:   Make threat impossible (e.g., input validation)
Detect:    Monitoring and alerting for exploitation attempts
Respond:   Incident response procedures for exploited threats
```

**Example Mitigations**:
- SQL injection → Parameterized queries + input validation
- Session hijacking → HTTPS + secure cookies + short timeouts
- MITM → Enforce TLS + certificate pinning
- DDoS → Cloud DDoS protection + rate limiting

**Document Decisions**:
- Which threats addressed
- Mitigation approach chosen and why
- Who's responsible for implementation
- Residual risks remaining

### 5. Validate Mitigations

**Security Testing**:
- Penetration testing to verify threats can't be exploited
- Security code reviews for proper implementation
- Automated security scanning for missed vulnerabilities
- Red team exercises for realistic attack scenarios

**Continuous Validation**:
- New threats emerge as technology changes
- Existing mitigations may become ineffective
- Application changes introduce new vulnerabilities

## Threat Modeling in Development Lifecycle

```yaml
Initial Design:    Comprehensive modeling to influence architecture
Feature Development: Model when adding features changing security boundaries
Regular Updates:   Periodic reviews as threat landscape evolves
Incident Response: Update models after security incidents
```

**Progressive Risk Reduction**: Iteratively refine threat model over time, don't attempt to address all risks at once

## Microsoft Threat Modeling Tool

**Key Capabilities**:
- Create visual representations using standard notation
- Automated threat identification based on components/data flows
- STRIDE-based threat analysis
- Automatic threat list generation
- Mitigation suggestions and tracking
- Reports for security reviews and compliance
- Azure DevOps integration (link threats to work items)

**Benefits**:
- Free tool from Microsoft
- Templates for common application types
- Built-in guidance for new users
- Accessible to all developers, not just security specialists

## Critical Notes
- ⚠️ Threat modeling shouldn't be one-time activity
- 💡 Conduct during initial design to influence architecture
- 🎯 Update after security incidents to incorporate lessons
- 📊 Tool auto-suggests threats based on application structure

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-secure-devops/7-understand-threat-modeling)
