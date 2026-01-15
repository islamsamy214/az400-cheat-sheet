# Examine Webhooks

## Overview

Webhooks provide a way to trigger Azure Automation runbooks from external systems using simple HTTP POST requests. Instead of scheduling runbooks or manually starting them, webhooks enable event-driven automation—respond instantly to Azure Monitor alerts, integrate with CI/CD pipelines, connect to third-party services, or create custom automation workflows.

Think of webhooks as "automation endpoints"—expose your runbooks as RESTful APIs that any system can call with a simple HTTP request.

This unit covers webhook fundamentals, creation and configuration, security considerations, parameter passing, response handling, and integration patterns for common scenarios.

## What is a Webhook?

A **webhook** is a unique HTTPS URL that, when called with an HTTP POST request, triggers a runbook execution in Azure Automation.

### Webhook Architecture

**Flow**:
```
External System (Azure DevOps, Monitoring Alert, Custom App)
    ↓ (HTTP POST)
HTTPS Webhook URL (https://...webhooktoken...)
    ↓
Azure Automation Webhook Service
    ↓
Queue Runbook Job
    ↓
Runbook Execution (with parameters from webhook)
    ↓
HTTP Response (202 Accepted, JobId returned immediately)
    ↓
External System (monitors job status if needed)
```

### Key Characteristics

**Webhooks Are**:
- **Event-Driven**: Triggered by external events, not schedules
- **Asynchronous**: Return immediately (HTTP 202), runbook runs in background
- **Stateless**: Each webhook call is independent
- **Secure**: URL contains unique security token
- **Parameterized**: Accept JSON payload with runbook parameters
- **Time-Limited**: Expire after configured period (default: 10 years)

**Use Cases**:
- **CI/CD Integration**: Trigger deployment runbooks from Azure DevOps pipelines
- **Monitoring Alerts**: Auto-remediate issues when Azure Monitor alerts fire
- **Incident Response**: ServiceNow or PagerDuty triggers runbooks for automated remediation
- **Third-Party Integration**: Slack, Teams, or custom apps trigger automation
- **IoT Workflows**: IoT devices trigger infrastructure scaling or maintenance
- **Cost Management**: Budget alerts trigger resource cleanup or rightsizing

## Creating Webhooks

### Method 1: Azure Portal

**Steps**:

1. **Navigate to Runbook**:
   - Automation Account → Runbooks → Select runbook
   - Ensure runbook is **Published** (webhooks only work with published runbooks)

2. **Add Webhook**:
   - Click **Webhooks** in left menu
   - Click **Add Webhook**
   - Click **Create new webhook**

3. **Configure Webhook**:
   ```
   Name: webhook-start-vms-from-azdo
   Enabled: Yes (toggle on)
   Expires: 12/31/2026 (or custom date)
   ```

4. **Copy Webhook URL** ⚠️ **CRITICAL**:
   ```
   https://s7events.azure-automation.net/webhooks\?token\=ABC123XYZ789...
   
   ⚠️ WARNING: This URL contains a security token and cannot be retrieved later.
   Copy it immediately and store securely (Azure Key Vault, password manager).
   ```

5. **Configure Parameters** (if runbook has parameters):
   ```json
   {
     "ResourceGroupName": "rg-production-vms",
     "Environment": "Production"
   }
   ```
   
   - These are **default values**
   - Webhook callers can override these values

6. **Configure Run Settings**:
   - **Run on**: Azure (cloud) or Hybrid Runbook Worker
   - **Hybrid Worker Group**: Select if targeting on-premises

7. **Create Webhook**:
   - Click **Create**
   - Webhook is now active and ready to receive calls

### Method 2: PowerShell

