# Integrate GitHub with Single Sign-On (SSO)

**Duration**: 5 minutes

Learn how to implement GitHub authentication strategies including SSH keys, Personal Access Tokens, OAuth apps, GitHub Apps, and SAML SSO for enterprise identity integration.

---

## Overview

GitHub supports **6 primary authentication methods** for human users and automated workloads, each optimized for specific security, automation, and integration scenarios. Enterprise organizations can enforce **SAML Single Sign-On (SSO)** for centralized identity management and multifactor authentication policies.

---

## GitHub Authentication Methods

### 1. Username + Password (with Two-Factor Authentication)

**Primary Method**: Traditional username/password authentication enhanced with mandatory two-factor authentication for code contributors.

#### Characteristics
- **Security Level**: Low (password alone) → High (with 2FA)
- **Use Case**: Human interactive sessions (web UI, CLI)
- **Automation**: Not recommended (credentials in scripts)
- **MFA Requirement**: Mandatory for organizations with security policies

#### Two-Factor Authentication Mechanisms

**TOTP (Time-Based One-Time Password)**:
```
1. User: Install authenticator app (Microsoft Authenticator, Google Authenticator, Authy)
2. GitHub: Generate QR code during 2FA setup
3. User: Scan QR code, link account
4. Authentication: Username + Password + 6-digit TOTP code
5. Validity: 30-second rotation window
```

**SMS Tokens**:
```
1. User: Register mobile phone number
2. GitHub: Send SMS with 6-digit code
3. Authentication: Username + Password + SMS code
4. Validity: Single-use, time-limited
```

**Security Keys (FIDO U2F/WebAuthn)**:
```
1. User: Register hardware security key (YubiKey, Titan Security Key)
2. Authentication: Username + Password + Physical key tap
3. Security: Phishing-resistant, strongest 2FA method
```

#### Best Practices
- ✅ **Enable 2FA immediately** for all accounts with write access
- ✅ **Use TOTP or security keys** (more secure than SMS)
- ✅ **Generate recovery codes** during 2FA setup (store securely)
- ❌ **Avoid password-only authentication** (deprecated for contributors)
- ❌ **Don't share credentials** across team members

#### When to Use
- **Personal accounts**: Primary authentication for web UI
- **Learning/testing**: Simple setup for individual developers
- **Interactive workflows**: Web-based repository management

---

### 2. SSH Keys (Asymmetric Cryptography)

**Technical Method**: Public-key cryptography for Git protocol authentication without password transmission.

#### SSH Key Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Developer Workstation                                       │
│  ┌────────────────────────┐    ┌────────────────────────┐  │
│  │  Private Key           │    │  Public Key            │  │
│  │  ~/.ssh/id_ed25519     │    │  ~/.ssh/id_ed25519.pub │  │
│  │  (NEVER share)         │    │  (upload to GitHub)    │  │
│  └──────────┬─────────────┘    └────────┬───────────────┘  │
│             │                            │                   │
│             │  1. Sign challenge         │  2. Upload        │
│             └────────────────────────────┼───────────────────┤
└──────────────────────────────────────────┼───────────────────┘
                                           │
                                           ▼
                              ┌────────────────────────────┐
                              │  GitHub                     │
                              │  Stored Public Keys         │
                              │                            │
                              │  3. Verify signature       │
                              │     with public key        │
                              │                            │
                              │  ✅ Authentication Success │
                              └────────────────────────────┘
```

#### SSH Key Generation

**Ed25519 Algorithm** (Recommended):
```bash
# Generate Ed25519 key pair (modern, secure)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Output:
# Generating public/private ed25519 key pair.
# Enter file in which to save the key (/home/user/.ssh/id_ed25519):
# Enter passphrase (empty for no passphrase):
# Your identification has been saved in /home/user/.ssh/id_ed25519
# Your public key has been saved in /home/user/.ssh/id_ed25519.pub
```

**RSA Algorithm** (Legacy compatibility):
```bash
# Generate RSA key pair (4096-bit for security)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**Key Passphrase** (Optional but recommended):
```
# Passphrase adds encryption layer to private key
# Even if private key is stolen, passphrase required for use
# Use ssh-agent to cache passphrase for convenience
```

