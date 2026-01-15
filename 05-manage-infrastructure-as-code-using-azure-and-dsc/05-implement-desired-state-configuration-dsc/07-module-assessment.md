# Module Assessment

Test your knowledge of Desired State Configuration (DSC) with these scenario-based questions.

## Question 1: Identifying Configuration Drift

**Scenario**: Your team manages 200 production web servers. Three months after initial deployment, you discover that 45 servers have port 3389 (RDP) open to the internet, 23 servers are missing the latest security patch, and 12 servers have unauthorized software installed. No one remembers making these changes.

**Question**: What problem is this scenario describing?

A. Security breach requiring incident response  
B. Configuration drift causing compliance violations  
C. Inadequate change management process  
D. Normal variation in production environments  

**Answer**: **B. Configuration drift causing compliance violations**

**Explanation**: This is a classic example of configuration drift. The servers started in a known good state but changed over time through manual changes, failed updates, or other untracked modifications. While inadequate change management (C) may have contributed, the core problem is drift. DSC with ApplyAndAutoCorrect mode would detect and remediate these issues automatically.

**DSC Solution**:
```powershell
Configuration SecurityBaseline {
    Node WebServer {
        # Close unauthorized ports
        Registry DisableRDP {
            Key = "HKLM:\System\CurrentControlSet\Control\Terminal Server"
            ValueName = "fDenyTSConnections"
            ValueData = 1
        }
        
        # Ensure only approved software
        Script RemoveUnauthorizedSoftware {
            TestScript = { !(Get-Package -Name "UnauthorizedApp" -ErrorAction SilentlyContinue) }
            SetScript = { Get-Package -Name "UnauthorizedApp" | Uninstall-Package -Force }
            GetScript = { @{Result = (Get-Package -Name "UnauthorizedApp" -ErrorAction SilentlyContinue) } }
        }
    }
}
```

---

## Question 2: DSC Components

**Scenario**: You're explaining DSC to a colleague. They ask: "What are the three main components that make DSC work?"

**Question**: Select all three correct components:

A. Configurations  
B. Resources  
C. Containers  
D. Local Configuration Manager (LCM)  
E. Azure Automation Account  

**Answer**: **A, B, and D**

**Explanation**:
- **Configurations** (A): PowerShell scripts that declare desired state
- **Resources** (B): Code that enforces state (WindowsFeature, File, Service, etc.)
- **Local Configuration Manager** (D): Engine on each node that applies configurations

Containers (C) are not a DSC component. Azure Automation Account (E) is optional infrastructure for centralized management, not a core DSC component.

---

## Question 3: Push vs Pull Mode Selection

**Scenario**: You're designing a DSC deployment strategy for two environments:
- **Environment A**: 5 test servers, frequently changing configurations, developers need immediate feedback
- **Environment B**: 500 production servers across 3 regions, strict compliance requirements, centralized monitoring needed

**Question**: Which deployment mode should you use for each environment?

A. Push mode for both environments  
B. Pull mode for both environments  
C. Push mode for Environment A, Pull mode for Environment B  
D. Pull mode for Environment A, Push mode for Environment B  

**Answer**: **C. Push mode for Environment A, Pull mode for Environment B**

**Explanation**:
- **Environment A** (test): Push mode is perfect for testing. Developers can immediately push configurations, see results, and iterate quickly. The small scale (5 servers) makes manual pushing practical.
- **Environment B** (production): Pull mode is essential for 500 servers. Centralized pull server provides automatic updates, compliance reporting, and scales to thousands of nodes without manual intervention.

**Implementation**:
```powershell
# Environment A: Push mode
Start-DscConfiguration -Path "C:\DSC\TestConfig" -ComputerName "TestServer01" -Wait -Verbose

# Environment B: Pull mode with Azure Automation
Register-AzAutomationDscNode -ResourceGroupName "rg-prod" `
    -AutomationAccountName "aa-prod" `
    -AzureVMName "ProdServer*" `
    -ConfigurationMode "ApplyAndAutoCorrect"
```

---

## Question 4: Azure Automation State Configuration Benefits

**Scenario**: Your organization currently runs a self-hosted DSC pull server on a Windows Server VM. It requires 4 hours of maintenance monthly, went down twice last quarter causing compliance gaps, and doesn't integrate with your monitoring solutions.

**Question**: Which benefit of Azure Automation State Configuration addresses these issues?

A. Built-in pull server with 99.9% SLA  
B. Support for Linux and Windows nodes  
C. Ability to use custom DSC resources  
D. Configuration data separation  

**Answer**: **A. Built-in pull server with 99.9% SLA**

**Explanation**: The scenario describes operational burden (4 hrs/month maintenance) and availability issues (downtime causing compliance gaps). Azure Automation State Configuration provides a managed pull server with high availability, eliminating maintenance overhead. It also integrates with Azure Monitor and Log Analytics for centralized monitoring.

**Cost comparison**:
```
Self-hosted pull server:
- VM cost: ~$100/month
- Maintenance: 4 hours × $50/hr = $200/month
- Downtime cost: 2 incidents × 4 hours × $500/hr = $4,000/quarter
- Total quarterly: ~$4,900

