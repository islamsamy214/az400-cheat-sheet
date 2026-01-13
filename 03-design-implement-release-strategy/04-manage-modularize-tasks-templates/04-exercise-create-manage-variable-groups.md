# Exercise: Create and Manage Variable Groups

⏱️ **Duration**: ~25 minutes | 🔬 **Type**: Hands-On Lab | 💻 **Environment**: Azure DevOps

## Overview

This hands-on exercise guides you through creating, configuring, and using **variable groups** for centralized configuration management across release pipelines. You'll create a variable group, populate it with environment-specific values, link it to release pipelines, and scope variables to specific stages.

---

## Lab Objectives

After completing this exercise, you'll be able to:
- ✅ Create variable groups in Azure DevOps Library
- ✅ Add normal and secret variables to groups
- ✅ Link variable groups to release pipelines
- ✅ Scope variable groups to specific stages
- ✅ Override variable group values at stage level
- ✅ Integrate Azure Key Vault with variable groups

---

## Prerequisites

Before starting this exercise, ensure you have:

| Requirement | Description |
|-------------|-------------|
| **Azure DevOps Account** | Free account at [dev.azure.com](https://dev.azure.com) |
| **Azure DevOps Project** | Existing project or create new one |
| **Release Pipeline** | Existing release pipeline with multiple stages (Dev, Test, Prod) |
| **Permissions** | Contributor access to pipelines and Library |
| **Optional: Azure Subscription** | For Azure Key Vault integration (Exercise 2) |

---

## Exercise 1: Create a Basic Variable Group

### Step 1: Navigate to Library

1. Open your Azure DevOps project
2. Click **Pipelines** in the left navigation
3. Click **Library**
4. Click **+ Variable group** button

**Screenshot Placeholder**: Azure DevOps Library page with "+ Variable group" button highlighted

---

### Step 2: Configure Variable Group

**Basic Information**:

| Field | Value |
|-------|-------|
| **Name** | `Website Test Product Details` |
| **Description** | `Test product configuration for website deployment` |

**Screenshot Placeholder**: Variable group creation form

---

### Step 3: Add Normal Variables

Click **+ Add** to add variables. Create the following:

#### Variable 1: ProductCode
```
Name: ProductCode
Value: REDPOLOXL
Type: Normal (default)
```

#### Variable 2: Quantity
```
Name: Quantity
Value: 12
Type: Normal
```

#### Variable 3: SalesUnit
```
Name: SalesUnit
Value: Each
Type: Normal
```

**Screenshot Placeholder**: Variable group with three variables added

---

### Step 4: Add Secret Variable

Add a secret variable for sensitive data:

#### Variable 4: ProductApiKey
```
Name: ProductApiKey
Value: abc123xyz789secretkey456  ← Type but not shown
Type: Secret (click padlock icon 🔒)
```

**To mark as secret**:
1. Click **+ Add**
2. Enter variable name: `ProductApiKey`
3. Enter value (will be hidden after saving)
4. Click the **padlock icon** 🔒 next to the value field
5. Value changes to dots: `••••••••••••••••••`

**Screenshot Placeholder**: Secret variable with masked value

---

### Step 5: Save Variable Group

1. Click **Save** button at top of page
2. Confirmation message appears: "Variable group saved successfully"

**Result**: Variable group created and available in Library

---

## Exercise 2: Link Variable Group to Release Pipeline

### Step 1: Open Release Pipeline

1. Navigate to **Pipelines** → **Releases**
2. Select your release pipeline (or create new one)
3. Click **Edit** button

---

### Step 2: Add Variable Group

1. Click **Variables** tab
2. Click **Variable groups** section
3. Click **Link variable group** button
4. Select `Website Test Product Details` from dropdown
5. Click **Link** button

**Screenshot Placeholder**: Variable groups section with "Link variable group" button

---

### Step 3: Configure Variable Group Scope

By default, variable group is linked to **all stages**. Let's scope it to specific stages:

1. In the **Variable groups** section, find `Website Test Product Details`
2. Click **Scope** dropdown next to the group name
3. Select specific stages:
   - ☑️ **Development**
   - ☑️ **Test Team A**
   - ☑️ **Test Team B**
   - ☐ **Production** (excluded)

**Why Scope?** Test product details should only be available in non-production environments.

**Screenshot Placeholder**: Scope selection dialog with stages checked

---

### Step 4: Save Release Pipeline

1. Click **Save** button at top of page
2. Add comment: "Linked Website Test Product Details variable group"
3. Click **OK**

---

## Exercise 3: Use Variables in Release Tasks

### Step 1: Add PowerShell Task to Stage

1. Edit release pipeline
2. Navigate to **Development** stage
3. Click **Tasks** tab
4. Click **+** to add task
5. Search for **PowerShell**
6. Select **PowerShell** task
7. Click **Add**

---

### Step 2: Configure PowerShell Task

**Task Configuration**:

| Field | Value |
|-------|-------|
| **Display name** | `Display Product Details` |
| **Type** | `Inline` |
| **Script** | See below |

**PowerShell Script**:
```powershell
# Access variables from variable group
Write-Host "=========================================="
Write-Host "Product Details from Variable Group"
Write-Host "=========================================="
Write-Host "Product Code: $(ProductCode)"
Write-Host "Quantity: $(Quantity)"
Write-Host "Sales Unit: $(SalesUnit)"
Write-Host "API Key: $(ProductApiKey)"  # Will be masked as ***
Write-Host "=========================================="

# Use in business logic
$productInfo = @{
    Code = "$(ProductCode)"
    Quantity = [int]"$(Quantity)"
    Unit = "$(SalesUnit)"
}

Write-Host "Product Info Object Created:"
Write-Host ($productInfo | ConvertTo-Json)
```

**Screenshot Placeholder**: PowerShell task configuration with inline script

---

### Step 3: Run Release

1. Click **Save** to save pipeline
2. Click **Create release** button
3. Select **Development** stage
4. Click **Create**
5. Wait for release to complete

---

### Step 4: Verify Output

1. Click on the release run
2. Navigate to **Development** stage
3. Click **Logs** tab
4. Find **Display Product Details** task
5. Verify output:

**Expected Log Output**:
```
==========================================
Product Details from Variable Group
==========================================
Product Code: REDPOLOXL
Quantity: 12
Sales Unit: Each
API Key: ***  ← Automatically masked!
==========================================
Product Info Object Created:
{
  "Code": "REDPOLOXL",
  "Quantity": 12,
  "Unit": "Each"
}
```

**✅ Success Indicator**: Secret variable (`ProductApiKey`) shows as `***` instead of actual value

**Screenshot Placeholder**: Release logs showing masked secret variable

---

## Exercise 4: Override Variable Group Values

### Scenario

**Problem**: `ProductCode` should be different for Test Team A vs Test Team B

**Solution**: Override variable group value at **stage level**

---

### Step 1: Add Stage Variable to Test Team A

1. Edit release pipeline
2. Navigate to **Test Team A** stage
3. Click **Variables** tab (stage-level)
4. Click **+ Add**
5. Configure override:

```
Name: ProductCode
Value: BLUEPOLOXL  ← Different from variable group (REDPOLOXL)
Scope: Test Team A (stage)
```

6. Click **Save**

---

### Step 2: Add Stage Variable to Test Team B

Repeat for **Test Team B** stage:

```
Name: ProductCode
Value: GREENPOLO
Scope: Test Team B (stage)
```

---

### Step 3: Run Release to All Stages

1. Create new release
2. Deploy to all stages: Development, Test Team A, Test Team B
3. Compare outputs:

**Development Stage Output**:
```
Product Code: REDPOLOXL  ← From variable group (default)
```

**Test Team A Stage Output**:
```
Product Code: BLUEPOLOXL  ← Stage variable overrides variable group
```

**Test Team B Stage Output**:
```
Product Code: GREENPOLO  ← Stage variable overrides variable group
```

**✅ Validation**: Stage variables successfully override variable group values

**Screenshot Placeholder**: Three stage logs showing different ProductCode values

---

## Exercise 5: Integrate Azure Key Vault (Optional)

**⚠️ Requires**: Azure subscription with Key Vault resource

---

### Step 1: Create Azure Key Vault

**Azure CLI**:
```bash
# Variables
RESOURCE_GROUP="devops-rg"
KEY_VAULT_NAME="myapp-keyvault-$(openssl rand -hex 4)"  # Unique name
LOCATION="eastus"

# Create resource group
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION

# Create Key Vault
az keyvault create \
  --name $KEY_VAULT_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION
```

**Output**:
```json
{
  "id": "/subscriptions/.../resourceGroups/devops-rg/providers/Microsoft.KeyVault/vaults/myapp-keyvault-a1b2c3d4",
  "name": "myapp-keyvault-a1b2c3d4",
  "properties": {
    "vaultUri": "https://myapp-keyvault-a1b2c3d4.vault.azure.net/"
  }
}
```

---

### Step 2: Add Secrets to Key Vault

```bash
# Add ProductApiKey secret
az keyvault secret set \
  --vault-name $KEY_VAULT_NAME \
  --name ProductApiKey \
  --value "abc123xyz789secretkey456"

# Add DatabaseConnectionString secret
az keyvault secret set \
  --vault-name $KEY_VAULT_NAME \
  --name DatabaseConnectionString \
  --value "Server=prod-db.database.windows.net;Database=mydb;User Id=admin;Password=***"

# Verify secrets
az keyvault secret list \
  --vault-name $KEY_VAULT_NAME \
  --output table
```

**Output**:
```
Name                          Enabled
----------------------------  ---------
DatabaseConnectionString      True
ProductApiKey                 True
```

---

### Step 3: Create Azure DevOps Service Connection

**Azure DevOps UI**:

1. Navigate to **Project Settings** (bottom left)
2. Click **Service connections**
3. Click **New service connection**
4. Select **Azure Resource Manager**
5. Select **Service principal (automatic)**
6. Configure connection:

| Field | Value |
|-------|-------|
| **Subscription** | Your Azure subscription |
| **Resource group** | `devops-rg` |
| **Service connection name** | `KeyVault-Connection` |
| **Grant access permission to all pipelines** | ☑️ Checked |

7. Click **Save**

**Screenshot Placeholder**: Service connection creation dialog

---

### Step 4: Link Key Vault to Variable Group

**Azure DevOps UI**:

1. Navigate to **Pipelines** → **Library**
2. Click **+ Variable group**
3. Configure variable group:

| Field | Value |
|-------|-------|
| **Name** | `Production-Secrets-KeyVault` |
| **Description** | `Production secrets from Azure Key Vault` |
| **Link secrets from an Azure key vault** | ☑️ **Checked** |

4. Configure Key Vault connection:

| Field | Value |
|-------|-------|
| **Azure subscription** | `KeyVault-Connection` (from dropdown) |
| **Key vault name** | `myapp-keyvault-a1b2c3d4` (auto-populated) |

5. Click **+ Add** and select secrets to import:
   - ☑️ **ProductApiKey**
   - ☑️ **DatabaseConnectionString**

6. Click **Authorize** (if prompted)
7. Click **Save**

**Screenshot Placeholder**: Variable group with Key Vault integration enabled

---

### Step 5: Grant Azure DevOps Access to Key Vault

**Azure CLI**:
```bash
# Get service principal object ID from Azure DevOps connection
# (Find in Project Settings → Service connections → KeyVault-Connection → Manage Service Principal)
SERVICE_PRINCIPAL_ID="<object-id-from-azure-devops>"

# Grant access policy
az keyvault set-policy \
  --name $KEY_VAULT_NAME \
  --object-id $SERVICE_PRINCIPAL_ID \
  --secret-permissions get list
```

**Verification**:
```bash
# Test access
az keyvault secret show \
  --vault-name $KEY_VAULT_NAME \
  --name ProductApiKey \
  --query "value" \
  --output tsv
```

**Output**: `abc123xyz789secretkey456` ✅

---

### Step 6: Use Key Vault Variables in Pipeline

**Link Variable Group to Release**:

1. Edit release pipeline
2. Navigate to **Variables** tab
3. Click **Variable groups** → **Link variable group**
4. Select `Production-Secrets-KeyVault`
5. Scope to **Production** stage only
6. Click **Save**

**Add PowerShell Task**:

```powershell
# Access Key Vault-backed secrets
Write-Host "Connecting to production database..."
Write-Host "Database Connection: $(DatabaseConnectionString)"  # Masked as ***
Write-Host "API Key: $(ProductApiKey)"  # Masked as ***

# Use in database connection (environment variable method)
# Note: Secrets automatically available as environment variables
$connectionString = $env:DATABASE_CONNECTION_STRING
$apiKey = $env:PRODUCT_API_KEY

Write-Host "✅ Secrets loaded from Azure Key Vault"
```

**Environment Variables** (configure in task):
```yaml
env:
  DATABASE_CONNECTION_STRING: $(DatabaseConnectionString)
  PRODUCT_API_KEY: $(ProductApiKey)
```

---

### Step 7: Run Release and Verify

1. Create new release
2. Deploy to **Production** stage
3. Check logs:

**Expected Output**:
```
Connecting to production database...
Database Connection: ***  ← From Key Vault, automatically masked
API Key: ***  ← From Key Vault, automatically masked
✅ Secrets loaded from Azure Key Vault
```

**✅ Success Indicators**:
- Secrets retrieved from Key Vault
- Values automatically masked in logs
- No hardcoded secrets in pipeline YAML

---

## Exercise 6: Update Key Vault Secret and Auto-Sync

**Scenario**: API key rotated, need to update across all pipelines

---

### Step 1: Update Secret in Key Vault

```bash
# Rotate API key
az keyvault secret set \
  --vault-name $KEY_VAULT_NAME \
  --name ProductApiKey \
  --value "newkey999xyz888updated"
```

---

### Step 2: Trigger Variable Group Refresh

**Azure DevOps automatically syncs** Key Vault secrets, but you can manually trigger refresh:

1. Navigate to **Pipelines** → **Library**
2. Select `Production-Secrets-KeyVault` variable group
3. Click **Refresh** icon (🔄) next to Key Vault connection
4. Wait for refresh to complete (~30 seconds)

**Screenshot Placeholder**: Variable group with refresh button

---

### Step 3: Run Release with Updated Secret

1. Create new release
2. Deploy to **Production** stage
3. Verify new secret is used (check application behavior, not logs since it's masked)

**✅ Verification Method**: Application successfully authenticates with new API key

**Benefit**: Updated secret in **one place** (Key Vault) → propagates to **all pipelines** using the variable group!

---

## Troubleshooting

### Issue 1: Variable Not Recognized

**Symptom**: `$(ProductCode)` shows as literal string in logs

**Causes & Solutions**:

| Cause | Solution |
|-------|----------|
| Variable group not linked | Link variable group in pipeline Variables tab |
| Variable group scoped to wrong stage | Check scope settings, add current stage |
| Typo in variable name | Verify exact name (case-sensitive) |
| Variable group not authorized | Authorize variable group in pipeline settings |

---

### Issue 2: Secret Variable Visible in Logs

**Symptom**: Secret value appears in plain text

**Causes & Solutions**:

| Cause | Solution |
|-------|----------|
| Variable not marked as secret | Edit variable, click padlock icon 🔒 |
| Value echoed via environment variable | Use `Write-Host "***"` instead of `echo $SECRET` |
| Value printed indirectly | Check for `ConvertTo-Json` or similar commands |

---

### Issue 3: Key Vault Access Denied

**Symptom**: `Error: The user, group or application does not have secrets get permission`

**Solution**:
```bash
# Grant correct permissions
az keyvault set-policy \
  --name $KEY_VAULT_NAME \
  --object-id $SERVICE_PRINCIPAL_ID \
  --secret-permissions get list
```

---

### Issue 4: Variable Group Changes Not Reflected

**Symptom**: Pipeline still uses old variable values

**Causes & Solutions**:

| Cause | Solution |
|-------|----------|
| Changes not saved | Click **Save** button in variable group |
| Pipeline using cached values | Create **new release** (don't re-deploy existing) |
| Key Vault not synced | Click **Refresh** in variable group settings |
| Stage variable overriding | Check stage-level variables for overrides |

---

## Best Practices Demonstrated

### ✅ 1. Centralized Configuration
- Created **single variable group** for product details
- Linked to **multiple stages** without duplication

### ✅ 2. Environment Scoping
- Variable group scoped to **non-production** stages
- Production secrets stored in **Azure Key Vault** (separate group)

### ✅ 3. Secret Management
- Used **secret variables** for sensitive data
- Integrated **Azure Key Vault** for enterprise secrets
- Verified **automatic masking** in logs

### ✅ 4. Override Flexibility
- **Stage variables** override variable group values
- Enables **environment-specific configuration** without duplication

### ✅ 5. Maintainability
- Update variable group → changes propagate to all pipelines
- Key Vault updates → automatic sync to variable groups

---

## Key Takeaways

- 📚 **Variable groups** centralize configuration across pipelines
- 🔒 **Secret variables** automatically mask sensitive data in logs
- 🔐 **Azure Key Vault** provides enterprise-grade secret management
- 🎯 **Scoping** controls variable group availability per stage
- 🔄 **Auto-sync** propagates Key Vault changes to pipelines
- ⚖️ **Precedence** allows stage variables to override variable groups
- 📊 **Access control** limits who can view/edit variable groups

---

## Challenge Exercises (Optional)

### Challenge 1: Multi-Environment Variable Groups

**Task**: Create separate variable groups for Dev, Test, and Prod environments

**Requirements**:
- 3 variable groups: `Azure-Dev`, `Azure-Test`, `Azure-Prod`
- Each contains: `azureSubscription`, `resourceGroup`, `location`
- Scope each group to appropriate stages
- Verify correct values used in each stage

---

### Challenge 2: Database Connection Rotation

**Task**: Implement automatic database connection string rotation

**Requirements**:
- Store connection string in Azure Key Vault
- Create Azure Function to rotate secret every 30 days
- Link Key Vault to variable group
- Verify pipelines use updated connection strings automatically

---

### Challenge 3: Audit Variable Usage

**Task**: Track which pipelines use specific variable groups

**Requirements**:
- Navigate to **Pipelines** → **Library**
- Select variable group
- Click **Security** tab → **Pipeline permissions**
- Document all pipelines with access
- Review and revoke unnecessary access

---

## Next Steps

✅ **Completed**: Variable group creation, linking, scoping, and Key Vault integration

**Continue to**: Unit 5 - Knowledge check (test your understanding)

---

## Additional Resources

- [Variable groups documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups)
- [Azure Key Vault integration](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/azure-key-vault)
- [Secrets management best practices](https://learn.microsoft.com/en-us/azure/devops/pipelines/security/secrets)
- [Variable group security](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups#security)

[↩️ Back to Module Overview](01-introduction.md) | [⬅️ Previous: Variables in Pipelines](03-explore-variables-release-pipelines.md) | [➡️ Next: Knowledge Check](05-knowledge-check.md)