#### SSH Key Registration

**Add Public Key to GitHub**:
```bash
# 1. Display public key
cat ~/.ssh/id_ed25519.pub

# Output example:
# ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOMqqnkVzrm0SdG6UOoqKLsabgH5C9okWi0dh2l9GKJl your_email@example.com

# 2. Copy output to clipboard

# 3. GitHub UI:
#    Settings → SSH and GPG keys → New SSH key
#    Title: "Work Laptop - Linux"
#    Key: <paste public key>
#    Add SSH key

# 4. Verify connection
ssh -T git@github.com

# Expected output:
# Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

#### GitHub CLI Automation

**Using GitHub CLI** (gh):
```bash
# Install GitHub CLI
# Ubuntu/Debian
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh

# Authenticate with GitHub
gh auth login

# Select authentication method:
# ? What account do you want to log into? GitHub.com
# ? What is your preferred protocol for Git operations? SSH
# ? Upload your SSH public key to your GitHub account? ~/.ssh/id_ed25519.pub
# ? How would you like to authenticate GitHub CLI? Login with a web browser

# GitHub CLI automatically:
# - Uploads your public key
# - Configures Git to use SSH
# - Stores authentication token
```

#### Best Practices
- ✅ **Use Ed25519** for new key generation (faster, more secure than RSA)
- ✅ **Protect private keys** with passphrases
- ✅ **Use ssh-agent** to cache passphrase (convenience without security loss)
- ✅ **Separate keys per device** (revoke compromised keys individually)
- ✅ **Delete unused SSH keys** from GitHub settings (retired laptops)
- ❌ **Never commit private keys** to repositories (add ~/.ssh/* to .gitignore)
- ❌ **Don't share private keys** across machines (generate per-device)

#### When to Use
- **Developer workstations**: Primary method for Git operations (clone, push, pull)
- **Git protocol**: Required for SSH-based repository URLs (git@github.com:user/repo.git)
- **Automation**: CI/CD systems with dedicated SSH keys (deploy keys)

---

### 3. Personal Access Tokens (PATs)

**Bearer Token Method**: Scoped API authentication tokens replacing passwords for programmatic access and automation workflows.

#### PAT Characteristics
- **Authentication Type**: Bearer token (HTTP Authorization header)
- **Security**: Scoped permissions (read/write repo, admin:org, workflow)
- **Expiration**: Configurable lifetime (7/30/60/90 days, custom, no expiration)
- **Revocation**: Instant invalidation via GitHub settings
- **Audit Trail**: Token usage logged in security audit log

#### PAT Scopes (Permission Granularity)

**Repository Scopes**:
- `repo` - Full control (public + private repos)
- `repo:status` - Access commit status
- `repo_deployment` - Access deployment status
- `public_repo` - Access public repositories only
- `repo:invite` - Access repository invitations

**Workflow Scopes**:
- `workflow` - Update GitHub Actions workflows
- `write:packages` - Upload packages to GitHub Packages
- `read:packages` - Download packages

**Organization Scopes**:
- `admin:org` - Full organization management
- `write:org` - Read/write organization membership
- `read:org` - Read organization membership

**Security Scopes**:
- `security_events` - Read/write security events
- `admin:repo_hook` - Repository webhook management

#### PAT Generation Workflow

**Classic Personal Access Token**:
```
1. GitHub Web UI Navigation:
   Settings → Developer settings → Personal access tokens → Tokens (classic)

2. Generate New Token:
   - Token name: "CI/CD Pipeline - Azure Pipelines"
   - Expiration: 90 days (recommended for automation)
   - Scopes: ✅ repo, ✅ workflow, ✅ write:packages

3. Generate Token:
   - Click "Generate token"
   - ⚠️ Copy immediately (displayed once only)
   - Store securely (password manager, Azure Key Vault)

