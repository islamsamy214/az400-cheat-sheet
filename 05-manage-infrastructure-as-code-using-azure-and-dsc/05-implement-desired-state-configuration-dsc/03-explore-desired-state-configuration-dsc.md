# Explore Desired State Configuration (DSC)

**Desired State Configuration (DSC)** is a configuration management approach that you can use for configuration, deployment, and management of systems to ensure that an environment is maintained in a state that you specify (defined state) and doesn't deviate from that state.

DSC helps eliminate configuration drift and ensures the state is maintained for compliance, security, and performance.

## What is Windows PowerShell DSC?

**Windows PowerShell DSC** is a management platform in PowerShell that provides desired state configuration capabilities. PowerShell DSC enables you to manage, deploy, and enforce configurations for physical or virtual machines, including **Windows and Linux systems**.

### Key Concepts

**Declarative vs Imperative**:

**Imperative (Traditional Scripting)**:
```powershell
# Tell the system HOW to do something (step-by-step)
if (-not (Get-WindowsFeature Web-Server).Installed) {
    Install-WindowsFeature -Name Web-Server
}
if (-not (Test-Path "C:\inetpub\wwwroot\index.html")) {
    New-Item -Path "C:\inetpub\wwwroot\index.html" -ItemType File
    Set-Content -Path "C:\inetpub\wwwroot\index.html" -Value "<h1>Hello</h1>"
}
$service = Get-Service W3SVC
if ($service.Status -ne "Running") {
    Start-Service W3SVC
}
```

**Problems with imperative**:
- ❌ Complex logic (checking current state)
- ❌ Not idempotent (may fail if run twice)
- ❌ No drift detection
- ❌ Error-prone

**Declarative (DSC)**:
```powershell
# Tell the system WHAT you want (desired state)
Configuration WebServerConfig {
    WindowsFeature IIS {
        Ensure = "Present"  # What you want
        Name = "Web-Server"
    }
    
    File HomePage {
        Ensure = "Present"
        DestinationPath = "C:\inetpub\wwwroot\index.html"
        Contents = "<h1>Hello</h1>"
    }
    
    Service W3SVC {
        Name = "W3SVC"
        State = "Running"
    }
}
```

**Benefits of declarative**:
- ✅ Simple syntax (just state what you want)
- ✅ Idempotent (safe to run multiple times)
- ✅ Automatic drift detection
- ✅ Self-healing

## DSC Components

DSC consists of **three primary components** that work together to enforce desired state:

### 1. Configurations

**Configurations** are declarative PowerShell scripts that define and configure instances of resources. Upon running the configuration, DSC (and the resources being called by the configuration) will apply the configuration, ensuring that the system exists in the state laid out by the configuration.

**DSC configurations are idempotent**: The Local Configuration Manager (LCM) will ensure that machines are configured in whatever state the configuration declares, regardless of how many times the configuration is applied.

**Example configuration**:
```powershell
Configuration MyWebServer {
    param(
        [string]$ServerName = "localhost",
        [string]$WebsiteName = "Default Web Site"
    )
    
    Import-DscResource -ModuleName PSDesiredStateConfiguration
    
    Node $ServerName {
        # Install IIS role
        WindowsFeature IIS {
            Ensure = "Present"
            Name = "Web-Server"
            IncludeAllSubFeature = $true
        }
        
        # Install ASP.NET 4.5
        WindowsFeature ASPNET45 {
            Ensure = "Present"
            Name = "Web-Asp-Net45"
            DependsOn = "[WindowsFeature]IIS"
        }
        
        # Create website directory
        File WebsiteContent {
            Ensure = "Present"
            Type = "Directory"
            DestinationPath = "C:\inetpub\wwwroot\myapp"
            DependsOn = "[WindowsFeature]IIS"
        }
        
        # Ensure IIS service is running
        Service W3SVC {
            Name = "W3SVC"
            State = "Running"
            StartupType = "Automatic"
            DependsOn = "[WindowsFeature]IIS"
        }
    }
}

# Generate MOF file
MyWebServer -ServerName "WebServer01"
```

