# Explore Best Practices for Versioning

## Key Concepts
There are several best practices to effectively use versions, views, and promotion flow for your versioning strategy.

## Core Versioning Best Practices

### 1. Have a Documented Versioning Strategy

Have a documented versioning strategy.

**Why this matters**:
- **Consistency**: Everyone follows the same versioning rules
- **Predictability**: Consumers know what to expect from version changes
- **Onboarding**: New team members understand versioning approach
- **Quality**: Reduces confusion and versioning mistakes

**What to document**:
- **Versioning scheme**: Which scheme you use (SemVer, date-based, etc.)
- **Increment rules**: When to increment major, minor, patch
- **Prerelease labels**: What labels mean (@alpha, @beta, @rc)
- **Promotion criteria**: What qualifies a package for each view
- **Special cases**: How to handle hotfixes, backports, deprecated features

**Example documentation**:
```markdown
# Versioning Strategy

## Version Format
We use Semantic Versioning 2.0 (Major.Minor.Patch-label)

## Increment Rules
- Major: Breaking changes, removed APIs
- Minor: New features, backward compatible
- Patch: Bug fixes only

## Prerelease Labels
- alpha: Early development, unstable
- beta: Feature complete, testing in progress
- rc: Release candidate, final testing

## Promotion Workflow
1. All packages start in @Local
2. Promote to @Prerelease after unit tests pass
3. Promote to @Release after QA approval and security scan
```

### 2. Adopt SemVer 2.0 for Your Versioning Scheme

Adopt SemVer 2.0 for your versioning scheme.

**Benefits of Semantic Versioning**:
- **Industry standard**: Widely understood and adopted
- **Clear communication**: Version numbers communicate intent
- **Tool support**: Package managers understand SemVer
- **Dependency management**: Enables automated dependency resolution

**SemVer format**:
```
Major.Minor.Patch-prerelease+buildmetadata

Examples:
1.0.0
1.2.3
2.0.0-beta.1
1.0.0+build.123
```

**SemVer rules**:
- **MAJOR**: Increment when you make incompatible API changes
- **MINOR**: Increment when you add functionality in a backward-compatible manner
- **PATCH**: Increment when you make backward-compatible bug fixes
- **Prerelease**: Append hyphen and label for prereleases
- **Build metadata**: Append plus and metadata for build information

### 3. Each Repository Should Only Reference One Feed

Each repository should only reference one feed.

**Why single feed per repository**:
- **Simplicity**: Easier to manage and configure
- **Consistency**: All dependencies come from same source
- **Security**: Single point for security scanning and approval
- **Performance**: Faster package resolution with single feed

**Implementation**:
```xml
<!-- NuGet.config - Single feed -->
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="MyFeed" value="https://pkgs.dev.azure.com/MyOrg/_packaging/MyFeed/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

**Exception - Upstream sources**: Instead of referencing multiple feeds, use upstream sources to aggregate multiple package sources into one feed.

```
Your Feed → Upstream to npmjs.com
         → Upstream to NuGet.org

Your repository → References only "Your Feed"
```

### 4. On Package Creation, Automatically Publish Packages Back to the Feed

On package creation, automatically publish packages back to the feed.

**Benefits of automated publishing**:
- **Consistency**: Every build creates a package
- **Traceability**: Each commit has an associated package
- **CI/CD**: Enables continuous delivery workflows
- **Early testing**: Packages available immediately for testing

**Implementation in Azure Pipelines**:
```yaml
trigger:
  - main

pool:
  vmImage: "ubuntu-latest"

steps:
  # Build the package
  - script: dotnet pack --configuration Release
    displayName: "Build package"

  # Automatically publish to feed
  - task: NuGetCommand@2
    displayName: "Publish package to feed"
    inputs:
      command: "push"
      packagesToPush: "$(Build.ArtifactStagingDirectory)/**/*.nupkg"
      nuGetFeedType: "internal"
      publishVstsFeed: "MyFeed"
      allowPackageConflicts: false
```

## Additional Best Practices

### Use Views for Quality Gates

Leverage views to indicate package quality:
- **@Local**: Development and testing
- **@Prerelease**: QA and beta testing
- **@Release**: Production-ready packages

**Promotion workflow**:
```yaml
# Automated promotion based on tests
- stage: Promote
  condition: succeeded()
  jobs:
    - job: PromotePackage
      steps:
        - task: AzureCLI@2
          inputs:
            scriptType: "bash"
            inlineScript: |
              az artifacts universal promote \
                --feed MyFeed \
                --name MyPackage \
                --version $(Build.BuildNumber) \
                --view Release