4. Token Format:
   ghp_1234567890abcdefghijklmnopqrstuvwxyz123
   │   └─────────────────────────────────┘
   │            Token payload
   └─ Prefix: ghp_ (Personal Access Token identifier)
```

**Fine-Grained Personal Access Token** (Beta):
```
1. Enhanced Granularity:
   - Repository-specific permissions (not all repos)
   - Resource-level scoping (Issues, Pull Requests, Actions)
   - Organization-approved token requirement (admin control)

2. Token Configuration:
   - Resource owner: Personal account or organization
   - Repository access: "Only select repositories" → [repo1, repo2]
   - Permissions:
     * Issues: Read and write
     * Pull requests: Read and write
     * Actions: Read and write
     * Contents: Read-only

3. Organization Approval Workflow:
   - Request token with organization resources
   - Organization admin reviews/approves
   - Token activated post-approval
```

#### PAT Usage Examples

**Git HTTPS Authentication**:
```bash
# Clone repository with PAT
git clone https://ghp_TOKEN@github.com/username/repository.git

# Configure credential caching (avoid repeated entry)
git config --global credential.helper cache

# First push prompts for credentials:
# Username: your_github_username
# Password: ghp_YOUR_PERSONAL_ACCESS_TOKEN

# Subsequent operations use cached token (15 minutes default)
```

**GitHub API Automation**:
```bash
# Create repository via API
curl -X POST \
  -H "Authorization: Bearer ghp_YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/user/repos \
  -d '{"name":"new-repo", "private":true}'

# List organization repositories
curl -H "Authorization: Bearer ghp_YOUR_TOKEN" \
  https://api.github.com/orgs/ORG_NAME/repos

# Create issue
curl -X POST \
  -H "Authorization: Bearer ghp_YOUR_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/issues \
  -d '{"title":"Bug report", "body":"Description"}'
```

**Azure Pipelines Integration**:
```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: Bash@3
  inputs:
    targetType: 'inline'
    script: |
      # Use PAT from pipeline secret variable
      git clone https://$(GITHUB_PAT)@github.com/org/private-repo.git
      cd private-repo
      # Perform operations
  env:
    GITHUB_PAT: $(GitHubPersonalAccessToken)  # Secret variable

# Pipeline Variables (Azure DevOps UI):
# Variable name: GitHubPersonalAccessToken
# Value: ghp_YOUR_TOKEN
# Keep this value secret: ✅
```

#### Security Best Practices
- ✅ **Minimum scope principle**: Only grant required permissions
- ✅ **Short expiration**: 30-90 days for automation, 7 days for temporary access
- ✅ **Secure storage**: Azure Key Vault, GitHub Secrets, password managers
- ✅ **Regular rotation**: Refresh tokens before expiration
- ✅ **Audit token usage**: Review security log for suspicious activity
- ✅ **Revoke compromised tokens immediately**: Settings → Developer settings → Revoke
- ❌ **Never commit tokens** to repositories (scan with git-secrets)
- ❌ **Don't share tokens** across teams (create per-service tokens)
- ❌ **Avoid no-expiration tokens** (perpetual security risk)

#### When to Use
- **CI/CD pipelines**: Azure Pipelines, GitHub Actions, Jenkins
- **API automation**: Repository management, issue tracking, deployment workflows
- **HTTPS Git operations**: Clone private repos without SSH key setup
- **Third-party integrations**: Tools without OAuth support

---

### 4. OAuth Apps

**Delegated Access Method**: Third-party application authentication with user permission delegation, enabling integrations without credential exposure.

#### OAuth 2.0 Authorization Flow

```
┌──────────────────────────────────────────────────────────────┐
│  1. User Action: "Connect GitHub Account"                    │
└──────────────────┬───────────────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────────────┐
│  2. Redirect to GitHub Authorization:                         │
│     https://github.com/login/oauth/authorize?                │
│       client_id=YOUR_CLIENT_ID                               │
│       &redirect_uri=https://yourapp.com/callback             │
│       &scope=repo,user:email                                 │
└──────────────────┬───────────────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────────────┐
│  3. User Reviews Permissions:                                 │
│     "YourApp wants to:"                                       │
│     ✅ Access private repositories                           │
│     ✅ Read your email address                               │
│     [Authorize YourApp] [Cancel]                             │
└──────────────────┬───────────────────────────────────────────┘
                   │ (User clicks Authorize)
                   ▼
