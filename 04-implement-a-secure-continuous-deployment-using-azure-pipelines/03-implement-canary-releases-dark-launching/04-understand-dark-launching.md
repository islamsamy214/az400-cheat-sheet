# Understand Dark Launching

**Duration**: 4 minutes

## Overview

**Dark launching** shares conceptual similarities with canary release methodologies while emphasizing distinct validation objectives and exposure characteristics.

Primary differentiation focuses on **frontend user response assessment** and behavioral analysis rather than backend performance validation, enabling user experience evaluation **without explicit feature visibility**.

## Dark Launching vs. Canary Releases

| Aspect | Dark Launching | Canary Release |
|--------|---------------|----------------|
| **User Visibility** | ❌ Hidden (users don't see feature) | ✅ Visible (users interact with feature) |
| **Primary Focus** | Backend validation, performance | User experience, behavior |
| **Data Processing** | Real production data | Real production data |
| **User Awareness** | 🕶️ Implicit (unconscious) | 👥 Explicit (conscious interaction) |
| **Results Display** | Discarded or shadow-logged | Shown to users |
| **Purpose** | Infrastructure/performance validation | Feature adoption validation |
| **Risk Level** | Very Low (no user impact) | Low (limited user impact) |

## Core Concept

Progressive exposure strategies target **limited user cohorts** for controlled feature exposure, maintaining minimal deployment scope during validation phases.

**Implicit testing methodologies** operate without explicit user notification, leveraging unconscious interaction patterns to generate authentic behavioral telemetry.

Feature opacity during validation justifies the **"dark" launching nomenclature** — features run in the background, invisible to users.

## Dark Launching Strategies

### Strategy 1: Shadow Mode Execution
Deploy feature that **processes real production data** but **discards results** (doesn't show to users).

**Example**: New recommendation algorithm
```python
# User requests product recommendations
def get_recommendations(user_id):
    # Production algorithm (visible to user)
    recommendations_v1 = legacy_algorithm.recommend(user_id)
    
    # Dark launch: New algorithm (hidden from user)
    recommendations_v2 = new_algorithm.recommend(user_id)  # Shadow execution
    
    # Log comparison for analysis (don't show v2 to user)
    log_comparison(recommendations_v1, recommendations_v2)
    
    # Return only v1 to user (v2 is "dark")
    return recommendations_v1
```

**Benefits**:
- ✅ Validate new algorithm with real data
- ✅ Compare results (old vs new)
- ✅ Zero risk to user experience
- ✅ Performance impact measurement

### Strategy 2: Parallel Processing
Run **both old and new implementations simultaneously**, compare outputs, display only old results.

**Example**: Payment processing
```python
def process_payment(order):
    # Old payment processor (live)
    result_old = legacy_processor.charge(order)
    
    # New payment processor (dark)
    result_new = new_processor.charge_test_mode(order)  # Test mode (no real charge)
    
    # Compare results
    if result_old.success != result_new.success:
        alert("Payment processor divergence detected!")
        log_divergence(result_old, result_new)
    
    # Return old result (new processor is "dark")
    return result_old
```

### Strategy 3: Hidden Feature Deployment
Deploy UI feature **hidden behind feature flag** at 0% rollout, but **code executes** for telemetry collection.

**Example**: New checkout flow
```python
def checkout(cart, user):
    # Legacy checkout (visible)
    result_legacy = legacy_checkout_flow(cart)
    
    # New checkout (dark - code runs but UI hidden)
    if random() < 0.10:  # 10% dark execution
        result_new = new_checkout_flow(cart, dry_run=True)  # Dry run mode
        
        # Measure performance
        log_performance({
            'legacy_time': result_legacy.duration_ms,
            'new_time': result_new.duration_ms,
            'potential_improvement': result_legacy.duration_ms - result_new.duration_ms
        })
    
    # Show only legacy to user
    return result_legacy
```

## Real-World Example: SpaceX Sensor Validation

**SpaceX exemplifies dark launching principles** through sensor validation methodologies documented in organizational literature describing Agile development adoption.

### Aerospace Engineering Application

**Parallel sensor deployment strategies**: New sensor versions operate **alongside established implementations** during validation periods.

```
┌─────────────────────────────────────────────────────────┐
│ Dual Sensor Architecture                                │
├─────────────────────────────────────────────────────────┤
│ Legacy Sensor (Production)    New Sensor (Dark)         │
│ ├─ Temperature reading        ├─ Temperature reading    │
│ ├─ Used for control systems   ├─ Logged for comparison  │
│ └─ Flight-critical            └─ Validation only        │
│                                                          │
│ Comparison Engine:                                       │
│ - Compare readings every 100ms                           │
│ - Alert if variance > 0.5°C                             │
│ - Track accuracy over 1000+ flights                     │
└─────────────────────────────────────────────────────────┘
```

**Validation Process**:
1. **Deploy new sensor alongside legacy**
2. **Collect telemetry from both** (compare readings)
3. **Legacy sensor remains authoritative** (controls flight systems)
4. **New sensor data logged** (not used for control)
5. **Performance parity validation** over multiple flights
6. **Replace legacy sensor** after proving reliability

**Result**: Zero risk during validation (legacy sensor always controls), full confidence before replacement.

## Software Implementation Pattern

Software implementations **mirror aerospace validation patterns** through parallel execution architectures where new features process production data and calculations **without user-facing exposure**, generating validation telemetry through shadow operation.

### Implementation Example: E-commerce Search

```python
class SearchService:
    def search(self, query, user_id):
        # Legacy search (production)
        start_legacy = time.now()
        results_legacy = self.legacy_search_engine.search(query)
        time_legacy = time.now() - start_legacy
        
        # New search (dark launch - 20% of requests)
        if self.should_dark_launch(user_id, percentage=20):
            start_new = time.now()
            results_new = self.new_search_engine.search(query)  # Shadow execution
            time_new = time.now() - start_new
            
            # Compare and log
            self.compare_search_results({
                'query': query,
                'legacy_count': len(results_legacy),
                'new_count': len(results_new),
                'legacy_time_ms': time_legacy,
                'new_time_ms': time_new,
                'overlap_percentage': self.calculate_overlap(results_legacy, results_new),
                'quality_score_delta': self.compare_quality(results_legacy, results_new)
            })
        
        # Return only legacy results (new engine is "dark")
        return results_legacy
```

### Metrics to Track

**Performance Comparison**:
```yaml
metrics:
  - response_time_delta: new_time - legacy_time
  - accuracy_comparison: user_clicks_on_results
  - resource_utilization: cpu_memory_network
  - error_rate_delta: new_errors - legacy_errors
```

**Quality Assessment**:
```yaml
quality:
  - result_overlap: % of same items in both result sets
  - ranking_correlation: spearman correlation of rankings
  - user_satisfaction: click-through rate simulation
  - business_impact: estimated revenue change
```

## How to Implement Dark Launching

Dark launching implementation leverages **identical technical foundations** as canary releases and feature toggle systems, employing deployment isolation and controlled exposure mechanisms.

### Implementation Steps

**Step 1: Deploy Feature**
```
Feature deployment precedes activation, maintaining dormant state until strategic exposure timing.
```

**Step 2: Configure Shadow Execution**
```python
# Feature toggle configuration
DARK_LAUNCH_CONFIG = {
    'new_recommendation_engine': {
        'enabled': True,
        'shadow_mode': True,  # Run but don't display
        'percentage': 25,      # 25% of requests
        'log_results': True,
        'compare_with_legacy': True
    }
}
```

**Step 3: Collect Telemetry**
```python
def log_dark_launch_comparison(feature_name, legacy_result, new_result):
    telemetry.log({
        'feature': feature_name,
        'timestamp': datetime.now(),
        'legacy': {
            'result': legacy_result,
            'performance': legacy_result.timing,
            'success': legacy_result.success
        },
        'new': {
            'result': new_result,
            'performance': new_result.timing,
            'success': new_result.success
        },
        'comparison': {
            'results_match': legacy_result == new_result,
            'performance_delta': new_result.timing - legacy_result.timing,
            'accuracy_delta': calculate_accuracy_delta(legacy_result, new_result)
        }
    })
```

**Step 4: Analyze and Decide**
```
Previously documented canary release and feature toggle techniques apply comprehensively, requiring only exposure strategy adjustments.
```

## Dark Launching Use Cases

### 1. Infrastructure Validation
**Test new database, cache, or API** without exposing results to users.

**Example**: New caching layer
```python
# Write to both caches (old + new)
cache_old.set(key, value)
cache_new.set(key, value)  # Dark

# Read from old cache (production)
result = cache_old.get(key)

# Also read from new cache (compare performance)
result_new = cache_new.get(key)  # Dark
log_cache_performance(cache_old_time, cache_new_time)

return result  # Only old cache result used
```

### 2. Algorithm Comparison
**Test new ML model, search algorithm, or recommendation engine**.

**Example**: Fraud detection
```python
# Legacy fraud detector (production)
is_fraud_legacy = legacy_detector.check(transaction)

# New ML fraud detector (dark)
is_fraud_ml = ml_detector.check(transaction)  # Shadow

# Compare predictions
if is_fraud_legacy != is_fraud_ml:
    log_prediction_divergence(transaction, is_fraud_legacy, is_fraud_ml)

# Block transaction only based on legacy detector
return is_fraud_legacy
```

### 3. Performance Benchmarking
**Measure new implementation performance** under real production load.

**Example**: Database query optimization
```python
# Old query (production)
start = time.now()
results_old = db.execute(legacy_query)
time_old = time.now() - start

# New optimized query (dark)
start = time.now()
results_new = db.execute(optimized_query)
time_new = time.now() - start

# Log performance improvement
log_query_performance({
    'improvement_ms': time_old - time_new,
    'improvement_pct': (time_old - time_new) / time_old * 100
})

return results_old  # Use legacy results
```

## Quick Reference

### Dark Launching Characteristics
- 🕶️ **Hidden from users**: Feature runs but results not displayed
- 🔬 **Shadow execution**: Parallel processing for comparison
- 📊 **Zero risk**: No user-facing impact
- 🔍 **Performance validation**: Real production load testing
- 📈 **Gradual confidence**: Validate before user exposure

### Implementation Techniques
| Technique | Use Case | Complexity |
|-----------|----------|------------|
| **Shadow mode** | Algorithm testing | Medium |
| **Parallel processing** | Payment/transaction systems | High |
| **Dry run mode** | Checkout/workflow validation | Low |
| **Dual sensor** | Infrastructure validation | Medium |

### When to Use Dark Launching
- ✅ **Infrastructure changes** (database, cache, API)
- ✅ **Algorithm updates** (search, recommendations, fraud detection)
- ✅ **Performance optimization** (query optimization, caching)
- ✅ **High-risk features** (payment, security, critical workflows)
- ❌ **UI/UX changes** (use canary releases instead)

### Critical Notes
- ⚠️ **Discard shadow results** or log only (don't show to users)
- 💡 **Monitor performance impact** of parallel execution
- 🎯 **Limit shadow execution percentage** (10-25% of requests)
- 📊 **Compare results continuously** (alert on divergence)
- 🔄 **Transition to canary** after dark launch validation

---

**Next**: Test your knowledge with the module assessment →

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-canary-releases-dark-launching/4-understand-dark-launching)
