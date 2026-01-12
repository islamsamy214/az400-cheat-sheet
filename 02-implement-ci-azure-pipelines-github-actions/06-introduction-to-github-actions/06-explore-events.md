# Explore Events

GitHub Actions workflows are triggered by events - specific activities that happen in your repository or on a schedule. Events are defined by the `on` clause in your workflow definition and determine when your automation runs.

## Event Categories

| Category | Examples | Use Case |
|----------|----------|----------|
| **Code Repository** | push, pull_request, release | CI/CD pipelines |
| **Scheduled** | schedule (cron) | Nightly builds, reports |
| **Manual** | workflow_dispatch | On-demand deployments |
| **Webhook** | issues, gollum, deployment | Project automation |
| **External** | repository_dispatch | Third-party integrations |

## Code Repository Events

The most common triggers respond to code changes in your repository.

### Basic Push and Pull Request Events

```yaml
# Single event
on: push

# Multiple events
on: [push, pull_request]

# Detailed configuration
on:
  push
  pull_request
```

### Filtered Events with Branches and Paths

```yaml
on:
  push:
    branches:
      - main
      - develop
      - 'releases/**'  # Wildcard patterns
    branches-ignore:
      - 'experimental/**'
    paths:
      - 'src/**'  # Only run when src changes
      - '!docs/**'  # Ignore docs changes
  
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]  # Specific PR actions
```

**Branch Patterns**:
- `main` - Exact match
- `releases/*` - Match one level: `releases/v1`, `releases/v2`
- `releases/**` - Match multiple levels: `releases/v1/hotfix`, `releases/v2/patch`

**Path Patterns**:
- `src/**` - Include: all files under src
- `!docs/**` - Exclude: all files under docs
- `**.js` - Include: all JavaScript files
- `. -name "*.md" | wc -l && echo "units created"*.md` - Exclude: all Markdown files

### Common Repository Events

| Event | Triggers When | CI/CD Use Case |
|-------|---------------|----------------|
| **push** | Code pushed to branch | Build and test every commit |
| **pull_request** | PR opened/updated/closed | Code review automation |
| **pull_request_target** | PR from fork (safe) | Dangerous actions on forks |
| **release** | Release published | Deploy to production |
| **create** | Branch/tag created | Initialize environments |
| **delete** | Branch/tag deleted | Cleanup resources |
| **workflow_run** | Another workflow completes | Chained workflows |

**Pull Request Types**:
```yaml
on:
  pull_request:
    types:
      - opened        # PR created
      - reopened      # Closed PR reopened
      - synchronize   # New commits pushed
      - ready_for_review  # Draft marked ready
      - closed        # PR closed/merged
```

## Scheduled Events

Schedule workflows to run at specific times using cron syntax.

```yaml
on:
  schedule:
    # Runs every weekday at 8 AM UTC
    - cron: "0 8 * * 1-5"
    
    # Runs every Sunday at midnight UTC
    - cron: "0 0 * * 0"
    
    # Multiple schedules
    - cron: "0 2 * * *"   # Daily at 2 AM
    - cron: "0 14 * * FRI"  # Every Friday at 2 PM
```

### Cron Syntax Breakdown

```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day (1 - 31)
│ │ │ ┌───────────── month (1 - 12 or JAN-DEC)
│ │ │ │ ┌───────────── weekday (0 - 6 or SUN-SAT)
│ │ │ │ │
* * * * *
```

**Cron Examples**:
- `'0 9-17 * * 1-5'` - Every hour from 9 AM to 5 PM, Monday-Friday
- `'30 2 * * *'` - Daily at 2:30 AM
- `'0 0 1 * *'` - First day of every month at midnight
- `'0 */6 * * *'` - Every 6 hours
- `'0 0 * * SUN'` - Every Sunday at midnight

**Important Notes**:
- ⚠️ Always quote cron strings in YAML
- 🕐 All times are in UTC
- ⏱️ Minimum interval: 5 minutes
- �� High-traffic times may have delays

## Manual Events

Trigger workflows manually from the GitHub Actions tab using `workflow_dispatch`.

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Deployment environment"
        required: true
        default: "staging"
        type: choice
        options:
          - staging
          - production
      version:
        description: "Version to deploy"
        required: true
        type: string
      logLevel:
        description: "Log level"
        required: false
        default: "info"
        type: choice
        options:
          - debug
          - info
          - warning
          - error
      dryRun:
        description: "Dry run (no actual deployment)"
        required: false
        type: boolean
        default: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Deploy
      run: |
        echo "Deploying to ${{ inputs.environment }}"
        echo "Version: ${{ inputs.version }}"
        echo "Log level: ${{ inputs.logLevel }}"
        echo "Dry run: ${{ inputs.dryRun }}"
```

**Input Types**:
- `string` - Text input
- `choice` - Dropdown selection
- `boolean` - Checkbox
- `environment` - Environment picker

**Note**: The workflow file must exist in the default branch to appear in the manual trigger UI.

## Webhook Events

GitHub provides many webhook events for repository activities.

```yaml
on:
  # Issues
  issues:
    types: [opened, edited, closed, labeled]
  
  # Issue comments
  issue_comment:
    types: [created, edited, deleted]
  
  # Wiki pages
  gollum:
  
  # Releases
  release:
    types: [published, created, edited]
  
  # Deployments
  deployment:
  deployment_status:
  
  # Packages
  package:
    types: [published, updated]