┌──────────────────────────────────────────────────────────────┐
│  4. GitHub Redirects with Authorization Code:                 │
│     https://yourapp.com/callback?code=AUTHORIZATION_CODE     │
└──────────────────┬───────────────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────────────┐
│  5. YourApp Exchanges Code for Access Token:                  │
│     POST https://github.com/login/oauth/access_token         │
│     client_id=YOUR_CLIENT_ID                                 │
│     client_secret=YOUR_CLIENT_SECRET                         │
│     code=AUTHORIZATION_CODE                                  │
└──────────────────┬───────────────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────────────┐
│  6. GitHub Returns Access Token:                              │
│     { "access_token": "gho_16C7e42F292c6912E7710c838347Ae178B4a" } │
└──────────────────┬───────────────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────────────┐
│  7. YourApp Makes API Requests:                               │
│     GET https://api.github.com/user/repos                    │
│     Authorization: Bearer gho_16C7e42F292c6912E7710c838347Ae178B4a │
└──────────────────────────────────────────────────────────────┘
```

#### OAuth App Scopes

**Available Scopes** (User grants permissions):
- `user` - Read/write user profile
- `user:email` - Access email addresses
- `repo` - Full repository access
- `public_repo` - Access public repositories
- `admin:org` - Organization administration
- `workflow` - GitHub Actions workflow access

#### OAuth App Registration

**GitHub Developer Settings**:
```
1. Register New OAuth App:
   Settings → Developer settings → OAuth Apps → New OAuth App

2. Application Configuration:
   - Application name: "CI Dashboard"
   - Homepage URL: https://dashboard.example.com
   - Authorization callback URL: https://dashboard.example.com/auth/github/callback
   - Description: "Continuous integration dashboard for team"

3. Credentials Generated:
   - Client ID: 1234567890abcdef1234
   - Client Secret: abcdef1234567890abcdef1234567890abcdef12
   - ⚠️ Store client secret securely (password manager)

4. User Authorization Flow:
   - User clicks "Login with GitHub" in your app
   - Redirect to GitHub authorization URL
   - User approves permissions
   - GitHub redirects with authorization code
   - Your app exchanges code for access token
   - Store token for API requests
```

#### OAuth vs PAT Comparison

| Feature | OAuth App | Personal Access Token |
|---------|-----------|----------------------|
| **User Consent** | Required (explicit approval) | Not required (self-generated) |
| **Credential Exposure** | None (token issued by GitHub) | User creates token |
| **Scope Granularity** | Per-authorization request | Per-token creation |
| **Revocation** | User revokes per-app | User revokes per-token |
| **Use Case** | Third-party integrations | Personal automation |
| **Multi-User** | Supports multiple users | Single-user token |

#### When to Use
- **Third-party services**: CI dashboards, project management tools
- **User delegation**: "Login with GitHub" authentication
- **Multi-user applications**: SaaS platforms with GitHub integration
- **Public integrations**: Marketplace apps, open-source tools

---

### 5. GitHub Apps (Recommended for Programmatic Access)

**Modern Integration Method**: Enhanced authentication with fine-grained permissions, dynamic token generation, and organization-level installation control.

#### GitHub Apps vs OAuth Apps

| Feature | GitHub Apps | OAuth Apps |
|---------|-------------|------------|
| **Permissions** | Fine-grained (per-resource) | Coarse-grained (scope-based) |
| **Token Lifetime** | 1 hour (auto-refresh) | Indefinite (until revoked) |
| **Installation** | Organization/repository-specific | User-account-wide |
| **Rate Limits** | Higher (5,000 req/hr per installation) | Lower (5,000 req/hr per user) |
| **Webhooks** | Dedicated webhook endpoint | Shared with user |
| **Best Practice** | ✅ Recommended | Legacy approach |

#### GitHub App Permissions (Fine-Grained)

**Repository Permissions**:
- `actions` - Read/write GitHub Actions (workflows, runs)
- `checks` - Create/update check runs
- `contents` - Read/write repository files
- `deployments` - Create/update deployments
- `issues` - Read/write issues
- `pull_requests` - Read/write pull requests
- `statuses` - Create/update commit status

**Organization Permissions**:
- `members` - Read organization membership
- `administration` - Manage organization settings
- `team_discussions` - Manage team discussions

**Account Permissions**:
- `emails` - Read user email addresses
- `followers` - Read user followers

#### GitHub App Creation Workflow

**Register GitHub App**:
```
1. Organization Settings:
   Organization → Settings → Developer settings → GitHub Apps → New GitHub App