Azure Automation State Configuration:
- First 5 nodes: Free
- Additional nodes: $6/node/month
- For 500 nodes: ~$2,970/month, $8,910/quarter
- 99.9% SLA, zero maintenance time
```

---

## Question 5: LCM Configuration Modes

**Scenario**: You need to configure LCM differently for three server groups:
- **Group A** (dev): Apply configurations once for testing, no automatic corrections
- **Group B** (staging): Monitor drift but don't auto-correct, alert on changes
- **Group C** (production): Automatically correct any drift every 15 minutes

**Question**: Match each group to the correct LCM ConfigurationMode:

**Answer**:
- **Group A**: `ApplyOnly`
- **Group B**: `ApplyAndMonitor`
- **Group C**: `ApplyAndAutoCorrect`

**Explanation**:
- **ApplyOnly**: Applies configuration once, doesn't check again (perfect for testing)
- **ApplyAndMonitor**: Checks for drift, reports it, but doesn't fix automatically (staging environments)
- **ApplyAndAutoCorrect**: Continuously monitors and automatically fixes drift (production with strict compliance)

**Configuration examples**:
```powershell
# Group A - Dev: ApplyOnly
[DSCLocalConfigurationManager()]
Configuration LCM_Dev {
    Node "DevServer" {
        Settings {
            ConfigurationMode = "ApplyOnly"
            RefreshMode = "Push"
        }
    }
}

# Group B - Staging: ApplyAndMonitor
[DSCLocalConfigurationManager()]
Configuration LCM_Staging {
    Node "StagingServer" {
        Settings {
            ConfigurationMode = "ApplyAndMonitor"
            RefreshMode = "Pull"
            RefreshFrequencyMins = 30
            ConfigurationModeFrequencyMins = 15
        }
    }
}

# Group C - Production: ApplyAndAutoCorrect
[DSCLocalConfigurationManager()]
Configuration LCM_Production {
    Node "ProdServer" {
        Settings {
            ConfigurationMode = "ApplyAndAutoCorrect"
            RefreshMode = "Pull"
            RefreshFrequencyMins = 30
            ConfigurationModeFrequencyMins = 15
            RebootNodeIfNeeded = $true
        }
    }
}
```

---

## Question 6: DSC Configuration Syntax

**Scenario**: You're reviewing a DSC configuration file and see this code:

```powershell
Configuration WebServerSetup {
    Node "WebServer01" {
        WindowsFeature IIS {
            Ensure = "Present"
            Name = "Web-Server"
        }
        
        File WebRoot {
            Ensure = "Present"
            Type = "Directory"
            DestinationPath = "C:\inetpub\wwwroot"
            DependsOn = "[WindowsFeature]IIS"
        }
    }
}
```

**Question**: What does `DependsOn = "[WindowsFeature]IIS"` accomplish?

A. Makes the File resource optional if IIS isn't installed  
B. Ensures IIS is installed before creating the directory  
C. Automatically uninstalls IIS if the directory is removed  
D. Groups IIS and File resources together for reporting  

**Answer**: **B. Ensures IIS is installed before creating the directory**

**Explanation**: `DependsOn` controls resource ordering. DSC will apply the `WindowsFeature IIS` resource first, wait for it to complete successfully, then apply the `File WebRoot` resource. This prevents errors from creating directories before required software is installed.

**Why it matters**:
```powershell
# Without DependsOn: Might fail
File WebRoot {
    DestinationPath = "C:\inetpub\wwwroot"  # Fails if C:\inetpub doesn't exist yet
}
WindowsFeature IIS {  # Creates C:\inetpub
    Name = "Web-Server"
}