**Configuration characteristics**:
- Written in PowerShell syntax
- Saved as `.ps1` files
- Version controlled (Git, Azure DevOps)
- Compiled to MOF files before application
- Support parameters for reusability
- Support dependencies between resources (`DependsOn`)

### 2. Resources

**Resources** contain the code that puts and keeps the target of a configuration in the specified state. Resources are in PowerShell modules and can be written to a model as generic as a file or a Windows process or as specific as a Microsoft Internet Information Services (IIS) server or a VM running in Azure.

**Built-in DSC Resources** (PSDesiredStateConfiguration module):

| Resource | Purpose | Example Use Case |
|----------|---------|------------------|
| **File** | Manage files and directories | Ensure config file exists |
| **Archive** | Extract .zip files | Deploy application package |
| **Environment** | Manage environment variables | Set JAVA_HOME |
| **Registry** | Manage registry keys/values | Disable SMBv1 protocol |
| **Script** | Run custom PowerShell | Complex custom logic |
| **Service** | Manage Windows services | Ensure IIS running |
| **User** | Manage local user accounts | Create service account |
| **Group** | Manage local groups | Add user to Administrators |
| **WindowsFeature** | Install/remove Windows features | Install IIS, .NET Framework |
| **WindowsProcess** | Manage processes | Ensure app.exe running |
| **Package** | Install/remove MSI packages | Install SQL Server |

**Example resource usage**:
```powershell
Configuration EnforceSecurityBaseline {
    Node WebServer {
        # Disable SMBv1 (security vulnerability)
        Registry DisableSMBv1 {
            Ensure = "Present"
            Key = "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters"
            ValueName = "SMB1"
            ValueData = "0"
            ValueType = "Dword"
        }
        
        # Ensure Guest account disabled
        User DisableGuest {
            UserName = "Guest"
            Disabled = $true
            Ensure = "Present"
        }
        
        # Ensure Windows Firewall enabled
        Service WindowsFirewall {
            Name = "MpsSvc"
            State = "Running"
            StartupType = "Automatic"
        }
        
        # Create local admin account
        User LocalAdmin {
            UserName = "AppAdmin"
            Password = (Get-AutomationPSCredential "AppAdminCred")
            Disabled = $false
            Ensure = "Present"
            PasswordChangeRequired = $false
            PasswordNeverExpires = $true
        }
        
        # Add to local Administrators group
        Group AddToAdmins {
            GroupName = "Administrators"
            Ensure = "Present"
            MembersToInclude = "AppAdmin"
            DependsOn = "[User]LocalAdmin"
        }
    }
}
```

**Community DSC Resources** (PowerShell Gallery):
- **xWebAdministration**: IIS configuration
- **xNetworking**: Network settings (firewall, IP config)
- **xSQLServer**: SQL Server configuration
- **xActiveDirectory**: Active Directory management
- **xComputerManagement**: Computer join domain, rename
- **xPendingReboot**: Detect pending reboots
- **xPSDesiredStateConfiguration**: Advanced DSC features

**Install community resources**:
```powershell
# Install from PowerShell Gallery
Install-Module -Name xWebAdministration -Force
Install-Module -Name xNetworking -Force
Install-Module -Name xSQLServer -Force

# Use in configuration
Configuration WebServerAdvanced {
    Import-DscResource -ModuleName xWebAdministration
    
    Node WebServer {
        xWebsite DefaultSite {
            Ensure = "Present"
            Name = "Default Web Site"
            State = "Stopped"
            PhysicalPath = "C:\inetpub\wwwroot"
        }
        
        xWebsite MySite {
            Ensure = "Present"
            Name = "MySite"
            State = "Started"
            PhysicalPath = "C:\websites\mysite"
            BindingInfo = @(
                MSFT_xWebBindingInformation {
                    Protocol = "HTTP"
                    Port = 8080
                    IPAddress = "*"
                }
            )
            DependsOn = "[xWebsite]DefaultSite"
        }
    }
}
```

