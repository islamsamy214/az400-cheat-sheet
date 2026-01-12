# Configure GitHub Tags to Organize Repositories

## Key Concepts
- **Git Tags**: Markers pointing to specific points in repository history
- **Semantic Versioning**: Standard for labeling software releases (MAJOR.MINOR.PATCH)
- **Tag Categories**: Release versions, features, bug fixes, maintenance, custom
- **Tag Protection**: Prevent unauthorized tag creation or deletion (beta feature)

## Purpose of Tags

Tags create structured timeline of project development:
- **Mark Milestones**: Important points in project history
- **Reference Versions**: Specific commits during build/release
- **Easy Navigation**: Quick access to significant changes
- **Release Management**: Foundation for GitHub releases

## Planning Tag Strategy

### Release Versions

**Semantic Versioning (SemVer)**:
```
Format: MAJOR.MINOR.PATCH

v1.0.0  → Initial stable release
v1.0.1  → Patch: Bug fixes only
v1.1.0  → Minor: New features, backward compatible
v2.0.0  → Major: Breaking changes

Pre-release versions:
v0.2.0-alpha    → Alpha testing
v1.5.0-beta.3   → Beta release 3
v2.0.0-rc.1     → Release candidate 1
```

**Best Practice**: Add "v" prefix (v1.0.0, not 1.0.0)

### Feature Releases

Tag commits introducing new features or significant changes:

```bash
# Feature tag examples
feature/new-login-page
feature/payment-integration
feature/dark-mode
feature/api-v2

# Provides clear history of feature additions
```

**Benefits**:
- Track progress of individual features
- Easy to identify when features were added
- Helps with feature rollback if needed

### Bug Fixes

Tag commits that fix reported bugs or issues:

```bash
# Bug fix tag examples
bugfix/issue-123
fix/critical-bug
hotfix/security-patch
bugfix/memory-leak
```

**Benefits**:
- Easy to find commits solving specific issues
- Ensures fixes applied consistently across versions
- Links tags to issue tracking system

### Maintenance Releases

Tag commits involving maintenance tasks:

```bash
# Maintenance tag examples
maintenance/code-refactor
update/documentation
maintenance/dependency-upgrade
chore/cleanup
```

**Purpose**:
- Track non-feature, non-bug work
- Document refactoring efforts
- Record dependency updates
- Ensure maintenance properly documented

### Custom Tags

Create tags based on organization-specific needs:

```bash
# Custom tag examples
documentation/v2024-01
performance/optimization
security/audit-passed
deployment/production
testing/load-test-baseline
```

**Use Cases**:
- Classify by focus area
- Mark certification/audit milestones
- Track deployment history
- Document performance baselines

## Tag Categories Summary

| Category | Format | Example | Purpose |
|----------|--------|---------|---------|
| **Release** | v{MAJOR}.{MINOR}.{PATCH} | v1.2.3 | Production releases |
| **Pre-release** | v{VERSION}-{STAGE} | v2.0.0-beta.1 | Testing versions |
| **Feature** | feature/{name} | feature/oauth2 | Feature milestones |
| **Bug Fix** | bugfix/issue-{n} | bugfix/issue-456 | Bug resolution |
| **Maintenance** | maintenance/{type} | maintenance/refactor | Code maintenance |
| **Custom** | {category}/{name} | security/audit-2024 | Organization-specific |

## Implementation

### Step 1: Identify Commit or Release

```bash
# Find commit hash in history
git log --oneline

# Output:
# abc123 Add new authentication feature
# def456 Fix memory leak issue
# ghi789 Update documentation

# Or identify release version
# v1.0.0, v1.1.0, etc.
```

### Step 2: Create Tag

```bash
# Create lightweight tag
git tag v1.0.0

# Create annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag specific commit
git tag -a v1.0.1 abc123 -m "Hotfix release"

# Tag with date
git tag -a v2024.01.15 -m "Monthly release January 2024"
```

**Lightweight vs Annotated**:

| Aspect | Lightweight | Annotated |
|--------|-------------|-----------|
| **Metadata** | Only commit reference | Includes tagger name, email, date, message |
| **Command** | `git tag <name>` | `git tag -a <name> -m "message"` |
| **Best For** | Temporary/local markers | Official releases |
| **Recommended** | No | Yes (for releases) |

### Step 3: Push Tag to GitHub

