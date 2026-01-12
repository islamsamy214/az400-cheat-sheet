# Integrate Other Code Quality Tools

## Overview
Many tools can help you find and fix technical debt. This unit covers popular options that integrate well with Azure DevOps.

## NDepend for .NET Projects

### What NDepend Provides
NDepend is a powerful tool for .NET developers that helps you:

| Feature | Description | Benefit |
|---------|-------------|---------|
| **Real-time tracking** | See debt added in the last hour | Immediate feedback |
| **Pre-commit checks** | Catch problems before committing | Early prevention |
| **Custom rules** | Write quality checks with C# LINQ | Project-specific standards |
| **Built-in rules** | Premade rules for common issues | Quick setup |

### Key Capabilities
- **Visual Studio extension**: Seamless integration with development workflow
- **Code metrics**: Cyclomatic complexity, coupling, cohesion
- **Dependency analysis**: Visualize architecture and dependencies
- **Trend analysis**: Track quality over time

### Example Custom Rule (C# LINQ)
```csharp
// Find methods with too many parameters
from m in Methods
where m.NbParameters > 5
select new { 
    m, 
    Issue = "Method has too many parameters: " + m.NbParameters 
}

// Find classes with high coupling
from t in Types
where t.TypesUsingMe.Count() > 10
select new { 
    t, 
    Issue = "Class is used by " + t.TypesUsingMe.Count() + " types" 
}
```

## ReSharper Code Quality Analysis

### Automated Quality Checks
ReSharper provides comprehensive code analysis that can:

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Command line execution** | Run from CLI | CI/CD pipeline integration |
| **Automated build failure** | Stop poor code reaching production | Quality gates |
| **Team standards enforcement** | Configure shared rules | Consistent code style |
| **Consistent feedback** | Same checks across all developers | Team alignment |

### Pipeline Integration
```bash
# Run ReSharper inspection from command line
inspectcode.exe MySolution.sln \
  --output=inspection-results.xml \
  --severity=WARNING \
  --format=Xml

# Fail build if issues found
if [ -s inspection-results.xml ]; then
  echo "Code quality issues found"
  exit 1
fi
```

### Features
- **Code inspections**: 2500+ code quality checks
- **Code cleanup**: Automated code formatting
- **Refactoring support**: Safe code transformations
- **Language support**: C#, VB.NET, JavaScript, TypeScript, CSS, HTML

## Find Tools in Azure DevOps Marketplace

### Discover Quality Tools
To find more code quality tools for your pipeline:

**Steps**:
1. Go to your **build pipeline** in Azure DevOps
2. Click **Add a new task**
3. Search for **"Quality"** in the marketplace
4. Browse available tools and extensions

### Available Tools Examples
- **NDepend**: .NET code quality and architecture
- **ReSharper**: Code analysis and cleanup
- **SonarQube**: Multi-language code quality
- **Checkmarx**: Security scanning
- **WhiteSource**: Open source management
- **Veracode**: Application security testing

## Choose the Right Tool for Your Project

### Selection Criteria

| Factor | Considerations | Questions to Ask |
|--------|---------------|------------------|
| **Programming language** | Tool must support your stack | Does it support our languages? |
| **Team size** | Some tools scale better | Does it work for 5 vs 50 developers? |
| **Integration needs** | Pipeline compatibility | Does it integrate with Azure DevOps? |
| **Budget** | Tool cost + setup time | What's the total cost of ownership? |

### Decision Matrix

```
Project Needs Assessment:
├── Languages: C#, JavaScript, TypeScript
├── Team Size: 15 developers
├── Current Tools: Visual Studio, Azure DevOps
└── Budget: $5000/year

Tool Evaluation:
├── NDepend: $500/dev, VS integration ✓, .NET only ⚠️
├── ReSharper: $300/dev, VS integration ✓, .NET + web ✓
├── SonarQube: Free (Community), All languages ✓, Self-hosted ⚠️
└── SonarCloud: $10/dev/month, All languages ✓, Cloud-hosted ✓

Decision: SonarCloud (best multi-language support, easy setup)
```

## Additional Quality Tools

### Language-Specific Tools
| Language | Recommended Tools |
|----------|-------------------|
| **JavaScript/TypeScript** | ESLint, TSLint, Prettier |
| **.NET** | NDepend, ReSharper, StyleCop |
| **Java** | SpotBugs, PMD, Checkstyle |
| **Python** | Pylint, Flake8, Black |
| **Go** | golint, go vet, staticcheck |

### Cross-Platform Tools
| Tool | Languages | Specialty |
|------|-----------|-----------|
| **SonarQube/Cloud** | 25+ languages | Code quality & security |
| **CodeClimate** | Multiple | Maintainability analysis |
| **Codacy** | Multiple | Automated code reviews |
| **DeepSource** | Multiple | Security & performance |

## Integration Best Practices

### Pipeline Configuration
1. **Run on every commit**: Catch issues early
2. **Set quality gates**: Define pass/fail criteria
3. **Track trends**: Monitor improvement over time
4. **Automate fixes**: Use auto-fix where possible
5. **Provide feedback**: Show results in PRs

### Example Azure Pipeline Task
```yaml
- task: SonarQubePrepare@5
  inputs:
    SonarQube: 'SonarQube-Connection'
    scannerMode: 'CLI'
    
- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    
- task: NDependConsole@1
  inputs:
    projectFile: 'MyProject.ndproj'
    
- task: ReSharperCLI@2
  inputs:
    solutionOrProjectPath: 'MySolution.sln'
```

## Next Steps Resources
- [NDepend documentation](https://www.ndepend.com/)
- Visual Studio Marketplace for more tools
- ReSharper Code Quality Analysis guides
- Azure DevOps Marketplace extensions

## Critical Notes
- 🎯 Choose tools based on language support, team size, and budget
- 💡 NDepend provides real-time debt tracking (last hour visibility)
- ⚠️ ReSharper can fail builds automatically to enforce quality gates
- 📊 Azure DevOps Marketplace offers 100+ code quality extensions
- 🔄 Command-line execution enables CI/CD pipeline integration
- ✨ Custom rules (C# LINQ) allow project-specific quality standards

[Learn More](https://learn.microsoft.com/en-us/training/modules/identify-technical-debt/7-integrate-other-code-quality-tools)
