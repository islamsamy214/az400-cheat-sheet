# Configure Repository Permissions Using GitHub

## Key Concepts
- **Permission Model**: Different access levels based on GitHub account type
- **Account Types**: Personal accounts, organizations, teams, and enterprise
- **Repository Roles**: Read, Triage, Write, Maintain, Admin
- **Base Permissions**: Default access for organization members

## GitHub Account Types

| Account Type | Use Case | Key Features |
|--------------|----------|--------------|
| **GitHub Free (Personal)** | Individual developers | Unlimited public repos, limited private repo features |
| **GitHub Pro** | Professional developers | Advanced code review, unlimited collaborators |
| **GitHub Free (Org)** | Small organizations | Team management, organization-level permissions |
| **GitHub Team** | Growing teams | Advanced collaboration, team permissions |
| **GitHub Enterprise** | Large enterprises | SSO, audit logs, compliance, branch-level permissions |

## GitHub Free for Personal Accounts

### Public Repositories
- Anyone can view and fork
- Open collaboration with community

### Private Repositories
- Repository owner has full control
- Up to 3 collaborators allowed
- Only one owner (cannot share ownership)
- Collaborators only get write access (no read-only option)

**Limitation**: Cannot give read-only access to private repos with personal accounts

## GitHub Pro

Same permission levels as GitHub Free for personal accounts, plus:
- **Protected Branches**: Prevent force pushes and deletions
- **Code Owners**: Automatically request reviews from designated owners
- **Advanced Collaboration**: More sophisticated workflow features

## GitHub Free for Organizations

### Organization Members

**Default Role**: Organization member with basic permissions
- Create repositories
- Basic access to organization resources

### Elevated Organization Roles

| Role | Capabilities |
|------|--------------|
| **Organization Moderator** | Hide comments in public repositories |
| **Security Manager** | Read all repositories in organization |
| **Owner** | All repository permissions, manage collaborators, write/delete repos |

### Base Permissions

Set default access level for all organization members:
- Applied to all repositories
- Does NOT apply to outside collaborators
- Individual repository permissions can override base permissions

**Example**:
```
Base Permission: Read
Override for Engineering Team: Write on engineering-app repo
Result: Most members have Read, engineers have Write
```

### Built-in Repository Roles

| Role | Use Case | Permissions |
|------|----------|-------------|
| **Read** | Non-code contributors | View content, open issues |
| **Triage** | Issue/PR managers | Manage issues and PRs without write access |
| **Write** | Active contributors | Push to repository, merge PRs |
| **Maintain** | Project managers | Manage repository without destructive actions |
| **Admin** | Repository administrators | Full access including sensitive/destructive actions |

### Permission Details

**Read**:
- View repository content
- Clone repository
- Open and comment on issues
- Submit pull requests from forks

**Triage**:
- All Read permissions
- Manage issues and pull requests
- Apply labels
- Close/reopen issues
- Request PR reviews

**Write**:
- All Triage permissions
- Push to repository
- Merge pull requests
- Create branches
- Edit wikis

**Maintain**:
- All Write permissions
- Manage repository settings
- Add collaborators
- Manage webhooks and integrations
- Cannot delete repository or change visibility

**Admin**:
- All Maintain permissions
- Delete repository
- Change repository visibility (public/private)
- Manage security settings
- Add/remove collaborators with any permission level

## GitHub Team

Includes all GitHub Free for Organizations features, plus:

| Feature | Benefit |
|---------|---------|
| **Team Support** | Create teams for group permissions |
| **Advanced Security** | Authorized IP ranges |
| **Enterprise Authentication** | SAML, LDAP integration |

### Teams Feature

**Organization Structure**:
```
Organization
├── Engineering Team
│   ├── Backend Team (nested)
│   └── Frontend Team (nested)
├── Design Team
└── Marketing Team
```

**Permission Assignment**:
- Assign teams to repositories
- Teams inherit permissions
- Nested teams inherit parent team permissions
- Individual overrides still possible

```bash
# Example: Team permissions
Backend Team → Write access → engineering-api repo
Frontend Team → Write access → engineering-ui repo
Engineering Team → Read access → all repos
```

## GitHub Enterprise Cloud

Designed for large enterprises with advanced needs:

### Enhanced Security

| Feature | Description |
|---------|-------------|
| **Single Sign-On (SSO)** | SAML integration with corporate identity |
| **Audit Logs** | Comprehensive activity tracking |
| **Compliance Controls** | Meet regulatory requirements |
| **IP Allowlisting** | Restrict access by IP address |

### Advanced Permission Features

**Branch-Level Permissions**:
```
main branch:
  - Require PR reviews: 2
  - Dismiss stale reviews: Yes
  - Restrict push access: Core Team only
  
develop branch:
  - Require PR reviews: 1
  - Allow force pushes: No
  - Restrict push access: Engineering Team
```

**Custom Roles**:
- Create roles beyond built-in options
- Fine-grained permission control
- Tailored to organization needs

Example custom roles:
- "Release Manager" - Can create releases but not delete repos
- "Documentation Writer" - Can edit docs but not merge code
- "Security Auditor" - Read-only access to all repos + security settings

## Permission Strategies

### Small Team (2-5 developers)
```
Strategy: Simple permissions
- All team members: Write access
- Use branch protection on main
- No need for teams or complex roles
```

### Medium Organization (20-50 people)
```
Strategy: Team-based permissions
- Create teams: Engineering, Design, DevOps
- Base permission: Read for all
- Team-specific write access to relevant repos
- Owners: 2-3 trusted leaders
```

### Large Enterprise (100+ people)
```
Strategy: Hierarchical permissions with custom roles
- Nested teams: Engineering → Backend → API Team
- Custom roles: "Deployment Manager", "Security Reviewer"
- Branch-level permissions on critical repos
- Audit logs for compliance
- SSO integration with corporate identity
```

## Best Practices

### Security Principles
1. **Least Privilege**: Give minimum necessary access
2. **Regular Audits**: Review permissions quarterly
3. **Remove Access**: Promptly remove access when people leave
4. **Use Teams**: Manage groups, not individuals
5. **Protected Branches**: Always protect main/production branches

### Collaboration Balance

| Aspect | Too Restrictive | Balanced | Too Open |
|--------|-----------------|----------|----------|
| **Access** | Slows work | Enable contribution | Security risk |
| **Branch Protection** | Blocks progress | Quality gates | No review process |
| **Admin Rights** | One person bottleneck | 2-3 trusted admins | Too many cooks |

### Audit Checklist

- [ ] Review organization members quarterly
- [ ] Check outside collaborator list
- [ ] Verify team memberships current
- [ ] Audit admin/owner access
- [ ] Review branch protection rules
- [ ] Check webhook and integration permissions
- [ ] Validate SSO/2FA enforcement (if applicable)

## Common Scenarios

### Onboarding New Developer

```bash
# Add to organization
# Assign to appropriate team(s)
# Team membership grants repository access automatically

Organization → Engineering Team → John Doe
Result: John gets Write access to engineering repos
```

### External Contractor

```bash
# Add as outside collaborator (not organization member)
# Give access to specific repository only
# Set expiration date for access

Add user@external.com as Collaborator
Repository: client-project
Permission: Write
Expires: 2024-12-31
```

### Departing Team Member

```bash
# Remove from organization
# All access automatically revoked
# Transfer ownership of their repos if needed

Remove from: Engineering Team
Transfer repo ownership: their-project → team-lead
```

## Critical Notes
- 🎯 Choose account type based on team size and needs
- 💡 Use teams to manage permissions at scale, not individual assignments
- ⚠️ Personal accounts cannot give read-only access to private repos
- 📊 GitHub Enterprise provides branch-level and custom role permissions
- 🔄 Regular permission audits essential for security
- ✨ Base permissions + team permissions + individual overrides = flexible model

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/12-configure-source-control-repository-permissions-use-github)
