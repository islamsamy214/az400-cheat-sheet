# Mark Releases with Git Tags

Git tags provide a powerful mechanism for marking specific points in your repository's history, making them ideal for releases, milestones, and version management. Combined with GitHub Actions, tags enable sophisticated release automation workflows.

## Understanding Git Tags and Releases

### What Are Git Tags?

Git tags are references that point to specific commits in your repository history. Unlike branches, tags don't move—they permanently mark a moment in time, making them perfect for releases.

**Two Types of Tags**:

1. **Lightweight Tags**: Simple pointers to commits
2. **Annotated Tags**: Full Git objects with metadata (recommended for releases)

### Tags vs Branches vs Releases

| Feature | Branch | Tag | GitHub Release |
|---------|--------|-----|----------------|
| **Purpose** | Active development | Mark specific version | Distribute software |
| **Mutable** | Yes (moves with commits) | No (fixed reference) | No (tied to tag) |
| **Metadata** | No | Yes (annotated) | Yes (extensive) |
| **Downloadable** | No | No | Yes (with assets) |
| **Changelog** | No | Tag message | Full release notes |

**Relationship**:
```
Commit → Tag → GitHub Release
   ↓       ↓          ↓
  Code   Version   Distribution
```

## Creating and Managing Tags

### Annotated Tags (Recommended)

Annotated tags include full metadata and are recommended for releases:

```bash
# Create annotated tag with message
git tag -a v1.0.0 -m "Release version 1.0.0: Initial stable release

New Features:
- User authentication system
- Dashboard with analytics
- RESTful API endpoints

Breaking Changes:
- API endpoint paths now use /v1/ prefix

Bug Fixes:
- Fixed memory leak in data processing
- Corrected timezone handling"

# Push tag to remote
git push origin v1.0.0

# Push all tags
git push --tags
```

**Annotated Tag Metadata**:
- Tagger name and email
- Tag creation date
- Tag message (like commit message)
- GPG signature (if signed)

### Lightweight Tags

Lightweight tags are simple pointers, useful for temporary markers:

```bash
# Create lightweight tag
git tag v1.0.0-rc1

# Push to remote
git push origin v1.0.0-rc1
```

**When to Use Each Type**:

- **Annotated**: Production releases, public versions, milestone markers
- **Lightweight**: Internal checkpoints, temporary markers, local reference points

### Tag Naming Best Practices

```bash
# ✅ Good: Semantic versioning
git tag -a v1.2.3 -m "Version 1.2.3"
git tag -a v2.0.0 -m "Version 2.0.0 - Major release"

# ✅ Good: Pre-release versions
git tag -a v1.0.0-alpha.1 -m "Alpha release 1"
git tag -a v1.0.0-beta.2 -m "Beta release 2"
git tag -a v1.0.0-rc.1 -m "Release candidate 1"

# ✅ Good: Environment-specific
git tag -a production-2024.01.15 -m "Production deployment Jan 15"

# ❌ Bad: Inconsistent naming
git tag release-1
git tag ver_2_0
git tag v3
```

## Semantic Versioning with Git Tags

### SemVer Format: MAJOR.MINOR.PATCH

```
v1.2.3
 │ │ └─ PATCH: Bug fixes, backward compatible
 │ └─── MINOR: New features, backward compatible
 └───── MAJOR: Breaking changes

```

**Version Increment Rules**:

1. **MAJOR** (v1.0.0 → v2.0.0): Breaking changes, API incompatibility
2. **MINOR** (v1.0.0 → v1.1.0): New features, backward compatible
3. **PATCH** (v1.0.0 → v1.0.1): Bug fixes, backward compatible

### Pre-Release Versions

```bash
# Alpha: Early testing, incomplete features
git tag -a v1.0.0-alpha.1 -m "Alpha release for internal testing"

# Beta: Feature complete, testing phase
git tag -a v1.0.0-beta.1 -m "Beta release for public testing"

# Release Candidate: Final testing before release
git tag -a v1.0.0-rc.1 -m "Release candidate 1"

# Stable: Production-ready
git tag -a v1.0.0 -m "Stable release 1.0.0"
```

