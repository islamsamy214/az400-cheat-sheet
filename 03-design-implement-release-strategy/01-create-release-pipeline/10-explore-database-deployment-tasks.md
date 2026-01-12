# Explore Database Deployment Tasks

Database deployments require special consideration for schema changes, data migrations, and rollback strategies. Azure Pipelines provides specialized tasks for deploying databases to SQL Server, Azure SQL Database, and other database platforms.

## Database Deployment Challenges

**Why database deployment is complex**:

```
Application Deployment (Stateless):
  Old version → New version
  ✅ Easy: Replace files, restart service
  ✅ Rollback: Redeploy previous version

Database Deployment (Stateful):
  Schema V1 + Data → Schema V2 + Migrated Data
  ⚠️ Complex: Data must be preserved
  ⚠️ Rollback: Can't simply "undo" (data changed)
  ⚠️ Downtime: Schema changes may require locks
```

**Key Challenges**:

| Challenge | Description | Solution |
|-----------|-------------|----------|
| **Data Preservation** | Can't lose production data | Test migrations thoroughly |
| **Schema Changes** | ALTER statements may lock tables | Use online schema changes |
| **Rollback Complexity** | Can't "undo" data changes | Forward-only migrations |
| **Downtime** | Some changes require locks | Blue-green database deployment |
| **Data Migration** | Transform existing data | Write migration scripts |
| **Referential Integrity** | Foreign keys complicate changes | Order migrations carefully |

## Azure SQL Database Deployment Tasks

### 1. SQL Azure DACPAC Deployment

**Deploy database schema using Data-Tier Application Package** (DACPAC):

```yaml
- task: SqlAzureDacpacDeployment@1
  displayName: 'Deploy Azure SQL Database'
  inputs:
    azureSubscription: 'MyAzureConnection'
    AuthenticationType: 'servicePrincipal'
    ServerName: 'myserver.database.windows.net'
    DatabaseName: 'MyProductionDB'
    SqlUsername: '$(sqlUsername)'  # Optional if using service principal
    SqlPassword: '$(sqlPassword)'  # Secure variable
    deployType: 'DacpacTask'
    DeploymentAction: 'Publish'
    DacpacFile: '$(Pipeline.Workspace)/database/MyDatabase.dacpac'
    AdditionalArguments: '/p:BlockOnPossibleDataLoss=true'
```

**What is a DACPAC?**

A **Data-Tier Application Package** containing:
- Database schema (tables, views, stored procedures)
- Deployment metadata
- Deployment scripts (pre/post)
- No actual data (schema only)

**DACPAC Workflow**:

```
1. Developer builds database project (SQL Server Data Tools)
   ↓
2. Build produces .dacpac file
   ↓
3. Pipeline publishes .dacpac as artifact
   ↓
4. Release pipeline deploys .dacpac to target database
   ↓
5. SqlPackage.exe compares schema and generates ALTER scripts
   ↓
6. Scripts execute, updating database schema
```

**DACPAC Deployment Options**:

```yaml
# Block deployment if data loss possible
AdditionalArguments: '/p:BlockOnPossibleDataLoss=true'

# Ignore specific differences
AdditionalArguments: '/p:IgnoreColumnOrder=true /p:IgnoreComments=true'

# Generate script only (don't execute)
AdditionalArguments: '/a:Script /of:true /op:DeploymentScript.sql'

# Drop objects not in source
AdditionalArguments: '/p:DropObjectsNotInSource=true'

# Backup database before deployment
AdditionalArguments: '/p:BackupDatabaseBeforeChanges=true'
```

### 2. Azure SQL Database Deployment Task

**Execute SQL scripts directly**:

```yaml
- task: SqlAzureDacpacDeployment@1
  displayName: 'Run SQL Scripts'
  inputs:
    azureSubscription: 'MyAzureConnection'
    ServerName: 'myserver.database.windows.net'
    DatabaseName: 'MyDatabase'
    deployType: 'SqlTask'
    SqlFile: '$(Pipeline.Workspace)/scripts/migration.sql'
    SqlAdditionalArguments: '-v Variable1="Value1" -v Variable2="Value2"'
```

**SQL Script Task vs DACPAC**:

| Aspect | SQL Scripts | DACPAC |
|--------|-------------|--------|
| **Schema Comparison** | ❌ No (runs script as-is) | ✅ Yes (compares and generates) |
| **Idempotency** | Manual (use IF EXISTS) | Automatic |
| **Data Loss Protection** | ❌ No | ✅ Yes (with flag) |
| **Flexibility** | ✅ High (any SQL) | ⚠️ Limited to schema |
| **Rollback** | Manual | Manual |
| **Use Case** | Data migrations, hotfixes | Schema deployments |

### 3. Inline SQL Execution

```yaml
- task: SqlAzureDacpacDeployment@1
  displayName: 'Execute Inline SQL'
  inputs:
    azureSubscription: 'MyAzureConnection'
    ServerName: 'myserver.database.windows.net'
    DatabaseName: 'MyDatabase'
    deployType: 'InlineSqlTask'
    SqlInline: |
      -- Update application version
      UPDATE Configuration
      SET [Value] = '$(Build.BuildNumber)'
      WHERE [Key] = 'AppVersion';
      
      -- Log deployment
      INSERT INTO DeploymentHistory (DeployedAt, Version, DeployedBy)
      VALUES (GETUTCDATE(), '$(Build.BuildNumber)', '$(Build.RequestedFor)');
```

## Database Migration Strategies

### 1. Expand-Contract Pattern

**Safe schema changes without downtime**:

```
Phase 1: Expand (Add new schema alongside old)
┌──────────────┐
│ Old Column   │ ← Application still uses this
│ New Column   │ ← Add new column (nullable)
└──────────────┘

Phase 2: Migrate (Copy data from old to new)
┌──────────────┐
│ Old Column   │ ← Data here
│ New Column   │ ← Copy data here
└──────────────┘

Phase 3: Cutover (Application uses new schema)
┌──────────────┐
│ Old Column   │ ← Application no longer uses
│ New Column   │ ← Application now uses this
└──────────────┘

Phase 4: Contract (Remove old schema)
┌──────────────┐
│ New Column   │ ← Old column removed
└──────────────┘
```

**Example: Rename Column**:

```sql
-- Phase 1: Expand (add new column)
ALTER TABLE Users ADD Email_New NVARCHAR(255);

-- Phase 2: Migrate (copy data)
UPDATE Users SET Email_New = Email_Old;

-- Phase 3: Cutover (deploy new application version using Email_New)
-- Application deployment happens here

-- Phase 4: Contract (remove old column)
ALTER TABLE Users DROP COLUMN Email_Old;
```

**YAML Pipeline for Expand-Contract**:

```yaml
stages:
- stage: Phase1_Expand
  jobs:
  - job: AddNewColumn
    steps:
    - task: SqlAzureDacpacDeployment@1
      inputs:
        SqlInline: 'ALTER TABLE Users ADD Email_New NVARCHAR(255);'

- stage: Phase2_Migrate
  dependsOn: Phase1_Expand
  jobs:
  - job: CopyData
    steps:
    - task: SqlAzureDacpacDeployment@1
      inputs:
        SqlInline: 'UPDATE Users SET Email_New = Email_Old WHERE Email_New IS NULL;'

- stage: Phase3_DeployApp
  dependsOn: Phase2_Migrate
  jobs:
  - job: DeployApplication
    steps:
    - task: AzureWebApp@1
      inputs:
        appName: 'myapp'
        package: '$(Pipeline.Workspace)/app.zip'

- stage: Phase4_Contract
  dependsOn: Phase3_DeployApp
  jobs:
  - job: RemoveOldColumn
    steps:
    - task: SqlAzureDacpacDeployment@1
      inputs:
        SqlInline: 'ALTER TABLE Users DROP COLUMN Email_Old;'
```

### 2. Blue-Green Database Deployment

**Maintain two database versions** for zero-downtime:

```
┌─────────────┐         ┌─────────────┐
│  Blue DB    │         │  Green DB   │
│  (Current)  │         │  (New)      │
│  Schema V1  │         │  Schema V2  │
└─────────────┘         └─────────────┘
      ↑                       ↑
      │                       │
┌─────┴─────┐           ┌─────┴─────┐
│  App V1   │           │  App V2   │
│ (Traffic) │           │ (Testing) │
└───────────┘           └───────────┘

After validation, switch traffic to Green:
                        ┌─────────────┐
                        │  Green DB   │
                        │  (Current)  │
                        │  Schema V2  │
                        └─────────────┘
                              ↑
                              │
                        ┌─────┴─────┐
                        │  App V2   │
                        │ (Traffic) │
                        └───────────┘
```