# With DependsOn: Always works
WindowsFeature IIS {  # Runs first
    Name = "Web-Server"
}
File WebRoot {
    DestinationPath = "C:\inetpub\wwwroot"  # C:\inetpub now exists
    DependsOn = "[WindowsFeature]IIS"
}
```

---

## Question 7: Linux DSC Resources

**Scenario**: You need to configure 50 Ubuntu servers to:
1. Install nginx and git packages
2. Create a service account named "webapp"
3. Create directory `/var/www/myapp` owned by webapp user
4. Ensure nginx service is running

**Question**: Which DSC for Linux resources would you use? Match each requirement to the correct resource:

**Answer**:
1. Install packages → **nxPackage**
2. Create service account → **nxUser**
3. Create directory with ownership → **nxFile**
4. Ensure service running → **nxService**

**Complete configuration**:
```powershell
Configuration UbuntuWebServer {
    Import-DscResource -ModuleName nx

    Node "UbuntuServer" {
        # 1. Install packages
        nxPackage InstallNginx {
            Name = "nginx"
            Ensure = "Present"
            PackageManager = "apt"
        }
        
        nxPackage InstallGit {
            Name = "git"
            Ensure = "Present"
            PackageManager = "apt"
        }
        
        # 2. Create service account
        nxUser WebAppUser {
            UserName = "webapp"
            Ensure = "Present"
            FullName = "Web Application Service Account"
            HomeDirectory = "/home/webapp"
        }
        
        # 3. Create directory with ownership
        nxFile AppDirectory {
            DestinationPath = "/var/www/myapp"
            Ensure = "Present"
            Type = "Directory"
            Mode = "755"
            Owner = "webapp"
            Group = "webapp"
            DependsOn = "[nxUser]WebAppUser"
        }
        
        # 4. Ensure nginx running
        nxService NginxService {
            Name = "nginx"
            Controller = "systemd"
            Enabled = $true
            State = "Running"
            DependsOn = "[nxPackage]InstallNginx"
        }
    }
}
```

---

## Question 8: Real-World Compliance Scenario

**Scenario**: Your company must comply with regulations requiring:
- All production servers must have Windows Firewall enabled
- No servers can have Guest account enabled
- All servers must have specific security software installed
- Compliance must be verified every 30 minutes
- Any non-compliant servers must be auto-remediated within 15 minutes
- You must generate compliance reports for auditors

You need to choose between:
- **Option A**: Weekly manual compliance checks with PowerShell scripts
- **Option B**: Azure Policy with deny effects
- **Option C**: DSC with Azure Automation State Configuration
- **Option D**: Group Policy in Active Directory

**Question**: Which solution best meets all requirements?

**Answer**: **C. DSC with Azure Automation State Configuration**

**Explanation**:

| Requirement | Option A (Manual) | Option B (Policy) | Option C (DSC) | Option D (GPO) |
|-------------|------------------|-------------------|----------------|----------------|
| Verify every 30 min | ❌ Weekly only | ✅ Continuous | ✅ Configurable | ✅ Every 90-120 min |
| Auto-remediate in 15 min | ❌ Manual fix | ❌ Prevents only | ✅ Auto-correct | ⚠️ Next GP refresh |
| Install software | ❌ Separate process | ❌ No provisioning | ✅ Full install | ⚠️ Limited |
| Compliance reports | ⚠️ Manual | ⚠️ Basic | ✅ Detailed | ⚠️ Basic |
| Works for Azure+hybrid | ✅ Yes | ✅ Azure only | ✅ Hybrid | ⚠️ Domain only |

**Why DSC wins**:
- **Verification**: RefreshFrequencyMins = 30
- **Remediation**: ConfigurationModeFrequencyMins = 15, ApplyAndAutoCorrect
- **Software installation**: Full Package resource support
- **Reports**: Azure Automation provides detailed compliance dashboards and Log Analytics integration
- **Hybrid**: Works for Azure VMs, on-premises, and other clouds

**Implementation**:
```powershell
Configuration ComplianceBaseline {
    Node "ProdServer" {
        # Requirement 1: Firewall enabled
        Service WindowsFirewall {
            Name = "MpsSvc"
            State = "Running"
            StartupType = "Automatic"
        }
        
        # Requirement 2: Disable Guest account
        User DisableGuest {
            UserName = "Guest"
            Disabled = $true
        }
        
        # Requirement 3: Security software
        Package SecuritySoftware {
            Name = "Contoso Security Agent"
            Path = "\\fileserver\software\security-agent.msi"
            ProductId = "12345678-1234-1234-1234-123456789012"
            Ensure = "Present"
        }
    }
}

# Register nodes with compliance settings
Register-AzAutomationDscNode `
    -ResourceGroupName "rg-compliance" `
    -AutomationAccountName "aa-compliance" `
    -AzureVMName "ProdServer*" `
    -ConfigurationMode "ApplyAndAutoCorrect" `
    -ConfigurationModeFrequencyMins 15 `
    -RefreshFrequencyMins 30 `
    -RebootNodeIfNeeded $true
```

---

## Score Interpretation

- **8/8**: Expert - Ready to implement DSC in production
- **6-7/8**: Proficient - Review missed topics before implementation
- **4-5/8**: Intermediate - Study DSC components and scenarios
- **0-3/8**: Beginner - Revisit module content

## Key Concepts Review

1. **Configuration drift**: Servers changing from original state over time
2. **DSC components**: Configurations (scripts), Resources (code), LCM (engine)
3. **Push vs Pull**: Push for testing/small scale, Pull for production/large scale
4. **Azure Automation**: Managed pull server, centralized management, reporting
5. **LCM modes**: ApplyOnly (test), ApplyAndMonitor (staging), ApplyAndAutoCorrect (prod)
6. **Dependencies**: Use `DependsOn` to control resource ordering
7. **Linux support**: 6 resources (nxFile, nxScript, nxUser, nxGroup, nxService, nxPackage)
8. **Compliance**: DSC excels at continuous compliance with auto-remediation

---

**Module**: Implement Desired State Configuration (DSC)  
**Unit**: 7 of 8  
**Next**: [Unit 8: Summary](#)  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-desired-state-configuration-dsc/7-knowledge-check
