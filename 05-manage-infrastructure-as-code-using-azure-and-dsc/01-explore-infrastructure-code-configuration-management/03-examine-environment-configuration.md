# Examine Environment Configuration

## Overview

While Infrastructure as Code (IaC) focuses on provisioning resources—creating virtual machines, networks, storage accounts, and databases—configuration management addresses what happens after resources exist. Configuration management encompasses the installation of software, application of security settings, management of files and permissions, and ongoing maintenance of system states. The distinction is critical: IaC creates a virtual machine; configuration management installs web server software, configures SSL certificates, sets up logging, and ensures the application runs correctly.

Configuration as Code applies the same software engineering rigor to configuration management that IaC applies to infrastructure provisioning. Rather than manually remoting into servers to install software or change settings, configuration as code defines desired states in version-controlled files that can be reviewed, tested, and automatically applied. This approach transforms configuration from an error-prone manual process into a reliable, repeatable, automated workflow that scales across hundreds or thousands of servers.

The challenge organizations face isn't just managing configuration for a single application or server—it's maintaining consistency across development, testing, and production environments, each potentially containing dozens of servers. Manual configuration quickly becomes unmaintainable as complexity grows, leading to configuration drift (environments diverging from documented states) and the dreaded "it works in test but not in production" syndrome.

## Key Concepts

### Manual Configuration Challenges

**The Complexity Problem**

Managing configuration for even a single application involves tracking numerous settings:
- Application configuration files (`web.config`, `appsettings.json`, `application.properties`)
- Environment variables defining behavior
- Database connection strings and credentials
- Feature flags enabling/disabling functionality
- Logging levels and output destinations
- Security certificates and their expiration dates
- Firewall rules and network ACLs
- Third-party service integration settings
- Performance tuning parameters

Multiply this by multiple applications, multiple servers per application, and multiple environments (development, QA, staging, production), and the configuration matrix becomes unmanageable through manual processes.

**Common Manual Configuration Problems**

**Outdated Documentation**
- Settings documented in spreadsheets that nobody maintains
- Wiki pages with conflicting information from different time periods
- Email threads containing "tribal knowledge" about special configurations
- Documentation written during initial setup but never updated
- New team members receiving outdated information leading to incorrect assumptions

**Example Scenario**: Production web server runs IIS with specific module configurations. Documentation says to enable Windows Authentication, but nobody mentions that custom authentication module must be disabled first. New administrator follows documentation, creates misconfiguration, causes outage.

**Tribal Knowledge Dependency**
- Critical configuration details exist only in specific people's heads
- "Ask Sarah, she configured that server three years ago"
- Key person leaves organization, taking knowledge with them
- Emergency changes during outages with no documentation
- Undocumented workarounds that nobody remembers why they exist

**Configuration Drift**
- Environments diverge over time through manual changes
- "Quick fix" in production never applied to other environments
- Different team members configure servers slightly differently
- No detection mechanism for unauthorized changes
- Accumulation of undocumented changes makes it impossible to reproduce environment

**Example**: Production database has specific collation settings that fix a bug. Development and test databases don't have these settings because they were configured before the fix. Developers don't see the bug, but it appears in production.

**Time-Consuming Manual Processes**
- Hours spent remoting into servers to make changes
- Waiting for change approvals and maintenance windows
- Repeating same process across multiple servers
- Verifying changes manually after application
- Rollback requires manually reversing changes

### Configuration as Code Solution

**Definition**

Configuration as code treats configuration files and settings like application source code:
- Stored in version control (Git)
- Reviewed before changes (pull requests)
- Deployed through automated pipelines (CI/CD)
- Tested in non-production environments before production
- Monitored for drift from desired state

**The Firewall Rule Example**

**Manual Process**:
1. Administrator receives ticket requesting new firewall rule
2. Logs into server via RDP or SSH
3. Opens firewall configuration tool
4. Creates rule allowing TCP port 8080
5. Tests connectivity
6. Documents change in spreadsheet (maybe)
7. Repeats for all servers in environment
8. Repeats for test and production environments
9. Hopes documentation stays current

**Configuration as Code Process**:
1. Developer edits configuration file:
   ```yaml
   # firewall_rules.yaml
   firewall_rules:
     - name: "Allow Application Port"
       port: 8080
       protocol: tcp
       source: "10.0.0.0/8"
       action: allow
   ```
