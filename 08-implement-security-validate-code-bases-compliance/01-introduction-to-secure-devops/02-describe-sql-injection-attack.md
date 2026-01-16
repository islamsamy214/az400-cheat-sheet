# Describe SQL Injection Attack

## Key Concepts
- **SQL Injection**: Attacker inserts malicious SQL code into application queries
- **Root Cause**: Application executes user input as code instead of treating it as data
- **Attack Vector**: Unsanitized user input combined with dynamic SQL queries

## How SQL Injection Works

**Vulnerable Query Example**:
```sql
SELECT * FROM users WHERE username = 'inputUsername' AND password = 'inputPassword'
```

**Attacker Input**: `admin' --` as username

**Resulting Query**:
```sql
SELECT * FROM users WHERE username = 'admin' --' AND password = 'inputPassword'
```
The `--` comment removes password check, bypassing authentication.

## What Attackers Can Accomplish

| Attack Type | Impact |
|-------------|--------|
| **Bypass Authentication** | Access accounts without credentials, elevate privileges |
| **Data Exfiltration** | Extract entire database contents (customer data, secrets, IP) |
| **Data Manipulation** | Insert, update, or delete records; add admin accounts |
| **OS Command Execution** | Execute system commands, compromise entire server |
| **Denial of Service** | Craft resource-intensive queries to crash application |

## Widespread Impact

**Affects All SQL Databases**:
- MySQL, Oracle, SQL Server, PostgreSQL, SQLite
- Vulnerability exists in **application code**, not database software
- Even secure databases vulnerable with poor application code

**Sensitive Data at Risk**:
- Customer information (names, addresses, payment details)
- Personal data (SSN, DOB, medical records, financial info)
- Business data (trade secrets, strategic plans, partner agreements)
- Authentication credentials (passwords, API keys, tokens)

## Why It Persists

- Legacy applications built before SQL injection awareness
- Developer security training gaps
- Complex codebases with many database query points
- Development deadline pressures leading to shortcuts
- OWASP Top 10 vulnerability (remains prominent)

## Prevention Strategies

```bash
# Essential defenses:
✓ Use parameterized queries/prepared statements
✓ Validate and sanitize ALL user input
✓ Apply least privilege to database accounts
✓ Conduct regular security testing and scanning
✓ Perform security-focused code reviews
✓ Monitor for suspicious database activity
✓ Train developers on secure coding practices
```

## Critical Notes
- ⚠️ One of the most dangerous web vulnerabilities
- 💡 Vulnerability is in code, not database software
- 🎯 Prevention requires multiple layers of defense
- 📊 Regular security testing is essential

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-secure-devops/2-describe-sql-injection-attack)
