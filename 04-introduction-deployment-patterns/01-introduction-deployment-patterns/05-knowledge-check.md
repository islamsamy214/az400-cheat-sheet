# Module Assessment

## Knowledge Check Questions

### Question 1: Microservices Architecture
**What is a primary benefit of microservices architecture for continuous delivery?**

A) Eliminates all deployment risks  
B) Requires no testing infrastructure  
C) Enables independent service deployment without affecting other services  
D) Automatically scales all services equally  

<details>
<summary>✅ Answer</summary>

**C) Enables independent service deployment without affecting other services**

**Explanation**: Microservices architecture allows each service to have its own deployment pipeline, enabling autonomous deployment workflows. Services can be deployed independently without requiring system-wide coordination, reducing deployment complexity and risk.

**Why others are wrong**:
- A: No architecture eliminates all risks (risk mitigation, not elimination)
- B: Testing infrastructure still required (automated tests essential for CD)
- D: Services scale independently based on demand (not equal scaling)

</details>

---

### Question 2: Classical Deployment Patterns
**What is a key limitation of classical deployment patterns?**

A) Too many deployment environments  
B) Production behavior uncertainty despite comprehensive preproduction testing  
C) Requires microservices architecture  
D) Cannot use automated testing  

<details>
<summary>✅ Answer</summary>

**B) Production behavior uncertainty despite comprehensive preproduction testing**

**Explanation**: Nonproduction environments cannot fully replicate production workload characteristics, usage patterns, and operational constraints. Real users behave unpredictably, and concurrent event combinations expose untested code paths unavailable in synthetic test environments.

**Why others are wrong**:
- A: Multiple environments are beneficial (not a limitation)
- C: Classical patterns work with monoliths (no microservices requirement)
- D: Automated testing fully compatible with classical patterns

</details>

---

### Question 3: Blue-Green Deployments
**What is the primary advantage of blue-green deployments?**

A) Eliminates need for testing  
B) Reduces infrastructure costs  
C) Enables instant rollback capabilities  
D) Requires no load balancer  

<details>
<summary>✅ Answer</summary>

**C) Enables instant rollback capabilities**

**Explanation**: Blue-green deployments maintain two parallel environments. If issues arise after switching traffic to the new (Green) environment, you can instantly rollback by switching the load balancer back to the old (Blue) environment.

**Why others are wrong**:
- A: Testing still required before switching traffic
- B: Actually increases costs (requires 2x infrastructure)
- D: Load balancer essential for traffic switching between environments

</details>

---

### Question 4: Canary Releases
**What is the main purpose of canary releases?**

A) Deploy to all users simultaneously  
B) Incremental traffic migration for gradual validation  
C) Eliminate all production testing  
D) Deploy only to development environments  

<details>
<summary>✅ Answer</summary>

**B) Incremental traffic migration for gradual validation**

**Explanation**: Canary releases gradually migrate traffic from old version to new version (e.g., 5% → 25% → 50% → 100%), monitoring metrics at each phase. This limits blast radius and enables early issue detection with minimal user impact.

**Why others are wrong**:
- A: Opposite of gradual rollout (that's big bang deployment)
- C: Actually enables production validation (not elimination)
- D: Canary releases specifically for production (not dev environments)

</details>

---

### Question 5: Feature Toggles
**How do feature toggles support modern deployment patterns?**

A) Eliminate need for version control  
B) Enable deployment without immediate user exposure through runtime feature control  
C) Automatically test all code paths  
D) Prevent all deployment failures  

<details>
<summary>✅ Answer</summary>

**B) Enable deployment without immediate user exposure through runtime feature control**

**Explanation**: Feature toggles (feature flags) allow deploying code to production while keeping features hidden. Features can be enabled/disabled at runtime without redeployment, enabling progressive rollout, A/B testing, and instant rollback.

**Why others are wrong**:
- A: Version control still required (feature toggles complement, not replace)
- C: Testing still required (toggles don't test code paths)
- D: Reduce risks but don't prevent all failures (enable faster recovery)

</details>

---

### Question 6: Progressive Exposure
**What is ring-based deployment (progressive exposure)?**

A) Deploy to single environment only  
B) Phased rollout through user cohorts (internal → early adopters → pilot → all users)  
C) Random deployment without monitoring  
D) Deploy all features simultaneously  

