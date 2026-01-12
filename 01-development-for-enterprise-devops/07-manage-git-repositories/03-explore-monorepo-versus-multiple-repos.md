# Explore Monorepo Versus Multiple Repos

## Key Concepts
- **Monorepo**: All source code in one repository
- **Multiple Repos**: Each project or component in separate repository
- **Trade-offs**: Balance between ease of access and team autonomy

## Repository Organization Approaches

### Monorepo (Single Repository)

**Structure**: All code in one place

**Advantages**:
- Easy access control - give everyone access to everything at once
- Simple to get started - clone once, have everything
- Easy cross-project refactoring
- Shared tooling and standards across all projects
- Atomic commits across multiple projects
- Simplified dependency management within the monorepo

**Disadvantages**:
- Can become very large over time
- Slower clone and operations as size grows
- All teams use same tools and workflows
- Harder to enforce different processes per team
- Build and CI complexity with multiple projects

### Multiple Repositories (Multi-Repo)

**Structure**: Separate repository per project/component

**Advantages**:
- Each team controls their own repository
- Teams choose libraries, tools, and workflows that work best
- Smaller, faster repositories
- Clear ownership and boundaries
- Independent versioning and releases
- Easier to restrict access to sensitive code

**Disadvantages**:
- Using code from another repository like third-party library
- Complex cross-repository changes
- Dependency version management overhead
- Harder to maintain consistency across projects

## The Cross-Repository Change Problem

### Multi-Repo Process for Bug Fix:
1. Find bug in shared library (Repository A)
2. Fix bug in Repository A
3. Publish new version of library
4. Return to your project (Repository B)
5. Update dependency to new library version
6. Test integration
7. Deploy your changes

**Challenge**: Slow process, especially with different codebases, tools, or workflows. May need to wait for other team.

### Monorepo Process for Same Bug Fix:
1. Find bug in shared library
2. Fix bug directly (same repository)
3. Test and commit changes

**Benefit**: Anyone can fix anything they need immediately

## Decision Framework

| Factor | Monorepo | Multiple Repos |
|--------|----------|----------------|
| **Team Size** | Small to medium teams | Large distributed teams |
| **Code Sharing** | Frequent cross-project changes | Clear project boundaries |
| **Tool Diversity** | Standard toolchain across projects | Teams need different tools |
| **Access Control** | All-or-nothing access OK | Fine-grained access needed |
| **Build Complexity** | Can handle complex builds | Prefer independent builds |
| **Team Autonomy** | Shared standards important | Team independence critical |

## Real-World Examples

### Companies Using Monorepo:
- Google (largest monorepo in world)
- Facebook/Meta
- Microsoft (for some projects)
- Twitter

### Companies Using Multi-Repo:
- Netflix
- Amazon
- Most open-source projects

## Azure DevOps Best Practice

> **Note**: In Azure DevOps, it's common to use a separate repository for each solution within a project.

This provides balance between:
- Manageable repository sizes
- Related code grouped together
- Team autonomy within project boundaries

## Managing Complex Dependencies

### Monorepo Challenges:
- Complex dependency relationships between projects
- Need sophisticated build systems
- Testing impact of changes across projects

### Multi-Repo Challenges:
- Coordinating changes across repositories
- Managing library versions
- Ensuring compatibility across projects

## Critical Notes
- 🎯 Choice between monorepo and multi-repo is about optimizing team collaboration
- 💡 Monorepo simplifies cross-project changes but requires strong tooling
- ⚠️ Multi-repo treats internal code like external dependencies - slower changes
- 📊 Azure DevOps commonly uses one repository per solution as middle ground
- 🔄 Can start with monorepo and split later (or vice versa) as needs change
- ✨ Large monorepos like Google's require specialized tools (Bazel, custom VCS)

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/3-explore-monorepo-versus-multiple-repos)
