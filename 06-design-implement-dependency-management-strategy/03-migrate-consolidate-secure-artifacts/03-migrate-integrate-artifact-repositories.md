# Migrate and Integrate Artifact Repositories

## Benefits of Migrating to Azure Artifacts

### Eliminate On-Premises Dependencies
Azure Artifacts provides hosted package feeds as a service, eliminating dependencies on:
- **File shares**: Network drives requiring VPN access and maintenance
- **Locally hosted instances**: NuGet.Server, npm registries, or Maven repositories on local servers
- **Infrastructure overhead**: Physical or virtual servers requiring updates, backups, and monitoring

### Universal CI/CD Integration
Feeds can be consumed by any CI system that supports authenticated feeds:
- Azure Pipelines: Native integration with seamless authentication
- Jenkins: Configure via credentials and feed URLs
- TeamCity: Add as external package sources
- GitHub Actions: Authenticate with Personal Access Tokens
- GitLab CI: Configure package sources in CI/CD pipelines

## Migration Strategies

### 1. Direct Migration
**Approach**: Migrate all packages at once to Azure Artifacts.

**When to Use:**
- Small to medium-sized repositories
- Limited number of consumers
- Can coordinate downtime with teams

**Steps:**
1. Create Azure Artifacts feed with appropriate visibility and settings
2. Export packages from existing repository
3. Publish to Azure Artifacts using package manager tools or scripts
4. Update consumers to reference new feed
5. Decommission old repository after validation period

**Advantages:**
- ✅ Clean break: No mixed sources
- ✅ Simplified management: Single source of truth immediately

**Challenges:**
- ⚠️ Coordination required: All teams must update at once
- ⚠️ Potential disruption: Risk if issues arise during migration

### 2. Gradual Migration with Upstream Sources
**Approach**: Use Azure Artifacts as primary feed with existing repository as upstream source.

**When to Use:**
- Large repositories with many consumers
- Need to minimize disruption
- Want to test Azure Artifacts gradually

**Steps:**
1. Create Azure Artifacts feed configured with upstream sources pointing to existing repository
2. Update consumers to reference Azure Artifacts feed
3. Migrate new packages directly to Azure Artifacts
4. Existing packages are cached from upstream when requested
5. Optionally migrate existing packages over time
6. Remove upstream source once migration is complete

**Advantages:**
- ✅ Zero downtime: Consumers transparently access packages
- ✅ Gradual transition: Move at your own pace
- ✅ Fallback option: Keep existing repository as backup

**Benefits of Upstream Sources:**
- Caching: Azure Artifacts caches packages from upstream, improving performance
- Unified access: Consumers configure only Azure Artifacts
- Security: Control external access through Azure Artifacts

### 3. Parallel Operation
**Approach**: Run Azure Artifacts alongside existing repository.

**When to Use:**
- Risk-averse organizations
- Need extended validation period
- Complex migration scenarios

**Steps:**
1. Create Azure Artifacts feed parallel to existing repository
2. Publish to both feeds initially
3. Gradually migrate consumers to Azure Artifacts
4. Monitor usage of old repository
5. Decommission when usage drops to zero

**Advantages:**
- ✅ Low risk: Can revert if issues arise
- ✅ Extended validation: Time to ensure Azure Artifacts meets needs

**Challenges:**
- ⚠️ Dual maintenance: Must publish to both locations
- ⚠️ Potential inconsistency: Versions may drift between sources

## Migration by Package Type

### NuGet Packages
**Migration Tools:**
- NuGet CLI: Use `nuget push` to publish packages
- Azure Artifacts CLI: `az artifacts universal publish` for bulk operations
- PowerShell scripts: Automate migration with scripts

**Configuration:**
```xml
<packageSources>
  <add key="AzureArtifacts" value="https://pkgs.dev.azure.com/{organization}/_packaging/{feed}/nuget/v3/index.json" />
</packageSources>
```

### npm Packages
**Migration Tools:**
- npm CLI: Use `npm publish` with configured registry
- Scripts: Automate with Node.js scripts

**Configuration (`.npmrc`):**
```
registry=https://pkgs.dev.azure.com/{organization}/_packaging/{feed}/npm/registry/
```

### Maven Packages
**Migration Tools:**
- Maven CLI: Use `mvn deploy` to publish
- Gradle: Configure publication with Gradle build scripts

**Configuration (`pom.xml` or `settings.xml`):**
```xml
<distributionManagement>
  <repository>
    <id>azure-artifacts</id>
    <url>https://pkgs.dev.azure.com/{organization}/_packaging/{feed}/maven/v1</url>
  </repository>
</distributionManagement>
```

### Python Packages
**Migration Tools:**
- twine: Use `twine upload` to publish packages
- pip: Configure index URL for consumption

**Configuration:**
```bash
pip install --index-url https://pkgs.dev.azure.com/{organization}/_packaging/{feed}/pypi/simple/ {package}
```

### Universal Packages
**Use For:**
- Build artifacts: Compiled applications
- Binary files: Any files previously in source control or file shares
- Mixed content: Folders with multiple file types

**Publish:**
```bash
az artifacts universal publish --organization https://dev.azure.com/{organization} --feed {feed} --name {package-name} --version {version} --path {path-to-folder}
```

## Migration Best Practices

### Planning
- **Inventory packages**: Know what you're migrating
- **Test first**: Migrate test repositories before production
- **Communicate**: Inform teams of migration plans and timelines

### Execution
- **Verify packages**: Ensure packages work after migration
- **Monitor consumption**: Track which packages are accessed from new feed
- **Maintain old repository**: Keep as read-only backup during transition

### Post-Migration
- **Document changes**: Update documentation with new feed URLs
- **Decommission gradually**: Only remove old repositories after validation period
- **Train teams**: Ensure everyone knows how to use Azure Artifacts

## Migration Strategy Comparison

| Strategy | Downtime | Complexity | Risk | Best For |
|----------|----------|------------|------|----------|
| **Direct** | Yes | Low | Medium | Small repositories |
| **Gradual with Upstream** | No | Medium | Low | Large repositories |
| **Parallel** | No | High | Very Low | Risk-averse orgs |

## Critical Notes
- 🎯 Choose migration strategy based on repository size and risk tolerance
- 💡 Upstream sources enable zero-downtime gradual migration
- ⚠️ Test migration with non-production feeds first
- 📊 Monitor consumption patterns during migration
- 🔄 Keep old repository as read-only backup during transition

[Learn More](https://learn.microsoft.com/en-us/training/modules/migrate-consolidating-secure-artifacts/3-migrate-integrating-artifact-repositories)
