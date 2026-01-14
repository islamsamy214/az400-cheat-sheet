# Explore CI/CD with Deployment Rings

**Duration**: 3 minutes

Ring-based deployment architecture for progressive exposure and controlled rollout validation.

---

## Definition

**Progressive exposure deployment** (alternatively designated **ring-based deployment**) originated from **Jez Humble's foundational continuous delivery methodology**, establishing architectural patterns for production-first operational strategies.

Ring-based architectures **minimize end-user impact through incremental validation**, deploying changes across progressively broader user cohorts while maintaining production stability guarantees.

---

## Ring-Based Deployment Fundamentals

### Core Concept

**Rings**: Concentric circles of progressively larger user populations, where each ring validates deployment before promotion to next broader audience.

```
┌───────────────────────────────────────────────────────────┐
│                    Ring 3: General Availability            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │            Ring 2: Broad Deployment                 │  │
│  │  ┌───────────────────────────────────────────────┐  │  │
│  │  │       Ring 1: Early Adopters                  │  │  │
│  │  │  ┌─────────────────────────────────────────┐  │  │  │
│  │  │  │    Ring 0: Internal / Canary           │  │  │  │
│  │  │  │         Smallest audience              │  │  │  │
│  │  │  │     Most risk-tolerant                 │  │  │  │
│  │  │  └─────────────────────────────────────────┘  │  │  │
│  │  │           ↓ Validate & Promote               │  │  │
│  │  └───────────────────────────────────────────────┘  │  │
│  │               ↓ Validate & Promote                 │  │
│  └─────────────────────────────────────────────────────┘  │
│                   ↓ Validate & Activate                   │
└───────────────────────────────────────────────────────────┘
```

### Key Characteristics

**1. Graduated Exposure**
- Start with smallest, most risk-tolerant cohort (Ring 0)
- Progressively expand to broader audiences after validation
- Each ring serves as validation gate for next ring

**2. Production-First Strategy**
- All rings deploy to **production** (not staging)
- Real user data and behavior inform validation
- Synthetic testing supplemented by actual usage patterns

**3. Automated Health Validation**
- Blast radius assessment at each ring
- Observability metrics, automated testing, telemetry analysis
- User feedback aggregation quantifies deployment risk

**4. Controlled Progression**
- Automated promotion criteria govern ring transitions
- Health validation gates between rings
- Deployment halts on failures (prevent cascade)

---

## Ring Architecture Design

### Ring 0: Internal / Canary

**Purpose**: First production validation with most risk-tolerant users

**Audience**:
- Internal organizational users (employees, QA teams)
- Controlled validation environments
- Direct engineering feedback channels

**Size**: 0.1-1% of total user base (100-10,000 users)

**Duration**: 24-48 hours minimum

**Validation Criteria**:
- ✅ No critical errors (error rate < 0.1%)
- ✅ Performance metrics stable (response time, throughput)
- ✅ Core functionality operational
- ✅ Internal user feedback positive

**Benefits**:
- Minimal external impact (internal users only)
- Rapid feedback from technical audience
- Quick rollback if issues detected
- Production data validation (real infrastructure)

### Ring 1: Early Adopters / Limited Production

**Purpose**: Expand validation to limited external user cohorts

**Audience**:
- Beta testers, early adopters (opted-in)
- Less risk-averse user segments
- Geographic/demographic subsets

**Size**: 1-10% of total user base (10K-100K users)

**Duration**: 3-7 days

**Validation Criteria**:
- ✅ Ring 0 validation passed
- ✅ External user feedback positive
- ✅ Business metrics stable (conversion, engagement)
- ✅ Support ticket volume normal
- ✅ No new critical issues

**Benefits**:
- Real-world user behavior patterns
- Diverse usage scenarios validated
- Business metric impact measured
- Sufficient sample size for statistical analysis

### Ring 2: Broad Deployment

**Purpose**: Near-universal deployment with minimal remaining risk

**Audience**:
- General user population (majority)
- All geographic regions and user segments
- Production-ready validation complete

**Size**: 50-90% of total user base (millions of users)

**Duration**: 7-14 days

**Validation Criteria**:
- ✅ Ring 1 validation passed
- ✅ No degradation at scale (infrastructure capacity)
- ✅ Business metrics improving or stable
- ✅ Monitoring dashboards healthy
- ✅ Operational runbooks validated

**Benefits**:
- Scale testing with real load
- Final validation before universal activation
- Comprehensive monitoring data collected
- Operational confidence established

### Ring 3: General Availability (GA)

