# Knowledge Check

**Duration**: 5 minutes

Test your understanding of identity management systems, authentication methods, permission models, and workload identities for secure DevOps workflows.

---

## Assessment Questions

### Question 1: GitHub Authentication Method Selection

**Scenario**: Your CI/CD pipeline needs to authenticate GitHub API requests to create issues, update pull requests, and publish packages. The pipeline runs in GitHub Actions workflows. Which authentication method should you use?

**A)** Personal Access Token (PAT) stored in GitHub Secrets  
**B)** GITHUB_TOKEN (automatically generated for workflows)  
**C)** GitHub App with fine-grained permissions  
**D)** OAuth App with user delegation  

<details>
<summary><strong>✅ Answer</strong></summary>

**Correct Answer: B) GITHUB_TOKEN (automatically generated for workflows)**

**Explanation**:
- **GITHUB_TOKEN** is automatically provisioned for each GitHub Actions workflow run with repository-scoped permissions
- **Zero credential management**: No manual token creation, storage, or rotation required
- **Secure**: Ephemeral tokens expire after workflow completion (single-run scope)
- **Configurable permissions**: Workflow-level or job-level permission grants
- **Built-in**: Available in `secrets.GITHUB_TOKEN` context without user action

**Why not other options?**:
- **A) Personal Access Token**: Requires manual creation, secure storage, rotation every 30-90 days. Suitable for external automation (Azure Pipelines), not GitHub Actions
- **C) GitHub App**: Excellent for programmatic API access with fine-grained permissions, but adds complexity for workflow automation where GITHUB_TOKEN suffices
- **D) OAuth App**: Designed for third-party user delegation (SaaS integrations), not workflow automation

**Best Practice**: Use GITHUB_TOKEN for GitHub Actions workflows interacting with repository resources (issues, PRs, packages). Reserve PATs/GitHub Apps for cross-repository or organization-wide automation.

**Example**:
```yaml
permissions:
  issues: write
  pull-requests: write
  packages: write

steps:
  - uses: actions/github-script@v6
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      script: |
        await github.rest.issues.create({
          owner: context.repo.owner,
          repo: context.repo.repo,
          title: 'Automated Issue',
          body: 'Created by workflow'
        });
```

**Reference**: Unit 2 - GitHub Authentication Methods
</details>

---

### Question 2: GitHub Organization Roles

**Scenario**: Your organization has 500 developers and needs a dedicated team to monitor security alerts (Dependabot, code scanning, secret scanning) across all repositories. The team should have read access to all private repositories to investigate vulnerabilities but should NOT be able to modify code or change repository settings. Which role should you assign?

**A)** Organization Owner  
**B)** Security Manager (Public Beta)  
**C)** Organization Moderator  
**D)** Organization Member with read access to all repositories  

<details>
<summary><strong>✅ Answer</strong></summary>

**Correct Answer: B) Security Manager (Public Beta)**

**Explanation**:
- **Security Manager** role provides:
  - **Universal read access**: View all repositories (public + private) without explicit grants
  - **Security alert visibility**: Access Dependabot, code scanning, secret scanning alerts across all repos
  - **Security configuration authority**: Enable/disable security features organization-wide
  - **Read-only code access**: Cannot push commits, merge PRs, or modify repository settings
  - **Perfect fit**: Dedicated security teams needing comprehensive visibility without write permissions

**Why not other options?**:
- **A) Organization Owner**: Excessive permissions (full administrative control, can delete repositories, manage billing). Violates least-privilege principle
- **C) Organization Moderator**: Community governance role (block users, hide comments, lock conversations). No security alert access or universal read permissions
- **D) Organization Member with read access**: Requires explicit repository-by-repository permission grants (500 repos = high management overhead). No centralized security alert visibility

**Trade-off**: Security Manager grants **universal read access to all private code**. Only assign to trusted security personnel with legitimate need for organization-wide visibility.

**Alternative**: If universal read access is unacceptable, create custom team with explicit read permissions to security-sensitive repositories + manual security alert sharing.

**Reference**: Unit 3 - GitHub Permissions (Security Manager)
</details>

---

### Question 3: Azure DevOps Access Control

