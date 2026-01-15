# Examine Release Views

## Key Concepts
When building packages from a pipeline, the package needs to have a version before being consumed and tested. Only after testing is the quality of the package known. Since package versions can't and shouldn't be changed, it becomes challenging to choose a specific version beforehand. Azure Artifacts solves this with views.

## Quality Levels and Views

### The Challenge
- **Version first**: Package must have version before testing
- **Quality unknown**: Quality is determined after testing
- **Immutability**: Can't change version after publishing
- **Solution**: Use views to indicate quality without changing version

### Views as Quality Descriptors
It fits well with the use of semantic versioning of the packages for predictability of the intent of a particular version. Still, its extra metadata from the Azure Artifacts feed is called a descriptor.

**Views provide**:
- **Quality indicator**: Separate packages by maturity level
- **Consumer choice**: Allow consumers to choose quality level
- **Promotion workflow**: Move packages through quality gates
- **Immutability maintained**: Package version doesn't change

## Default Feed Views

Feeds in Azure Artifacts have three different views by default. These views are added when a new feed is created.

### The Three Default Views

#### 1. @Local View
The `@Local` view contains all release and prerelease packages and the packages downloaded from upstream sources.

**Characteristics**:
- **Complete feed**: Shows all packages regardless of promotion status
- **Upstream packages**: Includes packages cached from upstream sources
- **Default view**: Used by default when no view is specified
- **Development usage**: Typically used during development

**Use cases**:
- **Development builds**: Developers testing latest packages
- **CI builds**: Build pipelines that need all package versions
- **Package discovery**: Finding all available packages

#### 2. @Prerelease View
The `@Prerelease` view contains all packages that have a label in their version number.

**Characteristics**:
- **Labeled versions only**: Only shows versions with prerelease labels (e.g., `1.0.0-beta`)
- **Quality gate**: Packages promoted here have passed initial quality checks
- **Testing phase**: Indicates packages ready for broader testing
- **Not production-ready**: Not recommended for production use

**Use cases**:
- **QA testing**: Quality assurance teams testing prerelease versions
- **Beta testing**: Early adopters testing new features
- **Integration testing**: Testing integration with other systems

#### 3. @Release View
The `@Release` view contains all packages that are considered official releases.

**Characteristics**:
- **Production-ready**: Only contains packages approved for production
- **No labels**: Typically contains only versions without prerelease labels
- **Stable versions**: Highest quality level
- **Supported**: Packages in this view are officially supported

**Use cases**:
- **Production deployments**: Applications deployed to production
- **Stable builds**: Release builds for customers
- **Official releases**: Packages distributed to end users

## Using Views

You can use views to offer help consumers of a package feed filter between released and unreleased versions of packages. Essentially, it allows a consumer to make a conscious decision to choose from released packages or opt-in to prereleases of a certain quality level.

### Default View URI Format
By default, the `@Local` view is used to offer the list of available packages:

```
https://pkgs.dev.azure.com/{organization}/_packaging/{feedname}/nuget/v3/index.json
```

**Components**:
- **{organization}**: Your Azure DevOps organization name
- **{feedname}**: The name of your feed
- **No view specified**: Defaults to @Local view

**Example**:
```
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed/nuget/v3/index.json
```

### View-Specific URI Format
When consuming a package feed by its URI endpoint, the address can have the requested view included:

```
https://pkgs.dev.azure.com/{organization}/_packaging/{feedname}@{Viewname}/nuget/v3/index.json
```

**View-specific examples**:
```
# @Local view (explicit)
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed@Local/nuget/v3/index.json

# @Prerelease view
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed@Prerelease/nuget/v3/index.json

# @Release view
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed@Release/nuget/v3/index.json
```

### Automatic View Filtering
The tooling will show and use the packages from the specified view automatically.

**Package manager behavior**:
- **View filtering**: Only packages in the specified view are visible
- **Automatic resolution**: Dependency resolution uses only visible packages
- **Transparent**: No additional configuration needed

## Views Across Package Types

Views work consistently across all package types:

### NuGet
```
# Default (Local)
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed/nuget/v3/index.json

# Release view
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed@Release/nuget/v3/index.json
```

### npm
```
# Default (Local)
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed/npm/registry/

# Release view
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed@Release/npm/registry/
```

### Maven
```
# Default (Local)
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed/maven/v1

# Release view
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed@Release/maven/v1
```

### Python
```
# Default (Local)
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed/pypi/simple/

# Release view
https://pkgs.dev.azure.com/contoso/_packaging/MyFeed@Release/pypi/simple/
```

## Prerelease Label versus View

### Two Separate Concepts