```powershell
# Create webhook via PowerShell
$webhookName = "webhook-stop-vms-from-monitoring"
$runbookName = "Stop-AzureVMs"
$resourceGroup = "rg-automation-prod"
$automationAccount = "automation-account-prod"

# Set expiration (10 years from now)
$expiryTime = (Get-Date).AddYears(10)

# Create webhook
$webhook = New-AzAutomationWebhook `
    -ResourceGroupName $resourceGroup `
    -AutomationAccountName $automationAccount `
    -Name $webhookName `
    -RunbookName $runbookName `
    -IsEnabled $true `
    -ExpiryTime $expiryTime `
    -Parameters @{
        ResourceGroupName = "rg-production-vms"
        Action = "Stop"
    } `
    -Force

# ⚠️ CRITICAL: Save webhook URL immediately (can't retrieve later)
Write-Output "Webhook URL (save immediately):"
Write-Output $webhook.WebhookURI

# Store in Key Vault for secure access
$secretValue = ConvertTo-SecureString $webhook.WebhookURI -AsPlainText -Force
Set-AzKeyVaultSecret `
    -VaultName "kv-automation-secrets" `
    -Name "Webhook-StopVMs-Monitoring" `
    -SecretValue $secretValue
```

**Output**:
```
Webhook URL (save immediately):
https://s7events.azure-automation.net/webhooks\?token\=1a2b3c4d5e6f7g8h9i0j...

Webhook URL stored in Key Vault: kv-automation-secrets/Webhook-StopVMs-Monitoring
```

### Method 3: Azure CLI

```bash
#!/bin/bash
# Create webhook via Azure CLI

RESOURCE_GROUP="rg-automation-prod"
AUTOMATION_ACCOUNT="automation-account-prod"
RUNBOOK_NAME="Restart-AzureVMs"
WEBHOOK_NAME="webhook-restart-vms-from-app"
EXPIRY_DATE="2034-12-31T23:59:59Z"  # 10 years

# Create webhook
WEBHOOK_URL=$(az automation webhook create \
    --resource-group $RESOURCE_GROUP \
    --automation-account-name $AUTOMATION_ACCOUNT \
    --name $WEBHOOK_NAME \
    --runbook-name $RUNBOOK_NAME \
    --expiry-time $EXPIRY_DATE \
    --is-enabled true \
    --parameters '{"ResourceGroupName":"rg-prod-vms","Environment":"Production"}' \
    --query uri -o tsv)

# Save URL
echo "⚠️ SAVE THIS URL IMMEDIATELY (cannot retrieve later):"
echo $WEBHOOK_URL

# Store in file (encrypt this file)
echo $WEBHOOK_URL > webhook-restart-vms.secret
chmod 600 webhook-restart-vms.secret
```

## Webhook Configuration Options

### Webhook Properties

| Property | Description | Example | Required |
|----------|-------------|---------|----------|
| **Name** | Unique identifier for webhook | webhook-start-vms-azdo | Yes |
| **Enabled** | Whether webhook accepts calls | true / false | Yes |
| **Expires** | Expiration date/time (UTC) | 2034-12-31T23:59:59Z | Yes |
| **URL** | Unique HTTPS endpoint with token | https://s7events.azure-automation.net/webhooks\?token\=... | Auto-generated |
| **Runbook** | Target runbook to execute | Start-AzureVMs | Yes |
| **Parameters** | Default parameter values (JSON) | {"ResourceGroupName":"rg-prod"} | Optional |
| **Run On** | Execution location | Azure or Hybrid Worker | Optional |

### Expiration Strategy

**Expiration Guidelines**:

```
Short-Term (30-90 days):
└── Use for: Temporary integrations, testing, proof-of-concepts
└── Example: Demo at conference, short-term project

Medium-Term (1-2 years):
└── Use for: Project-specific automation with defined lifecycle
└── Example: Migration project, time-bound initiative

Long-Term (5-10 years):
└── Use for: Production automation with long-term value
└── Example: CI/CD integration, monitoring alerts, incident response
└── ⚠️ Warning: Set calendar reminder for rotation before expiration
```

**Expiration Monitoring**:

```powershell
# Find webhooks expiring within 90 days
$webhooks = Get-AzAutomationWebhook -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod"