```bash
# Push specific tag
git push origin v1.0.0

# Push single tag (alternative)
git push origin <tagname>

# Push all tags
git push origin --tags

# Push tags and commits together
git push --follow-tags
```

### Step 4: Verify Tag on GitHub

Navigate to repository on GitHub:
1. Click **Releases** or **Tags** section
2. Verify tag appears in list
3. Check tag points to correct commit

## Tag Protection (Beta Feature)

**As of March 2024**: Tag protection rules in beta and subject to change

### Purpose
Prevent contributors from creating or deleting tags

### Configuration
```bash
# Protected tag patterns (in GitHub UI)
v*.*.*           # Protect all version tags
release-*        # Protect release tags
production-*     # Protect production deployment tags
```

### Protection Rules
- Only designated users/teams can create tags
- Prevents accidental tag deletion
- Enforces tag naming conventions

## Working with Tags

### List Tags

```bash
# List all tags
git tag

# List tags matching pattern
git tag -l "v1.*"

# Output:
# v1.0.0
# v1.0.1
# v1.1.0

# Show tag details
git show v1.0.0
```

### Checkout Tag

```bash
# Checkout specific tag (detached HEAD state)
git checkout v1.0.0

# Create branch from tag
git checkout -b hotfix-v1.0.1 v1.0.0
```

### Delete Tag

```bash
# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0

# Alternative syntax
git push origin :refs/tags/v1.0.0
```

### Move/Recreate Tag

```bash
# Delete existing tag
git tag -d v1.0.0
git push origin --delete v1.0.0

# Recreate on different commit
git tag -a v1.0.0 def456 -m "Corrected release tag"
git push origin v1.0.0
```

## Automated Tagging

### CI/CD Integration

```yaml
# GitHub Actions example
name: Auto-tag on merge

on:
  push:
    branches:
      - main

jobs:
  tag:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Bump version and push tag
        uses: anothrNick/github-tag-action@1.36.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          WITH_V: true
          DEFAULT_BUMP: patch
```

### Semantic Release

```json
// package.json
{
  "scripts": {
    "semantic-release": "semantic-release"
  },
  "devDependencies": {
    "semantic-release": "^19.0.0"
  }
}
```

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Best Practices

### Naming Conventions

✅ **Good Tag Names**:
- `v1.2.3` - Clear version number
- `v2.0.0-rc.1` - Release candidate
- `feature/oauth2` - Descriptive feature tag
- `hotfix/security-patch` - Clear purpose

❌ **Poor Tag Names**:
- `test` - Too vague
- `latest` - Ambiguous, changes over time
- `my-branch` - Confusing with branches
- `v1` - Not specific enough

### Tagging Workflow

```
1. Complete feature/fix
2. Merge to main branch
3. Create tag with semantic version
4. Push tag to remote
5. GitHub creates release (if configured)
6. Deploy from tag (not branch)
```

### Documentation

Document tagging strategy in repository:

```markdown
# TAGGING.md

## Tagging Strategy

### Release Tags
Format: `v{MAJOR}.{MINOR}.{PATCH}`
- v1.0.0: Initial release
- v1.0.x: Patch releases (bug fixes)
- v1.x.0: Minor releases (new features)
- v2.0.0: Major releases (breaking changes)

### Pre-release Tags
Format: `v{VERSION}-{STAGE}.{NUMBER}`
- v2.0.0-alpha.1: Alpha testing
- v2.0.0-beta.2: Beta release
- v2.0.0-rc.1: Release candidate

### Creating Tags
```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```
```

## Integration with Releases

Tags are foundation for GitHub releases:

```bash
# Create tag
git tag -a v1.0.0 -m "Version 1.0.0"
git push origin v1.0.0

# Create release from tag
gh release create v1.0.0 \
  --title "Version 1.0.0" \
  --notes "See CHANGELOG.md for details" \
  dist/*.zip
```

## Critical Notes
- 🎯 Use semantic versioning (v{MAJOR}.{MINOR}.{PATCH}) for release tags
- 💡 Always create annotated tags for official releases (includes metadata)
- ⚠️ Tag protection rules prevent unauthorized tag creation/deletion
- 📊 Tags are foundation for GitHub releases and deployment workflows
- 🔄 Deploy from tags, not branches, for reproducible releases
- ✨ Document tagging strategy in repository for team consistency

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/13-configure-github-tags-organize-repositories)