**Scenario**: A new developer joins your Azure DevOps project. They need to clone repositories, push commits, create work items, run build pipelines, and publish packages. They should NOT be able to delete repositories, change project settings, or manage team permissions. What is the appropriate configuration?

**A)** Add to [Project] Contributors group + Stakeholder access level  
**B)** Add to [Project] Contributors group + Basic access level  
**C)** Add to [Project] Project Administrators group + Basic access level  
**D)** Add to [Project] Readers group + Basic access level  

<details>
<summary><strong>✅ Answer</strong></summary>

**Correct Answer: B) Add to [Project] Contributors group + Basic access level**

**Explanation**:

**Contributors Group Permissions**:
- ✅ Read/write repositories (clone, push, create branches, PRs)
- ✅ Manage work items (create user stories, bugs, tasks, update states)
- ✅ Run build/release pipelines (queue builds, trigger releases)
- ✅ Publish artifacts (upload packages to Azure Artifacts)
- ✅ Edit wiki (documentation collaboration)
- ❌ Cannot delete repositories, change project settings, manage permissions

**Basic Access Level**:
- ✅ Full repository access (code browsing, Git operations)
- ✅ Build/release pipelines (create, edit, execute)
- ✅ Azure Artifacts (publish/consume packages)
- ✅ Wiki (create/edit documentation)
- ❌ Cannot access Test Plans (requires Basic + Test Plans license)

**Result**: **Contributors + Basic** = Standard developer access (read/write across all project features without administrative control)

**Why not other options?**:
- **A) Stakeholder access level**: Free tier with limited permissions. **Cannot access repositories** (code viewing blocked). Suitable for business stakeholders, not developers
- **C) Project Administrators**: Excessive permissions (delete repositories, change settings, manage teams). Violates least-privilege principle
- **D) Readers group**: Read-only access (cannot push commits, cannot create work items, cannot run pipelines). Suitable for auditors, not developers

**Best Practice**: **Contributors + Basic** is the **default configuration for all team members** (developers, QA engineers, analysts) enabling immediate productivity.

**Reference**: Unit 4 - Azure DevOps Permissions (Contributors Group)
</details>

---

### Question 4: Entra ID Integration Benefits

**Scenario**: Your organization uses Microsoft Entra ID (formerly Azure AD) for centralized identity management. You have 50+ Azure DevOps projects with 200+ developers. Currently, project administrators manually add/remove users from Azure DevOps groups when employees join, change roles, or leave the company. What Azure DevOps feature can automate this process?

**A)** Azure DevOps Project-Scoped Users  
**B)** Azure DevOps Group Rules (Entra ID group synchronization)  
**C)** Azure DevOps Service Principals  
**D)** Azure DevOps Personal Access Tokens  

<details>
<summary><strong>✅ Answer</strong></summary>

**Correct Answer: B) Azure DevOps Group Rules (Entra ID group synchronization)**

**Explanation**:

**Group Rules Workflow**:
```
1. Configure Group Rule:
   Organization Settings → Permissions → Group Rules → Add Group Rule
   
2. Map Entra ID Group → Azure DevOps Group:
   Entra ID Group: "Engineering-Backend" (25 developers)
   Azure DevOps Group: [API Project] Contributors
   Access Level: Basic (auto-assign)
   
3. Automated Provisioning:
   - User added to "Engineering-Backend" in Entra ID
     → Automatically added to [API Project] Contributors with Basic license
   - User removed from "Engineering-Backend" in Entra ID
     → Automatically removed from [API Project] Contributors (license released)
   - User changes role (moved to "Engineering-Frontend" group)
     → Automatically updated in Azure DevOps (removed from API Project, added to Web Project)
```

**Benefits**:
- ✅ **Zero manual Azure DevOps management**: HR system → Entra ID → Azure DevOps (automated chain)
- ✅ **Real-time synchronization**: Membership changes propagate immediately via webhooks
- ✅ **Automatic deprovisioning**: Terminated employees lose access instantly
- ✅ **Audit compliance**: All changes logged in Entra ID + Azure DevOps audit logs
- ✅ **Centralized source of truth**: Entra ID as single identity management system
- ✅ **80-90% reduction in access management overhead**

**Why not other options?**:
- **A) Project-Scoped Users**: Limits user visibility to specific projects (security feature for contractors), does not automate provisioning
- **C) Service Principals**: Workload identities for applications/pipelines, not human users
- **D) Personal Access Tokens**: Authentication mechanism for API access, not user lifecycle management

