# Use Secrets in Workflow

GitHub Actions secrets enable secure access to sensitive information within workflows. Understanding how to properly access, validate, and use secrets is essential for building secure CI/CD pipelines while maintaining operational efficiency.

## Accessing Secrets in Workflows

### Basic Secret Access Patterns

**1. As Environment Variables** (Recommended):

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy application
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          # Secrets available as environment variables
          echo "Deploying with API token (masked in logs)"
          ./deploy.sh
          # Script can access $API_TOKEN and $DATABASE_URL
```

**Why Environment Variables?**
- ✅ Secrets are automatically masked in logs
- ✅ Standard across different shells (bash, PowerShell, etc.)
- ✅ Easier to use in scripts and applications
- ✅ Follows 12-factor app principles

**2. Direct in Commands**:

```yaml
- name: Call API
  run: |
    curl -X POST https://api.example.com/deploy \
      -H "Authorization: Bearer ${{ secrets.API_TOKEN }}" \
      -d '{"version": "1.0.0"}'
  # Logs show: Authorization: Bearer ***
```

**3. As Action Inputs**:

```yaml
- name: Azure Login
  uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}

- name: Deploy to Heroku
  uses: akhileshns/heroku-deploy@v3.12.12
  with:
    heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
    heroku_app_name: "my-app"
    heroku_email: "deploy@example.com"
```

### Cross-Shell Secret Usage

Secrets work consistently across different runners:

```yaml
# Linux/macOS (bash)
- name: Use secret in bash
  shell: bash
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    echo "API Key configured: ${API_KEY:0:4}..."  # Show first 4 chars only
    ./script.sh

# Windows (PowerShell)
- name: Use secret in PowerShell
  shell: pwsh
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    Write-Host "API Key configured"
    .\script.ps1

# Python
- name: Use secret in Python
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    python << EOF
    import os
    api_key = os.environ.get('API_KEY')
    print(f"API Key length: {len(api_key)}")
    EOF
```

## Advanced Secret Usage Patterns

### Pattern 1: Conditional Secret Usage

Use secrets only when needed:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        if: github.ref == 'refs/heads/develop'
        env:
          STAGING_TOKEN: ${{ secrets.STAGING_API_TOKEN }}
        run: |
          echo "Deploying to staging"
          ./deploy.sh staging
      
      - name: Deploy to production
        if: github.ref == 'refs/heads/main'
        env:
          PROD_TOKEN: ${{ secrets.PRODUCTION_API_TOKEN }}
        run: |
          echo "Deploying to production"
          ./deploy.sh production
```

### Pattern 2: Secret Validation and Health Checks

Validate secrets before using them:

```yaml
jobs:
  validate-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Validate required secrets
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
          DB_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}
          AWS_KEY: ${{ secrets.AWS_ACCESS_KEY_ID }}
        run: |
          # Check all required secrets are set
          missing_secrets=()
          
          [ -z "$API_TOKEN" ] && missing_secrets+=("API_TOKEN")
          [ -z "$DB_PASSWORD" ] && missing_secrets+=("DATABASE_PASSWORD")
          [ -z "$AWS_KEY" ] && missing_secrets+=("AWS_ACCESS_KEY_ID")
          
          if [ ${#missing_secrets[@]} -gt 0 ]; then
            echo "❌ Missing required secrets: ${missing_secrets[*]}"
            exit 1
          fi
          
          echo "✅ All required secrets are configured"
      
      - name: Test API credentials
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
          API_ENDPOINT: ${{ secrets.API_ENDPOINT }}
        run: |
          # Health check with credentials
          HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" \
            -H "Authorization: Bearer $API_TOKEN" \
            "$API_ENDPOINT/health")
          
          if [ "$HTTP_CODE" = "200" ]; then
            echo "✅ API credentials valid"
          else
            echo "❌ API credentials invalid (HTTP $HTTP_CODE)"
            exit 1
          fi
      
      - name: Deploy application
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
        run: |
          # Only runs if validation passed
          ./deploy.sh
```

### Pattern 3: Secret Composition and Transformation

