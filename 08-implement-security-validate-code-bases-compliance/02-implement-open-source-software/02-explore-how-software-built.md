# Explore How Software is Built

## Key Concepts
- **Component-Based Model**: 80% existing components, 20% original code
- **Building → Assembling**: Fundamental shift from writing everything to combining components
- **Package Ecosystems**: npm, PyPI, NuGet, Maven Central, RubyGems, Crates.io

## Component-Based Software Model

Modern applications combine:
- **Original Business Logic**: Custom code for specific requirements, workflows, unique features
- **Open-Source Libraries/Frameworks**: Community-maintained reusable components (data processing, auth, UI, protocols)
- **Commercial Components**: Vendor libraries with specialized functionality, support, guarantees
- **Integration Code**: "Glue" code connecting components, adapting interfaces, orchestrating interactions

**Research Finding**: ~80% existing components (maintained outside project), ~20% original code

## Why Software is Built This Way

### Development Velocity
- **Proven Solutions**: Battle-tested components work reliably
- **Reduced Time**: Building framework/drivers from scratch = months/years; using existing = days/hours
- **Focus on Business Value**: Concentrate on unique logic, not reinventing infrastructure
- **Faster Time to Market**: Applications reach production sooner

### Quality and Reliability
- **Community Vetting**: Thousands of users identifying/reporting issues → robust code
- **Expert Development**: Specialists maintaining projects in specific domains
- **Continuous Improvement**: Regular updates, bug fixes, enhancements from worldwide contributors
- **Production Testing**: Components tested in diverse environments/scenarios

### Cost Efficiency
- **No Licensing Fees**: Most OSS free to use
- **Shared Maintenance**: Community contributes fixes/improvements
- **Reduced Staffing**: Don't need specialists for every layer
- **Lower TCO**: Similar functionality without licensing fees

### Access to Innovation
- **Cutting-Edge Features**: New technologies emerge first in OSS
- **Ecosystem Effects**: Popular frameworks create compatible components, tools, knowledge
- **Flexible Adoption**: Experiment without large financial commitments
- **Community Knowledge**: Extensive docs, tutorials, community support

## Open-Source vs Closed-Source Components

| Aspect | Open-Source | Closed-Source |
|--------|-------------|---------------|
| **Source Code** | Publicly available, inspectable | Binary distribution only |
| **Transparency** | Can audit for vulnerabilities/quality | Limited transparency |
| **Community** | Many contributors | Vendor control |
| **Support** | Community support | Commercial support, SLAs |
| **Licensing** | Ranges from unrestricted to copyleft | Proprietary terms |

**Popular Open-Source**:
- Languages/Runtimes: Python, Node.js, .NET Core, Go, Rust
- Web Frameworks: React, Angular, Vue.js, Express, Django, Spring Boot
- Databases: PostgreSQL, MySQL, MongoDB, Redis, Elasticsearch
- Dev Tools: VS Code, Git, Docker, Kubernetes
- Libraries: Lodash, Moment.js, NumPy, Pandas, TensorFlow

## How Components are Distributed

### Package Structure
```yaml
Binary Code:         Compiled libraries ready to use
Metadata:           Name, version, author, description
Dependencies:       Other required packages
License Information: Legal usage terms
Documentation:      Usage instructions, API refs, examples
```

### Package Ecosystems
- **npm**: JavaScript/TypeScript (2M+ packages, world's largest)
- **PyPI**: Python (data science, web dev, automation)
- **NuGet**: .NET (C#, F#, Visual Basic)
- **Maven Central**: Java (enterprise, Android)
- **RubyGems**: Ruby (web apps, automation)
- **Crates.io**: Rust (systems programming)

### Package Management Tools
- **Dependency Resolution**: Auto-determine/install required dependencies
- **Version Management**: Track which versions app uses
- **Update Notifications**: Alert when newer versions available
- **Vulnerability Scanning**: Some integrate security scanning

## Implications of Component-Based Development

### Dependency Management Complexity
- **Dependency Trees**: 20 direct deps → hundreds/thousands transitive deps
- **Version Conflicts**: Incompatible versions of shared dependencies
- **Update Cascades**: Updating one component requires updating many

### Security Considerations
- **Inherited Vulnerabilities**: Any dependency vulnerability affects your app
- **Supply Chain Attacks**: Malicious actors compromise popular packages
- **Unmaintained Dependencies**: No security updates for abandoned components

### License Compliance
- **License Obligations**: Each license has requirements (unrestricted to source-sharing)
- **License Proliferation**: Hundreds of packages with dozens of different licenses
- **Compliance Burden**: Must track obligations and ensure compliance

### Operational Dependencies
- **External Hosting**: Packages on public registries (could have outages)
- **Registry Availability**: Unavailable registry = failed builds/deployments
- **Package Removal**: Authors can remove packages, breaking dependent apps

## Critical Notes
- ⚠️ Modern apps are 80% existing components, 20% original code
- 💡 Component-based approach accelerates development but introduces complexity
- 🎯 Security vulnerabilities in any dependency affect entire application
- 📊 Dependency trees can contain hundreds/thousands of packages

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-open-source-software-azure/2-explore-how-software-built)