**Implementation Example**:
```
Entra ID Security Groups:
├── "Engineering-Backend" → [API Project] Contributors
├── "Engineering-Frontend" → [Web Project] Contributors
├── "Engineering-DevOps" → Collection Administrators
└── "Engineering-QA" → [All Projects] Contributors + Basic + Test Plans

Result: 200 developers automatically provisioned across 50 projects with zero manual Azure DevOps group management
```

**Reference**: Unit 4 - Best Practice #2 (Automate Entra ID Group Associations)
</details>

---

### Question 5: Workload Identity Types

**Scenario**: Your Azure Pipelines need to deploy resources to 5 different Azure subscriptions (dev, test, staging, prod-us, prod-eu). Each subscription requires Contributor role permissions. You want to minimize credential management overhead while maintaining audit trail and security. Which workload identity approach should you use?

**A)** Create 5 separate service principals (1 per subscription) with client secrets  
**B)** Create 1 service principal with Owner role on all 5 subscriptions  
**C)** Create 1 user-assigned managed identity with Contributor role on all 5 subscriptions + workload identity federation  
**D)** Use 5 different Personal Access Tokens stored in Azure Key Vault  

<details>
<summary><strong>✅ Answer</strong></summary>

**Correct Answer: C) Create 1 user-assigned managed identity with Contributor role on all 5 subscriptions + workload identity federation**

**Explanation**:

**User-Assigned Managed Identity + Workload Identity Federation**:
```
1. Create Managed Identity:
   az identity create --name mi-azdo-multi-subscription

2. Grant Permissions (5 subscriptions):
   az role assignment create --assignee <principal-id> --role Contributor --scope /subscriptions/dev-sub-id
   az role assignment create --assignee <principal-id> --role Contributor --scope /subscriptions/test-sub-id
   az role assignment create --assignee <principal-id> --role Contributor --scope /subscriptions/staging-sub-id
   az role assignment create --assignee <principal-id> --role Contributor --scope /subscriptions/prod-us-sub-id
   az role assignment create --assignee <principal-id> --role Contributor --scope /subscriptions/prod-eu-sub-id

3. Configure Federated Credential (Azure DevOps trust):
   Managed Identity → Federated credentials → Add credential
   Subject: sc://organization/project/service-connection-name

4. Azure DevOps Service Connection:
   Authentication: Workload Identity Federation (automatic)
   Managed Identity: mi-azdo-multi-subscription

Result:
   - 1 managed identity (shared across 5 subscriptions)
   - 0 secrets (passwordless authentication via OIDC)
   - 0 rotation overhead (automatic token management)
   - Full audit trail (token claims include pipeline, project, commit)
```

**Benefits**:
- ✅ **Zero secret management**: No client secrets to store, rotate, or leak
- ✅ **Centralized identity**: Single managed identity with permissions across 5 subscriptions
- ✅ **Workload identity federation**: OpenID Connect trust eliminates credential storage
- ✅ **Short-lived tokens**: 5-15 minute token lifetime (automatic expiration)
- ✅ **Audit trail**: Token claims include Azure DevOps organization, project, pipeline details

**Why not other options?**:
- **A) 5 separate service principals**: High operational overhead (5 client secrets to rotate every 12 months = 60 rotation operations annually). Credential leakage risk
- **B) 1 service principal with Owner role**: Excessive permissions (Owner can grant permissions, violates least-privilege). Still requires client secret management
- **D) 5 different PATs**: Personal Access Tokens are GitHub authentication (not Azure). PATs unsuitable for Azure Pipelines → Azure resource authentication

**Trade-off**: Single managed identity across 5 subscriptions = single point of compromise. If acceptable, proceed. If unacceptable, create per-environment managed identities (dev-mi, test-mi, prod-mi) for blast radius containment.

**Reference**: Unit 5 - Workload Identities; Unit 6 - Workload Identity Federation
</details>

---

### Question 6: Managed Identity Types

**Scenario**: You have 10 Azure Virtual Machines running a distributed application. All VMs need to access the same Azure Storage Account to read/write blob data. You want to avoid storing credentials in VM configuration and minimize identity management overhead. Should you use system-assigned or user-assigned managed identities?

