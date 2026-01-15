# Explore Package Feed Managers

## Key Concepts
Package managers handle installation, removal, publishing, consumption, searching, dependency resolution, and version management of packages through cross-platform CLI tools.

## Common Package Managers

### NuGet CLI (.NET)
**Purpose**: Manage .NET packages

**Key Commands:**
```bash
nuget install    # Install packages
nuget pack       # Create packages
nuget push       # Publish packages to feeds
nuget restore    # Restore packages for a solution
```

**Tools**: NuGet CLI, dotnet CLI (integrated with .NET SDK)

### npm (JavaScript/Node.js)
**Purpose**: Manage JavaScript/Node.js packages

**Key Commands:**
```bash
npm install     # Install packages
npm publish     # Publish packages to feeds
npm update      # Update packages
npm search      # Search for packages
```

**Configuration**: `package.json` defines dependencies and scripts

### Maven (Java)
**Purpose**: Manage Java packages and build automation

**Key Commands:**
```bash
mvn install                  # Install artifacts to local repository
mvn deploy                   # Deploy artifacts to remote repository
mvn dependency:resolve       # Resolve and download dependencies
```

**Configuration**: `pom.xml` (Project Object Model) defines project structure

**Alternative**: Gradle is another popular build tool for Java

### pip (Python)
**Purpose**: Manage Python packages

**Key Commands:**
```bash
pip install      # Install packages
pip uninstall    # Remove packages
pip list         # List installed packages
pip freeze       # Output installed packages in requirements format
```

**Configuration**: `requirements.txt` lists package dependencies

### Docker CLI (Containers)
**Purpose**: Manage container images

**Key Commands:**
```bash
docker pull     # Download images from registries
docker push     # Upload images to registries
docker build    # Build images from Dockerfiles
docker images   # List local images
```

**Registries**: Docker Hub, Azure Container Registry, private registries

## CLI Tooling Benefits
- ✅ **Scriptability**: Include commands in scripts for automation
- ✅ **CI/CD integration**: Use in build and release pipelines
- ✅ **Consistency**: Same commands work across different operating systems
- ✅ **Automation**: Create, publish, and consume packages automatically
- ✅ **Version control**: Script package management in version-controlled files

### Best Practice for Pipelines
Use CLI tooling in build and release pipelines for:
- Creating packages from built components
- Publishing packages to feeds automatically
- Consuming packages during builds
- Restoring dependencies before compilation

## Integrated Developer Tooling
Developers can use GUI tools integrated into IDEs.

**Examples:**
- **Visual Studio**: NuGet Package Manager UI, integrated package management
- **Visual Studio Code**: Extensions for npm, NuGet, and other package managers
- **Eclipse**: Maven integration (m2eclipse), Gradle integration
- **IntelliJ IDEA**: Maven and Gradle support built-in
- **PyCharm**: pip package management integrated

**Benefits:**
- Visual interface for developers not comfortable with CLI
- Browse and search packages visually
- Install packages with clicks instead of commands
- Visual notifications of available package updates
- UI assistance for resolving dependency conflicts

## Choosing Tools

### Selection Criteria
- **Developer preference**: CLI for automation, GUI for ease of use
- **CI/CD requirements**: CLI tools essential for pipelines
- **Team skills**: Match tooling to team expertise
- **Project needs**: Choose based on package types and workflows

### Hybrid Approach
Many developers use a combination:
- **Development**: GUI for quick package additions and exploration
- **Automation**: CLI for build scripts and CI/CD pipelines

## Quick Reference

| Package Manager | Technology | Config File | Primary Command |
|----------------|------------|-------------|-----------------|
| NuGet CLI | .NET | nuget.config, .csproj | `nuget` / `dotnet` |
| npm | JavaScript | package.json | `npm` |
| Maven | Java | pom.xml | `mvn` |
| pip | Python | requirements.txt | `pip` |
| Docker CLI | Containers | Dockerfile | `docker` |

## Critical Notes
- 🎯 CLI tools are essential for CI/CD automation
- 💡 GUI tools provide easier discovery and learning experience
- ⚠️ Most teams use both CLI (automation) and GUI (development)
- 📊 Package managers automate dependency resolution

[Learn More](https://learn.microsoft.com/en-us/training/modules/understand-package-management/4-explore-package-feed-managers)
