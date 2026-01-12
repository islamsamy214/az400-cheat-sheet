# Examine Other Considerations

## Key Concepts
- Personal Access Tokens (PAT) are the only authentication method for agent registration
- Agents can run as services (recommended) or interactive processes (for UI tests)
- Agent versions follow semantic versioning with automatic minor updates
- Self-hosted agents provide significant performance benefits over Microsoft-hosted
- Multiple agents per machine can cause resource contention

## Authentication Methods

### Personal Access Token (PAT)

**Only supported method** for Azure Pipelines agent registration:

```bash
# Register agent using PAT
./config.sh --url https://dev.azure.com/yourorg --auth pat --token YOUR_PAT_TOKEN

# PAT is used ONLY during registration, not for ongoing communication
# After registration, agent uses OAuth tokens for all operations
```

**PAT Requirements**:

| Permission Scope | Purpose | Access Level |
|------------------|---------|--------------|
| **Agent Pools** | Register/manage agents | Read & Manage |
| **Deployment Groups** | Register deployment agents | Read & Manage |
| **Project** | Access project resources | Read |

**Security Notes**:
- PAT stored securely after registration (encrypted)
- PAT not transmitted after initial setup
- Can revoke PAT without affecting registered agents
- Use short-lived PATs for registration, then rotate

### Administrative Requirements

**During Registration**:
- Must be member of **Administrator role** in the agent pool
- Must be **local administrator** on the agent server
- Administrator identity NOT stored on agent

**After Registration**:
- Agent runs with configured service account
- Service account needs permissions for build/deployment tasks
- No administrative privileges required for ongoing operations

## Agent Execution Modes

### Running as a Service (Recommended)

**Production Standard**:

```bash
# Linux/macOS - Install as systemd service
sudo ./svc.sh install

# Windows - Install as Windows service
.\config.cmd --runAsService --windowsLogonAccount "NT AUTHORITY\NetworkService"
```

**Advantages**:

| Benefit | Description |
|---------|-------------|
| **Reliability** | Continues running if user logs off |
| **Auto-start** | Starts automatically after system reboot |
| **Auto-upgrade** | Seamless agent updates without manual intervention |
| **OS Management** | Controlled through service manager (systemctl, sc.exe) |
| **Stability** | No user session dependencies |

**Best For**:
- ✅ Build jobs (compilation, unit tests, packaging)
- ✅ Deployment jobs (no UI interaction)
- ✅ Automated testing (API, integration, performance)
- ✅ Long-running processes

### Running as Interactive Process

**Special Scenarios Only**:

```bash
# Linux/macOS - Run interactively
./run.sh

# Windows - Configure auto-logon and interactive mode
.\config.cmd --runAsAutoLogon
```

**Requirements**:
- User must remain logged in
- Screen saver automatically disabled
- Computer must be physically secured
- May require domain policy exemptions

**Best For**:
- ✅ UI tests (Selenium, Coded UI, Appium)
- ✅ Desktop automation requiring GUI
- ✅ Video recording of test execution

**Security Risks**:
- ⚠️ Computer accessible to anyone with physical access
- ⚠️ Credentials visible in auto-logon configuration
- ⚠️ User session can be interrupted

### Choosing the Right Account

**Windows Authentication Tasks**:

```yaml
# Agent service account needs access to external resources
jobs:
- job: BuildWithExternalAccess
  pool:
    name: Windows-Pool
  steps:
  - script: |
      # Accesses network share using agent service account credentials
      copy \\fileserver\share\dependencies\* .
```

**UI Test Requirements**:

```yaml
# Browser launches in context of agent account
- task: VSTest@2
  inputs:
    testSelector: 'testAssemblies'
    testAssemblyVer2: |
      **\*CodedUI*.dll
      !**\*TestAdapter.dll
    # Requires interactive mode with logged-in user
```

## Agent Versions and Upgrades

### Version Format

**Semantic Versioning**: `{major}.{minor}.{patch}`

Example: `3.220.5` = Major 3, Minor 220, Patch 5

### Automatic Upgrade Behavior

| Update Type | Microsoft-Hosted | Self-Hosted (Service) | Self-Hosted (Interactive) |
|-------------|------------------|----------------------|---------------------------|
| **Minor Version** | Automatic | Automatic | Manual |
| **Major Version** | Automatic | Manual | Manual |
| **Patch** | Automatic | Automatic | Manual |

**Automatic Upgrade Triggers**:
```yaml
# Agent auto-upgrades when pipeline uses newer task version
- task: NodeTool@0
  inputs:
    versionSpec: '18.x'
# If NodeTool requires agent 3.x and you have 2.x, auto-upgrade occurs
```