**A)** System-assigned managed identity (enable on each VM)  
**B)** User-assigned managed identity (create 1, assign to all 10 VMs)  
**C)** Service principal with client secret (store in Azure Key Vault)  
**D)** Azure AD B2C identity  

<details>
<summary><strong>✅ Answer</strong></summary>

**Correct Answer: B) User-assigned managed identity (create 1, assign to all 10 VMs)**

**Explanation**:

**User-Assigned Managed Identity (Shared Identity Pattern)**:
```
1. Create User-Assigned Managed Identity:
   az identity create --resource-group rg-prod --name mi-app-cluster

2. Grant Storage Permissions:
   az role assignment create \
     --assignee <principal-id> \
     --role "Storage Blob Data Contributor" \
     --scope /subscriptions/.../resourceGroups/rg-prod/providers/Microsoft.Storage/storageAccounts/stproddata

3. Assign to All 10 VMs:
   az vm identity assign --resource-group rg-prod --name vm-app-01 --identities <identity-resource-id>
   az vm identity assign --resource-group rg-prod --name vm-app-02 --identities <identity-resource-id>
   ...
   az vm identity assign --resource-group rg-prod --name vm-app-10 --identities <identity-resource-id>

4. Application Code (Same on All VMs):
   from azure.identity import ManagedIdentityCredential
   from azure.storage.blob import BlobServiceClient
   
   credential = ManagedIdentityCredential(client_id="<user-assigned-mi-client-id>")
   blob_service = BlobServiceClient(account_url="https://stproddata.blob.core.windows.net", credential=credential)

Result:
   - 1 managed identity shared across 10 VMs
   - 1 role assignment (Storage Blob Data Contributor)
   - Centralized permission management (update 1 identity, affects all 10 VMs)
   - Zero credential storage (passwordless authentication)
```

**Benefits**:
- ✅ **Centralized management**: 1 identity with permissions, not 10 separate identities
- ✅ **Simplified role assignments**: Grant permissions once, effective for all VMs
- ✅ **Identity persistence**: Managed identity survives VM deletion/recreation (useful for VM scale sets)
- ✅ **Code simplicity**: Same application code across all VMs (consistent client ID)

**Why not other options?**:
- **A) System-assigned managed identity**: Creates 10 separate identities (1 per VM). Requires 10 role assignments. High management overhead for shared access scenarios. Suitable for single-resource authentication
- **C) Service principal with client secret**: Requires storing secret in Key Vault, retrieving in application, rotating every 12 months. Managed identity eliminates all credential management
- **D) Azure AD B2C identity**: Consumer identity solution for web/mobile applications (social logins, self-service). Not applicable for VM-to-Storage authentication

**When to Use System-Assigned Instead**:
```
✅ Single VM needs unique permissions (not shared with others)
   Example: VM-01 reads storage, VM-02 writes storage → Different permissions required

✅ Identity lifecycle matches VM lifecycle (delete VM = delete identity)
   Example: Development VMs with short lifespan

✅ Simplified setup (no separate identity object to create)
   Example: Single App Service accessing Key Vault (1:1 relationship)
```

**Reference**: Unit 6 - Managed Identity Types Comparison
</details>

---

### Question 7: Workload Identity Federation Benefits

**Scenario**: Your GitHub Actions workflows deploy infrastructure to Azure using a service principal with client secret. The client secret is stored in GitHub repository secrets and must be rotated every 12 months. You want to eliminate secret management overhead while maintaining secure authentication. What should you implement?

**A)** Store client secret in Azure Key Vault instead of GitHub Secrets  
**B)** Increase client secret expiration to 24 months to reduce rotation frequency  
**C)** Implement workload identity federation with user-assigned managed identity  
**D)** Use Personal Access Token (PAT) instead of service principal  

<details>
<summary><strong>✅ Answer</strong></summary>

**Correct Answer: C) Implement workload identity federation with user-assigned managed identity**

**Explanation**:

**Workload Identity Federation Eliminates Secrets**:
```
Traditional Service Principal (Before):
├── GitHub Secrets:
│   ├── AZURE_CLIENT_ID: 12345678-1234-1234-1234-123456789abc
│   ├── AZURE_CLIENT_SECRET: "abc123def456..." ⚠️ Secret storage required
│   ├── AZURE_TENANT_ID: tenant123-4567-8901-2345-678901234567
│   └── AZURE_SUBSCRIPTION_ID: sub456-7890-1234-5678-901234567890
├── Rotation Schedule: Every 12 months
│   ├── Generate new secret
│   ├── Update AZURE_CLIENT_SECRET in GitHub Secrets
│   ├── Test workflows
│   └── Delete old secret
└── Security Risk: Secret leakage (logs, console output, commit history)

Workload Identity Federation (After):
├── GitHub Secrets: None required for Azure authentication ✅
├── GitHub Workflow Permissions: id-token: write (OIDC token request)
├── Managed Identity: mi-github-actions-prod
│   ├── Federated Credential: repo:org/repo:environment:production
│   └── Role Assignment: Contributor on subscription
├── Authentication Flow:
│   1. GitHub Actions requests OIDC token from GitHub
│   2. GitHub issues token with claims (repo, workflow, commit)
│   3. azure/login@v1 presents token to Azure
│   4. Azure validates token with GitHub's public keys
│   5. Azure issues access token for managed identity
│   6. Workflow deploys Azure resources
└── Rotation Schedule: None (tokens auto-expire after 5-15 minutes)
```

**Benefits**:
- ✅ **Zero secrets**: No AZURE_CLIENT_SECRET to store, rotate, or leak
- ✅ **Automatic token management**: Short-lived tokens (5-15 minutes) auto-expire
- ✅ **OpenID Connect security**: Cryptographically signed tokens, GitHub's private keys
- ✅ **Granular trust**: Federated credential scoped to specific repo/branch/environment
- ✅ **Audit trail**: Token claims include repository, workflow, commit SHA

**Why not other options?**:
- **A) Store in Azure Key Vault**: Moves secret location (GitHub → Key Vault), doesn't eliminate secret. Still requires retrieval, rotation, and leakage prevention
- **B) Increase expiration to 24 months**: Increases security risk window. Longer-lived secrets = greater impact if compromised. Microsoft recommends 12 months maximum
- **D) Personal Access Token**: PATs are for GitHub API authentication, not Azure resource authentication. Unsuitable for Azure deployments

**Implementation Effort**:
```
1. Create user-assigned managed identity (5 minutes)
2. Configure federated credential (2 minutes)
3. Grant Azure permissions (2 minutes)
4. Update GitHub workflow (5 minutes):
   - Add permissions: id-token: write
   - Update azure/login@v1 with client-id (remove client-secret)
   - Test workflow
5. Delete AZURE_CLIENT_SECRET from GitHub Secrets (1 minute)

Total: ~15 minutes one-time setup
Result: Eliminate 12-month recurring secret rotation forever
```

**Reference**: Unit 6 - Workload Identity Federation
</details>

---

### Question 8: Permission Precedence

**Scenario**: In Azure DevOps, a user is a member of the Project Contributors group (which has "Contribute" permission set to "Allow" for a repository). You explicitly set "Contribute" permission to "Deny" for this user on the same repository. What is the effective permission?

**A)** Allow (group permission takes precedence)  
**B)** Deny (explicit user permission takes precedence)  
**C)** Not Set (permissions cancel each other)  
**D)** Inherited from organization-level permissions  

<details>
<summary><strong>✅ Answer</strong></summary>

**Correct Answer: B) Deny (explicit user permission takes precedence)**

**Explanation**:

**Azure DevOps Permission Precedence Rules**:
```
Resolution Order (Highest to Lowest Priority):
1. Explicit Deny (object-level) ← STRONGEST RULE
   ↓ Overrides all Allow rules
2. Explicit Allow (object-level)
   ↓ If no Deny
3. Inherited Deny (parent scope)
   ↓ If no explicit rules
4. Inherited Allow (parent scope)
   ↓ If no explicit rules
5. Not Set (no permission granted)
```

**Scenario Analysis**:
```
User: developer@contoso.com
Repository: api-service

Permission Resolution:
├── Group Membership: Project Contributors
│   └── Permission: "Contribute" → Allow (group-level)
│
├── Explicit User Permission: developer@contoso.com
│   └── Permission: "Contribute" → Deny (user-level)
│
└── Effective Permission: DENY
    Reason: Explicit Deny overrides group Allow
```

