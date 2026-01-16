# Understand DevSecOps

## Key Concepts
- **DevSecOps** (Secure DevOps): Integration of security practices throughout the DevOps lifecycle
- **Security as Code**: Automated security practices as robust and fast as other pipeline components
- **Shift Left**: Security from the beginning, not a gate at the end
- **Continuous Security**: Validation throughout development, deployment, and operation

## The Problem: Security vs Speed Tension

**Traditional Security Issues**:
- Security addressed only at end of development cycle
- Unplanned work appears before deployment under release pressure
- Late discoveries require expensive rework
- Security becomes a bottleneck blocking releases
- Critical vulnerabilities deprioritized due to deadlines

**DevSecOps Solution**:
- Integrates security with DevOps from the beginning
- Security and velocity become complementary, not opposing
- Automated security controls don't slow delivery

## Security Gaps in Cloud Applications

| Gap Type | Issues |
|----------|--------|
| **Encryption** | Unencrypted data at rest (databases, storage), unencrypted data in transit |
| **Session Protection** | Missing HTTP security headers (HSTS, CSP), insecure token transmission |
| **Token Exposure** | Session tokens intercepted and reused for impersonation |

## How Security Changes in DevSecOps

**Traditional Security Focus**:
- Access control (authentication/authorization)
- Environment hardening (patches, secure configs)
- Perimeter protection (firewalls, IDS, network segmentation)

**Expanded DevSecOps Scope**:
- **Pipeline Security**: Source repos, build servers, artifact storage, deployment tools
- **Infrastructure as Code**: Automated scanning of IaC templates
- **Configuration Management**: Secure config systems, secrets management
- **Continuous Validation**: Security at every stage, not single checkpoint

## Key Questions DevSecOps Addresses

```yaml
Third-Party Components:
  - Are pipeline components secure and from trusted sources?
  - Are signatures verified? Licenses compatible?

Vulnerability Management:
  - Known vulnerabilities in dependencies?
  - Process for tracking and updating vulnerable components?
  - Inventory of all dependencies including transitive?

Detection Speed:
  - How quickly can vulnerabilities be detected?
  - Are scans automated? Runtime monitoring active?

Remediation Speed:
  - How quickly can vulnerabilities be fixed?
  - Can fixes deploy through automated pipeline?
  - Emergency patch process defined?
```

## Security Automation Types

**Infrastructure Security**:
- Automated IaC template scanning
- Policy-as-code enforcement
- Compliance checking before deployment
- Continuous monitoring of security posture

**Application Security**:
- SAST (Static Application Security Testing) during builds
- DAST (Dynamic Application Security Testing) in staging
- SCA (Software Composition Analysis) for dependencies
- Container image scanning before deployment
- RASP (Runtime Application Self-Protection) in production

## Critical Notes
- ⚠️ DevSecOps = DevOps + Security (also called Secure DevOps)
- 💡 Security controls are automated and integrated, not manual
- 🎯 Secure the pipeline and everything flowing through it
- 📊 Every pipeline stage includes appropriate security controls

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-secure-devops/3-understand-devsecops)
