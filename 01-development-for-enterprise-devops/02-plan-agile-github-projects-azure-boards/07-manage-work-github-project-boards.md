# Manage Work with GitHub Project Boards

## Iteration Planning Framework

### Strategic Iteration Design Principles
**Create iterations to**:
- Associate items with time-boxed delivery cycles for predictable release cadence
- Set optimal duration based on team capacity and complexity requirements
- Include strategic breaks for planning, retrospectives, and team development
- Align with business milestones and customer delivery expectations

### Iteration Length Selection Guide

| Duration | Best For | Benefits | Considerations |
|----------|----------|----------|----------------|
| **1 week** | Fast-moving features, bug fixes | Rapid feedback, quick adjustments | Limited scope, frequent overhead |
| **2 weeks** | Standard agile teams, balanced workload | Good velocity tracking, manageable | Industry standard, proven approach |
| **3 weeks** | Complex features, research work | Deep focus, substantial deliverables | Risk of scope creep |
| **4 weeks** | Large initiatives, cross-team coordination | Strategic planning, major milestones | Reduced agility, delayed feedback |

⚠️ **Note**: When you first create an iteration field, **three iterations are automatically created** as a foundation.

## Advanced Iteration Field Configuration

### Creating Strategic Iteration Fields

**Command Palette Method** (Recommended) ⚡:
1. Open command palette: `Ctrl+K` (Windows/Linux) or `Command+K` (Mac)
2. Start typing "Create new field"
3. When "Create new field" displays, select it

**Interface Method** (Detailed Configuration):
1. Navigate to your project
2. Click plus (+) sign in rightmost field header
3. Select "New field" from dropdown
4. Configure iteration field strategically

### Strategic Iteration Field Configuration

**Strategic Naming**:
- Use clear, business-aligned names: "Sprint", "Release Cycle", "Development Phase"
- Include version/timeline indicators: "Q1 2024 Sprints", "Version 2.1 Cycles"
- Consider team understanding and adoption

**Configuration Options**:
1. **Name selection**: Reflect business context and team workflow
2. **Field type**: Select "Iteration" for time-boxed planning
3. **Start date strategy**:
   - Current day: For immediate project initiation
   - Strategic date: Align with business cycles, team availability, major releases
4. **Duration optimization**:
   - Standard teams: 2 weeks (14 days) for balanced planning/delivery
   - Research teams: 3-4 weeks for deeper investigation cycles
   - Maintenance teams: 1 week for rapid response and fixes
5. Click "Save and create"

### Enterprise Iteration Naming Conventions
```
Format: [Project]_[Year]_[Type]_[Number]
Examples:
- CustomerPortal_2024_Sprint_01
- API_2024_Release_Q1
- Mobile_2024_Feature_Phase2
```

## Strategic Iteration Management & Scaling

### Adding and Optimizing Iterations

**Systematic Iteration Planning**:
1. Navigate to project
2. Click settings menu (three dots) in top-right
3. Select "Settings" to access project configuration
4. Click iteration field name you want to enhance
5. Click "Add iteration" for standard duration cycles

**Advanced Iteration Customization**:
1. Click dropdown next to "Add iteration"
2. Strategic start date selection: Align with business quarters, team availability, dependency completion
3. Dynamic duration management: Adjust based on scope complexity and team capacity
4. Click "Add" to implement

### Strategic Break Planning & Team Development

**Iteration Break Best Practices**:

**Strategic Break Types**:
- **Planning sessions**: Requirements gathering, architecture design, sprint planning
- **Team development**: Training, conferences, skill development, team building
- **Process improvement**: Retrospectives, process optimization, tool evaluation
- **Maintenance windows**: Infrastructure updates, security patches, technical debt
- **Holiday periods**: Planned vacation time, company holidays, team recharge

**Break Duration Guidelines**:
```
- Planning breaks: 1-2 days between iterations
- Development breaks: 3-5 days quarterly
- Major maintenance: 1 week annually
- Holiday breaks: Variable based on team and region
```

### Enterprise-Scale Iteration Management

**Multi-Team Coordination Strategies**:

**Synchronized Iterations**:
- Align all teams to same iteration schedule for coordinated releases
- Shared planning and retrospective cycles
- Simplified dependency management and communication

**Staggered Iterations**:
- Offset team cycles to enable continuous integration and testing
- Reduced resource contention for shared services
- Improved deployment pipeline utilization

**Portfolio-Level Planning**:
- **Program increments**: 8-12 week cycles coordinating multiple teams
- **Release trains**: Coordinated delivery of integrated solutions
- **Milestone alignment**: Business-critical delivery dates and dependencies

### Iteration Health Monitoring
- **Velocity tracking**: Monitor story points or work items completed per iteration
- **Burndown analysis**: Track progress toward iteration goals and identify risks
- **Retrospective metrics**: Capture team satisfaction and process improvement opportunities
- **Capacity utilization**: Balance team workload and prevent burnout

## Reference Links
- [Managing iterations in projects - GitHub Docs](https://docs.github.com/issues/trying-out-the-new-projects-experience/managing-iterations)
- [Best practices for managing projects - GitHub Docs](https://docs.github.com/issues/trying-out-the-new-projects-experience/best-practices-for-managing-projects)

## Critical Notes
- 🎯 **2 weeks** is industry standard for iteration duration
- 💡 **Command palette** (`Ctrl+K` / `Command+K`) for fastest configuration
- ⚠️ **Strategic breaks** prevent burnout and improve planning
- 📊 **Synchronized iterations** simplify cross-team coordination
- 🔄 **Staggered iterations** enable continuous integration
- ✨ **3 iterations auto-created** when field is first created

[Learn More](https://learn.microsoft.com/en-us/training/modules/plan-agile-github-projects-azure-boards/7-manage-work-github-project-boards)