### 3. Local Configuration Manager (LCM)

The **Local Configuration Manager (LCM)** runs on the nodes or machines you wish to configure. It's the engine by which DSC facilitates the interaction between resources and configurations. The LCM regularly polls the system using the control flow implemented by resources to maintain the state defined by a configuration. If the system is out of state, the LCM calls the code in resources to apply the configuration according to specified.

**LCM Architecture**:
```
┌─────────────────────────────────────────┐
│         Target Node (Server)            │
│  ┌───────────────────────────────────┐  │
│  │  Local Configuration Manager      │  │
│  │  (DSC Engine)                     │  │
│  │                                   │  │
│  │  ┌─────────────────────────────┐ │  │
│  │  │ Configuration Mode:         │ │  │
│  │  │ - ApplyOnly                 │ │  │
│  │  │ - ApplyAndMonitor           │ │  │
│  │  │ - ApplyAndAutoCorrect       │ │  │
│  │  └─────────────────────────────┘ │  │
│  │                                   │  │
│  │  ┌─────────────────────────────┐ │  │
│  │  │ Refresh Mode:               │ │  │
│  │  │ - Push (default)            │ │  │
│  │  │ - Pull                      │ │  │
│  │  └─────────────────────────────┘ │  │
│  │                                   │  │
│  │  ┌─────────────────────────────┐ │  │
│  │  │ Check Frequency:            │ │  │
│  │  │ - 15 minutes (default)      │ │  │
│  │  │ - Configurable              │ │  │
│  │  └─────────────────────────────┘ │  │
│  └───────────────────────────────────┘  │
│           ↓                              │
│  ┌───────────────────────────────────┐  │
│  │  DSC Resources                    │  │
│  │  - WindowsFeature                 │  │
│  │  - File                           │  │
│  │  - Service                        │  │
│  │  - Registry                       │  │
│  │  - etc.                           │  │
│  └───────────────────────────────────┘  │
│           ↓                              │
│  ┌───────────────────────────────────┐  │
│  │  Target System                    │  │
│  │  - IIS                            │  │
│  │  - Files/Directories              │  │
│  │  - Services                       │  │
│  │  - Registry Keys                  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

**LCM Configuration Modes**:

| Mode | Behavior | Use Case |
|------|----------|----------|
| **ApplyOnly** | Applies configuration once, no monitoring | One-time setup |
| **ApplyAndMonitor** | Applies config, monitors drift, reports issues (no fix) | Testing, audit mode |
| **ApplyAndAutoCorrect** | Applies config, monitors drift, automatically remediates | Production (default) |

**LCM Refresh Modes**:

| Mode | How Configurations Applied | Use Case |
|------|---------------------------|----------|
| **Push** | Admin manually pushes MOF to node | Small environments, testing |
| **Pull** | Node pulls MOF from pull server (automatic) | Large-scale production |

**Configure LCM**:
```powershell
# Configure LCM on local machine
[DSCLocalConfigurationManager()]
Configuration LCMConfig {
    Node localhost {
        Settings {
            # Configuration mode: Monitor and auto-remediate drift
            ConfigurationMode = "ApplyAndAutoCorrect"
            
            # Refresh mode: Pull from pull server
            RefreshMode = "Pull"
            
            # Check every 30 minutes (default: 15)
            RefreshFrequencyMins = 30
            
            # Reboot if needed after config apply
            RebootNodeIfNeeded = $true
            
            # Action after reboot: Continue configuration
            ActionAfterReboot = "ContinueConfiguration"
            
            # Allow module overwrite during pull
            AllowModuleOverwrite = $true
        }
        
        # Pull server configuration
        ConfigurationRepositoryWeb AzureAutomation {
            ServerURL = "https://automation.azure.com/..."
            RegistrationKey = "YOUR_REGISTRATION_KEY"
            ConfigurationNames = @("WebServerConfig")
        }
    }
}

