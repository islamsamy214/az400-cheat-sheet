# Install a Package

## Key Concepts
Installing packages from GitHub Packages allows you to consume libraries, dependencies, and container images hosted on GitHub. You can install any package you have permission to view and use it as a dependency in your project.

## Discovering Packages

GitHub Packages provides multiple ways to discover packages:

### Search Options
| Search Method | Description |
|---------------|-------------|
| **Global search** | Search across all public packages on GitHub |
| **Organization search** | Find packages within a specific organization |
| **Repository search** | Browse packages published from a repository |
| **Package type filter** | Filter by npm, NuGet, Maven, Docker, etc. |

### Package Information

When you find a package, the package page provides essential information:
- **Description**: What the package does and its purpose
- **Installation instructions**: How to add the package to your project
- **Version history**: Available versions and release notes
- **Dependencies**: Required packages and version constraints
- **Usage examples**: Code samples showing common scenarios
- **License**: Legal terms for using the package
- **Download statistics**: How many times the package has been installed

> **Best practice**: Always read the package's installation and description instructions before installing to understand requirements, breaking changes, and compatibility.

## Installation Workflow

Installing a package follows a consistent two-step process regardless of package type:

### 1. Authenticate to GitHub Packages

Configure your package client to authenticate with GitHub Packages using a personal access token (PAT) with the `read:packages` scope.

**Authentication requirements**:
- **Public packages**: Authentication is required even for public packages
- **Private packages**: Requires PAT with appropriate repository access
- **Organization packages**: May require organization membership
- **Internal packages**: Available only within GitHub Enterprise organizations

### 2. Install the Package

Use your package client's native installation commands (npm install, dotnet add package, mvn install, etc.) to add the package to your project.

**Installation checklist**:
✅ Verify package name and version are correct
✅ Check compatibility with your project's framework version
✅ Review dependency tree for conflicts
✅ Test package functionality after installation
✅ Update documentation to reflect new dependency

## Installing NuGet Packages

NuGet packages can be installed using Visual Studio or the dotnet CLI.

### Method 1: Visual Studio

Visual Studio provides a graphical interface for managing NuGet packages:

**Step 1: Open NuGet Package Manager**
- Expand **Solution → Project**
- Right-click on **Dependencies**
- Select **Manage NuGet Packages...**

**Step 2: Browse and Install**
- Click the **Browse** tab
- Search for your package name
- Select the version you want
- Click **Install**

You can browse, install, and update dependencies from multiple registries simultaneously.

### Method 2: Editing .csproj Directly

You can add package references directly to your project file:

**Step 1: Authenticate to GitHub Packages**
Create or update your `nuget.config` file (see Publishing section for authentication details).

**Step 2: Add PackageReference**
Add an ItemGroup with a PackageReference in your `.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net8.0</TargetFramework>
        <PackageId>OctocatApp</PackageId>
        <Version>1.0.0</Version>
        <Authors>Octocat</Authors>
        <Company>GitHub</Company>
        <PackageDescription>This package adds an Octocat!</PackageDescription>
        <RepositoryUrl>https://github.com/OWNER/REPOSITORY</RepositoryUrl>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="OctokittenApp" Version="12.0.2" />
    </ItemGroup>
</Project>
```

**Step 3: Restore packages**
Install the packages using the restore command:
```bash
dotnet restore
```

### Method 3: Using dotnet CLI

You can install packages using the dotnet add command:
```bash
dotnet add package OctokittenApp --version 12.0.2 --source github
```

**Best practices for NuGet installation**:
- **Pin versions**: Specify exact versions for reproducible builds
- **Use central package management**: Consolidate versions in Directory.Packages.props
- **Review vulnerabilities**: Check for security advisories before installing
- **Test compatibility**: Ensure package works with your target framework

## Installing npm Packages

npm packages require configuring an `.npmrc` file to specify GitHub Packages as the registry.

### Configure .npmrc File

**Step 1: Authenticate to GitHub Packages**
Create or update your `~/.npmrc` file with your authentication token (see Publishing section for authentication details).

**Step 2: Create project .npmrc**
In the same directory as your `package.json` file, create or edit a `.npmrc` file:
```
@OWNER:registry=https://npm.pkg.github.com
```

Replace **OWNER** with the name of the user or organization account that owns the package.

**Step 3: Commit .npmrc to repository**
Add the `.npmrc` file to your repository so all team members use the same configuration.

### Add Dependency to package.json

Configure `package.json` to include the package as a dependency:

```json
{
  "name": "@my-org/server",
  "version": "1.0.0",
  "description": "Server app that uses the @octo-org/octo-app package",
  "main": "index.js",
  "author": "",
  "license": "MIT",
  "dependencies": {
    "@octo-org/octo-app": "1.0.0"
  }
}
```

**Dependency specification**:
| Format | Example | Behavior |
|--------|---------|----------|
| **Exact version** | `"1.0.0"` | Installs a specific version |
| **Caret range** | `"^1.0.0"` | Installs compatible minor/patch updates |
| **Tilde range** | `"~1.0.0"` | Installs compatible patch updates only |
| **Latest** | `"latest"` | Installs the newest version (not recommended for production) |

