# Promote Packages

## Key Concepts
Azure Artifacts has the notion of promoting packages to views to indicate that a version is of a certain quality level. By selectively promoting packages, you can plan when packages have a certain quality and are ready to be released and supported by the consumers.

## Package Promotion Fundamentals

You can promote packages to one of the available views as the quality indicator.

### Promotion Concept
- **Quality gates**: Views represent different quality levels
- **Explicit action**: Promotion is an explicit decision
- **No version change**: Package version remains the same
- **Metadata change**: Only the view association changes

### Default Views for Promotion
Release and Prerelease's two views might be sufficient, but you can create more views when you want finer-grained quality levels if necessary, such as `alpha` and `beta`.

**Common promotion workflows**:
```
@Local → @Prerelease → @Release
@Local → @Alpha → @Beta → @RC → @Release
```

## Package Visibility Rules

Packages will always show in the Local view, but only in a particular view after being promoted.

### View-Based Package Listing
Depending on the URL used to connect to the feed, the available packages will be listed.

**Visibility matrix**:
| Package State | @Local | @Prerelease | @Release |
|---------------|--------|-------------|----------|
| Just published | Yes | No | No |
| Promoted to Prerelease | Yes | Yes | No |
| Promoted to Release | Yes | Yes | Yes |

**Key principles**:
- **@Local always shows all**: Every package appears in @Local regardless of promotion
- **Other views require promotion**: Packages must be explicitly promoted to appear in other views
- **Cumulative visibility**: Promoting to @Release doesn't remove from @Prerelease

## Upstream Sources and Views

Upstream sources will only be evaluated when using the @Local view of the feed.

### Upstream Package Workflow
After they've been downloaded and cached in the @Local view, you can see and resolve the packages in other views after being promoted to it.

**Upstream package lifecycle**:
1. **Request package**: Consumer requests package from @Local view
2. **Upstream check**: Azure Artifacts checks upstream sources if not in feed
3. **Download and cache**: Package downloaded from upstream and cached in @Local
4. **Local only**: Package now available in @Local view
5. **Promotion**: Package can be promoted to @Prerelease or @Release views

**Example workflow**:
```
Developer requests lodash@4.17.21 from @Local view
↓
Azure Artifacts checks npmjs.com (upstream source)
↓
Package downloaded and cached in @Local view
↓
Package available in feed, can be promoted
↓
Promote to @Release for production use
```

**Important notes**:
- **Upstream only with @Local**: Other views don't trigger upstream source evaluation
- **Cache first**: Once cached, package is treated like any other feed package
- **Promotion control**: Control which upstream packages are approved for production

## Deciding When to Promote

It's up to you to decide how and when to promote packages to a specific view.

### Promotion Criteria

#### Promote to @Prerelease
- **Build success**: Package built successfully in CI pipeline
- **Unit tests passed**: All unit tests pass
- **Code quality**: Code quality gates met (coverage, static analysis)
- **Initial validation**: Basic smoke tests passed

#### Promote to @Release
- **Integration tests**: Integration tests passed successfully
- **QA approval**: Quality assurance team approves
- **Security scan**: Security vulnerability scan passed
- **Performance validated**: Performance tests meet requirements
- **Documentation complete**: Release notes and documentation finalized

### Manual versus Automated Promotion

#### Manual Promotion
- **Human judgment**: Requires human decision
- **Complex criteria**: When criteria are difficult to automate
- **High-risk packages**: Critical packages requiring extra scrutiny

#### Automated Promotion
- **Consistent criteria**: When criteria are clear and testable
- **Frequent releases**: High-velocity development teams
- **Lower-risk packages**: Internal libraries with good test coverage

## Automating Package Promotion

This process can be automated by using an Azure Pipelines task as part of the build pipeline.

### Using Azure Pipelines for Promotion

```yaml
# Promote package to Prerelease view
- task: UniversalPackages@0
  inputs:
    command: "publish"
    publishDirectory: "$(Build.ArtifactStagingDirectory)"
    feedsToUsePublish: "internal"
    vstsFeedPublish: "MyFeed"
    vstsFeedPackagePublish: "my-package"
    packagePublishDescription: "Package from build $(Build.BuildNumber)"

# Promote to Release view after tests
- task: AzureCLI@2
  inputs:
    azureSubscription: "Azure Subscription"
    scriptType: "bash"
    scriptLocation: "inlineScript"
    inlineScript: |
      az artifacts universal promote \
        --organization https://dev.azure.com/MyOrg \
        --feed MyFeed \
        --name my-package \
        --version $(packageVersion) \
        --view Release
```

### Multi-Stage Pipeline Example

Complete pipeline with promotion gates:

