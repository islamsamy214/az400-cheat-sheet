# Examine Permissions

## Key Concepts
- Azure Artifacts provides granular permissions that control what users can do with package feeds
- Permissions operate at feed level and view level
- Understanding permissions is critical for implementing proper access control

## Feed-Level Permissions

### Permissions Roles Matrix
Permission categories:
- **Read permissions**: List and restore packages
- **Write permissions**: Push, unlist, and promote packages
- **Administrative permissions**: Manage feed settings and permissions
- **Upstream permissions**: Save packages from upstream sources

### Assigning Permissions
You can assign users, teams, and groups to a specific role for each permission, giving the permissions corresponding to that role.

**Requirements:**
- Need to have Owner role to assign permissions

**Best Practices:**
- **Use teams**: Assign permissions to teams rather than individual users for easier management
- **Microsoft Entra groups**: Leverage Microsoft Entra ID groups for organization-wide permission management
- **Service principals**: Use service principals for automated systems and CI/CD pipelines

### Feed Users
Once an account has access to feed from permission to list and restore packages, it's considered a Feed user.

**Feed User Definition:**
- Minimum permission: Any user with at least Reader role
- Access scope: Can view and consume packages from feed
- Visibility: Shown in feed permissions list

## View-Level Permissions

### Default Views
Any feed user has access to all views, whether default views or newly created ones.

**View Types:**
- **@Local**: Contains all packages published directly to feed and packages saved from upstream sources
- **@Release**: Contains packages promoted to Release view, indicating they're production-ready
- **@Prerelease**: Contains pre-release versions for testing and validation before promoting to Release

**View Access:**
- Automatic access: Feed users automatically have access to all views
- No view-specific permissions: Views don't have separate permission settings
- Promotion control: Use roles to control who can promote packages between views

### Custom Views
**Creating Custom Views:**
- Purpose-specific: Create views for different stages (development, staging, production)
- Naming conventions: Use descriptive names like @Development, @QA, @Production
- Promotion workflow: Define promotion paths between views

## Feed Visibility

### Visibility Options

**Organization Visibility:**
- Description: Feed is visible to everyone in your Azure DevOps organization
- Discovery: Users can see feed exists, even if they don't have access to packages
- Use case: Internal shared libraries used across multiple projects

**Project Visibility:**
- Description: Feed is visible only to members of specific project
- Scope: Limited to project team members
- Use case: Project-specific packages not relevant to other teams

**Private Visibility:**
- Description: Feed is visible only to users, teams, and groups explicitly granted permissions
- Restricted discovery: Users outside permission list can't see feed exists
- Use case: Sensitive or proprietary packages requiring strict access control

## Granular Permission Operations

### List and Restore Packages
**Permission**: Reader role and above

**Operations:**
- List packages: View available packages and their metadata
- Restore/install packages: Download packages for use in projects
- View versions: See all available versions of packages
- Read metadata: Access package descriptions, dependencies, and release notes

### Save Packages from Upstream
**Permission**: Collaborator role and above

**Operations:**
- Cache upstream packages: Save packages from public registries to your feed
- Ensure availability: Make packages available even if upstream source is unavailable
- Version pinning: Lock specific versions from upstream sources

### Push and Unlist Packages
**Permission**: Contributor role and above

**Operations:**
- Push packages: Publish new packages or versions to feed
- Unlist packages: Hide specific versions from search results without deleting
- Promote packages: Move packages between views (@Prerelease → @Release)
- Deprecate packages: Mark packages as deprecated

### Delete/Unpublish Packages
**Permission**: Contributor role and above (may be restricted by feed settings)

**Operations:**
- Delete packages: Permanently remove package versions
- Unpublish: Remove packages from feed entirely
- Recycle bin: Deleted packages may be recoverable for a period

**Best Practice:**
Unlisting is preferred over deletion to maintain version history and prevent breaking builds that reference specific versions.

### Edit Feed
**Permission**: Owner role only