```

### Never Republish the Same Version

Package immutability is critical:
- **No overwrites**: Never publish same version twice with different content
- **Version increment**: Always increment version for any change
- **Trust**: Consumers trust versions don't change

**If you need to fix a broken package**:
```
Wrong: Republish 1.2.3 with fixes
Right: Publish 1.2.4 with fixes
```

### Use Retention Policies Wisely

Balance storage costs with retention needs:
- **Promote important packages**: Protected from deletion
- **Clean up @Local**: Let retention policies clean up unpromoted packages
- **Keep release history**: Always keep promoted versions

**Example retention policy**:
```
@Local view: Keep last 30 days
@Prerelease view: Keep last 90 days
@Release view: Keep indefinitely
```

### Version Source Code and Packages Together

Tag source code with package versions:

```bash
# Tag commit with version number
git tag v1.2.3
git push origin v1.2.3
```

**Benefits**:
- **Traceability**: Link package to source code
- **Debugging**: Find exact code that produced a package
- **Reproducibility**: Rebuild package from tagged source

### Include Version in Package Metadata

Comprehensive package metadata:

```json
{
  "name": "my-package",
  "version": "1.2.3",
  "description": "Package description",
  "author": "Team Name",
  "repository": {
    "type": "git",
    "url": "https://github.com/org/repo.git"
  },
  "releaseNotes": "Bug fixes and improvements"
}
```

### Test Before Promoting

Always test before promotion:
- **Unit tests**: Before promoting to @Prerelease
- **Integration tests**: Before promoting to @Release
- **Security scans**: Check for vulnerabilities
- **Performance tests**: Ensure no performance degradation

## Team Adoption Practices

### Integration with Definition of Done

It can be made part of the Definition of Done for work items related to publishing and consuming packages from feeds.

**Example Definition of Done items**:
- Package version follows SemVer 2.0
- Package published to feed automatically
- Source code tagged with version number
- Unit tests pass before promoting to @Prerelease
- Integration tests pass before promoting to @Release
- Release notes included in package metadata
- Security scan completed with no high/critical issues
- Package promoted to appropriate view based on quality

### Training and Onboarding

Share best practices with team:
- **Documentation**: Maintain versioning guidelines document
- **Training sessions**: Regular training on versioning practices
- **Code reviews**: Review version increments during code reviews
- **Templates**: Provide pipeline templates with best practices built in

### Monitoring and Enforcement

Track compliance with best practices:
- **Automated checks**: Build pipeline checks for versioning rules
- **Reports**: Regular reports on versioning patterns
- **Metrics**: Track version increment patterns, promotion velocity
- **Audits**: Periodic audits of package versions and promotion history

## Versioning Anti-Patterns to Avoid

Don't do these:

### Manual Version Management
- **Problem**: Inconsistent, error-prone
- **Solution**: Automate versioning in CI/CD pipeline

### Skipping Versions
- **Problem**: Creates gaps in version history
- **Solution**: Increment sequentially for every change

### Using Dates as Versions
- **Problem**: Doesn't communicate change scope
- **Solution**: Use SemVer; dates can be in build metadata

### Publishing Without Testing
- **Problem**: Broken packages reach consumers
- **Solution**: Always test before publishing

### No Version Strategy Documentation
- **Problem**: Team confusion, inconsistent versions
- **Solution**: Document and share strategy

## Version Strategy Checklist

Before implementing versioning strategy:

✅ Documented versioning scheme (SemVer 2.0 recommended)
✅ Automated version generation in build pipeline
✅ Single feed per repository configuration
✅ Automated package publishing to feed
✅ View promotion workflow defined
✅ Retention policies configured
✅ Test gates before promotion
✅ Source code tagging with versions
✅ Team training completed
✅ Definition of Done updated

## Quick Reference

### Best Practices Summary Table
| Practice | Why | How |
|----------|-----|-----|
| **Document strategy** | Consistency, onboarding | Create versioning guidelines doc |
| **Adopt SemVer 2.0** | Industry standard, clear communication | Major.Minor.Patch-label format |
| **Single feed per repo** | Simplicity, security | Use upstream sources for aggregation |
| **Automate publishing** | Consistency, traceability | CI/CD pipeline task |
| **Use views** | Quality gates, risk mitigation | @Local → @Prerelease → @Release |
| **Never republish** | Trust, immutability | Always increment version |
| **Retention policies** | Storage optimization | Promote important, clean up old |
| **Tag source code** | Traceability, reproducibility | Git tags matching versions |
| **Test before promote** | Quality assurance | Automated test gates |

### Anti-Patterns to Avoid
| Anti-Pattern | Impact | Solution |
|--------------|--------|----------|
| Manual versioning | Errors, inconsistency | Automate in pipeline |
| Skipping versions | Confusion, gaps | Sequential increments |
| Date-based versions | No change scope info | Use SemVer with dates in metadata |
| No testing | Broken packages | Automated test gates |
| No documentation | Team confusion | Written guidelines |

## Critical Notes
📝 **Document your strategy** - Written guidelines ensure consistency and help onboarding

🎯 **SemVer 2.0 is the standard** - Industry-wide adoption makes it the best choice

📦 **One feed per repository** - Use upstream sources instead of multiple feed references

🤖 **Automate everything** - From version generation to publishing to promotion

🏷️ **Views are quality gates** - Use @Local, @Prerelease, @Release for staged releases

🔒 **Immutability is critical** - Never republish the same version with different content

🧪 **Test before promoting** - Always run appropriate tests before moving to higher views

🏗️ **Build it into Definition of Done** - Make versioning practices part of team workflow

⚠️ **Avoid anti-patterns** - Manual versioning, skipping versions, and no testing are common pitfalls

## Learn More
- [Best practices for using Azure Artifacts](https://learn.microsoft.com/en-us/azure/devops/artifacts/concepts/best-practices)
