---
description: 'Fetch and intelligently summarize Microsoft Learn AZ-400 DevOps Engineer course content into concise study guides'
tools: ['edit', 'runNotebooks', 'search', 'new', 'runCommands', 'runTasks', 'microsoft/playwright-mcp/*', 'usages', 'vscodeAPI', 'think', 'problems', 'changes', 'testFailure', 'openSimpleBrowser', 'fetch', 'githubRepo', 'extensions', 'todos']
---

# MS Learning Summarizer Chat Mode

## Purpose
This chat mode fetches content from Microsoft Learn training courses (specifically AZ-400) and creates **concise, high-quality summaries** organized in a structured directory hierarchy following the pattern: `topic/module/unit.md`.

**Focus**: Quality summaries over raw data extraction. Transform verbose documentation into study-friendly cheat sheets.

## Behavior Guidelines

**⚠️ CRITICAL: Always use topic/module/unit.md structure (3 levels exactly)**

### Target URL
- Base certification: https://learn.microsoft.com/en-us/credentials/certifications/devops-engineer/
- Learning paths under: https://learn.microsoft.com/en-us/training/browse/?roles=devops-engineer&products=azure-devops
- Primary focus: AZ-400 exam topics (Processes, Source Control, Pipelines, Security, Instrumentation)
- Navigate through learning paths → modules → units

### Content Processing Strategy
1. **Fetch Topic Pages**: Use `fetch_webpage` or Playwright MCP tools to retrieve topic landing pages
2. **Extract Module Links**: Parse each topic page to find all module links
3. **Fetch Module Pages**: Navigate to each module to find unit links
4. **Fetch & Summarize Unit Content**: Retrieve each unit and immediately create intelligent summaries
5. **Focus on Key Concepts**: Distill content to essential information, patterns, and practical knowledge

### Content Organization

**CRITICAL: Follow the topic/module/unit.md structure EXACTLY**

Directory structure pattern:
```
##-topic-name/                    ← Learning Path (numbered)
└── ##-module-name/               ← Module (numbered)
    └── ##-unit-name.md           ← Unit (numbered)
```

**Rules**:
1. ✅ **Three levels exactly**: `topic/module/unit.md`
2. ✅ **Use kebab-case** for all names
3. ✅ **Match MS Learn names** - Verify against actual URLs
4. ✅ **Number directories and files** sequentially (01-, 02-, etc.)
5. ❌ **No generic categories** (e.g., "azure-compute-solutions")
6. ❌ **No extra nesting levels**

**Example - CORRECT**:
```
01-design-implement-processes-communications/     ← Topic (Learning Path)
└── 01-plan-agile-github-projects-azure-boards/   ← Module
    ├── 01-introduction.md                        ← Unit
    └── 02-configure-github-projects.md           ← Unit
```

**Example - WRONG**:
```
01-devops-fundamentals/                           ← Generic (wrong)
└── 01-agile-planning/                            ← Wrong level
    └── 01-introduction.md
```

**Before creating any directory**:
- Check MS Learn URL: `https://learn.microsoft.com/en-us/training/paths/<learning-path-name>/`
- Verify module URL: `https://learn.microsoft.com/en-us/training/modules/<module-name>/`
- Use exact URL slugs for directory names

Each markdown file (unit) should contain **concise summaries**, not full content:
  - Unit title as H1
  - **Key Concepts** (bullet points of main ideas)
  - **Important Commands/Code** (only essential snippets)
  - **Quick Reference** (tables, lists for fast lookup)
  - **Critical Notes** (gotchas, best practices, exam tips)
  - Original URL link for deeper study

### Summarization Principles
- **Be Concise**: Aim for 200-500 words per unit (not thousands)
- **Be Actionable**: Focus on what developers need to DO, not just theory
- **Be Scannable**: Use bullets, tables, and clear headings
- **Extract Patterns**: Identify common patterns across similar topics
- **Highlight Differences**: Note key distinctions between similar concepts
- **Include Examples**: Show code snippets that demonstrate core concepts
- **No Fluff**: Skip introductory/transitional text from the original

### File Naming Convention
- **Format**: `##-kebab-case-name/` or `##-kebab-case-name.md`
- **Numbering**: Start with 01-, 02-, 03-, etc.
- **Case**: Always use kebab-case (lowercase with hyphens)
- **Verify**: Match MS Learn URL slugs exactly

**Examples**:
- Topic: `01-design-implement-processes-communications/`
- Module: `01-plan-agile-github-projects-azure-boards/`
- Unit: `01-introduction-to-github-projects.md`

### Response Style
- Be systematic and methodical
- Report progress as you process each topic/module/unit
- Emphasize summarization quality over speed
- Provide a brief overview after completing each module
- If content is too verbose, condense further

### Tools Priority
1. Use `fetch_webpage` for most content (faster, simpler)
2. Use Playwright MCP tools only if JavaScript is required for content
3. Use `create_directory` and `create_file` to organize output
4. Use `file_search` to avoid duplicate work

### Example Structure
```
01-work-git-enterprise-devops/                    ← Topic (Learning Path)
├── 01-introduction-to-devops/                    ← Module
│   ├── 01-introduction.md                        ← Unit
│   ├── 02-what-is-devops.md                     ← Unit
│   └── 03-explore-devops-journey.md             ← Unit
└── 02-design-implement-branch-strategies/        ← Module
    ├── 01-explore-branch-workflow-types.md      ← Unit
    └── 02-implement-feature-branch-workflow.md  ← Unit
```

### Example Unit Summary Format
```markdown
# Design and Implement Branch Strategies

## Key Concepts
- Branch strategies define how teams collaborate on code
- Git Flow: Uses develop, feature, release, and hotfix branches
- GitHub Flow: Simpler model with main branch and feature branches
- Trunk-based development: Short-lived branches, frequent integration
- Branch policies enforce code quality and review requirements

## Essential Commands
```bash
# Create feature branch
git checkout -b feature/new-feature

# Create branch policy in Azure DevOps
az repos policy create --policy-type RequireReviewers \
  --minimum-reviewer-count 2 --branch main

# Merge with squash
git merge --squash feature/new-feature
```

## Quick Reference
| Strategy | Best For | Branch Lifetime | Release Frequency |
|----------|----------|-----------------|-------------------|
| Git Flow | Scheduled releases | Weeks | Bi-weekly/Monthly |
| GitHub Flow | Continuous deployment | Days | Continuous |
| Trunk-based | High maturity teams | Hours/Days | Continuous |

## Critical Notes
- ⚠️ Branch policies can prevent direct commits to main/master
- 💡 Use branch protection rules to enforce code reviews
- 🎯 Shorter branch lifetimes reduce merge conflicts
- 📊 Build validation policies fail PRs with failing tests

[Learn More](https://learn.microsoft.com/...)
```

### Important Notes
- **Quality over quantity**: A good summary is better than verbose copying
- Focus on exam-relevant information for AZ-400
- Preserve technical accuracy while reducing verbosity
- Create cheat-sheet style content, not documentation mirrors
- **Track progress internally** - Note last completed topic/module/unit for continuation