# Design a License Management Strategy

## Key Considerations

### Assess Your DevOps Phase
- What phase are you in?
- Draft architecture for DevOps implementation?
- Know which resources you'll consume?

### Team & Usage Analysis
- **How many people** are using the feature?
- **Queue tolerance**: How long willing to wait for pipelines?
- **Urgency level**: Is this critical or just validation?

### User Access Levels
**Question**: Should all users access all features?
- Stakeholders (view-only access)
- Basic users (standard features)
- Visual Studio license holders (enhanced features)

### Advanced Features
- **Package Management**: Need more Artifact storage space?
- **Parallel Jobs**: Want builds to run without queue dependency?
- **Multiple Teams**: Building solutions simultaneously?

## Licensing Decision Factors

| Factor | Question | Impact |
|--------|----------|--------|
| **Pipeline Parallelism** | Need parallel jobs? | Pay for parallel execution to avoid queue wait |
| **User Types** | Stakeholder vs. developer? | Different license tiers for different roles |
| **Artifact Storage** | Advanced package management? | May need additional Artifacts capacity |
| **Team Size** | How many teams building? | Scale license count accordingly |

## Resources
- **Azure DevOps Pricing**: [https://azure.microsoft.com/pricing/details/devops/azure-devops-services/](https://azure.microsoft.com/pricing/details/devops/azure-devops-services/)
- **GitHub Pricing**: [https://github.com/pricing/](https://github.com/pricing/)

## Critical Notes
- 💡 Understand your DevOps phase before purchasing licenses
- 🎯 Match license levels to user needs (don't over-license)
- ⚠️ Consider queue wait times vs. parallel job costs
- 📊 Review pricing regularly as usage scales

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-devops/9-design-license-management-strategy)
