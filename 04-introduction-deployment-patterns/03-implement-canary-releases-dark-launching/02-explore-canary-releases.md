# Explore Canary Releases

**Duration**: 4 minutes

## Origin: The Coal Mine Canary

**Canary release** terminology originates from historical mining practices where **canaries served as early warning systems** for toxic gas detection in coal mines.

### Historical Context
- 🐦 **Canaries demonstrated heightened sensitivity** to hazardous atmospheric conditions
- ⚠️ **Succumbed to toxic exposure before human miners**, providing critical escape time
- 🚨 **Early detection principle**: Small indicator detects danger before mass exposure

## Software Canary Release Concept

Canary release methodology applies this early detection principle to software deployment, enabling **problem identification through limited user exposure** before comprehensive rollout.

### Core Principles
- 🎯 **Progressive exposure**: Deploy to minimal user cohorts (1-5% initially)
- 📊 **Constrained impact scope**: Limit blast radius during validation
- 🔍 **Comprehensive monitoring**: Generate actionable telemetry
- ✅ **Data-driven decisions**: Continue or rollback based on metrics

## Canary Release Workflow

```
┌──────────────────────────────────────────────────────────────┐
│ Phase 1: Deploy to Canary (1-5% of users)                   │
├──────────────────────────────────────────────────────────────┤
│ - Deploy new version v2.0 to isolated environment           │
│ - Route 5% of traffic to v2.0 (canary users)               │
│ - Route 95% of traffic to v1.0 (stable version)            │
│ - Monitor: Error rates, performance, user behavior          │
├──────────────────────────────────────────────────────────────┤
│ Phase 2: Evaluate Metrics (24-48 hours)                     │
├──────────────────────────────────────────────────────────────┤
│ Decision Point:                                              │
│ ✅ Metrics Good → Proceed to Phase 3                        │
│ ❌ Metrics Bad → Rollback (redirect 100% to v1.0)          │
├──────────────────────────────────────────────────────────────┤
│ Phase 3: Gradual Expansion                                   │
├──────────────────────────────────────────────────────────────┤
│ - Day 2: 10% → v2.0 (expand canary)                        │
│ - Day 4: 25% → v2.0 (wider rollout)                        │
│ - Day 7: 50% → v2.0 (half and half)                        │
│ - Day 10: 100% → v2.0 (full rollout) ✅                    │
└──────────────────────────────────────────────────────────────┘
```

## Key Benefits

### 1. Limited Blast Radius
Performance degradation or scalability constraints affect **only canary users** (1-5%), not entire user base.

**Example**: Bug in new payment processor affects 5% of transactions, not 100%.

### 2. Real Production Validation
Test with **real users** in **real production environment** with actual traffic patterns and data.

**vs. Staging**: Staging uses synthetic data and limited traffic, doesn't replicate production conditions.

### 3. Data-Driven Rollback
Comprehensive monitoring generates **actionable telemetry** enabling data-driven continuation or rollback decisions.

**Metrics to Monitor**:
- 📉 Error rate (target: < 0.1% increase)
- ⚡ Response time (target: < 10% degradation)
- 💰 Conversion rate (target: no decrease)
- 🐛 Bug reports (target: no spike)
- 📊 User satisfaction (target: stable or improved)

### 4. Gradual Confidence Building
Extended monitoring periods at each stage authorize production environment promotion, expanding feature availability incrementally.

```
1% (48 hours) → 5% (48 hours) → 10% (72 hours) → 25% (72 hours) → 50% (96 hours) → 100%
```

## Implementation Techniques

Canary release implementation leverages **integrated deployment mechanisms** combining feature toggles, intelligent traffic routing, and deployment slot orchestration.

### Technique 1: Percentage-Based Traffic Routing
**Deployment slot configuration** enables controlled traffic distribution directing specified percentages to new feature implementations.

```yaml
# Azure Traffic Manager configuration
trafficRouting:
  - endpoint: app-v1.azurewebsites.net
    weight: 90  # 90% of traffic
  - endpoint: app-v2.azurewebsites.net
    weight: 10  # 10% of traffic (canary)
```

### Technique 2: Segment-Specific Targeting
**Feature toggle frameworks** enable precise user cohort targeting for granular exposure control.

```python
# Feature toggle with user segmentation
if user.email.endswith('@company.com'):
    # Internal employees (canary group)
    use_new_feature = True
elif user.in_group('beta_testers'):
    # Beta testers (early adopters)
    use_new_feature = True
elif random() < 0.05:
    # 5% of remaining users (random canary)
    use_new_feature = True
else:
    # 95% of users (stable version)
    use_new_feature = False
```

### Technique 3: Deployment Slot Orchestration
Azure App Service deployment slots enable **isolated environments** for canary versions.

