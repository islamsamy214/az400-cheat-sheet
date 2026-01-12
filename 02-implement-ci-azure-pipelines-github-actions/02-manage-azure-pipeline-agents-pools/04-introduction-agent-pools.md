# Introduction to Agent Pools

## Key Concepts
- **Agent pool** - Organizational boundary for sharing agents
- **Organization-level pools** - Shared across all projects
- **Project-level pools** - Limited to single project
- **Sharing mechanism** - Explicit adding of pools to projects
- **Automatic permissions** - Simplify pipeline access to pools

## Agent Pool Fundamentals

Instead of managing each agent individually, you **organize agents into agent pools**. An agent pool defines the sharing boundary for all agents in that pool.

```yaml
# Reference agent pool in pipeline
pool:
  name: 'MyAgentPool'  # Use specific agent pool

# Or use default Microsoft-hosted pool
pool:
  vmImage: 'ubuntu-latest'
```

## Agent Pool Scope

### Organization-Level Pools
```
Organization
├── Agent Pool 1 (org-wide)
│   ├── Agent A
│   ├── Agent B
│   └── Agent C
└── Agent Pool 2 (org-wide)
    ├── Agent D
    └── Agent E
```

**Characteristics**:
- Scoped to **entire organization**
- Can be shared across **multiple projects**
- Created and managed at organization level
- Agents added to org pool available to all assigned projects

**Benefits**:
- ✅ Centralized agent management
- ✅ Share resources across projects
- ✅ Reduce infrastructure duplication
- ✅ Consistent agent configurations

### Project-Level Pools
```
Project A
├── Agent Pool X (project-only)
│   ├── Agent 1
│   └── Agent 2
```

**Characteristics**:
- **Limited to single project** unless explicitly added to others
- Created within project scope
- Must be explicitly shared to other projects
- Provides project isolation when needed

**When to Use**:
- Project-specific compliance requirements
- Dedicated resources for critical projects
- Security isolation needs
- Specialized tools for single project

## Creating and Sharing Pools

### Share Agent Pool Across Projects

**Step-by-step process**:

1. **Create organization-scoped agent pool**
   ```
   Organization Settings → Agent pools → New agent pool
   - Name: "Production-Agents"
   - Grant access to all pipelines: ✓ (optional)
   ```

2. **Add pool to each project that needs access**
   ```
   Project Settings → Agent pools → Add pool
   - Choose: "Existing organization pool"
   - Select: "Production-Agents"
   ```

3. **Choose organization agent pool when adding existing pool**
   - Projects now reference same underlying agent pool
   - Agents available to all assigned projects

### Automatic Permissions

```yaml
# When creating new agent pool, can automatically grant access
# This simplifies management for teams

# Pipeline automatically uses pool if permissions granted
pool:
  name: 'Production-Agents'
steps:
- script: echo "Running on shared agent pool"
```

**Automatic Permission Options**:
| Option | Effect |
|--------|--------|
| **Grant access to all pipelines** | All pipelines can use pool immediately |
| **Restrict access** | Require explicit pipeline authorization |
| **Auto-provision** | Create project pools in all projects automatically |

## Pool Sharing Scenarios

### Scenario 1: Central Infrastructure Team
```
Organization (Central IT)
└── Agent Pool: "Enterprise-Agents"
    → Project A ✓
    → Project B ✓
    → Project C ✓
    → All projects can use same agents
```

**Benefits**:
- Central management and maintenance
- Cost optimization through sharing
- Consistent configurations
- Easier capacity planning

### Scenario 2: Project-Specific Pools
```
Project X
└── Agent Pool: "Project-X-Only"
    → Only Project X can access
    → Isolated resources
    → Dedicated capacity
```

**Benefits**:
- Security isolation
- Guaranteed resources
- Project-specific configurations
- Compliance requirements

### Scenario 3: Hybrid Approach
```
Organization
├── Agent Pool: "Shared-Build-Agents" (all projects)
├── Agent Pool: "Production-Deploy" (selected projects)
└── Agent Pool: "ProjectA-Exclusive" (Project A only)
```

**Benefits**:
- Flexibility for different needs
- Optimize resource allocation
- Balance sharing with isolation

## Agent Pool Management

### Creating New Pool (Organization)
```bash
# Via Azure DevOps CLI
az pipelines pool create \
  --name "NewAgentPool" \
  --auto-provision true
```

**Configuration Options**:
- **Name** - Descriptive pool name
- **Auto-provision** - Create project pools automatically
- **Grant access** - Allow all pipelines
- **Description** - Pool purpose and usage notes

### Adding Agents to Pool
```bash
# Configure agent to join pool
# During agent configuration:
./config.sh
# Enter pool name: NewAgentPool
# Agent will register to specified pool
```

## Pool Reference in Pipelines

### Specify Pool Name
```yaml
# Use named pool
pool:
  name: 'MyAgentPool'

stages:
- stage: Build
  pool:
    name: 'Build-Agents'  # Different pool per stage
  jobs:
  - job: CompileJob
    steps:
    - script: build.sh

- stage: Deploy
  pool:
    name: 'Deploy-Agents'  # Use deployment-specific pool
  jobs:
  - deployment: DeployJob
    steps:
    - script: deploy.sh
```

### Multiple Pools in Pipeline
```yaml
# Different pools for different jobs
jobs:
- job: LinuxBuild
  pool:
    name: 'Linux-Agents'
  steps:
  - script: ./build-linux.sh

- job: WindowsBuild
  pool:
    name: 'Windows-Agents'
  steps:
  - script: build-windows.bat

- job: Deploy
  pool:
    name: 'Production-Agents'
  steps:
  - script: ./deploy.sh
```

## Best Practices

### Pool Organization
- **Descriptive names** - Use clear, purpose-driven names
- **Consistent naming** - Follow organization naming conventions
- **Documentation** - Document pool purpose and usage
- **Lifecycle management** - Regularly review and clean up unused pools

### Sharing Strategy
- **Start with organization pools** - Easier to manage
- **Share by default** - Unless security requires isolation
- **Project-specific when needed** - For compliance or security
- **Review access** - Regular audits of pool permissions

## Critical Notes
- 🎯 Agent pools define sharing boundary for agents
- 💡 Organization-level pools can be shared across multiple projects
- ⚠️ Project-level pools are limited to single project unless explicitly shared
- 📊 Automatic permissions simplify pipeline access to pools
- 🔄 When creating new pool, can grant access to all pipelines automatically
- ✨ Organize agents into pools rather than managing individually

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-azure-pipeline-agents-pools/4-introduction-to-agent-pools)
