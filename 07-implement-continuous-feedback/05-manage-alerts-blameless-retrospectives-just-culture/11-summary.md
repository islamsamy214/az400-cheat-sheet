# Summary

## Module Overview
Build more reliable systems through:
- Effective alert management (help rather than overwhelm teams)
- Healthier organizational culture
- Retrospectives that improve systems (not blame individuals)

## Alert Management and Smart Detection

### Baseline Learning
- Application Insights establishes baselines over 8 days
- Learns typical behavior patterns for different operation types
- Understands certain requests naturally take longer

### Three Detection Categories
1. **Response time degradation**: Overall application slowness
2. **Dependency duration degradation**: External service problems
3. **Slow performance patterns**: Browser-specific, geographic, server-specific, user segment, operation-specific

### Investigation Tools
- **Performance blade**: Request distribution analysis
- **Application Insights Profiler**: Method-level timing and call stacks
- **Snapshot Debugger**: Captures exception state with variable values

### Notification Management
- Configure notification recipients
- Rate limiting prevents alert fatigue (1 email/day/resource)
- Integrate with action groups for routing to:
  - Incident management systems (PagerDuty, Opsgenie)
  - Communication platforms (Teams, Slack)

## Performance Improvement

### Triage Strategies
- Assess business impact (usage frequency, severity, affected user segments)
- Use impact data as guidance
- Gather additional business context

### Diagnostic Approach
Identify bottlenecks:
- **Send request time**: Server response
- **Receiving response time**: Large payloads and resources
- **Client processing time**: JavaScript execution
- **Dependency tracking**: Distinguish application vs. external service issues

### Optimization Techniques
Target based on diagnosis:
- Asynchronous loading
- Script bundling and code splitting
- Efficient data formats
- Geographic co-location of services
- Query efficiency improvements
- Caching strategies
- Connection pooling
- Capacity planning (horizontal/vertical scaling)

## Reducing Alert Noise

### Actionable Alert Characteristics (All Four Required)
| Criterion | Description |
|-----------|-------------|
| **Urgent** | Requires immediate attention |
| **Important** | Affects users or business functions |
| **Actionable** | Responders can take specific steps |
| **Real** | Genuine problem, not false positive |

### Problem Classification
1. **Availability and basic functionality**: Can users access system?
2. **Latency**: Is system responding within acceptable timeframes?
3. **Correctness**: Completeness, freshness, and durability of data
4. **Feature-specific problems**: Affecting particular features/segments

### Symptoms vs. Causes
- **Alert on symptoms** (what users experience)
- **Not causes** (underlying technical issues)
- Symptom-based alerting catches multiple causes with one rule
- Directly correlates with user impact
- Include cause info in dashboards for investigation

### Balancing Sensitivity
- Err on side of removing noisy alerts
- Over-monitoring harder to solve than under-monitoring
- Alert fatigue causes teams to ignore/silence notifications
- Real problems get lost in noise

### Managing Urgency Levels
| Level | Type | Response |
|-------|------|----------|
| **Page-worthy** | Urgent, active user impact | Immediate |
| **Next-business-day** | Important but not currently affecting | Business hours |
| **Track-and-trend** | Informational patterns | Regular review |

## Blameless Retrospectives

### The Problem with Blame
- Punishment prevents learning
- Engineers won't provide details needed to understand failures
- Lack of understanding guarantees failure repetition

### The Cycle of Blame
1. Reduces trust between engineers and management
2. Engineers silent about details to avoid punishment
3. "Cover-Your-Mistake" engineering develops
4. Errors more likely (latent conditions can't be identified)

### Blameless Process
Engineers provide detailed accounts without fear:
- Actions, observations, expectations
- Assumptions
- Understanding of events
- Transparency essential for learning

### Understanding Human Decision-Making
**Actions made sense at the time** based on:
- Information available
- Understanding of system state
- Goals and priorities at that moment
- Time pressures and competing demands
- Training and past experiences

**Hindsight**: What looks obvious after was not obvious before

### Erik Hollnagel's Principle
**Accidents don't happen because people gamble and lose.**

Happen because person believes:
- What is about to happen isn't possible, **OR**
- Has no connection to what they're doing, **OR**
- Possibility of intended outcome is worth the risk

### What Blameless Doesn't Mean
| ❌ Not | ✅ Is |
|--------|-------|
| Accountability-free | Standards maintained, focus on learning |
| Ignoring malicious actions | Addresses honest mistakes |
| No consequences | Consequences focus on system improvement |

## Just Culture

### Engineers as Experts in Their Errors
- Most expert in their specific error
- Should be heavily involved in developing remediation
- Not "off the hook" - on the hook for helping organization improve
- Transforms failure into opportunity for contribution

### Core Principles
1. **Encourage learning**: Regular blameless post-mortems
2. **Multiple perspectives**: Gather details without punishment
3. **Transform mistakes**: Enable education of rest of organization
4. **Human discretion**: Accept discretionary space exists
5. **Focus on environment**: System factors vs. personal characteristics
6. **Bridge gap**: Blunt end (management) understands sharp end (engineers)
7. **Define collaboratively**: Sharp end informs appropriate behavior boundaries

### Human Discretion
- Always discretionary space for decisions
- Judgment occurs in hindsight
- Eliminate Hindsight Bias: Ask "what made sense given what they knew then?"

### Focus on Environment
- Fundamental Attribution Error: Attribute to character vs. circumstances
- Focus on environment and circumstances
- Ask what system factors contributed

### Bridge Blunt End and Sharp End
- Blunt end (management/leadership)
- Sharp end (engineers interacting with systems)
- Understand actual work, not imagined work from plans/procedures

### Define Behavior Collaboratively
- Sharp end informs about appropriate/inappropriate behavior
- Not something blunt end can determine alone
- Develop standards collaboratively
- Not top-down rules ignoring operational reality

### Two Approaches to Failure

| Approach 1: Blame | Approach 2: Learning |
|-------------------|----------------------|
| Assume incompetence | Thorough investigation |
| Respond with blame | Treat engineers with respect |
| False comfort | Learn from event |
| Doesn't improve safety | Lasting improvements |

## Key Takeaways

### 1. Effective Alert Management Reduces Noise
- Configure alerts: urgent, important, actionable, real
- Alert on symptoms (not causes)
- Balance awareness vs. alert fatigue

### 2. Smart Detection Enables Proactive Monitoring
- Automatic performance monitoring learns baseline behavior
- Notifies of degradation before user complaints
- Use diagnostic tools for systematic investigation/resolution

### 3. Blameless Retrospectives Improve Safety
- Psychological safety enables detailed information sharing
- Understanding failure mechanisms enables system improvements
- Prevents recurrence through learning

### 4. Just Culture Balances Safety and Accountability
- Accountability focuses on learning/improvement (not punishment)
- Engineers contribute expertise to remediation
- Transforms failure into organizational learning opportunity

### 5. Context Matters
Understanding decision context essential:
- Information they had
- Pressures they faced
- Goals they were pursuing
- Enables learning and system improvement

## Additional Resources
- [Smart detection - performance anomalies](https://learn.microsoft.com/en-us/azure/azure-monitor/app/proactive-performance-diagnostics)
- [Overview of alerting and notification monitoring in Azure](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview)
- [Create, view, and manage Metric Alerts Using Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-metric)
- [Brian Harry's Blog - A good incident postmortem](https://blogs.msdn.microsoft.com/bharry/2018/03/02/a-good-incident-postmortem/)

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-alerts-blameless-retrospectives-just-culture/11-summary)