Combine multiple secrets into complex configurations:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Create database connection string
        env:
          DB_HOST: ${{ secrets.DATABASE_HOST }}
          DB_PORT: ${{ secrets.DATABASE_PORT }}
          DB_NAME: ${{ secrets.DATABASE_NAME }}
          DB_USER: ${{ secrets.DATABASE_USER }}
          DB_PASS: ${{ secrets.DATABASE_PASSWORD }}
        run: |
          # Compose connection string from individual secrets
          CONNECTION_STRING="postgresql://${DB_USER}:${DB_PASS}@${DB_HOST}:${DB_PORT}/${DB_NAME}"
          
          # Make available to subsequent steps
          echo "DATABASE_URL=$CONNECTION_STRING" >> $GITHUB_ENV
          
          echo "✅ Database connection configured"
      
      - name: Deploy with composed config
        run: |
          # DATABASE_URL is now available (and masked in logs)
          ./deploy.sh --database-url="$DATABASE_URL"
```

### Pattern 4: Multi-Environment Secret Selection

Automatically select secrets based on environment:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [staging, production]
    environment: ${{ matrix.environment }}
    
    steps:
      - name: Deploy to environment
        env:
          API_KEY: ${{ secrets.API_KEY }}           # Environment-specific
          DB_URL: ${{ secrets.DATABASE_URL }}       # Environment-specific
          DEPLOY_URL: ${{ secrets.DEPLOYMENT_URL }} # Environment-specific
        run: |
          echo "Deploying to ${{ matrix.environment }}"
          echo "API Endpoint: $DEPLOY_URL"
          ./deploy.sh
          # Each environment uses its own secrets automatically
```

### Pattern 5: Parsing JSON Secrets

Work with complex JSON-formatted secrets:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Parse Azure credentials
        env:
          AZURE_CREDS: ${{ secrets.AZURE_CREDENTIALS }}
        run: |
          # Parse JSON secret
          CLIENT_ID=$(echo $AZURE_CREDS | jq -r '.clientId')
          TENANT_ID=$(echo $AZURE_CREDS | jq -r '.tenantId')
          SUBSCRIPTION_ID=$(echo $AZURE_CREDS | jq -r '.subscriptionId')
          
          # Make individual values available
          echo "AZURE_CLIENT_ID=$CLIENT_ID" >> $GITHUB_ENV
          echo "AZURE_TENANT_ID=$TENANT_ID" >> $GITHUB_ENV
          echo "AZURE_SUBSCRIPTION_ID=$SUBSCRIPTION_ID" >> $GITHUB_ENV
          
          echo "✅ Azure credentials parsed"
      
      - name: Use parsed credentials
        run: |
          echo "Subscription: $AZURE_SUBSCRIPTION_ID"
          az login --service-principal \
            --username "$AZURE_CLIENT_ID" \
            --tenant "$AZURE_TENANT_ID"
```

## Working with Conditional Logic

### Branch-Based Secret Selection

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Set deployment secrets
        id: secrets
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "env=production" >> $GITHUB_OUTPUT
          elif [[ "${{ github.ref }}" == "refs/heads/develop" ]]; then
            echo "env=staging" >> $GITHUB_OUTPUT
          else
            echo "env=development" >> $GITHUB_OUTPUT
          fi
      
      - name: Deploy to production
        if: steps.secrets.outputs.env == 'production'
        env:
          API_KEY: ${{ secrets.PRODUCTION_API_KEY }}
          DB_URL: ${{ secrets.PRODUCTION_DATABASE_URL }}
        run: |
          echo "Deploying to production"
          ./deploy.sh production
      
      - name: Deploy to staging
        if: steps.secrets.outputs.env == 'staging'
        env:
          API_KEY: ${{ secrets.STAGING_API_KEY }}
          DB_URL: ${{ secrets.STAGING_DATABASE_URL }}
        run: |
          echo "Deploying to staging"
          ./deploy.sh staging
```

### Using Secrets in Workflow Conditions

```yaml
jobs:
  deploy:
    # Only run if production secret is configured
    if: secrets.PRODUCTION_API_KEY != ''
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          API_KEY: ${{ secrets.PRODUCTION_API_KEY }}
        run: ./deploy.sh

  notify:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Send Slack notification
        # Only notify if webhook secret exists
        if: secrets.SLACK_WEBHOOK_URL != ''
        env:
          WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
        run: |
          curl -X POST $WEBHOOK_URL \
            -H 'Content-Type: application/json' \
            -d '{"text":"Deployment completed successfully"}'
```

