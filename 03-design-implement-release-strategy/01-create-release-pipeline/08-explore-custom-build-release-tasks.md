# Explore Custom Build and Release Tasks

While Azure DevOps provides hundreds of built-in tasks, custom tasks enable organizations to implement specialized deployment logic, integrate proprietary systems, and create reusable components tailored to specific needs.

## Why Create Custom Tasks?

Custom build and release tasks offer significant advantages over standard approaches like command-line scripts or shell operations:

```
Standard Approach (Scripts):

- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: |
      # 50 lines of deployment logic
      # Hard to maintain
      # No parameter validation
      # Secrets exposed in logs
      # Copy-paste across pipelines

Custom Task Approach:

- task: MyCompanyDeploy@1
  inputs:
    environment: 'Production'
    apiKey: '$(SecretApiKey)'  # Secured
    deploymentSlot: 'Blue'
  # Clean interface
  # Reusable across pipelines
  # Centralized updates
```

## Advantages of Custom Tasks

### 1. Access to Restricted Variables

Custom tasks have access to system variables not available in standard scripts.

```typescript
// Custom task can access internal variables
import * as tl from 'azure-pipelines-task-lib/task';

// Variables not available to scripts
const buildId = tl.getVariable('Build.BuildId');
const sourceVersion = tl.getVariable('Build.SourceVersion');
const requestedFor = tl.getVariable('Build.RequestedFor');

// Access secure files
const secureFilePath = tl.getSecureFileInput('certificateFile', true);

// Access endpoint auth
const serviceEndpoint = tl.getEndpointUrl('MyServiceConnection', false);
const authToken = tl.getEndpointAuthorizationParameter(
  'MyServiceConnection',
  'apitoken',
  false
);
```

**Standard Script Limitations**:
```powershell
# Scripts have limited variable access
$buildId = $env:BUILD_BUILDID  # Available
$secureFile = ???  # Not accessible in script!
$serviceEndpoint = ???  # Not accessible!
```

### 2. Reusable Secure Endpoint Connections

Custom tasks can leverage service connections for secure authentication.

```
Service Connection Integration

Pipeline:
  - task: CustomDeployTask@1
    inputs:
      serviceConnection: 'ProductionAWS'  ← Secure connection
      region: 'us-east-1'

Custom Task:
  1. Retrieves credentials from service connection
  2. Authenticates to AWS
  3. Deploys application
  4. Credentials never exposed in logs

Benefits:
  ✅ Centralized credential management
  ✅ No secrets in pipeline YAML
  ✅ Credential rotation without pipeline changes
  ✅ Audit trail of credential usage
```

**Example - Custom Task Using Service Connection**:
```typescript
// custom-task.ts
import * as tl from 'azure-pipelines-task-lib/task';
import * as AWS from 'aws-sdk';

async function run() {
    try {
        // Get service connection name from task input
        const serviceConnection = tl.getInput('serviceConnection', true);
        
        // Retrieve credentials from service connection
        const awsEndpoint = tl.getEndpointUrl(serviceConnection, false);
        const accessKeyId = tl.getEndpointAuthorizationParameter(
            serviceConnection,
            'username',
            false
        );
        const secretAccessKey = tl.getEndpointAuthorizationParameter(
            serviceConnection,
            'password',
            false
        );
        
        // Configure AWS SDK (credentials never logged)
        AWS.config.update({
            accessKeyId: accessKeyId,
            secretAccessKey: secretAccessKey,
            region: tl.getInput('region', true)
        });
        
        // Perform deployment
        const s3 = new AWS.S3();
        await s3.putObject({
            Bucket: tl.getInput('bucketName', true),
            Key: 'app.zip',
            Body: fs.readFileSync('app.zip')
        }).promise();
        
        tl.setResult(tl.TaskResult.Succeeded, 'Deployment successful');
    } catch (err) {
        tl.setResult(tl.TaskResult.Failed, err.message);
    }
}

run();
```

### 3. Organization-Wide Distribution

Custom tasks can be shared privately within an organization or publicly via marketplace.

```
Distribution Options

Private Distribution (Organization-Only):
  ┌─────────────────────────────────┐
  │  Azure DevOps Organization      │
  │  ├── Custom Task Extension      │
  │  └── Available to all projects  │
  └─────────────────────────────────┘
  
  Benefits:
    ✅ Proprietary deployment logic
    ✅ Internal tools integration
    ✅ Company-specific workflows
    ✅ No public exposure

Public Distribution (Marketplace):
  ┌─────────────────────────────────┐
  │  Visual Studio Marketplace      │
  │  ├── Public task extension      │
  │  └── Available to all users     │
  └─────────────────────────────────┘
  
  Benefits:
    ✅ Community contributions
    ✅ Wider adoption
    ✅ Feedback from users
    ✅ Establish expertise
```