**Purpose**: Universal feature activation

**Audience**:
- 100% of user base
- No exclusions or limitations

**Activation Method**:
- **Feature toggle state transition** (instant activation)
- No additional deployment required
- Code already deployed via Ring 0-2 progression

**Validation**:
- ✅ All previous rings validated
- ✅ Monitoring confirms universal stability
- ✅ Business impact positive or neutral
- ✅ Rollback plan ready (toggle off if needed)

---

## Microsoft Windows Ring Implementation

### Enterprise-Scale Example

Microsoft Windows engineering demonstrates **enterprise-scale ring implementations** managing global user base deployments (billions of devices).

**Windows Deployment Rings**:

```
┌──────────────────────────────────────────────────────────────┐
│  Ring 0: Microsoft Employees (Internal)                      │
│  • 100,000+ devices                                          │
│  • Daily validation builds                                   │
│  • Telemetry-rich environment                                │
│  • Rapid feedback to engineering                             │
├──────────────────────────────────────────────────────────────┤
│  Ring 1: Windows Insiders (Early Adopters)                   │
│  • 10 million+ opted-in users                                │
│  • Weekly builds during development                          │
│  • Diverse hardware/software configurations                  │
│  • Community feedback channels                               │
├──────────────────────────────────────────────────────────────┤
│  Ring 2: Broad Deployment (General Users)                    │
│  • 500 million+ devices                                      │
│  • Monthly feature updates                                   │
│  • Standard consumer/enterprise PCs                          │
│  • Automated telemetry and error reporting                   │
├──────────────────────────────────────────────────────────────┤
│  Ring 3: Critical Systems (Delayed/Cautious)                 │
│  • 200 million+ devices                                      │
│  • Delayed rollout (additional 1-3 months)                   │
│  • Mission-critical systems (healthcare, finance, gov)       │
│  • Maximum stability requirement                             │
└──────────────────────────────────────────────────────────────┘
```

### Windows Ring Strategy

**Ring 0 (Microsoft Internal)**:
- **Purpose**: First validation on production-equivalent infrastructure
- **Cadence**: Daily builds during active development
- **Validation**: Automated test suites + manual exploratory testing
- **Feedback**: Direct bug reports to engineering teams
- **Impact**: Internal productivity impact acceptable (risk-tolerant)

**Ring 1 (Windows Insiders)**:
- **Purpose**: Diverse hardware/software configuration validation
- **Cadence**: Weekly builds (preview channel)
- **Validation**: Millions of device configurations tested
- **Feedback**: Community forums, feedback hub, telemetry
- **Impact**: Opted-in users accept instability (early adopter psychology)

**Ring 2 (Broad Deployment)**:
- **Purpose**: General user population rollout
- **Cadence**: Monthly feature updates (production release)
- **Validation**: Comprehensive quality gates passed
- **Feedback**: Automated telemetry, error reports, support tickets
- **Impact**: High stability requirement (general consumer expectations)

**Ring 3 (Critical Systems)**:
- **Purpose**: Maximum stability for mission-critical deployments
- **Cadence**: Delayed rollout after Ring 2 proves stable
- **Validation**: Extended monitoring period (1-3 months)
- **Feedback**: Enterprise IT feedback, deployment success metrics
- **Impact**: Zero tolerance for instability (critical infrastructure)

### Business Impact

**Benefits**:
- **99.9% quality before GA**: Ring 0-2 catch 99.9% of issues before broad deployment
- **Reduced support costs**: Fewer critical issues reach general users
- **Faster innovation**: Confidence to ship new features continuously
- **User trust**: Reliable updates maintain Windows ecosystem trust

**Metrics**:
- Ring 0 issues: ~10,000 bugs found/fixed per release
- Ring 1 issues: ~1,000 bugs found/fixed per release  
- Ring 2 issues: ~100 bugs found/fixed per release
- Ring 3 issues: <10 bugs reported per release

**Result**: Progressive validation reduces blast radius by **99.99%** (10,000 → <10 issues at GA)

---

## Ring-Based vs. Canary Deployment

### Comparison

| Aspect | Canary Deployment | Ring-Based Deployment |
|--------|-------------------|----------------------|
| **Stages** | 2 stages (canary + stable) | 3-5+ rings (graduated) |
| **Expansion** | Gradual percentage increase (1% → 100%) | Discrete ring progression (Ring 0 → GA) |
| **Audience** | Random percentage of users | Cohort-based segmentation |
| **Validation** | Single validation gate (health metrics) | Multiple validation gates (per ring) |
| **Duration** | Days to weeks | Weeks to months |
| **Use Case** | Quick validation, risk mitigation | Large-scale, mission-critical rollouts |
| **Automation** | Automated percentage expansion | Automated ring promotion |