# Apply LCM configuration
LCMConfig
Set-DscLocalConfigurationManager -Path .\LCMConfig -Verbose
```

**LCM Workflow**:
```
1. LCM reads configuration (MOF file)
   ↓
2. For each resource in configuration:
   a. Call resource's Test() method
      - Returns $true if compliant
      - Returns $false if drift detected
   b. If Test() returns $false:
      - Call resource's Set() method
      - Apply desired configuration
      - Remediate drift
   ↓
3. Report status (compliant/non-compliant)
   ↓
4. Wait for RefreshFrequencyMins (default: 15 minutes)
   ↓
5. Repeat from step 1
```

## DSC Implementation Methods

There are **two methods** of implementing DSC:

### Push Mode

**Push mode**: A user actively applies a configuration to a target node and pushes out the configuration. This method is useful for **testing** and **immediate configuration application**.

**How push mode works**:
```
┌─────────────────┐
│  Administrator  │
│   Workstation   │
└────────┬────────┘
         │ 1. Author configuration (.ps1)
         │ 2. Compile to MOF
         │ 3. Manually push to nodes
         ↓
┌─────────────────┐
│   Target Node   │
│      (VM)       │
│  ┌───────────┐  │
│  │    LCM    │  │ 4. Apply configuration
│  └───────────┘  │ 5. Monitor (if ApplyAndAutoCorrect)
└─────────────────┘
```

**Push mode workflow**:
```powershell
# Step 1: Create configuration
Configuration WebServerConfig {
    Node "Server01", "Server02", "Server03" {
        WindowsFeature IIS {
            Ensure = "Present"
            Name = "Web-Server"
        }
    }
}

# Step 2: Compile configuration (generates MOF files)
WebServerConfig -OutputPath "C:\DSC\Config"
# Creates: Server01.mof, Server02.mof, Server03.mof

# Step 3: Push configuration to nodes
Start-DscConfiguration -Path "C:\DSC\Config" -Wait -Verbose

# Step 4: Check compliance
Get-DscConfigurationStatus -CimSession "Server01", "Server02", "Server03"
```

**Push mode characteristics**:
- ✅ **Simple**: Easy to understand and implement
- ✅ **Immediate**: Configuration applied right away
- ✅ **Testing**: Great for dev/test environments
- ❌ **Manual**: Requires admin to push to each node
- ❌ **Not scalable**: Difficult with 100+ nodes
- ❌ **No central management**: Each node independent

**When to use push mode**:
- Development and testing environments
- Small environments (< 10 nodes)
- One-time configuration tasks
- Learning DSC fundamentals

### Pull Mode

**Pull mode**: Pull clients are automatically configured to get their desired state configurations from a remote pull service. This remote pull service is provided by a pull server that acts as central control and manager for the configurations, ensures that nodes conform to the desired state, and reports on their compliance status.

The pull server can be set up as an **SMB-based pull server** or an **HTTPS-based server**. HTTPS-based pull server uses the Open Data Protocol (OData) with the OData Web service to communicate using REST APIs. This is the **preferred model**, as it can be centrally managed and controlled.

**DSC pull mode workflow diagram**:

```
┌────────────────────────────────────────────────────────────────┐
│                     DSC Pull Server                            │
│                  (Azure Automation State Configuration)        │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Configuration Repository                                │  │
│  │  - WebServerConfig.mof                                   │  │
│  │  - DatabaseServerConfig.mof                              │  │
│  │  - AppServerConfig.mof                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Compliance Database                                     │  │
│  │  - Node status (compliant/non-compliant)                 │  │
│  │  - Last check-in time                                    │  │
│  │  - Applied configuration version                         │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────┬────────────────────────────┬────────────────────────────┘
        │                            │
        │ HTTPS (443)                │ HTTPS (443)
        │ Pull config every 30 min   │ Pull config every 30 min
        ↓                            ↓
