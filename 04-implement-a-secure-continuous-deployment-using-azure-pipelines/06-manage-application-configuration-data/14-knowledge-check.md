# Knowledge Check

**Duration**: 3 minutes

Test your understanding of application configuration management, Azure App Configuration, Key Vault integration, feature flags, and secure deployment patterns covered in this module.

---

## Question 1: Configuration Anti-Patterns

**Scenario**: Your team stores database passwords in `appsettings.json` files committed to Git. During a security audit, this is flagged as a critical vulnerability.

**What is the BEST solution to remediate this issue?**

A) Encrypt the `appsettings.json` file before committing to Git  
B) Store secrets in Azure Key Vault and reference them in the application  
C) Move secrets to environment variables on production servers  
D) Use Azure DevOps Secure Files to store `appsettings.json`

<details>
<summary><strong>Answer</strong></summary>

**Correct Answer: B) Store secrets in Azure Key Vault and reference them in the application**

**Explanation**:
- Azure Key Vault provides centralized, secure secret storage with encryption, access control, and audit logging
- Application retrieves secrets at runtime using managed identities (no hardcoded credentials)
- Secrets never committed to source control
- Supports automatic rotation and versioning

**Why other options are insufficient**:
- **A)** Encrypting files still exposes encryption keys; doesn't solve rotation or audit requirements
- **C)** Environment variables better than files, but lack centralized management, rotation, and audit trails
- **D)** Secure Files suitable for certificates/keys, not application secrets (pipeline-focused, not runtime)

**Implementation**:
```csharp
// .NET - Retrieve secret from Key Vault
var client = new SecretClient(
    new Uri("https://prod-keyvault.vault.azure.net"),
    new DefaultAzureCredential()
);
var password = (await client.GetSecretAsync("DatabasePassword")).Value.Value;
```

**AZ-400 Exam Relevance**: Secure secret management is a core DevOps practice (Exam Objective: Implement secure continuous deployment).
</details>

---

## Question 2: Separation of Concerns

**Scenario**: Your organization has 50 microservices, each with its own configuration file. Updating an API endpoint requires changing 50 files across 50 repositories. This process is error-prone and slow.

**Which pattern BEST addresses this problem?**

A) Centralized external configuration store (Azure App Configuration)  
B) Distributed configuration files with CI/CD automation  
C) Shared Git submodule containing all configuration  
D) Configuration management database (CMDB)

<details>
<summary><strong>Answer</strong></summary>

**Correct Answer: A) Centralized external configuration store (Azure App Configuration)**

**Explanation**:
- **Single source of truth**: One location for shared configuration (API endpoints, feature flags)
- **Dynamic updates**: Change configuration without redeploying applications
- **Hierarchical keys**: Organize configuration by environment, service, component
- **Version control**: Built-in history and rollback capabilities

**Why other options are less effective**:
- **B)** Still requires 50 file updates + 50 deployments (doesn't solve core problem)
- **C)** Git submodules fragile, require manual updates, don't support runtime refresh
- **D)** CMDBs track infrastructure, not application configuration; no runtime integration

**Implementation**:
```bash
# Centralized configuration
az appconfig kv set --name myapp-config --key "SharedApi:Endpoint" --value "https://api-v2.myapp.com"

# All 50 microservices automatically use new value (30-60s refresh)
```

**AZ-400 Exam Relevance**: External Configuration Store pattern reduces deployment complexity (Exam Objective: Design and implement deployment patterns).
</details>

---

## Question 3: Azure App Configuration Labels

**Scenario**: You need to manage configuration for Development, Staging, and Production environments. Each environment requires different database connection strings and API endpoints.

**What is the MOST efficient way to organize this in Azure App Configuration?**

A) Create three separate App Configuration stores (one per environment)  
B) Use labels to differentiate environment-specific configuration  
C) Use different key prefixes (`dev-`, `staging-`, `prod-`)  
D) Store environment-specific configuration in separate resource groups

<details>
<summary><strong>Answer</strong></summary>

**Correct Answer: B) Use labels to differentiate environment-specific configuration**