### 4. Encapsulated Implementation

Hide complex implementation details behind simple interface.

```
Complex Logic (Before Custom Task):

Pipeline YAML (200 lines):
  - PowerShell: Authenticate to proprietary system
  - PowerShell: Validate deployment prerequisites
  - PowerShell: Upload artifacts
  - PowerShell: Execute deployment
  - PowerShell: Run health checks
  - PowerShell: Send notifications
  - PowerShell: Update tracking system

Problems:
  ❌ Copy-pasted across 50 pipelines
  ❌ Inconsistent implementations
  ❌ Difficult to update
  ❌ Secrets in YAML
  ❌ No validation

With Custom Task:

Pipeline YAML (5 lines):
  - task: CompanyDeploy@2
    inputs:
      environment: 'Production'
      healthCheckUrl: 'https://api.company.com/health'

Benefits:
  ✅ Consistent across all pipelines
  ✅ Update once, applies everywhere
  ✅ Validated inputs
  ✅ Secured credentials
  ✅ Centralized logging
```

## Custom Task Development Process

### 1. Task Structure

A custom task is a Node.js application with specific structure:

```
my-custom-task/
├── task.json                 # Task definition (metadata, inputs, outputs)
├── index.ts                  # Task implementation
├── icon.png                  # Task icon (32x32)
├── package.json              # Node.js dependencies
├── tsconfig.json             # TypeScript configuration
└── tests/
    └── success.ts            # Unit tests
```

### 2. Task Definition (task.json)

```json
{
  "id": "12345678-1234-1234-1234-123456789abc",
  "name": "CustomDeployTask",
  "friendlyName": "Deploy to Custom Platform",
  "description": "Deploys applications to company's proprietary platform",
  "helpMarkDown": "[More Information](https://docs.company.com/deploy)",
  "category": "Deploy",
  "author": "CompanyName",
  "version": {
    "Major": 1,
    "Minor": 0,
    "Patch": 0
  },
  "instanceNameFormat": "Deploy to $(environment)",
  "inputs": [
    {
      "name": "serviceConnection",
      "type": "connectedService:CustomPlatform",
      "label": "Service Connection",
      "required": true,
      "helpMarkDown": "Select service connection to custom platform"
    },
    {
      "name": "environment",
      "type": "pickList",
      "label": "Environment",
      "required": true,
      "options": {
        "Dev": "Development",
        "QA": "Quality Assurance",
        "Prod": "Production"
      },
      "helpMarkDown": "Target environment for deployment"
    },
    {
      "name": "packagePath",
      "type": "filePath",
      "label": "Package Path",
      "required": true,
      "helpMarkDown": "Path to deployment package"
    },
    {
      "name": "healthCheckUrl",
      "type": "string",
      "label": "Health Check URL",
      "required": false,
      "helpMarkDown": "URL to validate deployment health"
    }
  ],
  "execution": {
    "Node10": {
      "target": "index.js"
    }
  }
}
```

### 3. Task Implementation (index.ts)