$expiringWebhooks = $webhooks | Where-Object {
    $_.ExpiryTime -lt (Get-Date).AddDays(90) -and $_.IsEnabled
}

if ($expiringWebhooks) {
    Write-Warning "Webhooks expiring within 90 days:"
    $expiringWebhooks | Select-Object Name, RunbookName, ExpiryTime | Format-Table
    
    # Send alert email
    $body = $expiringWebhooks | ConvertTo-Html -Property Name, RunbookName, ExpiryTime
    Send-MailMessage -To "automation-team@company.com" `
        -Subject "Azure Automation Webhooks Expiring Soon" `
        -Body ($body | Out-String) -BodyAsHtml
}
```

## Using Webhooks

### Basic Webhook Call

**cURL Example**:

```bash
#!/bin/bash
# Simple webhook invocation

WEBHOOK_URL="https://s7events.azure-automation.net/webhooks?token=ABC123XYZ789..."

# Call webhook (no parameters)
curl -X POST $WEBHOOK_URL \
    -H "Content-Type: application/json"
```

**Response**:
```
HTTP/1.1 202 Accepted
Content-Type: application/json

{
  "JobIds": ["12345678-1234-1234-1234-123456789012"]
}
```

### Webhook Call with Parameters

**PowerShell Example**:

```powershell
# Call webhook with custom parameters
$webhookUrl = "https://s7events.azure-automation.net/webhooks?token=ABC123..."

$body = @{
    ResourceGroupName = "rg-production-vms"
    Environment = "Production"
    Action = "Start"
    NotificationEmail = "ops-team@company.com"
} | ConvertTo-Json

$headers = @{
    "Content-Type" = "application/json"
}

try {
    $response = Invoke-RestMethod -Uri $webhookUrl -Method Post -Body $body -Headers $headers
    Write-Output "Webhook invoked successfully"
    Write-Output "Job ID: $($response.JobIds[0])"
}
catch {
    Write-Error "Webhook invocation failed: $_"
}
```

**Python Example**:

```python
import requests
import json

webhook_url = "https://s7events.azure-automation.net/webhooks?token=ABC123..."

payload = {
    "ResourceGroupName": "rg-production-vms",
    "Environment": "Production",
    "Action": "Stop",
    "NotificationEmail": "ops-team@company.com"
}

headers = {
    "Content-Type": "application/json"
}

try:
    response = requests.post(webhook_url, json=payload, headers=headers)
    response.raise_for_status()
    
    print(f"Webhook invoked successfully")
    print(f"Status Code: {response.status_code}")
    print(f"Job ID: {response.json()['JobIds'][0]}")
except requests.exceptions.RequestException as e:
    print(f"Webhook invocation failed: {e}")
```

### Accessing Webhook Data in Runbooks

**Runbook Side**:

```powershell
param(
    # Regular parameters (can be overridden by webhook)
    [string]$ResourceGroupName = "rg-default",
    [string]$Environment = "Development",
    
    # WebhookData parameter (automatically populated by webhook service)
    [object]$WebhookData
)

# Check if invoked via webhook
if ($WebhookData) {
    Write-Output "Runbook started from webhook: $($WebhookData.WebhookName)"
    
    # Access webhook properties
    Write-Output "Request Headers: $($WebhookData.RequestHeader | ConvertTo-Json)"
    Write-Output "Request Body: $($WebhookData.RequestBody)"
    
    # Parse parameters from webhook body
    if ($WebhookData.RequestBody) {
        $params = $WebhookData.RequestBody | ConvertFrom-Json
        
        # Override default parameters with webhook values
        if ($params.ResourceGroupName) {
            $ResourceGroupName = $params.ResourceGroupName
        }
        if ($params.Environment) {
            $Environment = $params.Environment
        }
        
        Write-Output "Using parameters from webhook:"
        Write-Output "  ResourceGroupName: $ResourceGroupName"
        Write-Output "  Environment: $Environment"
    }
}
else {
    Write-Output "Runbook started manually or via schedule"
    Write-Output "Using default/provided parameters:"
    Write-Output "  ResourceGroupName: $ResourceGroupName"
    Write-Output "  Environment: $Environment"
}

