# Knowledge Check

## Overview
Assessment questions to validate understanding of knowledge sharing, Azure DevOps wikis, and Microsoft Teams integration.

## Key Topics Covered
- Cost and impact of lost knowledge
- Documentation strategies
- Azure DevOps wiki types and features
- Markdown and Mermaid diagrams
- Microsoft Teams integrations (GitHub, Azure DevOps)

## Sample Assessment Questions

### Question 1: Knowledge Sharing Impact
**What is the primary cost of lost team knowledge?**

A) Increased cloud infrastructure expenses  
B) Repeated mistakes and slower problem resolution  
C) More meetings required  
D) Higher software licensing costs  

**Answer**: B) Repeated mistakes and slower problem resolution

**Explanation**: Lost knowledge forces teams to rediscover solutions, repeat past mistakes, and spend more time on problems that were previously solved. This impacts productivity and quality.

---

### Question 2: Documentation Best Practices
**Which approach is most effective for creating team documentation?**

A) Document everything comprehensively upfront  
B) Wait until project completion to document  
C) Start with "just enough" documentation when you feel pain  
D) Only senior developers should write documentation  

**Answer**: C) Start with "just enough" documentation when you feel pain

**Explanation**: The most effective approach is to document when the lack of information causes problems. Start small and grow documentation based on actual needs.

---

### Question 3: Wiki Types
**What is the key difference between a Project Wiki and a Published Wiki in Azure DevOps?**

A) Project wikis support Markdown, published wikis don't  
B) Project wikis are created per-project, published wikis are from existing Git repositories  
C) Published wikis don't support Mermaid diagrams  
D) Project wikis don't use Git for version control  

**Answer**: B) Project wikis are created per-project, published wikis are from existing Git repositories

**Explanation**: A project wiki is provisioned specifically for the project (creates new Git repo), while a published wiki converts an existing Git repository into a wiki.

---

### Question 4: Wiki Permissions
**What permission is required to create a project wiki in Azure DevOps?**

A) Project Administrator  
B) Build Administrator  
C) Create Repository  
D) Stakeholder access  

**Answer**: C) Create Repository

**Explanation**: Creating a project wiki requires the "Create Repository" permission because Azure DevOps creates a Git repository to store the wiki content.

---

### Question 5: Markdown Features
**Which of the following is a GitHub Flavored Markdown (GFM) extension?**

A) Headings  
B) Bold text  
C) Task lists with checkboxes  
D) Hyperlinks  

**Answer**: C) Task lists with checkboxes

**Explanation**: Task lists (`- [ ]` for unchecked, `- [x]` for checked) are a GFM extension. Headings, bold text, and hyperlinks are standard Markdown features.

---

### Question 6: Mermaid Diagrams
**What is the main advantage of using Mermaid diagrams in Azure DevOps wikis?**

A) They are more colorful than other diagrams  
B) They are text-based and version-controlled with the wiki  
C) They require specialized diagramming software  
D) They automatically generate from code  

**Answer**: B) They are text-based and version-controlled with the wiki

**Explanation**: Mermaid diagrams are defined in plain text, so they're stored and versioned alongside wiki content in Git. No external diagramming tools needed.

---

### Question 7: Mermaid Diagram Types
**Which Mermaid diagram type is best for showing interactions between system components over time?**

A) Flowchart  
B) Class diagram  
C) Sequence diagram  
D) Gantt chart  

**Answer**: C) Sequence diagram

**Explanation**: Sequence diagrams show interactions between participants (components) over time, making them ideal for API calls, authentication flows, and system interactions.

---

### Question 8: Teams Integration - GitHub
**What is the primary benefit of integrating GitHub with Microsoft Teams?**

A) Eliminates need for Git repositories  
B) Automatically fixes code bugs  
C) Reduces context switching by bringing notifications into Teams  
D) Replaces need for pull request reviews  

**Answer**: C) Reduces context switching by bringing notifications into Teams

**Explanation**: The integration brings GitHub events (PRs, issues, workflows) into Teams, allowing developers to stay in their collaboration tool and respond to events without switching to GitHub constantly.

---

### Question 9: Teams Integration - Subscriptions
**In Microsoft Teams, what does the following command do?**  
`@github subscribe owner/repo pulls:opened,merged+branch:main`

A) Subscribes to all GitHub events  
B) Subscribes to PRs opened or merged on the main branch only  
C) Creates a new GitHub repository  
D) Merges all open pull requests  

**Answer**: B) Subscribes to PRs opened or merged on the main branch only

**Explanation**: This command uses filters to subscribe only to pull request events (opened and merged) that target the main branch, reducing notification noise.

---

### Question 10: Teams Integration - Azure Boards
**Which Azure Boards feature allows you to create work items directly from Microsoft Teams?**

A) Subscriptions  
B) Compose extension  
C) Channel tabs  
D) Kanban board  

**Answer**: B) Compose extension

**Explanation**: The Azure Boards compose extension enables creating work items directly from Teams message compose area without leaving the Teams interface.

---

### Question 11: Teams Integration - Azure Pipelines
**What can you do with Azure Pipelines integration in Microsoft Teams?**

A) Write YAML pipeline definitions  
B) Approve deployments directly from Teams  
C) Edit build agents  
D) Modify repository code  

**Answer**: B) Approve deployments directly from Teams

**Explanation**: Azure Pipelines integration allows approving or rejecting deployment approvals directly from Teams notifications, enabling faster responses without switching to Azure DevOps.

---

### Question 12: Teams Integration Best Practices
**What is the recommended approach when setting up DevOps tool integrations in Microsoft Teams?**

A) Subscribe to all events from all repositories immediately  
B) Create one channel for all notifications  
C) Start with focused subscriptions and use filters to reduce noise  
D) Disable threading to keep conversations separate  

**Answer**: C) Start with focused subscriptions and use filters to reduce noise

**Explanation**: Best practice is to start narrow (high-value events only) with strategic filters, then expand based on team feedback. Too many notifications lead to "alert fatigue" and ignored channels.

---

## Quick Reference: Key Concepts for Exam

### Knowledge Sharing
| Concept | Key Points |
|---------|------------|
| **Cost of Lost Knowledge** | Repeated mistakes, slower resolution, inconsistent practices, extended onboarding |
| **What to Document** | Architecture decisions (ADRs), coding standards, setup guides, common problems, processes |
| **Documentation Strategies** | Start with "just enough", make discoverable, assign ownership, keep close to code, use templates, automate |

### Azure DevOps Wikis
| Concept | Key Points |
|---------|------------|
| **Project Wiki** | Provisioned per-project, dedicated Git repo, one per project |
| **Published Wiki** | From existing Git repository, multiple allowed, version-controlled |
| **Permissions** | Requires "Create Repository" permission; Contributors can edit; Stakeholders read-only |
| **Storage** | Git-backed (all changes are commits) |

### Markdown and Mermaid
| Concept | Key Points |
|---------|------------|
| **Markdown Fundamentals** | Headings (#), bold (**), italic (*), lists, links, images, code blocks, tables |
| **GFM Extensions** | Task lists, tables, strikethrough, autolinks, syntax highlighting, emoji |
| **Mermaid Benefits** | Text-based, version-controlled, no external tools, multiple diagram types |
| **Mermaid Types** | Flowchart, sequence, class, ER, Gantt, state, pie, git graph |

### Microsoft Teams Integration
| Tool | Integration Capabilities |
|------|--------------------------|
| **GitHub** | Subscribe to repos/PRs/issues/workflows, filters, link previews, PR reminders, direct actions |
| **Azure Boards** | Work item subscriptions, create items (compose extension), Kanban board tabs |
| **Azure Repos** | PR/commit notifications, threaded discussions, compose extension |
| **Azure Pipelines** | Build/release notifications, deployment approvals, pipeline triggers, filters |

### Teams Integration Best Practices
- Start with focused subscriptions
- Use filters strategically
- Organize with dedicated channels
- Enable threading for context
- Establish team conventions
- Review and refine regularly

## Study Tips
- **Understand the "why"**: Know why each tool/feature exists (e.g., wikis for lasting documentation, Teams integration to reduce context switching)
- **Compare and contrast**: Be clear on differences (project vs. published wiki, Markdown vs. GFM, different Mermaid diagram types)
- **Practical application**: Think about real-world scenarios (when to document, which tool to use, how to reduce notification noise)
- **Commands and syntax**: Know key commands (`@github subscribe`, `@azure boards create`) and basic Markdown/Mermaid syntax

## Critical Notes
- ⚠️ **One project wiki per project**: But multiple published wikis allowed
- 💡 **Git-backed means version-controlled**: Can revert any wiki change
- 🎯 **Filters reduce noise**: Critical for effective Teams integration
- 📊 **Text-based diagrams**: Mermaid's key advantage (version control, no external tools)
- 🔗 **Compose extensions**: Enable creation from Teams (work items, PRs)

[Learn More](https://learn.microsoft.com/en-us/training/modules/share-knowledge-within-teams/6-knowledge-check)
