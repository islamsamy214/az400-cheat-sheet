# Link GitHub to Azure Boards

## Overview
Integration creates a **hybrid approach**: GitHub's developer-friendly environment + Azure Boards' enterprise planning capabilities.

## Business Benefits
- **Unified visibility**: Stakeholders track progress without accessing GitHub
- **Compliance and audit**: Formal project tracking + developer workflows preserved
- **Cross-functional collaboration**: Bridge gap between PMs and dev teams
- **Automated synchronization**: Reduce manual overhead in keeping status current

## Azure Boards App: Integration Foundation

**Key Capabilities**:
- **Bidirectional linking**: Connect Azure Boards work items with GitHub commits, PRs, issues
- **Automated state transitions**: Complete work items when GitHub mentions include "fixes"/"fixed"
- **Real-time synchronization**: GitHub changes immediately reflect in Azure Boards
- **Flexible scope control**: Organization-wide or selective repository integration

### Prerequisites

**GitHub Requirements**:
- **Repository access**: Administrator or owner permissions
- **Organization permissions**: Owner role for org-wide installations
- **Marketplace access**: Ability to install applications from GitHub Marketplace

**Azure DevOps Requirements**:
- **Project collection administrator**: Required for initial connection setup
- **Project permissions**: Stakeholder access or higher to view linked work items
- **Authentication credentials**: PAT or OAuth for secure API access

## Authentication & Security

### GitHub Authentication Options

| Method | Security Level | Best For | Considerations |
|--------|----------------|----------|----------------|
| **Personal Access Token** | High | Automated integrations, CI/CD | Requires careful scope management |
| **Username/Password** | Medium | Individual user connections | Less secure, not recommended for prod |
| **OAuth Apps** | High | Organization-wide integrations | Centralized access management |

### Security Best Practices
- **Principle of least privilege**: Grant only necessary permissions
- **Token rotation**: Regularly update PATs and review access
- **Audit logging**: Monitor integration activity for unusual patterns
- **Repository isolation**: Connect only repos requiring Azure Boards integration

### Connection Process
1. Install Azure Boards App from GitHub Marketplace
2. Configure repository access (all repos or selective)
3. Authenticate with Azure DevOps using preferred method
4. Map repositories to projects in Azure Boards
5. Test integration with sample links and commits

## Configuration Flexibility & Management

### GitHub-Side Management
- **Repository scope control**: Add or remove specific repositories
- **Project mapping**: Configure which Azure Boards projects connect to each repo
- **Integration suspension**: Temporarily disable without losing configuration
- **Complete removal**: Uninstall app and remove all connections

### Azure Boards-Side Management
- **Multi-repository connections**: Link multiple GitHub repos to single projects
- **Cross-project linking**: Repositories connect to multiple Azure Boards projects
- **Connection health monitoring**: Track integration status, resolve connectivity issues
- **Permission management**: Control who can modify GitHub connections

## Supported Integration Scenarios

### From GitHub ✅
- Integrate all repositories or select repositories for GitHub account/organization
- Add or remove GitHub repositories from integration
- Configure which project they connect to
- Suspend Azure Boards-GitHub integration or uninstall app

### From Azure Boards ✅
- Connect one or more GitHub repositories to Azure Boards project
- Add or remove GitHub repositories from GitHub connection
- Completely remove GitHub connection for a project
- Allow GitHub repository to connect to multiple Azure Boards projects (same org/collection)

### Operational Tasks Supported ✅
- **Create links** between work items and GitHub commits, PRs, issues (via GitHub mentions)
- **State transitions**: Work items → Done/Completed when using "fix", "fixes", or "fixed" in GitHub mentions
- **Full traceability**: Post discussion comment to GitHub when linking from work item
- **Development section**: Show linked GitHub code artifacts in work item
- **Kanban annotations**: Show linked artifacts as annotations on Kanban board cards
- **Status badges**: Support Kanban board column status badges in GitHub repositories

### Not Supported ❌
- Query for work items with links to GitHub artifacts
  - **Workaround**: Query for work items with External Link Count > 0

## Reference Links
- [Connect Azure Boards to GitHub](https://learn.microsoft.com/en-us/azure/devops/boards/github/connect-to-github)
- [Change GitHub repository access or suspend integration](https://learn.microsoft.com/en-us/azure/devops/boards/github/change-azure-boards-app-github-repository-access)
- [Add or remove GitHub repositories](https://learn.microsoft.com/en-us/azure/devops/boards/github/add-remove-repositories)
- [Link GitHub commits, pull requests, and issues to work items](https://learn.microsoft.com/en-us/azure/devops/boards/github/link-to-from-github)

## Critical Notes
- 🎯 **Azure Boards App** is the technical bridge between platforms
- 💡 **Real-time sync**: Changes in GitHub immediately reflect in Azure Boards
- ⚠️ Use **OAuth Apps** for organization-wide integrations (best security)
- 📊 **Bidirectional linking** enables full traceability
- 🔒 **Token rotation** is essential for security
- ✨ Keywords "fix", "fixes", "fixed" automatically transition work items

[Learn More](https://learn.microsoft.com/en-us/training/modules/plan-agile-github-projects-azure-boards/5-link-github-to-azure-boards)
