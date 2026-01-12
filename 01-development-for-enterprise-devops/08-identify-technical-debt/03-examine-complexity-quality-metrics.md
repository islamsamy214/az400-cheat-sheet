# Examine Complexity and Quality Metrics

## DevOps Promise: Speed AND Quality
Historically, faster delivery meant lower quality, and higher quality took longer. DevOps processes break this trade-off by finding problems earlier, which usually means less time to fix them.

## Complexity Metrics

### Cyclomatic Complexity
Measures the number of different paths through a program's source code.

**Use**: Indicates code complexity and difficulty of testing
**Impact**: Higher complexity = more potential bugs and harder maintenance

### Halstead Complexity Measures
Analyzes multiple dimensions of code complexity:
- **Program vocabulary**: Unique operators and operands
- **Program length**: Total operators and operands
- **Calculated program length**: Theoretical minimum length
- **Volume**: Size based on vocabulary and length
- **Difficulty**: How hard the code is to write or understand
- **Effort**: Time/energy required to develop or maintain

## Common Quality-Related Metrics

### Build & Deployment Quality
| Metric | What It Measures | Target |
|--------|------------------|--------|
| **Failed builds percentage** | Overall percentage of builds failing | < 5% |
| **Failed deployments percentage** | Overall percentage of deployments failing | < 3% |

### Support & Maintenance Quality
| Metric | What It Measures | Target |
|--------|------------------|--------|
| **Ticket volume** | Overall volume of customer or bug tickets | Trending down |
| **Bug bounce percentage** | Percentage of customer/bug tickets reopened | < 10% |
| **Unplanned work percentage** | Percentage of overall work that is unplanned | < 20% |

## Code Analysis Tools
Should be part of every developer's toolbox and software build process.

**Check For**:
- Security vulnerabilities
- Performance issues
- Compatibility problems
- Language usage best practices
- Globalization support

**Benefits**:
- Automated quality checks
- Consistent enforcement of standards
- Educational feedback (teaches developers through rule explanations)
- Early problem detection

## Static Code Analysis Benefits
Running static analysis tools regularly helps developers:
- Learn new techniques and patterns
- Identify issues before code review
- Improve code quality proactively
- Understand security and performance implications

## Quality Metrics Strategy

### Leading Indicators (Predict Future Problems)
- Cyclomatic complexity
- Halstead measures
- Code analysis warnings
- Static analysis findings

### Lagging Indicators (Show Past Problems)
- Failed build percentage
- Failed deployment percentage
- Bug bounce rate
- Unplanned work percentage

## Critical Notes
- 🎯 DevOps finds problems earlier, making fixes faster and cheaper
- 💡 Higher cyclomatic complexity = exponentially more testing needed
- ⚠️ Unplanned work over 20% indicates serious quality issues
- 📊 Bug bounce percentage reveals whether fixes are thorough enough
- 🔄 Static analysis teaches developers while enforcing standards
- ✨ Regularly running code analysis improves developer skills over time

[Learn More](https://learn.microsoft.com/en-us/training/modules/identify-technical-debt/3-examine-complexity-quality-metrics)