# Continue with runbook logic using $ResourceGroupName and $Environment
# ... automation code ...
```

**WebhookData Structure**:

```json
{
  "WebhookName": "webhook-start-vms-from-azdo",
  "RequestHeader": {
    "Connection": "Keep-Alive",
    "Content-Length": "123",
    "Content-Type": "application/json",
    "User-Agent": "Azure-DevOps/1.0"
  },
  "RequestBody": "{\"ResourceGroupName\":\"rg-prod-vms\",\"Environment\":\"Production\"}"
}
```

## HTTP Response Codes

Webhook calls return different HTTP status codes based on the outcome:

| Status Code | Meaning | Description | Action |
|-------------|---------|-------------|--------|
| **202 Accepted** | Success | Runbook job queued successfully | Check JobId for status |
| **400 Bad Request** | Client Error | Invalid JSON payload or parameters | Fix request format |
| **404 Not Found** | Not Found | Webhook doesn't exist or expired | Verify URL, check expiration |
| **500 Internal Server Error** | Server Error | Azure Automation service issue | Retry after delay |

### Response Format

**Successful Response** (HTTP 202):

```json
{
  "JobIds": ["12345678-1234-1234-1234-123456789012"]
}
```

**Error Response** (HTTP 400):

```json
{
  "error": {
    "code": "BadRequest",
    "message": "Invalid JSON payload in request body"
  }
}
```

### Monitoring Job Status

After webhook returns JobId, monitor execution:

```powershell
# Call webhook
$response = Invoke-RestMethod -Uri $webhookUrl -Method Post -Body $body -Headers $headers
$jobId = $response.JobIds[0]

Write-Output "Job started: $jobId"

# Wait for job completion
$timeout = 300  # 5 minutes
$elapsed = 0
$interval = 10  # Check every 10 seconds

do {
    Start-Sleep -Seconds $interval
    $elapsed += $interval
    
    $job = Get-AzAutomationJob -Id $jobId `
        -ResourceGroupName "rg-automation-prod" `
        -AutomationAccountName "automation-account-prod"
    
    Write-Output "Job status: $($job.Status) (elapsed: ${elapsed}s)"
    
    if ($job.Status -in @("Completed", "Failed", "Stopped", "Suspended")) {
        break
    }
} while ($elapsed -lt $timeout)

# Check final status
if ($job.Status -eq "Completed") {
    Write-Output "✓ Job completed successfully"
    
    # Get job output
    $output = Get-AzAutomationJobOutput -Id $jobId `
        -ResourceGroupName "rg-automation-prod" `
        -AutomationAccountName "automation-account-prod" `
        -Stream Output
    
    Write-Output "Job Output:"
    Write-Output $output.Summary
}
elseif ($job.Status -eq "Failed") {
    Write-Error "✗ Job failed"
    
    # Get error details
    $errorOutput = Get-AzAutomationJobOutput -Id $jobId `
        -ResourceGroupName "rg-automation-prod" `
        -AutomationAccountName "automation-account-prod" `
        -Stream Error
    
    Write-Error "Error: $($errorOutput.Summary)"
}
else {
    Write-Warning "⚠ Job did not complete within timeout (status: $($job.Status))"
}
```

## Security Considerations

### Webhook URL Protection

**Treat Webhook URLs as Passwords**:

```
Webhook URL:
https://s7events.azure-automation.net/webhooks\?token\=SECRET_TOKEN_HERE
                                                       ↑
                                        This token is your authentication