**Implementation**:

```yaml
- task: SqlAzureDacpacDeployment@1
  displayName: 'Deploy to Green Database'
  inputs:
    ServerName: 'myserver.database.windows.net'
    DatabaseName: 'MyDatabase_Green'  # Deploy to green
    DacpacFile: '$(Pipeline.Workspace)/database/MyDatabase.dacpac'

- task: AzureWebApp@1
  displayName: 'Deploy App to Green Slot'
  inputs:
    appName: 'myapp'
    deployToSlotOrASE: true
    slotName: 'green'
    package: '$(Pipeline.Workspace)/app.zip'

# After validation, swap slots
- task: AzureAppServiceManage@0
  displayName: 'Swap to Production'
  inputs:
    action: 'Swap Slots'
    sourceSlot: 'green'
```

### 3. Shadow Database Testing

**Test migrations on a copy before production**:

```yaml
- task: SqlAzureDacpacDeployment@1
  displayName: 'Restore Production Backup to Shadow'
  inputs:
    ServerName: 'myserver.database.windows.net'
    DatabaseName: 'MyDatabase_Shadow'
    deployType: 'SqlTask'
    SqlFile: '$(Pipeline.Workspace)/scripts/restore-backup.sql'

- task: SqlAzureDacpacDeployment@1
  displayName: 'Test Migration on Shadow Database'
  inputs:
    DatabaseName: 'MyDatabase_Shadow'
    DacpacFile: '$(Pipeline.Workspace)/database/MyDatabase.dacpac'
    AdditionalArguments: '/p:BlockOnPossibleDataLoss=true'

- pwsh: |
    # Run validation queries
    $results = Invoke-Sqlcmd -Query "SELECT COUNT(*) FROM Users" -ServerInstance "myserver.database.windows.net" -Database "MyDatabase_Shadow"
    if ($results.Column1 -eq 0) {
      Write-Error "Data validation failed!"
      exit 1
    }
  displayName: 'Validate Shadow Database'

- task: SqlAzureDacpacDeployment@1
  displayName: 'Deploy to Production (validated)'
  inputs:
    DatabaseName: 'MyDatabase'
    DacpacFile: '$(Pipeline.Workspace)/database/MyDatabase.dacpac'
```

## Database Deployment Best Practices

### 1. Use Idempotent Scripts

**Scripts should be safe to run multiple times**:

```sql
-- ❌ Bad: Fails if column exists
ALTER TABLE Users ADD Email NVARCHAR(255);

-- ✅ Good: Check before adding
IF NOT EXISTS (SELECT * FROM sys.columns 
               WHERE object_id = OBJECT_ID('Users') 
               AND name = 'Email')
BEGIN
    ALTER TABLE Users ADD Email NVARCHAR(255);
END

-- ✅ Good: Drop and recreate (for views, stored procs)
DROP VIEW IF EXISTS vw_ActiveUsers;
GO
CREATE VIEW vw_ActiveUsers AS
SELECT * FROM Users WHERE IsActive = 1;

-- ✅ Good: Merge for data
MERGE INTO Configuration AS target
USING (SELECT 'AppVersion', '1.0.0') AS source (KeyName, Value)
ON target.KeyName = source.KeyName
WHEN MATCHED THEN
    UPDATE SET Value = source.Value
WHEN NOT MATCHED THEN
    INSERT (KeyName, Value) VALUES (source.KeyName, source.Value);
```

### 2. Use Transactions

```sql
BEGIN TRANSACTION;

BEGIN TRY
    -- All changes in one transaction
    ALTER TABLE Users ADD NewColumn NVARCHAR(255);
    
    UPDATE Users SET NewColumn = 'DefaultValue';
    
    ALTER TABLE Users ALTER COLUMN NewColumn NVARCHAR(255) NOT NULL;
    
    COMMIT TRANSACTION;
    PRINT 'Migration successful';
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT 'Migration failed: ' + ERROR_MESSAGE();
    THROW;
END CATCH
```

### 3. Version Database Schema

