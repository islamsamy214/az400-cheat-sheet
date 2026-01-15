# Module Assessment

Test your Bicep knowledge with these scenario-based questions.

## Question 1: Bicep vs ARM Templates

**Scenario**: Your team maintains 50+ ARM templates (JSON) totaling 15,000 lines. Developers complain about:
- Verbose syntax difficult to read
- Manual dependency management causing deployment failures
- No IntelliSense support

**Question**: What is the PRIMARY benefit of migrating to Bicep?

A. Multi-cloud support  
B. Simpler syntax with automatic dependency management  
C. Faster deployment times  
D. Built-in state management  

**Answer**: **B. Simpler syntax with automatic dependency management**

**Explanation**: Bicep reduces code by 50-70%, automatically infers dependencies from resource references, and provides excellent VS Code IntelliSense. It does NOT support multi-cloud (Azure only) or have state management (Azure is source of truth).

---

## Question 2: Automatic Dependencies

**Question**: When does Bicep automatically create dependencies between resources?

A. When you use `dependsOn` keyword  
B. When resources reference each other's properties  
C. When resources are in the same template  
D. Only when using modules  

**Answer**: **B. When resources reference each other's properties**

**Example**:
```bicep
resource plan 'Microsoft.Web/serverfarms@2021-02-01' = { ... }

resource webapp 'Microsoft.Web/sites@2021-02-01' = {
  properties: {
    serverFarmId: plan.id  // Automatic dependency!
  }
}
```

---

## Question 3: Modules

**Question**: What is the main purpose of Bicep modules?

A. Deploy to multiple subscriptions  
B. Code reuse and logical separation  
C. Improve deployment performance  
D. Enable conditional deployment  

**Answer**: **B. Code reuse and logical separation**

Modules allow you to write reusable Bicep files that can be referenced from other templates, following DRY principle.

---

## Question 4: Decorators

**Question**: Which decorator validates that a string parameter is between 3 and 24 characters?

A. `@length(3, 24)`  
B. `@validate(3, 24)`  
C. `@minLength(3) @maxLength(24)`  
D. `@range(3, 24)`  

**Answer**: **C. @minLength(3) @maxLength(24)**

```bicep
@minLength(3)
@maxLength(24)
param storageAccountName string
```

---

## Question 5: Deployment Scope

**Question**: What is the default deployment scope in Bicep?

A. Subscription  
B. Resource Group  
C. Management Group  
D. Tenant  

**Answer**: **B. Resource Group**

Change with `targetScope = 'subscription'` at top of file.

---

## Question 6: Transpilation

**Question**: What happens when you deploy a Bicep file?

A. Bicep directly creates Azure resources  
B. Bicep transpiles to ARM template, then ARM deploys  
C. Bicep compiles to executable, then deploys  
D. Bicep sends API calls directly to Azure  

**Answer**: **B. Bicep transpiles to ARM template, then ARM deploys**

Bicep is a DSL that converts to JSON ARM templates. Azure Resource Manager then deploys the ARM template.

---

## Question 7: Loops

**Question**: How do you create 5 storage accounts with a loop?

A. `for (int i = 0; i < 5; i++)`  
B. `[for i in range(0, 5): { ... }]`  
C. `foreach item in [1,2,3,4,5]`  
D. `loop count=5 { ... }`  

**Answer**: **B. [for i in range(0, 5): { ... }]**

```bicep
resource storage 'Microsoft.Storage/storageAccounts@2021-04-01' = [for i in range(0, 5): {
  name: 'storage${i}${uniqueString(resourceGroup().id)}'
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}]
```

---

## Question 8: CI/CD Integration

**Scenario**: You want to deploy Bicep templates automatically when code is pushed to the main branch, with validation before deployment.

**Question**: Which approach is BEST?

A. Manual deployment via Azure Portal  
B. Azure CLI script run locally  
C. Azure Pipelines with validation and deployment stages  
D. PowerShell script in Task Scheduler  

**Answer**: **C. Azure Pipelines with validation and deployment stages**

CI/CD pipelines provide:
- Automated validation
- What-If preview
- Staged deployments
- Manual approval gates
- Audit trail

---

## Score Interpretation

- **8/8**: Expert - Ready for production Bicep
- **6-7/8**: Proficient - Review missed concepts
- **4-5/8**: Intermediate - Practice more
- **0-3/8**: Beginner - Revisit module

## Key Concepts Review

1. **Simpler syntax**: 50-70% less code than ARM
2. **Automatic dependencies**: Inferred from references
3. **Modules**: Reusable template files
4. **Decorators**: Validate parameters
5. **Transpilation**: Bicep → ARM → Azure
6. **Type safety**: Design-time validation
7. **Loops/conditionals**: Advanced deployment patterns
8. **CI/CD**: Automate with Pipelines/GitHub Actions

---

**Module**: Implement Bicep  
**Unit**: 9 of 10  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-bicep/9-knowledge-check
