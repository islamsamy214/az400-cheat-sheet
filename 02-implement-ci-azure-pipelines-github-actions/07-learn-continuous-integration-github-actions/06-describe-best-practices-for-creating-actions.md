# Describe Best Practices for Creating Actions

Creating high-quality GitHub Actions requires thoughtful design, comprehensive documentation, and attention to security and performance. Following best practices ensures your actions are reliable, maintainable, and valuable to the community.

## Design Principles for Actions

### 1. Single Responsibility Principle

Each action should do one thing exceptionally well:

**❌ Avoid Multi-Purpose Actions**:
```yaml
# Bad: action tries to do too much
name: "Deploy Everything"
description: "Builds, tests, deploys, and monitors your application"
```

**✅ Create Focused Actions**:
```yaml
# Good: focused on a single task
name: "Deploy to Azure"
description: "Deploys artifacts to Azure App Service"
```

### 2. Composable Design

Build actions that work well with others:

```yaml
# Example: Composable workflow using focused actions
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3                    # Focused: source control
      - uses: actions/setup-node@v3                  # Focused: environment setup
      - uses: myorg/build-action@v1                  # Focused: build process
      - uses: myorg/test-action@v1                   # Focused: testing
      - uses: myorg/deploy-azure-action@v1           # Focused: deployment
```

**Benefits**:
- Easier to test and maintain individual components
- Teams can mix and match actions for custom workflows
- Better reusability across different projects

### 3. Predictable Behavior

Actions should behave consistently:

```yaml
# action.yml - Clear, predictable interface
inputs:
  environment:
    description: 'Deployment environment (dev/staging/prod)'
    required: true
  version:
    description: 'Application version to deploy'
    required: true
    
outputs:
  deployment-url:
    description: 'URL where application was deployed'
  deployment-time:
    description: 'Timestamp of deployment completion'
```

## Action Metadata and Documentation

### Complete action.yml Specification

Create comprehensive action metadata:

```yaml
name: 'Deploy to Azure App Service'
description: 'Deploys web applications to Azure App Service with health checks and rollback support'
author: 'Your Organization <devops@example.com>'
branding:
  icon: 'cloud'
  color: 'blue'

inputs:
  azure-credentials:
    description: 'Azure service principal credentials (JSON format)'
    required: true
  app-name:
    description: 'Name of the Azure App Service'
    required: true
  resource-group:
    description: 'Azure resource group name'
    required: true
  slot-name:
    description: 'Deployment slot name (default: production)'
    required: false
    default: 'production'
  package-path:
    description: 'Path to deployment package or artifact'
    required: true
  startup-command:
    description: 'Custom startup command (optional)'
    required: false
  health-check-path:
    description: 'Endpoint for health check validation'
    required: false
    default: '/health'
  timeout-minutes:
    description: 'Maximum time to wait for deployment (default: 10)'
    required: false
    default: '10'

outputs:
  deployment-url:
    description: 'URL of the deployed application'
  deployment-id:
    description: 'Unique identifier for this deployment'
  deployment-status:
    description: 'Final status: success, failed, or rolled-back'
  app-version:
    description: 'Version of the deployed application'

runs:
  using: 'node20'
  main: 'dist/index.js'
```

### README Documentation

Create comprehensive README with these essential sections:

```markdown
# Deploy to Azure App Service Action

## Description

Deploys web applications to Azure App Service with built-in health checks, automatic rollback on failure, and detailed deployment reporting.

## Features

- ✅ Zero-downtime deployment using deployment slots
- ✅ Automatic health check validation
- ✅ Rollback on deployment failure
- ✅ Comprehensive logging and error reporting
- ✅ Support for multiple deployment strategies
- ✅ Multi-environment configuration support

## Usage

### Basic Example

```yaml
- name: Deploy to Azure
  uses: myorg/deploy-azure-action@v1
  with:
    azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
    app-name: 'my-web-app'
    resource-group: 'my-resource-group'
    package-path: './dist'
```

### Advanced Example with Health Checks

```yaml
- name: Deploy to Azure with validation
  uses: myorg/deploy-azure-action@v1
  with:
    azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
    app-name: 'my-web-app'
    resource-group: 'my-resource-group'
    package-path: './dist'
    slot-name: 'staging'
    health-check-path: '/api/health'
    timeout-minutes: 15
    startup-command: 'npm run start:prod'
