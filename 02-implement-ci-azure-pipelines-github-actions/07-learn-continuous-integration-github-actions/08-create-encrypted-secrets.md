# Create Encrypted Secrets

GitHub encrypted secrets provide secure storage for sensitive information like API keys, passwords, and tokens. Secrets are encrypted before storage and automatically masked in workflow logs, making them essential for secure CI/CD pipelines.

## Understanding GitHub Secrets

### What Are Encrypted Secrets?

Encrypted secrets are environment variables stored securely in GitHub and made available to GitHub Actions workflows. They provide a secure way to store and use sensitive information without exposing it in your code or logs.

**Key Characteristics**:

✅ **Encrypted at Rest**: Secrets are encrypted using libsodium sealed boxes before storage  
✅ **Automatic Masking**: Secret values are automatically masked in workflow logs  
✅ **Controlled Access**: Secrets are only accessible to workflows with proper permissions  
✅ **Immutable**: Once created, secret values can be updated but not viewed through the UI

### Why Use Secrets?

**❌ Without Secrets** (Insecure):
```yaml
# NEVER do this - credentials exposed in code
- name: Deploy to production
  run: |
    curl -X POST https://api.example.com/deploy \
      -H "Authorization: Bearer sk_live_abc123xyz789"  # EXPOSED!
```

**✅ With Secrets** (Secure):
```yaml
# Secure: credentials stored as secrets
- name: Deploy to production
  run: |
    curl -X POST https://api.example.com/deploy \
      -H "Authorization: Bearer ${{ secrets.API_TOKEN }}"  # MASKED in logs
```

**Log Output**:
```
# What you see in logs when using secrets:
Authorization: Bearer ***
```

## Secret Scopes and Hierarchy

GitHub provides three levels of secret scopes, each with different visibility and access patterns:

### 1. Repository-Level Secrets

**Scope**: Available to all workflows in a single repository

**Use Cases**:
- Project-specific API keys
- Service credentials unique to one application
- Database connection strings for this repository

**Creating Repository Secrets**:

1. Navigate to: `Repository → Settings → Secrets and variables → Actions`
2. Click: `New repository secret`
3. Enter: Name and value
4. Click: `Add secret`

**CLI Creation**:
```bash
# Using GitHub CLI
gh secret set API_TOKEN --body "your-secret-value"

# From file
gh secret set SSH_PRIVATE_KEY < ~/.ssh/id_rsa

# From stdin
echo "my-secret-value" | gh secret set MY_SECRET

# Multiple repos (for organization)
gh secret set SHARED_SECRET --repos org/repo1,org/repo2
```

### 2. Organization-Level Secrets

**Scope**: Shared across multiple repositories within an organization

**Use Cases**:
- Shared infrastructure credentials (AWS, Azure)
- Organization-wide API keys (Slack, monitoring services)
- Common service tokens used by multiple projects

**Access Control**: Can be limited to:
- All repositories
- Private repositories only
- Selected repositories

**Creating Organization Secrets**:

1. Navigate to: `Organization → Settings → Secrets and variables → Actions`
2. Click: `New organization secret`
3. Configure: Name, value, and repository access
4. Click: `Add secret`

**Example Organization Secret Configuration**:
```yaml
Name: AWS_ACCESS_KEY_ID
Value: AKIAIOSFODNN7EXAMPLE
Repository access: Selected repositories
  - myorg/app-frontend
  - myorg/app-backend
  - myorg/app-infrastructure
```

### 3. Environment-Level Secrets

**Scope**: Specific to deployment environments (production, staging, development)

**Use Cases**:
- Environment-specific API endpoints
- Per-environment credentials
- Deployment keys for specific stages

**Protection Rules**: Environment secrets support:
- Required reviewers before deployment
- Wait timers before deployment
- Deployment branch restrictions

**Creating Environment Secrets**:

1. Navigate to: `Repository → Settings → Environments`
2. Select or create environment (e.g., "production")
3. Under "Environment secrets", click: `Add secret`
4. Enter: Name and value
5. Configure: Protection rules (optional)
6. Click: `Add secret`

**Using Environment Secrets in Workflows**:
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # Uses production environment secrets
    steps:
      - name: Deploy to production
        run: |
          echo "API Endpoint: ${{ secrets.API_ENDPOINT }}"  # production value
          echo "Database: ${{ secrets.DATABASE_URL }}"      # production value