2. Commits change to Git repository
3. Creates pull request for review
4. Automated tests validate rule syntax and security implications
5. Team reviews and approves pull request
6. Merge triggers CI/CD pipeline
7. Pipeline applies configuration to all servers automatically
8. Firewall rule created consistently across all environments

**Result**: Minutes instead of hours. No remote desktop sessions. No manual documentation. Perfect consistency across environments. Complete audit trail.

### Configuration Management vs. Infrastructure as Code

While the terms overlap and some practitioners use "Infrastructure as Code" to encompass both concepts, distinguishing between them clarifies responsibilities:

**Infrastructure as Code (Provisioning)**
- Creates resources that didn't exist
- Provisions virtual machines, networks, storage accounts, databases
- Defines resource properties: VM size, network address space, storage tier
- Manages relationships: which VM connects to which network
- Focus: "What infrastructure exists?"

**Configuration as Code (Configuration)**
- Configures resources after they exist
- Installs software, updates OS patches, configures services
- Manages application settings, feature flags, environment variables
- Maintains desired state: detects and corrects drift
- Focus: "How are resources configured?"

**Practical Example: Web Application Deployment**

IaC responsibilities:
```hcl
# Terraform creates infrastructure
resource "azurerm_virtual_machine" "web" {
  name                  = "web-server-01"
  location              = "East US"
  size                  = "Standard_D2s_v3"
  network_interface_ids = [azurerm_network_interface.main.id]
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }
  
  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
}
```

Configuration Management responsibilities:
```yaml
# Ansible configures the VM after creation
- name: Install and configure web server
  hosts: web_servers
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Configure SSL certificates
      copy:
        src: files/ssl-cert.pem
        dest: /etc/nginx/ssl/cert.pem
        mode: '0600'
    
    - name: Deploy application configuration
      template:
        src: templates/app-config.j2
        dest: /etc/app/config.json
    
    - name: Ensure Nginx running
      service:
        name: nginx
        state: started
        enabled: yes
```

**Team Responsibilities**

Organizations often split responsibilities:
- **Infrastructure team**: Provisions resources using IaC (Terraform, ARM templates)
- **Application teams**: Configure resources using configuration management (Ansible, DSC)
- **Shared responsibility**: Both teams collaborate on networking, security policies, monitoring

However, modern DevOps practices encourage cross-functional teams owning both infrastructure and configuration for their applications.

### Comparison: Manual Configuration vs. Configuration as Code

| Aspect | Manual Configuration | Configuration as Code |
|--------|---------------------|----------------------|
| **Bug Reproduction** | Hard to reproduce: "It works in test but not production—what's different?" | Easily reproducible: Same code produces same configuration every time |
| **Reliability** | Error-prone: People make mistakes during manual steps (typos, skipped steps, wrong values) | Reliable: Automation eliminates human error; tested configuration applied consistently |
| **Verification** | Extensive manual checks: Administrator validates each change across all servers | Fast deployments: Automated tests catch issues earlier in pipeline |
| **Documentation** | Documentation becomes outdated: Emergency changes bypass documentation process | Self-documenting: Configuration files ARE the documentation; always current |
| **Deployment Schedule** | Risky deployments: Weekend maintenance windows to allow recovery time if issues occur | Deploy anytime: Confidence in automated processes enables frequent deployments |
| **Release Cadence** | Slow iteration: Fear of breaking things limits deployment frequency | Fast iteration: Deploy changes quickly and safely with rollback capability |
| **Compliance** | Manual auditing: Screenshots, spreadsheets, and documentation for compliance reviews | Automated compliance: Version control provides complete audit trail; policy as code enforcement |
| **Disaster Recovery** | Slow recovery: Manually recreate configurations from documentation | Fast recovery: Apply configuration code to new resources automatically |
| **Scaling** | Difficult to scale: Each new server requires manual configuration | Easy to scale: Configuration automatically applied to new instances |
| **Team Collaboration** | Conflicts: Multiple people making concurrent changes cause conflicts | Code review: Team reviews configuration changes before deployment |

## Benefits of Configuration Management

### Reproducible Environments

**The Core Problem**: "It works on my machine" syndrome