```

### Complete Workflow Example

```yaml
name: Production Deployment

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build application
        run: npm ci && npm run build
        
      - name: Deploy to Azure
        id: deploy
        uses: myorg/deploy-azure-action@v1
        with:
          azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
          app-name: 'production-app'
          resource-group: 'production-rg'
          package-path: './dist'
          
      - name: Verify deployment
        run: |
          echo "Deployed to: ${{ steps.deploy.outputs.deployment-url }}"
          echo "Deployment ID: ${{ steps.deploy.outputs.deployment-id }}"
```

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `azure-credentials` | Yes | - | Azure service principal credentials in JSON format |
| `app-name` | Yes | - | Name of the Azure App Service |
| `resource-group` | Yes | - | Azure resource group containing the App Service |
| `slot-name` | No | `production` | Deployment slot (use `staging` for blue-green deployments) |
| `package-path` | Yes | - | Path to deployment package or build artifacts |
| `startup-command` | No | - | Custom startup command for the application |
| `health-check-path` | No | `/health` | Endpoint for health check validation |
| `timeout-minutes` | No | `10` | Maximum time to wait for deployment completion |

## Outputs

| Name | Description |
|------|-------------|
| `deployment-url` | Full URL where the application was deployed |
| `deployment-id` | Unique identifier for tracking this deployment |
| `deployment-status` | Final status: `success`, `failed`, or `rolled-back` |
| `app-version` | Version of the deployed application |

## Prerequisites

1. **Azure Service Principal**: Create credentials with Contributor role
2. **Repository Secret**: Store credentials as `AZURE_CREDENTIALS`
3. **App Service**: Existing Azure App Service instance

### Creating Azure Credentials

```bash
az ad sp create-for-rbac \
  --name "github-actions-deploy" \
  --role Contributor \
  --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
  --sdk-auth
```

## Error Handling

The action includes comprehensive error handling:

- **Authentication Failures**: Clear error messages with troubleshooting steps
- **Deployment Failures**: Automatic rollback to previous version
- **Health Check Failures**: Rollback with detailed failure logs
- **Timeout Handling**: Graceful timeout with status reporting

## Troubleshooting

### Common Issues

**Issue**: Authentication failed
```
Solution: Verify AZURE_CREDENTIALS secret is correctly formatted JSON
```

**Issue**: Health check timeout
```
Solution: Increase timeout-minutes or verify health-check-path is correct
```

**Issue**: Deployment stuck
```
Solution: Check Azure portal for App Service status and logs
```

## Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Support

- 📖 [Documentation](https://docs.example.com)
- 💬 [Discussions](https://github.com/myorg/deploy-azure-action/discussions)
- 🐛 [Issue Tracker](https://github.com/myorg/deploy-azure-action/issues)
```

## Version Management

### Semantic Versioning for Actions

Use semantic versioning (SemVer) for action releases:

```
MAJOR.MINOR.PATCH

MAJOR: Breaking changes (v1 → v2)
MINOR: New features, backward compatible (v1.0 → v1.1)
PATCH: Bug fixes, backward compatible (v1.0.0 → v1.0.1)
```

**Version Tagging Strategy**:

```bash
# Tag specific version
git tag v1.2.3
git push origin v1.2.3

# Update major version tag (recommended for users)
git tag -fa v1 -m "Update v1 tag"
git push origin v1 --force

# Update minor version tag
git tag -fa v1.2 -m "Update v1.2 tag"
git push origin v1.2 --force
```

**User Reference Pattern**:

```yaml
# Users can reference different version levels
- uses: myorg/my-action@v1       # Always gets latest v1.x.x (recommended)
- uses: myorg/my-action@v1.2     # Always gets latest v1.2.x
- uses: myorg/my-action@v1.2.3   # Pinned to specific version
```

### Release Automation

Automate version management with GitHub Actions:

```yaml
# .github/workflows/release.yml
name: Release Action

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build action
        run: |
          npm ci
          npm run build
          npm run package
      
      - name: Get version info
        id: version
        run: |
          echo "version=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT
          echo "major=${GITHUB_REF#refs/tags/v}" | cut -d. -f1 >> $GITHUB_OUTPUT
          
      - name: Update major version tag
        run: |
          git config user.name github-actions
          git config user.email github-actions@github.com
          git tag -fa v${{ steps.version.outputs.major }} -m "Update v${{ steps.version.outputs.major }} tag"
          git push origin v${{ steps.version.outputs.major }} --force
          
      - name: Create GitHub Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ steps.version.outputs.version }}
          release_name: Release ${{ steps.version.outputs.version }}
          body_path: CHANGELOG.md
          draft: false
          prerelease: false
```

