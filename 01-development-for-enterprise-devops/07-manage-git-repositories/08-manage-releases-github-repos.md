# Manage Releases with GitHub Repos

## Key Concepts
- **GitHub Releases**: Based on Git tags, marking important milestones
- **Release Notes**: Documentation of changes included in each release
- **Git Tags**: Snapshots of repository state at specific points
- **GitHub CLI**: Command-line tool for managing releases (`gh`)

## Releases in GitHub

### Foundation: Git Tags

Releases are based on [Git tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging):
- **Tag**: Snapshot of repository's current state
- **Purpose**: Mark important milestones or deliverable versions
- **Usage**: Reference specific version during build and release process

```bash
# Create annotated tag
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push tag to remote
git push origin v1.0.0

# Push all tags
git push origin --tags
```

### Release Features

GitHub releases extend Git tags with:
- **Release Notes**: Markdown description of changes
- **Binary Attachments**: Compiled executables, packages, or assets
- **@mentions**: Acknowledge contributors
- **Notification System**: Users can subscribe to new releases

![Release Notes Example](https://learn.microsoft.com/en-us/training/wwl-azure/manage-git-repositories/media/github-release-notes-b8ec3d9a.png)

*Reference: [Microsoft azure-pipelines-agent releases](https://github.com/Microsoft/azure-pipelines-agent/releases)*

### Additional Capabilities

| Feature | Description |
|---------|-------------|
| **Marketplace Publishing** | Publish GitHub Actions from release |
| **Git LFS Objects** | Control whether LFS objects included in archives |
| **Notifications** | Subscribe to new release alerts |
| **Pre-releases** | Mark releases as pre-release/beta |
| **Draft Releases** | Prepare releases before publishing |

## Creating a Release (GitHub CLI)

### Basic Creation

```bash
# Create release interactively
gh release create tag

# Follow prompts for:
# - Release title
# - Release notes
# - Attachments
```

### Create with Options

```bash
# Create prerelease with title and notes
gh release create v1.2.1 \
  --title "Version 1.2.1 Beta" \
  --notes "Beta release for testing" \
  --prerelease

# Create release with attached files
gh release create v2.0.0 \
  --title "Major Release 2.0" \
  --notes "See CHANGELOG.md for details" \
  dist/*.zip

# Create draft release
gh release create v1.5.0 \
  --draft \
  --notes-file RELEASE_NOTES.md
```

### @mention Contributors

```bash
# Mention users in release notes
gh release create v1.0.0 \
  --notes "Thanks to @contributor1 and @contributor2 for their work!"

# GitHub automatically:
# - Creates Contributors section
# - Shows avatar list of mentioned users
```

### Additional Commands

```bash
# List releases
gh release list

# View specific release
gh release view v1.0.0

# Download release assets
gh release download v1.0.0

# See all available commands
gh release --help
```

Reference: [GitHub CLI manual - gh release create](https://cli.github.com/manual/gh_release_create)

## Editing a Release

### Via GitHub CLI
⚠️ **Not Available**: Cannot edit releases with GitHub CLI

### Via Web Browser

1. Go to main repository page on GitHub.com
2. Click **Releases** (right side of file list)
3. Find release to edit
4. Click edit icon (right side, next to release)
5. Update release details
6. Click **Update release**

## Deleting a Release

```bash
# Delete release by tag
gh release delete tag -y

# Example
gh release delete v1.0.0 -y

# Without -y flag, prompts for confirmation
gh release delete v1.0.0
? Delete release v1.0.0 in owner/repo? Yes
```

**Note**: Deletes release but NOT the underlying Git tag

To also delete the tag:
```bash
# Delete release
gh release delete v1.0.0 -y

# Delete tag locally
git tag -d v1.0.0

# Delete tag on remote
git push origin :refs/tags/v1.0.0
```

## Release Workflow Integration

### Viewing Releases and Tags

```bash
# List all tags
git tag

# List tags matching pattern
git tag -l "v2.*"

# Show tag details
git show v1.0.0

# List releases via GitHub CLI
gh release list --limit 10
```

Reference: [Viewing your repository's releases and tags](https://docs.github.com/repositories/releasing-projects-on-github/viewing-your-repositorys-releases-and-tags)

### Automated Release Process

```yaml
# GitHub Actions workflow example
name: Create Release
on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build artifacts
        run: ./build.sh
      
      - name: Create Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh release create ${{ github.ref_name }} \
            --generate-notes \
            dist/*
```

## Best Practices

### Semantic Versioning

```bash
# Format: MAJOR.MINOR.PATCH
v1.0.0  # Initial release
v1.0.1  # Patch: bug fixes
v1.1.0  # Minor: new features, backward compatible
v2.0.0  # Major: breaking changes

# Pre-releases
v2.0.0-alpha.1
v2.0.0-beta.2
v2.0.0-rc.1
```

### Release Checklist

- [ ] All changes merged to main/release branch
- [ ] Version number updated in code
- [ ] CHANGELOG.md updated
- [ ] All tests passing
- [ ] Create and push tag
- [ ] Build release artifacts
- [ ] Create GitHub release with notes
- [ ] Attach binary files if applicable
- [ ] Mark as pre-release if not stable
- [ ] Notify team/users of new release

### Release Notes Template

```markdown
## What's Changed

### Features
- New dashboard view (#123)
- API rate limit increased (#145)

### Bug Fixes
- Fixed memory leak in processor (#156)
- Resolved authentication timeout (#167)

### Breaking Changes
- API endpoint `/v1/users` now requires authentication

### Contributors
Thanks to @user1, @user2, and @user3 for their contributions!

**Full Changelog**: https://github.com/org/repo/compare/v1.0.0...v1.1.0
```

## Additional Resources

- [Managing releases in a repository](https://docs.github.com/repositories/releasing-projects-on-github/managing-releases-in-a-repository)
- [Publishing an action in the GitHub Marketplace](https://docs.github.com/actions/creating-actions/publishing-actions-in-github-marketplace)
- [Managing Git LFS objects in archives](https://docs.github.com/github/administering-a-repository/managing-git-lfs-objects-in-archives-of-your-repository)
- [Viewing your subscriptions](https://docs.github.com/github/managing-subscriptions-and-notifications-on-github/viewing-your-subscriptions)

## Critical Notes
- 🎯 GitHub releases are built on Git tags - create tag first, then release
- 💡 Use semantic versioning (MAJOR.MINOR.PATCH) for clear version communication
- ⚠️ Deleting release does NOT delete underlying Git tag - must delete separately
- 📊 @mention contributors in notes to automatically create Contributors section
- 🔄 Automate release creation with GitHub Actions triggered on tag push
- ✨ Attach binary files to releases for easy distribution to users

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/8-manage-releases-github-repos)
