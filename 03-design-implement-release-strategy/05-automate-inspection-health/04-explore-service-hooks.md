# Explore Service Hooks

⏱️ **Duration**: ~1 minute | 📚 **Type**: Conceptual

## Overview

Service hooks enable **automated task execution** across external services triggered by Azure DevOps project events, facilitating cross-platform workflow automation and real-time integration capabilities. Service hooks eliminate polling mechanisms while enabling immediate response to project lifecycle events.

---

## What Are Service Hooks?

**Definition**: Event-driven integrations that automatically trigger actions in external services when specific events occur in Azure DevOps projects.

**Architecture**:
```
Azure DevOps Event (trigger)
    ↓
Service Hook (configuration)
    ↓
External Service Action (automated response)
```

---

## Integration Scenarios

### Scenario 1: Work Item Synchronization

**Use Case**: Sync Azure DevOps work items with project management platforms

**Example - Trello Integration**:
```
Azure DevOps Event:
├── Event Type: Work item created
├── Work Item Type: Bug
├── Project: MyApplication

Service Hook Trigger: ⚡

Trello Action:
├── Create card in "Bugs" board
├── Card Title: [Bug #4567] Payment gateway timeout
├── Card Description: 
│   Priority: High
│   Assigned to: John Doe
│   Steps to reproduce: [from Azure DevOps]
│   Expected result: [from Azure DevOps]
│   Actual result: [from Azure DevOps]
├── Labels: Priority-High, Component-Payment, Source-AzureDevOps
└── Due Date: Sprint end date

Result: Bug automatically visible in Trello for non-technical stakeholders
```

**Benefits**:
- ✅ **Cross-team visibility**: Product managers see bugs without Azure DevOps access
- ✅ **Automatic synchronization**: No manual card creation
- ✅ **Bi-directional updates** (optional): Changes in Trello can update Azure DevOps
- ✅ **Unified workflow**: Technical and non-technical teams use preferred tools

---

### Scenario 2: Team Communication Automation

**Use Case**: Mobile push notifications for build failures

**Example - Mobile App Integration**:
```
Azure DevOps Event:
├── Event Type: Build completed
├── Build Status: Failed
├── Build Definition: MyApp-CI
├── Branch: main

Service Hook Trigger: ⚡

Mobile Push Notification:
├── Recipient: Developer who triggered build + on-call engineer
├── Title: "🚨 Build Failed - MyApp-CI"
├── Message: 
│   Branch: main
│   Commit: abc1234 "Fix payment bug"
│   Error: Unit tests failed (3 of 120)
│   Duration: 8 minutes
├── Actions:
│   [View Logs] [Retry Build] [Assign to Me]
└── Urgency: High (main branch failure)

Result: Immediate awareness even when away from desk
```

**Benefits**:
- ✅ **Immediate notification**: < 10 seconds after build failure
- ✅ **Mobile accessibility**: Respond from anywhere
- ✅ **Actionable alerts**: Quick links to logs and retry options
- ✅ **On-call support**: Automatic escalation to on-call engineer

---

### Scenario 3: Custom Application Event Processing

**Use Case**: Feed deployment events into custom monitoring/analytics system

**Example - Deployment Tracking System**:
```
Azure DevOps Event:
├── Event Type: Release deployment completed
├── Environment: Production
├── Release: Release-456
├── Status: Succeeded
├── Duration: 12 minutes

Service Hook Trigger: ⚡

Custom Webhook: POST https://monitoring.company.com/api/deployments
├── Payload (JSON):
│   {
│     "eventType": "deployment_completed",
│     "timestamp": "2024-01-15T14:30:00Z",
│     "releaseId": "456",
│     "releaseName": "Release-456",
│     "environment": "production",
│     "status": "succeeded",
│     "duration": 720,
│     "artifacts": [
│       {
│         "name": "myapp-build",
│         "version": "1.2.3",
│         "commit": "abc1234"
│       }
│     ],
│     "triggeredBy": "john.doe@company.com"
│   }
│
└── Custom Application Response:
    ├── Create deployment marker in Grafana dashboard
    ├── Update deployment frequency metrics
    ├── Trigger post-deployment smoke tests
    ├── Send notification to #production-deployments Slack channel
    └── Return: {"deploymentId": "d-78901", "monitorId": "m-45678"}

Result: Centralized deployment tracking across all applications
```

**Benefits**:
- ✅ **Unified monitoring**: All deployments visible in single dashboard
- ✅ **Correlation analysis**: Link deployments to performance metrics
- ✅ **Automated actions**: Trigger follow-up workflows
- ✅ **Custom logic**: Implement organization-specific automation

---

## Efficient Event-Driven Automation

### Polling vs. Service Hooks

