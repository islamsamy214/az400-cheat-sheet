# Inspect and Validate Code Bases for Compliance

## Why Inspection and Validation Matter

**The Dependency Challenge**: 
- Direct deps: 20-30 packages
- Transitive deps: 200-500 packages total
- Responsible for vulnerabilities + licenses in ALL dependencies

**Critical Drivers**:
- Security: Attackers exploit known vulnerabilities in popular packages
- Compliance: License violations → legal action, forced open-sourcing, distribution restrictions
- Operational: Dependencies constantly change (new vulnerabilities daily, updates, abandonments)
- Continuous: One-time inspection insufficient; continuous validation required

## What to Inspect and Validate

### 1. Dependency Inventory (Complete Bill of Materials)
```yaml
Direct Dependencies:      Explicitly listed in manifest (package.json, requirements.txt, pom.xml, *.csproj)
Transitive Dependencies:  Required by direct deps, multiple levels deep
Development Dependencies: Used in dev/test, not shipped to production
Version Tracking:         Specific versions currently in use
Dependency Sources:       Registries providing deps (npm, PyPI, NuGet, Maven Central, private)
```

**Why Complete Inventories Matter**: Can't fix unknown vulnerabilities, track license obligations, plan updates, have supply chain visibility

### 2. Security Vulnerability Detection
```yaml
CVE Mapping:           Match versions against CVE databases
Severity Assessment:   CVSS scores (0-10)
Exploitability Analysis: Actively exploited in wild?
Patch Availability:    Which versions fix? Patches available?
```

**Vulnerability Databases**:
- National Vulnerability Database (NVD): US government CVE repository
- GitHub Advisory Database: Curated security advisories
- Security mailing lists: Language/framework-specific
- Vendor databases: Commercial SCA proprietary intelligence

**Vulnerability Categories**: Injection flaws, authentication issues, sensitive data exposure, security misconfiguration, DoS, deserialization flaws

### 3. License Compliance Validation
```yaml
License Detection:      Identify licenses for each dependency
License Classification: Permissive, weak copyleft, strong copyleft, proprietary
Compatibility Analysis: Verify licenses compatible when combined
Obligation Tracking:    Attribution, source code disclosure, derivative work licensing
```

**Common Compliance Issues**:
- Copyleft contamination (GPL in proprietary → force open-sourcing)
- Attribution failures (missing copyright notices)
- Incompatible combinations (conflicting licenses)
- License changes (between versions)

**License Risk Assessment**:
| Risk Level | Licenses | Use Case |
|------------|----------|----------|
| High | GPL, AGPL | Proprietary software distribution |
| Medium | LGPL, MPL | Careful integration required |
| Low | MIT, Apache 2.0, BSD | Minimal restrictions |
| Unknown | Custom, unclear, missing | Needs clarification |

### 4. Dependency Quality Assessment
```yaml
Maintenance Status:  Actively maintained or abandoned?
Update Frequency:    How often updates/bug fixes?
Community Health:    Contributor activity, issue response, engagement
Documentation:       Adequate docs for proper use?
Security Practices:  Responsible disclosure, security advisories?
```

**Quality Indicators**: Active maintenance, large community, clear communication, security awareness

**Red Flags**: Abandoned projects, single maintainer risk, poor quality, lack of security

## Manual Inspection Challenges

| Challenge | Issues |
|-----------|--------|
| **Volume Overwhelming** | Hundreds of deps per app × dozens/hundreds of apps, constant changes, human error |
| **Information Scattered** | Multiple sources (CVE, mailing lists, GitHub, vendors), license research, version-specific, conflicting info |
| **Time-Consuming** | Vulnerability assessment, license analysis, update planning, continuous monitoring need dedicated resources |
| **Delayed Detection** | Discovery lag (weeks/months), reactive response, audit cycles leave gaps, emergency patches disruptive |

## The Automated Solution (SCA Tools)

### Automated Discovery
- Dependency parsing (manifest files)
- Transitive resolution (complete BOM)
- Lock file analysis (exact versions)
- Binary scanning (containers, compiled binaries)

### Continuous Monitoring
- Real-time vulnerability alerts
- Automated updates (Dependabot PRs)
- Dashboard visibility (all apps)
- Scheduled scanning

### Comprehensive Databases
- Aggregated vulnerability data (NVD, GitHub, mailing lists, vendor)
- License databases (full texts, obligation summaries)
- Curation/verification (reduce false positives)
- Proprietary intelligence (commercial tools)

### Intelligent Analysis
- Severity scoring (auto-calculate CVSS, prioritize by risk)
- Reachability analysis (vulnerable code paths actually used?)
- Remediation guidance (version recommendations maintaining compatibility)
- Policy enforcement (auto-fail builds, block deployments on violations)

## Establishing Validation Baselines

### Security Policies
```yaml
Vulnerability Tolerance: Which severities acceptable (no critical/high, limited medium)
Patch Timeframes:       How quickly to remediate by severity
Exception Processes:    Accept risks when immediate patching infeasible
Reporting Requirements: Who notified, how quickly
```

### Compliance Policies
```yaml
Approved Licenses:      Always acceptable (MIT, Apache 2.0)
Restricted Licenses:    Special approval needed (LGPL) or prohibited (GPL for proprietary)
Attribution Requirements: How attribution provided in distributed software
Audit Trails:          Documentation for compliance evidence
```

### Quality Standards
```yaml
Maintenance:    Minimum expectations (updates within last year)
Community Size: Thresholds for community health metrics
Documentation:  Minimum requirements
Security:       Published security policies, responsive vulnerability handling
```

## Critical Notes
- ⚠️ Typical app: 20-30 direct deps, 200-500 transitive deps
- 💡 Manual tracking doesn't scale; automated SCA essential
- 🎯 Delayed detection = weeks/months lag vs real-time alerts
- 📊 Complete inventory prerequisite for vulnerability/compliance management

[Learn More](https://learn.microsoft.com/en-us/training/modules/software-composition-analysis/2-inspect-validate-code-bases-for-compliance)
