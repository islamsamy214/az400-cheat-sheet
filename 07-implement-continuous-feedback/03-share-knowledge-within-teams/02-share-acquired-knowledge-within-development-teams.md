# Share Acquired Knowledge Within Development Teams

## Key Concepts
- **Cost of lost knowledge**: Repeated mistakes, slower problem resolution, inconsistent practices, extended onboarding
- **What to document**: Architecture decisions, coding standards, setup/configuration, common problems, processes, system knowledge
- **Documentation strategies**: Start with "just enough", make discoverable, assign ownership, keep close to code, use templates, automate
- **Tools for sharing**: Azure DevOps Wikis, GitHub Wikis/READMEs, Azure Boards, code comments, Microsoft Teams

## The Cost of Lost Knowledge

| Impact | Consequence | Example |
|--------|-------------|---------|
| **Repeated Mistakes** | Same bugs, identical architecture errors | Team debugs same DB config issue twice (6 months apart) |
| **Slower Resolution** | Rediscovering solutions to solved problems | 3 days debugging without documented troubleshooting guide |
| **Inconsistent Practices** | Each developer creates own approach | Different testing, code style, deployment per developer |
| **Extended Onboarding** | Months to become productive | Knowledge exists only in conversations/experience |
| **Expensive Context Switching** | Work stops when expert unavailable | One person understands system, work blocked when out |

## What Knowledge to Document

### Architecture Decisions
**Why**: Future developers need rationale behind technology choices
**Examples**:
- Microservices vs. monolith
- Database selection criteria
- Authentication approach
- API design principles

**Format**: Architecture Decision Records (ADRs)
- Context: What problem needs solving
- Decision: What was decided
- Consequences: Trade-offs and implications

### Coding Standards and Conventions
**Why**: Consistent code is easier to read, review, maintain
**Examples**:
- Naming conventions
- File organization
- Error handling patterns
- Testing requirements
- Code review checklist

**Format**: Style guides with examples, linting rules, PR templates

### Setup and Configuration
**Why**: Developers and ops need consistent environments
**Examples**:
- Development environment setup
- Local testing procedures
- Deployment configuration
- Tool installation guides

**Format**: Step-by-step guides with commands, screenshots, troubleshooting

### Common Problems and Solutions
**Why**: Teams repeatedly encounter similar issues
**Examples**:
- Known bugs and workarounds
- Performance optimization techniques
- Integration gotchas
- Deployment issues

**Format**: Troubleshooting guides, FAQ pages, runbooks

### Process and Workflows
**Why**: Clear processes enable consistent, efficient work
**Examples**:
- Code submission for review
- Deployment approval process
- Incident response procedures
- Release checklist

**Format**: Flowcharts, checklists, process documentation

### System Knowledge
**Why**: Understanding behavior helps modification and troubleshooting
**Examples**:
- Authentication flow
- Data flow through system
- Integration points
- Dependency maps

**Format**: Diagrams, sequence flows, component overviews

## Documentation Strategies That Work

### Start with "Just Enough"
```yaml
Approach:
  - Document when you feel pain from missing information
  - Don't create comprehensive docs upfront
  - Begin with simple README files
  - Grow documentation as questions arise
  - Focus on 20% of docs that answer 80% of questions
```

### Make Documentation Discoverable
```yaml
Strategies:
  - Use consistent locations (team knows where to find)
  - Create clear navigation and search
  - Link from code comments to detailed docs
  - Include documentation in onboarding checklists
```

### Assign Ownership
```yaml
Practices:
  - Designate documentation owners for each area
  - Include doc updates in definition of done
  - Review documentation currency in retrospectives
  - Make documentation part of code review checklist
```

### Keep Documentation Close to Code
```yaml
Methods:
  - Store docs in same repository as code when possible
  - Use README files for component-specific docs
  - Reference wiki from code comments for details
  - Publish code as wiki when appropriate
```

### Use Templates
```yaml
Benefits:
  - Create templates for common document types:
    * Architecture Decision Records (ADRs)
    * Troubleshooting guides
    * Setup instructions
  - Templates ensure consistency
  - Reduce friction in documentation creation
  - Include prompts for essential information
```

### Automate Where Possible
```yaml
Automation Examples:
  - Generate API docs from code comments
  - Auto-generate dependency graphs
  - Extract configuration documentation from code
  - Use tools that update docs as code changes
```

## Tools for Knowledge Sharing

| Tool | Purpose | Best For | Features |
|------|---------|----------|----------|
| **Azure DevOps Wikis** | Structured documentation | Project overviews, architecture, processes, onboarding | Markdown, Mermaid diagrams, attachments, permissions |
| **GitHub Wikis/READMEs** | Repository documentation | Project docs, API guides, contribution guidelines | Git-backed, Markdown, easy editing |
| **Azure Boards Work Items** | Capture decisions with work | Feature requirements, bug details, research findings | Rich text, attachments, links to code |
| **Code Comments/Docstrings** | Document why & non-obvious details | Complex algorithms, business logic, integration points | Lives with code, visible during development |
| **Microsoft Teams** | Real-time communication | Questions, discussions, alerts, coordination | Threading, file sharing, DevOps integration |

## Decision: Tool Selection

**Use Azure DevOps Wikis for**:
- Lasting documentation (architecture, processes)
- Cross-cutting project information
- Structured knowledge with navigation

**Use Azure Boards Work Items for**:
- Feature requirements and context
- Bug details and research
- Decisions tied to specific work

**Use Code Comments for**:
- Implementation details
- Why code exists (not what it does)
- Complex logic explanations

**Use Microsoft Teams for**:
- Real-time questions
- Quick coordination
- Transient knowledge sharing
- Integration with DevOps events

## Reflection Questions

### Current State
- Which knowledge-sharing tools does your team currently use?
- Where does critical team knowledge currently reside?
- How do new team members learn your systems and practices?
- When couldn't your team solve a problem because the knowledgeable person was unavailable?

### Pain Points
- What information do team members repeatedly ask about?
- Where do you waste time searching for information?
- What knowledge would be catastrophic to lose if specific people left?
- How current and accurate is your existing documentation?

### Improvement Opportunities
- What documentation would provide the most value if created today?
- Which repetitive questions could be answered with documentation?
- What knowledge sharing tools would reduce your team's friction?
- How could you make documentation creation easier and more consistent?

## Critical Notes
- ⚠️ **Don't document everything upfront**: Start when you feel pain
- 💡 **20/80 rule**: 20% of docs answer 80% of questions
- 🎯 **Ownership is key**: Without accountability, docs decay
- 📊 **Close to code**: README files + code comments + wiki references

## Quick Commands

**Example README Structure**:
```markdown
# Project Name

## Overview
Brief description of what this project does

## Getting Started
### Prerequisites
### Installation
### Configuration

## Usage
Common use cases and examples

## Architecture
High-level architecture overview (link to wiki for details)

## Troubleshooting
Common issues and solutions

## Contributing
How to contribute (link to full guide)
```

**Architecture Decision Record (ADR) Template**:
```markdown
# ADR-001: Use Microservices Architecture

## Status
Accepted

## Context
We need to scale different components independently. Monolith deployment is slow and risky.

## Decision
We will adopt a microservices architecture with domain-driven design boundaries.

## Consequences
**Positive**: Independent scaling, faster deployments, team autonomy
**Negative**: Increased complexity, distributed system challenges, operational overhead
**Risks**: Service discovery, data consistency, observability requirements
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/share-knowledge-within-teams/2-share-acquired-development-teams)
