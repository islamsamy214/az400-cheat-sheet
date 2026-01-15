# What is Bicep?

Azure Bicep is the next revision of ARM templates designed to solve issues developers face when deploying resources to Azure. It's an open-source, domain-specific language (DSL) that provides a declarative means to define infrastructure, describing cloud resource topology such as VMs, web apps, and networks. Bicep encourages code reuse and modularity in infrastructure as code.

## What is Bicep?

**Bicep** = Better Infrastructure as Code EP (Evolving Platform)

**Definition**: A declarative language for deploying Azure resources that transpiles to ARM templates, providing improved syntax while maintaining 100% ARM template capabilities.

**Key characteristics**:
- **Open source**: GitHub repository with active community
- **Declarative**: Describe *what* you want, not *how* to create it
- **Azure-native**: First-party Azure support, not third-party
- **Transpiles to ARM**: Converts to JSON ARM templates at deployment
- **Day-0 support**: New Azure features available immediately

## How Bicep Works

```
┌─────────────┐       ┌──────────────┐       ┌────────────────┐
│   Bicep     │       │   Bicep CLI  │       │  ARM Template  │
│  Template   │ ───▶  │  (Transpile) │ ───▶  │     (JSON)     │
│  (.bicep)   │       │              │       │                │
└─────────────┘       └──────────────┘       └────────────────┘
                                                       │
                                                       ▼
                                              ┌────────────────┐
                                              │     Azure      │
                                              │   Deployment   │
                                              └────────────────┘
```

**Process**:
1. Write infrastructure in Bicep (.bicep files)
2. Bicep CLI converts to ARM template (JSON)
3. Azure Resource Manager deploys resources
4. No runtime overhead—standard ARM deployment

## Benefits of Bicep

### 1. Simpler Syntax

**The Problem**: ARM templates are verbose and difficult to read.

**ARM Template** (JSON):
```json
{
  "type": "Microsoft.Storage/storageAccounts",
  "apiVersion": "2021-04-01",
  "name": "[parameters('storageName')]",
  "location": "[resourceGroup().location]",
  "sku": {
    "name": "[parameters('storageSKU')]"
  },
  "kind": "StorageV2",
  "properties": {
    "supportsHttpsTrafficOnly": true
  }
}
```

**Bicep**:
```bicep
resource storage 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: storageName
  location: resourceGroup().location
  sku: { name: storageSKU }
  kind: 'StorageV2'
  properties: { supportsHttpsTrafficOnly: true }
}
```

**Result**: 50-70% less code, cleaner syntax, easier maintenance.

### 2. Automatic Dependency Management

**The Problem**: ARM templates require manual `dependsOn` arrays.

**ARM Template**:
```json
{
  "type": "Microsoft.Web/sites",
  "dependsOn": [
    "[resourceId('Microsoft.Web/serverfarms', parameters('planName'))]"
  ]
}
```

**Bicep**:
```bicep
resource appServicePlan 'Microsoft.Web/serverfarms@2021-02-01' = { ... }

resource webApp 'Microsoft.Web/sites@2021-02-01' = {
  name: webAppName
  properties: {
    serverFarmId: appServicePlan.id  // Automatic dependency!
  }
}
```

**Result**: Bicep infers dependencies from resource references. No manual `dependsOn` needed.

### 3. Bicep CLI

The Bicep command-line interface provides powerful capabilities:

**Installation**:
```bash
# Via Azure CLI (recommended)
az bicep install

# Standalone installation
# Windows: choco install bicep
# Linux: curl -Lo bicep https://github.com/Azure/bicep/releases/latest/download/bicep-linux-x64
# macOS: brew install bicep
```

**Key commands**:
```bash
# Build Bicep to ARM template
az bicep build --file main.bicep

# Deploy Bicep directly
az deployment group create --template-file main.bicep

# Decompile ARM template to Bicep
az bicep decompile --file template.json

# Upgrade Bicep CLI
az bicep upgrade

# Verify installation
az bicep version
```

**Decompilation note**: Converting ARM to Bicep may have issues—some features not fully supported yet.

### 4. Visual Studio Code Integration

Excellent VS Code extension provides enhanced authoring experience:

**Key features**:
- **IntelliSense**: Auto-completion for resource types and properties
  - Type `resource` and get full syntax completion
  - See all properties for `Microsoft.Storage/storageAccounts`
  - Parameter and variable suggestions
  
- **Validation**: Real-time syntax and type validation
  - Red squiggles for errors
  - Yellow warnings for best practices
  - Validates API versions and property names
  
- **Snippets**: Pre-built templates for common resources
  - Type `res-` for resource snippets
  - `param-` for parameter templates
  - `output-` for output patterns
  
