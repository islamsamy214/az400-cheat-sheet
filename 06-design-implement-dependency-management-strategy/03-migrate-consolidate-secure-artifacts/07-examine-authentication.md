# Examine Authentication

## Key Concepts
- Authentication verifies identity of users and services accessing Azure Artifacts
- Multiple authentication mechanisms available for different scenarios
- Understanding authentication is critical for secure and seamless access

## Azure DevOps User Authentication

### Transparent Authentication
Azure DevOps users authenticate against Microsoft Entra ID (formerly Azure Active Directory).

**Authorization versus Authentication:**
- **Authentication**: Verifies user identity through Microsoft Entra ID
- **Authorization**: Roles for user, based on identity or team and group membership

**User Experience:**
After successful authentication, users won't have to provide any credentials to Azure Artifacts itself. When access is allowed, user can navigate to Azure Artifacts section without additional authentication prompts.

## Azure Pipelines Authentication

### Build Identity Authentication
Authentication from Azure Pipelines to Azure Artifacts feeds is taken care of transparently.

**Default Build Identity:**
- Project Collection Build Service: Organization-wide build identity with access to feeds
- Project-scoped build identity: Project-specific identity that may need explicit permissions

**Automatic Authorization:**
- No credentials required: Pipelines automatically authenticate using build service identity
- Role-based access: Access granted based on build identity's assigned role
- Seamless integration: No configuration needed for feeds in same organization

## Internal versus External Authentication

### External Feed Authentication
When accessing secured feeds outside Azure Artifacts, such as other package sources, you most likely must provide credentials.

**External Feed Scenarios:**
- Third-party registries: npmjs.com, NuGet.org with authentication
- Private registries: Self-hosted package servers
- Other Azure DevOps organizations: Feeds in different organizations

**Authentication Requirements:**
Each package type has its way of handling credentials. Command-line tooling provides support in authentication process.

### Service Connections for Pipelines
For build tasks in Azure Pipelines, provide credentials via Service connection.

**Service Connection Types:**
- NuGet service connection: For external NuGet feeds
- npm service connection: For external npm registries
- Generic service connection: For other feed types with basic authentication

## Personal Access Tokens (PATs)

### Creating a PAT
1. User settings: Navigate to user settings in Azure DevOps
2. Personal Access Tokens: Select "Personal access tokens" from security section
3. New token: Click "New Token" to create new PAT
4. Configure scopes: Select required scopes for Azure Artifacts
5. Set expiration: Choose expiration date for token
6. Create: Generate token and save it securely

**Required Scopes:**
- **Packaging (Read)**: For consuming packages from feeds
- **Packaging (Read & write)**: For consuming and publishing packages
- **Packaging (Read, write, & manage)**: For full feed management

### Using PATs

**In npm (`.npmrc`):**
```
//pkgs.dev.azure.com/{organization}/_packaging/{feed}/npm/registry/:username={organization}
//pkgs.dev.azure.com/{organization}/_packaging/{feed}/npm/registry/:_password={base64_encoded_PAT}
//pkgs.dev.azure.com/{organization}/_packaging/{feed}/npm/registry/:email=npm@contoso.com
//pkgs.dev.azure.com/{organization}/_packaging/{feed}/npm/:always-auth=true
```

**In NuGet:**
```bash
nuget sources add -Name "MyFeed" -Source "https://pkgs.dev.azure.com/{organization}/_packaging/{feed}/nuget/v3/index.json" -Username {username} -Password {PAT}
```

**In Maven (`settings.xml`):**
```xml
<server>
  <id>azure-feed</id>
  <username>{organization}</username>
  <password>{PAT}</password>
</server>
```

### PAT Best Practices

**Security:**
- Minimal scopes: Grant only scopes needed for task
- Short expiration: Use short expiration periods (30-90 days)
- Rotate regularly: Replace tokens before they expire
- Secure storage: Never commit PATs to source control

**Environment Variables:**
```bash
# Set environment variable
export AZURE_ARTIFACTS_PAT="your_pat_here"

# Use in scripts
echo $AZURE_ARTIFACTS_PAT | base64 | pbcopy
```

## Credential Providers

### Azure Artifacts Credential Provider

**Installation:**
```bash
# Install for .NET/NuGet
iex "& { $(irm https://aka.ms/install-artifacts-credprovider.ps1) }"

# Or using PowerShell
wget -qO- https://aka.ms/install-artifacts-credprovider.ps1 | sh
```

**How It Works:**
- Automatic authentication: Prompts for credentials when accessing authenticated feeds
- Token caching: Stores credentials securely for reuse
- Interactive and non-interactive modes: Supports both user interaction and automated scenarios