### Relationship

**Ring-based deployment fundamentally extends canary deployment patterns**, implementing **multiple graduated exposure stages** rather than single canary validation phases.

**Think of it as**: "Canary on steroids" with 3-5+ progressive validation checkpoints instead of just one canary cohort.

**Example Evolution**:
```
Canary Deployment:
  Canary (5%) → Expand (10% → 25% → 50%) → Full (100%)

Ring-Based Deployment:
  Ring 0 (Internal) → Ring 1 (Early) → Ring 2 (Broad) → GA (100%)
      ↓                  ↓                 ↓                ↓
  Validate            Validate          Validate       Activate
  (48h)               (7 days)          (14 days)      (instant)
```

---

## DevOps Pipeline Integration

### Ring Implementation in CI/CD

Rings typically implemented as **distinct deployment stages** in Azure DevOps/GitHub Actions pipelines, with **automated promotion criteria** governing progression between audience tiers.

**Azure Pipelines Example**:

```yaml
stages:
  # Build stage (common)
  - stage: Build
    jobs:
      - job: BuildApp
        steps:
          - script: dotnet build
          - script: dotnet test
          - task: PublishBuildArtifacts

  # Ring 0: Internal deployment
  - stage: Ring0_Internal
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: DeployRing0
        environment: production-ring0
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'prod-connection'
                    appName: 'myapp-ring0'
                    slot: 'ring0'
      
      # Health validation gate (24-48h)
      - job: ValidateRing0
        dependsOn: DeployRing0
        steps:
          - task: InvokeRESTAPI@1
            displayName: 'Check Application Insights'
            inputs:
              urlSuffix: '/v1/apps/$(appId)/metrics/errors'
              waitForCompletion: 'false'
          
          - script: |
              # Check error rate < 0.1%
              if [ $(ERROR_RATE) -gt 0.001 ]; then
                echo "Error rate too high: $(ERROR_RATE)"
                exit 1
              fi
              
              # Check response time < 500ms
              if [ $(AVG_RESPONSE_TIME) -gt 500 ]; then
                echo "Response time degraded: $(AVG_RESPONSE_TIME)ms"
                exit 1
              fi
              
              echo "Ring 0 validation passed ✅"

  # Ring 1: Early adopters
  - stage: Ring1_EarlyAdopters
    dependsOn: Ring0_Internal
    condition: succeeded()
    jobs:
      - deployment: DeployRing1
        environment: production-ring1
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    appName: 'myapp-ring1'
                    slot: 'ring1'
      
      # Health validation gate (3-7 days)
      - job: ValidateRing1
        dependsOn: DeployRing1
        steps:
          - script: |
              # Check business metrics
              python validate_ring.py \
                --ring ring1 \
                --duration 7d \
                --metrics conversion_rate,engagement_time \
                --thresholds 0.95,0.95  # 95% of baseline

  # Ring 2: Broad deployment
  - stage: Ring2_BroadDeployment
    dependsOn: Ring1_EarlyAdopters
    condition: succeeded()
    jobs:
      - deployment: DeployRing2
        environment: production-ring2
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    appName: 'myapp-ring2'
      
      # Health validation gate (7-14 days)
      - job: ValidateRing2
        dependsOn: DeployRing2
        steps:
          - script: |
              # Scale validation
              python validate_ring.py \
                --ring ring2 \
                --duration 14d \
                --scale-test true

  # Ring 3: General Availability
  - stage: Ring3_GeneralAvailability
    dependsOn: Ring2_BroadDeployment
    condition: succeeded()
    jobs:
      - deployment: ActivateGA
        environment: production-ga
        strategy:
          runOnce:
            deploy:
              steps:
                # Feature toggle activation (instant, no deployment)
                - task: AzureAppConfiguration@4
                  inputs:
                    azureSubscription: 'prod-connection'
                    AppConfigurationEndpoint: '$(AppConfigEndpoint)'
                    KeyValueFilter: 'myfeature.enabled'
                    Value: 'true'
                
                - script: |
                    echo "Feature activated for 100% of users via feature toggle ✅"
```

### Release Gates

**Post-Deployment Release Gates** implement ring-specific health validation criteria enabling automated progression workflows advancing deployments following stability confirmation periods.

**Example Health Validation Gates**:

