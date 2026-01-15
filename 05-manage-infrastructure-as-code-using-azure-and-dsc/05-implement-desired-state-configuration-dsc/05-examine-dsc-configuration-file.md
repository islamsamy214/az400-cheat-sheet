# Examine DSC Configuration File

DSC configurations are Windows PowerShell scripts that define a special type of function. This unit explores the syntax, structure, and elements of DSC configuration files.

## DSC Configuration Elements

DSC configurations consist of three main elements: **Configuration block**, **Node block**, and **Resource blocks**.

### Basic Configuration Example

```powershell
configuration LabConfig
{
    Node WebServer
    {
        WindowsFeature IIS
        {
            Ensure = 'Present'
            Name = 'Web-Server'
            IncludeAllSubFeature = $true
        }
    }
}
```

### Configuration Block

The **Configuration block** is the outermost script block. In this example, the name of the configuration is `LabConfig`. Notice the curly brackets to define the block.

**Key characteristics**:
- Configuration keyword starts the block
- Name follows PowerShell function naming (Verb-Noun recommended)
- Contains one or more Node blocks
- Can accept parameters for reusability

### Node Block

There can be one or more **Node blocks**. The Node block defines the nodes (computers and VMs) that you're configuring. In this example, the node targets a computer called `WebServer`. You could also call it `localhost` and use it locally on any server.

**Node block patterns**:
```powershell
# Single node
Node WebServer { }

# Multiple specific nodes
Node "Server01", "Server02", "Server03" { }

# All nodes (using configuration data)
Node $AllNodes.NodeName { }

# Localhost (for testing)
Node localhost { }
```

### Resource Blocks

There can be one or more **resource blocks**. The resource block is where the configuration sets the properties for the resources. In this case, there's one resource block called `WindowsFeature`. Notice the parameters that are defined.

**Common resource parameters**:
- `Ensure`: "Present" or "Absent" (desired state)
- `DependsOn`: Array of resources that must complete first
- Resource-specific parameters (Name, Path, State, etc.)

## Advanced DSC Configuration Example

Here's an example with parameters and multiple resource blocks:

```powershell
Configuration MyDscConfiguration
{
    param
    (
        [string[]]$ComputerName='localhost'
    )

    Node $ComputerName
    {
        WindowsFeature MyFeatureInstance
        {
            Ensure = 'Present'
            Name = 'RSAT'
        }

        WindowsFeature My2ndFeatureInstance
        {
            Ensure = 'Present'
            Name = 'Bitlocker'
        }
    }
}

MyDscConfiguration
```

In this example, you specify the node's name by passing it as the `ComputerName` parameter when you compile the configuration. The name defaults to `localhost`.

## Creating DSC Configurations

Within a Configuration block, you can do almost anything that you normally could in a PowerShell function:

### Variables
```powershell
Configuration WebServerWithVariables {
    $websitePath = "C:\inetpub\wwwroot"
    $websiteName = "Default Web Site"
    
    Node WebServer {
        File WebRoot {
            Ensure = "Present"
            Type = "Directory"
            DestinationPath = $websitePath
        }
    }
}
```

### Parameters
```powershell
Configuration WebServerWithParams {
    param(
        [Parameter(Mandatory)]
        [string]$WebsiteName,
        
        [int]$Port = 80,
        
        [ValidateSet("HTTP", "HTTPS")]
        [string]$Protocol = "HTTP"
    )
    
    Node WebServer {
        WindowsFeature IIS {
            Ensure = "Present"
            Name = "Web-Server"
        }
    }
}

# Call with parameters
WebServerWithParams -WebsiteName "ContosoSite" -Port 8080 -Protocol "HTTPS"
```

### Conditional Logic
```powershell
Configuration ConditionalConfig {
    param([string]$Environment)
    
    Node WebServer {
        if ($Environment -eq "Production") {
            Registry DisableDebug {
                Ensure = "Present"
                Key = "HKLM:\Software\MyApp"
                ValueName = "DebugMode"
                ValueData = "0"
            }
        }
        else {
            Registry EnableDebug {
                Ensure = "Present"
                Key = "HKLM:\Software\MyApp"
                ValueName = "DebugMode"
                ValueData = "1"
            }
        }
    }
}
```