Developers frequently encounter bugs that appear in one environment but not another. Root cause: configuration differences nobody documented. Configuration as code solves this by ensuring identical configurations across environments.

**Example Scenario**

**Before Configuration as Code**:
- Developer builds feature on laptop running Node.js 14.x
- Test environment runs Node.js 16.x
- Production runs Node.js 12.x (never upgraded)
- Feature uses async/await syntax available in 14.x but deprecated in 12.x
- Works in development and test, fails in production
- Hours spent troubleshooting "random" production issue

**After Configuration as Code**:
```yaml
# Ansible playbook ensures consistent Node.js version
- name: Ensure correct Node.js version
  hosts: all_environments
  tasks:
    - name: Install Node.js 16.x LTS
      apt:
        name: nodejs=16.x
        state: present
    
    - name: Verify Node.js version
      command: node --version
      register: node_version
      failed_when: "'v16.' not in node_version.stdout"
```

- All environments run identical Node.js version
- Configuration code validated in CI/CD pipeline
- Developers confidently deploy knowing environments match

### Environment Parity

**Dev/Prod Parity Principle** (from 12-Factor App methodology):

Keep development, staging, and production environments as similar as possible. Minimizing differences reduces bugs and makes deployments predictable.

**Common Parity Violations**:
- Different operating system versions (Windows in dev, Linux in prod)
- Different database engines (SQLite in dev, PostgreSQL in prod)
- Different dependency versions (cached old versions in dev, new versions in prod)
- Different security configurations (lax in dev, strict in prod)
- Different resource sizes (small VM in dev, large VM in prod)

**Configuration as Code Solution**:
```yaml
# Parameterized configuration maintains parity
environments:
  development:
    database: postgresql-12
    app_server: nginx-1.20
    vm_size: Standard_B2s  # Smaller but same type
    security_policy: strict  # Same policies, all environments
  
  production:
    database: postgresql-12  # Same version
    app_server: nginx-1.20  # Same version
    vm_size: Standard_D4s_v3  # Larger but same configuration
    security_policy: strict  # Identical security
```

**Benefits**:
- Bugs surface in development, not production
- Deployment confidence: if it works in staging, it works in production
- Easier troubleshooting: fewer variables to investigate
- Lower risk: predictable behavior across environments

### Faster Deployments

**Quantifying Speed Improvements**

**Manual Approach**: Deploying configuration changes to 10-server cluster
- 30 minutes per server (RDP/SSH, make changes, verify)
- 5 hours total if done sequentially
- 1-2 hours if done in parallel (requires multiple administrators)
- High risk of mistakes due to repetition and fatigue

**Automated Approach**: Same 10-server deployment
```bash
# Single command deploys configuration to all servers
ansible-playbook -i production_inventory site.yml
```
- 15 minutes total (parallel execution across all servers)
- Zero manual steps (no RDP/SSH sessions)
- Consistent changes (no variation between servers)
- Logged output for verification

**Annual Impact**:
- 20 configuration deployments per year
- Manual: 100 hours (5 hours × 20 deployments)
- Automated: 5 hours (15 minutes × 20 deployments)
- **Savings**: 95 hours per year, plus reduced errors

### Reduced Documentation Burden

**Traditional Documentation Challenges**:
- Separate documentation must be written and maintained
- Documentation becomes outdated immediately after creation
- No link between documentation and actual configuration
- Different formats: Word documents, wikis, SharePoint, email threads
- Hard to find: "Which version of the runbook is current?"

**Configuration as Code Documentation**:

**Self-Documenting Code**:
```yaml
# The code IS the documentation
- name: Configure web server with TLS 1.3 only
  hosts: web_servers
  tasks:
    - name: Disable TLS 1.0 and 1.1 (PCI-DSS compliance requirement)
      ansible.builtin.lineinfile:
        path: /etc/nginx/nginx.conf
        regexp: '^ssl_protocols'
        line: 'ssl_protocols TLSv1.3;'
        backup: yes
      notify: reload nginx
    
    - name: Set secure cipher suites (OWASP recommendations)
      ansible.builtin.lineinfile:
        path: /etc/nginx/nginx.conf
        regexp: '^ssl_ciphers'
        line: 'ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;'
      notify: reload nginx
```