**Explanation**:
- **Labels**: Built-in mechanism for environment differentiation (dev, staging, production)
- **Single store**: Centralized management, reduced cost, simplified operations
- **Selective loading**: Applications load configuration with specific label

**Implementation**:
```bash
# Same key, different values per environment
az appconfig kv set --name myapp-config --key "Database:ConnectionString" \
  --value "Server=dev-db;..." --label development

az appconfig kv set --name myapp-config --key "Database:ConnectionString" \
  --value "Server=staging-db;..." --label staging

az appconfig kv set --name myapp-config --key "Database:ConnectionString" \
  --value "Server=prod-db;..." --label production

# Application loads environment-specific configuration
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .Select(KeyFilter.Any, "production");  // Load only production label
});
```

**Why other options are less efficient**:
- **A)** Three stores = triple cost, triple management overhead
- **C)** Key prefixes work but less elegant; no built-in filtering support
- **D)** Resource group separation for billing/RBAC, not configuration management

**AZ-400 Exam Relevance**: Labels enable multi-environment configuration management (Exam Objective: Implement secure continuous deployment).
</details>

---

## Question 4: Azure Key Vault Integration with Pipelines

**Scenario**: Your Azure Pipeline needs to deploy a web application with database credentials stored in Azure Key Vault. The pipeline fails with "The user, group or application does not have secrets get permission."

**What is the MOST LIKELY cause?**

A) Key Vault firewall blocks Azure Pipelines IP addresses  
B) Service connection lacks Key Vault access (no RBAC role or access policy)  
C) Key Vault soft delete is enabled, preventing secret retrieval  
D) Pipeline YAML syntax error in AzureKeyVault@2 task

<details>
<summary><strong>Answer</strong></summary>

**Correct Answer: B) Service connection lacks Key Vault access (no RBAC role or access policy)**

**Explanation**:
- Service connection (service principal) requires **Key Vault Secrets User** role (RBAC) or **Get/List permissions** (access policies)
- Without proper authorization, pipeline cannot retrieve secrets

**Resolution**:
```bash
# Option 1: RBAC (recommended)
az role assignment create \
  --assignee {service-principal-id} \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault

# Option 2: Access Policy (legacy)
az keyvault set-policy \
  --name prod-keyvault \
  --spn {service-principal-id} \
  --secret-permissions get list
```

**Why other options unlikely**:
- **A)** Key Vault allows Azure services by default (firewall not typical issue)
- **C)** Soft delete doesn't prevent retrieval of active secrets (only affects deleted secrets)
- **D)** YAML syntax errors produce different error messages (e.g., "task not found")

**AZ-400 Exam Relevance**: Service connection authorization is fundamental to Azure Pipelines security (Exam Objective: Implement secure continuous deployment).
</details>

---

## Question 5: Feature Flag Filters

**Scenario**: You want to roll out a new checkout flow to 25% of users, but enable it for all beta testers immediately. After one week, you plan to increase to 50% of general users while keeping 100% for beta testers.

**Which feature flag filter combination achieves this?**

A) Percentage filter (25%) only  
B) Targeting filter (beta-testers: 100%) + Percentage filter (25%)  
C) Targeting filter with group rollout percentages (beta-testers: 100%, default: 25%)  
D) Time window filter (start: now, end: 1 week) + Percentage filter (25%)

<details>
<summary><strong>Answer</strong></summary>

**Correct Answer: C) Targeting filter with group rollout percentages (beta-testers: 100%, default: 25%)**

**Explanation**:
- **Targeting filter**: Supports both group-specific rollout and default rollout percentage
- **Group rollout**: Beta testers get 100% enablement
- **Default rollout**: General users get 25% (progressive rollout)

**Implementation**:
```bash
az appconfig feature filter add \
  --name myapp-config \
  --feature NewCheckoutFlow \
  --filter-name Microsoft.Targeting \
  --filter-parameters '{
    "Audience": {
      "Groups": [
        {"Name": "beta-testers", "RolloutPercentage": 100}
      ],
      "DefaultRolloutPercentage": 25
    }
  }'

# After 1 week, increase general users to 50%
az appconfig feature filter update \
  --name myapp-config \
  --feature NewCheckoutFlow \
  --filter-name Microsoft.Targeting \
  --filter-parameters '{
    "Audience": {
      "Groups": [
        {"Name": "beta-testers", "RolloutPercentage": 100}
      ],
      "DefaultRolloutPercentage": 50
    }
  }'
```

