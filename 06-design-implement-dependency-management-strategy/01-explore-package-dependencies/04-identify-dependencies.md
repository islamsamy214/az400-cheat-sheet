# Identify Dependencies

## Key Concepts
- Software projects already use various dependencies from libraries and frameworks
- Dependencies need to be officially acknowledged and managed
- Thorough investigation of codebase is necessary to identify components

## Types of Dependencies

| Type | Description | Examples |
|------|-------------|----------|
| **External** | Third-party libraries and packages | NuGet packages, npm modules, Maven artifacts |
| **Internal** | Code components developed within organization | Shared libraries across projects |
| **Development** | Tools needed only during development | Testing frameworks, build tools |

## Steps to Identify Dependencies

### 1. Investigate Your Codebase
Ask these questions:
- Which code components are used across multiple projects?
- Which components have clear, well-defined interfaces?
- Which components change independently of the main application?
- Which components could benefit from separate versioning?

### 2. Analyze Code Usage Patterns
Look for:
- **Code duplication**: Same code copied across multiple projects
- **Shared libraries**: Internal libraries that multiple teams rely on
- **Utility functions**: Common helper functions used throughout codebase
- **Domain models**: Business logic representing core domain concepts

### 3. Review Project References
Examine:
- **Project-to-project references**: In solution files
- **Assembly references**: In compiled languages (C#, Java)
- **Import statements**: In source code files
- **Configuration files**: Package manifests (`package.json`, `pom.xml`, `.csproj`)

### 4. Categorize Dependencies
Once identified, categorize by:
- **Type**: External vs. internal
- **Scope**: Application dependency vs. development dependency
- **Criticality**: Core functionality vs. optional features
- **Volatility**: Stable vs. frequently changing

## Identifying Internal Dependencies

### Example Scenario
Consider code that encapsulates a specific business domain model. This code might be:
- Integrated as source code directly in your project
- Copied and used across different projects and teams
- Maintained independently by a dedicated team

This code should be recognized as a dependency and managed accordingly.

## Benefits of Proper Identification
- ✅ **Improved maintainability**: Clear boundaries between components
- ✅ **Better versioning**: Components can be versioned independently
- ✅ **Enhanced reusability**: Components can be shared across projects
- ✅ **Reduced duplication**: Eliminates code copying
- ✅ **Better testing**: Components can be tested in isolation

## Refining Your Components
The identification process leads to:
- Modifying how you arrange your code
- Constructing the solution differently
- Contributing to the refinement of your components

## Critical Notes
- 🎯 Code duplication is a strong indicator of potential dependencies
- 💡 Components with clear interfaces are good candidates for extraction
- ⚠️ Internal dependencies are as important as external ones
- 📊 Proper categorization helps prioritize dependency management efforts

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-package-dependencies/4-identify-dependencies)
