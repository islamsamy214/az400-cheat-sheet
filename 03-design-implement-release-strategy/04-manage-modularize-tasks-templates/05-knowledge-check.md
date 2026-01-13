# Knowledge Check

⏱️ **Duration**: ~5 minutes | 📝 **Type**: Assessment | ✅ **Passing Score**: 80% (8/10 correct)

## Overview

Test your understanding of **task groups**, **variable groups**, and **variable management** in Azure DevOps release pipelines. This assessment covers concepts from Units 1-4.

---

## Instructions

- Read each question carefully
- Select the best answer for multiple-choice questions
- Check your answers against the answer key at the end
- Review explanations for any incorrect answers

---

## Questions

### Question 1: Task Group Scope

**What is the primary limitation of task groups in Azure DevOps?**

A) Task groups cannot be versioned  
B) Task groups only work with Classic Pipelines, not YAML pipelines  
C) Task groups cannot contain more than 5 tasks  
D) Task groups cannot be shared across different projects

<details>
<summary>💡 <b>Hint</b></summary>
Think about the pipeline types supported by task groups.
</details>

---

### Question 2: Variable Precedence

**You have a variable named `appName` defined at multiple levels:**
- Pipeline level: `appName = 'myapp-pipeline'`
- Variable group: `appName = 'myapp-group'`
- Stage level: `appName = 'myapp-stage'`
- Job level: `appName = 'myapp-job'`

**Which value will be used when the job executes?**

A) `myapp-pipeline`  
B) `myapp-group`  
C) `myapp-stage`  
D) `myapp-job`

<details>
<summary>💡 <b>Hint</b></summary>
Remember the precedence hierarchy: Job > Stage > Pipeline > Variable Group > System
</details>

---

### Question 3: Secret Variables

**What happens when you echo a secret variable in a pipeline script?**

```yaml
variables:
  apiKey: 'abc123xyz789'  # Marked as secret

steps:
- script: echo "API Key: $(apiKey)"
```

**What will appear in the logs?**

A) `API Key: abc123xyz789`  
B) `API Key: ***`  
C) `API Key: $(apiKey)` (literal string)  
D) The task will fail with an error

<details>
<summary>💡 <b>Hint</b></summary>
Azure DevOps automatically masks secret variable values in logs.
</details>

---

### Question 4: Variable Group Benefits

**What is the primary benefit of using variable groups instead of pipeline-level variables?**

A) Variable groups are faster to process  
B) Variable groups can store more variables (no limit)  
C) Variable groups enable cross-pipeline sharing and centralized management  
D) Variable groups automatically encrypt all values

<details>
<summary>💡 <b>Hint</b></summary>
Think about reusability across multiple pipelines.
</details>

---

### Question 5: Azure Key Vault Integration

**Your team stores database connection strings in Azure Key Vault. You need to use these secrets in 50 different release pipelines. What is the BEST approach?**

A) Create 50 secret variables, one in each pipeline, and manually copy values from Key Vault  
B) Store connection strings in pipeline YAML files (encrypted)  
C) Create a variable group linked to Azure Key Vault, then link the group to all 50 pipelines  
D) Use Azure CLI to fetch secrets during each pipeline run

<details>
<summary>💡 <b>Hint</b></summary>
Consider maintainability and automatic sync when secrets change.
</details>

---

### Question 6: Task Group Versioning

**You updated a task group from version 1.0.0 to version 2.0.0 with a breaking change (new required parameter). Which pipelines will automatically use version 2.0.0?**

**Pipeline configurations**:
- Pipeline A: Uses `DeployTask@*` (latest)
- Pipeline B: Uses `DeployTask@1` (major version)
- Pipeline C: Uses `DeployTask@1.0.0` (exact version)

A) Only Pipeline A  
B) Pipelines A and B  
C) Pipelines A, B, and C  
D) None (manual update required)

<details>
<summary>💡 <b>Hint</b></summary>
Think about semantic versioning and auto-upgrade behavior for major version changes.
</details>

---

### Question 7: Variable Scoping

**You have a variable group named "Azure-Config" linked to a release pipeline with three stages: Dev, Test, and Prod. The variable group is scoped to only the Dev and Test stages. What happens if a task in the Prod stage references a variable from this group?**

A) The task uses the variable value successfully  
B) The task fails with "variable not found" error  
C) The task uses a default empty value  
D) The pipeline prompts for manual input

<details>
<summary>💡 <b>Hint</b></summary>
Variable groups respect scope boundaries.
</details>

---

### Question 8: Task Group Parameters

**What is the purpose of extracting parameters in a task group?**

