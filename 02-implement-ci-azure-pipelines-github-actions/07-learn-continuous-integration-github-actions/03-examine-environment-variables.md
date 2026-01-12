# Examine Environment Variables

Environment variables are essential for creating flexible and maintainable GitHub Actions workflows. They allow you to configure behavior, pass data between steps, and adapt workflows to different environments without hardcoding values.

## Understanding Environment Variables in GitHub Actions

Environment variables provide a way to store and access configuration data within workflows. They can be set at different scopes and used throughout your automation pipeline for consistent and secure configuration management.

### Variable Scopes and Hierarchy

GitHub Actions supports environment variables at multiple levels:

| Scope | Availability | Use Case | Priority |
|-------|--------------|----------|----------|
| **Workflow-level** | All jobs in workflow | Global configuration | Lowest (overridden by job/step) |
| **Job-level** | All steps in specific job | Job-specific settings | Medium (overridden by step) |
| **Step-level** | Only that specific step | Step-specific values | Highest (overrides all) |

```yaml
name: Multi-level Environment Variables

# Workflow-level variables (available everywhere)
env:
  NODE_VERSION: "20"
  BUILD_CONFIGURATION: "Release"

jobs:
  build:
    runs-on: ubuntu-latest

    # Job-level variables (available to all steps in this job)
    env:
      DATABASE_NAME: "production_db"
      API_TIMEOUT: "30000"

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}  # Uses workflow-level

      - name: Run tests with custom config
        run: npm test
        # Step-level variables (only available in this step)
        env:
          TEST_ENVIRONMENT: "ci"
          LOG_LEVEL: "debug"
```

**Variable Resolution Order** (highest priority first):
1. Step-level `env`
2. Job-level `env`
3. Workflow-level `env`
4. Built-in GitHub variables

## Built-in GitHub Environment Variables

GitHub automatically provides numerous environment variables with information about the workflow context.

### Essential Built-in Variables

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `GITHUB_WORKFLOW` | Name of the workflow | `"CI Pipeline"` |
| `GITHUB_REPOSITORY` | Repository name (owner/repo) | `"microsoft/vscode"` |
| `GITHUB_REF` | Branch or tag reference | `"refs/heads/main"` |
| `GITHUB_SHA` | Commit SHA that triggered workflow | `"ffac537e6cbb..."` |
| `GITHUB_ACTOR` | Username who triggered workflow | `"octocat"` |
| `GITHUB_EVENT_NAME` | Event that triggered workflow | `"push"` |
| `RUNNER_OS` | Operating system of runner | `"Linux"` |
| `GITHUB_WORKSPACE` | Workspace directory path | `"/home/runner/work/repo/repo"` |
| `GITHUB_RUN_NUMBER` | Sequential run number | `"42"` |

### Using Built-in Variables

```yaml
steps:
  - name: Display workflow context
    run: |
      echo "Workflow: $GITHUB_WORKFLOW"
      echo "Repository: $GITHUB_REPOSITORY"
      echo "Branch: ${GITHUB_REF#refs/heads/}"  # Extract branch name
      echo "Commit: $GITHUB_SHA"
      echo "Actor: $GITHUB_ACTOR"
      echo "Event: $GITHUB_EVENT_NAME"
      echo "Runner OS: $RUNNER_OS"
      echo "Run Number: $GITHUB_RUN_NUMBER"
```

**Important Naming Rules**:
- ✅ Built-in variables use the `GITHUB_` prefix
- ❌ You **cannot** create custom variables with the `GITHUB_` prefix
- ✅ Variable names are **case-sensitive**

## Practical Environment Variable Patterns

### Pattern 1: Configuration Management

```yaml
name: Environment-specific Deployment

env:
  APP_NAME: "my-awesome-app"
  DOCKER_REGISTRY: "ghcr.io"

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    env:
      ENVIRONMENT: "staging"
      API_URL: "https://api.staging.example.com"
      DATABASE_TIER: "basic"
      LOG_LEVEL: "debug"
    steps:
      - name: Deploy to staging
        run: |
          echo "Deploying $APP_NAME to $ENVIRONMENT"
          echo "API URL: $API_URL"

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    env:
      ENVIRONMENT: "production"
      API_URL: "https://api.example.com"
      DATABASE_TIER: "premium"
      LOG_LEVEL: "info"
    steps:
      - name: Deploy to production
        run: |
          echo "Deploying $APP_NAME to $ENVIRONMENT"
```

### Pattern 2: Dynamic Variable Creation