```

**Security Best Practices**:

1. **Never Commit to Source Control**:
   ```bash
   # ❌ BAD: Hardcoded webhook URL in code
   WEBHOOK_URL="https://s7events.azure-automation.net/webhooks?token=ABC123..."
   
   # ✅ GOOD: Store in environment variable or Key Vault
   WEBHOOK_URL=$(az keyvault secret show --vault-name "kv-automation" --name "webhook-start-vms" --query value -o tsv)
   ```

2. **Use Azure Key Vault**:
   ```powershell
   # Store webhook URL in Key Vault
   $secretValue = ConvertTo-SecureString $webhookUrl -AsPlainText -Force
   Set-AzKeyVaultSecret -VaultName "kv-automation-secrets" `
       -Name "Webhook-StartVMs-Production" `
       -SecretValue $secretValue
   
   # Retrieve when needed
   $webhookUrl = Get-AzKeyVaultSecret -VaultName "kv-automation-secrets" `
       -Name "Webhook-StartVMs-Production" -AsPlainText
   ```

3. **Limit Access**:
   - Grant Key Vault access only to authorized users/apps
   - Use Managed Identity for apps calling webhooks
   - Rotate webhook URLs periodically (recreate webhook)

4. **Use HTTPS Only**:
   - Webhook URLs are HTTPS by default
   - Never downgrade to HTTP
   - Validate SSL certificates

### Parameter Validation in Runbooks

**Defend Against Malicious Input**:

```powershell
param(
    [object]$WebhookData
)

if ($WebhookData) {
    $params = $WebhookData.RequestBody | ConvertFrom-Json
    
    # ✅ GOOD: Validate all parameters
    if ($params.ResourceGroupName) {
        # Whitelist validation: Only allow specific resource groups
        $allowedResourceGroups = @("rg-prod-vms", "rg-staging-vms", "rg-dev-vms")
        
        if ($params.ResourceGroupName -notin $allowedResourceGroups) {
            Write-Error "Invalid resource group: $($params.ResourceGroupName)"
            Write-Error "Allowed values: $($allowedResourceGroups -join ', ')"
            throw "Parameter validation failed"
        }
        
        $resourceGroupName = $params.ResourceGroupName
    }
    
    # Validate action parameter
    if ($params.Action) {
        $validActions = @("Start", "Stop", "Restart")
        
        if ($params.Action -notin $validActions) {
            Write-Error "Invalid action: $($params.Action)"
            throw "Parameter validation failed"
        }
        
        $action = $params.Action
    }
    
    # Sanitize string inputs (prevent injection)
    if ($params.NotificationEmail) {
        # Validate email format
        if ($params.NotificationEmail -notmatch "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$") {
            Write-Error "Invalid email format: $($params.NotificationEmail)"
            throw "Parameter validation failed"
        }
        
        $notificationEmail = $params.NotificationEmail
    }
}
```

### Webhook Auditing

**Track Webhook Usage**:

```powershell
# Log webhook invocations to Log Analytics
param([object]$WebhookData)

if ($WebhookData) {
    $auditLog = @{
        Timestamp = (Get-Date).ToUniversalTime().ToString("o")
        WebhookName = $WebhookData.WebhookName
        CallerIP = $WebhookData.RequestHeader["X-Forwarded-For"]
        UserAgent = $WebhookData.RequestHeader["User-Agent"]
        Parameters = $WebhookData.RequestBody
        JobId = $PSPrivateMetadata.JobId.Guid
    } | ConvertTo-Json
    
    Write-Output "Audit Log: $auditLog"
    
    # Send to Log Analytics (Application Insights)
    $workspaceId = Get-AutomationVariable -Name "LogAnalytics-WorkspaceId"
    $sharedKey = Get-AutomationVariable -Name "LogAnalytics-SharedKey"
    
    # ... Send log data to Log Analytics ...
}
```

## Integration Patterns

### Pattern 1: Azure DevOps Pipeline Integration

**Scenario**: Deploy infrastructure from Azure DevOps pipeline

**Azure Pipeline YAML**:

```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: automation-webhooks  # Variable group with webhook URLs

stages:
  - stage: Deploy
    jobs:
      - job: TriggerInfrastructureDeployment
        steps:
          - task: Bash@3
            displayName: 'Trigger Azure Automation Runbook'
            inputs:
              targetType: 'inline'
              script: |
                # Retrieve webhook URL from variable
                WEBHOOK_URL="$(webhook-deploy-infrastructure)"
                
                # Build payload
                PAYLOAD=$(cat <<EOF
                {
                  "Environment": "Production",
                  "ResourceGroupName": "rg-prod-app",
                  "DeploymentId": "$(Build.BuildId)",
                  "RequestedBy": "$(Build.RequestedFor)"
                }
                EOF
                )
                
                # Invoke webhook
                RESPONSE=$(curl -X POST "$WEBHOOK_URL" \
                  -H "Content-Type: application/json" \
                  -d "$PAYLOAD" \
                  -w "%{http_code}" -o response.json)
                
                # Check response
                if [ $RESPONSE -eq 202 ]; then
                  echo "✓ Runbook triggered successfully"
                  JOB_ID=$(cat response.json | jq -r '.JobIds[0]')
                  echo "Job ID: $JOB_ID"
                  echo "##vso[task.setvariable variable=AutomationJobId]$JOB_ID"
                else
                  echo "✗ Webhook invocation failed with status $RESPONSE"
                  exit 1
                fi
          
          - task: PowerShell@2
            displayName: 'Monitor Runbook Execution'
            inputs:
              targetType: 'inline'
              script: |
                $jobId = "$(AutomationJobId)"
                $timeout = 600  # 10 minutes
                
                Write-Host "Monitoring job: $jobId"
                
                # Poll job status
                # ... (monitoring logic from earlier example) ...
```

### Pattern 2: Azure Monitor Alert Integration

**Scenario**: Auto-remediate when CPU exceeds threshold

**Alert Action Group Configuration**:

1. **Create Action Group**:
   - Azure Portal → Monitor → Alerts → Action groups → Create
   - Name: "ag-auto-remediate-high-cpu"
   - Action Type: **Webhook**
   - Webhook URL: `https://s7events.azure-automation.net/webhooks?token=...`

2. **Alert Rule**:
   - Resource: Virtual Machine
   - Condition: CPU > 90% for 5 minutes
   - Action Group: ag-auto-remediate-high-cpu

3. **Runbook** (receives alert context):