**Supporting Documentation**:
```markdown
# README.md in repository
## Overview
This playbook configures Nginx web servers for production use.

## Prerequisites
- Ansible 2.10+
- SSH access to target servers
- SSL certificates in files/ssl/ directory

## Usage
\`\`\`bash
# Deploy to all web servers
ansible-playbook -i inventory/production site.yml --tags web_config

# Deploy to specific server
ansible-playbook -i inventory/production site.yml --limit web-01
\`\`\`

## Security Notes
- TLS 1.3 required for PCI-DSS compliance
- Certificate rotation occurs automatically via Let's Encrypt
```

**Advantages**:
- Code shows exact current state
- README provides context and usage instructions
- Git history shows what changed and why
- Code comments explain complex logic
- No separate documentation to maintain

### Easy Scaling

**Horizontal Scaling Scenario**

**Manual Approach**: Adding 5 new web servers to handle increased load
1. Provision VMs (IaC handles this)
2. Manually configure each new server:
   - Install web server software
   - Copy SSL certificates
   - Configure application settings
   - Set up monitoring agents
   - Configure log shipping
   - Join server to load balancer
3. Verify each server works correctly
4. Update documentation with new servers

**Configuration as Code Approach**:
```yaml
# Update inventory to include new servers
[web_servers]
web-01.prod.example.com
web-02.prod.example.com
web-03.prod.example.com
web-04.prod.example.com
web-05.prod.example.com
web-06.prod.example.com  # NEW
web-07.prod.example.com  # NEW
web-08.prod.example.com  # NEW
web-09.prod.example.com  # NEW
web-10.prod.example.com  # NEW

# Run playbook - automatically configures new servers
ansible-playbook -i inventory/production site.yml --limit web-06:web-10
```

**Result**: New servers configured identically to existing ones in minutes. No manual steps. Configuration automatically applied.

### Full Change History

**Version Control Advantages**

**Git Commit History**:
```bash
$ git log --oneline configuration/web-servers.yml
a3d2e19 Update Nginx to version 1.20.2 for security patch
f4b8c77 Add rate limiting to prevent DDoS attacks
e9a1d44 Configure HTTP/2 support for better performance
c2b5f33 Update SSL cipher suites per security audit
b7e9a22 Initial web server configuration
```

**Detailed Change View**:
```bash
$ git show a3d2e19
commit a3d2e19 Update Nginx to version 1.20.2 for security patch
Author: Jane Smith <jane@company.com>
Date:   Mon Mar 15 14:32:00 2024 -0500

    Update Nginx to version 1.20.2
    
    CVE-2024-1234 requires upgrading from 1.18.x to 1.20.2+
    Security advisory: https://nginx.org/security_advisories/
    
    Tested in staging environment with no issues.
    
    Fixes: SEC-456

diff --git a/configuration/web-servers.yml b/configuration/web-servers.yml
--- a/configuration/web-servers.yml
+++ b/configuration/web-servers.yml
@@ -10,7 +10,7 @@
   tasks:
     - name: Install Nginx
       apt:
-        name: nginx=1.18.*
+        name: nginx=1.20.2
```

**Audit Trail Benefits**:
- Who made change: Author field
- When: Timestamp
- What: Diff shows exact changes
- Why: Commit message explains rationale
- Context: Links to tickets, security advisories

### Drift Detection and Correction

**Configuration Drift**: Unauthorized changes to resources that cause them to diverge from defined desired state.

**Common Drift Causes**:
- Emergency fixes applied manually during outages
- "Quick tests" that are never reverted
- Automated processes that aren't idempotent
- Manual changes by administrators
- External tools or agents making changes

**Detection Mechanisms**:

**Azure Automation State Configuration** (PowerShell DSC):
```powershell
# Desired state definition
Configuration WebServerConfig {
    Node "web-server-*" {
        WindowsFeature IIS {
            Name = "Web-Server"
            Ensure = "Present"
        }
        
        WindowsFeature ASPNet {
            Name = "Web-Asp-Net45"
            Ensure = "Present"
        }
        
        Registry DisableSSLv3 {
            Key = "HKLM:\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server"
            ValueName = "Enabled"
            ValueData = 0
            ValueType = "Dword"
            Ensure = "Present"
        }
    }
}
```