2. App Configuration:
   - GitHub App name: "Deployment Automator"
   - Description: "Automates deployment workflows"
   - Homepage URL: https://docs.example.com/deployment-automator
   - Webhook URL: https://api.example.com/webhooks/github
   - Webhook secret: <generate strong secret>

3. Permissions:
   Repository permissions:
     ✅ Actions: Read and write
     ✅ Contents: Read and write
     ✅ Deployments: Read and write
     ✅ Pull requests: Read and write
   
   Organization permissions:
     ✅ Members: Read-only

4. Subscribe to Events:
   ✅ Deployment
   ✅ Deployment status
   ✅ Pull request
   ✅ Push
   ✅ Workflow run

5. Where can this GitHub App be installed?
   ○ Only on this account
   ● Any account (public availability)

6. Create GitHub App:
   - App ID: 123456
   - Client ID: Iv1.a1b2c3d4e5f6g7h8
   - Client secret: <generate>
   - Private key: <download PEM file>
```

**Private Key Storage**:
```bash
# Download private key (PEM format)
# deployment-automator.2024-01-01.private-key.pem

# Store in secure location (Azure Key Vault, AWS Secrets Manager)
# Never commit to repository

# Use in application:
# - Read private key from secure storage
# - Generate JWT (JSON Web Token) for authentication
# - Exchange JWT for installation access token
```

#### GitHub App Authentication Flow

**JWT Generation → Installation Token**:
```javascript
// Node.js example using @octokit/auth-app
const { createAppAuth } = require("@octokit/auth-app");

const auth = createAppAuth({
  appId: 123456,
  privateKey: process.env.GITHUB_APP_PRIVATE_KEY, // From secure storage
  installationId: 789012, // Obtained during app installation
});

// Generate installation access token (1-hour lifetime)
const { token } = await auth({ type: "installation" });

// Use token for API requests
const octokit = new Octokit({ auth: token });

// Create deployment
await octokit.repos.createDeployment({
  owner: "organization",
  repo: "application",
  ref: "main",
  environment: "production",
  auto_merge: false,
});
```

**Token Refresh** (Automatic):
```javascript
// Octokit automatically refreshes tokens
// No manual refresh logic required
// Tokens expire after 1 hour
// New token generated on demand
```

#### GitHub App Installation

**Install App to Organization/Repository**:
```
1. Organization Settings:
   Organization → Settings → GitHub Apps → Install "Deployment Automator"

2. Repository Selection:
   ○ All repositories
   ● Only select repositories
     ✅ application-frontend
     ✅ application-backend
     ✅ infrastructure

3. Installation Confirmation:
   - Review permissions
   - Confirm installation
   - Installation ID generated: 789012

4. Webhook Delivery:
   - Events sent to webhook URL
   - Authenticate with webhook secret
   - Process events (deployment, pull request, push)