```yaml
trigger:
  - main

stages:
  # Build and publish to Local
  - stage: Build
    jobs:
      - job: BuildPackage
        steps:
          - script: dotnet pack -o $(Build.ArtifactStagingDirectory)
            displayName: "Build package"

          - task: NuGetCommand@2
            inputs:
              command: "push"
              packagesToPush: "$(Build.ArtifactStagingDirectory)/**/*.nupkg"
              nuGetFeedType: "internal"
              publishVstsFeed: "MyFeed"

  # Promote to Prerelease after unit tests
  - stage: PromoteToPrerelease
    dependsOn: Build
    jobs:
      - job: RunTests
        steps:
          - script: dotnet test
            displayName: "Run unit tests"

          - task: AzureCLI@2
            displayName: "Promote to Prerelease"
            inputs:
              azureSubscription: "Azure Subscription"
              scriptType: "bash"
              scriptLocation: "inlineScript"
              inlineScript: |
                az artifacts universal promote \
                  --organization https://dev.azure.com/MyOrg \
                  --feed MyFeed \
                  --name MyPackage \
                  --version $(Build.BuildNumber) \
                  --view Prerelease

  # Promote to Release after integration tests and approval
  - stage: PromoteToRelease
    dependsOn: PromoteToPrerelease
    jobs:
      - job: IntegrationTests
        steps:
          - script: dotnet test --filter Category=Integration
            displayName: "Run integration tests"

      - deployment: PromoteToRelease
        dependsOn: IntegrationTests
        environment: "Production-Approval"
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureCLI@2
                  displayName: "Promote to Release"
                  inputs:
                    azureSubscription: "Azure Subscription"
                    scriptType: "bash"
                    scriptLocation: "inlineScript"
                    inlineScript: |
                      az artifacts universal promote \
                        --organization https://dev.azure.com/MyOrg \
                        --feed MyFeed \
                        --name MyPackage \
                        --version $(Build.BuildNumber) \
                        --view Release
```

## Manual Promotion via Web Interface

Promote packages through Azure DevOps portal:

1. **Navigate to Artifacts**: Go to Artifacts in Azure DevOps
2. **Select feed**: Choose the feed containing your package
3. **Find package**: Locate the package version to promote
4. **Package details**: Select the package to view details
5. **Promote button**: Select the "Promote" button
6. **Select view**: Choose the target view (@Prerelease or @Release)
7. **Confirm**: Confirm the promotion

## Promotion and Retention Policies

Packages that have been promoted to a view won't be deleted based on the retention policies.

### Retention Protection
Promotion provides retention protection:
- **@Local packages**: Subject to retention policies
- **Promoted packages**: Protected from automatic deletion
- **Long-term storage**: Released packages kept indefinitely

**Retention policy behavior**:
| Package State | Retention Policy Applies | Protected from Deletion |
|---------------|-------------------------|------------------------|
| @Local only | Applies | No |
| @Prerelease | Applies to @Local, not @Prerelease | Yes |
| @Release | Applies to @Local, not @Release | Yes |

**Example**:
```
Feed retention policy: Keep only last 30 days in @Local

Package version 1.0.0:
- Published to @Local on Day 1
- Promoted to @Release on Day 5
- Day 31: Still available because promoted to @Release
- Would be deleted if not promoted
```

### Configuring Retention Policies
Set retention policies per feed:
1. **Feed settings**: Navigate to feed settings
2. **Retention policies**: Select retention policy section
3. **Configure**: Set maximum days to keep packages in @Local
4. **Save**: Apply the policy

**Best practices**:
- **Promote important packages**: Ensure important packages are promoted
- **Regular cleanup**: Let retention policies clean up unpromoted packages
- **Balance storage**: Balance between storage costs and retention needs

## Promotion Best Practices

### Clear Criteria
- **Document promotion criteria**: Define what qualifies a package for each view
- **Consistent application**: Apply criteria consistently across all packages
- **Team alignment**: Ensure team understands promotion workflows

### Automation
- **Automate when possible**: Automate promotion for clear criteria
- **Manual gates for critical packages**: Use manual approval for production promotions
- **Testing before promotion**: Always test before promoting

### Communication
- **Release notes**: Include release notes when promoting to @Release
- **Change logs**: Maintain changelog of what changed in each version
- **Notifications**: Notify consumers when packages are promoted to @Release

### Rollback Strategy
- **Keep previous versions**: Don't delete previous @Release versions
- **Quick rollback**: Allow consumers to quickly revert to previous versions
- **Version pinning**: Support consumers pinning to specific versions

## Monitoring Package Promotions

### Track Promotion History
- **Audit logs**: Azure DevOps logs all promotion activities
- **Package history**: View promotion history for each package
- **Reports**: Generate reports on promotion patterns

### Metrics to Track
| Metric | Description |
|--------|-------------|
| **Time to promote** | How long from @Local to @Release |
| **Promotion failures** | Packages that fail promotion criteria |
| **Rollback rate** | How often packages need to be rolled back |
| **Promotion velocity** | Number of promotions per time period |

## Quick Reference

### Promotion Workflow
```
Build → @Local (automatic)
   ↓
Unit Tests Pass → Promote to @Prerelease (automated/manual)
   ↓
Integration Tests + QA Approval → Promote to @Release (manual gate)
```

### Promotion Decision Matrix
| Criteria | @Prerelease | @Release |
|----------|-------------|----------|
| Build success | ✅ Required | ✅ Required |
| Unit tests pass | ✅ Required | ✅ Required |
| Integration tests pass | ⚠️ Optional | ✅ Required |
| QA approval | ❌ Not needed | ✅ Required |
| Security scan | ⚠️ Optional | ✅ Required |
| Performance validated | ❌ Not needed | ✅ Required |

## Critical Notes
🎯 **Promotion is explicit** - Packages must be explicitly promoted; it doesn't happen automatically

📦 **Version stays the same** - Promotion changes view association, not the version number

🔒 **Retention protection** - Promoted packages are protected from automatic deletion

🔄 **Cumulative visibility** - Packages in @Release also appear in @Prerelease and @Local

⚠️ **Upstream packages need promotion** - Cached upstream packages must be promoted to use in other views

🏗️ **Automate promotion** - Use Azure Pipelines to automate promotion based on test results

✅ **Test before promoting** - Always run appropriate tests before promoting to higher quality views

## Learn More
- [Azure Artifacts Views](https://learn.microsoft.com/en-us/azure/devops/artifacts/concepts/views)
- [Package Promotion](https://learn.microsoft.com/en-us/azure/devops/artifacts/feeds/views)
