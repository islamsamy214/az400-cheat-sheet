# Consume Packages

## Developer Workflow for Consuming Packages

### Standard Pattern
1. **Identify a required dependency**: Determine what functionality is needed
2. **Find a component**: Search for component that satisfies requirements
3. **Search package sources**: Look for package offering correct component version
4. **Install the package**: Use package manager to install package into codebase
5. **Implement functionality**: Create software implementation using new components

## Configuring Package Sources

### Default Sources by Package Type
- **NuGet**: NuGet.org
- **npm**: npmjs.com
- **Maven**: Maven Central
- **pip**: PyPI.org
- **Docker**: Docker Hub

### Configuration Examples

**NuGet** (`nuget.config`):
```xml
<packageSources>
  <add key="NuGet.org" value="https://api.nuget.org/v3/index.json" />
  <add key="MyPrivateFeed" value="https://pkgs.dev.azure.com/myorg/_packaging/myfeed/nuget/v3/index.json" />
</packageSources>
```

**npm** (`.npmrc`):
```
registry=https://registry.npmjs.org/
@myorg:registry=https://pkgs.dev.azure.com/myorg/_packaging/myfeed/npm/registry/
```

**Maven** (`pom.xml` or `settings.xml`):
```xml
<repositories>
  <repository>
    <id>central</id>
    <url>https://repo.maven.apache.org/maven2</url>
  </repository>
  <repository>
    <id>myprivatefeed</id>
    <url>https://pkgs.dev.azure.com/myorg/_packaging/myfeed/maven/v1</url>
  </repository>
</repositories>
```

**pip** (`pip.conf` or command line):
```bash
pip install --index-url https://pypi.org/simple/ \
  --extra-index-url https://pkgs.dev.azure.com/myorg/_packaging/myfeed/pypi/simple/ \
  mypackage
```

## Upstream Sources

### How Upstream Sources Work
Package resolution order:
1. Package manager evaluates the primary source first
2. If package isn't found, it switches to upstream source
3. Upstream source might be official public or private sources
4. Upstream source could refer to another upstream source (chain of sources)

### Typical Scenario
Use a private package source referring to a public upstream source.

**Benefits:**
- ✅ **Enhances packages**: Effectively enhances packages in upstream source with private feed packages
- ✅ **Avoids public publishing**: Keeps private packages from being published publicly
- ✅ **Single configuration**: Developers configure only the private feed, which includes upstream packages

### Caching Behavior
A source with upstream source defined may download and cache requested packages.

**How Caching Works:**
- Source includes downloaded packages
- Acts as a cache for the upstream source
- Tracks packages from external upstream source
- Improves performance by reducing repeated downloads

### Proxy for External Sources
Upstream source can avoid direct access of developers and build machines to external sources.

**Proxy Pattern:**
- Private feed uses upstream source as proxy for external source
- Feed manager and private source communicate to outside
- Developers and build machines only access private feed
- Only privileged roles can add upstream sources

**Benefits:**
- ✅ **Security**: Control external package access through single point
- ✅ **Governance**: Monitor and control which external packages are used
- ✅ **Reliability**: Continue using cached packages even if external source is unavailable
- ✅ **Performance**: Faster package downloads from local cache

## Package Graphs

### Benefits of Package Graphs
- **Team-specific feeds**: Different teams maintain their own feeds with shared upstream sources
- **Environment separation**: Development, staging, and production feeds with different configurations
- **Framework layers**: Base framework packages → Company libraries → Application packages

### Considerations
Package graphs can become complex when not correctly understood or designed.

**Best Practices:**
- Keep it simple: Avoid unnecessary complexity in upstream source chains
- Document structure: Clearly document the package graph for your organization
- Limit depth: Avoid deep chains of upstream sources
- Monitor usage: Track which packages come from which sources

## Authentication

### Authentication Methods for Private Feeds
- **Personal Access Tokens (PAT)**: Token-based authentication for scripts and tools
- **Credential providers**: Automatic credential management for package managers
- **Service principals**: For CI/CD pipelines and automation
- **Integrated authentication**: Use existing organizational credentials

### Example with Azure Artifacts
- Install Azure Artifacts Credential Provider
- Package managers automatically prompt for credentials when needed
- Credentials are cached for future use

## Quick Reference

| Concept | Purpose | Benefit |
|---------|---------|---------|
| **Default sources** | Pre-configured public feeds | No configuration needed for public packages |
| **Upstream sources** | Chain package sources | Single configuration point, caching |
| **Package graphs** | Multiple source hierarchy | Team autonomy, environment separation |
| **Authentication** | Secure private feed access | Controlled access to proprietary packages |

## Critical Notes
- 🎯 Package managers search primary source first, then upstream sources
- 💡 Upstream sources enable caching and proxy patterns
- ⚠️ Keep package graphs simple to avoid complexity
- 📊 Authentication required for private feeds
- 🔒 Proxy pattern provides security and governance benefits

[Learn More](https://learn.microsoft.com/en-us/training/modules/understand-package-management/7-consume-packages)