```

### Secret Hierarchy and Precedence

When secrets with the same name exist at multiple levels:

```
Environment Secrets (highest priority)
    ↓
Organization Secrets
    ↓
Repository Secrets (lowest priority)
```

**Example**:
```yaml
# Repository secret: API_KEY = "repo-key-123"
# Organization secret: API_KEY = "org-key-456"
# Production environment secret: API_KEY = "prod-key-789"

jobs:
  deploy:
    environment: production
    steps:
      - run: echo ${{ secrets.API_KEY }}
      # Output (masked): *** (actual value: "prod-key-789")
      # Environment secret takes precedence
```

## Creating and Managing Secrets

### Secret Naming Conventions

**✅ Good Secret Names**:
```
AWS_ACCESS_KEY_ID
DATABASE_PASSWORD
SLACK_WEBHOOK_URL
NPM_TOKEN
AZURE_CREDENTIALS
DOCKER_REGISTRY_PASSWORD
```

**❌ Avoid These Patterns**:
```
secret1                    # Too generic
my-api-key                 # Use underscores, not hyphens
ApiKey                     # Use UPPER_SNAKE_CASE
GITHUB_TOKEN               # Reserved name (automatically provided)
```

**Best Practices**:
- Use `UPPER_SNAKE_CASE`
- Be descriptive and specific
- Include service or purpose in name
- Avoid special characters (use underscores only)

### Secret Management Workflow

```yaml
# .github/workflows/manage-secrets.yml
name: Secret Management Example

on: [push]

jobs:
  use-secrets:
    runs-on: ubuntu-latest
    steps:
      # Access repository secrets
      - name: Use repository secret
        run: echo "Using API token: ${{ secrets.API_TOKEN }}"
      
      # Access organization secrets
      - name: Use organization secret
        run: |
          echo "Org AWS Key: ${{ secrets.AWS_ACCESS_KEY_ID }}"
          echo "Org AWS Secret: ${{ secrets.AWS_SECRET_ACCESS_KEY }}"
      
      # Conditional secret usage
      - name: Use secret conditionally
        if: github.ref == 'refs/heads/main'
        run: echo "Production API: ${{ secrets.PRODUCTION_API_KEY }}"
      
      # Pass secrets to actions
      - name: Deploy with secrets
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
```

### Updating Existing Secrets

**Through GitHub UI**:
1. Navigate to: `Settings → Secrets → Actions`
2. Click secret name to update
3. Enter new value
4. Click: `Update secret`

**Through GitHub CLI**:
```bash
# Update secret with new value
gh secret set API_TOKEN --body "new-secret-value"

# Update from file
gh secret set SSH_KEY < ~/.ssh/new_key

# Remove secret
gh secret remove API_TOKEN
```

**Rotation Workflow** (Automated):
```yaml
# Workflow to remind about secret rotation
name: Secret Rotation Reminder

on:
  schedule:
    - cron: '0 0 1 */3 *'  # First day of every quarter

jobs:
  rotation-reminder:
    runs-on: ubuntu-latest
    steps:
      - name: Create rotation issue
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: '🔑 Quarterly Secret Rotation Reminder',
              body: `
              Time to rotate the following secrets:
              
              - [ ] \`AWS_ACCESS_KEY_ID\`
              - [ ] \`AWS_SECRET_ACCESS_KEY\`
              - [ ] \`DATABASE_PASSWORD\`
              - [ ] \`API_TOKENS\`
              
              **Rotation Checklist**:
              1. Generate new credentials
              2. Update secrets in GitHub
              3. Deploy with new credentials
              4. Verify functionality
              5. Revoke old credentials
              `,
              labels: ['security', 'maintenance']
            });
```

## Security Best Practices

### Principle 1: Minimize Secret Scope

**✅ Use Minimal Scope**:
```yaml
# Good: Use environment-specific secrets
jobs:
  deploy-staging:
    environment: staging
    steps:
      - run: deploy.sh ${{ secrets.STAGING_API_KEY }}
  
  deploy-production:
    environment: production
    steps:
      - run: deploy.sh ${{ secrets.PRODUCTION_API_KEY }}