<details>
<summary>✅ Answer</summary>

**B) Phased rollout through user cohorts (internal → early adopters → pilot → all users)**

**Explanation**: Ring-based deployment progressively expands feature exposure through defined user rings (e.g., Ring 0: internal users, Ring 1: beta testers, Ring 2: pilot customers, Ring 3: all users), monitoring at each stage before proceeding.

**Why others are wrong**:
- A: Multiple rings/environments (not single)
- C: Systematic with comprehensive monitoring (not random)
- D: Gradual rollout (opposite of simultaneous)

</details>

---

### Question 7: Dark Launching
**What is the purpose of dark launching?**

A) Hide features from developers  
B) Hidden feature deployment for infrastructure validation  
C) Delete old features  
D) Disable all production monitoring  

<details>
<summary>✅ Answer</summary>

**B) Hidden feature deployment for infrastructure validation**

**Explanation**: Dark launching deploys new features to production and routes real traffic through them, but discards results (doesn't show to users). This validates infrastructure capacity, performance, and integrations under real load without user impact.

**Why others are wrong**:
- A: Hidden from end users, not developers (developers need access to test)
- C: Not about deletion (about hidden feature validation)
- D: Requires intensive monitoring (not disabling)

</details>

---

### Question 8: A/B Testing
**What is A/B testing in deployment contexts?**

A) Testing in development and staging only  
B) Controlled experimentation for data-driven decision making  
C) Testing with two developers  
D) Deploy to production or staging (not both)  

<details>
<summary>✅ Answer</summary>

**B) Controlled experimentation for data-driven decision making**

**Explanation**: A/B testing splits users into groups (Variant A and Variant B), measures outcomes (conversion rates, engagement, performance), and chooses the better-performing variant based on data.

**Why others are wrong**:
- A: A/B testing specifically in production (not pre-production only)
- C: About user groups, not developer teams
- D: Both variants deployed to production simultaneously

</details>

---

### Question 9: Architecture Readiness
**What is a critical assessment consideration for CD readiness?**

A) Company size only  
B) Independent deployment capability and quality assurance scalability  
C) Number of developers  
D) Office location  

<details>
<summary>✅ Answer</summary>

**B) Independent deployment capability and quality assurance scalability**

**Explanation**: Architecture must support independent module deployment and automated quality guarantees for high-frequency deployments. Monolithic architectures with tight coupling create CD challenges, while component-based architectures enable autonomous deployment.

**Why others are wrong**:
- A: Architecture matters more than company size
- C: Team size relevant but not critical technical assessment
- D: Location irrelevant to CD readiness

</details>

---

### Question 10: Modern vs Classical Patterns
**What distinguishes modern deployment patterns from classical patterns?**

A) Modern patterns eliminate all testing  
B) Modern patterns enable production validation with controlled exposure strategies  
C) Modern patterns require mainframe computers  
D) Modern patterns only work on weekends  

<details>
<summary>✅ Answer</summary>

**B) Modern patterns enable production validation with controlled exposure strategies**

**Explanation**: Modern patterns (canary, blue-green, feature toggles, ring-based) enable production testing through progressive rollout controls and targeted availability management, minimizing risk while maximizing real-world validation coverage.

**Why others are wrong**:
- A: More testing required (production + pre-production)
- C: Modern patterns use cloud infrastructure (not mainframes)
- D: On-demand deployment (not scheduled weekend releases)

</details>

---

## Score Interpretation
- **9-10 correct**: Excellent understanding of deployment patterns
- **7-8 correct**: Good grasp, review missed topics
- **5-6 correct**: Fair understanding, re-read module
- **<5 correct**: Needs review, focus on key concepts

## Critical Notes
- ⚠️ **Modern patterns = production validation**: Controlled exposure, not elimination of testing
- 💡 **Feature toggles foundation**: Enable most modern deployment patterns
- 🎯 **Progressive rollout reduces risk**: Gradual exposure limits blast radius
- 📊 **Microservices simplify CD**: Not prerequisite, but significant benefit
- 🔄 **Classical patterns still valid**: Appropriate for certain contexts (regulated industries, infrequent releases)
- ✨ **Patterns can combine**: Real-world deployments often use multiple patterns together

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-deployment-patterns/5-knowledge-check)