```yaml
steps:
  - name: Generate build metadata
    run: |
      BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
      BUILD_NUMBER=$GITHUB_RUN_NUMBER
      COMMIT_SHORT=${GITHUB_SHA::8}

      # Set for later steps in this job
      echo "BUILD_DATE=$BUILD_DATE" >> $GITHUB_ENV
      echo "BUILD_NUMBER=$BUILD_NUMBER" >> $GITHUB_ENV
      echo "COMMIT_SHORT=$COMMIT_SHORT" >> $GITHUB_ENV

  - name: Use generated variables
    run: |
      echo "Build Date: $BUILD_DATE"
      echo "Build Number: $BUILD_NUMBER"
      echo "Commit: $COMMIT_SHORT"
```

### Pattern 3: Conditional Environment Variables

```yaml
steps:
  - name: Set environment-specific variables
    run: |
      if [ "$GITHUB_REF" = "refs/heads/main" ]; then
        echo "LOG_LEVEL=info" >> $GITHUB_ENV
        echo "CACHE_TTL=3600" >> $GITHUB_ENV
      elif [ "$GITHUB_REF" = "refs/heads/develop" ]; then
        echo "LOG_LEVEL=debug" >> $GITHUB_ENV
        echo "CACHE_TTL=300" >> $GITHUB_ENV
      else
        echo "LOG_LEVEL=warn" >> $GITHUB_ENV
        echo "CACHE_TTL=60" >> $GITHUB_ENV
      fi
```

## Security Best Practices

### Principle 1: Sensitive Data Handling

```yaml
# ❌ DON'T: Store secrets in plain environment variables
env:
  DATABASE_PASSWORD: 'super-secret-password'  # Visible in logs!

# ✅ DO: Use GitHub Secrets for sensitive data
env:
  DATABASE_HOST: 'db.example.com'
  DATABASE_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}  # Secure!
```

### Principle 2: Environment Variable Validation

```yaml
steps:
  - name: Validate required environment variables
    run: |
      required_vars=("API_URL" "DATABASE_HOST" "ENVIRONMENT")
      
      for var in "${required_vars[@]}"; do
        if [ -z "${!var}" ]; then
          echo "❌ ERROR: Required variable $var is not set"
          exit 1
        else
          echo "✅ OK: $var is set"
        fi
      done
```

### Principle 3: Minimal Exposure Scope

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        # No sensitive data needed
      
      - name: Deploy application
        env:
          # Secrets only available in this specific step
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: ./deploy.sh --token="$DEPLOY_TOKEN"
```

## Advanced Techniques

### Multi-line Environment Variables

```yaml
steps:
  - name: Set multi-line variable
    run: |
      {
        echo 'CONFIG_JSON<<EOF'
        cat << 'INNER_EOF'
{
  "api": {
    "url": "https://api.example.com",
    "timeout": 30000
  }
}
INNER_EOF
        echo EOF
      } >> $GITHUB_ENV
  
  - name: Use multi-line variable
    run: echo "$CONFIG_JSON" | jq '.api.url'
```

### Default Values and Fallbacks

```yaml
steps:
  - name: Use with fallback
    run: |
      # Use environment variable or fallback to default
      NODE_VER="${NODE_VERSION:-18}"
      ENV="${ENVIRONMENT:-dev}"
      
      echo "Using Node.js version: $NODE_VER"
      echo "Deploying to environment: $ENV"
```

## Critical Notes

🎯 **Scope Wisely**: Use the most specific scope (step > job > workflow) to minimize exposure.

💡 **Validate Early**: Check required variables exist at start of workflow to fail fast.

⚠️ **No Secrets in Env**: Never put sensitive data directly in `env:`. Always use `secrets.`.

📊 **Use Expressions**: `${{ env.VAR }}` for GitHub Actions syntax, `$VAR` for shell commands.

🔄 **Dynamic Generation**: Use `$GITHUB_ENV` to create variables dynamically in steps.

✨ **Document Variables**: Comment your environment variables for team understanding.

## Quick Reference

### Setting Variables

```yaml
# Workflow level
env:
  VAR: value

# Job level
jobs:
  build:
    env:
      VAR: value

# Step level
steps:
  - name: Step
    env:
      VAR: value
    run: command

# Dynamically (in step)
run: echo "VAR=value" >> $GITHUB_ENV
```

### Accessing Variables

```yaml
# In GitHub Actions expressions
${{ env.VAR }}

# In shell commands
$VAR
${VAR}
${VAR:-default}  # With fallback
```

### Common Built-in Variables

```yaml
${{ github.repository }}       # owner/repo
${{ github.ref_name }}         # branch or tag name
${{ github.sha }}              # commit SHA
${{ github.actor }}            # username
${{ github.event_name }}       # push, pull_request, etc.
${{ runner.os }}               # Linux, Windows, macOS
${{ github.run_number }}       # sequential number
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/learn-continuous-integration-github-actions/3-examine-environment-variables)
