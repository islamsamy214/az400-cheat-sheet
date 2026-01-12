# Plan Effective Code Reviews

## Purpose
Code reviews are one of the **best ways to catch technical debt** before it enters your codebase. When done well, reviews improve code quality and help your team learn together.

## Create the Right Mindset

### Focus on Learning, Not Blame
**Approach**:
- Treat reviews as **mentoring sessions** where people share ideas
- Avoid making reviews feel like **interrogations**
- Remember that **everyone wants to do good work**

**Example**:
```
❌ Bad: "Why did you write this terrible code?"
✅ Good: "I see what you're doing here. Have you considered using 
          the Strategy pattern instead? It might make this easier 
          to test."
```

### Share Knowledge Openly
**Principles**:
- Use reviews as **teaching opportunities** for the whole team
- Highlight what's been **done well**, not just problems
- Help everyone learn **new techniques and approaches**

**Example Comments**:
```
✅ "Great use of dependency injection here! This makes the code 
   much more testable."
   
✅ "I learned about the async/await pattern from your PR. Nice work!"

✅ "This is a good start. Here's a technique I use for similar 
   situations: [example]"
```

## Make Reviews Effective

### Keep Groups Small
**Best Practices**:
- Work in **pairs or small groups** (2-3 people)
- Avoid large team meetings that become overwhelming
- Create space for **real discussion and learning**

| Group Size | Effectiveness | Why |
|------------|---------------|-----|
| 1 reviewer | Good | Fast, focused |
| 2-3 reviewers | Best | Multiple perspectives, learning |
| 4+ reviewers | Poor | Too many opinions, slow |

### Focus on Improvement
**Look For**:
- Opportunities to **reduce technical debt**
- Better **patterns and practices**
- Knowledge about **tools and techniques**

**Review Checklist**:
```
□ Can this code be simplified?
□ Are there reusable components?
□ Is the naming clear and descriptive?
□ Are there potential performance issues?
□ Is error handling comprehensive?
□ Are there security concerns?
```

### Balance Feedback
**Effective Feedback Includes**:
- Point out what's **working well**
- Offer **specific suggestions** for improvement
- Ask **questions** to understand the author's thinking

**Feedback Formula**:
```
1. Positive: "This method is well-structured and easy to follow."
2. Question: "I'm curious why you chose approach X over Y?"
3. Suggestion: "Have you considered using Z? It might simplify this."
4. Positive: "Overall, this is a solid implementation!"
```

## Build a Supportive Culture

### Emphasize Long-Term Benefits
**Help Team Understand**:
- Quality **reduces costs over time**
- Reviews **prevent bugs and maintenance issues**
- Celebrate **improvements in code quality**

### Foster Collaboration
**Create Environment Where**:
- It's **safe to ask questions**
- **Everyone participates** in reviews
- **Disagreements are learning opportunities**

**Example Culture Behaviors**:
```
✅ "That's an interesting approach I hadn't considered."
✅ "I don't understand this part. Can you explain your reasoning?"
✅ "Both approaches have merit. Let's discuss trade-offs."
❌ "This is obviously wrong."
❌ "I would never write code like this."
```

## Common Code Review Goals

During reviews, look for these opportunities to prevent technical debt:

| Goal | Questions to Ask | Red Flags |
|------|------------------|-----------|
| **Readability** | Easy to understand and maintain? | Cryptic names, deep nesting |
| **Consistency** | Follows team standards and patterns? | Mixed styles, inconsistent formatting |
| **Performance** | Obvious efficiency improvements? | N+1 queries, inefficient algorithms |
| **Security** | Follows secure coding practices? | SQL injection, XSS vulnerabilities |
| **Testing** | Properly tested and testable? | No tests, untestable code |

### Detailed Review Criteria

#### Readability Checks
```csharp
// ❌ Bad: Unclear, hard to follow
var x = data.Where(d => d.Type == 1 && d.Status == "A").Select(d => d.Value).Sum();

// ✅ Good: Clear, self-documenting
var activeCustomerTotalValue = customers
    .Where(customer => customer.IsActive && customer.Status == CustomerStatus.Active)
    .Sum(customer => customer.AccountValue);
```

#### Consistency Checks
- **Naming conventions**: PascalCase, camelCase, snake_case used consistently
- **Error handling**: Same approach throughout (exceptions vs return codes)
- **Logging**: Consistent log levels and formats
- **Documentation**: Comments follow team standards

#### Performance Checks
```csharp
// ❌ Bad: N+1 query problem
foreach (var customer in customers)
{
    var orders = db.Orders.Where(o => o.CustomerId == customer.Id).ToList();
    // Process orders...
}

// ✅ Good: Single query with join
var customersWithOrders = db.Customers
    .Include(c => c.Orders)
    .ToList();
```

## Remember the Bigger Picture

### Benefits of Code Reviews
Code reviews might seem to slow development, but they actually:
- **Catch bugs** before they reach production
- **Share knowledge** across the team
- **Prevent technical debt** from accumulating
- **Improve overall code quality**
- **Help team members grow** their skills

### Time Investment Returns
| Investment | Return |
|------------|--------|
| 30 min review | Catches bug that would take 4 hours to fix in production |
| 15 min discussion | Prevents design flaw affecting future features |
| 10 min feedback | Teaches better pattern used in 10 future PRs |

### Long-Term Impact
The time invested in good code reviews pays dividends in:
- **Easier maintenance**: Clean code is faster to modify
- **Fewer bugs**: Issues caught before production
- **More skilled team**: Continuous learning through reviews
- **Better architecture**: Shared understanding of patterns

## Code Review Process

### Recommended Workflow
1. **Author**: Create PR with clear description and context
2. **Reviewers**: Review within 24 hours
3. **Discussion**: Ask questions, suggest improvements
4. **Author**: Address feedback, explain decisions
5. **Approval**: At least one reviewer approves
6. **Merge**: Author or reviewer merges

### PR Description Template
```markdown
## Summary
Brief description of changes and why they're needed

## Changes Made
- Specific change 1
- Specific change 2

## Testing
- Unit tests added/updated
- Manual testing performed

## Screenshots (if UI changes)
[Include relevant screenshots]

## Checklist
- [ ] Code follows team standards
- [ ] Tests pass
- [ ] Documentation updated
```

## Critical Notes
- 🎯 Code reviews are the best way to catch technical debt early
- 💡 Keep review groups small (2-3 people) for effectiveness
- ⚠️ Focus on learning and improvement, not blame
- 📊 Balance feedback: positive + questions + suggestions
- 🔄 Reviews catch bugs before production and share knowledge
- ✨ Time invested in reviews pays dividends in maintenance and quality

[Learn More](https://learn.microsoft.com/en-us/training/modules/identify-technical-debt/8-plan-effective-code-reviews)
