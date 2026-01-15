# Examine Roles

## Key Concepts
- Azure Artifacts uses role-based access control (RBAC) model
- Four roles with incremental permissions
- Understanding roles is essential for proper security and access control

## The Four Azure Artifacts Roles

### Role Hierarchy
Roles are in incremental order:
- **Reader**: Can list and restore (or install) packages from feed
- **Collaborator**: Can save packages from upstream sources
- **Contributor**: Can push and unlist packages in feed
- **Owner**: Has all available permissions for package feed

**Permission Inheritance:**
- Reader → Collaborator → Contributor → Owner
- Each level adds more permissions while retaining all permissions from lower levels
- This incremental model simplifies role management

## Role Definitions

### Reader
Can list and restore (or install) packages from the feed.

**Specific Permissions:**
- List packages: View all packages in feed
- Restore packages: Download and install packages
- View package metadata: Read package descriptions, versions, and dependencies
- View feed details: See feed settings and configuration

**Common Use Cases:**
- Developers who only need to consume packages from feed
- Build agents that download packages during builds
- Teams that use packages but don't need to publish
- External partners with limited access to specific packages

**When to Assign:**
Assign Reader role when users only need to consume packages, not modify them. This is the default role for most organization members.

### Collaborator
Can save packages from upstream sources.

**Specific Permissions:**
- All Reader permissions: Can list and restore packages
- Save packages from upstream: Can save packages from public registries (npmjs.com, NuGet.org, etc.) to your feed
- Cache packages: Store copies of upstream packages in your feed for offline access

**Common Use Cases:**
- Build services: Project Collection Build Service that needs to cache upstream packages
- Developers who work with upstream sources and want to ensure package availability
- Automated systems that populate feeds with external packages

**When to Assign:**
Assign Collaborator role when you use upstream sources and need users or services to cache packages from external registries. This role was added specifically to support upstream sources functionality.

### Contributor
Can push and unlist packages in the feed.

**Specific Permissions:**
- All Collaborator permissions: Can list, restore, and save packages
- Push packages: Publish new packages to feed
- Unlist packages: Hide specific package versions from search results
- Promote packages: Move packages between views (e.g., from Prerelease to Release)
- Delete packages: Remove packages from feed (may be restricted by feed settings)

**Common Use Cases:**
- Package maintainers who create and publish packages
- Build pipelines: Automated builds that publish packages
- Development teams with authority to publish packages for their projects

**When to Assign:**
Assign Contributor role to users and service principals that need to publish packages to your feed. This is appropriate for developers on teams that own specific packages.

**Security Considerations:**
- Package integrity: Contributors can publish packages, so ensure they're trusted
- Version management: Contributors can unlist but typically not delete packages
- Review workflows: Consider implementing manual reviews before publishing to production feeds

### Owner
Has all available permissions for a package feed.

**Specific Permissions:**
- All Contributor permissions: Can list, restore, save, push, and unlist packages
- Manage feed settings: Configure feed settings, upstream sources, views, and retention policies
- Manage permissions: Add or remove users, assign roles, and modify access control
- Delete feed: Remove the entire feed
- Edit feed details: Change feed name, description, and visibility

**Common Use Cases:**
- Feed administrators responsible for feed configuration and management
- Project administrators who oversee all aspects of package management
- DevOps engineers managing feed infrastructure

**When to Assign:**
Assign Owner role sparingly to users who need full administrative control over the feed. Typically, this should be limited to a small number of trusted administrators.

**Security Considerations:**
- Principle of least privilege: Only assign Owner role when necessary
- Multiple owners: Ensure multiple owners to prevent single points of failure
- Audit trails: Monitor owner actions for compliance

## Default Role Assignments

### Project Collection Build Service
When creating Azure Artifacts feed, the Project Collection Build Service is given Contributor rights by default.

**Build Identity:**
- Organization-wide build identity in Azure Pipelines
- Can access feeds it needs when running tasks

**Project-Level Build Identity:**
If you changed build identity to project level, need to give that identity permissions to access feed.

**Best Practice:**
For publishing packages from builds, consider using service principal or Personal Access Token with appropriate permissions.

### Team Project Contributors
Any contributors to team project are also contributors to feed.

**Automatic Inheritance:**
- Project role mapping: Project contributors automatically get Contributor role on feeds
- Convenience: Simplifies permission management for development teams
- Override capability: Can be changed if more restrictive access is needed

### Project Collection Administrators
Project Collection Administrators and administrators of team project, plus feed's creator, are automatically made owners of feed.

**Administrative Access:**
- Full control: Administrators have complete control over feed settings
- Multiple owners: Ensures administrative continuity
- Feed creator: User who created feed gets Owner role

### Modifying Default Assignments
The roles for these users and groups can be changed or removed.

**Customization Options:**
- Remove default assignments: If automatic assignments don't match security requirements
- Change roles: Downgrade or upgrade roles as needed
- Add custom permissions: Grant access to additional users, teams, or groups

## Role Assignment Best Practices

### Principle of Least Privilege
- Start with Reader for everyone
- Elevate to Collaborator only if using upstream sources
- Assign Contributor to package publishers
- Limit Owner to feed administrators

### Team-Based Assignments
- **Scalability**: Manage permissions for groups, not individuals
- **Maintainability**: Add/remove users from teams, not feeds
- **Clarity**: Team names indicate the purpose of access

### Service Principal Access
- **Build pipelines**: Use service principals with Contributor role
- **Deployment pipelines**: Use service principals with Reader role
- **Upstream caching**: Use service principals with Collaborator role

### Regular Access Reviews
- Remove stale access when users change roles or leave
- Verify necessity: Ensure users still need their assigned roles
- Audit compliance: Document access reviews for compliance

## Role Comparison Table

| Role | List/Restore | Save from Upstream | Push/Unlist | Manage Feed |
|------|--------------|-------------------|-------------|-------------|
| **Reader** | ✓ | ✗ | ✗ | ✗ |
| **Collaborator** | ✓ | ✓ | ✗ | ✗ |
| **Contributor** | ✓ | ✓ | ✓ | ✗ |
| **Owner** | ✓ | ✓ | ✓ | ✓ |

## Changing Role Assignments

### In Azure DevOps Web Portal
1. Navigate to Artifacts: Select your feed
2. Feed settings: Click the gear icon for settings
3. Permissions: Select the Permissions tab
4. Add users: Click "Add users/groups" and select users, teams, or groups
5. Assign role: Choose appropriate role from dropdown
6. Save: Confirm the changes

### Using Azure CLI
```bash
# Add user to feed with Contributor role
az artifacts feed permission add --feed <feed-name> --role contributor --user <user-email>
```

### Using REST API
Use Azure DevOps REST API to programmatically manage role assignments for automation scenarios.

## Critical Notes
- 🎯 Four incremental roles: Reader → Collaborator → Contributor → Owner
- 💡 Default assignments include build service, project contributors, and administrators
- ⚠️ Use principle of least privilege when assigning roles
- 📊 Team-based assignments are more scalable than individual assignments
- 🔒 Limit Owner role to trusted administrators
- 🔄 Regularly review and adjust role assignments

[Learn More](https://learn.microsoft.com/en-us/training/modules/migrate-consolidating-secure-artifacts/5-examine-roles)
