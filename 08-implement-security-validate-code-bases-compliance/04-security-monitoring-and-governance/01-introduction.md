# Introduction

## Module Overview
**Scenario**: Contoso Financial Services faces security challenges:
- Pipeline security gaps (hardcoded secrets, weak service connections)
- Cloud security posture issues (unpatched VMs, misconfigured storage)
- Governance enforcement gaps (non-compliant deployments)
- Identity threats (credential theft, lateral movement)
- Development workflow security (vulnerable dependencies, exposed secrets)

**Duration**: 2 hours | **Units**: 13

## Learning Objectives
- **Pipeline security**: Secrets management, service connections, agent security, deployment gates, audit logging
- **Microsoft Defender for Cloud**: CSPM/CWP configuration, Secure Score, threat detection, compliance monitoring
- **Azure Policy**: Policy-as-code governance, compliance enforcement, remediation automation
- **Resource locks**: Protection against accidental deletion/modification
- **Microsoft Defender for Identity**: Identity threat detection, credential theft prevention
- **GitHub Advanced Security**: Code scanning, secret scanning, dependency management integration

## Prerequisites
- DevOps fundamentals (CI/CD, pipelines, Git)
- Azure experience (resource management, RBAC, networking)
- Security awareness (authentication, encryption, threat models)
- GitHub familiarity (repositories, actions, pull requests)

## Key Topics
| Topic | Focus Area | Tools/Services |
|-------|-----------|----------------|
| **Pipeline Security** | Secrets, connections, agents, gates | Azure Key Vault, RBAC, OWASP ZAP |
| **Cloud Security** | Posture management, threat protection | Defender for Cloud, Secure Score |
| **Governance** | Policy enforcement, compliance | Azure Policy, Initiatives |
| **Resource Protection** | Prevent accidental changes | Resource Locks |
| **Identity Security** | Credential theft, lateral movement | Defender for Identity |
| **Code Security** | Vulnerable code, secrets, dependencies | GitHub Advanced Security |

## Critical Notes
- 🎯 **Shift-left security**: Integrate security checks early in development lifecycle
- 💡 **Defense in depth**: Multiple security layers (pipeline + cloud + identity + code)
- ⚠️ **Compliance automation**: Use policy-as-code for consistent enforcement
- 🔒 **Least privilege**: Apply minimum necessary permissions across all systems
- 📊 **Continuous monitoring**: Real-time threat detection and response

[Learn More](https://learn.microsoft.com/en-us/training/modules/security-monitoring-and-governance/)