Azure Automation DSC:
- Monitors servers every 30 minutes (configurable)
- Compares actual state to desired state
- Reports drift when differences detected
- Automatically corrects drift (optional)
- Alerts administrators of manual changes

**Ansible Check Mode**:
```bash
# Dry run - reports what would change
ansible-playbook site.yml --check --diff
```

Output shows drift:
```
TASK [Ensure Nginx config correct] *****************************
changed: [web-01]  # WARNING: Manual changes detected
--- before: /etc/nginx/nginx.conf
+++ after: /tmp/ansible-cfg-12345
@@ -1,1 +1,1 @@
-worker_processes 2;  # Someone manually changed this!
+worker_processes 4;  # Our desired state
```

**Remediation**: Run playbook without `--check` to correct drift.

### Team Collaboration

**Code Review for Configuration Changes**

Configuration as code enables same collaboration practices as application development:

**Pull Request Example**:
```markdown
## Change Summary
Update database connection pool size for improved performance

## Changes
- Increase max connections from 50 to 100
- Adjust connection timeout from 30s to 60s
- Enable connection recycling

## Testing
- Tested in dev environment with load testing tool
- CPU utilization decreased by 15%
- Response times improved by 200ms

## Risks
- Higher memory usage (monitored, acceptable)
- Database server can handle 100 concurrent connections per app server

## Rollback Plan
- Revert this PR
- Deploy previous version via pipeline
- ETA: 5 minutes
```

**Review Process**:
- Team members review changes
- Ask questions: "Why increase timeout?"
- Suggest improvements: "Consider enabling connection pooling"
- Approve when satisfied
- Automated tests run before merge

**Benefits**:
- Knowledge sharing: entire team learns about changes
- Error prevention: multiple eyes catch mistakes
- Best practices: senior engineers mentor junior engineers
- Accountability: clear record of who approved changes

### Compliance and Auditing

**Regulatory Requirements**

Industries like healthcare (HIPAA), finance (PCI-DSS, SOX), and government (FedRAMP) require:
- Complete audit trail of configuration changes
- Proof of who made changes and when
- Documentation of change approval process
- Regular compliance scanning and reporting
- Demonstration of security controls

**Configuration as Code Advantages**:

**Audit Trail**:
- Git commit history shows all changes
- Pull requests show approval workflow
- CI/CD logs prove automated deployment
- No manual changes possible (enforced by policies)

**Compliance Scanning**:
```yaml
# Azure Policy as code
policies:
  - name: "Require HTTPS for web apps"
    type: "Audit"
    rule: |
      {
        "if": {
          "allOf": [
            {"field": "type", "equals": "Microsoft.Web/sites"},
            {"field": "Microsoft.Web/sites/httpsOnly", "equals": false}
          ]
        },
        "then": {
          "effect": "audit"
        }
      }
```

Automated scanning:
- Runs on every deployment
- Prevents non-compliant configurations
- Generates compliance reports for auditors
- Alerts on policy violations

### Disaster Recovery

**Rapid Service Restoration**

**Scenario**: Data center failure destroys production servers.

**Traditional Recovery**:
1. Provision new servers (hours)
2. Find configuration documentation (hours)
3. Manually apply configurations (hours to days)
4. Discover documentation incomplete/outdated (more hours)
5. Troubleshoot configuration issues (more hours)
6. **Total RTO**: Days

**Configuration as Code Recovery**:
1. Provision new servers using IaC (minutes via Azure Site Recovery or IaC templates)
2. Apply configuration from Git repository (minutes)
   ```bash
   ansible-playbook -i disaster_recovery_inventory site.yml
   ```
3. Restore application data from backups (varies by data size)
4. Verify services operational (minutes)
5. **Total RTO**: Hours

**Advantages**:
- Predictable recovery process (tested regularly)
- No dependency on documentation or tribal knowledge
- Configuration guaranteed to be current (from Git)
- Can restore to different region/cloud provider if needed

### Testing Capabilities

**Test Configuration Changes Before Production**

