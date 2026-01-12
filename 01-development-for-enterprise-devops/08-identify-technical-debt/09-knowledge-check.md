# Knowledge Check

## Assessment Overview
This knowledge check tests your understanding of technical debt identification and management covered in this module.

## Key Topics to Review

### 1. Code Quality Metrics
Review the five key quality traits:
- **Reliability**: Defect counts, MTBF
- **Maintainability**: Style warnings, Halstead complexity
- **Testability**: Test case requirements, cyclomatic complexity
- **Portability**: Cross-platform compatibility
- **Reusability**: Interdependency counts

### 2. Complexity Metrics
Understand how to measure code complexity:
- **Cyclomatic complexity**: Number of paths through code
- **Halstead measures**: Vocabulary, length, volume, difficulty, effort
- **Quality-related metrics**: Failed builds, bug bounce rate, unplanned work

### 3. Technical Debt Definition and Impact
Remember the key concepts:
- **Definition**: Future cost from choosing easy solution today
- **Causes**: Tight deadlines, lack of standards, poor design
- **Impact**: Reduced productivity, increased support costs
- **Snowball effect**: Compounds like financial debt

### 4. Measurement and Management
Know how to measure and manage technical debt:
- **Automated tools**: SonarCloud, SonarQube, GitHub Advanced Security
- **Quality gates**: Minimum standards for code to pass
- **Prioritization**: Critical (immediate) → High (this sprint) → Medium (next sprint)
- **Time allocation**: 20% of sprint capacity for debt reduction

### 5. GitHub Advanced Security (GHAS)
Understand the three main components:
- **CodeQL analysis**: Find code problems and security issues
- **Dependency scanning**: Identify outdated/risky dependencies
- **Code scanning**: Catch vulnerabilities before production

### 6. Code Quality Tools
Know different tools and their purposes:
- **NDepend**: Real-time tracking, custom C# LINQ rules
- **ReSharper**: Automated quality checks, build failure
- **SonarCloud**: Multi-language, cloud-based
- **Tool selection**: Consider language, team size, budget

### 7. Effective Code Reviews
Remember best practices:
- **Keep small**: 2-3 reviewers
- **Focus on learning**: Not blame
- **Balance feedback**: Positive + questions + suggestions
- **Common goals**: Readability, consistency, performance, security, testing

## Sample Questions

### Question 1: Code Quality Traits
**Which metric best measures code maintainability?**
- A) Mean Time Between Failures (MTBF)
- B) Cyclomatic complexity and style warnings
- C) Number of test cases
- D) Cross-platform compatibility

**Answer**: B - Maintainability is measured by complexity and style warnings

### Question 2: Technical Debt
**What is the primary reason technical debt occurs?**
- A) Lack of developer skill
- B) Tight deadlines leading to shortcuts
- C) Absence of code standards
- D) Poor technology choices

**Answer**: B - Tight deadlines are the most common cause

### Question 3: Quality Metrics
**What does a bug bounce percentage over 10% indicate?**
- A) Too many new bugs
- B) Builds are failing frequently
- C) Bug fixes aren't thorough enough
- D) Too much unplanned work

**Answer**: C - High bounce rate means tickets are reopened due to incomplete fixes

### Question 4: GitHub Advanced Security
**Which GHAS component automatically creates PRs to update vulnerable dependencies?**
- A) CodeQL
- B) Secret scanning
- C) Dependabot
- D) Code scanning

**Answer**: C - Dependabot automates dependency updates

### Question 5: SonarCloud Integration
**What percentage of sprint capacity should be reserved for technical debt reduction?**
- A) 5%
- B) 10%
- C) 20%
- D) 30%

**Answer**: C - 20% is the recommended allocation

### Question 6: Code Quality Tools
**Which tool allows writing custom quality rules using C# LINQ queries?**
- A) ReSharper
- B) NDepend
- C) SonarQube
- D) CodeQL

**Answer**: B - NDepend supports C# LINQ for custom rules

### Question 7: Code Reviews
**What is the ideal number of reviewers for effective code reviews?**
- A) 1 reviewer
- B) 2-3 reviewers
- C) 4-5 reviewers
- D) Entire team

**Answer**: B - 2-3 reviewers provide multiple perspectives without overwhelming

### Question 8: Complexity Metrics
**What does cyclomatic complexity measure?**
- A) Number of lines of code
- B) Number of different paths through code
- C) Number of dependencies
- D) Code execution time

**Answer**: B - Cyclomatic complexity counts code paths

## Study Tips
- Focus on practical application of tools (SonarCloud, GHAS, NDepend)
- Understand the relationship between technical debt and project delays
- Know how to prioritize technical debt (Critical → High → Medium → Low)
- Remember that quality and speed improve together in DevOps
- Understand code review best practices (small groups, learning focus)

## Critical Notes
- 🎯 Know the five key quality traits: Reliability, Maintainability, Testability, Portability, Reusability
- 💡 Technical debt compounds like financial debt over time
- ⚠️ Cyclomatic complexity measures paths through code, not lines
- 📊 20% of sprint capacity should be reserved for debt reduction
- 🔄 Dependabot automates dependency updates in GHAS
- ✨ Code reviews should focus on learning, not blame

[Learn More](https://learn.microsoft.com/en-us/training/modules/identify-technical-debt/9-knowledge-check)