### Install the Package

Run the npm install command:
```bash
npm install
```

Or install a specific package:
```bash
npm install @octo-org/octo-app
```

### Multiple Organizations

If you need to install packages from multiple organizations, add additional lines to your `.npmrc` file:
```
@first-org:registry=https://npm.pkg.github.com
@second-org:registry=https://npm.pkg.github.com
@third-org:registry=https://npm.pkg.github.com
```

**Best practices for npm installation**:
- **Use package-lock.json**: Commit lockfile for consistent installs across environments
- **Audit dependencies**: Run `npm audit` to check for vulnerabilities
- **Use npm ci**: In CI/CD pipelines, use `npm ci` for faster, more reliable installs
- **Version constraints**: Use semantic version ranges to balance stability and updates
- **Private registry first**: Configure scopes to check GitHub Packages before public npm

## Installing in CI/CD Pipelines

You can install packages in GitHub Actions workflows using the same methods:

### npm in GitHub Actions

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: "18"
    registry-url: "https://npm.pkg.github.com"
    scope: "@OWNER"
- run: npm ci
  env:
    NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### NuGet in GitHub Actions

```yaml
- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: "8.0.x"
    source-url: https://nuget.pkg.github.com/OWNER/index.json
  env:
    NUGET_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
- run: dotnet restore
```

**Benefits of GITHUB_TOKEN**:
- **No PAT required**: Automatically available in workflows
- **Scoped permissions**: Limited to workflow's repository access
- **Automatic rotation**: Token is regenerated for each workflow run
- **Secure**: Never exposed in logs or artifacts

## Troubleshooting Installation Issues

Common problems and solutions:

| Error | Cause | Solution |
|-------|-------|----------|
| **401 Unauthorized** | Missing or invalid PAT | Verify token has read:packages scope |
| **404 Not Found** | Package name or registry URL incorrect | Check package exists and owner is correct |
| **403 Forbidden** | No permission to access package | Request access or verify organization membership |
| **Version not found** | Requested version doesn't exist | Check available versions on package page |
| **Dependency conflicts** | Incompatible package versions | Update other dependencies or use resolutions |

## Quick Reference

### Installation Workflow Summary
```
1. Authenticate with PAT (read:packages scope)
   ↓
2. Configure package client (.npmrc, nuget.config)
   ↓
3. Add package to project (package.json, .csproj)
   ↓
4. Install using native commands
   ↓
5. Test package functionality
```

### Installation Commands by Package Type
| Package Type | Add Dependency | Install Command |
|--------------|----------------|-----------------|
| **NuGet** | Edit .csproj or use dotnet add | `dotnet restore` |
| **npm** | Edit package.json | `npm install` |
| **Maven** | Edit pom.xml | `mvn install` |
| **Gradle** | Edit build.gradle | `gradle build` |

### Authentication Files by Package Type
| Package Type | Config File | Location |
|--------------|-------------|----------|
| **NuGet** | nuget.config | Project directory |
| **npm** | .npmrc | Project directory or ~/.npmrc |
| **Maven** | settings.xml | ~/.m2/settings.xml |
| **Docker** | config.json | ~/.docker/config.json |

### Version Constraint Examples (npm)
| Constraint | Matches | Example |
|------------|---------|---------|
| `1.0.0` | Exact | Only 1.0.0 |
| `^1.0.0` | Compatible | 1.0.0, 1.1.0, 1.9.9 (not 2.0.0) |
| `~1.0.0` | Patch | 1.0.0, 1.0.1, 1.0.9 (not 1.1.0) |
| `>=1.0.0 <2.0.0` | Range | 1.0.0 to 1.9.9 |

## Critical Notes
🔐 **Authentication required** - Even public packages require authentication to install

📦 **PAT with read:packages** - Personal access token must have `read:packages` scope

🎯 **Native commands** - Use familiar package manager commands (npm install, dotnet restore)

✅ **Test after installation** - Always verify package works in your project

🔒 **GITHUB_TOKEN in CI/CD** - Use built-in token in GitHub Actions instead of PATs

📝 **Commit config files** - Add .npmrc and nuget.config to repository for team consistency

⚠️ **Check compatibility** - Verify package version works with your framework version

🔍 **Review security** - Run npm audit or dotnet list package --vulnerable

## Learn More
- [Working with a GitHub Packages registry](https://docs.github.com/packages/working-with-a-github-packages-registry)
- [Working with the NuGet registry](https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
- [Working with the npm registry](https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-npm-registry)
- [About permissions for GitHub Packages](https://docs.github.com/packages/learn-github-packages/about-permissions-for-github-packages)
- [Searching for packages](https://docs.github.com/search-github/searching-on-github/searching-for-packages)
- [Create and remove project dependencies](https://learn.microsoft.com/en-us/visualstudio/ide/how-to-create-and-remove-project-dependencies)
- [Adding a file to a repository](https://docs.github.com/repositories/working-with-files/managing-files/adding-a-file-to-a-repository)