**Pre-Deployment Testing**:
```yaml
# Test playbook in check mode first
- name: Run in check mode
  hosts: staging
  check_mode: yes
  tasks:
    - name: Update application config
      template:
        src: app-config.j2
        dest: /etc/app/config.json

# If check mode succeeds, apply to staging
- name: Deploy to staging
  hosts: staging
  tasks:
    - name: Update application config
      template:
        src: app-config.j2
        dest: /etc/app/config.json

# If staging succeeds, deploy to production
- name: Deploy to production
  hosts: production
  tasks:
    - name: Update application config
      template:
        src: app-config.j2
        dest: /etc/app/config.json
```

**Automated Testing Pipelines**:
```yaml
# GitHub Actions workflow
name: Test Configuration Changes
on: [pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      
      - name: Lint Ansible playbooks
        run: ansible-lint playbooks/
      
      - name: Test in Docker container
        run: |
          docker run -v $(pwd):/workspace ansible-test
          ansible-playbook --syntax-check site.yml
      
      - name: Deploy to test environment
        run: ansible-playbook -i test_inventory site.yml
      
      - name: Run integration tests
        run: pytest tests/integration/
      
      - name: Security scan
        run: ansible-playbook site.yml --check --diff | tee scan_results.txt
```

**Benefits**:
- Catch errors before production deployment
- Validate changes in realistic environment
- Regression testing ensures changes don't break existing functionality
- Security scanning identifies misconfigurations

## Real-World Implementation Scenarios

### Scenario 1: Multi-Tier Application Configuration

**Challenge**: Configure 3-tier application (web, app, database) consistently across dev, test, and production.

**Solution**:
```yaml
# Group variables define tier-specific configuration
group_vars/
├── web_servers.yml
│   └── web_server_port: 80
│       ssl_enabled: true
│       max_connections: 1000
├── app_servers.yml
│   └── app_pool_size: 20
│       cache_size_mb: 512
│       log_level: info
└── database_servers.yml
    └── max_connections: 100
        buffer_pool_size: 2G
        backup_enabled: true

# Inventory defines which servers in which tiers
inventory/production:
[web_servers]
web-01.prod.company.com
web-02.prod.company.com

[app_servers]
app-01.prod.company.com
app-02.prod.company.com
app-03.prod.company.com

[database_servers]
db-01.prod.company.com

# Playbook applies tier-specific configuration
site.yml:
- hosts: web_servers
  roles:
    - web_server
    - ssl_certificates
    - monitoring

- hosts: app_servers
  roles:
    - app_server
    - java_runtime
    - monitoring

- hosts: database_servers
  roles:
    - postgresql
    - backup_agent
    - monitoring
```

**Results**:
- Each tier configured appropriately for its role
- Consistent configuration across environments
- Easy to add new servers to any tier
- Changes propagate automatically to all servers in tier

### Scenario 2: Secrets Management Integration

**Challenge**: Configure application with database credentials and API keys without hardcoding secrets in configuration files.

**Solution**:
```yaml
# Playbook retrieves secrets from Azure Key Vault
- name: Configure application with secrets
  hosts: app_servers
  tasks:
    - name: Retrieve database password from Key Vault
      azure.azcollection.azure_rm_keyvaultsecret_info:
        vault_uri: "https://prod-keyvault.vault.azure.net"
        name: "DatabasePassword"
      register: db_secret
    
    - name: Retrieve API key from Key Vault
      azure.azcollection.azure_rm_keyvaultsecret_info:
        vault_uri: "https://prod-keyvault.vault.azure.net"
        name: "ThirdPartyAPIKey"
      register: api_secret
    
    - name: Generate application configuration file
      template:
        src: appsettings.json.j2
        dest: /opt/app/appsettings.json
        mode: '0600'
      vars:
        database_password: "{{ db_secret.secrets[0].secret }}"
        api_key: "{{ api_secret.secrets[0].secret }}"
```