```

**❌ Avoid Broad Scope**:
```yaml
# Bad: Single secret for all environments
jobs:
  deploy:
    steps:
      - run: deploy.sh ${{ secrets.API_KEY }}  # Same key for all environments
```

### Principle 2: Never Log or Echo Secrets

**❌ NEVER Do This**:
```yaml
# DANGEROUS: Exposes secret in logs
- name: Debug secret
  run: echo "API Key is: ${{ secrets.API_KEY }}"  # DON'T DO THIS!

# DANGEROUS: Secret in command that might be logged
- name: Deploy
  run: |
    set -x  # Enables command echoing - will expose secrets!
    curl -H "Authorization: Bearer ${{ secrets.TOKEN }}" api.example.com
```

**✅ Safe Practices**:
```yaml
# Good: Secret used but never echoed
- name: Deploy safely
  run: curl -H "Authorization: Bearer ${{ secrets.TOKEN }}" api.example.com
  # Logs show: Authorization: Bearer ***

# Good: Use secrets in environment variables
- name: Deploy with env var
  env:
    API_TOKEN: ${{ secrets.API_TOKEN }}
  run: |
    # Use $API_TOKEN in script (masked in logs)
    deploy-script.sh
```

### Principle 3: Validate and Sanitize Secret Usage

```yaml
# Validate secrets before use
- name: Validate credentials
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    if [ -z "$API_KEY" ]; then
      echo "Error: API_KEY secret not set"
      exit 1
    fi
    
    if [ ${#API_KEY} -lt 32 ]; then
      echo "Error: API_KEY appears invalid (too short)"
      exit 1
    fi
    
    echo "✅ Credentials validated"
```

### Principle 4: Least Privilege Access

**Organization Secret Access Control**:
```
# Configure organization secrets with minimal access:

Secret: PRODUCTION_DATABASE_PASSWORD
Access: Selected repositories only
  - myorg/production-api
  - myorg/production-worker

NOT:
Access: All repositories  # Too broad!
```

**Environment Protection Rules**:
```yaml
# Settings → Environments → production
Protection rules:
  ✅ Required reviewers: 2 reviewers minimum
  ✅ Wait timer: 5 minutes before deployment
  ✅ Deployment branches: Only 'main' branch
```

## Advanced Secret Patterns

### 1. Multi-Value Secrets (JSON Format)

Store related credentials together:

```yaml
# Secret: AZURE_CREDENTIALS (value is JSON)
{
  "clientId": "12345678-1234-1234-1234-123456789012",
  "clientSecret": "secret-value-here",
  "subscriptionId": "12345678-1234-1234-1234-123456789012",
  "tenantId": "12345678-1234-1234-1234-123456789012"
}

# Usage in workflow
- name: Azure Login
  uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}  # Entire JSON passed

# Or parse individual values
- name: Use parsed credentials
  env:
    AZURE_CREDS: ${{ secrets.AZURE_CREDENTIALS }}
  run: |
    CLIENT_ID=$(echo $AZURE_CREDS | jq -r '.clientId')
    # Use parsed values...
```

### 2. Secret Inheritance and Composition

```yaml
# Compose secrets for complex configurations
- name: Create combined config
  env:
    DB_HOST: ${{ secrets.DATABASE_HOST }}
    DB_PORT: ${{ secrets.DATABASE_PORT }}
    DB_USER: ${{ secrets.DATABASE_USER }}
    DB_PASS: ${{ secrets.DATABASE_PASSWORD }}
  run: |
    # Create connection string from individual secrets
    CONNECTION_STRING="postgres://${DB_USER}:${DB_PASS}@${DB_HOST}:${DB_PORT}/mydb"
    
    # Use in application (CONNECTION_STRING value is masked in logs)
    export DATABASE_URL="$CONNECTION_STRING"
    npm start
```

### 3. Secret Validation Before Deployment

```yaml
# Workflow to test secrets are valid before production deployment
name: Validate Secrets

on:
  workflow_dispatch:

jobs:
  test-secrets:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [staging, production]
    environment: ${{ matrix.environment }}
    
    steps:
      - name: Test API credentials
        env:
          API_KEY: ${{ secrets.API_KEY }}
          API_ENDPOINT: ${{ secrets.API_ENDPOINT }}
        run: |
          # Test authentication
          RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" \
            -H "Authorization: Bearer $API_KEY" \
            "$API_ENDPOINT/health")
          
          if [ "$RESPONSE" = "200" ]; then
            echo "✅ ${{ matrix.environment }} credentials valid"
          else
            echo "❌ ${{ matrix.environment }} credentials invalid (HTTP $RESPONSE)"
            exit 1
          fi
```

## Common Pitfalls and Solutions

### Pitfall 1: Secret Not Available

**Problem**: `secrets.MY_SECRET` is undefined

**Solutions**:
```yaml
# 1. Check secret exists at correct scope
- run: |
    if [ -z "${{ secrets.MY_SECRET }}" ]; then
      echo "Error: MY_SECRET not configured"
      exit 1
    fi

# 2. Verify secret name (case-sensitive)
- run: echo "${{ secrets.API_KEY }}"     # ✅ Correct
- run: echo "${{ secrets.api_key }}"     # ❌ Wrong case

# 3. Check environment is specified if using environment secrets
deploy:
  environment: production  # Required for environment secrets
  steps:
    - run: echo "${{ secrets.PROD_SECRET }}"
```

### Pitfall 2: Secret Exposed in Error Messages

**Problem**: Error messages reveal secret values

**Solution**:
```yaml
# ❌ Bad: Secret might appear in error
- name: API call
  run: |
    response=$(curl -H "Authorization: Bearer ${{ secrets.TOKEN }}" api.example.com)
    echo "Response: $response"  # Might contain secret in error message

# ✅ Good: Sanitize errors
- name: API call
  env:
    API_TOKEN: ${{ secrets.API_TOKEN }}
  run: |
    response=$(curl -s -H "Authorization: Bearer $API_TOKEN" api.example.com) || {
      echo "Error: API call failed (credentials hidden)"
      exit 1
    }
```

### Pitfall 3: Secrets in Fork Pull Requests

**Problem**: Secrets not available to pull requests from forks (security feature)

**Solution**:
```yaml
# Use pull_request_target with caution for fork PRs
on:
  pull_request_target:  # Has access to secrets, but dangerous!
    types: [labeled]

jobs:
  secure-deploy:
    if: github.event.label.name == 'safe-to-deploy'  # Manual approval
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}  # PR code
      
      - name: Deploy with secrets
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          # Only runs after manual label approval
          deploy.sh