### Breaking Change Communication

Clearly document breaking changes:

```markdown
# CHANGELOG.md

## [2.0.0] - 2024-01-15

### ⚠️ BREAKING CHANGES

- **Input Renamed**: `app-name` is now `azure-app-name` for clarity
- **Output Changed**: `url` is now `deployment-url` with full path
- **Minimum Version**: Requires Node.js 20+ (was Node.js 16)

### Migration Guide

**Before (v1.x)**:
```yaml
- uses: myorg/action@v1
  with:
    app-name: my-app
```

**After (v2.x)**:
```yaml
- uses: myorg/action@v2
  with:
    azure-app-name: my-app  # Renamed input
```

## [1.5.2] - 2024-01-10

### Fixed
- Health check timeout handling
- Error message clarity for auth failures
```

## Security Best Practices

### 1. Input Validation

Always validate inputs before using them:

```javascript
// src/index.js
const core = require('@actions/core');

function validateInputs() {
  const appName = core.getInput('app-name', { required: true });
  const resourceGroup = core.getInput('resource-group', { required: true });
  const environment = core.getInput('environment', { required: true });
  
  // Validate environment value
  const validEnvironments = ['dev', 'staging', 'production'];
  if (!validEnvironments.includes(environment)) {
    throw new Error(
      `Invalid environment: ${environment}. ` +
      `Must be one of: ${validEnvironments.join(', ')}`
    );
  }
  
  // Validate naming patterns
  if (!/^[a-z0-9-]+$/.test(appName)) {
    throw new Error(
      `Invalid app-name: ${appName}. ` +
      `Must contain only lowercase letters, numbers, and hyphens.`
    );
  }
  
  return { appName, resourceGroup, environment };
}
```

### 2. Secure Secret Handling

Never expose secrets in logs or outputs:

```javascript
// ❌ BAD: Exposes secrets
console.log(`Deploying with credentials: ${credentials}`);
core.setOutput('api-key', apiKey);

// ✅ GOOD: Secrets are masked
core.setSecret(credentials);  // Masks in logs
console.log('Credentials configured successfully');

// Store secrets securely
core.setOutput('deployment-id', deploymentId);  // Safe: not a secret
// Never output: API keys, passwords, tokens
```

### 3. Least Privilege Principle

Request only necessary permissions:

```yaml
# action.yml
name: 'My Action'
# ...

# Document required permissions
# Required Permissions:
#   - contents: read (for checking out code)
#   - packages: write (for publishing packages)

# Example workflow usage:
# permissions:
#   contents: read
#   packages: write
```

### 4. Dependency Security

Regularly audit and update dependencies:

```json
{
  "scripts": {
    "audit": "npm audit",
    "audit-fix": "npm audit fix",
    "outdated": "npm outdated"
  },
  "devDependencies": {
    "@vercel/ncc": "^0.38.0",
    "eslint": "^8.50.0"
  }
}
```

Set up automated security scanning:

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * 0'  # Weekly

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run npm audit
        run: npm audit --audit-level=moderate
        
      - name: Run CodeQL Analysis
        uses: github/codeql-action/analyze@v2
```

## Performance Guidelines

### 1. Efficient Resource Usage

Minimize action execution time:

```javascript
// ✅ Use efficient approaches
const core = require('@actions/core');
const github = require('@actions/github');

async function run() {
  try {
    const startTime = Date.now();
    
    // Process in parallel when possible
    const [result1, result2, result3] = await Promise.all([
      operation1(),
      operation2(),
      operation3()
    ]);
    
    // Cache expensive operations
    const cached = getCachedResult();
    if (cached) {
      return cached;
    }
    
    const duration = Date.now() - startTime;
    core.info(`Action completed in ${duration}ms`);
    
  } catch (error) {
    core.setFailed(error.message);
  }
}
```

### 2. Container Action Optimization

For Docker-based actions, optimize container size:

```dockerfile
# Use specific, minimal base images
FROM node:20-alpine AS builder

# Install only production dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Multi-stage build to reduce final image size
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .

ENTRYPOINT ["node", "/app/index.js"]
```

### 3. Caching Strategies

Leverage caching for better performance:

```yaml
# Example workflow using your action with caching
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    
- name: Use your action
  uses: myorg/my-action@v1
  with:
    # Action benefits from cached dependencies
    enable-cache: true
```

## Testing and Quality Assurance

### Comprehensive Test Coverage

```javascript
// __tests__/action.test.js
const core = require('@actions/core');
const { run } = require('../src/index');

describe('Deploy Action', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  test('successful deployment', async () => {
    // Mock inputs
    jest.spyOn(core, 'getInput').mockImplementation((name) => {
      const inputs = {
        'app-name': 'test-app',
        'resource-group': 'test-rg',
        'package-path': './dist'
      };
      return inputs[name];
    });
    
    // Run action
    await run();
    
    // Verify outputs
    expect(core.setOutput).toHaveBeenCalledWith('deployment-status', 'success');
  });
  
  test('handles invalid inputs', async () => {
    // Test error handling
    jest.spyOn(core, 'getInput').mockImplementation(() => 'invalid!@#');
    
    await run();
    
    expect(core.setFailed).toHaveBeenCalled();
  });
});
```

### Integration Testing Workflow

```yaml
# .github/workflows/test.yml
name: Test Action

on: [push, pull_request]

jobs:
  test-unit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test
      - run: npm run lint
      
  test-integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # Test the action itself
      - name: Test action
        uses: ./
        with:
          app-name: 'test-app'
          resource-group: 'test-rg'
          package-path: './test-fixtures'
```

## Marketplace Guidelines

### Publishing to GitHub Marketplace

**Checklist before publishing**:

- ✅ Comprehensive README with usage examples
- ✅ Complete action.yml with all metadata
- ✅ LICENSE file (MIT, Apache 2.0, etc.)
- ✅ Semantic versioning tags
- ✅ Icon and color branding
- ✅ Tested with sample workflows
- ✅ Security audit passed
- ✅ No hardcoded secrets or tokens

**action.yml metadata for Marketplace**:

```yaml
name: 'Deploy to Azure App Service'
description: 'Deploy web applications to Azure with health checks and rollback'
author: 'Your Organization'

branding:
  icon: 'cloud'
  color: 'blue'

# Marketplace will use this info for search and display
```

### Marketplace Best Practices

1. **Clear Action Name**: Use descriptive, searchable names
2. **Detailed Description**: Explain what problem your action solves
3. **Usage Examples**: Provide multiple examples for different scenarios
4. **Version Strategy**: Keep v1, v1.2, v1.2.3 tags updated
5. **Responsive Maintenance**: Respond to issues and pull requests promptly
6. **Changelog**: Maintain clear release notes

## Critical Notes

🎯 **Single Responsibility**: Each action should do one thing well—avoid creating Swiss Army knife actions.

💡 **Version Tags**: Maintain major version tags (v1, v2) that always point to latest compatible version—users prefer this.

⚠️ **Breaking Changes**: Use major version bumps for breaking changes and provide migration guides.

📊 **Input Validation**: Always validate inputs to prevent errors and security issues—fail fast with clear messages.

🔄 **Documentation**: Comprehensive README is essential—include usage examples, troubleshooting, and contribution guidelines.

✨ **Testing**: Test your actions thoroughly before publishing—include unit tests and integration tests in CI/CD.

## Quick Reference

### Action Development Checklist

**Planning**:
- [ ] Single, clear purpose defined
- [ ] Inputs and outputs designed
- [ ] Security requirements identified

**Implementation**:
- [ ] Complete action.yml metadata
- [ ] Input validation implemented
- [ ] Error handling comprehensive
- [ ] Secrets properly masked

**Documentation**:
- [ ] README with usage examples
- [ ] CHANGELOG maintained
- [ ] Inline code comments
- [ ] Troubleshooting section

**Quality**:
- [ ] Unit tests written
- [ ] Integration tests pass
- [ ] Security audit completed
- [ ] Performance optimized

**Release**:
- [ ] Semantic versioning applied
- [ ] Tags created (v1, v1.0, v1.0.0)
- [ ] GitHub Release created
- [ ] Marketplace listing updated

[Learn More](https://learn.microsoft.com/en-us/training/modules/learn-continuous-integration-github-actions/6-describe-best-practices-creating-actions)
