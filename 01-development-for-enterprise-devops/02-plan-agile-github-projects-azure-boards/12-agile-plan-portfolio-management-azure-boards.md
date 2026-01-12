# Agile Plan and Portfolio Management with Azure Boards

## Lab Overview

**Duration**: 60 minutes

**Purpose**: Hands-on experience with Azure Boards agile planning and portfolio management tools for enterprise-scale DevOps teams.

## Learning Objectives

After completing this lab, you'll be able to:

| Objective | Skills Gained |
|-----------|---------------|
| **Manage teams, areas, and iterations** | Organizational structure configuration, sprint planning setup |
| **Manage work items** | Backlog management, work item creation, hierarchy management |
| **Manage sprints and capacity** | Sprint planning, capacity allocation, velocity tracking |
| **Customize Kanban boards** | Workflow optimization, column configuration, WIP limits |
| **Define dashboards** | Real-time metrics visualization, team performance monitoring |
| **Customize team process** | Workflow customization, field configuration, business rules |

## Lab Exercises Structure

### Exercise 0: Configure Lab Prerequisites
**Tasks**:
- Set up Azure DevOps organization (if not already available)
- Configure initial project settings
- Import sample data for realistic scenarios

**Requirements**:
- Microsoft Edge or Azure DevOps-supported browser
- Azure DevOps organization ([Create one](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization))

### Exercise 1: Manage an Agile Project
**Core Activities**:

**Team and Area Management**:
- Create team hierarchies for enterprise-scale organizations
- Configure area paths for component-based or feature-based organization
- Set up iteration paths for sprint cadences and release schedules

**Work Item Management**:
- Create and configure work items (Epics, Features, User Stories, Tasks, Bugs)
- Establish parent-child relationships in work item hierarchy
- Link work items to source code commits and pull requests

**Sprint Planning and Capacity**:
- Configure sprint schedules and iteration dates
- Allocate team capacity based on availability and skillsets
- Track velocity and burndown metrics for predictive planning

**Kanban Board Customization**:
- Configure columns to match team workflow stages
- Set Work-in-Progress (WIP) limits to prevent bottlenecks
- Customize card styling with tags, colors, and fields
- Create swimlanes for priority or work item type visualization

### Exercise 2 (Optional): Define Dashboards
**Dashboard Creation Activities**:

**Dashboard Widget Configuration**:
- Add velocity charts for sprint performance tracking
- Configure burndown/burnup widgets for progress visualization
- Display cumulative flow diagrams for workflow health
- Add query-based widgets for custom metrics

**Real-Time Monitoring**:
- Create executive dashboard for high-level portfolio view
- Build team dashboard for daily standup and sprint tracking
- Configure personal dashboard for individual work focus

## Key Concepts Covered in Lab

### Work Item Hierarchy

```
Epic (Strategic Initiative)
└── Feature (Business Capability)
    └── User Story (Customer Value)
        └── Task (Implementation Work)
```

### Azure Boards Core Components

| Component | Purpose | Use Case |
|-----------|---------|----------|
| **Product Backlog** | Prioritized list of all work | Long-term planning, roadmap visualization |
| **Sprint Backlog** | Work committed for current sprint | Sprint planning, daily standup reference |
| **Task Board** | Detailed task tracking | Individual developer workflow, standup updates |
| **Kanban Board** | Visual workflow management | Continuous flow, WIP limit enforcement |
| **Delivery Plans** | Multi-team coordination | Program management, dependency tracking |

### Sprint Management Best Practices

**Capacity Planning Formula**:
```
Team Capacity = (Team Size × Working Days × Hours per Day) - (Planned Time Off + Meeting Overhead)

Example:
- Team: 5 developers
- Sprint: 10 working days
- Hours/day: 6 productive hours (accounting for meetings)
- Time off: 2 developer-days
- Capacity: (5 × 10 × 6) - (2 × 6) = 300 - 12 = 288 hours
```

### Kanban Board Configuration

**WIP Limit Strategy**:

| Stage | Typical WIP Limit | Rationale |
|-------|-------------------|-----------|
| **To Do** | No limit | Backlog refinement flexibility |
| **In Progress** | 2-3 per developer | Focus, reduce context switching |
| **Code Review** | 50% of team size | Ensure timely reviews |
| **Testing** | Equal to team size | Prevent testing bottleneck |
| **Done** | No limit | Completed work archive |

## Dashboard Widgets for Monitoring

### Essential Dashboard Widgets

**Velocity Tracking**:
- **Velocity Chart**: Historical sprint completion trends for predictive planning
- **Forecast**: Projected completion dates based on current velocity

**Sprint Progress**:
- **Sprint Burndown**: Daily remaining work visualization
- **Sprint Capacity**: Team capacity vs. planned work comparison

**Workflow Health**:
- **Cumulative Flow Diagram**: Work item distribution across workflow stages
- **Lead Time/Cycle Time**: Efficiency metrics from start to completion

**Quality Metrics**:
- **Bug Trends**: Defect creation vs. resolution rates
- **Test Results**: Pass/fail rates and test coverage

## Hands-On Lab Access

**Lab Environment**: [Launch Interactive Lab](https://go.microsoft.com/fwlink/?linkid=2269557)

**Lab Features**:
- Pre-configured Azure DevOps project with sample data
- Step-by-step guided exercises with screenshots
- Real-world scenarios and best practices
- Estimated completion time: 60 minutes

## Post-Lab Skills Application

**Enterprise Implementation Steps**:
1. **Assess current state**: Evaluate existing project management processes
2. **Configure organizational structure**: Set up teams, areas, iterations
3. **Migrate work items**: Import backlog from current tools (Jira, Trello, etc.)
4. **Train team members**: Conduct workshops on Azure Boards best practices
5. **Establish metrics baseline**: Define KPIs and create monitoring dashboards
6. **Iterate and improve**: Gather feedback and optimize workflows

## Critical Notes
- 🎯 **Lab duration**: 60 minutes with hands-on exercises
- 💡 **Six key skills**: Teams, work items, sprints, Kanban, dashboards, process customization
- ⚠️ **Prerequisites**: Azure DevOps organization required before starting
- 📊 **Work item hierarchy**: Epic → Feature → User Story → Task
- 🔄 **WIP limits**: 2-3 items per developer in progress prevents context switching
- ✨ **Dashboards**: Velocity, burndown, cumulative flow for real-time monitoring

[Launch Lab](https://go.microsoft.com/fwlink/?linkid=2269557) | [Learn More](https://learn.microsoft.com/en-us/training/modules/plan-agile-github-projects-azure-boards/12-agile-plan-portfolio-management-azure-boards)