**Critical Rule**: **Deny ALWAYS wins** (overrides all Allow rules, whether explicit or inherited)

**Use Cases for Explicit Deny**:
```
1. Prevent Specific User from Critical Operation:
   - All Project Admins can delete repositories (Allow via group)
   - Except junior-admin@contoso.com (Explicit Deny on "Delete repository")
   - Result: Junior admin cannot delete repos despite Project Admin membership

2. Protect Critical Repositories:
   - All Contributors can force push to branches (Allow via group)
   - Except "production-api" repository (Explicit Deny on "Force push")
   - Result: No one can force push to production-api (not even Project Admins)

3. Temporary Access Revocation:
   - User on leave but remains in Contributors group
   - Explicit Deny on all repositories
   - Result: User cannot access repos without removing group membership
```

**Best Practice**: Use Deny sparingly. Explicit Deny creates complexity and troubleshooting difficulty. Prefer removing users from groups or adjusting group permissions.

**Why not other options?**:
- **A) Allow (group permission)**: Incorrect. Deny overrides all Allow rules (explicit or inherited)
- **C) Not Set (permissions cancel)**: Incorrect. Permissions don't "cancel" - precedence rules apply (Deny wins)
- **D) Inherited from organization**: Incorrect. Explicit user-level Deny has higher priority than inherited organization permissions

**Reference**: Unit 4 - Permission Management (Tier 2)
</details>

---

### Question 9: GitHub Outside Collaborator vs Organization Member

**Scenario**: You're hiring a contractor for 6 months to build a feature in a single repository. The contractor should NOT see your organization's member list, other private repositories, or organization structure. They need read/write access to one repository only. Cost optimization is important. What should you assign?

**A)** Organization Member with read/write permissions to specific repository  
**B)** Organization Owner (limited to 6 months)  
**C)** Outside Collaborator with Write permission to repository  
**D)** Organization Moderator  

<details>
<summary><strong>✅ Answer</strong></summary>

**Correct Answer: C) Outside Collaborator with Write permission to repository**

**Explanation**:

**Outside Collaborator Characteristics**:
```
Outside Collaborator: contractor@vendor.com
├── Repository Access: "project-feature-x" (Write permission)
│   ├── Clone, push, pull
│   ├── Create issues, pull requests
│   ├── Review pull requests
│   └── Manage issue labels, milestones
│
├── Organization Visibility: Minimal ✅
│   ├── Cannot see organization member list
│   ├── Cannot view other private repositories
│   ├── Cannot see organization structure
│   └── No organization-wide access
│
├── Duration: 6-month contract
│   └── Revoke access: Remove collaborator (easy cleanup)
│
└── Cost: FREE (doesn't count toward license) ✅
```

**Benefits**:
- ✅ **Minimal organization visibility**: Contractor sees only granted repository, not org structure
- ✅ **Repository-scoped access**: Explicit per-repo grant (granular control)
- ✅ **Easy revocation**: Remove collaborator when contract ends (no org-wide impact)
- ✅ **Cost-effective**: Free seat (doesn't consume paid license)
- ✅ **Security isolation**: Cannot discover other projects or members

**Why not other options?**:
- **A) Organization Member**: Grants organization-wide visibility (member list, private repo names, org structure). Costs $4/user/month (GitHub Team) or $21/user/month (GitHub Enterprise). Excessive access for single-repo contractor
- **B) Organization Owner**: Catastrophic over-privileging (full administrative control, can delete org, manage billing, remove members). Violates least-privilege principle
- **D) Organization Moderator**: Community governance role (block users, hide comments, lock conversations). No repository write access for feature development

**Contractor Workflow**:
```
1. Add Outside Collaborator:
   Repository → Settings → Collaborators → Add people
   Enter: contractor@vendor.com
   Permission: Write

2. Contractor Receives Email:
   "You've been invited to collaborate on organization/project-feature-x"
   Accept invitation

3. Contractor Access:
   ✅ Clone repository: git clone git@github.com:organization/project-feature-x.git
   ✅ Push commits: git push origin feature-branch
   ✅ Create PR: Submit pull request for review
   ❌ View other repos: Cannot see organization/other-private-repo

4. Contract Ends (Month 6):
   Repository → Settings → Collaborators → Remove contractor@vendor.com
   Result: Contractor loses all access immediately
```

