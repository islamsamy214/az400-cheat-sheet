# Introduction

## Key Concepts
- **Knowledge sharing** preserves organizational learning, reduces repeated mistakes, and accelerates onboarding
- **Common challenges**: Knowledge lives in people's heads, documentation becomes outdated, information scattered
- **Azure DevOps Wikis**: Git-backed documentation with Markdown, Mermaid diagrams, and access control
- **Microsoft Teams integration**: Receive DevOps notifications, approvals, and discussions in unified platform
- **Documentation strategies**: "Just enough" documentation, ownership, proximity to code, templates, automation

## Why Knowledge Sharing Matters

| Benefit | Description | Impact |
|---------|-------------|--------|
| **Preserve Learning** | Prevent knowledge loss when people leave | Retain competitive advantage |
| **Reduce Mistakes** | Captured lessons from incidents/bugs | Avoid repeating failures |
| **Accelerate Onboarding** | Clear documentation explains systems | New members productive faster |
| **Support Decisions** | Documented rationale helps consistent choices | Better architecture decisions |
| **Enable Async Collaboration** | Written docs support distributed teams | Work across time zones |

## Common Knowledge Sharing Challenges

**Knowledge Lives in People's Heads**:
- Critical information remains undocumented
- Accessible only through verbal questions
- Risk of loss when people leave

**Documentation Becomes Outdated**:
- Initial docs created but not maintained
- Systems evolve without doc updates
- Teams lose trust in documentation

**Information Scattered**:
- Fragments across emails, chat, wikis, tickets
- Difficult to find when needed
- No single source of truth

**No Clear Ownership**:
- Without accountability, gaps persist
- Quality degrades over time
- Nobody responsible for updates

## Module Coverage

### Knowledge Sharing Strategies
- What knowledge to document (architecture, standards, setup, problems, processes)
- When to document (pain-driven, not comprehensive upfront)
- How to maintain documentation (ownership, close to code, templates)

### Azure DevOps Wikis
- Create project wikis (dedicated repository)
- Publish code as wiki (existing folder)
- Configure permissions (Contributors, readers, administrators)
- Organize structure (pages, sub-pages, navigation)

### Markdown & Documentation
- Markdown syntax (headings, lists, links, code blocks, tables)
- GitHub Flavored Markdown (GFM) extensions (task lists, strikethrough)
- File attachments and multimedia
- Code syntax highlighting

### Mermaid Diagrams
- Flowcharts (processes, workflows)
- Sequence diagrams (system interactions)
- Class diagrams (structure, relationships)
- ER diagrams (data models)
- Gantt charts (timelines)

### Microsoft Teams Integration
- **GitHub + Teams**: PR/issue notifications, workflow approvals, issue creation
- **Azure Boards + Teams**: Work item monitoring, subscriptions, create items
- **Azure Repos + Teams**: Repository monitoring, PR notifications, discussions
- **Azure Pipelines + Teams**: Pipeline monitoring, deployment approvals, run notifications

## Learning Objectives

After completing this module, you'll be able to:
1. Implement documentation strategies to preserve team knowledge
2. Create and manage Azure DevOps project wikis
3. Write technical documentation using Markdown and GFM
4. Create diagrams using Mermaid syntax
5. Integrate GitHub with Microsoft Teams
6. Integrate Azure DevOps with Microsoft Teams
7. Configure notifications and subscriptions for collaboration

## Prerequisites

**Recommended Experience**:
- DevOps fundamentals: Software development and operations practices
- Azure DevOps or GitHub: Repositories, work items, or issues
- Team collaboration tools: Microsoft Teams or Slack

## Critical Notes
- ⚠️ **Documentation without maintenance**: Creates tech debt and lost trust
- 💡 **Start small**: "Just enough" docs, grow based on pain points
- 🎯 **Assign ownership**: Include docs in definition of done
- 📊 **Tool selection**: Wikis (lasting docs), work items (context), code comments (implementation), Teams (real-time)

## Quick Reference

**Documentation Decision Tree**:
```yaml
Lasting documentation (processes, architecture)?
  → Azure DevOps Wiki or GitHub Wiki

Feature context and requirements?
  → Azure Boards work items or GitHub Issues

Implementation details and code rationale?
  → Code comments and docstrings

Real-time discussion and questions?
  → Microsoft Teams channels

Architecture decisions?
  → Architecture Decision Records (ADRs) in Wiki
```

**Common Knowledge Types**:
| Knowledge Type | Where to Document | Update Frequency |
|----------------|-------------------|------------------|
| Architecture decisions | Wiki (ADRs) | When decisions made |
| Coding standards | Wiki + linting rules | Quarterly review |
| Setup guides | Wiki or README | When process changes |
| Troubleshooting | Wiki (FAQ, runbooks) | After incidents |
| Processes | Wiki with flowcharts | Quarterly review |
| System knowledge | Wiki with diagrams | When system changes |

[Learn More](https://learn.microsoft.com/en-us/training/modules/share-knowledge-within-teams/1-introduction)
