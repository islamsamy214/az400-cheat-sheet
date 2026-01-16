# Explore Common Open-Source Licenses

## The License Spectrum

```
Permissive ←――――――――――――――――――――――――→ Copyleft
(Minimal)     Weak Copyleft        (Strong)
MIT, Apache   LGPL, MPL           GPL, AGPL
```

| Category | Restrictions | Examples |
|----------|--------------|----------|
| **Permissive** | Minimal (attribution only) | MIT, Apache 2.0, BSD |
| **Weak Copyleft** | Library-friendly, component-level | LGPL, MPL, EPL |
| **Copyleft** | Strong reciprocal requirements | GPL v2/v3, AGPL |

## Common Permissive Licenses

### MIT License
```yaml
Permissions:  Use, copy, modify, merge, publish, distribute, sublicense, sell
Conditions:   Include copyright notice + license text
Limitations:  No warranty, no liability
```

**Why Projects Choose MIT**:
- Maximum adoption (minimal restrictions)
- Simple, clear, short license
- Commercial-friendly (no barriers to proprietary use)
- Complete flexibility

**Popular Projects**: React, Angular, Node.js, jQuery, Rails, .NET Core

**Organizational Implications**:
- ✅ Safe for commercial use without restrictions
- ⚠️ Attribution required (license text in docs/About dialogs)
- ⚠️ No explicit patent grant (potential ambiguity)

### Apache License 2.0
```yaml
Permissions:  Use, reproduce, derivative works, display, perform, sublicense, distribute
Conditions:   Copyright notice, license text, notice of modifications, attribution
Patent Grant: Explicit grant from contributors
Patent Retaliation: Grants terminate if licensee initiates patent litigation
Limitations:  No warranty, no liability, no trademark rights
```

**Why Projects Choose Apache 2.0**:
- Patent clarity (explicit grants = legal certainty)
- Modification transparency (document changes)
- Corporate confidence (clear terms, patent protections)
- GPL v3 compatible (not GPL v2)

**Popular Projects**: Kubernetes, TensorFlow, Android, Spring Framework, Hadoop, Kafka

**Organizational Implications**:
- ✅ Patent protection reduces litigation risk
- ⚠️ Must indicate modified files
- ⚠️ More complex attribution (NOTICE files)
- ⚠️ Defensive termination if you sue contributors

### BSD Licenses (2-Clause and 3-Clause)

**BSD 2-Clause (Simplified)**:
- Redistribution with/without modification
- Preserve copyright notice, conditions, disclaimer
- Retain attribution in binary docs

**BSD 3-Clause (Modified)**:
- Same as 2-Clause + name usage restriction
- Cannot use copyright holder names to endorse without permission

**Popular Projects**: FreeBSD, OpenBSD, macOS/iOS parts

**Organizational Implications**:
- Similar to MIT (minimal, commercial-friendly)
- 3-Clause prevents marketing with project names
- Long history in academic/commercial software

## Common Copyleft Licenses

### GNU General Public License (GPL) v2 and v3

**GPL v2 Key Terms**:
```yaml
Permissions: Use, modify, distribute
Conditions:  Distribute source with binaries, derivatives use GPL v2
Copyleft Scope: Derivative + combined works linking with GPL code
Limitations: No warranty, no liability
```

**GPL v3 Enhancements**:
- Explicit patent grants (like Apache 2.0)
- Tivoization prevention (hardware restrictions blocked)
- International legal clarity
- Apache 2.0 compatible

**Why Projects Choose GPL**:
- Ensures modifications remain open-source
- Prevents proprietary forks
- Encourages community sharing
- Free software philosophy alignment

**Popular Projects**: Linux kernel (v2), Git (v2), WordPress (v2), GCC (v3), Bash (v3)

**Organizational Implications**:
- ⚠️ Modified/derivative works must be open-sourced under GPL
- ⚠️ Linking concerns (proprietary + GPL libraries)
- ✅ Can sell GPL software (must provide source to customers)
- ℹ️ SaaS doesn't require disclosure (unless AGPL)

### GNU Affero General Public License (AGPL)