```bash
# Deploy v2.0 to staging slot
az webapp deployment source config-zip \
  --resource-group myResourceGroup \
  --name myapp \
  --slot staging \
  --src v2.0.zip

# Route 10% traffic to staging (canary)
az webapp traffic-routing set \
  --resource-group myResourceGroup \
  --name myapp \
  --distribution staging=10 production=90
```

## Canary Release vs. Other Patterns

| Aspect | Canary Release | Blue-Green | Feature Toggle |
|--------|---------------|------------|----------------|
| **Gradual Rollout** | ✅ Yes (1% → 100%) | ❌ No (instant 100% switch) | ✅ Yes (percentage-based) |
| **Infrastructure** | 2 versions running | 2 environments idle/active | 1 environment |
| **Traffic Control** | Percentage-based | All-or-nothing switch | User-based targeting |
| **Rollback Speed** | Instant (redirect traffic) | Instant (swap back) | Instant (toggle off) |
| **Monitoring Period** | Days/weeks (gradual) | 24-48 hours (full traffic) | Days/weeks (gradual) |
| **Best For** | Risk-averse rollouts | Zero-downtime requirement | Targeted user exposure |

## Monitoring & Observability

### Critical Metrics to Track

**Performance Metrics**:
```
- Response time (p50, p95, p99)
- Throughput (requests/second)
- Error rate (4xx, 5xx responses)
- Database query performance
```

**Business Metrics**:
```
- Conversion rate
- Cart abandonment
- User engagement
- Revenue per user
```

**Infrastructure Metrics**:
```
- CPU utilization
- Memory usage
- Network latency
- Database connections
```

### Automated Canary Analysis

```yaml
# Azure Pipelines canary analysis
canaryAnalysis:
  duration: 2h
  metrics:
    - name: error_rate
      threshold: 0.5%  # Max 0.5% error rate
      action: rollback
    - name: response_time_p95
      threshold: 500ms  # Max 500ms p95 latency
      action: rollback
    - name: cpu_utilization
      threshold: 80%  # Max 80% CPU
      action: alert
  onFailure: rollback
  onSuccess: expand_to_25_percent
```

## Real-World Example: E-commerce Checkout

### Scenario
Deploy new single-page checkout (replacing 3-step checkout).

### Canary Rollout Plan

**Week 1: Internal Validation (1%)**
```
Traffic: 1% to new checkout (internal employees only)
Duration: 48 hours
Metrics:
  - Error rate: 0.1% (✅ Pass)
  - Response time: 300ms (✅ Pass, 100ms faster)
  - Conversion: 18% (✅ Pass, was 17%)
Decision: PROCEED to 5%
```

**Week 2: Early Adopters (5%)**
```
Traffic: 5% to new checkout (opt-in beta users)
Duration: 72 hours
Metrics:
  - Error rate: 0.2% (✅ Pass)
  - Response time: 320ms (✅ Pass)
  - Conversion: 18.5% (✅ Pass, improving)
Decision: PROCEED to 25%
```

**Week 3: Wider Rollout (25%)**
```
Traffic: 25% to new checkout (random users)
Duration: 96 hours
Metrics:
  - Error rate: 0.15% (✅ Pass)
  - Response time: 310ms (✅ Pass)
  - Conversion: 19% (✅ Pass, significant improvement)
Decision: PROCEED to 100%
```

**Week 4: Full Rollout (100%)**
```
Traffic: 100% to new checkout
Duration: Ongoing
Result: 19% conversion (2% improvement = +$500K/month revenue)
```

## Quick Reference

### Canary Release Checklist
- [ ] Deploy new version to isolated environment
- [ ] Configure traffic routing (start with 1-5%)
- [ ] Set up comprehensive monitoring
- [ ] Define rollback criteria (error thresholds)
- [ ] Monitor for 24-48 hours per stage
- [ ] Gradually expand (5% → 10% → 25% → 50% → 100%)
- [ ] Document lessons learned

### Key Characteristics
- 🐤 **Early detection**: Canary users detect issues first
- 📊 **Limited exposure**: 1-5% initial rollout
- 📈 **Gradual expansion**: Incremental traffic increase
- 🔍 **Continuous monitoring**: Real-time metrics analysis
- ⚡ **Fast rollback**: Instant traffic redirection
- ✅ **Data-driven**: Metrics guide rollout decisions

### Critical Notes
- ⚠️ **Monitor continuously** during canary phase (24/7 if critical)
- 💡 **Define rollback criteria** before deployment (automated if possible)
- 🎯 **Start small** (1-5%, not 20-30%)
- 📊 **Compare metrics** between canary and stable versions
- 🔄 **Wait 24-48 hours** per stage before expanding

---

**Next**: Learn about Azure Traffic Manager for intelligent traffic distribution →

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-canary-releases-dark-launching/2-explore-canary-releases)