**Ring 0 → Ring 1 Gate**:
```yaml
gates:
  - task: QueryWorkItems@0
    displayName: 'Check for critical bugs'
    inputs:
      maxThreshold: 0  # Zero critical bugs allowed
  
  - task: InvokeRESTAPI@1
    displayName: 'Check Application Insights'
    inputs:
      method: 'GET'
      urlSuffix: '/v1/apps/$(appId)/metrics/errors'
      successCriteria: 'lt(root.errors.count, 10)'  # < 10 errors
  
  - task: ManualValidation@0
    displayName: 'Product team approval'
    inputs:
      notifyUsers: 'product-team@company.com'
      instructions: 'Review Ring 0 telemetry and approve Ring 1 promotion'
```

**Ring 1 → Ring 2 Gate**:
```yaml
gates:
  - task: QueryAzureMonitor@1
    displayName: 'Check conversion rate'
    inputs:
      azureSubscription: 'prod-connection'
      resourceGroupName: 'myapp-rg'
      resourceType: 'Microsoft.Insights/components'
      metricName: 'customMetrics/conversion_rate'
      timeRange: '7d'
      aggregation: 'Average'
      threshold: 0.95  # 95% of baseline minimum
```

### Automated Rollback

**Health validation failures** trigger **automatic deployment halts** preventing cascading impact across subsequent rings, minimizing blast radius through controlled containment strategies.

**Rollback Trigger Example**:
```yaml
- job: MonitorRing0
  steps:
    - script: |
        # Continuous monitoring (every 5 minutes for 24h)
        while [ $ELAPSED_TIME -lt 86400 ]; do
          ERROR_RATE=$(get_error_rate)
          
          if [ $ERROR_RATE -gt 0.005 ]; then
            echo "ERROR: Ring 0 error rate exceeded threshold"
            echo "Triggering automated rollback..."
            
            # Rollback deployment
            az webapp deployment slot swap \
              --resource-group myapp-rg \
              --name myapp \
              --slot ring0 \
              --target-slot previous
            
            # Disable feature toggle
            az appconfig kv set \
              --endpoint $(AppConfigEndpoint) \
              --key myfeature.enabled \
              --value false
            
            # Alert engineering team
            send_alert "Ring 0 deployment failed validation. Automated rollback executed."
            
            exit 1
          fi
          
          sleep 300  # 5 minutes
          ELAPSED_TIME=$((ELAPSED_TIME + 300))
        done
        
        echo "Ring 0 validation passed after 24h ✅"
```

---

## Blast Radius Assessment

**Blast radius assessment** incorporates **observability metrics**, **automated testing validation**, **telemetry analysis**, and **user feedback aggregation** to quantify deployment risk exposure.

### Key Metrics by Ring

**Ring 0 (Internal)**:
- Error rate (4xx, 5xx responses)
- Response time (p50, p95, p99)
- Throughput (requests/second)
- Internal user feedback (direct reports)

**Ring 1 (Early Adopters)**:
- All Ring 0 metrics +
- Conversion rate (primary business metric)
- User engagement (session duration, feature usage)
- Support ticket volume
- User feedback sentiment analysis

**Ring 2 (Broad Deployment)**:
- All Ring 1 metrics +
- Infrastructure capacity (CPU, memory, network)
- Cost metrics (cloud spend)
- Regional performance (geographic distribution)
- Operational runbook validation

**Ring 3 (General Availability)**:
- All Ring 2 metrics +
- Business impact (revenue, retention)
- User satisfaction (NPS, CSAT)
- Competitive positioning
- Ecosystem health (partner integrations)

### Blast Radius Quantification

**Formula**: Blast Radius = (Users Affected) × (Impact Severity) × (Duration)

**Example Comparison**:

| Scenario | Users Affected | Severity | Duration | Blast Radius |
|----------|----------------|----------|----------|--------------|
| **Traditional (100% deploy)** | 10M | High | 2h | 20M user-hours |
| **Canary (5% initial)** | 500K | High | 30min | 250K user-hours |
| **Ring 0 (internal)** | 10K | High | 15min | 2.5K user-hours |

**Result**: Ring-based deployment reduces blast radius by **99.99%** vs. traditional deployment (20M → 2.5K user-hours).

---

## User Cohort Segmentation

**Ring architecture design requires user cohort segmentation analysis** identifying appropriate deployment boundaries and risk tolerance characteristics justifying incremental rollout investment.

### Segmentation Strategies

**1. User Type**
- Internal employees (Ring 0)
- Beta testers / early adopters (Ring 1)
- General users (Ring 2)
- Enterprise/critical users (delayed ring)

