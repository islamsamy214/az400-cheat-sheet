# Examine Code Quality

## Why Measure Code Quality
Code quality shouldn't be based on personal opinions. Individual developers rate their own code highly, but that's unreliable. Different teams use different definitions based on what they're building (automotive software vs. web applications require different quality standards).

## Key Quality Insight
- Individual programmers find less than 50% of bugs in their own software
- Most testing forms only find 35% of bugs
- Code quality affects overall software quality

## Five Key Quality Traits

### 1. Reliability
Measures how likely a system is to run without failure over time.

**Key Metrics**:
- Defect count (measured via static analysis)
- Mean Time Between Failures (MTBF)
- Software availability

**Goal**: Low defect counts are crucial for reliable code

### 2. Maintainability
Measures how easily software can be maintained.

**Related Factors**:
- Code size, consistency, structure, complexity
- Testability and understandability

**Metrics to Consider**:
- Number of style warnings
- Halstead complexity measures

**Approach**: Requires both automation AND human reviewers - no single metric ensures maintainability

### 3. Testability
Measures how well software supports testing efforts.

**Depends On**:
- Control, observability, isolation, automation capabilities
- Number of test cases needed to find potential faults

**Impact**: Size and complexity directly affect testability

**Method**: Apply cyclomatic complexity at code level to improve component testability

### 4. Portability
Measures how usable software is in different environments (platform independence).

**Best Practices**:
- Regularly test code on different platforms (don't wait until end)
- Set compiler warning levels as high as possible
- Use at least two compilers
- Follow coding standards

**Note**: No specific metric exists - preventive practices are key

### 5. Reusability
Measures whether existing assets (like code) can be used again.

**Characteristics for Reusability**:
- Modularity
- Loose coupling

**Metric**: Number of interdependencies

**Tool**: Static analyzer can identify interdependencies

## Quality Measurement Strategy

| Trait | Primary Metric | Measurement Tool | Goal |
|-------|---------------|------------------|------|
| Reliability | Defect count, MTBF | Static analysis | Low defects |
| Maintainability | Style warnings, Halstead | Automation + Reviews | Consistent, clean code |
| Testability | Test case count, complexity | Cyclomatic complexity | Easy testing |
| Portability | N/A | Cross-platform testing | Platform independence |
| Reusability | Interdependency count | Static analyzer | Modular, loosely coupled |

## Critical Notes
- 🎯 Code quality is objective, not subjective - use measurable metrics
- 💡 Different project types require different quality standards
- ⚠️ Single developers find less than 50% of their own bugs - peer review is essential
- 📊 No single metric ensures maintainability - use multiple measures
- 🔄 Both automated tools and human reviewers are necessary for quality
- ✨ Static analysis tools teach developers by identifying rule violations

[Learn More](https://learn.microsoft.com/en-us/training/modules/identify-technical-debt/2-examine-code-quality)
