# Reduce Meaningless and Non-Actionable Alerts

## Key Concepts
- Alert fatigue from too many alerts is worse than under-monitoring
- Alert on symptoms users experience, not causes
- Distinguish urgent (page-worthy) from important (next-day) issues
- Higher abstraction alerts catch more problems with fewer rules
- Vague alerts ("something seems weird") create alert fatigue

## The Purpose of Alerting

### When to Alert (Human Intervention Needed)
Trigger alerts only when:
- Real problem affects users (or will soon)
- Problem requires human decision-making or manual action
- Automatic remediation failed or unavailable

### Clear Investigation Path Required
Alert recipients should be able to:
1. Determine if genuine problem exists
2. Assess severity and user impact
3. Mitigate immediate problem
4. Investigate root cause
5. Implement preventive measures

### Avoid Vague Alerts
- Don't alert because "something seems a bit weird"
- Vague alerts create fatigue and train teams to ignore notifications
- Exception: Security auditing on narrowly scoped system components

## Principles for Actionable Alerts

### Urgency and Importance (All Four Required)

| Criterion | Description | Example |
|-----------|-------------|---------|
| **Urgent** | Requires attention now, not tomorrow | Don't page at 3 AM if it can wait |
| **Important** | Affects users, revenue, critical business functions | Internal tooling ≠ wake-up call |
| **Actionable** | Recipient can take specific steps | If requires escalation, reconsider recipient |
| **Real** | Genuine problem, not false positive | Not from overly sensitive thresholds |

### Problem Representation

| Type | Description | Alert? |
|------|-------------|--------|
| **Ongoing problems** | Users currently experiencing issues | ✅ Yes |
| **Imminent problems** | Will occur soon if not addressed (disk full in 2h) | ✅ Yes |
| **Historical problems** | Already resolved (30s spike 10 minutes ago) | ❌ No |

### Balancing Sensitivity

**Err on side of removing noisy alerts** - Over-monitoring harder to solve than under-monitoring

**Over-monitoring Consequences**:
- Alert fatigue → teams ignore/silence notifications
- Real problems lost in noise
- Increased on-call stress without benefit
- More time on false positives than real issues

**Under-monitoring Risks**:
- Might miss some issues
- Team remains responsive to actual problems
- Easier to fix: add targeted alerts based on real incidents
- Over-monitoring requires difficult work of removing established alerts

**Practical Approach**:
1. Start with fewer alerts covering critical paths
2. Add alerts based on real incidents where alerting would have helped
3. Remove alerts that frequently fire without real problems

## Problem Classification

### 1. Availability and Basic Functionality
**Most critical alerts**:
- Service completely unavailable
- Authentication fails preventing user access
- Critical transactions fail (payment processing, data submission)
- Database connectivity lost

### 2. Latency
Is system responding within acceptable timeframes?
- Response time exceeds SLA thresholds
- User-facing operations exceed user tolerance
- Background jobs miss processing deadlines

### 3. Correctness
Is system producing correct results?

| Aspect | Issues |
|--------|--------|
| **Completeness** | Missing data in reports, incomplete transaction processing |
| **Freshness** | Stale dashboard data, delayed processing pipelines |
| **Durability** | Failed backups, data replication issues |

### 4. Feature-Specific Problems
Issues affecting particular features/user segments:
- Specific API endpoints failing (others work)
- Mobile broken, desktop working
- Features unavailable in certain regions

## Symptoms vs. Causes

### Alert on Symptoms (What Users Experience)
- Response time is slow
- Errors occur
- Data is missing

**Why Symptom-Based Alerting is Better**:
- One symptom alert catches multiple underlying causes
- Symptoms directly correlate with user impact
- No need to anticipate every possible cause
- Alerts remain valid as infrastructure changes

**Example**: 
- ✅ Alert: "Response time exceeds 2s for 5 minutes"
- ❌ Don't create separate alerts for: high CPU, database slowness, network issues
- Symptom alert fires regardless of underlying cause

### Include Causes in Context (Don't Alert on Them)

**Why Not Alert on Causes**:
- High CPU doesn't necessarily mean users affected
- System might handle load fine
- Database query increases might not impact users (if cached)

**Use Causes for Investigation**:
- Provide cause info in dashboards/runbooks/monitoring tools
- Helps diagnose root causes without separate alerts

**Example**:
- Symptom alert: "API response time elevated"
- Dashboard shows: Database query time tripled
- Responder investigates database (no separate "database slow" alert needed)

## Alert Scope and Abstraction

### Higher Abstraction Benefits
Alerting at higher levels (user-facing metrics > infrastructure):
- Broader problem coverage with fewer alert rules
- Better correlation with user impact
- More resilient as infrastructure changes

**Example**: "Checkout conversion rate drops 20%" catches:
- Database issues
- Payment gateway problems
- Application bugs
- All with one rule

### Don't Over-Abstract

**Problems with Too Much Abstraction**:
- "Something is wrong" doesn't help start investigation
- Overly broad alerts fire for unrelated reasons (difficult pattern recognition)
- Insufficient detail for severity/response determination

**Balance**: Alert at highest level that still provides actionable context
- User journey monitoring provides good balance
- Tracks specific user flows through system

## Managing Non-Critical Issues

### Separate Urgency Levels

| Level | Definition | Response Time | Examples |
|-------|------------|---------------|----------|
| **Page-worthy (urgent)** | Active user impact | Immediate | User impact, revenue loss, data loss risk, security breaches |
| **Next-business-day (important)** | Should be addressed soon | Business hours | Disk 70% (days before critical), certificate expires in week, elevated errors on non-critical features |
| **Track-and-trend (informational)** | Worth monitoring for patterns | Regular review | Gradual performance degradation, increasing resource utilization in normal ranges |

### Implementation Approaches

**Different Notification Channels**:
- Urgent → Paging systems
- Important → Team channels (business hours monitoring)
- Informational → Dashboards and reports

**Work Queues**:
- On-call handles urgent items immediately
- Teams address important items during regular hours
- Informational items inform capacity planning/optimization

**Regular Review**:
- Schedule reviews of non-urgent alerts
- Catch gradual degradation before it becomes urgent

## Best Practices Summary

| Do | Don't |
|----|-------|
| ✅ Alert on symptoms users experience | ❌ Alert on every possible cause |
| ✅ Start with fewer critical alerts | ❌ Over-monitor "just in case" |
| ✅ Remove noisy alerts aggressively | ❌ Keep alerts that frequently false positive |
| ✅ Use higher abstraction when possible | ❌ Create dozens of low-level alerts |
| ✅ Separate urgency levels | ❌ Page for everything |
| ✅ Make alerts actionable | ❌ Alert on vague "something seems weird" |

## Critical Notes
- ⚠️ Over-monitoring harder to fix than under-monitoring
- 💡 One symptom alert can catch multiple causes
- 🎯 Alert at highest level that still provides actionable context
- 📊 Separate urgent (page) vs. important (next-day) vs. informational (track)
- 🔍 Provide cause information in dashboards, not as separate alerts

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-alerts-blameless-retrospectives-just-culture/7-reduce-meaningless-non-actionable-alerts)