### Version Management

**Check Agent Version**:

```bash
# Navigate to Azure DevOps
https://dev.azure.com/{organization}/_settings/agentpools

# Select pool → Select agent → Capabilities tab
# Look for: Agent.Version = 3.220.5
```

**Manual Upgrade**:

```bash
# Linux/macOS
cd /path/to/agent
./config.sh remove  # Unregister
# Download new version
./config.sh         # Re-register

# Windows
.\config.cmd remove
# Download new version
.\config.cmd
```

## Performance Considerations

### Self-Hosted Agent Advantages

**Incremental Builds**:

```yaml
# Microsoft-hosted: Always clean checkout
- checkout: self
  clean: true  # Required for Microsoft-hosted

# Self-hosted: Can preserve workspace
- checkout: self
  clean: false  # Reuse previous checkout
  fetchDepth: 1  # Shallow clone
```

**Performance Comparison**:

| Factor | Microsoft-Hosted | Self-Hosted |
|--------|------------------|-------------|
| **Build Time** | Slower (clean each time) | Faster (incremental builds) |
| **Agent Start** | 2-5 minutes provisioning | Instant (always available) |
| **Artifacts** | Lost after job | Preserved between runs |
| **Dependencies** | Re-download each time | Cached locally |
| **Custom Tools** | Not available | Pre-installed |

**Example Performance Gains**:

```yaml
# Self-hosted with dependency caching
steps:
- task: Cache@2
  inputs:
    key: 'npm | "$(Agent.OS)" | package-lock.json'
    path: $(npm_config_cache)
  displayName: 'Cache npm packages'
  # First run: 3 minutes to download dependencies
  # Subsequent runs: 10 seconds to restore from cache

- script: npm ci
  # Microsoft-hosted: Always 3 minutes
  # Self-hosted with cache: 10 seconds
```

### Microsoft-Hosted Agent Limitations

**Clean Environment**:
- ✅ **Advantage**: Consistent, reproducible builds
- ❌ **Disadvantage**: No build acceleration

**Allocation Delays**:
- ⚠️ Can take several minutes during high load
- ⚠️ Variable start times based on system demand
- ⚠️ No control over when agent becomes available

## Multiple Agents Per Machine

### Effective Scenarios

**Release Orchestration**:

```yaml
# Multiple lightweight deployment jobs
- job: DeployRegion1
  pool: Release-Pool
- job: DeployRegion2
  pool: Release-Pool
- job: DeployRegion3
  pool: Release-Pool
# Each job uses minimal CPU/memory
```

**When It Works**:
- ✅ Deployment coordination (low resource usage)
- ✅ Approval gates and manual interventions
- ✅ API calls and notifications
- ✅ Script execution with minimal processing

### Potential Issues

**Resource Contention**:

| Resource | Conflict Scenario | Impact |
|----------|------------------|--------|
| **Disk I/O** | Multiple builds writing simultaneously | Slower builds, timeouts |
| **CPU** | Parallel compilation jobs | Thrashing, failed builds |
| **Memory** | Multiple test suites running | Out of memory errors |
| **Network** | Simultaneous package downloads | Bandwidth saturation |

**Tool Conflicts**:

```bash
# Problem: Two agents installing same npm package globally
Agent1: npm install -g typescript@4.5
Agent2: npm install -g typescript@4.8
# Result: Unpredictable which version is active

# Problem: Port conflicts
Agent1: Starting test server on port 3000
Agent2: Starting test server on port 3000
# Result: Second agent fails with "port already in use"
```

**Best Practice**:

```yaml
# Use containers to isolate agents
pool:
  name: Default
  demands:
  - agent.name -equals Build-Server-01
container: node:18
# Each job runs in isolated container
```

## Critical Notes

- 🎯 **PAT only used for registration** - After setup, agents use OAuth tokens; safe to rotate PATs without re-registering agents
- 💡 **Run as service for production** - Interactive mode creates security risks; only use for UI testing scenarios
- ⚠️ **Automatic upgrades happen during jobs** - Minor version updates can occur mid-pipeline if newer task versions required
- 📊 **Self-hosted agents are faster** - Incremental builds, cached dependencies, and no provisioning time significantly reduce build duration
- 🔄 **Multiple agents need isolation** - Use containers or separate machines to avoid disk, memory, and tool conflicts
- ✨ **Check agent version regularly** - Navigate to Capabilities tab to verify agent version and ensure compatibility with pipeline requirements

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-azure-pipeline-agents-pools/9-examine-other-considerations)