A) To reduce the number of tasks in the group  
B) To enable configuration flexibility and reusability across different contexts  
C) To improve pipeline performance  
D) To automatically create documentation

<details>
<summary>💡 <b>Hint</b></summary>
Think about how parameters enable the same task group to be used with different values.
</details>

---

### Question 9: Secret Variable Access in Scripts

**Which of the following is the CORRECT way to use a secret variable in a Bash script within an Azure Pipeline?**

**Given secret variable**: `databasePassword = 'MySecretP@ss'` (marked as secret)

A) 
```bash
script: echo "Password: $(databasePassword)"
```

B)
```bash
script: echo "Password: $DATABASE_PASSWORD"
env:
  DATABASE_PASSWORD: $(databasePassword)
```

C)
```bash
script: |
  PASSWORD=$(databasePassword)
  echo "Password: $PASSWORD"
```

D)
```bash
script: cat $(databasePassword) > password.txt
```

<details>
<summary>💡 <b>Hint</b></summary>
Secret variables should be passed as environment variables to avoid logging issues.
</details>

---

### Question 10: Variable Group Updates

**You updated a variable value in a variable group from `apiUrl = 'https://dev.api.com'` to `apiUrl = 'https://prod.api.com'`. You have 10 release pipelines linked to this variable group. How many pipelines need to be manually updated to use the new value?**

A) 10 (all pipelines)  
B) 5 (half the pipelines)  
C) 1 (only the first pipeline)  
D) 0 (automatic propagation)

<details>
<summary>💡 <b>Hint</b></summary>
Variable group changes automatically propagate to linked pipelines.
</details>

---

## Answer Key

### Question 1: B
**Correct Answer**: B) Task groups only work with Classic Pipelines, not YAML pipelines

**Explanation**:
Task groups are a feature of **Classic Pipelines** (UI-based pipelines) only. For YAML pipelines, you must use **YAML templates** instead. This is a critical limitation to remember when modernizing pipelines.

**Workaround**: Convert task groups to YAML templates when migrating to YAML pipelines.

**Why other answers are wrong**:
- A) Task groups CAN be versioned (semantic versioning supported)
- C) No limit on number of tasks in a task group
- D) Task groups CAN be shared across projects (organization-level sharing)

---

### Question 2: D
**Correct Answer**: D) `myapp-job`

**Explanation**:
Azure DevOps variable precedence hierarchy (highest to lowest):
```
1. Job-level variables    ← HIGHEST PRIORITY (myapp-job wins)
2. Stage-level variables
3. Pipeline-level variables
4. Variable group variables
5. Predefined system variables
```

Job-level variables have the **highest precedence** and will override all other levels.

**Real-world scenario**: Use job-level variables for temporary overrides during debugging or testing.

**Why other answers are wrong**:
- A, B, C) Lower precedence than job-level variables

---

### Question 3: B
**Correct Answer**: B) `API Key: ***`

**Explanation**:
Azure DevOps **automatically masks** secret variable values in pipeline logs to prevent accidental exposure. When you attempt to print a secret variable, the output shows `***` instead of the actual value.

**Log masking behavior**:
```
Secret variable value: "abc123xyz789"
Logged output: "***" (automatically replaced)
```

**Important**: While secret variables are masked in logs, they can still be accessed by tasks through environment variables. This prevents logging but allows legitimate use.

**Why other answers are wrong**:
- A) Secret values are masked, not visible
- C) Variables are resolved, not shown as literal strings
- D) Task succeeds; masking is automatic, not an error

---

### Question 4: C
**Correct Answer**: C) Variable groups enable cross-pipeline sharing and centralized management

**Explanation**:
The primary benefit of variable groups is **reusability**:

| Scenario | Without Variable Groups | With Variable Groups |
|----------|------------------------|---------------------|
| **50 pipelines** need Azure subscription name | Copy variable to 50 pipelines | Link 1 variable group to 50 pipelines |
| **Subscription name changes** | Update 50 pipelines manually | Update 1 variable group (auto-propagates) |
| **Audit who uses subscription** | Check 50 pipelines individually | Check 1 variable group permissions |

**Key Benefits**:
- ✅ Single source of truth
- ✅ Automatic propagation of changes
- ✅ Reduced duplication
- ✅ Centralized access control

**Why other answers are wrong**:
- A) No performance difference
- B) Both can store unlimited variables
- D) Only secret variables are encrypted (not all)

---

### Question 5: C
**Correct Answer**: C) Create a variable group linked to Azure Key Vault, then link the group to all 50 pipelines

**Explanation**:
This approach provides:

**✅ Benefits**:
- **Automatic sync**: Key Vault secret updates → variable group updates → all 50 pipelines updated
- **Centralized management**: Rotate secrets in Key Vault once
- **Enterprise security**: HSM-backed encryption, audit logging
- **Compliance**: Meet regulatory requirements (HIPAA, PCI-DSS)
- **Access control**: Manage permissions at Key Vault level

**Implementation**:
```
Azure Key Vault (secrets)
    ↓ Links to
Variable Group (Azure DevOps)
    ↓ Links to
50 Release Pipelines
```

**Why other answers are wrong**:
- A) Manual copying = high maintenance, error-prone
- B) Secrets in YAML = security risk, version control exposure
- D) Fetching at runtime = no caching, additional API calls, complexity

---

### Question 6: A
**Correct Answer**: A) Only Pipeline A

**Explanation**:
**Semantic Versioning Auto-Upgrade Rules**:

| Pipeline Reference | v1.0.0 → v1.1.0 (minor) | v1.1.0 → v2.0.0 (major) |
|-------------------|------------------------|------------------------|
| `@*` (latest) | ✅ Auto-upgrades | ✅ Auto-upgrades |
| `@1` (major) | ✅ Auto-upgrades | ❌ Stays on v1.x.x |
| `@1.0.0` (exact) | ❌ Stays on v1.0.0 | ❌ Stays on v1.0.0 |

**Reasoning**:
- **Pipeline A** (`@*`): Always uses latest version → auto-upgrades to v2.0.0
- **Pipeline B** (`@1`): Pinned to major version 1 → stays on v1.x.x (won't upgrade to v2.0.0)
- **Pipeline C** (`@1.0.0`): Pinned to exact version → never auto-upgrades

**Best Practice**: 
- Use `@*` in dev/test for latest features
- Use `@1` in production for stability (minor updates only)
- Use `@1.0.0` for critical pipelines (zero changes)

**Why other answers are wrong**:
- B) Pipeline B won't auto-upgrade to v2.0.0 (major version boundary)
- C, D) Pipeline C is pinned to exact version (no auto-upgrade)

---

### Question 7: B
**Correct Answer**: B) The task fails with "variable not found" error

**Explanation**:
**Variable group scoping** is a **hard boundary**. When a variable group is scoped to specific stages, it is completely unavailable to excluded stages.

**Scenario**:
```
Variable Group: "Azure-Config"
├── Scope: ☑️ Dev, ☑️ Test, ☐ Prod
└── Variables: azureSubscription, resourceGroup

Prod Stage Task:
- task: AzureWebApp@1
  inputs:
    azureSubscription: $(azureSubscription)  ← FAILS: variable not found
```

**Error Message**:
```
##[error]The variable 'azureSubscription' is not defined.
```

**Solution**: Extend scope to include Prod stage OR create separate variable group for Prod.

**Why other answers are wrong**:
- A) Scoped variables are not accessible
- C) No default empty value; explicit error thrown
- D) No manual prompt; pipeline fails

---

### Question 8: B
**Correct Answer**: B) To enable configuration flexibility and reusability across different contexts

**Explanation**:
**Parameter extraction** transforms hardcoded values into configurable inputs:

**Before (hardcoded)**:
```yaml
tasks:
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'Production-Sub'  # Fixed value
    appName: 'myapp-prod'                 # Fixed value
```
**Problem**: Can't reuse for different subscriptions or app names!

**After (parameterized)**:
```yaml
parameters:
- name: azureSubscription
  type: string
- name: appName
  type: string

tasks:
- task: AzureWebApp@1
  inputs:
    azureSubscription: '$(azureSubscription)'  # Configurable
    appName: '$(appName)'                      # Configurable
```

**Usage**:
```yaml
# Pipeline 1: Dev environment
- task: DeployTask@1
  inputs:
    azureSubscription: 'Dev-Sub'
    appName: 'myapp-dev'

# Pipeline 2: Prod environment (same task group)
- task: DeployTask@1
  inputs:
    azureSubscription: 'Prod-Sub'
    appName: 'myapp-prod'
```

**Benefits**:
- ✅ **Reusability**: Same task group for multiple environments
- ✅ **Flexibility**: Different values per pipeline
- ✅ **Maintainability**: Update logic once, applies everywhere
- ✅ **DRY Principle**: Don't Repeat Yourself

**Why other answers are wrong**:
- A) Parameters don't reduce task count
- C) No performance impact
- D) Documentation is manual, not automatic

---

### Question 9: B
**Correct Answer**: B)
```bash
script: echo "Password: $DATABASE_PASSWORD"
env:
  DATABASE_PASSWORD: $(databasePassword)
```

**Explanation**:
**Correct Pattern for Secret Variables in Scripts**:

```yaml
variables:
  databasePassword: 'MySecretP@ss'  # Marked as secret

steps:
- script: |
    # Use environment variable (not macro syntax)
    mysql -h myserver -u admin -p$DATABASE_PASSWORD
  env:
    DATABASE_PASSWORD: $(databasePassword)  # Pass secret as env var
```

**Why This Works**:
1. Secret variable passed to script via `env:` mapping
2. Environment variables not logged by default
3. Script accesses via `$DATABASE_PASSWORD` (env var syntax)
4. Secret still masked if accidentally printed

**Log Output**:
```
Password: ***  ← Still masked even with env var method
```

**Why other answers are wrong**:
- A) `$(databasePassword)` in script logs the secret (masked, but avoid pattern)
- C) Macro syntax doesn't work in bash variable assignment
- D) Writing secrets to files = security risk

**Best Practice**: Always use environment variable mapping for secrets in scripts.

---

### Question 10: D
**Correct Answer**: D) 0 (automatic propagation)

**Explanation**:
**Variable group updates automatically propagate** to all linked pipelines:

**Scenario**:
```
1. Initial state:
   Variable Group: "API-Config"
   └── apiUrl: 'https://dev.api.com'
   Linked to: 10 pipelines

2. Update variable group:
   └── apiUrl: 'https://prod.api.com'  ← Changed

3. Next pipeline run:
   All 10 pipelines automatically use new value ✅
```

**How It Works**:
- Variable groups stored in Azure DevOps Library (centralized)
- Pipelines **reference** variable groups (not copy)
- Each run fetches latest values from Library
- No pipeline re-configuration required

**Timeline**:
```
Update variable group → Save
  ↓ (immediate)
Next pipeline run → Uses new value ✅
```

**Important**: Changes apply to **new pipeline runs**, not in-progress runs.

**Why other answers are wrong**:
- A, B, C) No manual updates needed; propagation is automatic

---

## Scoring Guide

**Count your correct answers**:

| Score | Grade | Interpretation |
|-------|-------|----------------|
| **10/10** | 🏆 **Excellent** | Complete mastery of module concepts |
| **8-9/10** | ✅ **Pass** | Strong understanding, ready to proceed |
| **6-7/10** | ⚠️ **Review** | Good foundation, review missed concepts |
| **0-5/10** | ❌ **Revisit** | Re-study Units 1-4 before proceeding |

---

## Detailed Explanations Summary

### Key Concepts Reinforced

**1. Task Groups**:
- ❌ Classic Pipelines only (not YAML)
- ✅ Versioned with semantic versioning
- ✅ Parameters enable reusability
- ✅ Auto-propagation of updates

**2. Variables**:
- **Precedence**: Job > Stage > Pipeline > Variable Group > System
- **Scoping**: Hard boundaries (excluded stages cannot access)
- **Secret Variables**: Automatically masked in logs
- **Best Practice**: Use environment variables for secrets in scripts

**3. Variable Groups**:
- **Primary Benefit**: Cross-pipeline sharing and centralized management
- **Updates**: Automatic propagation to all linked pipelines (0 manual updates)
- **Security**: Link to Azure Key Vault for enterprise secret management
- **Access Control**: Manage permissions at Library level

**4. Azure Key Vault Integration**:
- **Best Practice**: Store production secrets in Key Vault
- **Sync**: Automatic updates from Key Vault to variable groups
- **Compliance**: Meet regulatory requirements
- **Audit**: Track secret access with diagnostic logs

---

## What You've Learned

After completing this knowledge check, you should be confident in:
- ✅ Task group capabilities and limitations
- ✅ Variable precedence and scoping rules
- ✅ Secret variable management best practices
- ✅ Variable group benefits and use cases
- ✅ Azure Key Vault integration for enterprise security
- ✅ Automatic propagation of configuration changes

---

## Next Steps

✅ **Completed**: Knowledge check and self-assessment

**Continue to**: Unit 6 - Summary (module recap and additional resources)

---

## Additional Practice

**Hands-On Reinforcement**:
1. Create a variable group with 5 variables (2 secrets)
2. Link to 3 pipelines with different stage scopes
3. Update variable group → verify automatic propagation
4. Set up Azure Key Vault integration (if Azure subscription available)
5. Test variable precedence by creating overrides at different levels

**Recommended Review** (if score < 8/10):
- Re-read Unit 2: Examine task groups
- Re-read Unit 3: Explore variables in release pipelines
- Repeat Exercise: Create and manage variable groups (Unit 4)

[↩️ Back to Module Overview](01-introduction.md) | [⬅️ Previous: Exercise](04-exercise-create-manage-variable-groups.md) | [➡️ Next: Summary](06-summary.md)