**2. Geographic Region**
- Low-traffic regions first (e.g., New Zealand)
- High-traffic regions last (e.g., US, Europe)

**3. Device/Platform**
- Mobile iOS (Ring 0)
- Mobile Android (Ring 1)
- Desktop web (Ring 2)

**4. User Behavior**
- Power users (early ring - more tolerant)
- Casual users (later ring - less tolerant)

**5. Business Tier**
- Free tier users (early ring)
- Premium users (later ring - higher SLA)

### Cohort Assignment Methods

**1. Opt-In**
- Users volunteer for early access
- Example: Windows Insiders program

**2. Geographic**
- Deploy to specific regions
- Example: Launch in APAC first, then EU, then Americas

**3. Random Sampling**
- Random subset of users (hash-based)
- Example: User ID hash % 100 < 10 → Ring 1 (10%)

**4. Behavioral**
- Users exhibiting specific behavior patterns
- Example: High-engagement users → Ring 1

**5. Organizational**
- Internal company employees
- Example: All @company.com emails → Ring 0

---

## Feature Toggle Integration

Organizations implementing canary methodologies frequently establish **multiple deployment slots** representing distinct ring tiers supporting graduated exposure strategies.

**General availability release** activates features universally, typically implemented through **feature toggle state transitions** enabling **instantaneous activation** without additional deployment operations.

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Feature Toggle Service                      │
│         (Azure App Configuration / LaunchDarkly)         │
└────────────────────┬────────────────────────────────────┘
                     │
          ┌──────────┴──────────┐
          │                     │
    ┌─────▼─────┐         ┌─────▼─────┐
    │   Ring 0  │         │   Ring 1  │
    │  Enabled  │         │  Enabled  │
    └─────┬─────┘         └─────┬─────┘
          │                     │
          └──────────┬──────────┘
                     │
          ┌──────────▼──────────┐
          │      Ring 2-3       │
          │   Disabled (wait)   │
          └─────────────────────┘

After Ring 2 validation passes:
          │
          ▼
  Toggle all rings → Enabled
  (instant activation, no deployment)
```

### Feature Toggle Configuration Example

```json
{
  "feature_flags": {
    "new_checkout_flow": {
      "enabled": true,
      "conditions": {
        "ring0": {
          "enabled": true,
          "users": ["@company.com"]
        },
        "ring1": {
          "enabled": true,
          "percentage": 10,
          "targeting": {
            "groups": ["early_adopters", "beta_testers"]
          }
        },
        "ring2": {
          "enabled": true,
          "percentage": 90
        },
        "ga": {
          "enabled": false  // Will flip to true after Ring 2 validation
        }
      }
    }
  }
}
```

**Benefits**:
- **Instant GA activation**: Flip toggle without redeployment (< 1 second)
- **Instant rollback**: Disable feature if issues arise (< 1 second)
- **Independent ring control**: Enable/disable per ring
- **A/B testing integration**: Can run experiments within rings

---

## Key Takeaways

### Core Principles
1. 🎯 **Progressive exposure**: Deploy across graduated user cohorts (Ring 0 → GA)
2. 🔒 **Blast radius containment**: Each ring limits failure impact
3. 📊 **Production-first validation**: All rings use real production data
4. ✅ **Automated health gates**: Health validation governs ring progression
5. 🚫 **Automated halt**: Failures prevent cascade to subsequent rings
6. 🔄 **Feature toggle activation**: GA = instant activation without deployment

### Implementation Strategies
- **Ring 0**: Internal users, 0.1-1%, 24-48h validation
- **Ring 1**: Early adopters, 1-10%, 3-7 days validation
- **Ring 2**: Broad deployment, 50-90%, 7-14 days validation
- **Ring 3/GA**: Feature toggle activation, 100%, instant

### Success Metrics
- **Blast radius reduction**: 99.99% vs traditional deployment
- **Quality improvement**: 99.9% issues caught before GA
- **Confidence building**: Multiple validation checkpoints
- **Business continuity**: Minimal disruption to user experience

---

## Additional Resources

- [Progressively expose your releases using deployment rings - Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/migrate/phase-rollout-with-rings)
- [What is Continuous Delivery? - Azure DevOps](https://learn.microsoft.com/en-us/devops/deliver/what-is-continuous-delivery)
- [Azure App Configuration feature management](https://learn.microsoft.com/en-us/azure/azure-app-configuration/concept-feature-management)

---

**Next**: Test your knowledge with the module assessment →

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-test-progressive-exposure-deployment/3-explore-ci-cd-deployment-rings)