**Why other options don't meet requirements**:
- **A)** Percentage filter alone can't distinguish beta testers (25% applies to everyone)
- **B)** Two separate filters conflict (OR logic); targeting handles both scenarios
- **D)** Time window for duration, not user segmentation

**AZ-400 Exam Relevance**: Progressive rollouts with targeting reduce deployment risk (Exam Objective: Implement deployment patterns).
</details>

---

## Question 6: Dynamic Configuration Refresh

**Scenario**: Your application uses Azure App Configuration with a 30-second cache expiration. You update 10 configuration values and need all values to refresh atomically (no partial updates).

**What is the BEST approach?**

A) Update all 10 values simultaneously using Azure CLI batch command  
B) Use a sentinel key and update it after changing all 10 values  
C) Decrease cache expiration to 5 seconds for faster refresh  
D) Trigger application restart after configuration changes

<details>
<summary><strong>Answer</strong></summary>

**Correct Answer: B) Use a sentinel key and update it after changing all 10 values**

**Explanation**:
- **Sentinel key pattern**: Application only refreshes when sentinel key changes
- **Atomic updates**: Change all values first, then update sentinel (triggers refresh)
- **Consistent state**: No partial configuration (all or nothing)

**Implementation**:
```bash
# 1. Update all configuration values
az appconfig kv set --name myapp-config --key "Database:Host" --value "new-db.com"
az appconfig kv set --name myapp-config --key "Database:Port" --value "1433"
az appconfig kv set --name myapp-config --key "Api:Endpoint" --value "https://api-v2.com"
# ... (7 more updates)

# 2. Update sentinel key (triggers refresh for all values)
az appconfig kv set --name myapp-config --key "Sentinel" --value "$(date +%s)"

# Application refreshes when sentinel changes (all 10 values updated together)
```

**Application Configuration**:
```csharp
.ConfigureRefresh(refresh =>
{
    refresh.Register("Sentinel", refreshAll: true)  // Monitor sentinel only
           .SetCacheExpiration(TimeSpan.FromSeconds(30));
});
```

**Why other options are less effective**:
- **A)** Azure CLI doesn't guarantee atomic updates; applications may refresh mid-update
- **C)** Reduces refresh window but doesn't guarantee atomicity; increases API calls/cost
- **D)** Restart defeats purpose of dynamic configuration (downtime)

**AZ-400 Exam Relevance**: Sentinel key pattern ensures configuration consistency (Exam Objective: Implement deployment patterns).
</details>

---

## Question 7: DevOps Inner and Outer Loop

**Scenario**: A developer wants fast local iteration (5-second feedback) without network dependencies. Production deployments require Azure Key Vault secrets.

**What is the BEST configuration strategy?**

A) Use Azure Key Vault for both local development and production  
B) Use user secrets locally; Azure Key Vault in production (environment-based)  
C) Use hardcoded secrets in both environments (for consistency)  
D) Use environment variables for all secrets (local and production)

<details>
<summary><strong>Answer</strong></summary>

**Correct Answer: B) Use user secrets locally; Azure Key Vault in production (environment-based)**

**Explanation**:
- **Inner loop (local)**: User secrets or local files (fast, offline, no cloud costs)
- **Outer loop (production)**: Azure Key Vault (secure, audited, centralized)
- **Environment-based switching**: Application automatically selects source based on `ASPNETCORE_ENVIRONMENT`

**Implementation**:
```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsDevelopment())
{
    // Inner loop: User secrets (fast local iteration)
    builder.Configuration.AddUserSecrets<Program>();
}
else
{
    // Outer loop: Azure Key Vault (production security)
    builder.Configuration.AddAzureAppConfiguration(options =>
    {
        options.Connect(connectionString)
               .ConfigureKeyVault(kv =>
               {
                   kv.SetCredential(new DefaultAzureCredential());
               });
    });
}
```