```typescript
import * as tl from 'azure-pipelines-task-lib/task';
import * as fs from 'fs';
import axios from 'axios';

async function run() {
    try {
        // Get task inputs
        const serviceConnection = tl.getInput('serviceConnection', true);
        const environment = tl.getInput('environment', true);
        const packagePath = tl.getPathInput('packagePath', true);
        const healthCheckUrl = tl.getInput('healthCheckUrl', false);
        
        // Validate inputs
        if (!fs.existsSync(packagePath)) {
            throw new Error(`Package not found: ${packagePath}`);
        }
        
        tl.debug(`Deploying to environment: ${environment}`);
        
        // Get credentials from service connection
        const apiEndpoint = tl.getEndpointUrl(serviceConnection, false);
        const apiKey = tl.getEndpointAuthorizationParameter(
            serviceConnection,
            'apitoken',
            false
        );
        
        // Upload package
        console.log(`Uploading package: ${packagePath}`);
        const packageData = fs.readFileSync(packagePath);
        
        const uploadResponse = await axios.post(
            `${apiEndpoint}/deploy`,
            packageData,
            {
                headers: {
                    'Authorization': `Bearer ${apiKey}`,
                    'X-Environment': environment,
                    'Content-Type': 'application/zip'
                }
            }
        );
        
        console.log(`Deployment initiated: ${uploadResponse.data.deploymentId}`);
        
        // Wait for deployment to complete
        const deploymentId = uploadResponse.data.deploymentId;
        let status = 'pending';
        let attempts = 0;
        const maxAttempts = 60;
        
        while (status === 'pending' && attempts < maxAttempts) {
            await sleep(5000);  // Wait 5 seconds
            
            const statusResponse = await axios.get(
                `${apiEndpoint}/deploy/${deploymentId}`,
                {
                    headers: {
                        'Authorization': `Bearer ${apiKey}`
                    }
                }
            );
            
            status = statusResponse.data.status;
            console.log(`Deployment status: ${status}`);
            attempts++;
        }
        
        if (status !== 'completed') {
            throw new Error(`Deployment failed with status: ${status}`);
        }
        
        // Health check (if URL provided)
        if (healthCheckUrl) {
            console.log(`Running health check: ${healthCheckUrl}`);
            const healthResponse = await axios.get(healthCheckUrl, { timeout: 10000 });
            
            if (healthResponse.status !== 200) {
                throw new Error(`Health check failed: ${healthResponse.status}`);
            }
            
            console.log('Health check passed ✅');
        }
        
        // Set output variable
        tl.setVariable('DeploymentId', deploymentId);
        
        tl.setResult(tl.TaskResult.Succeeded, 'Deployment completed successfully');
        
    } catch (err) {
        tl.setResult(tl.TaskResult.Failed, err.message);
    }
}

function sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
}

run();
```

### 4. Package and Publish

```bash
# Install dependencies
npm install azure-pipelines-task-lib --save

# Compile TypeScript
tsc

# Create extension manifest (vss-extension.json)
{
  "manifestVersion": 1,
  "id": "custom-deploy-extension",
  "publisher": "CompanyName",
  "version": "1.0.0",
  "name": "Custom Deployment Tasks",
  "description": "Custom tasks for deploying to company platform",
  "categories": ["Azure Pipelines"],
  "targets": [{"id": "Microsoft.VisualStudio.Services"}],
  "files": [
    {"path": "CustomDeployTask"}
  ],
  "contributions": [
    {
      "id": "custom-deploy-task",
      "type": "ms.vss-distributed-task.task",
      "targets": ["ms.vss-distributed-task.tasks"],
      "properties": {
        "name": "CustomDeployTask"
      }
    }
  ]
}

# Package extension
tfx extension create --manifest-globs vss-extension.json

# Publish to marketplace (public)
tfx extension publish --manifest-globs vss-extension.json --share-with CompanyName

# Or upload to Azure DevOps (private)
# Extensions → Browse Marketplace → Upload Extension
```

## Input Types

Custom tasks support various input types:

```json
{
  "inputs": [
    // String input
    {
      "name": "textInput",
      "type": "string",
      "label": "Text Input"
    },
    
    // Multi-line text
    {
      "name": "multilineInput",
      "type": "multiLine",
      "label": "Multi-line Input"
    },
    
    // Boolean checkbox
    {
      "name": "booleanInput",
      "type": "boolean",
      "label": "Enable Feature"
    },
    
    // Pick list (dropdown)
    {
      "name": "picklistInput",
      "type": "pickList",
      "label": "Select Option",
      "options": {
        "option1": "Option 1",
        "option2": "Option 2"
      }
    },
    
    // File path
    {
      "name": "fileInput",
      "type": "filePath",
      "label": "Select File"
    },
    
    // Secure file
    {
      "name": "secureFileInput",
      "type": "secureFile",
      "label": "Certificate File"
    },
    
    // Service connection
    {
      "name": "serviceConnectionInput",
      "type": "connectedService:AWS",
      "label": "AWS Connection"
    }
  ]
}
```

## Real-World Examples

### Example 1: Custom Notification Task

```typescript
// Notify company's internal chat system
import * as tl from 'azure-pipelines-task-lib/task';
import axios from 'axios';

async function run() {
    try {
        const chatUrl = tl.getInput('chatWebhookUrl', true);
        const message = tl.getInput('message', true);
        const priority = tl.getInput('priority', false) || 'normal';
        
        // Build rich notification
        const notification = {
            text: message,
            priority: priority,
            buildId: tl.getVariable('Build.BuildId'),
            buildNumber: tl.getVariable('Build.BuildNumber'),
            requestedFor: tl.getVariable('Build.RequestedFor'),
            sourceVersion: tl.getVariable('Build.SourceVersion'),
            repository: tl.getVariable('Build.Repository.Name')
        };
        
        await axios.post(chatUrl, notification);
        
        tl.setResult(tl.TaskResult.Succeeded, 'Notification sent');
    } catch (err) {
        tl.setResult(tl.TaskResult.Failed, err.message);
    }
}

run();
```

