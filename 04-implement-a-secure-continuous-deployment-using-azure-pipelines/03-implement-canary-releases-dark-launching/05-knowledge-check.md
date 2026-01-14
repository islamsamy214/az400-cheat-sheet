# Module Assessment

**Duration**: 5 minutes

Test your understanding of canary releases, Azure Traffic Manager, and dark launching concepts.

---

## Question 1: Canary Release Origin

**What is the origin of the term "canary release"?**

A) A type of bird known for its bright color  
B) Historical mining practice using canaries to detect toxic gas  
C) A software development methodology from the Canary Islands  
D) A code name for early Microsoft deployment strategies

<details>
<summary>✅ <strong>Answer</strong></summary>

**B) Historical mining practice using canaries to detect toxic gas**

### Explanation
Canary release terminology originates from historical coal mining practices where canaries served as **early warning systems** for toxic gas detection. Canaries demonstrated heightened sensitivity to hazardous atmospheric conditions, succumbing to toxic exposure before human miners, providing critical escape time from lethal environments. This early detection principle applies to software deployment: deploy to small user cohort (canaries) who detect issues before broader rollout.

### Why Other Answers Are Wrong
- **A) Bright color**: ❌ While canaries are yellow, the term relates to their sensitivity to toxins, not their appearance.
- **C) Canary Islands**: ❌ No connection to the geographic location. The term comes from the bird's use in mines.
- **D) Microsoft code name**: ❌ Not a Microsoft-specific term. It's an industry-wide deployment pattern named after the mining practice.

### Key Concept
**Early detection principle**: Small indicator (canary users) detects danger (bugs/performance issues) before mass exposure (full rollout).
</details>

---

## Question 2: Canary Release Percentage

**What is the typical starting percentage for a canary release?**

A) 25-50% of users  
B) 1-5% of users  
C) 50-75% of users  
D) 10-20% of users

<details>
<summary>✅ <strong>Answer</strong></summary>

**B) 1-5% of users**

### Explanation
Canary releases typically start with **1-5% of users** to minimize blast radius during initial validation. This constrained impact scope allows problem identification through limited user exposure before comprehensive rollout. The progression typically follows: 1% → 5% → 10% → 25% → 50% → 100% over days/weeks, with 24-48 hour monitoring periods at each stage.

### Why Other Answers Are Wrong
- **A) 25-50%**: ❌ Too aggressive for initial canary. This is mid-rollout percentage after successful 1-5% validation.
- **C) 50-75%**: ❌ This is late-stage rollout, not canary phase. Defeats purpose of limited exposure.
- **D) 10-20%**: ❌ This is early expansion phase, not initial canary. Start smaller to minimize risk.

### Typical Canary Progression
```
Day 1:  1% (canary)
Day 3:  5% (expanded canary)
Day 5:  10% (early adopters)
Day 7:  25% (wider rollout)
Day 10: 50% (majority)
Day 14: 100% (full rollout)
```

### Key Principle
**Start small, expand gradually**. Limited initial exposure constrains potential impact during validation.
</details>

---

## Question 3: Azure Traffic Manager Routing

**Which Azure Traffic Manager routing method is most appropriate for canary releases?**

A) Priority routing  
B) Weighted distribution  
C) Performance-based routing  
D) Geographic routing

<details>
<summary>✅ <strong>Answer</strong></summary>

**B) Weighted distribution**

### Explanation
**Weighted distribution methodology predominates** in Continuous Delivery implementations, enabling **percentage-based traffic allocation** essential for canary releases. Configure weights like 90 (stable) and 10 (canary) to route 10% of traffic to new version, then adjust weights gradually (75/25, 50/50, 0/100) as canary proves stable.

### Configuration Example
```bash
# Stable endpoint: 90% weight
az network traffic-manager endpoint create \
  --name stable --weight 90

# Canary endpoint: 10% weight
az network traffic-manager endpoint create \
  --name canary --weight 10

# Result: 90% → stable, 10% → canary
```

### Why Other Answers Are Wrong
- **A) Priority routing**: ❌ All-or-nothing failover (primary → backup), not gradual percentage allocation. Can't do 10% canary.
- **C) Performance-based routing**: ❌ Routes based on latency (geographic proximity), not controlled percentage. Can't control canary percentage.
- **D) Geographic routing**: ❌ Routes by region (Europe → EU endpoint), not percentage. Can't do 10% canary across all regions.

