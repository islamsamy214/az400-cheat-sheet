# Automate Release Notes with GitHub

## Key Concepts
- **Auto-Generated Release Notes**: GitHub automatically creates release notes from PR history
- **Customizable Templates**: Configure which PRs to include and how to categorize
- **Label-Based Organization**: Use PR labels to group changes into categories
- **`.github/release.yml`**: Configuration file for release notes automation

## Creating Automatically Generated Release Notes

### Using Auto-Generate Feature

When creating a release in GitHub UI:

1. Navigate to repository **Releases** page
2. Click **Draft a new release**
3. Select or create tag for release
4. Check **"Auto-generate release notes"** option
5. GitHub generates notes from PRs between tags
6. Customize generated content if needed
7. Publish release

![Create Release Notes](https://learn.microsoft.com/en-us/training/wwl-azure/manage-git-repositories/media/create-release-note-3cb0bee2.png)

### What Gets Included

| First Release | Subsequent Releases |
|---------------|---------------------|
| All changes from repository history | Changes since last release |
| All merged PRs | Merged PRs since last tag |
| All contributors | New contributors since last release |

### Default Output Format

```markdown
## What's Changed
* Feature: Add user dashboard by @user1 in #123
* Fix: Memory leak in processor by @user2 in #145
* Update dependencies by @user3 in #167

## New Contributors
* @user4 made their first contribution in #189

**Full Changelog**: https://github.com/org/repo/compare/v1.0.0...v1.1.0
```

## Setting Up Custom Release Notes Template

### Configuration File Location

Create `.github/release.yml` in repository root:

![Create File](https://learn.microsoft.com/en-us/training/wwl-azure/manage-git-repositories/media/github-create-new-file-f3e1148b.png)

### File Path Structure

```
repository-root/
└── .github/
    └── release.yml    ← Release notes configuration
```

![Release File Creation](https://learn.microsoft.com/en-us/training/wwl-azure/manage-git-repositories/media/github-release-file-creation-83568335.png)

### Example Configuration

```yaml
# .github/release.yml

changelog:
  # Exclude certain labels and authors
  exclude:
    labels:
      - ignore-for-release
      - dependencies
      - documentation
    authors:
      - octocat
      - dependabot
  
  # Categorize PRs by label
  categories:
    - title: Breaking Changes 🛠
      labels:
        - Semver-Major
        - breaking-change
    
    - title: Exciting New Features 🎉
      labels:
        - Semver-Minor
        - enhancement
        - feature
    
    - title: Bug Fixes 🐛
      labels:
        - bug
        - fix
    
    - title: Documentation 📚
      labels:
        - documentation
        - docs
    
    - title: Other Changes
      labels:
        - "*"  # Catch-all for unlabeled PRs
```

### Configuration Options

#### Exclude Section

| Option | Purpose |
|--------|---------|
| `labels` | Skip PRs with these labels |
| `authors` | Skip PRs from these users |

```yaml
exclude:
  labels:
    - ignore-for-release
    - duplicate
    - wontfix
  authors:
    - bot-user
    - dependabot[bot]
```

#### Categories Section

| Field | Description |
|-------|-------------|
| `title` | Category name in release notes (supports emojis) |
| `labels` | PR labels that belong in this category |

```yaml
categories:
  - title: Security Updates 🔒
    labels:
      - security
      - vulnerability
```

**Special Label `"*"`**: Matches any PR not in other categories

### Commit Configuration

```bash
# Create .github directory
mkdir -p .github

# Create release.yml
cat > .github/release.yml << 'EOF'
changelog:
  exclude:
    labels:
      - ignore-for-release
    authors:
      - octocat
  categories:
    - title: Breaking Changes 🛠
      labels:
        - Semver-Major
        - breaking-change
    - title: Exciting New Features 🎉
      labels:
        - Semver-Minor
        - enhancement
    - title: Other Changes
      labels:
        - "*"
EOF

# Commit to repository
git add .github/release.yml
git commit -m "Add automated release notes configuration"
git push origin main
```

![Commit File](https://learn.microsoft.com/en-us/training/wwl-azure/manage-git-repositories/media/github-commit-new-file-56536fe6.png)

## Testing Configuration

```bash
# Create new release to test template
gh release create v1.0.0 --generate-notes

# Or via GitHub UI:
# 1. Go to Releases
# 2. Draft new release
# 3. Select tag
# 4. Click "+ Auto-generate release notes"
# 5. Preview generated notes with your categories
```

## Advanced Configuration

### Full Example with All Options

```yaml
# .github/release.yml

changelog:
  # Exclude certain items
  exclude:
    labels:
      - ignore-for-release
      - duplicate
      - invalid
      - wontfix
    authors:
      - octocat
      - dependabot[bot]
      - github-actions[bot]
  
  # Organize PRs into categories
  categories:
    # High priority changes
    - title: ⚠️ Breaking Changes
      labels:
        - breaking-change
        - breaking
        - Semver-Major
    
    # New features
    - title: ✨ New Features
      labels:
        - feature
        - enhancement
        - Semver-Minor
    
    # Bug fixes
    - title: 🐛 Bug Fixes
      labels:
        - bug
        - fix
        - bugfix
        - hotfix
    
    # Performance improvements
    - title: ⚡ Performance
      labels:
        - performance
        - optimization
    
    # Security fixes
    - title: 🔒 Security
      labels:
        - security
        - vulnerability
    
    # Documentation updates
    - title: 📚 Documentation
      labels:
        - documentation
        - docs
    
    # Dependency updates
    - title: 📦 Dependencies
      labels:
        - dependencies
        - deps
    
    # Catch-all category
    - title: 🔧 Other Changes
      labels:
        - "*"
```

### Label Naming Strategy

**Recommended Approach**: Use consistent label prefixes

```yaml
# Organize labels by type
Type: Feature       → New functionality
Type: Bug          → Bug fixes
Type: Docs         → Documentation
Type: Refactor     → Code improvements
Priority: High     → High priority items
Priority: Low      → Low priority items
```

## Best Practices

### PR Title Guidelines

Auto-generated notes use PR titles - make them clear:

✅ **Good PR Titles**:
- "Add user authentication with OAuth2"
- "Fix memory leak in background processor"
- "Update React to v18.2.0"

❌ **Poor PR Titles**:
- "Fix bug"
- "Update stuff"
- "WIP"

### Label Strategy

| Label | Category Usage |
|-------|----------------|
| `breaking-change` | Breaking Changes |
| `feature`, `enhancement` | New Features |
| `bug`, `fix` | Bug Fixes |
| `security` | Security Updates |
| `documentation` | Documentation |
| `dependencies` | Dependencies |

### Workflow Integration

```yaml
# .github/workflows/release.yml
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
      
      - name: Create Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh release create ${{ github.ref_name }} \
            --generate-notes \
            --title "Release ${{ github.ref_name }}"
```

## Additional Resources

- [Automatically generated release notes](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes)
- [Configuration options](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes#configuration-options)
- [About releases](https://docs.github.com/repositories/releasing-projects-on-github/about-releases)
- [Linking to releases](https://docs.github.com/repositories/releasing-projects-on-github/linking-to-releases)
- [Automation for release forms with query parameters](https://docs.github.com/repositories/releasing-projects-on-github/automation-for-release-forms-with-query-parameters)

## Critical Notes
- 🎯 Auto-generated notes pull from merged PRs - good PR titles essential
- 💡 Use labels consistently across PRs for effective categorization
- ⚠️ Configuration changes apply to future releases, not existing ones
- 📊 Exclude bot accounts (dependabot, github-actions) to reduce noise
- 🔄 Test configuration by creating draft release before publishing
- ✨ Emojis in category titles improve visual scanning of release notes

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/9-automate-release-notes-github)