### Example 2: Custom Database Migration Task

```typescript
// Run database migrations with rollback support
import * as tl from 'azure-pipelines-task-lib/task';
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function run() {
    try {
        const connectionString = tl.getInput('connectionString', true);
        const migrationsPath = tl.getPathInput('migrationsPath', true);
        const targetVersion = tl.getInput('targetVersion', false);
        
        // Create backup before migration
        console.log('Creating database backup...');
        await createBackup(connectionString);
        
        // Run migrations
        console.log(`Running migrations from: ${migrationsPath}`);
        const migrationCommand = targetVersion
            ? `migrate up ${targetVersion}`
            : `migrate up`;
            
        const { stdout, stderr } = await execAsync(
            `migrate -path ${migrationsPath} -database "${connectionString}" ${migrationCommand}`
        );
        
        console.log(stdout);
        
        // Set output variables
        tl.setVariable('MigrationVersion', stdout.trim());
        
        tl.setResult(tl.TaskResult.Succeeded, 'Migrations completed');
    } catch (err) {
        console.error('Migration failed:', err.message);
        
        // Attempt rollback
        console.log('Attempting rollback...');
        await rollback(connectionString);
        
        tl.setResult(tl.TaskResult.Failed, err.message);
    }
}

async function createBackup(connectionString: string): Promise<void> {
    // Backup logic
}

async function rollback(connectionString: string): Promise<void> {
    // Rollback logic
}

run();
```

## Best Practices

### 1. Input Validation

```typescript
// Validate all inputs
const environment = tl.getInput('environment', true);
const allowedEnvironments = ['dev', 'qa', 'staging', 'prod'];

if (!allowedEnvironments.includes(environment.toLowerCase())) {
    throw new Error(`Invalid environment: ${environment}`);
}
```

### 2. Detailed Logging

```typescript
// Use appropriate log levels
tl.debug('Detailed debug information');
console.log('Standard information');
tl.warning('Warning message');
tl.error('Error message');

// Log sections
console.log('##[section]Starting deployment');
console.log('##[command]npm install');
```

### 3. Secret Handling

```typescript
// Never log secrets
const apiKey = tl.getEndpointAuthorizationParameter('connection', 'apitoken', false);

// ❌ BAD: Logs secret
console.log(`Using API key: ${apiKey}`);

// ✅ GOOD: Doesn't log secret
tl.debug('API key retrieved from service connection');

// Mark secret in logs
tl.setSecret(apiKey);  // Masks value in logs
```

### 4. Error Handling

```typescript
try {
    // Task logic
} catch (err) {
    // Log detailed error
    tl.error(`Deployment failed: ${err.message}`);
    tl.debug(`Stack trace: ${err.stack}`);
    
    // Set failure result
    tl.setResult(tl.TaskResult.Failed, err.message);
}
```

## Critical Notes

🎯 **Reusability**: Custom tasks enable consistent, reusable deployment logic across all pipelines.

💡 **Security**: Custom tasks can securely access service connections and secrets not available to scripts.

⚠️ **Maintenance**: Keep custom tasks updated—breaking changes affect all pipelines using the task.

📊 **Distribution**: Share privately within organization or publicly via marketplace.

🔄 **Versioning**: Use semantic versioning—major version changes indicate breaking changes.

✨ **Documentation**: Provide comprehensive help markdown and examples for task users.

## Quick Reference

### Custom Task Checklist

```
Development:
  ✅ Create task.json (metadata, inputs, outputs)
  ✅ Implement task logic (index.ts)
  ✅ Validate inputs
  ✅ Handle errors gracefully
  ✅ Log appropriately (don't log secrets)
  ✅ Write unit tests

Packaging:
  ✅ Compile TypeScript (tsc)
  ✅ Create extension manifest (vss-extension.json)
  ✅ Package extension (tfx extension create)
  ✅ Test in dev environment

Distribution:
  ✅ Publish to marketplace (public) OR
  ✅ Upload to organization (private)
  ✅ Document usage with examples
  ✅ Provide support contact
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline-devops/8-explore-custom-build-release-tasks)