### Key Capability
Weighted distribution enables **configurable proportional traffic allocation** across endpoints for gradual rollout scenarios.
</details>

---

## Question 4: Dark Launching vs. Canary

**What is the primary difference between dark launching and canary releases?**

A) Dark launching uses Azure Traffic Manager, canary releases don't  
B) Dark launching hides feature results from users, canary releases show them  
C) Dark launching is slower than canary releases  
D) Dark launching only works for backend services

<details>
<summary>✅ <strong>Answer</strong></summary>

**B) Dark launching hides feature results from users, canary releases show them**

### Explanation
**Dark launching** runs new features in "shadow mode" where they **process production data but results are hidden** from users (discarded or logged only). **Canary releases** expose new features to users who **consciously interact** with them. Both use real production data, but dark launching enables validation **without user-facing exposure**, making it zero-risk.

### Comparison Table
| Aspect | Dark Launching | Canary Release |
|--------|---------------|----------------|
| **User Visibility** | ❌ Hidden | ✅ Visible |
| **Results Display** | Discarded/logged | Shown to users |
| **User Awareness** | Unconscious | Conscious interaction |
| **Risk Level** | Very Low (no impact) | Low (limited impact) |
| **Purpose** | Infrastructure/performance | Feature adoption |

### Why Other Answers Are Wrong
- **A) Traffic Manager usage**: ❌ Both can use Traffic Manager. The difference is result visibility, not tooling.
- **C) Speed difference**: ❌ Both can be fast or slow. Speed isn't the differentiator.
- **D) Backend only**: ❌ Dark launching works for frontend too (e.g., shadow UI rendering without display).

### Example: Dark Launch
```python
# User searches
results_old = legacy_search(query)  # Production
results_new = new_search(query)     # Dark (hidden)
log_comparison(results_old, results_new)
return results_old  # Only show old results
```
</details>

---

## Question 5: Traffic Manager Health Checks

**What happens when an endpoint fails health checks in Azure Traffic Manager?**

A) Traffic Manager shuts down all endpoints  
B) The endpoint continues receiving 50% of traffic  
C) The endpoint is automatically excluded from routing  
D) Traffic Manager sends alerts but continues routing

<details>
<summary>✅ <strong>Answer</strong></summary>

**C) The endpoint is automatically excluded from routing**

### Explanation
Traffic routing **exclusively targets available endpoints**, automatically **excluding unhealthy or unreachable services** from distribution algorithms. When an endpoint fails health checks, Traffic Manager marks it as "Stopped" and removes it from the routing pool. All traffic redirects to healthy endpoints only.

### Endpoint Health States
```
✅ Healthy → Included in routing (receives traffic)
⚠️ Degraded → Included with warning (receives traffic)
❌ Stopped → Excluded from routing (NO traffic)
🔄 Checking → Temporary exclusion (probing)
```

### Example Scenario
```
Endpoints:
- app-v1.azure.net: Healthy (weight 90)
- app-v2.azure.net: Stopped (weight 10)

Traffic Distribution:
- 100% → app-v1 (only healthy endpoint)
- 0% → app-v2 (excluded due to health check failure)
```

### Why Other Answers Are Wrong
- **A) Shuts down all endpoints**: ❌ Only the failed endpoint is affected. Other healthy endpoints continue serving traffic.
- **B) Continues receiving 50%**: ❌ Unhealthy endpoints receive 0% traffic (completely excluded).
- **D) Alerts but continues**: ❌ Traffic Manager excludes unhealthy endpoints immediately, doesn't continue routing to them.

### Health Check Configuration
```bash
# Configure health check
az network traffic-manager profile create \
  --monitor-protocol HTTPS \
  --monitor-path "/health" \
  --monitor-interval 10 \
  --monitor-timeout 5 \
  --monitor-tolerated-failures 2
```
</details>

---

## Question 6: SpaceX Dark Launching

**How did SpaceX apply dark launching principles to sensor validation?**

A) By testing sensors only in simulations  
B) By running new sensors alongside legacy sensors in production  
C) By deploying sensors to a small percentage of rockets  
D) By using canary releases for sensor updates

<details>
<summary>✅ <strong>Answer</strong></summary>

**B) By running new sensors alongside legacy sensors in production**