**Organization Member vs Outside Collaborator**:

| Feature | Outside Collaborator | Organization Member |
|---------|---------------------|---------------------|
| **Org visibility** | Minimal (no member list) | Full (see all members) |
| **Repo access** | Explicit per-repo grants | Team-based inheritance |
| **Private repo names** | Cannot see other repos | Can see all repo names |
| **Cost** | FREE | Paid ($4-21/month) |
| **Use case** | Contractors, temporary access | Employees, long-term contributors |

**Reference**: Unit 3 - GitHub Permissions (Outside Collaborator)
</details>

---

### Question 10: Managed Identity Authentication Flow

**Scenario**: An Azure Function App with system-assigned managed identity needs to retrieve a secret from Azure Key Vault. What is the correct authentication flow?

**A)** Function App retrieves client secret from environment variables → Authenticates to Entra ID → Gets access token → Accesses Key Vault  
**B)** Function App requests token from Azure Instance Metadata Service (IMDS) → Receives access token → Accesses Key Vault  
**C)** Function App uses certificate stored in App Settings → Authenticates to Entra ID → Gets access token → Accesses Key Vault  
**D)** Function App uses connection string with username/password → Authenticates directly to Key Vault  

<details>
<summary><strong>✅ Answer</strong></summary>

**Correct Answer: B) Function App requests token from Azure Instance Metadata Service (IMDS) → Receives access token → Accesses Key Vault**

**Explanation**:

**Managed Identity Authentication Flow**:
```
┌─────────────────────────────────────────────────────────────┐
│  1. Application Code (Azure Function)                        │
│     from azure.identity import DefaultAzureCredential        │
│     credential = DefaultAzureCredential()                    │
│     secret_client = SecretClient(                            │
│         vault_url="https://kv-prod.vault.azure.net",         │
│         credential=credential                                │
│     )                                                        │
└────────────────────┬────────────────────────────────────────┘
                     │ Request access token
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  2. Azure Instance Metadata Service (IMDS)                   │
│     Endpoint: http://169.254.169.254/metadata/identity/...   │
│     (Internal Azure endpoint, not internet-accessible)       │
│                                                              │
│     Request:                                                 │
│     GET http://169.254.169.254/metadata/identity/oauth2/token?resource=https://vault.azure.net │
│     Metadata: true                                           │
└────────────────────┬────────────────────────────────────────┘
                     │ IMDS verifies caller is authorized Azure resource
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  3. Microsoft Entra ID (Identity Provider)                   │
│     IMDS forwards request to Entra ID with managed identity  │
│     principal ID                                             │
│                                                              │
│     Entra ID validates:                                      │
│     - Request originates from authorized Azure resource      │
│     - Managed identity exists and enabled                    │
│     - Principal ID matches Function App's managed identity   │
└────────────────────┬────────────────────────────────────────┘
                     │ Issue access token
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  4. Access Token Returned to Function App                    │
│     {                                                        │
│       "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",         │
│       "expires_on": "1642345678",  // Unix timestamp         │
│       "resource": "https://vault.azure.net",                │
│       "token_type": "Bearer"                                 │
│     }                                                        │
│                                                              │
│     Token Lifetime: 1-24 hours (auto-refreshed by SDK)      │
└────────────────────┬────────────────────────────────────────┘
                     │ Present token to Key Vault
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  5. Azure Key Vault Access                                   │
│     GET https://kv-prod.vault.azure.net/secrets/DatabasePassword │
│     Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...          │
│                                                              │
│     Key Vault validates:                                     │
│     - Token signature (issued by Entra ID)                   │
│     - Token expiration (not expired)                         │
│     - Token audience (https://vault.azure.net)               │
│     - Managed identity has "Key Vault Secrets User" role     │
└────────────────────┬────────────────────────────────────────┘
                     │ Return secret value
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  6. Secret Returned to Function App                          │
│     {                                                        │
│       "value": "P@ssw0rd123!",                               │
│       "id": "https://kv-prod.vault.azure.net/.../DatabasePassword", │
│       "attributes": { ... }                                  │
│     }                                                        │
│                                                              │
│     Function App uses secret (database connection)           │
└─────────────────────────────────────────────────────────────┘
```

