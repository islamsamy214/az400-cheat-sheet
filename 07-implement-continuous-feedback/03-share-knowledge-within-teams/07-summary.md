# Summary: Share Knowledge Within Teams

## Module Overview
This module covered strategies and tools for effective knowledge sharing in DevOps teams, focusing on Azure DevOps wikis, Markdown documentation, Mermaid diagrams, and Microsoft Teams integrations.

## Key Concepts Learned

### 1. The Cost of Lost Knowledge
**Understanding the Impact**:
- **Repeated mistakes**: Teams rediscover problems and make same errors
- **Slower resolution**: Time wasted rediscovering known solutions
- **Inconsistent practices**: Each developer creates own approach
- **Extended onboarding**: New members take months to become productive
- **Context switching costs**: Work blocks when expert unavailable

**Takeaway**: Knowledge sharing is not optional—it's essential for team efficiency, quality, and resilience.

---

### 2. What Knowledge to Document
**Critical Documentation Areas**:
- **Architecture decisions**: Technology choices, trade-offs, rationale (ADRs)
- **Coding standards**: Naming conventions, patterns, testing requirements
- **Setup and configuration**: Environment setup, deployment procedures
- **Common problems**: Known issues, workarounds, troubleshooting guides
- **Processes and workflows**: Code review, deployment, incident response
- **System knowledge**: Data flows, authentication, integration points

**Effective Strategy**: Start with "just enough" documentation when you feel pain from missing information. Don't try to document everything upfront—grow organically based on team needs.

---

### 3. Documentation Strategies That Work
**Best Practices**:
- **Start small**: Begin with simple README files, grow as questions arise
- **Make discoverable**: Consistent locations, clear navigation, search
- **Assign ownership**: Documentation owners for each area, part of definition of done
- **Keep close to code**: README files, code comments, wiki references
- **Use templates**: ADRs, troubleshooting guides, setup instructions for consistency
- **Automate where possible**: Generate API docs, dependency graphs, config docs from code

**Avoid**: Comprehensive upfront documentation (gets stale), documentation separate from code (hard to find), no ownership (docs decay).

---

### 4. Azure DevOps Wikis

#### Wiki Types
| Feature | Project Wiki | Published Wiki |
|---------|--------------|----------------|
| **Purpose** | General project documentation | Version-controlled docs from Git repo |
| **Creation** | Provisioned per project | Publish existing repository |
| **Quantity** | One per project | Multiple allowed |
| **Storage** | Dedicated Git repo | Existing repo branch/folder |
| **Use Case** | Architecture, onboarding, processes | API docs, code guides, release notes |

#### Key Features
- **Git-backed storage**: Every wiki edit is a Git commit (full version control)
- **Markdown-based**: Lightweight formatting, easy to write and read
- **Permission control**: Project-level or repository-level security
- **Navigation**: Auto-generated from page hierarchy
- **Rich content**: Images, attachments, code blocks, tables, diagrams

#### When to Use Each Type
- **Project Wiki**: For lasting project-wide documentation not tied to specific code versions
- **Published Wiki**: When documentation should version with code (API docs, feature guides)

---

### 5. Markdown and GitHub Flavored Markdown (GFM)

#### Standard Markdown
- Headings (`#`, `##`, `###`)
- Bold (`**text**`), italic (`*text*`)
- Lists (unordered `-`, ordered `1.`)
- Links (`[text](url)`), images (`![alt](url)`)
- Code blocks (triple backticks)
- Tables (pipe-separated)

#### GFM Extensions
- **Task lists**: `- [ ]` unchecked, `- [x]` checked
- **Strikethrough**: `~~text~~`
- **Autolinks**: URLs automatically become links
- **Syntax highlighting**: Language-specific code formatting
- **Emoji**: `:smile:`, `:rocket:`, `:white_check_mark:`
- **Mentions**: `@username`, `#workitem-id`

**Advantage**: Plain text, version-controlled, no special tools required, readable raw and rendered.

---

### 6. Mermaid Diagrams

#### What is Mermaid?
Text-based diagram syntax that renders as graphics in Azure DevOps wikis. Diagrams are stored as plain text and version-controlled with wiki content.

#### Supported Diagram Types
| Type | Purpose | Example Use |
|------|---------|-------------|
| **Flowchart** | Process flows, decision trees | CI/CD pipelines, troubleshooting |
| **Sequence** | Component interactions over time | API calls, authentication flows |
| **Class** | Object relationships | Software architecture, data models |
| **ER Diagram** | Database schema | Database design, entity relationships |
| **Gantt** | Project timelines | Sprint planning, milestones |
| **State** | State machines | Application state, workflows |
| **Pie** | Data distribution | Metrics, usage statistics |
| **Git Graph** | Repository branching | Branch strategies, release flows |

#### Benefits
- **Version-controlled**: Text-based, stored in Git with wiki pages
- **No external tools**: Create diagrams directly in Markdown
- **Collaborative**: Team can edit diagram text like any code
- **Consistent**: Single source of truth alongside documentation

**Best Practice**: Use Mermaid for diagrams that need to stay in sync with documentation. Use external tools (Visio, Lucidchart) only for complex, presentation-quality diagrams.

---

### 7. Microsoft Teams Integration

#### Why Integrate DevOps Tools with Teams?
- **Reduce context switching**: Stay in Teams, act on DevOps events
- **Increase visibility**: Real-time notifications for team
- **Accelerate responses**: Faster reviews, approvals, incident response
- **Centralize communication**: Discuss DevOps events where team collaborates

#### GitHub + Teams
**Capabilities**:
- Subscribe to repositories (PRs, issues, commits, releases, workflows, deployments)
- Filter notifications (specific branches, labels, event types)
- Link previews for GitHub URLs
- PR reminders (daily/weekly)
- Direct actions (approve, merge, comment)

**Example**: `@github subscribe owner/repo pulls:opened,merged+branch:main`

#### Azure Boards + Teams
**Capabilities**:
- Work item subscriptions (created, updated, state changed, assigned, commented)
- Filter by type, area path, priority, state
- Create work items via compose extension
- Kanban board tabs in Teams channels

**Example**: `@azure boards subscribe https://dev.azure.com/org/project/_workitems type="Bug"`

#### Azure Repos + Teams
**Capabilities**:
- PR and commit notifications
- Threaded conversations (organized discussions under events)
- Compose extension for searching/sharing PRs
- Subscription management

**Example**: `@azure repos subscribe https://dev.azure.com/org/project/_git/repo pullrequest`

#### Azure Pipelines + Teams
**Capabilities**:
- Build/release notifications (started, completed, stage events)
- Deployment approval directly from Teams
- Filter by pipeline, branch, stage, status
- Compose extension for triggering pipelines

**Example**: `@azure pipelines subscribe https://dev.azure.com/org/project/_build?definitionId=123 runStage:failure`

#### Integration Best Practices
1. **Start focused**: Subscribe to high-value events only, expand based on feedback
2. **Use filters**: Reduce noise (specific branches, failure-only, priorities)
3. **Dedicated channels**: Organize by purpose (#dev-prs, #dev-builds, #prod-deployments)
4. **Enable threading**: Keep conversations organized under events
5. **Team conventions**: Establish emoji/response patterns for common actions
6. **Review regularly**: Adjust subscriptions quarterly based on team feedback

**Anti-Pattern**: Subscribing to all events from all repositories = alert fatigue and ignored channels.

---

## Practical Applications

### Scenario 1: New Team Member Onboarding
**Problem**: New developer takes 2 months to become productive  
**Solution**:
- Create wiki "Onboarding" section with setup guides
- Document environment configuration with step-by-step instructions
- Include Mermaid diagram showing system architecture
- Link to coding standards, testing guidelines, PR process
- Teams integration shows real-time project activity

**Result**: Onboarding reduced to 2 weeks, new members productive faster.

---

### Scenario 2: Recurring Production Issues
**Problem**: Same database connection error debugged 3 times in 6 months  
**Solution**:
- Create wiki "Troubleshooting" page with known issues
- Add Mermaid flowchart for diagnosing connection problems
- Document resolution steps with code examples
- Link from error message documentation
- Teams notification when issue occurs → team references wiki immediately

**Result**: Issue resolved in minutes instead of hours, no rediscovery needed.

---

### Scenario 3: Slow PR Review Process
**Problem**: PRs sit for days awaiting review  
**Solution**:
- Integrate GitHub with Teams dedicated #dev-prs channel
- Subscribe to `pulls:opened+branch:main`
- Enable PR reminders (daily)
- Team sees notifications immediately, reviews faster
- Approve directly from Teams

**Result**: Average PR review time drops from 3 days to 4 hours.

---

### Scenario 4: Architecture Decision Confusion
**Problem**: Developers unsure why certain technology choices were made  
**Solution**:
- Create wiki section for Architecture Decision Records (ADRs)
- Document each major decision with context, decision, consequences
- Include Mermaid architecture diagrams
- Reference ADRs in code comments
- Teams integration shares ADR updates in #architecture channel

**Result**: Team understands rationale, makes consistent decisions, reduces technical debt.

---

## Tools Summary

| Tool | Purpose | Key Features | When to Use |
|------|---------|--------------|-------------|
| **Azure DevOps Project Wiki** | Lasting project documentation | Git-backed, Markdown, Mermaid, permissions | Architecture, onboarding, processes, runbooks |
| **Azure DevOps Published Wiki** | Code-versioned documentation | From existing repo, multiple wikis, branching | API docs, feature guides, release notes |
| **Markdown/GFM** | Lightweight text formatting | Plain text, version-controlled, readable | All documentation (README, wiki, comments) |
| **Mermaid Diagrams** | Text-based diagrams | Version-controlled, no external tools, types | Architecture, flows, data models, timelines |
| **Microsoft Teams Integrations** | Reduce context switching | Real-time notifications, direct actions, filters | Daily collaboration, PR reviews, builds, approvals |

---

## Key Takeaways

1. **Knowledge sharing is essential**: Lost knowledge costs time, quality, and team resilience
2. **Document strategically**: Start small when you feel pain, not everything upfront
3. **Use the right tool**: Wikis for lasting docs, code comments for implementation details, Teams for real-time coordination
4. **Make it discoverable**: Consistent locations, clear navigation, close to code
5. **Assign ownership**: Documentation decays without accountability
6. **Leverage automation**: Generate docs from code, use templates, auto-update
7. **Integrate with collaboration**: Teams integration brings DevOps events to where team works
8. **Start focused**: Reduce notification noise with filters and focused subscriptions
9. **Version control everything**: Git-backed wikis and text-based diagrams enable collaboration and rollback
10. **Iterate and improve**: Review documentation and integrations regularly, adjust based on feedback

---

## Additional Resources

### Official Microsoft Documentation
- [Azure DevOps Wikis Overview](https://learn.microsoft.com/en-us/azure/devops/project/wiki/)
- [Markdown Syntax for Wikis](https://learn.microsoft.com/en-us/azure/devops/project/wiki/markdown-guidance)
- [Integrate Azure DevOps with Microsoft Teams](https://learn.microsoft.com/en-us/azure/devops/pipelines/integrations/microsoft-teams)

### GitHub Resources
- [GitHub + Microsoft Teams Integration](https://github.com/integrations/microsoft-teams)
- [GitHub Flavored Markdown Spec](https://github.github.com/gfm/)
- [GitHub Documentation Best Practices](https://docs.github.com/en/communities/documenting-your-project-with-wikis)

### Mermaid Diagram Tools
- [Mermaid Live Editor](https://mermaid.live/) - Test diagrams before adding to wiki
- [Mermaid Documentation](https://mermaid.js.org/) - Complete syntax reference
- [Mermaid Cheat Sheet](https://jojozhuang.github.io/tutorial/mermaid-cheat-sheet/)

### Knowledge Sharing Practices
- [Architecture Decision Records (ADRs)](https://adr.github.io/) - Template and examples
- [Documentation Best Practices](https://www.writethedocs.org/guide/) - Write the Docs community
- [Awesome README](https://github.com/matiassingers/awesome-readme) - README examples and templates

---

## What's Next?
Continue learning about implementing continuous feedback:
- **Module 4**: Design processes to automate application analytics
- **Module 5**: Manage alerts, blameless retrospectives, and a just culture

Build on knowledge sharing with monitoring, alerting, and continuous improvement practices.

---

## Critical Notes
- ⚠️ **Git-backed = version-controlled**: All wiki changes are Git commits, can revert
- 💡 **Start small, grow organically**: Don't document everything upfront
- 🎯 **Ownership is accountability**: Assign owners or docs decay
- 📊 **Filters are essential**: Too many notifications = ignored notifications
- 🔗 **Close to code**: README + code comments + wiki references

[Learn More](https://learn.microsoft.com/en-us/training/modules/share-knowledge-within-teams/7-summary)
