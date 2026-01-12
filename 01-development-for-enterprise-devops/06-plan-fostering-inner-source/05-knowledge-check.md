# Module Assessment

## Knowledge Check Topics

This assessment validates your understanding of:

### Inner Source Fundamentals
- Definition and benefits of inner source development
- How inner source differs from traditional siloed development
- Microsoft's inner source transformation journey
- Cultural aspects of adopting inner source practices

### Fork Workflow Implementation
- When to use forks vs. branches
- Creating and managing forks in Azure DevOps
- Fork structure and what's included/excluded
- Working with origin and upstream remotes

### Collaboration Patterns
- Pull request flow from fork to upstream
- Syncing forks with upstream repositories
- Using topic branches within forks
- Contributing to projects without write access

### Decision Criteria
- Choosing between branches and forks based on:
  - Team size (2-5 developers vs. larger teams)
  - Contributor types (core team vs. casual contributors)
  - Access requirements (write access vs. read-only)
  - Isolation needs (immediate vs. reviewed integration)

## Self-Assessment Questions

Consider these scenarios:

1. **When should you use forks instead of branches?**
   - Small team with write access → Branches
   - Large team with casual contributors → Forks
   - Need to isolate changes for review → Forks

2. **What's included when you create a fork?**
   - ✅ Repository contents, commits, branches
   - ❌ Permissions, policies, build pipelines

3. **How do you sync your fork with upstream?**
   ```bash
   git fetch upstream main
   git rebase upstream/main
   git push origin
   ```

4. **What permission is needed to fork a repository?**
   - Read access to original repository
   - Create Repository permission in target project

5. **Why work in topic branches even in your fork?**
   - Maintain multiple independent workstreams
   - Reduce confusion when syncing with upstream
   - Better organization and clarity

## Critical Concepts to Remember
- 🎯 Forks enable inner source by allowing contribution without write access
- 💡 Pull requests from fork to upstream apply upstream's policies and reviewers
- ⚠️ Regular syncing prevents your fork from diverging too far from upstream
- 📊 Fork workflow is most effective for larger teams and casual contributors
- 🔄 Inner source breaks down silos and enables cross-team collaboration
- ✨ Azure DevOps provides built-in fork support for inner source workflows

[Learn More](https://learn.microsoft.com/en-us/training/modules/plan-fostering-inner-source/5-knowledge-check)
