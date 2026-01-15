# Module Assessment

## Key Concepts
This unit contains knowledge check questions to assess understanding of the versioning strategy implementation concepts covered in this module.

## Assessment Topics

The module assessment covers the following key areas:

### Versioning Fundamentals
- Understanding the importance of versioning in dependency management
- Package immutability principles
- Version increment rules and types of changes
- Technical implementation across different package types

### Semantic Versioning (SemVer)
- Major.Minor.Patch format and increment rules
- Prerelease labels and their conventions
- Version precedence and comparison rules
- Build metadata usage

### Azure Artifacts Views
- Understanding the three default views (@Local, @Prerelease, @Release)
- View-specific URI formats
- Differences between prerelease labels and @Prerelease view
- View usage across different package types

### Package Promotion
- Promotion fundamentals and workflows
- Package visibility rules in different views
- Upstream sources and view interaction
- Automated vs manual promotion strategies

### Best Practices
- Documented versioning strategies
- Adopting SemVer 2.0
- Single feed per repository approach
- Automated publishing and promotion
- Testing before promotion
- Retention policies and version control

## Assessment Format

The knowledge check typically includes:
- Multiple choice questions
- Scenario-based questions
- True/false questions
- Questions about best practices and recommended approaches

## Key Concepts to Review

Before taking the assessment, ensure you understand:

1. **Version immutability** - Published packages cannot be changed
2. **SemVer format** - Major.Minor.Patch-label+metadata
3. **View hierarchy** - @Local (all) → @Prerelease (tested) → @Release (production)
4. **Promotion workflow** - Explicit action to move packages between quality gates
5. **Best practices** - Documented strategy, automated processes, testing gates

## Common Assessment Topics

### Version Increment Scenarios
Understanding when to increment:
- **Major version**: Breaking changes, removed functionality
- **Minor version**: New features, backward compatible
- **Patch version**: Bug fixes, security patches

### View Usage Scenarios
Knowing which view to use:
- **@Local**: Development, all packages, upstream sources
- **@Prerelease**: QA testing, labeled versions
- **@Release**: Production deployments, stable packages

### Promotion Decisions
When and how to promote packages:
- Build success and unit tests → @Prerelease
- Integration tests and QA approval → @Release
- Automated vs manual promotion criteria

## Quick Reference for Assessment

### Key Principles
| Principle | Description |
|-----------|-------------|
| **Immutability** | Published versions cannot be changed |
| **Semantic versioning** | Major.Minor.Patch format communicates change scope |
| **Views as quality gates** | Separate packages by maturity level |
| **Explicit promotion** | Packages must be promoted to appear in views |
| **Automated workflows** | Use CI/CD for consistent versioning and promotion |

### Version Increment Quick Guide
| Change Type | Version Change | Example |
|-------------|----------------|---------|
| Breaking changes | Major (X.0.0) | 1.5.3 → 2.0.0 |
| New features | Minor (x.Y.0) | 1.5.3 → 1.6.0 |
| Bug fixes | Patch (x.y.Z) | 1.5.3 → 1.5.4 |

### View Characteristics
| View | Contains | Upstream | Production Ready |
|------|----------|----------|------------------|
| @Local | All packages | Yes | No |
| @Prerelease | Labeled versions | No | No |
| @Release | Stable packages | No | Yes |

## Critical Notes
📝 **Review all units** - Make sure you've read through all module content before taking the assessment

🎯 **Understand scenarios** - Focus on when to apply concepts, not just what they are

🔄 **Practice examples** - Work through the code examples and workflows provided in the module

✅ **Know best practices** - Understand why certain practices are recommended

⚠️ **Common mistakes** - Be aware of anti-patterns and what not to do

## Preparation Tips

1. **Review key concepts** from each unit
2. **Understand workflows** - Promotion, versioning, publishing
3. **Know the differences** - Between views, version types, labels
4. **Practice scenarios** - Think through real-world applications
5. **Memorize formats** - SemVer format, view URIs, promotion workflows

## Learn More
- Review all units in this module before attempting the assessment
- [Azure Artifacts Documentation](https://learn.microsoft.com/en-us/azure/devops/artifacts/)
- [Semantic Versioning 2.0.0](https://semver.org/)