**Polling (Legacy Approach)**:
```
Custom Application:
├── Every 1 minute: Call Azure DevOps API
│   GET /api/builds?status=completed&since=lastCheck
├── Check for new completed builds
├── If new builds found: Process them
└── If no new builds: Wait 1 minute, repeat

Problems:
❌ API rate limits (200 requests/minute per IP)
❌ Wasted API calls (95% return "no new data")
❌ Delayed detection (up to 1 minute latency)
❌ Resource waste (server constantly polling)
❌ Scalability issues (100 applications = 100 polling loops)
```

**Service Hooks (Modern Approach)**:
```
Azure DevOps:
├── Build completes
├── Service hook immediately triggers webhook
└── POST https://app.company.com/api/build-completed

Custom Application:
├── Receives webhook instantly (< 1 second)
├── Processes build event
└── Waits for next webhook (passive)

Benefits:
✅ No API rate limits (webhooks not counted)
✅ No wasted requests (only real events sent)
✅ Instant detection (< 1 second latency)
✅ Resource efficient (no polling loops)
✅ Infinitely scalable (1 webhook per event)
```

**Efficiency Comparison**:

| Metric | Polling | Service Hooks | Improvement |
|--------|---------|---------------|-------------|
| **API Calls (1 event/hour)** | 60 calls/hour | 1 webhook/hour | 98.3% reduction |
| **Latency** | 0-60 seconds | < 1 second | 99% reduction |
| **Server Load** | Continuous | Event-only | 99% reduction |
| **Scalability** | Limited (rate limits) | Unlimited | Infinite |

---

## Azure DevOps Native Service Hook Integration

Azure DevOps provides **40+ native integrations** across multiple service categories:

### Supported Services by Category

#### CI/CD Tools
| Service | Use Case |
|---------|----------|
| **AppVeyor** | Trigger AppVeyor builds from Azure DevOps code commits |
| **Bamboo** | Integrate Atlassian Bamboo CI builds with Azure DevOps |
| **Jenkins** | Trigger Jenkins jobs from Azure Repos commits/PRs |
| **MyGet** | Publish NuGet/npm packages to MyGet feeds |

#### Communication Platforms
| Service | Use Case |
|---------|----------|
| **Campfire** | Post build/release notifications to Campfire rooms |
| **Flowdock** | Send Azure DevOps events to Flowdock team inbox |
| **HipChat** | Notify HipChat rooms of builds, deployments, work item updates |
| **Slack** | Real-time Slack notifications for project events |
| **Hubot** | Trigger Hubot scripts from Azure DevOps events |

#### Project Management & Support
| Service | Use Case |
|---------|----------|
| **Trello** | Auto-create/update Trello cards from work items |
| **UserVoice** | Sync feature requests between UserVoice and Azure Boards |
| **Zendesk** | Create Zendesk tickets from Azure DevOps bugs |

#### Cloud Services
| Service | Use Case |
|---------|----------|
| **Azure Service Bus** | Publish events to Service Bus for enterprise integration |
| **Azure Storage** | Store event logs/artifacts in Azure Blob Storage |

#### Custom Integration
| Service | Use Case |
|---------|----------|
| **Web Hooks** | Send HTTP POST to any custom endpoint |
| **Zapier** | Connect to 5,000+ apps via Zapier workflows |

---

## Service Hook Configuration Example

### Slack Integration Configuration

**Scenario**: Notify #production-alerts channel on production deployment failures

**Configuration Steps**:
```
1. Navigate to: Project Settings → Service hooks
2. Click: + Create subscription
3. Select Service: Slack
4. Configure Event:
   ├── Trigger: Release deployment completed
   ├── Status: Failed
   ├── Release definition: MyApp-Production
   └── Stage name: Production

5. Configure Slack:
   ├── Webhook URL: https://hooks.slack.com/services/T00/B00/Xx...
   ├── Channel: #production-alerts
   ├── Message format: Detailed
   └── Icon: 🚨

6. Test subscription: Send test notification
7. Save subscription
```

**Result - Slack Message**:
```
🚨 Production Deployment Failed

Project: MyApplication
Release: Release-456
Definition: MyApp-Production
Stage: Production (Deploy to Azure Web App)
Status: ❌ Failed
Error: Health check failed - HTTP 500 errors detected
Duration: 8 minutes 34 seconds
Triggered by: john.doe@company.com
Time: 2024-01-15 14:30:00 UTC

[View Release] [View Logs] [Retry Deployment]
```

---

## Webhook Payload Example

**Service Hook**: Web Hooks (custom endpoint)

**Event**: Build completed

