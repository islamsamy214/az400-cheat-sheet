# Implement Automation of Git History Documentation

## Key Concepts
- **Automated Documentation**: Automatically generate and publish release documentation
- **Git History Integration**: Include commit history in release notes
- **CI/CD Pipeline Integration**: Automate documentation updates on every release
- **Multi-Source Documentation**: Combine API docs, release notes, and Git history

## Benefits of Automation

| Benefit | Impact |
|---------|--------|
| **Transparency** | Developers track changes easily |
| **Collaboration** | Better communication across teams |
| **Accuracy** | Documentation synchronized with code |
| **Efficiency** | Save time through automation |
| **Consistency** | Standardized documentation format |
| **Traceability** | Link changes to commits and issues |

## Automating API Documentation Generation

### Tools and Approaches

| Tool | Use Case |
|------|----------|
| **OpenAPI (Swagger)** | REST API documentation |
| **Swagger Codegen** | Generate docs from spec |
| **Redocly** | Modern API documentation portal |
| **Stoplight** | API design and documentation |

### Integration with CI/CD

```yaml
# Azure Pipeline example
trigger:
  branches:
    include:
      - main
  tags:
    include:
      - v*

stages:
  - stage: GenerateAPIDocs
    jobs:
      - job: BuildDocs
        steps:
          # Extract API definitions from code
          - task: Npm@1
            displayName: 'Generate OpenAPI Spec'
            inputs:
              command: 'custom'
              customCommand: 'run generate-openapi'
          
          # Build documentation site
          - task: Npm@1
            displayName: 'Build API Documentation'
            inputs:
              command: 'custom'
              customCommand: 'run build-api-docs'
          
          # Publish artifacts
          - task: PublishBuildArtifacts@1
            inputs:
              PathtoPublish: 'api-docs'
              ArtifactName: 'api-documentation'
```

### OpenAPI Generation from Code

```python
# FastAPI automatically generates OpenAPI spec
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    description="Automated API documentation",
    version="1.0.0"
)

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    """
    Get user by ID
    
    Returns user information including name, email, and role.
    """
    return {"user_id": user_id}

# OpenAPI spec automatically available at:
# /docs (Swagger UI)
# /redoc (ReDoc)
# /openapi.json (JSON spec)
```

### Automation Workflow

```
Code Change → CI/CD Triggered → Extract API Definitions → 
Generate OpenAPI Spec → Build Documentation Site → 
Deploy to Hosting Platform → Notify Team
```

## Creating Release Notes from Git History

### Using Git Commands

```bash
# Extract commit messages between versions
git log v1.0.0..v2.0.0 --pretty=format:"%s" > release-notes-draft.txt

# Group by type (if using conventional commits)
git log v1.0.0..v2.0.0 --pretty=format:"%s" | \
  grep "^feat:" > features.txt
git log v1.0.0..v2.0.0 --pretty=format:"%s" | \
  grep "^fix:" > fixes.txt
```

### Using GitChangelog Tool

```bash
# Install gitchangelog
pip install gitchangelog

# Configure .gitchangelog.rc
cat > .gitchangelog.rc << EOF
output_engine = mustache("markdown")
section_regexps = [
    ('New Features', [r'^feat\:']),
    ('Bug Fixes', [r'^fix\:']),
    ('Other', None),
]
EOF

# Generate changelog
gitchangelog > RELEASE_NOTES.md
```

### Automated Release Notes Generation

```yaml
# GitHub Actions workflow
name: Generate Release Notes

on:
  release:
    types: [created]

jobs:
  generate-notes:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Get full history
      
      - name: Get previous tag
        id: previoustag
        run: |
          PREV_TAG=$(git describe --abbrev=0 --tags $(git rev-list --tags --skip=1 --max-count=1))
          echo "tag=$PREV_TAG" >> $GITHUB_OUTPUT
      
      - name: Generate changelog
        run: |
          git log ${{ steps.previoustag.outputs.tag }}..HEAD \
            --pretty=format:"* %s (%an)" \
            --no-merges > CHANGELOG.txt
      
      - name: Update release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh release edit ${{ github.event.release.tag_name }} \
            --notes-file CHANGELOG.txt
```

### Customize Release Notes Format

```bash
# Extract relevant information
git log --pretty=format:"%h - %s (%an, %ar)" v1.0.0..v2.0.0 | \
  grep -E "(feat|fix|breaking)" > release-notes.txt

# Result:
# abc123 - feat: add user authentication (John, 2 days ago)
# def456 - fix: memory leak in processor (Jane, 1 week ago)
# ghi789 - breaking: remove deprecated API (Bob, 3 days ago)
```

## Including Git History in Release Documentation

### Structured Changelog Format

```markdown
# Release v2.0.0 (2024-01-15)

## API Documentation
See [API Reference](https://docs.example.com/api/v2.0.0)

## What's Changed

### New Features
- User authentication with OAuth2 (#123) - @john
- Dashboard analytics view (#145) - @jane
- Export data to CSV (#167) - @bob

### Bug Fixes
- Fixed memory leak in background processor (#156)
- Resolved timeout issues with large datasets (#178)

### Breaking Changes
⚠️ **Important**: This release contains breaking changes

- Removed deprecated `/v1/users` endpoint - use `/v2/users` instead
- Changed authentication header from `X-API-Key` to `Authorization: Bearer`

## Git History

### Commits Since v1.0.0
- abc123: feat: add OAuth2 authentication (John Doe, 2024-01-10)
- def456: fix: resolve memory leak (Jane Smith, 2024-01-12)
- ghi789: docs: update API examples (Bob Johnson, 2024-01-14)

**Full Diff**: https://github.com/org/repo/compare/v1.0.0...v2.0.0

## Contributors
Thanks to @john, @jane, and @bob for their contributions!
```

