# Summary

## Key Learning Achievements

### Foster Inner Source Development
- **Break Down Silos**: Use Git to enable cross-team collaboration within your organization
- **Cultural Transformation**: Apply open source principles inside company firewall
- **Universal Contribution**: Any team can contribute to any internal project
- **Microsoft's Success**: Company-wide adoption using Azure Repos demonstrates scalability

### Implement Fork Workflow
- **Create Forks**: Copy repositories without needing write access
- **Independent Work**: Make changes without affecting original repository
- **Contribute Back**: Use pull requests to propose changes to upstream
- **Manage Remotes**: Work with origin (your fork) and upstream (original) repositories

### Choose the Right Approach

| Scenario | Solution | Key Factors |
|----------|----------|-------------|
| **Small teams (2-5)** | Branches | Direct write access, simpler workflow |
| **Larger teams** | Forks | Better isolation, scalability |
| **Casual contributors** | Forks | No write access needed |
| **Core team** | Branches | Full repository permissions |
| **Need isolation** | Forks | Changes reviewed before integration |

### Share Code Effectively

#### Pull Request Flow
```bash
# From fork to upstream (most common)
git push origin feature/my-feature
# Create PR in Azure DevOps UI

# Sync fork with upstream
git fetch upstream main
git rebase upstream/main
git push origin
```

#### Synchronization Pattern
1. Fetch latest from upstream
2. Rebase your work on upstream changes
3. Push updated fork
4. Create pull request when ready

## Implementation Checklist

- [ ] Understand when to use forks vs. branches for your team size
- [ ] Know how to create fork in Azure DevOps (requires Create Repository permission)
- [ ] Configure remotes: origin (fork) and upstream (original)
- [ ] Work in topic branches even within your fork
- [ ] Create pull requests from fork to upstream
- [ ] Regularly sync fork with upstream to avoid conflicts
- [ ] Leverage upstream's policies and reviewers through PRs
- [ ] Foster culture of cross-team contribution

## Benefits Summary

| Benefit | Impact |
|---------|--------|
| **Safety** | Changes isolated until reviewed and approved |
| **Collaboration** | Multiple teams work on different features simultaneously |
| **Quality** | All changes go through consistent review process |
| **Flexibility** | Contributors don't need write access to contribute |
| **Knowledge Sharing** | Developers learn from code across teams |
| **Code Reuse** | Build on existing work instead of duplicating effort |

## Additional Resources
- [Fork your repository - Azure Repos](https://learn.microsoft.com/en-us/azure/devops/repos/git/forks)
- [Clone an existing Git repo](https://learn.microsoft.com/en-us/azure/devops/repos/git/clone)
- [Azure Repos Git Tutorial](https://learn.microsoft.com/en-us/azure/devops/repos/git/gitworkflow)

## Critical Notes
- 🎯 Inner source applies open source collaboration inside your organization
- 💡 Forks enable contribution without repository write access
- ⚠️ Fork doesn't copy permissions, policies, or build pipelines from upstream
- 📊 Pull requests can flow in both directions: fork↔upstream
- 🔄 Regular synchronization keeps your fork aligned with upstream
- ✨ Microsoft uses this approach company-wide for all engineering projects

[Learn More](https://learn.microsoft.com/en-us/training/modules/plan-fostering-inner-source/6-summary)
