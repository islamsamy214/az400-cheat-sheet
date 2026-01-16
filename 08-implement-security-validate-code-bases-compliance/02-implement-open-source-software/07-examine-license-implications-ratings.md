# Examine License Implications and Ratings

## License Implications for Commercial Software

### Permissive License Implications (MIT, Apache 2.0, BSD)

**Minimal Restrictions**:
- ✅ Incorporate into proprietary software without open-sourcing
- ✅ Build and sell commercial products
- ✅ No source code provision requirement
- ✅ Can distribute under your own license

**Primary Obligation**: Attribution (preserve copyright notices, license text in docs/About dialogs)

**Patent Considerations**:
- MIT/BSD: No explicit patent grant (potential ambiguity)
- Apache 2.0: Explicit patent grant + defensive termination

**Business Implications**: Safe, simple compliance, maximum flexibility for any business model

### Weak Copyleft Implications (LGPL, MPL)

**Library Use Allowed**:
- ✅ Use in proprietary applications
- ✅ Link proprietary code (especially dynamic linking)
- ✅ No full copyleft trigger for entire app

**Modification Requirements**:
- ⚠️ Library modifications must be open-sourced
- ⚠️ MPL: File-level tracking (clearer boundaries)
- ⚠️ Derivative works trigger open-source requirements

**Compliance Considerations**:
- Must provide source for library (including modifications)
- Maintain license terms for library
- Clear separation between library + proprietary code

**Business Implications**: Generally acceptable, modification creates ongoing obligations, prefer dynamic linking

### Strong Copyleft Implications (GPL, AGPL)

**Copyleft Propagation**:
- ⚠️ Derivative/combined works trigger copyleft
- ⚠️ Entire application under GPL (not just component)
- ⚠️ Must provide complete source code
- ⚠️ Derivative works must use GPL

**Distribution Triggers**:
- Binary distribution → source code required
- AGPL: Network use triggers requirements
- Internal use only → doesn't trigger requirements

**Linking Concerns**:
```yaml
Static Linking:    Clearly creates derivative work (GPL required)
Dynamic Linking:   Legal interpretation varies
Process Separation: Microservices might avoid derivative status
```

**Business Implications**: Incompatible with proprietary distribution, OK for open-source products, GPL v2/v3 OK for SaaS (AGPL not), high risk for commercial proprietary

## License Risk Ratings

| Risk Level | Licenses | Use Cases | Compliance Burden |
|------------|----------|-----------|-------------------|
| **Low (Green)** | MIT, BSD, Apache 2.0, ISC | Safe for any commercial use | Low (attribution) |
| **Medium (Yellow)** | LGPL, MPL, EPL | OK for unmodified libraries, caution on modifications | Moderate (track/provide library source) |
| **High (Red)** | GPL v2/v3, AGPL | Incompatible with proprietary distribution | High (complete source required) |
| **Unknown (Orange)** | Custom, unclear, missing, obsolete | Avoid until legal review | Unknown |

### Factors Affecting Risk Assessment

**Business Model**:
```yaml
Open-Source Products: GPL acceptable
Proprietary Software: GPL unacceptable
SaaS Offerings:      GPL v2/v3 OK, AGPL unacceptable
Internal Tools:      All licenses typically OK
```

**Distribution Method**:
- Binary distribution → triggers GPL
- SaaS deployment → triggers AGPL
- Internal use only → doesn't trigger most requirements
- Library provision → triggers LGPL/MPL

**Modification Extent**:
- Unmodified → lower burden
- Modified → increased obligations (especially copyleft)
- Deep integration → more complex compliance

**Legal Environment**:
- Jurisdictional interpretation differences
- Enforcement history
- Patent clause interactions

## Intellectual Property Considerations

### Proprietary IP Protection

| License Type | IP Status |
|--------------|-----------|
| **Permissive** | Proprietary code remains proprietary, trade secrets protected, competitive advantage maintained |
| **Strong Copyleft** | IP disclosure required, trade secrets lost, competitive advantage reduced |

### Patent Considerations
```yaml
Apache 2.0: Explicit patent grant + defensive termination
GPL v3:     Patent grant + anti-tivoization
MIT/BSD:    No explicit provisions (potential ambiguity)

Defensive Termination: Patent grants terminate if you sue contributors
Strategic Implication:  Complications for aggressive patent strategies
```

