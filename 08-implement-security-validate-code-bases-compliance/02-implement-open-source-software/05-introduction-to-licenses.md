# Introduction to Open-Source Licenses

## Key Concepts
- **Open-Source License**: Legal agreement defining how OSS can be used, modified, distributed
- **Permissive Licenses**: Minimal requirements, allow proprietary use (MIT, Apache, BSD)
- **Copyleft Licenses**: Derivative works must use same license (GPL, LGPL, MPL)
- **Without License**: Copyright law prohibits use, modification, distribution

## What Open-Source Licenses Define

### Permissions Granted
```yaml
Use Rights:         Use for any purpose (including commercial)
Modification Rights: Modify source for needs, fix bugs, add features
Distribution Rights: Share software (original or modified)
Sublicensing Rights: Sometimes license to others under different terms
```

### Obligations Imposed
- **Attribution Requirements**: Preserve copyright notices, license text
- **Source Code Disclosure**: Some require providing source with binaries
- **License Preservation**: Include license text with distributed copies
- **Derivative Work Licensing**: Some require same license (copyleft)
- **Patent Grants**: Some include explicit patent grants/defensive termination

### Liability and Warranty Disclaimers
- **No Warranty**: "As is" without merchantability/fitness guarantees
- **No Liability**: Authors not liable for damages
- **User Risk**: Users accept all risks
- Protects free OSS developers from legal liability

## The Open Source Definition (OSI)

### Core Requirements

| Requirement | Description |
|-------------|-------------|
| **Free Redistribution** | No restrictions on selling/giving away; no royalties |
| **Source Code** | Must include or provide clear instructions; no obfuscation |
| **Derived Works** | Must permit modifications under same terms |
| **Author Integrity** | May require modifications as patch files; different naming |
| **No Person Discrimination** | Cannot discriminate against any person/group |
| **No Field Discrimination** | Cannot restrict use in specific fields (business, research) |
| **License Distribution** | Rights apply automatically to all recipients |
| **Product Independence** | Rights not dependent on specific distribution |
| **No Other Software Restrictions** | Cannot impose restrictions on bundled software |
| **Technology Neutral** | No interface/execution method requirements |

### Why Requirements Matter
- **Protects User Freedom**: Prevents hidden restrictions
- **Enables Commercial Use**: Businesses can build products using OSS
- **Promotes Compatibility**: Reduces compatibility problems
- **Prevents Fragmentation**: Avoids proliferation of incompatible licenses

## Categories of Open-Source Licenses

### Permissive Licenses
```yaml
Characteristics: Minimal requirements, allow proprietary use
Requirements:    Only attribution (copyright notices, license text)
Commercial Use:  Fully compatible
Examples:        MIT License, Apache 2.0, BSD Licenses
Philosophy:      Maximize user freedom
```

### Copyleft Licenses
```yaml
Characteristics: Derivative works must use same license
Requirements:    Distribute source code, same license for derivatives
Commercial Use:  Allowed, but derivatives must be open-sourced
Examples:        GPL, LGPL, MPL
Philosophy:      Prioritize software freedom over user freedom
```

### Weak Copyleft
```yaml
Library Use:         Link to libraries in proprietary apps without open-sourcing app
Modification Restrictions: Modifications to library itself must be open-sourced
Examples:           GNU LGPL, Mozilla Public License
Balance:            Promote OSS development + enable commercial use
```

## License Selection by Projects

**Factors**:
- **Maximizing Adoption**: Permissive licenses (no significant obligations)
- **Ensuring Freedom**: Copyleft licenses (derivatives remain open-source)
- **Preventing Proprietary Forks**: Copyleft prevents proprietary versions
- **Patent Protection**: Licenses with explicit patent grants (Apache 2.0)
- **Compatibility**: Compatible with dependencies/integration targets

## Multiple Licenses

| Strategy | Description |
|----------|-------------|
| **Dual Licensing** | Offer OSS + commercial licenses; users choose |
| **License Stacking** | Different components have different licenses |
| **License Evolution** | Change licenses over time (requires all contributors' agreement) |

## The Transparency Paradox

### Security Benefits of Transparency
- **Many Eyes**: Thousands can review for vulnerabilities
- **Faster Disclosure**: Public patching informs all users
- **Community Patches**: Security-conscious devs contribute fixes
- **Audit Capability**: Organizations can audit (impossible with closed-source)

### Security Risks of Transparency
- **Vulnerability Discovery**: Attackers analyze source for exploits
- **Exploit Development**: Implementation details help attackers
- **Target Identification**: Identify apps using vulnerable versions
- **Zero-Day Exploitation**: Discover/exploit before public disclosure

### The Balance
```yaml
Linus's Law:          "Given enough eyeballs, all bugs are shallow"
Obscurity ≠ Security: Hiding source doesn't prevent vulnerabilities
Responsible Disclosure: Community practices balance security + transparency
Practical Reality:    Most breaches involve closed-source or misconfiguration
```

**Research Conclusion**: Transparency provides net security benefits

## License Comparison Summary

| License Type | Proprietary Use | Source Disclosure | Patent Grant | Copyleft |
|--------------|-----------------|-------------------|--------------|----------|
| **Permissive** | ✅ Yes | ❌ No | Varies | ❌ No |
| **Weak Copyleft** | ✅ Library linking | ⚠️ For library mods | Varies | ⚠️ Library only |
| **Copyleft** | ❌ No | ✅ Yes | Varies | ✅ Yes |

## Critical Notes
- ⚠️ Without explicit license, copyright law prohibits use/modification/distribution
- 💡 All OSS licenses disclaim liability and warranties ("as is")
- 🎯 Open Source Definition ensures meaningful freedom, prevents hidden restrictions
- 📊 Transparency generally finds/fixes vulnerabilities faster than closed-source

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-open-source-software-azure/5-introduction-to-licenses)