┌───────────────┐            ┌───────────────┐
│   Node 1      │            │   Node 2      │
│ ┌───────────┐ │            │ ┌───────────┐ │
│ │    LCM    │ │            │ │    LCM    │ │
│ └───────────┘ │            │ └───────────┘ │
│  WebServer    │            │  AppServer    │
└───────────────┘            └───────────────┘
        ↑                            ↑
        │ Check every 15 min         │ Check every 15 min
        │ Auto-remediate drift       │ Auto-remediate drift
        ↓                            ↓
    IIS, Files,                  App binaries,
    Registry                     Configuration
```

**Pull mode workflow**:
```
1. Administrator:
   a. Author configuration (.ps1)
   b. Upload to Azure Automation
   c. Compile to MOF (Azure does this)
   d. Assign configuration to nodes

2. Target Nodes (automatically, every 30 minutes):
   a. LCM contacts pull server
   b. Check for configuration updates
   c. Download new MOF (if available)
   d. Apply configuration
   e. Report compliance status back to pull server

3. LCM on each node (every 15 minutes):
   a. Check current state
   b. Compare to desired state (from MOF)
   c. If drift detected:
      - Remediate automatically
      - Report drift event to pull server
```

**Pull mode characteristics**:
- ✅ **Scalable**: Manages 1 to 10,000+ nodes
- ✅ **Centralized**: Single source of truth (pull server)
- ✅ **Automated**: Nodes pull configurations automatically
- ✅ **Reporting**: Centralized compliance dashboard
- ✅ **Version control**: Configuration history tracked
- ⚠️ **Complex**: More initial setup required
- ⚠️ **Pull server**: Requires pull server infrastructure

**When to use pull mode**:
- Production environments
- Large-scale deployments (50+ nodes)
- Centralized management required
- Compliance reporting needed
- Azure Automation State Configuration (recommended)

## DSC Workflow: Complete Example

**Scenario**: Configure 100 web servers with IIS, custom app, security baseline

### Step 1: Author Configuration
```powershell
Configuration WebServerCompleteSetup {
    param(
        [Parameter(Mandatory)]
        [string]$ComputerName,
        
        [Parameter(Mandatory)]
        [PSCredential]$AppPoolIdentity
    )
    
    Import-DscResource -ModuleName PSDesiredStateConfiguration
    Import-DscResource -ModuleName xWebAdministration
    
    Node $ComputerName {
        # Install IIS and features
        WindowsFeature IIS {
            Ensure = "Present"
            Name = "Web-Server"
            IncludeAllSubFeature = $true
        }
        
        WindowsFeature ASPNET45 {
            Ensure = "Present"
            Name = "Web-Asp-Net45"
            DependsOn = "[WindowsFeature]IIS"
        }
        
        # Disable default website
        xWebsite DefaultSite {
            Ensure = "Present"
            Name = "Default Web Site"
            State = "Stopped"
            PhysicalPath = "C:\inetpub\wwwroot"
            DependsOn = "[WindowsFeature]IIS"
        }
        
        # Create application directory
        File AppDirectory {
            Ensure = "Present"
            Type = "Directory"
            DestinationPath = "C:\WebApps\MyApp"
            DependsOn = "[WindowsFeature]IIS"
        }
        
        # Create custom website
        xWebsite MyAppSite {
            Ensure = "Present"
            Name = "MyApp"
            State = "Started"
            PhysicalPath = "C:\WebApps\MyApp"
            BindingInfo = @(
                MSFT_xWebBindingInformation {
                    Protocol = "HTTP"
                    Port = 80
                    IPAddress = "*"
                }
                MSFT_xWebBindingInformation {
                    Protocol = "HTTPS"
                    Port = 443
                    IPAddress = "*"
                    CertificateThumbprint = "CERT_THUMBPRINT"
                    CertificateStoreName = "My"
                }
            )
            DependsOn = @("[WindowsFeature]IIS", "[File]AppDirectory")
        }
        
        # Security: Disable SSLv3
        Registry DisableSSLv3 {
            Ensure = "Present"
            Key = "HKLM:\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server"
            ValueName = "Enabled"
            ValueData = "0"
            ValueType = "Dword"
        }
        
        # Security: Enable TLS 1.2
        Registry EnableTLS12 {
            Ensure = "Present"
            Key = "HKLM:\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server"
            ValueName = "Enabled"
            ValueData = "1"
            ValueType = "Dword"
        }
        
        # Ensure IIS service running
        Service W3SVC {
            Name = "W3SVC"
            State = "Running"
            StartupType = "Automatic"
            DependsOn = "[WindowsFeature]IIS"
        }
    }
}
```

### Step 2: Upload to Azure Automation
```powershell
# Upload configuration to Azure Automation
Import-AzAutomationDscConfiguration `
    -SourcePath "C:\DSC\WebServerCompleteSetup.ps1" `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod" `
    -Published