**Non-Interactive Authentication:**
```bash
# Set environment variable for non-interactive mode
export VSS_NUGET_EXTERNAL_FEED_ENDPOINTS='{"endpointCredentials": [{"endpoint":"https://pkgs.dev.azure.com/{organization}/_packaging/{feed}/nuget/v3/index.json", "username":"optional", "password":"{PAT}"}]}'
```

### npm Credential Provider

**Using vsts-npm-auth:**
```bash
# Install globally
npm install -g vsts-npm-auth

# Run to authenticate
vsts-npm-auth -config .npmrc
```

**Automatic Authentication:**
The tool updates `.npmrc` file with credentials that are automatically refreshed.

## Service Principals

### Creating a Service Principal
1. Microsoft Entra ID: Navigate to Microsoft Entra ID in Azure portal
2. App registrations: Create new app registration
3. Credentials: Generate client secret or certificate
4. Azure DevOps permissions: Add service principal to Azure DevOps organization
5. Feed permissions: Grant service principal appropriate role on feeds

### Using Service Principals

**In CI/CD Pipelines:**
```yaml
# Azure Pipelines - using service connection
steps:
  - task: NuGetAuthenticate@1
    inputs:
      serviceConnection: "MyServiceConnection"

  - script: |
      dotnet pack
      dotnet nuget push "**/*.nupkg" --source "MyFeed"
    displayName: "Publish package"
```

**Benefits:**
- No user dependency: Not tied to specific user account
- Long-lived: Don't expire like user PATs
- Auditable: Clear separation between user and service accounts

## Authentication for Different Scenarios

### Developer Workstations
**Interactive Authentication:**
- Microsoft Entra ID: Single sign-on through Azure DevOps portal
- Credential providers: Automatic prompts for credentials
- PATs: Manual configuration for command-line tools

### Build and Release Pipelines
**Automated Authentication:**
- Build service identity: Automatic for feeds in same organization
- Service connections: For external feeds and cross-organization access
- Service principals: For production deployments and long-lived automation

### External Tools and Scripts
**Programmatic Authentication:**
- PATs: For scripts and automation
- Service principals: For applications and services
- API tokens: For REST API access

## Troubleshooting Authentication

### Common Issues

**401 Unauthorized:**
- Check credentials: Verify PAT is correct and not expired
- Check scopes: Ensure PAT has appropriate scopes for operation
- Check permissions: Verify user/service has correct role on feed

**403 Forbidden:**
- Check permissions: User is authenticated but doesn't have required role
- Check feed visibility: Ensure user has access to feed

**Credential Caching Issues:**
- Clear credentials: Remove cached credentials and re-authenticate
- Update credential provider: Ensure using latest version

### Diagnostic Commands
```bash
# Test NuGet authentication
nuget sources list

# Test npm authentication
npm whoami --registry=https://pkgs.dev.azure.com/{organization}/_packaging/{feed}/npm/registry/

# Test Azure CLI authentication
az artifacts universal download --organization https://dev.azure.com/{organization} --feed {feed} --name {package} --version {version}
```

## Security Best Practices

### Credential Management
**Never Commit Credentials:**
- Use .gitignore: Exclude credential files from source control
- Environment variables: Store secrets in environment variables
- Secret management: Use Azure Key Vault or similar services

### Token Hygiene
**Regular Rotation:**
- Set expiration: Always set expiration dates on PATs
- Rotate proactively: Replace tokens before they expire
- Revoke unused: Remove tokens that are no longer needed

### Monitoring
**Audit Access:**
- Review logs: Monitor who is accessing feeds and when
- Alert on failures: Set up alerts for repeated authentication failures
- Track PAT usage: Monitor PAT creation and usage patterns

## Authentication Method Comparison

| Method | Use Case | Lifetime | Best For |
|--------|----------|----------|----------|
| **Microsoft Entra ID** | User authentication | Session-based | Interactive users |
| **PATs** | Scripts, tools | 30-365 days | Developer automation |
| **Service Principals** | Applications | Long-lived | CI/CD pipelines |
| **Credential Providers** | Package managers | Token refresh | Developer workstations |
| **Build Service Identity** | Azure Pipelines | Perpetual | Internal builds |

## Critical Notes
- 🎯 Multiple authentication methods available for different scenarios
- 💡 Credential providers simplify authentication for package managers
- ⚠️ PATs should have minimal scopes and short expiration
- 📊 Service principals recommended for production CI/CD
- 🔒 Never commit credentials to source control
- 🔄 Rotate tokens regularly for security
- 🔐 Build service identity provides automatic authentication for Azure Pipelines

[Learn More](https://learn.microsoft.com/en-us/training/modules/migrate-consolidating-secure-artifacts/7-examine-authentication)