```

#### When to Use
- **Programmatic API access**: Automation requiring fine-grained permissions
- **Organization-wide integrations**: Multi-repository workflows
- **High rate limits**: Applications with heavy API usage
- **Marketplace apps**: Public GitHub integrations
- **Recommended approach**: Modern replacement for OAuth Apps

---

### 6. GITHUB_TOKEN (Workflow Automation)

**Auto-Generated Token Method**: Ephemeral authentication token automatically provisioned for GitHub Actions workflows with repository-scoped permissions.

#### GITHUB_TOKEN Characteristics
- **Lifetime**: Single workflow run (job-specific)
- **Scope**: Repository where workflow executes
- **Permissions**: Configurable (read/write on contents, issues, PRs, packages)
- **Security**: No manual token management (zero credential leakage)
- **Automatic**: No user action required (GitHub provisions automatically)

#### GITHUB_TOKEN Permissions

**Default Permissions** (GitHub Actions settings):
```yaml
# Repository Settings → Actions → General → Workflow permissions
# Option 1: Read and write permissions (default)
#   - contents: write
#   - issues: write
#   - pull-requests: write
#   - packages: write

# Option 2: Read repository contents and packages permissions
#   - contents: read
#   - packages: read
```

**Workflow-Level Override**:
```yaml
# .github/workflows/deploy.yml
name: Deploy Application

on:
  push:
    branches: [main]

# Override default permissions (least-privilege)
permissions:
  contents: read        # Read repository files
  deployments: write    # Create deployments
  packages: write       # Publish packages

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to production
        run: |
          # GITHUB_TOKEN available in secrets context
          echo "Token: ${{ secrets.GITHUB_TOKEN }}"
```

**Job-Level Permissions**:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build package
        run: npm run build

  deploy:
    runs-on: ubuntu-latest
    permissions:
      deployments: write
      contents: read
    steps:
      - name: Deploy application
        run: ./deploy.sh
```

#### GITHUB_TOKEN Usage Examples

**Comment on Pull Request**:
```yaml
name: PR Comment Bot

on:
  pull_request:
    types: [opened]

jobs:
  comment:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write  # Required for creating comments
    steps:
      - name: Comment on PR
        uses: actions/github-script@v6
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const prNumber = context.payload.pull_request.number;
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: prNumber,
              body: '👋 Thanks for opening this PR! We will review it shortly.'
            });
```

**Publish Package to GitHub Packages**:
```yaml
name: Publish Package

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          registry-url: 'https://npm.pkg.github.com'
      
      - name: Publish to GitHub Packages
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Create Deployment**:
```yaml
name: Production Deployment

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      deployments: write
      contents: read
    steps:
      - uses: actions/checkout@v3
      
      - name: Create deployment
        uses: chrnorm/deployment-action@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          environment: production
          ref: ${{ github.sha }}
```

**Update Commit Status**:
```yaml
name: Build Status

on:
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      statuses: write
    steps:
      - uses: actions/checkout@v3
      
      - name: Run tests
        id: tests
        run: npm test
      
      - name: Update commit status
        if: always()
        uses: actions/github-script@v6
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            await github.rest.repos.createCommitStatus({
              owner: context.repo.owner,
              repo: context.repo.repo,
              sha: context.sha,
              state: '${{ steps.tests.outcome }}' === 'success' ? 'success' : 'failure',
              context: 'CI Tests',
              description: 'Test suite execution'
            });