## Compliance Implementation

### Dependency Inventory
```yaml
Bill of Materials:      Complete inventory of all OSS components
Transitive Dependencies: Track deps of deps
Version Tracking:       Specific versions (licenses change)
Update Monitoring:      Continuous monitoring for updates/licenses
```

**Automated Tools**:
- Package managers: npm, pip, Maven (direct deps)
- SBOM tools: Generate comprehensive inventories
- License scanners: FOSSA, WhiteSource, Black Duck (identify licenses)

### License Compatibility Verification
```yaml
Automated Scanning: Tools identify incompatibilities
Legal Review:       Complex cases need legal expertise
Approval Workflows: Process for reviewing new dependencies
```

**Common Incompatibilities**:
- ❌ GPL v2 + Apache 2.0
- ❌ GPL + Proprietary (for distributed software)
- ❌ Multiple copyleft licenses (generally)

### Attribution Compliance
```yaml
License Aggregation:   Collect all license texts (LICENSES.txt, THIRD_PARTY_NOTICES)
About Dialogs:        Include attributions in app About/settings
Documentation:        Include in product docs
Automated Generation: Tools generate attribution files from dependency data
```

### Source Code Provision (Copyleft)
- Provide complete source for GPL components + derivatives
- Internet download now acceptable (not just same medium)
- Can offer written offer to provide source
- Include compilation/build instructions

## Software Supply Chain Security

### Vulnerability Management
```yaml
Chain Dependency:        Security = weakest component
Vulnerability Propagation: Any dep vulnerability affects all apps
Update Urgency:         Critical vulns need rapid updates

Scanning Tools: Snyk, Dependabot, WhiteSource
CVE Monitoring: Track Common Vulnerabilities and Exposures
CVSS Scoring:   Prioritize remediation
Rapid Response: Quick update processes
```

### Supply Chain Attack Mitigation
```yaml
Package Verification: Verify signatures + checksums
Source Reputation:   Active maintenance, large user base, reputable maintainers
Private Registries:  Mirror public packages for control
Dependency Pinning:  Lock versions to prevent auto-updates to compromised versions
```

### Component Quality Assessment
```yaml
Active Maintenance:  Regular updates
Community Size:     Large communities = better sustainability
Documentation:      Professional maintenance indicator
Test Coverage:      Automated testing = quality focus
Security Practices: Responsible disclosure, security advisories
```

## Organizational Policies

### Approval Workflows
```yaml
Pre-Use Evaluation:
  - Security review (scan for vulnerabilities)
  - License review (verify compatibility)
  - Quality assessment (code quality, maintenance, community)
  - Alternative analysis (approved alternatives exist?)

Approved Package Lists:
  - Pre-vetted components
  - Faster adoption
  - Periodic re-evaluation
```

### Developer Education
```yaml
Training Programs:
  - License awareness (types, implications)
  - Security practices (component evaluation)
  - Compliance processes (request approval)
  - Risk awareness (organizational concerns)
```

### Continuous Monitoring
```yaml
Dependency Updates:    Monitor new versions, security patches
License Changes:      Track when projects change licenses
Vulnerability Disclosure: Subscribe to security advisories
Compliance Audits:    Periodic application audits
```

## Quick Decision Matrix

| Scenario | Permissive | Weak Copyleft | Strong Copyleft |
|----------|------------|---------------|-----------------|
| **Proprietary Product** | ✅ Use freely | ⚠️ Use library, don't modify | ❌ Avoid |
| **SaaS Offering** | ✅ Use freely | ⚠️ Use library, don't modify | ⚠️ GPL OK, AGPL avoid |
| **Open-Source Project** | ✅ Use freely | ✅ Use freely | ✅ Use freely |
| **Internal Tool** | ✅ Use freely | ✅ Use freely | ✅ Use freely |

## Critical Notes
- ⚠️ GPL incompatible with proprietary distribution (high risk for commercial software)
- 💡 AGPL triggers on network use (closes SaaS loophole)
- 🎯 License compatibility verification essential before combining components
- 📊 Automated tools (SBOM, license scanners) required for comprehensive tracking

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-open-source-software-azure/7-examine-license-implications-ratings)
