# Configure Smart Detection Notifications

## Key Concepts
- Smart detection notifications enabled by default
- Rate limiting prevents alert fatigue (1 email/day/resource max)
- Default recipients based on Azure RBAC roles
- Customizable through portal or email "Configure" link
- Integration with action groups for advanced workflows

## Default Notification Configuration

### Automatic Enablement
- Enabled by default when configuring Application Insights
- No additional setup overhead
- Performance issues surfaced immediately

### Default Recipients (Azure RBAC Roles)
| Role | Access Level |
|------|--------------|
| **Owners** | Full control over Application Insights resource |
| **Contributors** | Modify configuration and view data |
| **Readers** | Read-only access to data |

- Ensures people managing/monitoring application receive alerts automatically

## Customizing Notification Recipients

### Access Configuration (Two Methods)
1. **From email notifications**: Click "Configure" link in smart detection email
2. **From Azure portal**: Navigate to Application Insights → Smart Detection settings

### Modification Capabilities
- Add additional email addresses
- Remove users from notification list
- Change notification routing for incident management systems
- Configure action groups for advanced scenarios

### When to Customize Recipients
- On-call rotation requires specific team members (not all contributors)
- Integration with PagerDuty, Opsgenie, or other incident management platforms
- Different teams monitor different application aspects
- Reduce notification volume for users not needing immediate alerts

## Managing Notification Frequency

### Rate Limiting (Prevents Alert Fatigue)

| Limit Type | Configuration | Purpose |
|------------|---------------|---------|
| **One email per day** | Per Application Insights resource | Prevent constant interruptions |
| **New issue requirement** | Daily email only if new issues detected | Ensure fresh information |
| **No duplicates** | Once reported, no repeated notifications | Avoid reminders about known issues |

### Why Rate Limiting Matters
- Without it: Dozens of related anomalies = separate notifications
- Consolidation: Keep informed without notification storms
- Prevents teams from ignoring alerts due to overload

## Unsubscribing from Notifications

### Individual User Control
- Each email includes unsubscribe link
- Opt out without affecting other recipients

### When Unsubscribing Makes Sense
- Role transition - no longer monitoring application performance
- Team uses centralized incident management tools
- Receiving notifications for multiple resources, want to focus on specific ones
- Prefer checking Application Insights portal directly

### Re-subscribing
- Update subscription through Smart Detection settings in Azure portal

## Integrating with Alerting Workflows

### Beyond Email Notifications
For larger organizations needing sophisticated workflows:

### Action Groups
Route smart detection findings to:
- **Incident management**: PagerDuty, Opsgenie, ServiceNow
- **Collaboration tools**: Microsoft Teams, Slack
- **Webhook endpoints**: Custom notification handling
- **SMS/Voice**: Critical alerts
- **Automation**: Azure Functions or Logic Apps for automated responses

### Alert Rules
Create custom rules based on smart detection to:
- Apply filtering by severity or affected user count
- Route different issue types to different teams
- Escalate unresolved alerts after specific time periods
- Suppress notifications during maintenance windows

## Notification Strategy Considerations

### Balance Awareness vs. Alert Fatigue
| Priority Level | Routing Strategy |
|----------------|------------------|
| **Critical issues** | On-call engineers immediately |
| **Less urgent anomalies** | Team channels for business hours investigation |
| **Multiple related issues** | Consolidate notifications |
| **Context provision** | Enable rapid assessment without portal access |
| **Metrics tracking** | Identify if alert fatigue developing |

## Configuration Workflow

```bash
# Navigate to Smart Detection settings
Azure Portal → Application Insights → Smart Detection → Configure

# Modify recipients
Add emails → Save
Remove users → Save

# Configure action groups
Smart Detection → Action groups → Add
Select incident management integration
```

## Critical Notes
- ⚠️ Maximum 1 email per day per resource
- 💡 Use action groups for advanced notification scenarios
- 🎯 No duplicate notifications for same issue
- 📊 Track notification metrics to prevent alert fatigue
- 🔍 Customize recipients for on-call rotations and team separation

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-alerts-blameless-retrospectives-just-culture/4-explore-smart-detection-notifications)
