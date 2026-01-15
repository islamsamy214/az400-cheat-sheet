# Install Bicep

To start working with Bicep, install the Bicep CLI and/or the Visual Studio Code extension. Having both provides an enhanced authoring experience.

## Installing Bicep CLI

**Recommended**: Install via Azure CLI (version 2.20.0 or later required)

```bash
# Install Bicep CLI
az bicep install

# Verify installation
az bicep version

# Upgrade to latest version
az bicep upgrade
```

**Why Azure CLI method?**:
- Automatic updates with Azure CLI
- Integrated deployment commands
- Simpler installation process
- Cross-platform (Windows, macOS, Linux)

## Manual Installation

### Windows

**Option 1: Chocolatey**
```powershell
choco install bicep
```

**Option 2: Winget**
```powershell
winget install -e --id Microsoft.Bicep
```

**Verify**:
```powershell
bicep --help
bicep --version
```

### Linux

```bash
# Download latest Bicep CLI
curl -Lo bicep https://github.com/Azure/bicep/releases/latest/download/bicep-linux-x64

# Make executable
chmod +x ./bicep

# Move to PATH
sudo mv ./bicep /usr/local/bin/bicep

# Verify
bicep --help
```

### macOS

**Option 1: Homebrew (recommended)**
```bash
brew tap azure/bicep
brew install bicep
bicep --version
```

**Option 2: Manual script** (same as Linux method above)

## Installing VS Code Extension

The Bicep extension provides language support, IntelliSense, and linting.

**Installation steps**:
1. Open Visual Studio Code
2. Go to Extensions (Ctrl+Shift+X / Cmd+Shift+X)
3. Search for "Bicep"
4. Click "Install" on "Bicep" by Microsoft
5. Create a `.bicep` file to verify (language mode changes to "Bicep")

**Extension features**:
- **Language support**: Syntax highlighting for Bicep files
- **IntelliSense**: Auto-completion for resource types, properties, functions
- **Linting**: Real-time validation and error detection
- **Snippets**: Quick templates for common resources
- **Visualization**: Graphical view of resource dependencies

**Testing the extension**:
```bicep
// Type "resource" and press Tab
resource storage 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: 'mystorageaccount'  // IntelliSense suggests properties
  location: resourceGroup().location
  // Press Ctrl+Space to see all available properties
}
```

## Troubleshooting Installation

**Issue**: `az bicep` commands not found
**Solution**: Upgrade Azure CLI to 2.20.0+
```bash
az upgrade
az bicep install
```

**Issue**: Bicep extension not activating in VS Code
**Solution**: 
1. Ensure file extension is `.bicep`
2. Reload VS Code (Ctrl+Shift+P → "Reload Window")
3. Check VS Code version (1.52.0+ required)

**Issue**: "bicep: command not found" on Linux/macOS
**Solution**: Ensure `/usr/local/bin` is in PATH
```bash
echo $PATH | grep /usr/local/bin
# If missing, add to ~/.bashrc or ~/.zshrc:
export PATH="/usr/local/bin:$PATH"
```

For additional issues, visit: https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/installation-troubleshoot/

## Verifying Setup

**Test Bicep CLI**:
```bash
# Create sample file
echo 'param location string = resourceGroup().location' > test.bicep

# Build to ARM template
az bicep build --file test.bicep

# Should create test.json
ls test.json
```

**Test VS Code Extension**:
1. Open test.bicep in VS Code
2. Type `resource` → Should see IntelliSense
3. Hover over `resourceGroup()` → Should see documentation
4. Save file → Should see no errors (linting works)

## Best Practices

1. **Use Azure CLI method**: Easier updates and integration
2. **Install VS Code extension**: Dramatically improves productivity
3. **Keep updated**: Run `az bicep upgrade` regularly
4. **Enable auto-save**: VS Code settings → Auto Save: afterDelay
5. **Use Git**: Version control Bicep files from day 1

---

**Module**: Implement Bicep  
**Unit**: 3 of 10  
**Next**: [Unit 4: Exercise - Create Bicep Templates](#)  
**Original**: https://learn.microsoft.com/en-us/training/modules/implement-bicep/3-install-bicep