Template file (appsettings.json.j2):
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=db.prod.company.com;Database=AppDB;User=appuser;Password={{ database_password }}"
  },
  "ExternalServices": {
    "ThirdPartyAPI": {
      "BaseUrl": "https://api.thirdparty.com",
      "ApiKey": "{{ api_key }}"
    }
  }
}
```

**Results**:
- Secrets never stored in Git
- Key Vault provides centralized secret management
- Secrets rotated without code changes
- Audit trail of secret access in Key Vault logs

## Assessment Questions

### Question 1: Configuration Management Scope

**Question**: Which tasks are within the scope of configuration management (as opposed to infrastructure provisioning)? (Select all that apply)

A) Creating an Azure Virtual Network  
B) Installing Nginx web server on a VM  
C) Provisioning an Azure SQL Database  
D) Configuring Nginx virtual hosts and SSL certificates  
E) Creating a storage account

**Answer**: B, D

**Explanation**: Configuration management handles software installation and configuration on existing resources. B (installing Nginx) and D (configuring Nginx) are configuration tasks. A, C, E are infrastructure provisioning tasks (creating resources that don't exist).

### Question 2: Configuration Drift

**Question**: Your monitoring alerts that production web server configurations no longer match desired state defined in code. Investigation reveals an administrator manually disabled a service during a late-night incident but never reverted the change. What's the best long-term solution?

A) Document the manual change in a runbook  
B) Implement policy preventing manual server access  
C) Configure automated drift detection and remediation  
D) Require email approval before manual changes

**Answer**: C

**Explanation**: Automated drift detection (Azure DSC, Ansible) detects unauthorized changes and can automatically remediate them, ensuring configurations stay consistent. Option B is too restrictive (may need emergency access). Options A and D don't prevent drift. Best practice: combine C with monitoring and alerts so team is aware when drift occurs and can investigate root cause.

### Question 3: Environment Parity

**Question**: Your application works perfectly in development but crashes in production with database connection errors. Review shows development uses PostgreSQL 12 while production uses PostgreSQL 11. How should you prevent this issue?

A) Require developers to test on production database  
B) Create documentation specifying exact versions  
C) Use configuration as code to enforce identical versions across environments  
D) Upgrade production database without testing

**Answer**: C

**Explanation**: Configuration as code ensures consistent software versions across environments through automated deployment. Same Ansible playbook or DSC configuration installs identical versions in all environments. Options A and D are dangerous. Option B (documentation) doesn't prevent drift.

### Question 4: Disaster Recovery with Configuration Management

**Question**: Your data center floods, destroying all servers. You have infrastructure code in Git and application data backups. What's your recovery process?

A) Manually rebuild servers following documentation  
B) Restore server images from backup tapes  
C) Use IaC to provision servers, then apply configuration from Git  
D) Wait for hardware replacement in flooded data center

**Answer**: C

**Explanation**: Best practice for disaster recovery: (1) provision infrastructure in new location using IaC, (2) apply configuration using configuration management code from Git, (3) restore application data from backups. This approach works even if disaster destroyed original data center. Options A, B, D are slower and less reliable.

### Question 5: Team Collaboration

**Question**: Your team needs to update database connection pool settings. What configuration as code workflow enables collaboration?

A) One person makes change and informs others via email  
B) Edit configuration file, create pull request, team reviews, approve, merge, auto-deploy  
C) Update production server, then update other environments  
D) Each team member updates their assigned servers

**Answer**: B

**Explanation**: Pull request workflow enables collaboration: proposed changes reviewed by team, discussion of trade-offs, automated tests validate changes, approval before deployment. This is standard configuration as code practice. Options A, C, D represent manual, error-prone approaches without collaboration or review.

## Summary

Environment configuration management represents the critical second half of infrastructure automation, complementing Infrastructure as Code provisioning with ongoing configuration and maintenance. Configuration as code transforms manual, error-prone configuration processes into automated, repeatable, testable workflows that scale across hundreds of servers.

**Key Takeaways**:

1. **Configuration vs. Provisioning**: IaC provisions resources; configuration management configures them. Both essential for complete automation.

2. **Manual Configuration Problems**: Documentation becomes outdated, tribal knowledge creates dependencies, configuration drift causes inconsistencies, manual processes don't scale.

3. **Configuration as Code Benefits**: Reproducible environments, drift detection/correction, faster deployments, complete audit trail, team collaboration, disaster recovery capability.

4. **Version Control Integration**: Storing configuration in Git enables change tracking, code review, rollback, and automated testing before production deployment.

5. **Compliance Advantages**: Configuration as code satisfies regulatory requirements through complete audit trails, policy enforcement, and automated compliance scanning.

Next unit explores the fundamental choice in configuration approaches: imperative (step-by-step procedures) versus declarative (desired state definitions), helping you select the appropriate methodology for different scenarios.

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-infrastructure-code-configuration-management/3-examine-environment-configuration)