### Loops
```powershell
Configuration InstallMultipleFeatures {
    $features = @("Web-Server", "Web-Asp-Net45", "Web-Mgmt-Tools")
    
    Node WebServer {
        foreach ($feature in $features) {
            WindowsFeature "Install_$feature" {
                Ensure = "Present"
                Name = $feature
            }
        }
    }
}
```

## Configuration Data

**Configuration Data** separates environment-specific data from configuration logic:

```powershell
# ConfigurationData.psd1
@{
    AllNodes = @(
        @{
            NodeName = "WebServer01"
            Role = "WebServer"
            Environment = "Production"
            WebsiteName = "ContosoSite"
            Port = 80
        },
        @{
            NodeName = "WebServer02"
            Role = "WebServer"
            Environment = "Production"
            WebsiteName = "ContosoSite"
            Port = 80
        },
        @{
            NodeName = "TestServer01"
            Role = "WebServer"
            Environment = "Test"
            WebsiteName = "TestSite"
            Port = 8080
        }
    )
}

# Configuration using data
Configuration WebServerConfig {
    Node $AllNodes.NodeName {
        WindowsFeature IIS {
            Ensure = "Present"
            Name = "Web-Server"
        }
        
        File WebsiteDir {
            Ensure = "Present"
            Type = "Directory"
            DestinationPath = "C:\inetpub\$($Node.WebsiteName)"
        }
    }
}

# Compile with configuration data
$configData = Import-PowerShellDataFile -Path "ConfigurationData.psd1"
WebServerConfig -ConfigurationData $configData
```

## Dependencies with DependsOn

Use `DependsOn` to ensure resources are applied in the correct order:

```powershell
Configuration WebServerWithDependencies {
    Node WebServer {
        # Step 1: Install IIS
        WindowsFeature IIS {
            Ensure = "Present"
            Name = "Web-Server"
        }
        
        # Step 2: Create directory (depends on IIS)
        File WebRoot {
            Ensure = "Present"
            Type = "Directory"
            DestinationPath = "C:\inetpub\mysite"
            DependsOn = "[WindowsFeature]IIS"
        }
        
        # Step 3: Create config file (depends on directory)
        File WebConfig {
            Ensure = "Present"
            DestinationPath = "C:\inetpub\mysite\web.config"
            Contents = "<configuration></configuration>"
            DependsOn = "[File]WebRoot"
        }
        
        # Step 4: Start service (depends on IIS and config)
        Service W3SVC {
            Name = "W3SVC"
            State = "Running"
            DependsOn = @("[WindowsFeature]IIS", "[File]WebConfig")
        }
    }
}
```

## Best Practices

1. **Use parameters** for reusability across environments
2. **Separate data from logic** with ConfigurationData
3. **Use DependsOn** to control resource ordering
4. **Add comments** to explain complex logic
5. **Version control** configurations in Git
6. **Test locally** before deploying to production
7. **Use meaningful names** for resources (avoid Resource1, Resource2)
8. **Validate parameters** with ValidateSet, ValidateRange, etc.

## Compiling Configurations

```powershell
# Compile configuration (generates MOF files)
MyDscConfiguration -ComputerName "Server01", "Server02" -OutputPath "C:\DSC\Config"

# Output:
# C:\DSC\Config\Server01.mof
# C:\DSC\Config\Server02.mof

# Apply configuration (push mode)
Start-DscConfiguration -Path "C:\DSC\Config" -Wait -Verbose
```

You can create the configuration in any editor, such as PowerShell ISE or Visual Studio Code, and save the file as a PowerShell script with a `.ps1` file type extension.

## Key Takeaways

- **Configuration block**: Outermost container, defines configuration name
- **Node block**: Specifies target machines
- **Resource blocks**: Define desired state for specific resources
- **Parameters**: Make configurations reusable
- **ConfigurationData**: Separate environment-specific data
- **DependsOn**: Control resource execution order
- **Compilation**: Converts .ps1 to .mof files

---

**Module**: Implement Desired State Configuration (DSC)  
**Unit**: 5 of 8  
**Next**: [Unit 6: Implement DSC and Linux Automation](#)  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-desired-state-configuration-dsc/5-examine-dsc-configuration-file