```powershell
param([object]$WebhookData)

if ($WebhookData) {
    # Parse Azure Monitor alert data
    $alertContext = ($WebhookData.RequestBody | ConvertFrom-Json).data.context
    
    $resourceId = $alertContext.resourceId
    $vmName = $resourceId.Split('/')[-1]
    $resourceGroup = $alertContext.resourceGroupName
    
    Write-Output "High CPU alert for VM: $vmName in $resourceGroup"
    Write-Output "CPU value: $($alertContext.condition.allOf[0].metricValue)%"
    
    # Auto-remediation: Restart VM
    Write-Output "Restarting VM to remediate high CPU..."
    
    # Authenticate
    $connection = Get-AutomationConnection -Name "AzureRunAsConnection"
    Connect-AzAccount -ServicePrincipal `
        -Tenant $connection.TenantID `
        -ApplicationId $connection.ApplicationID `
        -CertificateThumbprint $connection.CertificateThumbprint
    
    # Restart VM
    Restart-AzVM -ResourceGroupName $resourceGroup -Name $vmName -NoWait
    
    Write-Output "✓ VM restart initiated"
    
    # Send notification
    $teamsWebhook = Get-AutomationVariable -Name "Teams-OpsChannel-Webhook"
    $message = @{
        text = "🔄 Auto-remediation: VM '$vmName' restarted due to high CPU ($($ alertContext.condition.allOf[0].metricValue)%)"
    } | ConvertTo-Json
    
    Invoke-RestMethod -Uri $teamsWebhook -Method Post -Body $message -ContentType "application/json"
}
```

### Pattern 3: ServiceNow Integration

**Scenario**: Trigger runbook from ServiceNow incident

**ServiceNow Workflow**:

```javascript
// ServiceNow Business Rule: On Incident Creation
(function executeRule(current, previous) {
    try {
        var webhookUrl = gs.getProperty('azure.automation.webhook.incident.response');
        var payload = {
            IncidentNumber: current.number.toString(),
            Priority: current.priority.toString(),
            ShortDescription: current.short_description.toString(),
            AssignedTo: current.assigned_to.getDisplayValue(),
            Category: current.category.toString()
        };
        
        var request = new sn_ws.RESTMessageV2();
        request.setEndpoint(webhookUrl);
        request.setHttpMethod('POST');
        request.setRequestHeader('Content-Type', 'application/json');
        request.setRequestBody(JSON.stringify(payload));
        
        var response = request.execute();
        var statusCode = response.getStatusCode();
        
        if (statusCode == 202) {
            var responseBody = JSON.parse(response.getBody());
            var jobId = responseBody.JobIds[0];
            
            gs.addInfoMessage('Azure Automation runbook triggered. Job ID: ' + jobId);
            current.work_notes = 'Azure Automation runbook triggered for automated remediation. Job ID: ' + jobId;
            current.update();
        } else {
            gs.addErrorMessage('Failed to trigger Azure Automation runbook. Status: ' + statusCode);
        }
    } catch (ex) {
        gs.error('Error triggering Azure Automation webhook: ' + ex.message);
    }
})(current, previous);
```

### Pattern 4: Slack/Teams Bot Integration

**Scenario**: Trigger runbooks via chat commands

**Slack Bot**:

```python
# Slack bot using Bolt framework
from slack_bolt import App
import requests
import os

app = App(token=os.environ["SLACK_BOT_TOKEN"])

@app.command("/start-vms")
def start_vms_command(ack, command, say):
    ack()
    
    # Parse parameters from command text
    # Example: /start-vms env=prod rg=rg-prod-vms
    params = {}
    for param in command['text'].split():
        key, value = param.split('=')
        params[key] = value
    
    # Validate required parameters
    if 'env' not in params or 'rg' not in params:
        say("❌ Missing required parameters. Usage: /start-vms env=<environment> rg=<resource-group>")
        return
    
    # Retrieve webhook URL from environment
    webhook_url = os.environ["AZURE_WEBHOOK_START_VMS"]
    
    # Build payload
    payload = {
        "Environment": params['env'],
        "ResourceGroupName": params['rg'],
        "RequestedBy": command['user_name']
    }
    
    try:
        # Invoke webhook
        response = requests.post(webhook_url, json=payload)
        response.raise_for_status()
        
        job_id = response.json()['JobIds'][0]
        
        say(f"✅ VM start runbook triggered!\n"
            f"Environment: {params['env']}\n"
            f"Resource Group: {params['rg']}\n"
            f"Job ID: `{job_id}`\n"
            f"Monitor: https://portal.azure.com/...")
    except requests.exceptions.RequestException as e:
        say(f"❌ Failed to trigger runbook: {str(e)}")

if __name__ == "__main__":
    app.start(port=3000)
```

## Best Practices

### 1. Design for Idempotency

Webhooks may be called multiple times (retries, duplicate events):

```powershell
param([object]$WebhookData)

# Use idempotency key to prevent duplicate execution
if ($WebhookData) {
    $params = $WebhookData.RequestBody | ConvertFrom-Json
    $idempotencyKey = $params.IdempotencyKey
    
    if ($idempotencyKey) {
        # Check if this key was already processed
        $processedKeys = Get-AutomationVariable -Name "ProcessedIdempotencyKeys"
        
        if ($processedKeys -contains $idempotencyKey) {
            Write-Output "Duplicate request detected (key: $idempotencyKey). Skipping execution."
            exit
        }
        
        # Mark key as processed
        $processedKeys += $idempotencyKey
        Set-AutomationVariable -Name "ProcessedIdempotencyKeys" -Value $processedKeys
    }
}