```

### Issue Automation Example

```yaml
name: Auto-Label Issues
on:
  issues:
    types: [opened]

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
    - name: Add bug label
      if: contains(github.event.issue.title, 'bug')
      uses: actions/github-script@v7
      with:
        script: |
          github.rest.issues.addLabels({
            owner: context.repo.owner,
            repo: context.repo.repo,
            issue_number: context.issue.number,
            labels: ['bug']
          })
```

## External Events

Use `repository_dispatch` to trigger workflows from external systems via GitHub's REST API.

```yaml
on:
  repository_dispatch:
    types: [deploy-staging, run-tests, update-docs]

jobs:
  handle-dispatch:
    runs-on: ubuntu-latest
    steps:
    - name: Handle event
      run: |
        echo "Event type: ${{ github.event.action }}"
        echo "Client payload: ${{ toJson(github.event.client_payload) }}"
```

### Triggering Externally

```bash
curl -X POST   -H "Authorization: token YOUR_TOKEN"   -H "Accept: application/vnd.github.v3+json"   https://api.github.com/repos/OWNER/REPO/dispatches   -d '{
    "event_type":"deploy-staging",
    "client_payload":{
      "environment":"staging",
      "version":"1.2.3"
    }
  }'
```

**Use Cases**:
- Trigger from CI/CD tools (Jenkins, Azure DevOps)
- Trigger from monitoring systems (alert → workflow)
- Trigger from custom applications
- Cross-repository workflows

## Event Filter Patterns

### Combining Events

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  release:
    types: [published]
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch:
```

### Complex Branch Filters

```yaml
on:
  push:
    branches:
      - main
      - develop
      - 'feature/**'
      - 'hotfix/*'
      - '!experimental/**'  # Exclude experimental branches
```

### Path Filters for Monorepos

```yaml
name: API Service CI
on:
  push:
    paths:
      - 'services/api/**'
      - 'shared/models/**'
      - 'package*.json'
    paths-ignore:
      - '**/*.md'
      - 'docs/**'
```

### Tag-Based Releases

```yaml
on:
  push:
    tags:
      - 'v*'        # v1, v2, v1.0, v2.1.3
      - 'v[0-9]+.*' # v1.*, v2.*, v10.*
```

## Best Practices for Events

### 1. Be Specific with Filters

```yaml
# ❌ Too broad - runs on every push
on: push

# ✅ Specific - runs only on main/develop
on:
  push:
    branches: [main, develop]
    paths:
      - 'src/**'
      - 'tests/**'
    paths-ignore:
      - '**/*.md'
```

### 2. Combine Related Events

```yaml
# Group similar triggers in one workflow
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:  # Allow manual runs
```

### 3. Use Appropriate Event Types

```yaml
# ✅ Use specific PR types for efficiency
on:
  pull_request:
    types: [opened, synchronize, reopened]
    # Skip ready_for_review, labeled, etc.
```

### 4. Test Manually First

```yaml
# Always include workflow_dispatch during development
on:
  push:
    branches: [main]
  workflow_dispatch:  # Manual testing
```

### 5. Monitor Usage

Review workflow runs to optimize event configuration. Remove unnecessary triggers.

## Event Context

Access event details in workflows:

```yaml
steps:
  - name: Event information
    run: |
      echo "Event name: ${{ github.event_name }}"
      echo "Repository: ${{ github.repository }}"
      echo "Ref: ${{ github.ref }}"
      echo "SHA: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
      
      # Event-specific data
      echo "PR number: ${{ github.event.pull_request.number }}"
      echo "Issue title: ${{ github.event.issue.title }}"
      echo "Release tag: ${{ github.event.release.tag_name }}"
```

## Critical Notes

🎯 **UTC Time Zone**: All scheduled triggers use UTC, not local time. Convert your local time to UTC for scheduling.

💡 **Event Filters**: Use `paths` and `branches` filters to reduce unnecessary workflow runs and save minutes.

⚠️ **Public Repos**: Be cautious with `pull_request_target` - it runs in the context of the base repo with access to secrets.

📊 **Workflow Dispatch**: Must exist in default branch to be triggerable manually from UI.

🔄 **Event Payload**: Full event payload available in `${{ toJson(github.event) }}` for debugging.

✨ **Multiple Schedules**: You can have multiple cron schedules in a single workflow for different frequencies.

## Quick Reference

### Event Syntax Patterns

```yaml
# Single event
on: push

# Multiple events
on: [push, pull_request]

# Detailed configuration
on:
  push:
    branches: [main]
    paths: ['src/**']
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch:
```

### Common Event Combinations

| Scenario | Events | Use Case |
|----------|--------|----------|
| **CI** | `push, pull_request` | Build/test every change |
| **CD** | `push` (main branch only) | Deploy production |
| **Release** | `release` (published) | Create release artifacts |
| **Nightly** | `schedule` (cron) | Nightly builds/tests |
| **On-Demand** | `workflow_dispatch` | Manual deployments |

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-github-actions/6-explore-events)