**Pre-Release Order** (lowest to highest):
```
v1.0.0-alpha.1 < v1.0.0-alpha.2 < v1.0.0-beta.1 < v1.0.0-rc.1 < v1.0.0
```

### Build Metadata

Add build information (doesn't affect version precedence):

```bash
# Include build number
git tag -a v1.0.0+build.123 -m "Build 123"

# Include commit SHA
git tag -a v1.0.0+sha.$(git rev-parse --short HEAD) -m "Release with commit ref"

# Combined pre-release and build metadata
git tag -a v1.0.0-beta.1+build.456 -m "Beta 1, Build 456"
```

## Automating Release Creation with GitHub Actions

### Automatic Tag Creation on Merge

```yaml
# .github/workflows/auto-tag.yml
name: Auto Tag on Release

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  auto-tag:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Full history for tag operations
      
      - name: Determine version bump
        id: version
        run: |
          # Get latest tag
          LATEST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
          echo "Latest tag: $LATEST_TAG"
          
          # Parse version components
          VERSION=${LATEST_TAG#v}
          IFS='.' read -r MAJOR MINOR PATCH <<< "$VERSION"
          
          # Determine bump type from PR labels
          if [[ "${{ github.event.pull_request.labels.*.name }}" == *"breaking"* ]]; then
            MAJOR=$((MAJOR + 1))
            MINOR=0
            PATCH=0
          elif [[ "${{ github.event.pull_request.labels.*.name }}" == *"feature"* ]]; then
            MINOR=$((MINOR + 1))
            PATCH=0
          else
            PATCH=$((PATCH + 1))
          fi
          
          NEW_VERSION="v$MAJOR.$MINOR.$PATCH"
          echo "new_version=$NEW_VERSION" >> $GITHUB_OUTPUT
          
      - name: Create and push tag
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git tag -a ${{ steps.version.outputs.new_version }} -m "Release ${{ steps.version.outputs.new_version }}"
          git push origin ${{ steps.version.outputs.new_version }}
```

### Create GitHub Release from Tag

```yaml
# .github/workflows/release.yml
name: Create Release

on:
  push:
    tags:
      - 'v*.*.*'  # Matches v1.0.0, v2.1.3, etc.

jobs:
  create-release:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Get tag info
        id: tag
        run: |
          TAG=${GITHUB_REF#refs/tags/}
          echo "tag=$TAG" >> $GITHUB_OUTPUT
          echo "version=${TAG#v}" >> $GITHUB_OUTPUT
      
      - name: Generate changelog
        id: changelog
        run: |
          # Get commits since last tag
          PREVIOUS_TAG=$(git describe --tags --abbrev=0 HEAD^ 2>/dev/null || echo "")
          
          if [ -n "$PREVIOUS_TAG" ]; then
            COMMITS=$(git log $PREVIOUS_TAG..HEAD --pretty=format:"- %s (%h)" --no-merges)
          else
            COMMITS=$(git log HEAD --pretty=format:"- %s (%h)" --no-merges)
          fi
          
          # Save changelog to file
          cat > CHANGELOG.txt << EOF
          # What's Changed
          
          $COMMITS
          
          **Full Changelog**: https://github.com/${{ github.repository }}/compare/$PREVIOUS_TAG...${{ steps.tag.outputs.tag }}
          EOF
      
      - name: Build artifacts
        run: |
          # Build your application
          npm ci
          npm run build
          
          # Create distribution archives
          mkdir -p dist
          tar -czf dist/app-${{ steps.tag.outputs.version }}-linux-x64.tar.gz -C build .
          zip -r dist/app-${{ steps.tag.outputs.version }}-linux-x64.zip build/
      
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          tag_name: ${{ steps.tag.outputs.tag }}
          name: Release ${{ steps.tag.outputs.tag }}
          body_path: CHANGELOG.txt
          draft: false
          prerelease: ${{ contains(steps.tag.outputs.tag, '-alpha') || contains(steps.tag.outputs.tag, '-beta') || contains(steps.tag.outputs.tag, '-rc') }}
          files: |
            dist/*.tar.gz
            dist/*.zip
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Notify team
        run: |
          echo "✅ Release ${{ steps.tag.outputs.tag }} created successfully"
          echo "📦 Artifacts uploaded"
          echo "🔗 https://github.com/${{ github.repository }}/releases/tag/${{ steps.tag.outputs.tag }}"
```

### Manual Release Trigger with Version Input

```yaml
# .github/workflows/manual-release.yml
name: Manual Release Creation

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to release (e.g., 1.2.3)'
        required: true
        type: string
      prerelease:
        description: 'Pre-release type (leave empty for stable)'
        required: false
        type: choice
        options:
          - ''
          - alpha
          - beta
          - rc

jobs:
  create-release:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Validate version format
        run: |
          if ! [[ "${{ inputs.version }}" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
            echo "Error: Invalid version format. Use MAJOR.MINOR.PATCH (e.g., 1.2.3)"
            exit 1
          fi
      
      - name: Build tag name
        id: tag
        run: |
          VERSION="v${{ inputs.version }}"
          if [ -n "${{ inputs.prerelease }}" ]; then
            VERSION="${VERSION}-${{ inputs.prerelease }}.1"
          fi
          echo "tag=$VERSION" >> $GITHUB_OUTPUT
      
      - name: Create and push tag
        run: |
          git config user.name "${{ github.actor }}"
          git config user.email "${{ github.actor }}@users.noreply.github.com"
          git tag -a ${{ steps.tag.outputs.tag }} -m "Release ${{ steps.tag.outputs.tag }}"
          git push origin ${{ steps.tag.outputs.tag }}
      
      - name: Create release
        uses: softprops/action-gh-release@v1
        with:
          tag_name: ${{ steps.tag.outputs.tag }}
          name: Release ${{ steps.tag.outputs.tag }}
          generate_release_notes: true
          prerelease: ${{ inputs.prerelease != '' }}
```

## Advanced Tagging Strategies

### Branch-Specific Tagging

```yaml
# Tag production vs staging releases
name: Environment Release

on:
  push:
    branches:
      - main
      - staging

jobs:
  tag-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Create environment tag
        run: |
          TIMESTAMP=$(date +%Y.%m.%d-%H%M%S)
          BRANCH=${GITHUB_REF#refs/heads/}
          
          if [ "$BRANCH" = "main" ]; then
            TAG="production-$TIMESTAMP"
          else
            TAG="staging-$TIMESTAMP"
          fi
          
          git tag -a $TAG -m "Deployed to $BRANCH at $TIMESTAMP"
          git push origin $TAG
```

### Rolling Version Tags

Maintain moving tags for major versions:

```yaml
# Update v1, v1.2 tags to point to latest
name: Update Rolling Tags

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  update-rolling-tags:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Update major and minor tags
        run: |
          # Extract version components
          TAG=${GITHUB_REF#refs/tags/}
          VERSION=${TAG#v}
          IFS='.' read -r MAJOR MINOR PATCH <<< "$VERSION"
          
          # Update v1 tag
          git tag -fa v$MAJOR -m "Latest v$MAJOR release: $TAG"
          git push origin v$MAJOR --force
          
          # Update v1.2 tag
          git tag -fa v$MAJOR.$MINOR -m "Latest v$MAJOR.$MINOR release: $TAG"
          git push origin v$MAJOR.$MINOR --force
```

## Managing Tags

### Viewing and Searching Tags

```bash
# List all tags
git tag

# List tags matching pattern
git tag -l "v1.*"
git tag -l "v2.*.0"

# Show tag details (annotated tags)
git show v1.0.0

# List tags with commit messages
git tag -n

# List tags sorted by version
git tag -l | sort -V

# Find tag pointing to specific commit
git describe --exact-match <commit-sha>

# Find latest tag
git describe --tags --abbrev=0
```

### Cleaning Up Old Tags

```bash
# Delete local tag
git tag -d v1.0.0-beta.1

# Delete remote tag
git push origin --delete v1.0.0-beta.1

# Delete multiple tags
git tag -d v1.0.0-alpha.1 v1.0.0-alpha.2 v1.0.0-beta.1
git push origin --delete v1.0.0-alpha.1 v1.0.0-alpha.2 v1.0.0-beta.1
```

**Automated Cleanup Workflow**:

```yaml
# Delete old pre-release tags
name: Cleanup Old Tags

on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday
  workflow_dispatch:

jobs:
  cleanup-tags:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Delete old pre-release tags
        run: |
          # Get tags older than 90 days
          for tag in $(git tag -l "v*-alpha*" "v*-beta*"); do
            TAG_DATE=$(git log -1 --format=%ai $tag | cut -d' ' -f1)
            TAG_AGE=$(($(date +%s) - $(date -d "$TAG_DATE" +%s)))
            DAYS_OLD=$((TAG_AGE / 86400))
            
            if [ $DAYS_OLD -gt 90 ]; then
              echo "Deleting old tag: $tag (${DAYS_OLD} days old)"
              git push origin --delete $tag
            fi
          done
```

### Tag Security and Verification

```bash
# Create signed tag (requires GPG setup)
git tag -s v1.0.0 -m "Signed release v1.0.0"

# Verify signed tag
git tag -v v1.0.0

# Push signed tags
git push origin v1.0.0

# Configure Git to always sign tags
git config tag.gpgSign true
```

## Best Practices

### 1. Tag Naming Conventions

```bash
# ✅ Consistent format
v1.0.0, v1.1.0, v2.0.0  # Production releases
v1.0.0-beta.1           # Pre-releases
production-2024.01.15   # Environment deployments

# ❌ Inconsistent formats
release-1, ver_2.0, v3  # Hard to sort and parse
```

### 2. Meaningful Tag Messages

```bash
# ✅ Descriptive messages
git tag -a v2.0.0 -m "Version 2.0.0 - Major Architecture Refactor

Breaking Changes:
- New authentication system (OAuth2)
- API endpoint structure changed

New Features:
- Real-time notifications
- Advanced analytics dashboard

Migration guide: docs/migration-v2.md"

# ❌ Vague messages
git tag -a v2.0.0 -m "new version"
```

### 3. Automated Tag Validation

```yaml
# Validate tag format before creating release
name: Validate Tag

on:
  push:
    tags:
      - 'v*'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Validate tag format
        run: |
          TAG=${GITHUB_REF#refs/tags/}
          
          # Check semantic versioning format
          if ! [[ $TAG =~ ^v[0-9]+\.[0-9]+\.[0-9]+(-[a-z]+\.[0-9]+)?$ ]]; then
            echo "Error: Tag $TAG doesn't match semantic versioning"
            echo "Expected format: vMAJOR.MINOR.PATCH[-prerelease.number]"
            exit 1
          fi
          
          echo "✅ Tag format valid: $TAG"
```

### 4. Tag Protection

Enable tag protection in GitHub repository settings:

**Settings → Tags → Add tag protection rule**:
- Pattern: `v*`
- Require status checks to pass
- Require review before creation
- Restrict who can create matching tags

## Critical Notes

🎯 **Immutable References**: Tags should never be moved or deleted once released—they're permanent markers.

💡 **Semantic Versioning**: Use SemVer (MAJOR.MINOR.PATCH) for consistent, predictable versioning.

⚠️ **Annotated Tags**: Always use annotated tags (`-a`) for releases—they include metadata and are verifiable.

📊 **Automation**: Automate tag creation from PR labels or commit messages to reduce manual errors.

🔄 **Rolling Tags**: Maintain major version tags (v1, v2) that move to latest compatible release for easy upgrades.

✨ **GitHub Releases**: Always create GitHub Release for tags—provides changelog, assets, and user-friendly interface.

## Quick Reference

### Tag Commands

```bash
# Create annotated tag
git tag -a v1.0.0 -m "Release message"

# Create lightweight tag
git tag v1.0.0

# Push specific tag
git push origin v1.0.0

# Push all tags
git push --tags

# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0

# List tags
git tag -l "v1.*"

# Show tag details
git show v1.0.0
```

### SemVer Quick Reference

```
MAJOR.MINOR.PATCH[-prerelease][+build]

Examples:
v1.0.0           # Stable release
v1.0.0-alpha.1   # Alpha pre-release
v1.0.0-beta.2    # Beta pre-release
v1.0.0-rc.1      # Release candidate
v1.0.0+build.123 # With build metadata
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/learn-continuous-integration-github-actions/7-mark-releases-git-tags)