# Continue with runbook logic
```

### 2. Return Quickly (Asynchronous Design)

Webhooks timeout after 30 seconds. Don't wait for long operations:

```powershell
# ❌ BAD: Wait for operation to complete (may timeout)
Start-AzVM -ResourceGroupName $rg -Name $vmName  # Blocks until VM starts

# ✅ GOOD: Start asynchronously and return immediately
Start-AzVM -ResourceGroupName $rg -Name $vmName -NoWait  # Returns immediately
```

### 3. Implement Retry Logic (Caller Side)

```python
import requests
import time

def invoke_webhook_with_retry(webhook_url, payload, max_retries=3):
    for attempt in range(1, max_retries + 1):
        try:
            response = requests.post(webhook_url, json=payload, timeout=30)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            if attempt < max_retries:
                wait_time = 2 ** attempt  # Exponential backoff: 2, 4, 8 seconds
                print(f"Attempt {attempt} failed: {e}. Retrying in {wait_time}s...")
                time.sleep(wait_time)
            else:
                print(f"All {max_retries} attempts failed")
                raise
```

### 4. Monitor Webhook Health

```powershell
# Scheduled runbook: Test all webhooks weekly
$webhooks = Get-AzAutomationWebhook -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" | Where-Object {$_.IsEnabled}

foreach ($webhook in $webhooks) {
    # Check expiration
    $daysUntilExpiry = ($webhook.ExpiryTime - (Get-Date)).TotalDays
    
    if ($daysUntilExpiry -lt 30) {
        Write-Warning "Webhook '$($webhook.Name)' expires in $([math]::Round($daysUntilExpiry)) days"
    }
    
    # Note: Cannot test webhook invocation automatically (URL not retrievable)
    # Manual testing or integration tests required
}
```

## Troubleshooting

### Issue 1: HTTP 404 Not Found

**Causes**:
- Webhook expired
- Webhook deleted
- Incorrect URL (typo, truncation)

**Solution**:
```powershell
# Verify webhook exists and is enabled
Get-AzAutomationWebhook -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" -Name "webhook-name" |
    Select-Object Name, IsEnabled, ExpiryTime
```

### Issue 2: Webhook Returns 202 but Runbook Doesn't Execute

**Causes**:
- Runbook not published
- Permissions issue (Run As account)
- Module dependencies missing

**Solution**:
```powershell
# Check runbook state
Get-AzAutomationRunbook -Name "MyRunbook" -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod" | Select-Object State

# If state is "Edit", publish it:
Publish-AzAutomationRunbook -Name "MyRunbook" -ResourceGroupName "rg-automation-prod" `
    -AutomationAccountName "automation-account-prod"
```

### Issue 3: Parameters Not Reaching Runbook

**Cause**: Incorrect JSON structure

**Solution**:
```bash
# ❌ BAD: Invalid JSON
curl -X POST "$WEBHOOK_URL" -d "ResourceGroupName=rg-prod"

# ✅ GOOD: Valid JSON
curl -X POST "$WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"ResourceGroupName":"rg-prod","Environment":"Production"}'
```

## Next Steps

Now that you understand webhooks for event-driven automation, proceed to **Unit 7: Explore Source Control Integration** to learn how to manage your runbooks in GitHub or Azure DevOps, enabling version control, collaboration, and CI/CD for your automation code.

**Quick Start Actions**:
1. Create a simple test runbook (e.g., writes parameters to output)
2. Create a webhook for the runbook
3. Test the webhook using cURL or PowerShell
4. Monitor the job execution in Azure Portal
5. Store webhook URL in Azure Key Vault

---

**Module**: Explore Azure Automation with DevOps  
**Learning Path**: Manage infrastructure as code using Azure and DSC  
**Original Content**: https://learn.microsoft.com/en-us/training/modules/explore-azure-automation-devops/6-examine-webhooks