### Explanation
SpaceX exemplifies dark launching through **parallel sensor deployment strategies** where new sensor versions operate **alongside established implementations** during validation periods. **Dual sensor architectures** enable comprehensive telemetry collection from both legacy and candidate implementations, generating comparative datasets for validation analysis. Legacy sensor remains authoritative (controls flight), new sensor data is logged only (dark).

### Dual Sensor Architecture
```
Flight Control System:
├─ Legacy Sensor (Production Authority)
│  ├─ Temperature readings → Flight control
│  └─ Flight-critical decisions
│
└─ New Sensor (Dark Validation)
   ├─ Temperature readings → Logging only
   └─ Comparison analysis (no control decisions)

Validation Process:
1. Compare readings every 100ms
2. Alert if variance > threshold
3. Track accuracy over 1000+ flights
4. Replace legacy after proving reliability
```

### Why Other Answers Are Wrong
- **A) Simulations only**: ❌ SpaceX validated with **real production flights**, not just simulations. Dark launching uses real production data.
- **C) Small percentage of rockets**: ❌ This would be canary release. Dark launching runs on **same rocket** (parallel sensors).
- **D) Canary releases**: ❌ Canary would mean some rockets use new sensor for control. Dark launching means new sensor runs but doesn't control anything.

### Software Parallel
```python
# Legacy algorithm (production)
result_legacy = legacy_algorithm.process(data)

# New algorithm (dark)
result_new = new_algorithm.process(data)  # Shadow execution

# Compare and log (don't use new result)
log_comparison(result_legacy, result_new)

# Return legacy result only
return result_legacy
```

### Key Principle
**Performance parity validation** through parallel execution without risk (legacy remains authoritative).
</details>

---

## Question 7: Monitoring Canary Releases

**Which metric is MOST critical to monitor during a canary release?**

A) Number of users in canary cohort  
B) Error rate comparison between canary and stable  
C) Total lines of code deployed  
D) Number of commits in the release

<details>
<summary>✅ <strong>Answer</strong></summary>

**B) Error rate comparison between canary and stable**

### Explanation
**Error rate comparison** is the most critical metric because it directly indicates whether the canary version is **more buggy** than stable. Comprehensive monitoring generates **actionable telemetry** enabling data-driven continuation or rollback decisions. If canary error rate exceeds stable by threshold (e.g., +0.5%), rollback immediately.

### Critical Metrics to Monitor
**Performance Metrics** (Most Critical):
```
1. Error rate (4xx, 5xx responses)
   - Target: < 0.1% increase vs stable
   - Action: Rollback if > 0.5% increase

2. Response time (p50, p95, p99)
   - Target: < 10% degradation
   - Action: Alert if > 20% degradation

3. Throughput (requests/second)
   - Target: Equivalent to stable
   - Action: Investigate if > 15% drop
```

**Business Metrics**:
```
4. Conversion rate
5. User engagement
6. Revenue per user
```

### Automated Canary Analysis
```yaml
canaryAnalysis:
  duration: 2h
  metrics:
    - name: error_rate
      threshold: 0.5%  # Rollback if error rate > 0.5%
      action: rollback
    - name: response_time_p95
      threshold: 500ms
      action: rollback
  onFailure: rollback
  onSuccess: expand_to_25_percent
```

### Why Other Answers Are Wrong
- **A) Number of users**: ❌ Important for planning but not for **health monitoring**. You know the percentage (1-5%), need to monitor if it's working.
- **C) Lines of code**: ❌ Code metrics don't indicate runtime health. A 1-line change can cause 100% error rate.
- **D) Number of commits**: ❌ Development metric, not operational health indicator. Focus on runtime behavior.

### Decision Matrix
```
Error Rate Increase:
< 0.1%  → ✅ Continue (healthy)
0.1-0.5% → ⚠️ Monitor closely (warning)
> 0.5%  → ❌ Rollback immediately (unhealthy)
```
</details>

---

## Question 8: Dark Launching Use Case

**Which scenario is BEST suited for dark launching?**

A) Testing a new UI color scheme  
B) Validating a new payment processor's performance  
C) Gathering user feedback on a new feature  
D) A/B testing button placement

<details>
<summary>✅ <strong>Answer</strong></summary>

**B) Validating a new payment processor's performance**

### Explanation
Dark launching is **ideal for infrastructure/backend validation** like payment processors where you can run **parallel processing** (old + new) and compare results **without exposing users to risk**. Process payment with old processor (production), also process with new processor in **test mode** (dark), compare performance/accuracy, but only commit the old processor's transaction.