```

### Step 3: Compile Configuration
```powershell
# Compile configuration to MOF
$params = @{
    ComputerName = "WebServer"
    AppPoolIdentity = (Get-AutomationPSCredential "AppPoolCred")
}

Start-AzAutomationDscCompilationJob `
    -ConfigurationName "WebServerCompleteSetup" `
    -Parameters $params `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod"
```

### Step 4: Onboard Nodes
```powershell
# Onboard 100 Azure VMs to Azure Automation State Configuration
$vms = Get-AzVM -ResourceGroupName "rg-web-servers"

foreach ($vm in $vms) {
    Register-AzAutomationDscNode `
        -ResourceGroupName "rg-automation" `
        -AutomationAccountName "automation-prod" `
        -AzureVMName $vm.Name `
        -NodeConfigurationName "WebServerCompleteSetup.WebServer"
}
```

### Step 5: Monitor Compliance
```powershell
# Check node compliance status
Get-AzAutomationDscNode `
    -ResourceGroupName "rg-automation" `
    -AutomationAccountName "automation-prod" |
    Select-Object Name, Status, LastSeen, @{
        Name = "ConfigurationName"
        Expression = {$_.NodeConfiguration.Name}
    }

# Output:
# Name         Status      LastSeen            ConfigurationName
# ----         ------      --------            -----------------
# WebServer01  Compliant   2024-01-15 10:30    WebServerCompleteSetup.WebServer
# WebServer02  Compliant   2024-01-15 10:28    WebServerCompleteSetup.WebServer
# WebServer03  Pending     2024-01-15 10:25    WebServerCompleteSetup.WebServer
# ...
```

## Key Takeaways

1. **DSC is declarative**: Describe WHAT you want, not HOW to achieve it
2. **Three core components**: Configurations (scripts), Resources (building blocks), LCM (engine)
3. **Idempotent**: Safe to run multiple times, always achieves same result
4. **Self-healing**: LCM detects drift and automatically remediates
5. **Two modes**: Push (manual, testing) vs Pull (automatic, production)
6. **Scalable**: Manage 1 to 10,000+ nodes with same effort
7. **Cross-platform**: Windows and Linux support

## Next Steps

Now that you understand DSC fundamentals, components, and implementation modes, let's explore **Azure Automation State Configuration**—Microsoft's managed DSC pull server that eliminates infrastructure overhead and provides centralized management at scale.

---

**Module**: Implement Desired State Configuration (DSC)  
**Unit**: 3 of 8  
**Duration**: 3 minutes  
**Next**: [Unit 4: Explore Azure Automation State Configuration](#)  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/implement-desired-state-configuration-dsc/3-explore-desired-state-configuration-dsc