**Local Development**:
```bash
# Developer sets user secrets (stored outside project directory)
dotnet user-secrets set "DatabasePassword" "local-dev-password"
dotnet run  # Fast startup, no network calls
```

**Why other options inappropriate**:
- **A)** Key Vault adds network latency, requires authentication, increases dev loop time
- **C)** Hardcoded secrets violate security best practices (never acceptable)
- **D)** Environment variables better than hardcoding, but lack audit trails and rotation

**AZ-400 Exam Relevance**: Balancing developer productivity with production security (Exam Objective: Implement secure continuous deployment).
</details>

---

## Question 8: Azure DevOps Variable Groups

**Scenario**: You have 15 pipelines that all use the same database connection string. Currently, the connection string is duplicated in each pipeline YAML file. When the database changes, you must update all 15 files.

**What is the MOST maintainable solution?**

A) Use Azure Pipelines library variable groups linked to Key Vault  
B) Create a template YAML file with shared variables  
C) Use Azure App Configuration references in each pipeline  
D) Store connection string in Azure DevOps project settings

<details>
<summary><strong>Answer</strong></summary>

**Correct Answer: A) Use Azure Pipelines library variable groups linked to Key Vault**

**Explanation**:
- **Variable group**: Centralized collection of variables shared across pipelines
- **Key Vault integration**: Automatically syncs secret values (no manual updates)
- **Single source of truth**: Update Key Vault secret → all 15 pipelines use new value

**Implementation**:
```
Pipelines → Library → Variable Groups → Create
├── Name: Shared-Database-Config
├── Link to Azure Key Vault: prod-keyvault
└── Select secrets: DatabaseConnectionString

# Use in all 15 pipelines
variables:
- group: Shared-Database-Config

steps:
- script: |
    echo "Connection: $(DatabaseConnectionString)"
```

**Benefits**:
- One-time setup (variable group)
- Zero pipeline changes when secret updates
- Audit trail (Key Vault access logs)
- Secret masking (automatic in pipeline logs)

**Why other options less maintainable**:
- **B)** Template reduces duplication but still requires manual secret management
- **C)** App Configuration for application runtime, not CI/CD pipelines
- **D)** Project settings not designed for secrets; no Key Vault integration

**AZ-400 Exam Relevance**: Variable groups reduce pipeline maintenance overhead (Exam Objective: Implement continuous integration).
</details>

---

## Question 9: Feature Flag Lifecycle

**Scenario**: You successfully rolled out a new payment provider to 100% of users 30 days ago. The old provider has been decommissioned. The feature flag `NewPaymentProvider` is still in the codebase.

**What should you do?**

A) Keep the feature flag indefinitely for potential rollback  
B) Disable the feature flag but leave the code in place  
C) Delete the feature flag from App Configuration and remove the conditional code  
D) Convert the feature flag to a configuration setting

<details>
<summary><strong>Answer</strong></summary>

**Correct Answer: C) Delete the feature flag from App Configuration and remove the conditional code**

**Explanation**:
- **Technical debt**: Obsolete feature flags clutter codebase and configuration
- **Cleanup best practice**: After 100% rollout + stabilization period (typically 30 days), remove flag
- **Simplify code**: Delete conditional logic, keep winning implementation only

**Before Cleanup**:
```csharp
if (await _featureManager.IsEnabledAsync("NewPaymentProvider"))
{
    return await _stripeService.ProcessPaymentAsync(order);  // New (winner)
}
else
{
    return await _legacyPaymentService.ProcessPaymentAsync(order);  // Old (decommissioned)
}
```

**After Cleanup**:
```csharp
// Simple, maintainable code (no conditional logic)
return await _stripeService.ProcessPaymentAsync(order);
```

**Cleanup Steps**:
```bash
# 1. Delete feature flag from App Configuration
az appconfig feature delete --name myapp-config --feature NewPaymentProvider --yes

# 2. Remove feature flag checks from code
# 3. Delete old implementation (_legacyPaymentService)
# 4. Deploy simplified code
```