```

## Critical Notes

🎯 **Automatic Masking**: Secrets are automatically masked in logs, but never intentionally echo them—defense in depth.

💡 **Immutable Values**: Secret values can't be viewed after creation—store them securely outside GitHub as backup.

⚠️ **Fork Restrictions**: Pull requests from forks don't have access to secrets—this is a security feature, not a bug.

📊 **Scope Hierarchy**: Environment secrets override organization secrets, which override repository secrets—use this for flexibility.

🔄 **Regular Rotation**: Rotate secrets quarterly or when team members leave—automate reminders.

✨ **JSON Secrets**: Store related credentials as JSON for cleaner secret management—parse with `jq` in workflows.

## Quick Reference

### Creating Secrets

```bash
# Repository secret (GitHub CLI)
gh secret set SECRET_NAME

# Organization secret
gh secret set SECRET_NAME --org myorg

# Environment secret (via UI)
Settings → Environments → [env name] → Add secret
```

### Using Secrets in Workflows

```yaml
# As environment variable
env:
  API_KEY: ${{ secrets.API_KEY }}

# Direct in step
run: command --token=${{ secrets.TOKEN }}

# With action input
with:
  token: ${{ secrets.GITHUB_TOKEN }}

# In environment-specific job
environment: production
steps:
  - run: echo ${{ secrets.PROD_SECRET }}
```

### Secret Scopes

| Scope | Availability | Access Control | Use Case |
|-------|--------------|----------------|----------|
| **Repository** | Single repo | Admins/maintainers | Project-specific credentials |
| **Organization** | Multiple repos | Org owners | Shared infrastructure |
| **Environment** | Specific environment | Environment rules | Per-environment configs |

[Learn More](https://learn.microsoft.com/en-us/training/modules/learn-continuous-integration-github-actions/8-create-encrypted-secrets)
