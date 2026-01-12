# Explore Foster Inner Source

## Key Concepts
- **Inner Source**: "Internal open source" bringing open-source practices inside your company
- **Fork-Based Workflow**: Popular with open-source projects, now available for internal collaboration
- **Cross-Team Contribution**: Teams can contribute to projects they don't normally develop
- **Before Forks**: Traditional pull request workflow only worked within same repository

## Traditional Team Workflow (Before Forks)
1. Push a new branch to your repository
2. Open a pull request for code review
3. Azure Repos checks branch policies
4. Merge to main and deploy when approved

**Limitation**: Only works within your own project team

## What is Inner Source?

Inner source brings open-source benefits inside your organization:

| Benefit | Description |
|---------|-------------|
| **Cross-Team Collaboration** | Teams work together on projects even without regular collaboration |
| **Knowledge Sharing** | Developers learn from code written by other teams |
| **Code Reuse** | Build on existing functionality instead of rebuilding multiple times |
| **Quality Improvement** | More reviewers and contributors lead to better software quality |
| **Faster Innovation** | Build on existing solutions rather than starting from scratch |

## Microsoft's Inner Source Journey

### Before Inner Source
- **Siloed Teams**: Only Windows engineers could read Windows code
- **No Cross-Team Access**: Office developers couldn't view Windows code
- **Blocked Contributions**: Visual Studio engineers couldn't fix bugs in Windows/Office
- **Isolated Development**: Teams worked in complete isolation

### After Inner Source
- **Universal Access**: All project source code open to everyone within Microsoft
- **Fork-Based Contribution**: Anyone can fork and contribute without write access
- **Better Collaboration**: Teams work together across traditional boundaries
- **Faster Bug Fixes**: Anyone can fix bugs they discover
- **Reduced Duplication**: Teams build on existing work

## How Forks Enable Inner Source
- Fork repositories without needing write access
- Make changes in your fork independently
- Submit pull requests to contribute back
- Original team retains control through PR review process

## Critical Notes
- 🎯 Forks are at the heart of inner source practices
- 💡 You only need read access to fork a repository, not write access
- ⚠️ Inner source requires cultural shift toward openness and collaboration
- 📊 Microsoft uses this approach company-wide with Azure Repos
- 🔄 Fork workflow enables contribution from anyone in the organization
- ✨ Inner source keeps code secure within your organization while enabling collaboration

[Learn More](https://learn.microsoft.com/en-us/training/modules/plan-fostering-inner-source/2-explore-foster-inner-source)