### Multiple Condition Secret Handling

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy with optional features
        env:
          BASE_TOKEN: ${{ secrets.API_TOKEN }}
          SLACK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SENTRY_DSN: ${{ secrets.SENTRY_DSN }}
        run: |
          # Required secret (must exist)
          if [ -z "$BASE_TOKEN" ]; then
            echo "❌ API_TOKEN is required"
            exit 1
          fi
          
          # Deploy application
          ./deploy.sh --token="$BASE_TOKEN"
          
          # Optional: Send Slack notification
          if [ -n "$SLACK_URL" ]; then
            echo "📢 Sending Slack notification"
            curl -X POST "$SLACK_URL" -d '{"text":"Deployed!"}'
          fi
          
          # Optional: Configure Sentry
          if [ -n "$SENTRY_DSN" ]; then
            echo "🐛 Configuring Sentry monitoring"
            ./configure-sentry.sh "$SENTRY_DSN"
          fi
          
          echo "✅ Deployment complete"
```

## Security Best Practices When Using Secrets

### 1. Minimize Exposure Scope

```yaml
# ❌ Bad: Secret exposed to entire job
jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      SECRET_KEY: ${{ secrets.SECRET_KEY }}  # Available to all steps
    steps:
      - run: echo "Step 1"
      - run: echo "Step 2"
      - run: deploy.sh  # Only this step needs the secret

# ✅ Good: Secret exposed only where needed
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Step 1"
      - run: echo "Step 2"
      - name: Deploy with secret
        env:
          SECRET_KEY: ${{ secrets.SECRET_KEY }}  # Only this step
        run: deploy.sh
```

### 2. Safe Error Handling

```yaml
# ❌ Bad: Error might expose secret
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    result=$(api-call --key="$API_KEY") || {
      echo "Error details: $result"  # Might contain secret!
      exit 1
    }

# ✅ Good: Sanitized error handling
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    if ! api-call --key="$API_KEY" > /tmp/result 2>&1; then
      echo "❌ API call failed (details hidden for security)"
      echo "Check workflow logs for more information"
      exit 1
    fi
    
    echo "✅ API call successful"
```

### 3. Audit Secret Usage

```yaml
# Log when secrets are accessed (without exposing values)
- name: Audit secret access
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    echo "::notice::API_KEY accessed by ${{ github.actor }} in workflow ${{ github.workflow }}"
    echo "::notice::Repository: ${{ github.repository }}"
    echo "::notice::Ref: ${{ github.ref }}"
    
    # Use the secret
    ./deploy.sh
```

### 4. Rotate Secrets After Exposure

```yaml
# Detect potential secret exposure
- name: Check for exposed secrets
  run: |
    # Scan code for hardcoded patterns
    if grep -r "sk_live_" .; then
      echo "::error::Potential API key found in code!"
      echo "::error::Rotate secrets immediately if confirmed"
      exit 1
    fi
```

## Understanding Secret Limitations

### Secret Size Limitations

**Limits**:
- **Single secret**: 64 KB maximum
- **Organization secrets**: 100 secrets per organization
- **Repository secrets**: 100 secrets per repository
- **Environment secrets**: 100 secrets per environment

**Workaround for Large Secrets**:

```yaml
# Option 1: Store large secrets externally
- name: Fetch large configuration
  env:
    S3_ACCESS_KEY: ${{ secrets.AWS_ACCESS_KEY_ID }}
    S3_SECRET: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  run: |
    # Download large config from S3
    aws s3 cp s3://my-bucket/large-config.json ./config.json
    # Use config.json in deployment

# Option 2: Split large secrets
- name: Reconstruct large certificate
  env:
    CERT_PART1: ${{ secrets.CERTIFICATE_PART1 }}
    CERT_PART2: ${{ secrets.CERTIFICATE_PART2 }}
    CERT_PART3: ${{ secrets.CERTIFICATE_PART3 }}
  run: |
    echo "$CERT_PART1$CERT_PART2$CERT_PART3" > certificate.pem
    # Use certificate.pem