### Automation Script

```python
# generate_release_docs.py
import subprocess
import sys

def get_commits_between_tags(from_tag, to_tag):
    """Get commits between two tags"""
    cmd = [
        'git', 'log',
        f'{from_tag}..{to_tag}',
        '--pretty=format:%h - %s (%an, %ar)',
        '--no-merges'
    ]
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

def categorize_commits(commits):
    """Categorize commits by type"""
    features = []
    fixes = []
    others = []
    
    for commit in commits.split('\n'):
        if 'feat:' in commit:
            features.append(commit)
        elif 'fix:' in commit:
            fixes.append(commit)
        else:
            others.append(commit)
    
    return features, fixes, others

def generate_release_notes(from_tag, to_tag):
    """Generate formatted release notes"""
    commits = get_commits_between_tags(from_tag, to_tag)
    features, fixes, others = categorize_commits(commits)
    
    notes = f"# Release {to_tag}\n\n"
    
    if features:
        notes += "## New Features\n"
        for feat in features:
            notes += f"- {feat}\n"
        notes += "\n"
    
    if fixes:
        notes += "## Bug Fixes\n"
        for fix in fixes:
            notes += f"- {fix}\n"
        notes += "\n"
    
    if others:
        notes += "## Other Changes\n"
        for other in others:
            notes += f"- {other}\n"
    
    return notes

if __name__ == "__main__":
    from_tag = sys.argv[1]
    to_tag = sys.argv[2]
    print(generate_release_notes(from_tag, to_tag))
```

## Adding Release Notes to Documentation Pipeline

### Azure Pipelines Integration

```yaml
# azure-pipelines.yml
trigger:
  tags:
    include:
      - v*

stages:
  - stage: Documentation
    jobs:
      - job: GenerateAndPublish
        steps:
          # Generate API documentation
          - script: npm run generate-api-docs
            displayName: 'Generate API Docs'
          
          # Generate release notes from Git history
          - script: |
              python generate_release_notes.py v$(PreviousTag) $(Build.SourceBranchName) > RELEASE_NOTES.md
            displayName: 'Generate Release Notes'
          
          # Combine all documentation
          - script: |
              mkdir -p docs/releases/$(Build.SourceBranchName)
              cp RELEASE_NOTES.md docs/releases/$(Build.SourceBranchName)/
              cp api-docs/* docs/api/
            displayName: 'Organize Documentation'
          
          # Publish to documentation site
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'DocsSub'
              appName: 'company-docs'
              package: 'docs'
```

### GitHub Actions Integration

```yaml
# .github/workflows/documentation.yml
name: Documentation Pipeline

on:
  push:
    tags:
      - 'v*'

jobs:
  documentation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
      
      - name: Generate API Documentation
        run: npm run generate-api-docs
      
      - name: Generate Release Notes
        run: |
          PREV_TAG=$(git describe --abbrev=0 --tags $(git rev-list --tags --skip=1 --max-count=1))
          python generate_release_notes.py $PREV_TAG ${{ github.ref_name }}
      
      - name: Build Documentation Site
        run: mkdocs build
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
```

## Automating Documentation Publishing

### Publishing Options

| Platform | Use Case | Setup |
|----------|----------|-------|
| **GitHub Pages** | Public open-source docs | Free, built-in |
| **Azure DevOps Wikis** | Internal team docs | Integrated with ADO |
| **Azure Static Web Apps** | Modern docs site | CI/CD integrated |
| **ReadTheDocs** | Python project docs | Auto-builds from repo |
| **Netlify** | Modern JAMstack docs | Easy deployment |

### GitHub Pages Setup

```yaml
# .github/workflows/gh-pages.yml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build all documentation
        run: |
          npm run build-api-docs
          mkdocs build
      
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
          cname: docs.example.com  # Custom domain
```

### Azure Static Web Apps

```yaml
# .github/workflows/azure-static-web-apps.yml
name: Deploy to Azure Static Web Apps

on:
  push:
    branches:
      - main
  pull_request:
    types: [opened, synchronize, reopened, closed]

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Documentation
        run: |
          npm install
          npm run build-docs
      
      - name: Deploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "docs"
          output_location: "build"
```

## Complete Automation Workflow

```
1. Developer commits code
2. CI/CD pipeline triggered
3. Extract API definitions from code annotations
4. Generate OpenAPI specification
5. Build API documentation site
6. Extract Git commit history
7. Generate release notes from commits
8. Combine API docs + release notes + Git history
9. Publish to central documentation platform
10. Notify team of documentation updates
```

## Critical Notes
- 🎯 Automate API documentation generation to keep it synchronized with code
- 💡 Use Git history to automatically create release notes with context
- ⚠️ Add documentation generation to CI/CD pipeline for every release
- 📊 Combine API docs, release notes, and Git history in one location
- 🔄 Publish documentation automatically to centralized accessible platform
- ✨ Use conventional commits to enable automatic categorization in release notes

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/11-implement-automation-git-history-documentation)