- **Visualization**: Graphical representation of resource dependencies
  - View dependency graph
  - Identify circular dependencies
  - Understand deployment order

**Installation**:
1. Open VS Code
2. Extensions → Search "Bicep"
3. Install "Bicep" by Microsoft
4. Create `.bicep` file → Automatic language support

**Extension benefits**:
- Faster authoring (50% time savings reported)
- Fewer errors (type validation catches issues)
- Better understanding (visualization helps learning)
- Team consistency (linting unifies style)

### 5. Type Safety

**ARM templates**: Runtime errors (deployment fails)

**Bicep**: Design-time validation (errors before deployment)

**Example**:
```bicep
// Type error caught immediately in VS Code
param storageCount int = 'five'  // ❌ Error: Cannot assign string to int

// Property validation
resource storage 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  sku: { name: 'InvalidSKU' }  // ❌ Error: InvalidSKU not valid SKU
}
```

### 6. Modularity and Reusability

**Modules** enable code reuse across templates:

**storage.bicep** (reusable module):
```bicep
param name string
param location string = resourceGroup().location

resource storage 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: name
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}

output storageId string = storage.id
```

**main.bicep** (uses module):
```bicep
module devStorage './storage.bicep' = {
  name: 'devStorageDeploy'
  params: { name: 'devstore123' }
}

module prodStorage './storage.bicep' = {
  name: 'prodStorageDeploy'
  params: { name: 'prodstore456' }
}
```

**Result**: Write once, use everywhere. DRY (Don't Repeat Yourself) principle.

### 7. No State Management

Unlike Terraform, Bicep doesn't require state files:
- **No state file**: Azure is the source of truth
- **No state drift**: Azure tracks actual vs desired state
- **No locking issues**: No concurrent state file conflicts
- **Simpler operations**: Fewer failure points

### 8. Day-0 Azure Feature Support

**The advantage**: New Azure services available in Bicep immediately

**Process**:
1. Azure releases new service (e.g., Azure OpenAI)
2. Bicep supports it Day 0 (same day)
3. No waiting for provider updates

**Comparison**:
- **Terraform**: Wait weeks/months for provider update
- **Bicep**: Use immediately with latest API version

## Bicep vs Alternatives

| Feature | Bicep | Terraform | ARM Templates | Pulumi |
|---------|-------|-----------|---------------|--------|
| **Language** | DSL | HCL | JSON | TypeScript/Python/C#/Go |
| **Azure native** | ✅ Yes | ❌ No | ✅ Yes | ❌ No |
| **Multi-cloud** | ❌ Azure only | ✅ Yes | ❌ Azure only | ✅ Yes |
| **State file** | ❌ No | ✅ Yes | ❌ No | ✅ Yes |
| **Learning curve** | Easy | Medium | Hard | Medium-Hard |
| **Day-0 features** | ✅ Yes | ❌ Delayed | ✅ Yes | ❌ Delayed |
| **Type safety** | ✅ Strong | ⚠️ Weak | ❌ Runtime only | ✅ Strong |
| **Tooling** | ✅ Excellent | ✅ Good | ⚠️ Basic | ✅ Good |

**When to use Bicep**:
- ✅ Azure-only infrastructure
- ✅ Need latest Azure features immediately
- ✅ Want simple, clean syntax
- ✅ Prefer Azure-native tools
- ✅ Don't want state file management

**When to consider alternatives**:
- ❌ Multi-cloud deployments (AWS, GCP, Azure)
- ❌ Need existing Terraform modules
- ❌ Require programmatic logic (Pulumi)

## Real-World Impact

**Before Bicep** (ARM templates):
- 15,000 lines of JSON for infrastructure
- 2 hours to author simple template
- 30% deployment failure rate (dependency errors)
- Difficult to review in PRs
- Hard to maintain

**After Bicep**:
- 4,500 lines of Bicep (70% reduction)
- 45 minutes to author template (VS Code IntelliSense)
- 5% deployment failure rate (type validation)
- Easy to review (clean syntax)
- Simple maintenance (modules)

## Key Takeaways

- **Bicep** is Azure's native IaC language, improving on ARM templates
- **Simpler syntax**: 50-70% less code than ARM templates
- **Automatic dependencies**: Infers from resource references
- **Type safety**: Validates at design time, not runtime
- **Excellent tooling**: VS Code extension with IntelliSense, validation, visualization
- **Modularity**: Reusable modules for DRY principle
- **Day-0 support**: New Azure features available immediately
- **No state**: Azure is source of truth, simpler than Terraform
- **Transpiles to ARM**: Full ARM template compatibility

---

**Module**: Implement Bicep  
**Unit**: 2 of 10  
**Duration**: 2 minutes  
**Next**: [Unit 3: Install Bicep](#)  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-bicep/2-what-bicep  
**Reference**: https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview
