# Publish Packages

## Key Concepts
Publishing packages to GitHub Packages allows you to share code libraries, container images, and dependencies with your team or the public. GitHub Packages uses native package tooling commands, so you can publish packages using the same tools and workflows you already know.

## Understanding Package Registries

GitHub Packages supports multiple package ecosystems, each with its own format and client tools:

| Ecosystem | File Format | Client Tool |
|-----------|-------------|-------------|
| **JavaScript** | package.json | npm |
| **Ruby** | Gemfile | gem |
| **Java** | pom.xml | mvn |
| **Java** | build.gradle or build.gradle.kts | gradle |
| **.NET** | nupkg | dotnet CLI |
| **Containers** | Dockerfile | Docker |

### Package Metadata and Documentation

When creating a package, you can provide rich metadata on the package page:
- **Description**: Clear explanation of what the package does
- **Installation instructions**: Step-by-step guide for consumers
- **Usage examples**: Code samples showing common use cases
- **Dependencies**: Required packages and versions
- **License information**: Legal terms for package usage
- **Version history**: Changelog of updates and fixes

**Good documentation helps people consuming the package understand how to use it and its purposes. Consider including**:
- **Quick start guide**: Get users up and running quickly
- **API reference**: Detailed documentation of public interfaces
- **Troubleshooting tips**: Common issues and solutions
- **Migration guides**: How to upgrade between major versions

### Security Advisories

If a new package version fixes a security vulnerability, you can publish a security advisory to your repository. This notifies users of the vulnerability and provides guidance on upgrading to secure versions.

> **Tip**: You can connect a repository to more than one package. For example, a monorepo might publish multiple npm packages or a Docker image alongside a NuGet package. Ensure the README and description provide information about each package to avoid confusion.

## Publishing Workflow

To publish a package to GitHub Packages, follow these three core steps:

### 1. Create a Personal Access Token (PAT)

Authentication is required to publish packages. Create a personal access token with appropriate scopes:

| Scope | Purpose |
|-------|---------|
| `write:packages` | Required to publish packages |
| `read:packages` | Required to download packages |
| `delete:packages` | Required to delete package versions |
| `repo` | Required for packages scoped to private repositories |

