# Knowledge Check

## Module Assessment Topics

This assessment validates your understanding of:

### Working with Large Repositories
- Why repositories become large (history, binary files)
- Shallow clone techniques
- VFS for Git functionality
- Scalar features and benefits
- When to use each optimization technique

### Repository Organization
- Monorepo vs multi-repo trade-offs
- Cross-repository dependencies
- Team collaboration models
- Azure DevOps best practices

### Changelog and Release Notes
- Changelog purpose and format
- Automated vs manual changelog generation
- Tools: gitchangelog, github-changelog-generator
- GitHub auto-generated release notes
- `.github/release.yml` configuration

### Scalar and Cross-Repository Sharing
- Scalar implementation and benefits
- Git submodules vs subrepos
- Centralized package registries
- Modular repository structure

### Data Recovery
- `git log`, `git checkout`, `git cherry-pick`
- `git reflog` for deleted commits
- `git fsck --lost-found`
- File and commit recovery procedures

### Repository Cleanup
- git filter-repo tool
- BFG Repo-Cleaner usage
- Removing sensitive data
- History rewriting safety

### Release Management
- Git tags and GitHub releases
- GitHub CLI (`gh release`)
- Semantic versioning
- Release automation

### API Documentation
- OpenAPI/Swagger documentation
- Azure DevOps vs GitHub approaches
- MkDocs, Docusaurus, Jekyll
- Documentation automation

### Git History Automation
- Automating API docs generation
- Release notes from Git history
- CI/CD documentation pipelines
- Publishing strategies

### Repository Permissions
- GitHub account types comparison
- Repository roles (Read, Triage, Write, Maintain, Admin)
- Organization vs personal permissions
- Team-based access control

### GitHub Tags
- Tag categories and naming conventions
- Semantic versioning format
- Tag creation and management
- Tag protection rules

## Self-Assessment Questions

### Large Repositories

**Q: When should you use Scalar?**
- Small repository (< 1 GB) → Not needed
- Large repository with history (> 50 GB) → Yes
- Microsoft Windows/Office repos → Yes (designed for this)

**Q: What does shallow clone do?**
- Limits history depth downloaded
- Saves disk space
- Faster initial clone
- Good for CI/CD systems

### Monorepo vs Multi-Repo

**Q: When to choose monorepo?**
- ✅ Small to medium team
- ✅ Frequent cross-project changes
- ✅ Shared tools and standards
- ❌ Need team-specific tools

**Q: When to choose multi-repo?**
- ✅ Large distributed teams
- ✅ Clear project boundaries
- ✅ Team autonomy important
- ❌ Frequent shared code changes

### Release Management

**Q: How to create GitHub release?**
```bash
# 1. Create tag
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin v1.0.0

# 2. Create release
gh release create v1.0.0 --generate-notes
```

**Q: What's semantic versioning format?**
- v{MAJOR}.{MINOR}.{PATCH}
- v1.0.0 → Initial release
- v1.0.1 → Patch (bug fixes)
- v1.1.0 → Minor (new features)
- v2.0.0 → Major (breaking changes)

### Data Recovery

**Q: How to recover deleted file?**
```bash
# 1. Find last commit with file
git log -- deleted-file.txt

# 2. Restore from that commit
git checkout abc123^ -- deleted-file.txt

# 3. Commit restoration
git add deleted-file.txt
git commit -m "Restored deleted-file.txt"
```

**Q: How to recover deleted branch?**
```bash
# 1. Find branch in reflog
git reflog

# 2. Recreate branch
git branch recovered-branch abc123
```

### Repository Cleanup

**Q: How to remove sensitive data?**
```bash
# Using BFG Repo-Cleaner
bfg --delete-files passwords.txt
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
```

**Q: What happens after history rewrite?**
- All team members must re-clone
- Cannot pull into old clones
- Force push required
- Old history incompatible

### Permissions

**Q: What permission for non-code contributor?**
- **Read** - View, clone, open issues

**Q: What permission for PR/issue manager?**
- **Triage** - Manage PRs/issues without code write

**Q: What permission for active developer?**
- **Write** - Push code, merge PRs

### Automated Release Notes

**Q: How to configure categories?**
```yaml
# .github/release.yml
changelog:
  categories:
    - title: New Features 🎉
      labels:
        - feature
        - enhancement
    - title: Bug Fixes 🐛
      labels:
        - bug
        - fix
```

**Q: How to exclude items?**
```yaml
changelog:
  exclude:
    labels:
      - ignore-for-release
    authors:
      - dependabot
```

## Key Concepts Checklist

### Repository Optimization
- [ ] Understand when to use shallow clone vs Scalar
- [ ] Know VFS for Git enables on-demand file downloads
- [ ] Scalar automatically enables multiple Git optimizations

### Organization Strategy
- [ ] Monorepo: All code in one place
- [ ] Multi-repo: Separate repos per component
- [ ] Trade-offs: Access vs autonomy

### Release Workflow
- [ ] Tags mark milestones in Git history
- [ ] GitHub releases built on Git tags
- [ ] Semantic versioning: v{MAJOR}.{MINOR}.{PATCH}

### Data Safety
- [ ] Git reflog keeps 90 days of commit history
- [ ] Recovery possible for "deleted" data
- [ ] History rewriting requires team coordination

### Documentation
- [ ] Automate API docs in CI/CD
- [ ] Generate release notes from Git history
- [ ] Publish to central accessible location

### Permissions
- [ ] Use teams for group management
- [ ] Repository roles: Read → Triage → Write → Maintain → Admin
- [ ] GitHub Enterprise adds custom roles

## Critical Concepts to Remember

- 🎯 Scalar essential for repositories > 50 GB with extensive history
- 💡 Monorepo simplifies cross-project changes, multi-repo enables team autonomy
- ⚠️ History rewriting is destructive - backup and coordinate with team
- 📊 GitHub releases use Git tags as foundation
- 🔄 Automate documentation generation to keep it current with code
- ✨ Use semantic versioning for clear version communication

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/14-knowledge-check)