**Operations:**
- Change feed settings: Modify retention policies, upstream sources, and views
- Rename feed: Change feed name (affects package URLs)
- Delete feed: Permanently remove feed and all packages
- Configure retention: Set how long to keep old package versions

### Administer Permissions
**Permission**: Owner role only

**Operations:**
- Add users/teams/groups: Grant access to feed
- Assign roles: Set Reader, Collaborator, Contributor, or Owner roles
- Remove access: Revoke permissions from users or groups
- View audit logs: Review permission changes and access history

## Permission Best Practices

### Principle of Least Privilege

**Start Restrictive:**
- Default to Reader: Grant read-only access by default
- Elevate as needed: Promote to higher roles only when justified
- Regular reviews: Periodically audit and reduce unnecessary permissions

### Separation of Concerns

**Different Feeds for Different Purposes:**
- Development feed: Contributors can push frequently
- Production feed: Only automated pipelines can push after approvals
- Shared libraries feed: Tightly controlled with limited contributors

### Automated Systems

**Service Principals for CI/CD:**
- Build pipelines: Use service principal with Contributor role to publish packages
- Deployment pipelines: Use service principal with Reader role to consume packages
- Token management: Rotate Personal Access Tokens regularly

### Permission Inheritance

**Leverage Azure DevOps Groups:**
- Project contributors: Automatically inherit Contributor role on project feeds
- Administrators: Automatically get Owner role
- Override when needed: Remove default assignments for more restrictive access

## Managing Permissions

### In Azure DevOps Portal
1. Navigate to feed: Go to Artifacts and select your feed
2. Feed settings: Click gear icon to access settings
3. Permissions tab: Select Permissions section
4. Add permissions: Click "Add users/groups" and search for users, teams, or groups
5. Select role: Choose appropriate role from dropdown
6. Save changes: Confirm permission assignment

### Using Azure CLI
```bash
# Grant user Contributor permission
az artifacts feed permission add --feed <feed-name> --role contributor --user <user-email>

# Remove user permission
az artifacts feed permission remove --feed <feed-name> --user <user-email>

# List all permissions
az artifacts feed permission list --feed <feed-name>
```

### Using REST API
Use Azure DevOps REST API for automated permission management in large organizations.

```http
POST https://feeds.dev.azure.com/{organization}/_apis/packaging/Feeds/{feedId}/permissions?api-version=7.0
```

## Monitoring and Auditing

### Permission Changes
**Track Modifications:**
- Audit logs: Azure DevOps logs all permission changes
- Change notifications: Configure alerts for permission modifications
- Compliance reports: Generate reports for security audits

### Access Patterns
**Monitor Usage:**
- Package downloads: Track which users download which packages
- Publish operations: Monitor who publishes packages and when
- Failed access attempts: Alert on unauthorized access attempts

## Permission Matrix

| Operation | Reader | Collaborator | Contributor | Owner |
|-----------|--------|--------------|-------------|-------|
| **List packages** | ✓ | ✓ | ✓ | ✓ |
| **Restore packages** | ✓ | ✓ | ✓ | ✓ |
| **Save from upstream** | ✗ | ✓ | ✓ | ✓ |
| **Push packages** | ✗ | ✗ | ✓ | ✓ |
| **Unlist packages** | ✗ | ✗ | ✓ | ✓ |
| **Delete packages** | ✗ | ✗ | ✓* | ✓ |
| **Edit feed** | ✗ | ✗ | ✗ | ✓ |
| **Manage permissions** | ✗ | ✗ | ✗ | ✓ |

*May be restricted by feed settings

## Critical Notes
- 🎯 Granular permissions provide fine-grained access control
- 💡 Feed users automatically have access to all views
- ⚠️ Use principle of least privilege for all permissions
- 📊 Separate feeds by purpose and trust level
- 🔒 Monitor and audit all permission changes
- 🔄 Regular permission reviews are essential for security

[Learn More](https://learn.microsoft.com/en-us/training/modules/migrate-consolidating-secure-artifacts/6-examine-permissions)
