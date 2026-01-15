# Understand Source and Package Componentization

## Two Approaches to Componentization

### Source Componentization
Organizing source code by splitting it into separate parts around identified components.

**How It Works:**
- Code organized into logical modules or projects within a solution
- Components defined by folder structure and project boundaries
- Teams work on different components within the same repository

**When to Use:**
- ✅ Single team or closely coordinated teams working in same repository
- ✅ Tightly coupled components that change frequently together
- ✅ Early development when component boundaries are still being defined
- ✅ Internal use only when source code isn't shared outside project

**Advantages:**
- Simplified development: All code in one place
- Easier refactoring: Component boundaries can be adjusted without changing external contracts
- Unified builds: Everything compiles together, ensuring compatibility

**Limitations:**
- ❌ Sharing challenges: Requires distributing source code or binary artifacts
- ❌ Version control complexity: Large repositories become difficult to manage
- ❌ Build times: Entire solution must be built even for small changes
- ❌ Limited autonomy: Teams must coordinate changes across components

### Package Componentization
Using packages as a formal way of wrapping and handling components.

**How It Works:**
- Components built and packaged separately
- Packages published to package feeds (Azure Artifacts, NuGet.org, npm, Maven Central)
- Projects consume components by referencing package versions
- Each component has its own version and release cycle

**When to Use:**
- ✅ Multiple teams maintaining different components
- ✅ Loosely coupled components that can evolve independently
- ✅ External sharing across organizations or publicly
- ✅ Mature components with stable interfaces

**Advantages:**
- Independent versioning: Each component versioned and released independently
- Better dependency management: Explicit declarations with version constraints
- Improved governance: Control over what versions are consumed
- Team autonomy: Teams work on components independently
- Reusability: Components easily shared across projects and organizations

**Characteristics Added by Packages:**
- **Versioning**: Track and manage different versions
- **Metadata**: Include information about authors, licenses, and dependencies
- **Distribution**: Formal distribution channel through package feeds
- **Dependency resolution**: Automatic resolution of transitive dependencies

## Choosing the Right Approach

| Factor | Source Componentization | Package Componentization |
|--------|------------------------|--------------------------|
| **Team size** | Single or small team | Multiple teams |
| **Component coupling** | Tightly coupled | Loosely coupled |
| **Release cadence** | Unified releases | Independent releases |
| **Sharing scope** | Internal only | Internal and external |
| **Maturity** | Early development | Stable components |

## Hybrid Approach
Many organizations use a combination:
- Use **source componentization** for core application code
- Use **package componentization** for shared libraries and frameworks

## Quick Reference

### Source Componentization
```
Solution/
├── Core.Project/
├── Data.Project/
└── API.Project/
```
All built together in one solution.

### Package Componentization
```
Project A → References Package Core v1.2.0
Project B → References Package Core v1.2.0
Project C → References Package Core v1.3.0
```
Each project references specific package versions.

## Critical Notes
- 🎯 Source componentization works best for tightly coupled, evolving code
- 💡 Package componentization provides better control and autonomy for mature components
- ⚠️ Hybrid approaches are common and often most practical
- 📊 Choice depends on team structure, component maturity, and release requirements

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-package-dependencies/5-understand-source-package-componentization)