### Implementation Example
```python
def process_payment(order):
    # Old processor (production - real charge)
    result_old = legacy_processor.charge(order)
    
    # New processor (dark - test mode, no real charge)
    result_new = new_processor.charge_test_mode(order)
    
    # Compare performance
    log_comparison({
        'success_match': result_old.success == result_new.success,
        'time_old': result_old.duration_ms,
        'time_new': result_new.duration_ms,
        'improvement': result_old.duration_ms - result_new.duration_ms
    })
    
    # Return old result only (new is "dark")
    return result_old
```

### Why Other Answers Are Wrong
- **A) UI color scheme**: ❌ Users MUST see colors to provide feedback. Can't be "dark". Use canary release or A/B test instead.
- **C) User feedback**: ❌ Dark launching is **hidden from users** (no conscious awareness). Can't gather explicit feedback on invisible features. Use canary.
- **D) A/B testing buttons**: ❌ Users must **see** button placements. Use A/B testing (variant of canary), not dark launching.

### Dark Launching Ideal Use Cases
| Use Case | Why Ideal | Risk Level |
|----------|-----------|------------|
| **Payment processors** | Parallel processing possible | Very Low |
| **Database migrations** | Shadow reads/writes | Very Low |
| **Search algorithms** | Compare results without showing | Very Low |
| **Fraud detection** | Parallel scoring | Very Low |
| **Caching layers** | Write both, read one | Very Low |

### Key Principle
Dark launching is for **backend/infrastructure validation** where features can run **without user-facing exposure**.
</details>

---

## Question 9: Canary Release Rollback

**What is the primary advantage of canary releases for rollback scenarios?**

A) No rollback is ever needed  
B) Rollback affects only 1-5% of users instead of 100%  
C) Automatic rollback without human intervention  
D) Rollback completes within milliseconds

<details>
<summary>✅ <strong>Answer</strong></summary>

**B) Rollback affects only 1-5% of users instead of 100%**

### Explanation
Canary releases' primary rollback advantage is **limited blast radius**. If issues are detected, only the canary cohort (1-5% of users) experienced problems, not the entire user base (100%). Rollback is simply redirecting canary traffic back to stable version (instant via Traffic Manager weight adjustment).

### Comparison: Traditional vs. Canary
**Traditional Deployment**:
```
Deploy v2.0 to 100% of users
Issue detected → 100% of users affected 🔥
Rollback required → Redeploy v1.0 to 100% (30-60 minutes)
Impact: ALL users experience downtime
```

**Canary Release**:
```
Deploy v2.0 to 5% of users (canary)
Issue detected → Only 5% of users affected ⚠️
Rollback required → Redirect traffic (30 seconds)
Impact: Only 5% experience brief issue
```

### Rollback Process
```bash
# Issue detected in canary
# Rollback: Adjust Traffic Manager weights
az network traffic-manager endpoint update \
  --name stable --weight 100  # All traffic → stable

az network traffic-manager endpoint update \
  --name canary --weight 0    # Zero traffic → canary

# Rollback complete in ~30 seconds (DNS TTL)
```

### Why Other Answers Are Wrong
- **A) No rollback needed**: ❌ Issues still occur in canary versions. Canary minimizes impact, doesn't prevent bugs.
- **C) Automatic rollback**: ❌ Possible but not guaranteed. Requires automated canary analysis configuration. Manual rollback common.
- **D) Milliseconds**: ❌ DNS TTL affects speed (typically 5-60 seconds for Traffic Manager). Faster than redeployment (30-60 min) but not milliseconds.

### Impact Comparison
| Deployment Type | Users Affected | Rollback Time | Business Impact |
|-----------------|----------------|---------------|-----------------|
| **Traditional** | 100% | 30-60 minutes | High (all users) |
| **Blue-Green** | 100% | 30 seconds | Medium (all users briefly) |
| **Canary** | 1-5% | 30 seconds | Low (few users briefly) |

### Key Advantage
**Risk mitigation through limited exposure**. Canary users serve as "early warning system" before mass rollout.
</details>

---

## Question 10: Combining Patterns

**How can canary releases and dark launching be combined effectively?**