**Key Characteristics**:
- ✅ **No credentials in code**: No client secrets, certificates, or passwords
- ✅ **IMDS endpoint**: Internal Azure metadata service (169.254.169.254) not accessible from internet
- ✅ **Automatic token refresh**: Azure SDK handles token expiration and renewal
- ✅ **Short-lived tokens**: 1-24 hour lifetime (automatic rotation)
- ✅ **Platform-managed**: Azure handles entire authentication flow

**Why not other options?**:
- **A) Client secret from environment variables**: Service principal pattern (not managed identity). Requires credential storage and rotation. Managed identities eliminate all secrets
- **C) Certificate stored in App Settings**: Service principal with certificate authentication (not managed identity). Still requires certificate management (creation, storage, rotation)
- **D) Connection string with username/password**: Legacy authentication (not managed identity). Credentials stored in configuration. High security risk, rotation overhead

**Code Example** (Python):
```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# DefaultAzureCredential automatically uses managed identity when running in Azure
credential = DefaultAzureCredential()

# Access Key Vault (passwordless)
secret_client = SecretClient(
    vault_url="https://kv-prod.vault.azure.net",
    credential=credential
)

# Retrieve secret (automatic token handling)
secret = secret_client.get_secret("DatabasePassword")
connection_string = f"Server=sql.database.windows.net;Database=prod;Password={secret.value}"
```

**Reference**: Unit 6 - Managed Identity Authentication Flow
</details>

---

## Score Interpretation

**9-10 Correct**: 🎯 Excellent mastery of identity management systems! You understand authentication methods, permission models, workload identities, and managed identity patterns for secure DevOps workflows.

**7-8 Correct**: ✅ Good grasp of identity concepts! Review specific topics: workload identity federation, permission precedence rules, or managed identity types for comprehensive understanding.

**5-6 Correct**: ⚠️ Fair understanding, needs reinforcement! Revisit units on GitHub/Azure DevOps permissions, service principals vs managed identities, and authentication method selection criteria.

**<5 Correct**: ❌ Review all units! Focus on core concepts: GitHub authentication methods, Azure DevOps 3-tier access control, workload identity types, and managed identity benefits over service principals.

---

## Key Concepts Review

### Authentication Method Decision Tree
```
Human Users:
├── Personal GitHub: Username + 2FA (TOTP/Security Key)
└── Enterprise: SAML SSO + MFA Enforcement

Git Operations:
├── Developer Workstations: SSH Keys (Ed25519)
└── CI/CD Systems: GITHUB_TOKEN (Actions) or Deploy Keys

API Automation:
├── Personal Scripts: Personal Access Token (PAT)
├── Organization Services: GitHub App (fine-grained)
└── Third-Party: OAuth App (user delegation)

Azure Resource Authentication:
├── Modern (Recommended): Managed Identity (passwordless)
├── CI/CD Pipelines: Workload Identity Federation (OIDC)
└── Legacy: Service Principal (client secrets)
```

### Permission Model Comparison
```
GitHub Organizations:
├── 7 Roles: Owner, Member, Moderator, Billing Manager, Security Manager, App Manager, Outside Collaborator
├── Model: Role-based (flat hierarchy)
└── Integration: SAML SSO + Team Sync

Azure DevOps:
├── 3-Tier Model: Membership + Permissions + Access Levels
├── Groups: Collection/Project-scoped (nested hierarchy)
└── Integration: Native Entra ID Group Rules (automated sync)
```

### Workload Identity Comparison
```
Service Principal:
├── Credentials: Client Secret (manual rotation every 12 months)
├── Use Case: Azure Pipelines (traditional), non-Azure automation
└── Overhead: High (secret storage, rotation, leakage risk)

Managed Identity:
├── Credentials: None (passwordless, platform-managed)
├── Use Case: Azure resource-to-resource authentication
└── Overhead: Zero (automatic token management)

Workload Identity Federation:
├── Credentials: None (OIDC trust, short-lived tokens)
├── Use Case: GitHub Actions, Azure Pipelines (modern)
└── Overhead: Minimal (one-time federated credential setup)
```

---

**Next**: Review key takeaways and additional resources in the summary →

---

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-identity-management-systems/7-knowledge-check)