```

#### Security Best Practices
- ✅ **Least-privilege permissions**: Only grant required permissions per job
- ✅ **Workflow-level scoping**: Override defaults for specific workflows
- ✅ **Audit permission usage**: Review Actions logs for token activity
- ❌ **Don't use GITHUB_TOKEN for external APIs**: Use dedicated PATs/secrets
- ❌ **Avoid broad write permissions**: Default "read and write" enables excessive access

#### When to Use
- **GitHub Actions workflows**: Primary authentication method
- **Repository automation**: Issues, PRs, deployments, packages
- **No credential management**: Zero-overhead token provisioning
- **Ephemeral access**: Single-run scope prevents long-term exposure

---

## SAML Single Sign-On (SSO) for Enterprise

**Enterprise Identity Integration**: Centralized authentication with Microsoft Entra ID, Okta, OneLogin, or other SAML 2.0 identity providers.

### SAML SSO Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  1. User Accesses GitHub:                                    │
│     https://github.com/organizations/COMPANY                 │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│  2. GitHub Detects SSO Requirement:                          │
│     "COMPANY organization requires SAML SSO"                 │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│  3. Redirect to Identity Provider (IdP):                     │
│     https://login.microsoftonline.com/TENANT_ID/saml2        │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│  4. User Authenticates with IdP:                             │
│     - Username: user@company.com                             │
│     - Password: <corporate password>                         │
│     - MFA: <Authenticator code>                              │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│  5. IdP Issues SAML Assertion:                               │
│     <saml:Assertion>                                         │
│       <saml:Subject>user@company.com</saml:Subject>          │
│       <saml:Attribute Name="email">user@company.com</saml:Attribute> │
│     </saml:Assertion>                                        │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│  6. GitHub Validates SAML Assertion:                         │
│     - Verify signature with IdP certificate                  │
│     - Check assertion expiration                             │
│     - Extract user attributes                                │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│  7. GitHub Grants Access:                                    │
│     "Welcome, user@company.com"                              │
│     ✅ Authenticated via COMPANY SSO                         │
└─────────────────────────────────────────────────────────────┘
```

### SAML SSO Configuration

**GitHub Organization Settings**:
```
1. Enable SAML SSO:
   Organization → Settings → Security → SAML single sign-on
   ✅ Enable SAML authentication

2. IdP Configuration (Microsoft Entra ID Example):
   - Sign-on URL: https://login.microsoftonline.com/TENANT_ID/saml2
   - Issuer: https://sts.windows.net/TENANT_ID/
   - Public certificate: <upload X.509 certificate from Entra ID>

3. Attribute Mapping:
   - GitHub Username: user.userprincipalname
   - Email: user.mail
   - Name: user.displayname

4. Test SAML Configuration:
   - Click "Test SAML configuration"
   - Authenticate with IdP
   - Verify assertion attributes

5. Require SAML SSO:
   ✅ Require SAML SSO authentication for all members
   Result: All organization members must authenticate via IdP
```

**Microsoft Entra ID Configuration**:
```
1. Register GitHub Enterprise Application:
   Entra ID → Enterprise applications → New application → GitHub

2. Single Sign-On Configuration:
   - SSO Method: SAML
   - Basic SAML Configuration:
     * Identifier (Entity ID): https://github.com/orgs/ORGANIZATION
     * Reply URL: https://github.com/orgs/ORGANIZATION/saml/consume
   
3. User Attributes & Claims:
   - Unique User Identifier: user.userprincipalname
   - Email: user.mail
   - Name: user.displayname
   - Given name: user.givenname
   - Surname: user.surname

4. SAML Signing Certificate:
   - Download Certificate (Base64)
   - Upload to GitHub organization settings

5. Assign Users/Groups:
   - Entra ID → Enterprise applications → GitHub → Users and groups
   - Add assignment: "Engineering" security group
   - Result: All "Engineering" members can access GitHub org
```

### SAML SSO + MFA Enforcement

**Conditional Access Policy** (Microsoft Entra ID):
```
1. Create Conditional Access Policy:
   Entra ID → Security → Conditional Access → New policy

2. Policy Configuration:
   - Name: "GitHub - Require MFA"
   - Users: Include "Engineering" group
   - Cloud apps: GitHub enterprise application
   - Conditions: Any device, any location
   - Grant: Require multifactor authentication
   - Session: Sign-in frequency: Every 8 hours

3. Result:
   - Users accessing GitHub via SAML SSO MUST complete MFA
   - MFA enforced by Entra ID (not GitHub)
   - Organization-wide policy (centralized control)
```

### Team Synchronization

