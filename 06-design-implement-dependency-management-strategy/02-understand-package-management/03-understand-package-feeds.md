# Understand Package Feeds

## Key Concepts
- Package feeds are centralized storage for packages
- Also called repositories or registries depending on context
- Enable distribution and consumption of packaged components

**Terminology:**
- **Feed**: General term (Azure Artifacts and others)
- **Repository**: Common in Maven and Docker contexts
- **Registry**: Used by npm and Docker

## Feed Characteristics

### Type-Specific Feeds
Each package type typically has its own type of feed:
- **NuGet feeds**: Store .NET packages
- **npm feeds**: Store JavaScript/Node.js packages
- **Maven repositories**: Store Java packages
- **PyPI feeds**: Store Python packages
- **Docker registries**: Store container images

### Versioned Storage
Package feeds offer versioned storage of packages.

**Benefits:**
- **Compatibility**: Maintain compatibility with different project requirements
- **Rollback**: Revert to previous versions if issues arise
- **Testing**: Test with specific versions before upgrading
- **Stability**: Pin dependencies to known-good versions for production

## Private and Public Feeds

### Public Feeds
Accessible to anyone without authentication.

**Characteristics:**
- Open access without credentials
- Anonymous consumption, no account required
- Optional authentication for publishing
- Packages discoverable through search and browsing

**Examples:**
- NuGet.org (for .NET packages)
- npmjs.com (for JavaScript packages)
- Maven Central (for Java packages)
- PyPI.org (for Python packages)
- Docker Hub (for container images)

**Typical Use Cases:**
- Open-source projects
- Community packages
- Free access to reusable components

### Private Feeds
Only accessible with authentication.

**Why Use Private Feeds:**
- **Intellectual property**: Contains proprietary code or business logic
- **Internal use**: Components developed for internal use
- **Security**: Sensitive code that shouldn't be publicly exposed
- **Licensing**: Packages with restricted licensing agreements
- **Development stage**: Pre-release or experimental packages

**Key Difference**: Main difference is the need for authentication
- Public feeds: Can be anonymously accessible and optionally authenticated
- Private feeds: Require authentication with proper credentials

**Access Control:**
- User-based access control
- Team-based access control
- Organization-based restriction
- Role-based permissions (readers, contributors, owners)

## Feed Visibility Options

| Visibility | Access | Use Cases |
|------------|--------|-----------|
| **Public** | Anyone can consume | Open-source packages, community libraries |
| **Private** | Only authenticated users | Internal components, proprietary code |
| **Organization** | Members of your organization | Shared across teams within company |
| **Project** | Members of specific project | Project-specific packages |

## Azure Artifacts Feed Configuration
- **Feed permissions**: Control who can read, contribute, and administer
- **Upstream sources**: Include packages from public sources
- **Feed views**: Create filtered views of feed (e.g., Release, Prerelease)

## Critical Notes
- 🎯 Feeds are centralized storage for package distribution
- 💡 Each feed typically contains one type of package
- ⚠️ Private feeds require authentication and proper access control
- 📊 Versioned storage enables rollback and compatibility management

[Learn More](https://learn.microsoft.com/en-us/training/modules/understand-package-management/3-understand-package-feeds)