**To create a PAT**:
1. Navigate to **Settings → Developer settings → Personal access tokens**
2. Click **Generate new token (classic)**
3. Select the required scopes based on your needs
4. Set an expiration date (shorter is more secure)
5. Copy the token immediately (you won't see it again)

### 2. Authenticate to GitHub Packages

Use your PAT to authenticate with your package client. Authentication methods vary by package type, but typically involve configuring a credentials file or environment variable.

**Security best practices**:
- **Never commit tokens to source code**: Use environment variables or secure credential storage
- **Use minimum required scopes**: Don't grant more permissions than necessary
- **Rotate tokens regularly**: Set expiration dates and renew periodically
- **Use organization secrets**: For GitHub Actions, store PATs as encrypted secrets

### 3. Publish the Package

Use your package client's native commands to publish. The specific command depends on your package type (npm, NuGet, Maven, Docker, etc.).

**Publishing checklist**:
✅ Version number follows semantic versioning (MAJOR.MINOR.PATCH)
✅ Package metadata is complete (description, author, license)
✅ Dependencies are correctly specified with version constraints
✅ Documentation is included (README, API reference)
✅ Tests pass successfully before publishing
✅ Package name doesn't conflict with existing packages

## Publishing NuGet Packages

NuGet packages are used for .NET applications and libraries. You can publish them using the dotnet CLI.

### Configure Authentication

Create a `nuget.config` file in your project directory to specify GitHub Packages as a source:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources>
        <clear />
        <add key="github" value="https://nuget.pkg.github.com/OWNER/index.json" />
    </packageSources>
    <packageSourceCredentials>
        <github>
            <add key="Username" value="USERNAME" />
            <add key="ClearTextPassword" value="TOKEN" />
        </github>
    </packageSourceCredentials>
</configuration>
```

**Configuration parameters**:
- **OWNER**: GitHub username or organization name that owns the repository
- **USERNAME**: Your GitHub username
- **TOKEN**: Your personal access token with `write:packages` scope

### Publish the Package

You can publish using two methods:

**Method 1: Using nuget.config file**
```bash
dotnet nuget push "bin/Release/OctocatApp.1.0.0.nupkg" --source "github"
```

**Method 2: Using --api-key option**
```bash
dotnet nuget push "bin/Release/OctocatApp.1.0.0.nupkg" --api-key YOUR_GITHUB_PAT --source "github"
```

**Best practices for NuGet publishing**:
- **Use Release configuration**: Build with `dotnet pack -c Release` for optimized packages
- **Include symbols**: Publish symbol packages (.snupkg) for debugging support
- **Version automatically**: Use MSBuild properties to version from Git tags
- **Validate package**: Test package installation before publishing
- **Document breaking changes**: Update release notes for major version changes

## Publishing npm Packages

npm packages are used for JavaScript and Node.js applications. You can publish them using the npm CLI.

### Configure Authentication

You have two authentication options:

**Option 1: Edit ~/.npmrc file**

Add your PAT to the per-user .npmrc file:
```
//npm.pkg.github.com/:_authToken=TOKEN
```

Create a new `~/.npmrc` file if one doesn't exist. This method stores credentials globally for all projects.

**Option 2: Login via command line**

Use the npm login command for interactive authentication:
```bash
$ npm login --scope=@OWNER --registry=https://npm.pkg.github.com

> Username: USERNAME
> Password: TOKEN
> Email: PUBLIC-EMAIL-ADDRESS
```

**Authentication parameters**:
- **USERNAME**: Your GitHub username
- **TOKEN**: Your personal access token with `write:packages` scope
- **PUBLIC-EMAIL-ADDRESS**: Your email address (public or private)
- **OWNER**: GitHub username or organization for scoped packages

### Configure package.json

Ensure your `package.json` includes the correct registry and scope:

```json
{
  "name": "@OWNER/package-name",
  "version": "1.0.0",
  "repository": {
    "type": "git",
    "url": "https://github.com/OWNER/REPOSITORY.git"
  },
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}
```

### Publish the Package

Use the standard npm publish command:
```bash
npm publish
```

**Best practices for npm publishing**:
- **Use scoped packages**: Prefix with @owner/ to avoid name conflicts
- **Test before publishing**: Run `npm pack` and test the .tgz file locally
- **Use .npmignore**: Exclude unnecessary files to reduce package size
- **Version with npm**: Use `npm version patch/minor/major` to update versions
- **Publish from CI/CD**: Automate publishing with GitHub Actions

### Viewing Published Packages

After you publish a package, you can view it on GitHub:
1. Navigate to your repository
2. Click **Packages** in the sidebar
3. Select your package to view details, versions, and download statistics

## Automating Package Publishing with GitHub Actions

You can automate package publishing using GitHub Actions workflows. This ensures packages are published automatically when code changes.

### Example Workflow for npm

```yaml
name: Publish Package

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "18"
          registry-url: "https://npm.pkg.github.com"
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Example Workflow for NuGet

```yaml
name: Publish Package

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: "8.0.x"
      - run: dotnet pack -c Release
      - run: dotnet nuget push "**/*.nupkg" --source github --api-key ${{ secrets.GITHUB_TOKEN }}
```

**Benefits of automated publishing**:
- **Consistency**: Every release creates a package automatically
- **Speed**: No manual steps required to publish
- **Traceability**: Package versions match Git tags
- **Quality gates**: Run tests before publishing
- **Security**: Use built-in GITHUB_TOKEN instead of PATs

## Quick Reference

### Publishing Workflow Summary
```
1. Create PAT with write:packages scope
   ↓
2. Authenticate package client with PAT
   ↓
3. Publish using native package commands
   ↓
4. View published package on GitHub
```

### PAT Scopes Quick Reference
| Scope | Publish | Download | Delete | Private Repos |
|-------|---------|----------|--------|---------------|
| `write:packages` | ✅ | ✅ | ❌ | ❌ |
| `read:packages` | ❌ | ✅ | ❌ | ❌ |
| `delete:packages` | ❌ | ❌ | ✅ | ❌ |
| `repo` | ✅* | ✅* | ✅* | ✅ |

*For private repository packages

### Package-Specific Commands
| Package Type | Build Command | Publish Command |
|--------------|---------------|-----------------|
| **NuGet** | `dotnet pack -c Release` | `dotnet nuget push *.nupkg --source github` |
| **npm** | `npm pack` | `npm publish` |
| **Maven** | `mvn package` | `mvn deploy` |
| **Docker** | `docker build -t image:tag .` | `docker push ghcr.io/owner/image:tag` |

## Critical Notes
🔐 **PAT required** - Personal access token with `write:packages` scope is required for publishing

📦 **Use scoped packages** - For npm, always use @owner/package-name format

🎯 **Native commands** - Use the same publish commands you're already familiar with

✅ **SemVer versioning** - Follow semantic versioning (MAJOR.MINOR.PATCH) for version numbers

🤖 **Automate with Actions** - Use GitHub Actions workflows to publish on release creation

🔒 **GITHUB_TOKEN for CI/CD** - Use built-in token in workflows instead of PATs for security

📝 **Document thoroughly** - Include README, API reference, and usage examples

⚠️ **Never commit PATs** - Use environment variables or GitHub secrets for tokens

## Learn More
- [Working with the Container registry](https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Working with a GitHub Packages registry](https://docs.github.com/packages/working-with-a-github-packages-registry)
- [About GitHub Security Advisories](https://docs.github.com/github/managing-security-vulnerabilities/about-github-security-advisories)
- [Working with the NuGet registry](https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
- [Working with the npm registry](https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-npm-registry)
- [Creating a personal access token](https://docs.github.com/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