#### Prerelease Labels (SemVer)
- **Version-based**: Part of the version number (e.g., `1.0.0-beta`)
- **Semantic versioning**: Follows SemVer conventions
- **Package manager feature**: Package managers can filter based on labels
- **Universal**: Works across all package systems

**Example**:
```
1.0.0-alpha.1  ← Prerelease label
1.0.0          ← Stable version
```

#### @Prerelease View (Azure Artifacts)
- **Feed metadata**: Azure Artifacts-specific quality indicator
- **Promotion-based**: Packages promoted to this view
- **Quality gate**: Indicates a certain quality level
- **Azure-specific**: Only applies to Azure Artifacts feeds

### How They Work Together
- **Complementary**: Both help manage package quality
- **Independent**: A package can have a label but not be in @Prerelease view
- **Workflow**: Typically, labeled versions are promoted to @Prerelease view

> **Important distinction**: Tooling may offer an option to select prerelease versions (like Visual Studio NuGet dialog). It doesn't relate or refer to the `@Prerelease` view of a feed. Instead, it relies on the presence of prerelease labels of semantic versioning to include or exclude packages in the search results.

## Configuring Package Consumers

### Development Configuration
Use @Local view for active development:

```bash
# NuGet - Add source with Local view
dotnet nuget add source "https://pkgs.dev.azure.com/contoso/_packaging/MyFeed/nuget/v3/index.json" --name MyFeed-Local

# npm - Configure in .npmrc
registry=https://pkgs.dev.azure.com/contoso/_packaging/MyFeed/npm/registry/
```

### QA/Testing Configuration
Use @Prerelease view for testing:

```bash
# NuGet - Add source with Prerelease view
dotnet nuget add source "https://pkgs.dev.azure.com/contoso/_packaging/MyFeed@Prerelease/nuget/v3/index.json" --name MyFeed-Prerelease
```

### Production Configuration
Use @Release view for production:

```bash
# NuGet - Add source with Release view
dotnet nuget add source "https://pkgs.dev.azure.com/contoso/_packaging/MyFeed@Release/nuget/v3/index.json" --name MyFeed-Release

# npm - Configure in .npmrc for production
registry=https://pkgs.dev.azure.com/contoso/_packaging/MyFeed@Release/npm/registry/
```

## Custom Views

Beyond the three default views, you can create custom views for finer-grained quality levels.

**Custom view examples**:
- **@Alpha**: Very early versions
- **@Beta**: Feature-complete testing versions
- **@RC**: Release candidate versions
- **@Stable**: Long-term stable versions

**Creating custom views**:
1. **Navigate to feed settings**: Go to your feed in Azure Artifacts
2. **Views tab**: Select the Views section
3. **Add view**: Select "Add view" and provide a name
4. **Configure**: Set visibility and promotion rules

## View Benefits

### Quality Control
- **Staged releases**: Move packages through quality gates
- **Risk mitigation**: Prevent untested packages in production
- **Clear communication**: Views indicate quality level

### Consumer Flexibility
- **Choice**: Consumers choose their risk tolerance
- **Opt-in**: Consciously opt into prerelease versions
- **Safety**: Production systems only see released packages

### Workflow Integration
- **CI/CD**: Integrate promotion into pipelines
- **Automation**: Automate promotion based on test results
- **Governance**: Enforce quality gates through views

## Quick Reference

### View Comparison Table
| View | Contains | Upstream | Use Case | Production Ready |
|------|----------|----------|----------|------------------|
| **@Local** | All packages | ✅ Yes | Development, CI builds | ❌ No |
| **@Prerelease** | Labeled versions | ❌ No | QA testing, beta testing | ❌ No |
| **@Release** | Stable packages | ❌ No | Production deployments | ✅ Yes |

### Common Promotion Workflows
```
Simple:  @Local → @Prerelease → @Release
Complex: @Local → @Alpha → @Beta → @RC → @Release
```

## Critical Notes
🎯 **Views solve version immutability** - Version can't change, but quality indicator (view) can

📦 **@Local is default** - When no view specified, @Local view is used

🏷️ **Prerelease label ≠ @Prerelease view** - These are two separate concepts that work together

🔒 **Production should use @Release** - Only production-ready packages should be in @Release view

🔄 **Upstream only with @Local** - Other views don't trigger upstream source evaluation

✨ **Custom views available** - Create additional views for finer-grained quality levels

## Learn More
- [Views on Azure DevOps Services feeds](https://learn.microsoft.com/en-us/azure/devops/artifacts/concepts/views)
- [Communicate package quality with prerelease and release views](https://learn.microsoft.com/en-us/azure/devops/artifacts/feeds/views)