**Payload** (JSON sent via HTTP POST):
```json
{
  "subscriptionId": "12345678-1234-1234-1234-123456789012",
  "notificationId": 1,
  "id": "98765432-4321-4321-4321-210987654321",
  "eventType": "build.complete",
  "publisherId": "tfs",
  "message": {
    "text": "Build MyApp-CI 20240115.1 succeeded",
    "html": "Build <a href=\"...\">MyApp-CI 20240115.1</a> succeeded",
    "markdown": "Build [MyApp-CI 20240115.1](...) succeeded"
  },
  "detailedMessage": {
    "text": "Build MyApp-CI 20240115.1 succeeded\nRequested by John Doe",
    "html": "Build <a href=\"...\">MyApp-CI 20240115.1</a> succeeded<br/>Requested by John Doe"
  },
  "resource": {
    "id": 123,
    "buildNumber": "20240115.1",
    "status": "succeeded",
    "result": "succeeded",
    "queueTime": "2024-01-15T14:20:00Z",
    "startTime": "2024-01-15T14:21:00Z",
    "finishTime": "2024-01-15T14:30:00Z",
    "reason": "manual",
    "requestedFor": {
      "displayName": "John Doe",
      "uniqueName": "john.doe@company.com"
    },
    "definition": {
      "id": 5,
      "name": "MyApp-CI"
    },
    "sourceBranch": "refs/heads/main",
    "sourceVersion": "abc123def456"
  },
  "resourceVersion": "5.1",
  "resourceContainers": {
    "collection": {
      "id": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
    },
    "project": {
      "id": "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb"
    }
  },
  "createdDate": "2024-01-15T14:30:05Z"
}
```

**Custom Application Processing**:
```javascript
// Express.js webhook handler
app.post('/api/azure-devops-webhook', (req, res) => {
  const event = req.body;
  
  if (event.eventType === 'build.complete') {
    const build = event.resource;
    
    // Extract key information
    const buildNumber = build.buildNumber;
    const status = build.result; // succeeded, failed, partiallySucceeded
    const duration = (new Date(build.finishTime) - new Date(build.startTime)) / 1000; // seconds
    const branch = build.sourceBranch.replace('refs/heads/', '');
    
    // Custom logic
    if (status === 'failed' && branch === 'main') {
      // Alert on-call engineer
      sendPagerDutyAlert({
        severity: 'high',
        summary: `Main branch build ${buildNumber} failed`,
        details: event
      });
    }
    
    // Store in database for analytics
    db.builds.insert({
      buildNumber,
      status,
      duration,
      branch,
      timestamp: build.finishTime
    });
    
    // Update dashboard
    dashboard.updateBuildMetrics();
  }
  
  // Acknowledge receipt
  res.status(200).json({ received: true });
});
```

---

## Service Hook Marketplace Extensions

**Expanding Ecosystem**: Service hook capabilities continuously expand through:
- **Marketplace extensions**: 3rd party integrations (PagerDuty, Datadog, New Relic)
- **Custom webhook implementations**: Build your own integrations
- **Zapier**: Connect to 5,000+ apps (Gmail, Google Sheets, Salesforce, etc.)

**Example - Zapier Multi-Step Workflow**:
```
Azure DevOps Event: Production deployment succeeded
    ↓
Zapier Workflow:
├── Step 1: Parse deployment details
├── Step 2: Create row in Google Sheets (Deployment Log)
├── Step 3: Send Gmail notification to stakeholders
├── Step 4: Post to Twitter: "Version 1.2.3 now live!"
└── Step 5: Create calendar event for post-deployment review
```

---

## Quick Reference

### Service Hooks vs. Notifications

| Feature | Service Hooks | Notifications |
|---------|---------------|---------------|
| **Purpose** | Automated external integrations | Stakeholder communication |
| **Trigger** | Any Azure DevOps event | Any Azure DevOps event |
| **Target** | External services (Slack, webhooks) | Email, IM, mobile |
| **Customization** | Webhook payload format | Email/message templates |
| **Use Case** | Cross-platform automation | Human awareness |

---

## Key Takeaways

- 🔗 **Service hooks** automate task execution in external services triggered by Azure DevOps events
- 🎯 **Integration scenarios**: Work item sync (Trello), team communication (Slack), custom processing (webhooks)
- ⚡ **Event-driven efficiency**: Eliminate polling, enable real-time response (< 1 second latency)
- 🛠️ **40+ native integrations**: CI/CD tools, communication platforms, project management, cloud services
- 📡 **Custom webhooks**: Build unlimited integrations via HTTP POST to custom endpoints
- 🔌 **Zapier integration**: Connect to 5,000+ apps without coding

---

## Next Steps

✅ **Completed**: Service hooks overview and integration scenarios

**Continue to**: Unit 5 - Configure Azure DevOps notifications (hands-on configuration guide)

---

## Additional Resources

- [Service Hooks in Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/service-hooks/overview)
- [Create a service hook for Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks)
- [Slack integration for Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/slack)
- [Azure DevOps Marketplace - Service Hooks](https://marketplace.visualstudio.com/search?target=AzureDevOps&category=Service%20Hooks)

[↩️ Back to Module Overview](01-introduction.md) | [⬅️ Previous: Events and Notifications](03-explore-events-notifications.md) | [➡️ Next: Configure Azure DevOps Notifications](05-configure-azure-devops-notifications.md)
