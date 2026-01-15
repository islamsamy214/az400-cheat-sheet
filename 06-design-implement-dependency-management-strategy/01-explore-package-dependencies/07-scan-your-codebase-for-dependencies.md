# Scan Your Codebase for Dependencies

## Key Concepts
- Systematic approach to identifying which parts of code should be componentized
- Multiple strategies and tools available for effective dependency identification
- Combines pattern recognition, code analysis, and architectural review

## Strategies for Identifying Dependencies

### 1. Duplicate Code
Duplicate code appears when certain pieces are repeated in several places.

**Key Considerations:**
- Not always bad, but centralization has benefits over managing duplicate versions
- First step: Centralize duplicate code in codebase
- Then: Organize appropriately for the type of code

**Detection Approaches:**
- Look for similar code patterns across files
- Identify copy-paste sections with minor variations
- Find utility functions repeated in multiple projects

### 2. High Cohesion and Low Coupling
Find code elements that have high cohesion and low coupling with other parts.

**What to Look For:**
- Object models with business logic (domain entities and behaviors)
- Utility or helper code with specific responsibilities
- Foundation code (base classes or frameworks)
- Well-defined interfaces with clear contracts and boundaries

**Benefits:**
- High cohesion: Components are self-contained and easier to maintain
- Low coupling: Components can change independently

### 3. Individual Lifecycle
Look for parts of code with similar lifecycle that can be deployed and released individually.

**Indicators:**
- **Independent release cycles**: Code updated without affecting other parts
- **Team ownership**: Code maintained by separate team from main codebase
- **Versioning needs**: Code benefits from independent version tracking

### 4. Stable Parts
Code with slow rate of change is stable and isn't altered often.

**How to Identify:**
- Check code repository history for low change frequency
- Analyze commit patterns for areas unchanged for months/years
- Review change logs for components with infrequent updates

**Why Stable Code Makes Good Components:**
- Less risk of breaking changes
- Fewer updates required for consuming projects
- Predictable behavior and interfaces

### 5. Independent Code and Components
Code and components that are independent and unrelated to other parts of system.

**Characteristics:**
- **Self-contained functionality**: Doesn't rely heavily on other system parts
- **Clear boundaries**: Well-defined inputs and outputs
- **Minimal external dependencies**: Few connections to rest of codebase
- **Reusable across contexts**: Can be used in different applications or scenarios

## Tools for Scanning Your Codebase

### Code Analysis Tools

**Duplicate Code Detection:**
- **Visual Studio**: Built-in code clone detection
- **SonarQube**: Comprehensive code quality and duplication analysis
- **ReSharper**: Code inspection and duplication detection for .NET
- **PMD Copy/Paste Detector (CPD)**: Language-agnostic duplicate code finder

**Dependency Analysis:**
- **NDepend**: .NET dependency analysis and architecture visualization
- **Structure101**: Java and C# architecture analysis
- **Dependency Cruiser**: JavaScript/TypeScript dependency analysis
- **jdeps**: Java dependency analysis tool

**Coupling and Cohesion Metrics:**
- **Visual Studio Code Metrics**: Calculate complexity and coupling metrics
- **Understand**: Static code analysis with metrics computation
- **Lattix**: Architecture and dependency structure analysis

### Visualization Tools

**Solution Dependency Graphs:**
- **Visual Studio Dependency Diagrams**: Visualize .NET project dependencies
- **Graphviz**: Generate dependency graphs from various inputs
- **Dependency Graph Visualizers**: IDE plugins for visualizing module relationships

**Benefits of Visualization:**
- Quickly identify circular dependencies
- Understand overall architecture
- Find tightly coupled components

## Best Practices for Scanning

### Combine Multiple Strategies
- Use automated tools alongside manual code reviews
- Apply multiple identification strategies to validate findings

### Start with High-Value Areas
- Focus on code that's frequently duplicated or heavily used
- Prioritize components that would benefit multiple teams

### Document Findings
- Keep track of identified components and their characteristics
- Create a roadmap for componentization based on priority

### Validate with Teams
- Consult with developers who work with the code daily
- Ensure identified components align with team boundaries and workflows

## Quick Reference

| Strategy | What to Look For | Tools |
|----------|------------------|-------|
| **Duplicate Code** | Repeated code patterns | SonarQube, ReSharper, CPD |
| **High Cohesion/Low Coupling** | Self-contained components | NDepend, Structure101 |
| **Individual Lifecycle** | Independently releasable code | Git history analysis |
| **Stable Parts** | Infrequently changing code | Git blame, change logs |
| **Independent Components** | Self-contained functionality | Dependency analyzers |

## Scanning Workflow
```
1. Run Automated Tools
   ↓
2. Analyze Results
   ↓
3. Manual Code Review
   ↓
4. Document Findings
   ↓
5. Validate with Teams
   ↓
6. Create Componentization Roadmap
```

## Critical Notes
- 🎯 Duplicate code is the easiest indicator to find with automated tools
- 💡 High cohesion and low coupling are key architectural principles for componentization
- ⚠️ Stable code makes better candidates for early extraction
- 📊 Use visualization tools to understand overall system architecture
- 🔄 Combine automated analysis with team knowledge for best results

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-package-dependencies/7-scan-your-codebase-for-dependencies)
