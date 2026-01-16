# Understand the Inner Loop

## Key Concepts
- **Inner loop**: Rapid feedback cycle during local development (code → build → test → debug)
- Runs entirely on developer's workstation before code enters version control
- Goal: Keep loop as short as possible for maximum productivity
- Typical execution: Many times per day during active development

## Inner Loop Activities Categorization
| Category | Purpose | Examples |
|----------|---------|----------|
| **Experimentation** | Add customer value | Coding, designing, prototyping |
| **Feedback Collection** | Verify quality | Building, testing, debugging, linting |
| **Tax** | Necessary but no value | Committing, configuration, documentation |

## Inner Loop Optimization Principles
1. **Speed ∝ Change Size**: Execution time proportional to changes made
2. **Maximize Feedback Quality, Minimize Time**: Balance comprehensive testing with rapid iteration
3. **Minimize/Defer Tax**: Reduce unnecessary overhead
4. **Combat Complexity Growth**: Prevent slowdowns as codebase grows

## Strategies for Large Codebase Optimization
```bash
# 1. Incremental Builds
# Only rebuild changed components
bazel build //changed:targets

# 2. Distributed Caching
# Share build artifacts across team
--remote_cache=grpc://build-cache.company.com

# 3. Test Selection
# Run only affected tests
jest --onlyChanged
```

## Tangled Loops Problem
**Scenario**: Modularizing framework into separate repo creates outer loop overhead

```
Bad: Framework change → PR review → Build → Publish → Update app → Test
Good: Monorepo or stable interfaces → Rare framework changes
```

**Warning Signs**:
- ⚠️ Every feature requires framework changes
- ⚠️ Tight coupling between components
- ⚠️ Local workarounds (symlinks, git submodules)

## Inner vs. Outer Loop
| Aspect | Inner Loop | Outer Loop |
|--------|-----------|------------|
| Location | Local machine | CI/CD pipeline |
| Speed | Seconds to minutes | Minutes to hours |
| Tax | Low | High (reviews, scans, approvals) |
| Feedback | Fast, individual | Slow, team-wide |
| Purpose | Development velocity | Quality gates, integration |

## Best Practices
- 📊 **Monitor metrics**: Build time, test execution time, feedback delay
- 🎯 **Address slowdowns proactively**: Don't let builds slow gradually
- ⚖️ **Balance trade-offs**: Faster builds vs. configuration complexity
- 🤝 **Team alignment**: Make inner loop optimization a shared priority
- 🔄 **Continuous improvement**: Treat optimization as ongoing work

## Critical Notes
- 💡 No silver bullet: Every optimization has trade-offs
- ⚠️ Modularize carefully: Split based on technical boundaries, not convenience
- 🎯 Measure everything: Track before/after to validate improvements
- 📈 Context switching kills productivity: Keep builds under 5 minutes

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-tools-track-usage-flow/2-understand-inner-loop)
