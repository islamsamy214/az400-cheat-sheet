# Module Assessment

## Key Concepts
This unit contains knowledge check questions to assess understanding of GitHub Packages concepts covered in this module.

## Assessment Topics

The module assessment covers the following key areas:

### GitHub Packages Fundamentals
- Understanding what GitHub Packages is and its key benefits
- Supported package registries (npm, NuGet, Maven, Docker, etc.)
- Integration with GitHub features and workflows
- Use cases for GitHub Packages

### Publishing Packages
- Personal access token (PAT) creation and scopes
- Authentication methods for different package types
- Publishing workflow (authenticate, configure, publish)
- NuGet publishing configuration and commands
- npm publishing configuration and commands
- Automating publishing with GitHub Actions
- Best practices for package publishing

### Installing Packages
- Package discovery methods
- Installation workflow (authenticate, install)
- NuGet installation methods (Visual Studio, CLI, .csproj)
- npm installation configuration (.npmrc, package.json)
- Installing packages in CI/CD pipelines
- Troubleshooting installation issues
- Using GITHUB_TOKEN in workflows

### Deleting and Restoring Packages
- Understanding deletion permissions and restrictions
- Public package deletion limits (5,000 downloads)
- Deleting package versions vs entire packages
- 30-day restoration window requirements
- Using REST API for package management
- Best practices for package lifecycle management

### Access Control and Visibility
- Repository-inherited permissions vs granular permissions
- Visibility options (public, private, internal)
- Container image permission roles (read, write, admin)
- Configuring access for container images
- Security best practices and organizational strategies

## Assessment Format

The knowledge check typically includes:
- Multiple choice questions
- Scenario-based questions
- True/false questions
- Questions about best practices and recommended approaches

## Key Concepts to Review

Before taking the assessment, ensure you understand:

1. **PAT scopes** - Which scopes are required for publishing, downloading, and deleting
2. **Authentication methods** - How to authenticate for NuGet and npm
3. **GitHub Actions integration** - Using GITHUB_TOKEN in workflows
4. **Deletion restrictions** - 5,000 download limit for public packages
5. **Restoration window** - 30-day limit for restoring deleted packages
6. **Permission models** - Repository-inherited vs granular (containers)
7. **Visibility options** - Public, private, and internal differences
8. **Container permissions** - Read, write, and admin roles

## Quick Reference for Assessment

### PAT Scopes
| Scope | Purpose |
|-------|---------|
| `write:packages` | Publish packages |
| `read:packages` | Download packages |
| `delete:packages` | Delete package versions |
| `repo` | Access private repository packages |

### Package Registry Commands
| Registry | Publish | Install |
|----------|---------|---------|
| **NuGet** | `dotnet nuget push` | `dotnet restore` |
| **npm** | `npm publish` | `npm install` |

### Deletion Rules
| Package Type | Download Limit | Admin Required |
|--------------|----------------|----------------|
| **Private** | No limit | ✅ Yes |
| **Public** | 5,000 max | ✅ Yes |

### Permission Models
| Model | Package Types | Separate from Repo |
|-------|--------------|-------------------|
| **Repository-inherited** | npm, NuGet, Maven, RubyGems, Gradle | ❌ No |
| **Granular** | Container images | ✅ Yes |

### Visibility Types
| Type | Who Can See | Cost | Availability |
|------|-------------|------|--------------|
| **Public** | Anyone | Free | All plans |
| **Private** | Users with permission | Paid | All plans |
| **Internal** | All org members | Paid | Enterprise only |

## Common Assessment Topics

### Publishing Scenarios
**Question type**: "What PAT scope is required to publish a package?"
**Answer**: `write:packages`

**Question type**: "How do you automate package publishing?"
**Answer**: Use GitHub Actions workflows with GITHUB_TOKEN

### Installation Scenarios
**Question type**: "What file configures npm to use GitHub Packages?"
**Answer**: `.npmrc` file with registry configuration

**Question type**: "Do you need authentication for public packages?"
**Answer**: Yes, authentication is required even for public packages

### Deletion Scenarios
**Question type**: "Can you delete a public package with 10,000 downloads?"
**Answer**: No, the 5,000 download limit prevents deletion

**Question type**: "How long do you have to restore a deleted package?"
**Answer**: 30 days from deletion

### Access Control Scenarios
**Question type**: "Which package type supports granular permissions?"
**Answer**: Container images (Container registry)

**Question type**: "What visibility type requires GitHub Enterprise?"
**Answer**: Internal packages

## Critical Notes
📝 **Review all units** - Make sure you've read through all module content

🎯 **Focus on practical scenarios** - Understand when to use each feature

🔐 **Know authentication** - PAT scopes and configuration methods

⏰ **Remember limits** - 5,000 downloads for public deletion, 30-day restoration

🔒 **Understand permissions** - Two models, different capabilities

✅ **Best practices matter** - Questions often ask about recommended approaches

## Preparation Tips

1. **Review key workflows** - Publishing, installing, deleting, restoring
2. **Understand authentication** - PAT scopes, .npmrc, nuget.config
3. **Know the limits** - Download limits, restoration windows
4. **Compare permission models** - Repository-inherited vs granular
5. **Memorize visibility types** - Public, private, internal differences

## Learn More
- Review all units in this module before attempting the assessment
- [GitHub Packages Documentation](https://docs.github.com/packages)
- [Working with a GitHub Packages registry](https://docs.github.com/packages/working-with-a-github-packages-registry)
