# Configure Agent Demands

Agent demands ensure pipeline jobs run on agents with required capabilities. Understanding how to configure demands prevents build failures due to missing dependencies.

## What Are Agent Capabilities?

**Capabilities** = Name-value pairs describing what an agent can do

### System Capabilities

Automatically discovered by agent:
- Operating system (`Agent.OS = Windows_NT`, `Linux`, `Darwin`)
- Machine name (`Agent.MachineName`)
- Agent name (`Agent.Name`)
- Installed software (Node.js, Python, Docker, etc.)

### User-Defined Capabilities

Custom capabilities you define:
- Custom tools (`HasPaymentService = true`)
- Environment identifiers (`Environment = Production`)
- Special configurations (`GPUEnabled = true`)

## Viewing Agent Capabilities

**Navigate to Organization Settings**:
```
Organization Settings → Agent Pools → [Select Pool] → [Select Agent] → Capabilities tab
```

**What You'll See**:
| Type | Examples | Purpose |
|------|----------|---------|
| **System** | `Agent.OS`, `Agent.Version`, `java`, `npm`, `docker` | Auto-detected software/OS |
| **User-Defined** | `HasSpecialTool`, `DatabaseAccess`, Custom values | Your custom requirements |

## Configuring Agent Demands

### In Pipeline Definition

**YAML Syntax**:
```yaml
# Demand agent with specific OS
pool:
  demands:
  - Agent.OS -equals Windows_NT

# Demand agent with Docker installed
pool:
  demands:
  - docker

# Multiple demands (AND logic)
pool:
  demands:
  - Agent.OS -equals Linux
  - docker
  - java
```

## Types of Demand Conditions

### 1. Exists Demand

**Syntax**: `-<CapabilityName>` or just `<CapabilityName>`

Requires agent to have the capability, regardless of value.

```yaml
demands:
- docker                    # Agent must have 'docker' capability
- HasPaymentService         # Agent must have 'HasPaymentService' capability
```

### 2. Equals Demand

**Syntax**: `<CapabilityName> -equals <Value>`

Requires capability to match specific value.

```yaml
demands:
- Agent.OS -equals Windows_NT     # Must be Windows
- java -equals 11.0                # Must have Java 11
- Environment -equals Production   # Custom capability match
```

## Practical Examples

### Example 1: Windows with .NET

```yaml
trigger:
- main

pool:
  vmImage: 'windows-latest'
  demands:
  - Agent.OS -equals Windows_NT
  - msbuild
  - visualstudio

steps:
- task: NuGetToolInstaller@1
- task: NuGetCommand@2
- task: VSBuild@1
```

### Example 2: Linux with Docker

```yaml
pool:
  name: 'Default'
  demands:
  - Agent.OS -equals Linux
  - docker

steps:
- script: docker build -t myapp:latest .
- script: docker push myapp:latest
```

### Example 3: Custom Capability

```yaml
# Requires agent with database access
pool:
  name: 'Production-Pool'
  demands:
  - HasDatabaseAccess -equals true
  - Environment -equals Production

steps:
- script: ./run-db-migration.sh
```

## Adding User-Defined Capabilities

### Via Azure DevOps UI

```
1. Organization Settings → Agent Pools
2. Select pool → Select agent
3. Capabilities tab → Add a user capability
4. Name: HasPaymentService
5. Value: true
6. Save
```

### Self-Hosted Agent Configuration

**Edit .agent file** or use environment variables:

```bash
# Linux/macOS
export VSTS_AGENT_INPUT_CAPABILITY_HasPaymentService=true

# Windows
set VSTS_AGENT_INPUT_CAPABILITY_HasPaymentService=true
```

## Demand Matching Logic

### AND Logic (Multiple Demands)

All demands must be satisfied:

```yaml
demands:
- docker          # AND
- java            # AND  
- Agent.OS -equals Linux  # AND

# Agent must have ALL three
```

### OR Logic (Multiple Jobs)

Use separate jobs for OR logic:

```yaml
jobs:
- job: WindowsBuild
  pool:
    demands:
    - Agent.OS -equals Windows_NT
  steps:
  - script: echo "Windows build"

- job: LinuxBuild
  pool:
    demands:
    - Agent.OS -equals Linux
  steps:
  - script: echo "Linux build"
```

## Common Use Cases

| Scenario | Demand Configuration | Benefit |
|----------|---------------------|---------|
| **OS-Specific Builds** | `Agent.OS -equals Windows_NT` | Ensure Windows-only tools available |
| **Container Builds** | `docker` | Require Docker engine on agent |
| **Database Migrations** | `HasDatabaseAccess -equals true` | Route to agents with DB connectivity |
| **GPU Workloads** | `GPUEnabled -equals true` | Use agents with GPU hardware |
| **Production Deployments** | `Environment -equals Production` | Isolate prod-capable agents |

## Troubleshooting

### Issue: "No agents match demands"

**Symptoms**: Pipeline queues indefinitely, log shows "waiting for agent"

**Solutions**:
1. Verify agent has required capabilities (check Capabilities tab)
2. Check demand spelling/capitalization (case-sensitive)
3. Add custom capability to agent if missing
4. Use less restrictive demands

### Issue: Wrong agent selected

**Problem**: Job runs on unexpected agent

**Solution**: Add more specific demands to narrow selection

```yaml
# Too broad
demands:
- docker

# More specific
demands:
- docker
- Agent.OS -equals Linux
- Agent.Name -equals BuildAgent-01
```

## Best Practices

1. **Document Custom Capabilities**: Maintain list of user-defined capabilities and their purposes
2. **Use Specific Demands**: Avoid vague demands that could match unintended agents
3. **Test Demands**: Verify demands work before production use
4. **Minimize Demands**: Only require truly necessary capabilities to maximize agent pool
5. **Standardize Names**: Use consistent naming conventions across organization

## Critical Notes

- 🎯 **Demands = Agent selection criteria** - Ensures jobs run on agents with required capabilities; prevents "missing tool" build failures
- 💡 **System capabilities auto-detected** - OS, installed software automatically registered; user-defined for custom requirements
- ⚠️ **Case-sensitive matching** - `docker` ≠ `Docker`; verify exact capability names in agent's Capabilities tab
- 📊 **AND logic for multiple demands** - All demands must be satisfied; use separate jobs for OR logic
- 🔄 **Overly restrictive demands = queuing** - Too many/specific demands may result in no matching agents; balance specificity with availability
- ✨ **Custom capabilities enable routing** - Add `Environment=Production` to specific agents; route prod deployments to secure agents only

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-pipeline-strategy/2-configure-agent-demands)
