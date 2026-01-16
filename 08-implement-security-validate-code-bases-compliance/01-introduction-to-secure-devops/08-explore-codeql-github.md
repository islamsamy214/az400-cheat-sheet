# Explore CodeQL in GitHub

## Key Concepts
- **CodeQL**: Semantic code analysis engine that treats code as queryable database
- **Semantic Analysis**: Understands code structure/meaning, not just pattern matching
- **Variant Analysis**: Find similar vulnerabilities across entire codebase
- **Data Flow Tracking**: Tracks data from sources (user input) to sinks (sensitive operations)

## What is CodeQL

**Traditional vs CodeQL**:
- Traditional tools: Simple pattern matching → many false positives
- CodeQL: Semantic approach → understands context, relationships, data flow → accurate detection

**Treats Code as Database**:
- Converts source to database capturing syntax trees, control flow graphs, data flow paths
- Queryable using specialized query language
- Supports multiple languages (C/C++, C#, Java, JavaScript/TypeScript, Python, Ruby, Go, Swift)
- Analyzes entire codebases with interconnections

**Key Analysis Capabilities**:
- **Variant Analysis**: Find similar issues across entire codebase
- **Data Flow Analysis**: Track data movement from sources to sinks
- **Taint Tracking**: Identify untrusted data reaching sensitive operations
- **Control Flow Analysis**: Find vulnerabilities under specific conditions

## How CodeQL Works (3 Phases)

### Phase 1: Create CodeQL Database
```yaml
Code Extraction:  Analyze source files during compilation/static analysis
Database Creation: Build comprehensive database (AST, control flow, dependencies)
Metadata Capture:  Record locations, line numbers, scopes, calls, hierarchies
Optimization:     Index for efficient querying on large codebases
```
Created once, queried multiple times for efficient iterative analysis

### Phase 2: Run CodeQL Queries
```yaml
Standard Query Packs: OWASP Top 10, CWE standards (curated by GitHub)
Custom Queries:       Organization-specific patterns/standards
Query Execution:      Run against indexed database (not raw files)
```

**Query Categories**:
- Injection vulnerabilities (SQL, command, XSS)
- Authentication issues (weak passwords, missing checks, insecure sessions)
- Cryptography problems (weak algorithms, hard-coded credentials)
- Resource management (memory leaks, exhaustion)

### Phase 3: Interpret Results
```yaml
Result Ranking:       Prioritize by severity, confidence, exploitability
Contextual Info:      File locations, line numbers, code snippets, data flow paths
Remediation Guidance: Vulnerability explanations + fix recommendations
Integration:          GitHub Security tab, PR annotations, SARIF files
```

## CodeQL Query Language

**Object-Oriented Logic Programming**:
```codeql
import javascript

from SqlExecution sql, Source source
where source.flowsTo(sql.getAnArgument())
select sql, "SQL query vulnerable to injection from $@.", source, "user input"
```

**Query Features**:
- Classes and predicates for code elements (functions, variables, expressions)
- Declarative approach (describe what to find, not how)
- Pattern matching with predicates
- Composability (build complex queries from simpler ones)

**Standard Query Libraries**:
- Security queries (OWASP Top 10, CWE, language-specific issues)
- Code quality queries (code smells, maintainability, performance)
- Community contributions (thousands of queries from researchers)
- Regular updates from GitHub Security Lab

## CodeQL in GitHub Security

### Code Scanning
```yaml
Default Setup:       One-click enable in repo settings
Scheduled Scans:     Every push, PR, or schedule
Multi-Language:      Auto-detects languages, runs appropriate queries
Result Presentation: Security tab with detailed explanations
```

### Pull Request Integration
```yaml
Inline Annotations:  Comments on vulnerable code lines
Blocking Checks:     Required check before merge
Differential Scanning: Only reports new vulnerabilities (reduces noise)
Developer Feedback:  Immediate feedback while code fresh
```

### GitHub Advanced Security (Organizations)
- Private repository scanning
- Custom query execution
- Security overview dashboard across repos
- Alert management (triage, assign, track)

## CI/CD Pipeline Integration

### GitHub Actions
```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v2
  with:
    languages: javascript, python

- name: Perform CodeQL Analysis
  uses: github/codeql-action/analyze@v2
```

### Azure Pipelines
```bash
# Install CodeQL CLI
# Create database: codeql database create
# Analyze: codeql database analyze
# Export SARIF for Azure DevOps
```

### Other Platforms
- Jenkins (build steps)
- GitLab CI/CD (SARIF output)
- CircleCI (workflows)
- Any platform with CLI support

### Security Gates
- Fail builds on high-severity vulnerabilities
- Track security metrics over time
- Generate compliance evidence
- Trigger automated remediation workflows

## CodeQL Development Tools

### VS Code Extension
- Query development (syntax highlighting, autocomplete, docs)
- Local database analysis
- Result visualization with source navigation
- Debugging support

### CLI
```bash
codeql database create    # Extract code to queryable format
codeql database analyze   # Run queries, generate results
codeql test run          # Validate queries against test cases
codeql pack             # Download/manage query packs
```

## Benefits

| Category | Benefits |
|----------|----------|
| **Developer Productivity** | Shift-left security, faster remediation, learning opportunities, reduced context switching |
| **Accurate Results** | Low false positives, contextual info with data flow, prioritized findings, variant discovery |
| **Comprehensive Coverage** | Entire codebase (dependencies + legacy), multiple languages, consistent standards |
| **Scalable Security** | Automated scanning, continuous monitoring, security as code, knowledge sharing via queries |
| **Compliance** | Scan history, finding lifecycle tracking, policy enforcement, audit reports |

## Critical Notes
- ⚠️ Database created once, queried multiple times (efficient)
- 💡 Semantic analysis reduces false positives vs pattern matching
- 🎯 Differential scanning in PRs reduces noise (only new vulnerabilities)
- 📊 Query libraries capture institutional security knowledge

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-secure-devops/8-explore-codeql-github)