```

### Fork Repository Limitations

**Behavior**: Pull requests from forks **DO NOT** have access to secrets (security feature)

**Problem**:
```yaml
# This workflow WON'T work for fork PRs
on: pull_request

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests with API
        env:
          API_KEY: ${{ secrets.API_KEY }}  # Not available from forks!
        run: npm test
```

**Solutions**:

**Option 1: Use `pull_request_target` with Caution**:

```yaml
# DANGEROUS: Only use with manual approval
on:
  pull_request_target:  # Has secret access, but runs in base repo context

jobs:
  test-with-secrets:
    if: github.event.pull_request.head.repo.full_name == github.repository
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}  # PR code
      
      - name: Run tests
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: npm test
```

**Option 2: Manual Approval with Labels**:

```yaml
on:
  pull_request_target:
    types: [labeled]

jobs:
  test-approved:
    # Only run after manual review and label
    if: github.event.label.name == 'safe-to-test'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}
      
      - name: Run tests with secrets
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: npm test
```

**Option 3: Separate Workflows**:

```yaml
# .github/workflows/test-no-secrets.yml
# Runs on all PRs (including forks)
on: pull_request

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test  # Tests that don't need secrets

# .github/workflows/test-with-secrets.yml
# Only runs for trusted PRs
on:
  pull_request:
    branches: [main]

jobs:
  test-with-secrets:
    if: github.event.pull_request.head.repo.full_name == github.repository
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Integration tests
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: npm run test:integration
```

### Environment-Specific Restrictions

```yaml
# Environment secrets require environment specification
jobs:
  deploy:
    # ❌ This won't work - environment not specified
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          SECRET: ${{ secrets.PRODUCTION_SECRET }}  # Not available!
        run: ./deploy.sh

  deploy-correct:
    # ✅ Correct - environment specified
    runs-on: ubuntu-latest
    environment: production  # Now PRODUCTION_SECRET is available
    steps:
      - name: Deploy
        env:
          SECRET: ${{ secrets.PRODUCTION_SECRET }}
        run: ./deploy.sh
```

## Critical Notes

🎯 **Environment Variables**: Always use environment variables for secrets—more secure and portable than direct interpolation.

💡 **Validation First**: Validate secrets exist and are valid before attempting deployment—fail fast with clear error messages.

⚠️ **Fork Security**: Secrets not available to fork PRs is a security feature—use `pull_request_target` only with extreme caution.

📊 **Minimal Scope**: Expose secrets only to steps that need them—job-level environment variables expose to all steps.

🔄 **Error Sanitization**: Never echo error messages that might contain secret values—sanitize all output.

✨ **JSON Parsing**: Use `jq` to parse complex JSON secrets into individual environment variables for easier use.

## Quick Reference

### Secret Access Patterns

```yaml
# Environment variable (recommended)
env:
  SECRET: ${{ secrets.MY_SECRET }}

# Direct in command
run: command --token=${{ secrets.TOKEN }}

# Action input
with:
  token: ${{ secrets.GITHUB_TOKEN }}

# Conditional usage
if: secrets.MY_SECRET != ''
```

### Common Validation Pattern

```yaml
- name: Validate secrets
  env:
    SECRET1: ${{ secrets.SECRET1 }}
    SECRET2: ${{ secrets.SECRET2 }}
  run: |
    [ -z "$SECRET1" ] && echo "Missing SECRET1" && exit 1
    [ -z "$SECRET2" ] && echo "Missing SECRET2" && exit 1
    echo "✅ All secrets validated"
```

### Composition Pattern

```yaml
- name: Compose configuration
  env:
    HOST: ${{ secrets.DB_HOST }}
    PORT: ${{ secrets.DB_PORT }}
    USER: ${{ secrets.DB_USER }}
    PASS: ${{ secrets.DB_PASS }}
  run: |
    URL="postgresql://${USER}:${PASS}@${HOST}:${PORT}/db"
    echo "DATABASE_URL=$URL" >> $GITHUB_ENV
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/learn-continuous-integration-github-actions/9-use-secrets-workflow)
