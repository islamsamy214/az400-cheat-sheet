# Explore Package Access Control and Visibility

## Key Concepts
Access control and visibility settings determine who can view, download, and manage your packages in GitHub Packages. Understanding these settings is crucial for security, compliance, and collaboration.

## Understanding Permission Models

GitHub Packages supports two permission models depending on the package type:

### Repository-Inherited Permissions

Most package registries (npm, NuGet, Maven, RubyGems, Gradle) inherit permissions from their repository:

**Characteristics**:
- **Same access control**: Package permissions match repository permissions
- **Simplified management**: One set of permissions for both code and packages
- **Automatic updates**: Permission changes to repository automatically apply to packages
- **Team-based access**: Repository teams and collaborators have package access

**Pros and cons**:
| Pro | Con |
|-----|-----|
| Simple and consistent with repository access | Cannot separate code access from package access |
| No additional permission configuration needed | All repository collaborators can publish packages |

### Granular Permissions (Container Registry Only)

Container registry packages support independent access control:

**Characteristics**:
- **Separate from repository**: Package permissions can differ from repository
- **User-scoped**: Grant access to individual users
- **Organization-scoped**: Grant access to teams within organizations
- **Role-based**: Assign read, write, or admin roles independently

**Pros and cons**:
| Pro | Con |
|-----|-----|
| Fine-grained control over package access | More complex permission management |
| Can share packages without sharing code | Only available for container images |

> **Important**: You can change package access control and visibility separately from the repository for container registry packages. For other package types, visibility and access are tied to the repository.

## Package Visibility Options

Visibility determines who can discover and access your packages:

### Public Packages

**Characteristics**:
- **Anyone can discover**: Package appears in search results for all users
- **Anyone can download**: No authentication required to install (but authentication required for uploads)
- **Open source friendly**: Ideal for community libraries and tools
- **Free storage**: No storage or bandwidth limits for public packages
- **Usage tracking**: Download statistics visible to everyone

**Use cases for public packages**:
- Open-source libraries and frameworks
- Community tools and utilities
- Public API clients and SDKs
- Educational examples and templates

### Private Packages

**Characteristics**:
- **Organization/user only**: Only authenticated users with permission can discover
- **Access controlled**: Must have read permission to download
- **Secure by default**: Ideal for internal libraries and proprietary code
- **Usage limits**: Subject to GitHub plan storage and bandwidth limits
- **Usage tracking**: Statistics visible only to authorized users

**Use cases for private packages**:
- Internal company libraries
- Proprietary dependencies
- Pre-release versions
- Client-specific customizations

### Internal Packages (GitHub Enterprise Only)

**Characteristics**:
- **Organization-wide access**: All organization members can discover and download
- **Simplified sharing**: No individual permission grants needed
- **Enterprise feature**: Available only with GitHub Enterprise Cloud or Server
- **Balanced security**: More accessible than private, more secure than public

**Use cases for internal packages**:
- Shared organizational libraries
- Cross-team dependencies
- Internal standards and frameworks
- Common utilities and helpers

## Container Image Permissions

If you have admin permissions to a container image, you can configure granular access control:

### Setting Visibility
- **Public visibility**: Anyone can pull the container image without authentication
- **Private visibility**: Only users with explicit permissions can pull the image

### Granting Access Permissions

**Personal account containers**:
- Grant any GitHub user an access role
- Specify individual permissions for each user
- Manage access independently from repository

**Organization containers**:
- Grant access to any person in the organization
- Grant access to any team in the organization
- Combine individual and team permissions
- Inherit permissions from organization membership

### Permission Roles

| Role | Capabilities |
|------|-------------|
| **read** | Can download the package. Can read package metadata. |
| **write** | Can upload and download this package. Can read and write package metadata. |
| **admin** | Can upload, download, delete, and manage this package. Can read and write package metadata. Can grant package permissions. |

**Permission inheritance**:
- **Admin role**: Includes all write and read permissions
- **Write role**: Includes all read permissions
- **Read role**: Minimum permissions for consuming packages

### Configuring Access

To configure access control and visibility for container images:

1. Navigate to the package page
2. Click **Package settings**
3. Under **Danger Zone**, change visibility (public/private)
4. Under **Manage access**, add users or teams
5. Select role for each user or team (read, write, admin)
6. Click **Add** to grant permissions

## Best Practices for Access Control

### Security Recommendations
- **Principle of least privilege**: Grant minimum necessary permissions
- **Regular audits**: Review and remove unnecessary access periodically
- **Team-based access**: Use teams instead of individual users when possible
- **Public carefully**: Ensure public packages don't expose sensitive information
- **Document access**: Maintain records of who has access and why
- **Automate reviews**: Use tools to detect permission changes

### Organizational Strategies
- **Standardize visibility**: Establish defaults for different package types
- **Access request process**: Define how users request package access
- **Ownership clarity**: Assign clear ownership for package management
- **Integration with SSO**: Use SAML/SCIM for centralized access management
- **Compliance alignment**: Ensure access controls meet regulatory requirements

## Quick Reference

### Permission Model Comparison
| Aspect | Repository-Inherited | Granular (Container) |
|--------|---------------------|---------------------|
| **Package types** | npm, NuGet, Maven, RubyGems, Gradle | Container images only |
| **Separate from repo** | ❌ No | ✅ Yes |
| **Individual user access** | Via repo access | ✅ Yes |
| **Team access** | Via repo teams | ✅ Yes |
| **Role-based** | ❌ No | ✅ Yes (read/write/admin) |
| **Complexity** | Simple | Complex |

### Visibility Comparison
| Visibility | Who Can Discover | Who Can Download | Storage Cost | Use Case |
|------------|-----------------|------------------|--------------|----------|
| **Public** | Anyone | Anyone | Free | Open source, community packages |
| **Private** | Users with permission | Users with read permission | Paid | Internal, proprietary code |
| **Internal** | All org members | All org members | Paid | Organization-wide sharing |

### Container Permission Roles Matrix
| Action | Read | Write | Admin |
|--------|------|-------|-------|
| **Download package** | ✅ | ✅ | ✅ |
| **View metadata** | ✅ | ✅ | ✅ |
| **Upload package** | ❌ | ✅ | ✅ |
| **Edit metadata** | ❌ | ✅ | ✅ |
| **Delete package** | ❌ | ❌ | ✅ |
| **Grant permissions** | ❌ | ❌ | ✅ |
| **Change visibility** | ❌ | ❌ | ✅ |

## Critical Notes
🔐 **Two permission models** - Repository-inherited (most packages) vs Granular (containers only)

🎯 **Container images special** - Only package type with independent access control

📦 **Public doesn't mean open** - Authentication still required to publish public packages

🏢 **Internal requires Enterprise** - Internal visibility only available with GitHub Enterprise

✅ **Least privilege principle** - Grant minimum necessary permissions for security

👥 **Team-based is better** - Use teams instead of individual users for easier management

🔒 **Regular audits essential** - Review and remove unnecessary access periodically

⚠️ **Public carefully** - Ensure no sensitive information in public packages

## Learn More
- [About permissions for GitHub Packages](https://docs.github.com/packages/learn-github-packages/about-permissions-for-github-packages)
- [Configuring a package's access control and visibility](https://docs.github.com/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility)