**Additional AGPL Requirement**:
```yaml
Network Copyleft: Modified AGPL software + network interaction = 
                  must provide source to users
Closes ASP Loophole: Prevents SaaS use without contributing back
```

**Why Projects Choose AGPL**:
- SaaS protection (cloud services must contribute)
- Maximum copyleft protection

**Popular Projects**: MongoDB (changed from AGPL), RocketChat, Grafana

**Organizational Implications**:
- ⚠️ Avoid for SaaS (requires open-sourcing modifications)
- ✅ Internal use OK (if not network-exposed to users)
- ⚠️ Carefully evaluate "interacting over network" qualifier

## Common Weak Copyleft Licenses

### GNU Lesser General Public License (LGPL)
```yaml
Library Use:         Link to LGPL libraries from proprietary software OK
Library Modifications: Modifications to LGPL library must be open-sourced
Dynamic Linking:     Explicitly allows with proprietary code
Derivative Works:    Complete apps not derivatives just from using LGPL libraries
```

**Why Projects Choose LGPL**:
- Library adoption (encourages proprietary use)
- Balances openness + commercial viability
- Suitable for standard component libraries

**Popular Projects**: Qt (dual-licensed), GTK, GStreamer, many C libraries

**Organizational Implications**:
- ✅ Use in proprietary applications
- ⚠️ Must provide source for library modifications
- ⚠️ Dynamic linking safer than static

### Mozilla Public License (MPL) 2.0
```yaml
File-Level Copyleft:    Requirements apply only to MPL files
Larger Work Exemption:  Combine MPL + proprietary files in same app
Source Code Disclosure: Must provide source for MPL files
Patent Grant:          Explicit grant + defensive termination
GPL Compatibility:     Compatible with GPL
```

**Why Projects Choose MPL 2.0**:
- Balance (stronger than permissive, flexible than GPL)
- Commercial use enabled
- File-level copyleft easier to track

**Popular Projects**: Firefox, Thunderbird, LibreOffice

**Organizational Implications**:
- ✅ Mix with proprietary code (easier than GPL)
- ⚠️ File-level tracking (clear boundaries required)
- ⚠️ Changes to MPL files shared; separate files OK

## License Compatibility

### Compatible Combinations
```
✅ MIT + Apache 2.0
✅ MIT + GPL v3
✅ Apache 2.0 + GPL v3
✅ LGPL + GPL (can upgrade LGPL to GPL)
```

### Incompatible Combinations
```
❌ GPL v2 + Apache 2.0
❌ GPL + Proprietary
❌ Different copyleft licenses (generally)
```

### Compatibility Considerations
- License inventory for all dependencies
- Verify component licenses compatible
- Complex cases need legal review

## Dual Licensing Strategies

| Strategy | Description | Example |
|----------|-------------|---------|
| **OSS + Commercial** | GPL or commercial license choice | Qt, MySQL, MongoDB |
| **Multiple OSS Licenses** | Choose from compatible licenses | Apache 2.0 or MIT options |

## Quick Reference Matrix

| License | Commercial Use | Derivative Work Requirement | Patent Grant | Complexity |
|---------|----------------|----------------------------|--------------|------------|
| **MIT** | ✅ Yes | None | ❌ No | Low |
| **Apache 2.0** | ✅ Yes | None | ✅ Yes | Medium |
| **BSD** | ✅ Yes | None | ❌ No | Low |
| **LGPL** | ✅ Library linking | Library mods only | Varies | Medium |
| **MPL 2.0** | ✅ Yes | MPL files only | ✅ Yes | Medium |
| **GPL v2/v3** | ⚠️ Must open-source | Must use GPL | v3: ✅ | High |
| **AGPL** | ⚠️ Must open-source | Must use AGPL + network | ✅ Yes | Very High |

## Critical Notes
- ⚠️ Most OSS uses relatively small number of popular licenses
- 💡 GPL v3 + Apache 2.0 compatible; GPL v2 + Apache 2.0 incompatible
- 🎯 AGPL closes "SaaS loophole" - avoid for service offerings
- 📊 License compatibility requires careful inventory + legal review for complex cases

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-open-source-software-azure/6-explore-common-licenses)