A) Use canary for frontend, dark launching for backend  
B) Dark launch first (0% visible), then canary (1% → 100%)  
C) Alternate between canary and dark launching weekly  
D) Use only one pattern at a time

<details>
<summary>✅ <strong>Answer</strong></summary>

**B) Dark launch first (0% visible), then canary (1% → 100%)**

### Explanation
Effective pattern combination: **Dark launch first** to validate infrastructure/performance with **zero user-facing risk** (shadow execution), then transition to **canary release** for user experience validation with **limited exposure** (1% → 5% → 100%). This two-phase approach minimizes risk at each stage.

### Combined Workflow
```
┌──────────────────────────────────────────────────────────┐
│ Phase 1: Dark Launch (Week 1-2)                         │
├──────────────────────────────────────────────────────────┤
│ - Deploy v2.0 with feature toggle (0% visible)          │
│ - Run shadow execution (20% of requests)                │
│ - Process real data, discard results                    │
│ - Validate: Performance, accuracy, resource usage       │
│ - Result: Backend proves stable ✅                      │
├──────────────────────────────────────────────────────────┤
│ Phase 2: Canary Release (Week 3-5)                      │
├──────────────────────────────────────────────────────────┤
│ - Week 3: Enable for 1% of users (visible)             │
│ - Week 4: Expand to 10% → 25%                          │
│ - Week 5: Expand to 50% → 100%                         │
│ - Validate: User experience, behavior, business metrics │
│ - Result: Full rollout with confidence ✅              │
└──────────────────────────────────────────────────────────┘
```

### Real-World Example: Search Engine Redesign
```python
# Week 1-2: Dark Launch
if random() < 0.20:  # 20% dark execution
    results_new = new_search_engine.search(query)  # Shadow
    log_comparison(results_legacy, results_new)
    # Don't show results_new (dark)

# Week 3: Canary (1%)
if user_in_canary_cohort(percentage=1):
    return new_search_engine.search(query)  # Visible!
else:
    return legacy_search_engine.search(query)

# Week 4-5: Expand canary (10% → 25% → 100%)
```

### Why Other Answers Are Wrong
- **A) Frontend/backend split**: ❌ Oversimplified. Dark launching works for both frontend (shadow UI rendering) and backend. Sequential combination (B) is more strategic.
- **C) Alternate weekly**: ❌ Doesn't make sense. They serve different purposes (validation vs. exposure). Use sequentially, not alternately.
- **D) Only one at a time**: ❌ Missing opportunity. Combining them provides **layered risk mitigation** (validate backend first, then expose to users).

### Benefits of Sequential Combination
1. **Dark launch** validates infrastructure (0% risk)
2. **Canary** validates user experience (1-5% risk)
3. **Gradual expansion** builds confidence (10% → 100%)
4. **Two-layer validation** catches issues at appropriate stage

### Pattern Selection Guide
```
New feature deployment:
├─ Step 1: Dark launch (shadow execution)
│  └─ Validate: Performance, accuracy, scalability
│     ├─ Pass → Step 2
│     └─ Fail → Fix and retry Step 1
│
├─ Step 2: Canary release (1-5% users)
│  └─ Validate: User experience, business metrics
│     ├─ Pass → Step 3
│     └─ Fail → Rollback, fix, retry Step 1
│
└─ Step 3: Gradual expansion (10% → 100%)
   └─ Monitor continuously, expand incrementally
```
</details>

---

## Score Interpretation

### 9-10 Correct (Excellent) 🏆
Outstanding! You have excellent understanding of canary releases, Azure Traffic Manager, and dark launching strategies. Ready for production implementation.

### 7-8 Correct (Good) ✅
Good grasp of core concepts. Review missed questions focusing on:
- Traffic Manager weighted routing configuration
- Dark launching vs. canary release differences
- Monitoring metrics and rollback criteria

### 5-6 Correct (Fair) ⚠️
Fair understanding. Re-read the module focusing on:
- Canary release implementation workflow (1% → 100%)
- Azure Traffic Manager routing methods
- Dark launching use cases and patterns

### <5 Correct (Needs Review) 📚
Review the module completely. Key areas:
- Canary release fundamentals (Units 2-3)
- Traffic Manager configuration (Unit 3)
- Dark launching strategies (Unit 4)
- Pattern comparison tables and examples

---

**Next**: Review key takeaways in the module summary →

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-canary-releases-dark-launching/5-knowledge-check)