```sql
-- Create schema version table
CREATE TABLE SchemaVersion (
    Version NVARCHAR(50) PRIMARY KEY,
    AppliedAt DATETIME2 DEFAULT GETUTCDATE(),
    AppliedBy NVARCHAR(255),
    Description NVARCHAR(500)
);

-- At end of migration script
INSERT INTO SchemaVersion (Version, AppliedBy, Description)
VALUES ('1.2.3', '$(Build.RequestedFor)', 'Added Email column to Users table');
```

**Check version before deployment**:

```yaml
- task: SqlAzureDacpacDeployment@1
  displayName: 'Check Schema Version'
  inputs:
    deployType: 'InlineSqlTask'
    SqlInline: |
      DECLARE @CurrentVersion NVARCHAR(50) = (SELECT TOP 1 Version FROM SchemaVersion ORDER BY AppliedAt DESC);
      DECLARE @TargetVersion NVARCHAR(50) = '$(Build.BuildNumber)';
      
      IF @CurrentVersion >= @TargetVersion
      BEGIN
        RAISERROR('Target version %s is not newer than current version %s', 16, 1, @TargetVersion, @CurrentVersion);
      END
```

### 4. Backup Before Deployment

```yaml
- task: SqlAzureDacpacDeployment@1
  displayName: 'Backup Database Before Deployment'
  inputs:
    deployType: 'SqlTask'
    SqlFile: '$(Pipeline.Workspace)/scripts/backup.sql'

# backup.sql content:
# BACKUP DATABASE [MyDatabase]
# TO URL = 'https://mystorageaccount.blob.core.windows.net/backups/MyDatabase_$(Build.BuildId).bak'
# WITH CREDENTIAL = 'AzureStorageCredential', COMPRESSION;
```

### 5. Deploy Database Before Application

```yaml
stages:
- stage: DeployDatabase
  jobs:
  - job: DatabaseDeployment
    steps:
    - task: SqlAzureDacpacDeployment@1
      inputs:
        DatabaseName: 'MyDatabase'
        DacpacFile: '$(Pipeline.Workspace)/database/MyDatabase.dacpac'

- stage: DeployApplication
  dependsOn: DeployDatabase  # Wait for database
  condition: succeeded()
  jobs:
  - job: AppDeployment
    steps:
    - task: AzureWebApp@1
      inputs:
        appName: 'myapp'
        package: '$(Pipeline.Workspace)/app.zip'
```

## Entity Framework Core Migrations

**Deploy EF Core migrations** in CI/CD:

```yaml
- task: DotNetCoreCLI@2
  displayName: 'Apply EF Core Migrations'
  inputs:
    command: 'custom'
    custom: 'ef'
    arguments: 'database update --connection "$(ConnectionString)" --project MyApp.Data --startup-project MyApp.Web'

# Alternative: Generate SQL script and apply
- task: DotNetCoreCLI@2
  displayName: 'Generate Migration Script'
  inputs:
    command: 'custom'
    custom: 'ef'
    arguments: 'migrations script --output $(Build.ArtifactStagingDirectory)/migration.sql --idempotent --project MyApp.Data'

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)/migration.sql'
    artifactName: 'migrations'

# In release pipeline
- task: SqlAzureDacpacDeployment@1
  displayName: 'Apply EF Core Migration Script'
  inputs:
    deployType: 'SqlTask'
    SqlFile: '$(Pipeline.Workspace)/migrations/migration.sql'
```

## Database Deployment Pipeline Example

**Complete database deployment workflow**:

```yaml
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - database/*

variables:
  - group: DatabaseSecrets  # Contains connection strings

stages:
- stage: Build
  jobs:
  - job: BuildDACPAC
    pool:
      vmImage: 'windows-latest'
    steps:
    - task: VSBuild@1
      displayName: 'Build Database Project'
      inputs:
        solution: 'database/MyDatabase.sqlproj'
        msbuildArgs: '/p:OutDir=$(Build.ArtifactStagingDirectory)'
    
    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'database'

- stage: DeployDev
  dependsOn: Build
  jobs:
  - deployment: DeployToDevDB
    environment: Dev
    strategy:
      runOnce:
        deploy:
          steps:
          - task: SqlAzureDacpacDeployment@1
            displayName: 'Deploy to Dev Database'
            inputs:
              azureSubscription: 'AzureConnection'
              ServerName: '$(DevServerName)'
              DatabaseName: '$(DevDatabaseName)'
              SqlUsername: '$(DevUsername)'
              SqlPassword: '$(DevPassword)'
              DacpacFile: '$(Pipeline.Workspace)/database/MyDatabase.dacpac'
              AdditionalArguments: '/p:BlockOnPossibleDataLoss=false'

- stage: DeployQA
  dependsOn: DeployDev
  jobs:
  - deployment: DeployToQADB
    environment: QA
    strategy:
      runOnce:
        deploy:
          steps:
          - task: SqlAzureDacpacDeployment@1
            displayName: 'Test Migration on QA Shadow'
            inputs:
              ServerName: '$(QAServerName)'
              DatabaseName: '$(QADatabaseName)_Shadow'
              DacpacFile: '$(Pipeline.Workspace)/database/MyDatabase.dacpac'
              AdditionalArguments: '/p:BlockOnPossibleDataLoss=true'
          
          - pwsh: |
              # Validate schema
              # Run integration tests
              Write-Host "Validating QA Shadow database..."
            displayName: 'Validate Migration'
          
          - task: SqlAzureDacpacDeployment@1
            displayName: 'Deploy to QA Database'
            inputs:
              ServerName: '$(QAServerName)'
              DatabaseName: '$(QADatabaseName)'
              DacpacFile: '$(Pipeline.Workspace)/database/MyDatabase.dacpac'

- stage: DeployProd
  dependsOn: DeployQA
  jobs:
  - deployment: DeployToProdDB
    environment: Production
    strategy:
      runOnce:
        deploy:
          steps:
          - task: SqlAzureDacpacDeployment@1
            displayName: 'Backup Production Database'
            inputs:
              deployType: 'SqlTask'
              SqlFile: '$(Pipeline.Workspace)/scripts/backup.sql'
          
          - task: SqlAzureDacpacDeployment@1
            displayName: 'Deploy to Production Database'
            inputs:
              ServerName: '$(ProdServerName)'
              DatabaseName: '$(ProdDatabaseName)'
              DacpacFile: '$(Pipeline.Workspace)/database/MyDatabase.dacpac'
              AdditionalArguments: '/p:BlockOnPossibleDataLoss=true /p:BackupDatabaseBeforeChanges=true'
          
          - task: SqlAzureDacpacDeployment@1
            displayName: 'Record Deployment'
            inputs:
              deployType: 'InlineSqlTask'
              SqlInline: |
                INSERT INTO DeploymentHistory (Version, DeployedAt, DeployedBy)
                VALUES ('$(Build.BuildNumber)', GETUTCDATE(), '$(Build.RequestedFor)');
```

## Critical Notes

🎯 **DACPAC vs SQL Scripts**: Use DACPAC for schema deployments (automatic comparison), SQL scripts for data migrations and hotfixes.

💡 **Expand-Contract Pattern**: Safest strategy for zero-downtime schema changes—add new schema, migrate data, switch application, remove old schema.

⚠️ **Backup First**: Always backup production database before deployment—`/p:BackupDatabaseBeforeChanges=true` or manual backup task.

📊 **Test on Shadow**: Restore production backup to shadow database, test migration there first, then deploy to production.

🔄 **Idempotent Scripts**: Write scripts that can run multiple times safely—use `IF NOT EXISTS`, `DROP IF EXISTS`, `MERGE` statements.

✨ **Deploy Database Before App**: Always deploy database changes before application—new app version may depend on new schema.

## Quick Reference

### DACPAC Deployment

```yaml
- task: SqlAzureDacpacDeployment@1
  inputs:
    azureSubscription: 'AzureConnection'
    ServerName: 'myserver.database.windows.net'
    DatabaseName: 'MyDatabase'
    DacpacFile: '$(Pipeline.Workspace)/database/MyDatabase.dacpac'
    AdditionalArguments: '/p:BlockOnPossibleDataLoss=true'
```

### SQL Script Execution

```yaml
- task: SqlAzureDacpacDeployment@1
  inputs:
    deployType: 'SqlTask'
    SqlFile: '$(Pipeline.Workspace)/scripts/migration.sql'
```

### Inline SQL

```yaml
- task: SqlAzureDacpacDeployment@1
  inputs:
    deployType: 'InlineSqlTask'
    SqlInline: 'UPDATE Configuration SET Value = ''$(Build.BuildNumber)'''
```

[Learn More](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/sql-azure-dacpac-deployment)