**Why other options suboptimal**:
- **A)** Old provider decommissioned (no rollback possible); flag serves no purpose
- **B)** Disabled flag still technical debt; doesn't simplify code
- **D)** Not configuration (value that changes); was temporary release control

**AZ-400 Exam Relevance**: Feature flag lifecycle management prevents technical debt (Exam Objective: Implement deployment patterns).
</details>

---

## Question 10: Secret Rotation

**Scenario**: Your organization's security policy requires rotating all secrets every 90 days. You have 50 secrets across 10 Key Vaults used by 100 applications.

**What is the MOST scalable approach to enforce this policy?**

A) Create calendar reminders to manually rotate secrets every 90 days  
B) Use Azure Key Vault expiration policies and Event Grid automation  
C) Implement a monthly audit to identify expired secrets  
D) Require developers to rotate secrets before each deployment

<details>
<summary><strong>Answer</strong></summary>

**Correct Answer: B) Use Azure Key Vault expiration policies and Event Grid automation**

**Explanation**:
- **Expiration policies**: Set secret lifetime (90 days) in Key Vault
- **Event Grid**: Triggers automation 30 days before expiration
- **Azure Function**: Rotates secret, updates target system, stores new version
- **Scalable**: Handles 50 secrets across 10 Key Vaults automatically

**Implementation**:
```bash
# 1. Set expiration policy on secret
az keyvault secret set \
  --vault-name prod-keyvault \
  --name DatabasePassword \
  --value "P@ssw0rd123!" \
  --expires "2026-04-14T00:00:00Z"  # 90 days from creation

# 2. Configure Event Grid subscription
az eventgrid event-subscription create \
  --name secret-expiry-handler \
  --source-resource-id /subscriptions/{sub-id}/resourceGroups/myapp-rg/providers/Microsoft.KeyVault/vaults/prod-keyvault \
  --endpoint https://myapp-rotate-func.azurewebsites.net/api/RotateSecret \
  --included-event-types Microsoft.KeyVault.SecretNearExpiry

# 3. Azure Function rotates secret automatically
[FunctionName("RotateSecret")]
public static async Task Run([EventGridTrigger] EventGridEvent eventGridEvent)
{
    var newPassword = GenerateSecurePassword();
    await UpdateDatabasePasswordAsync(newPassword);  // Update database
    await UpdateKeyVaultSecretAsync(newPassword);    // Store new version
}
```

**Why other options don't scale**:
- **A)** Manual rotation error-prone, doesn't scale to 50 secrets, no audit trail
- **C)** Reactive (identifies expired, doesn't prevent expiration), monthly gaps
- **D)** Inconsistent (deployments irregular), increases deployment time

**AZ-400 Exam Relevance**: Automated secret rotation reduces security risk and operational overhead (Exam Objective: Implement security and validate code bases).
</details>

---

## Scoring Guide

- **9-10 correct**: Excellent understanding of configuration management patterns
- **7-8 correct**: Good grasp of core concepts, review specific topics
- **5-6 correct**: Basic understanding, revisit module content
- **0-4 correct**: Recommend reviewing module before proceeding

---

## Key Concepts Covered

✅ **Configuration anti-patterns**: Hardcoded secrets, Git-committed passwords  
✅ **Separation of concerns**: External configuration stores, centralized management  
✅ **Azure App Configuration**: Labels, hierarchical keys, dynamic refresh  
✅ **Key Vault integration**: Pipelines, service connections, RBAC/access policies  
✅ **Feature flags**: Filters (percentage, targeting, time window), lifecycle management  
✅ **Dynamic configuration**: Sentinel key pattern, cache expiration, atomic updates  
✅ **Inner/outer loops**: User secrets (local), Key Vault (production)  
✅ **Variable groups**: Pipeline reusability, Key Vault linking  
✅ **Secret rotation**: Expiration policies, Event Grid automation  
✅ **Best practices**: Technical debt cleanup, automated security compliance  

---

**Next**: Review module summary and key takeaways →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/14-knowledge-check)