**Entra ID Group → GitHub Team Mapping**:
```
1. Enable Team Synchronization:
   Organization → Settings → Security → Team synchronization
   ✅ Enable team synchronization

2. Map Entra ID Group to GitHub Team:
   Organization → Teams → "Backend Engineers"
   - Link Entra ID group: "Engineering-Backend"
   - Sync frequency: Real-time (webhook-based)

3. Automated Membership:
   - User added to "Engineering-Backend" in Entra ID
   - GitHub automatically adds to "Backend Engineers" team
   - Repository access inherited from team permissions
   - User removed from Entra ID group → GitHub membership revoked

4. Result:
   - Zero manual GitHub team management
   - Centralized access control in Entra ID
   - Audit trail in both systems
```

---

## Authentication Method Selection Guide

### Decision Tree

```
Start: What is your use case?
├── Human Interactive Access
│   ├── Personal Account → Username + 2FA (TOTP/Security Key)
│   └── Enterprise Organization → SAML SSO + MFA Enforcement
│
├── Git Operations (Clone, Push, Pull)
│   ├── Developer Workstation → SSH Keys (Ed25519)
│   └── CI/CD System → GITHUB_TOKEN (Actions) or Deploy Keys
│
├── API Automation
│   ├── Personal Scripts → Personal Access Token (short expiration)
│   ├── Organization Service → GitHub App (fine-grained permissions)
│   └── Third-Party Integration → OAuth App (user delegation)
│
└── GitHub Actions Workflows
    └── Repository Automation → GITHUB_TOKEN (auto-generated)
```

### Comparison Table

| Authentication Method | Security | Automation | Scope | Recommended For |
|----------------------|----------|------------|-------|-----------------|
| **Username + 2FA** | High (with MFA) | No | Full account | Human interactive |
| **SSH Keys** | High | Yes | Git protocol | Developer workstations |
| **Personal Access Token** | Medium | Yes | Scoped permissions | Personal automation |
| **OAuth App** | Medium | Yes | User-delegated | Third-party services |
| **GitHub App** | High | Yes | Fine-grained | Organization integrations |
| **GITHUB_TOKEN** | High | Yes | Repository-scoped | Workflow automation |
| **SAML SSO** | Very High | N/A | Organization-wide | Enterprise identity |

---

## Security Best Practices Summary

### 1. Enable Multifactor Authentication
- ✅ **Personal accounts**: Settings → Password and authentication → Two-factor authentication
- ✅ **Organizations**: Settings → Security → Require two-factor authentication
- ✅ **Recommended method**: TOTP (authenticator app) or Security Keys (YubiKey)

### 2. Use SSH Keys for Git Operations
- ✅ **Generate Ed25519 keys**: `ssh-keygen -t ed25519 -C "email@example.com"`
- ✅ **Protect with passphrase**: Add encryption layer to private key
- ✅ **Separate keys per device**: Enable granular revocation

### 3. Scope Personal Access Tokens
- ✅ **Minimum permissions**: Only grant required scopes
- ✅ **Short expiration**: 30-90 days for automation
- ✅ **Secure storage**: Azure Key Vault, AWS Secrets Manager

### 4. Prefer GitHub Apps Over OAuth Apps
- ✅ **Fine-grained permissions**: Resource-level control
- ✅ **Automatic token refresh**: 1-hour tokens prevent long-term exposure
- ✅ **Organization control**: Admins approve app installations

### 5. Implement SAML SSO for Enterprise
- ✅ **Centralized authentication**: Single identity source (Entra ID)
- ✅ **MFA enforcement**: Conditional Access policies
- ✅ **Team synchronization**: Automated membership management

### 6. Audit Authentication Activity
- ✅ **Organization audit log**: Settings → Audit log
- ✅ **Review token usage**: Security log → API request tracking
- ✅ **Monitor failed attempts**: Detect brute force attacks

---

## Next Steps

✅ **Completed**: GitHub authentication methods and SAML SSO integration

**Next Unit**: Learn about GitHub permission models and role-based access control →

---

[Learn More](https://learn.microsoft.com/en-us/training/modules/integrate-identity-management-systems/2-integrate-github-single-sign-sso)
