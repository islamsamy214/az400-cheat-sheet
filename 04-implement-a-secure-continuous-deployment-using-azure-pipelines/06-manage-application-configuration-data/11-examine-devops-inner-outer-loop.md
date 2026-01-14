# Examine DevOps Inner and Outer Loop

**Duration**: 4 minutes

DevOps workflows separate into two complementary cycles: the **inner loop** (individual developer iteration) and the **outer loop** (team-wide CI/CD integration), each requiring distinct configuration management strategies.

---

## Overview

### Inner Loop (Developer Loop)
**Definition**: Rapid iteration cycle within a developer's local environment before committing code.

**Activities**: Code → Build → Test → Debug → Repeat

**Duration**: Minutes to hours

**Scope**: Individual developer

### Outer Loop (Team Loop)
**Definition**: Automated CI/CD pipeline executing after code commit, integrating changes across the team.

**Activities**: Commit → Build → Test → Deploy → Monitor

**Duration**: Minutes to days

**Scope**: Entire team and production environment

---

## Inner Loop Workflow

### Traditional Inner Loop

```
┌─────────────┐
│ Write Code  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│Local Build  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Run Tests   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    Debug    │
└──────┬──────┘
       │
       ▼
   ┌───────┐
   │ Fixed?│───No──┐
   └───┬───┘       │
       Yes         │
       │           │
       ▼           │
┌─────────────┐   │
│   Commit    │◄──┘
└─────────────┘
```

**Cycle Time**: 5-30 minutes per iteration

### Modern Inner Loop (Cloud-Native)

```
┌──────────────┐
│  Write Code  │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Live Reload  │ ← Hot reload (no full rebuild)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│Test in Docker│ ← Local container matches prod
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Debug in IDE │ ← Attach debugger to container
└──────┬───────┘
       │
       ▼
┌──────────────┐
│    Commit    │
└──────────────┘
```

**Cycle Time**: 30 seconds - 5 minutes (faster feedback)

---

## Outer Loop Workflow

### CI/CD Pipeline

```
┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐
│  Commit  │─────▶│  Build   │─────▶│   Test   │─────▶│  Deploy  │─────▶│ Monitor  │
└──────────┘      └──────────┘      └──────────┘      └──────────┘      └──────────┘
                       │                  │                  │                  │
                       │                  │                  │                  │
                  Compile Code      Unit Tests         Staging           Metrics
                  Static Analysis   Integration Tests  Production        Logs
                  Package           Security Scans     Canary Deploy     Alerts
```

**Cycle Time**: 10 minutes - 2 hours (depends on pipeline complexity)

### Multi-Stage Outer Loop

```yaml
# Azure Pipelines example
trigger:
  branches:
    include:
    - main

stages:
- stage: Build
  jobs:
  - job: CompileAndTest
    steps:
    - task: DotNetCoreCLI@2
      inputs:
        command: 'build'
    - task: DotNetCoreCLI@2
      inputs:
        command: 'test'

- stage: DeployStaging
  dependsOn: Build
  jobs:
  - deployment: StagingDeploy
    environment: 'Staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              appName: 'myapp-staging'

- stage: DeployProduction
  dependsOn: DeployStaging
  condition: succeeded()
  jobs:
  - deployment: ProductionDeploy
    environment: 'Production'  # Manual approval required
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              appName: 'myapp-prod'
```

---

## Configuration Management Strategies

### Inner Loop Configuration

#### Local Configuration Files

**appsettings.Development.json**:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=myapp_dev;User Id=dev;Password=dev;"
  },
  "ApiSettings": {
    "BaseUrl": "http://localhost:5000",
    "ApiKey": "local-dev-key"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  }
}
```

**Why Local Config**:
- ✅ Fast iteration (no network calls)
- ✅ Offline development (no cloud dependency)
- ✅ Simplified debugging (predictable values)

#### Docker Compose for Local Services

**docker-compose.yml**:
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "5000:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=db;Database=myapp;User Id=sa;Password=Dev@123
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
    ports:
      - "5432:5432"

  redis:
    image: redis:7
    ports:
      - "6379:6379"
```

**Benefits**:
- Environment parity (local matches production architecture)
- Isolated dependencies (no conflicts with other projects)
- Quick reset (destroy and recreate containers)

### Outer Loop Configuration

#### Azure App Configuration

**Pipeline Integration**:
```yaml
- task: AzureAppConfiguration@5
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    AppConfigurationEndpoint: 'https://myapp-config.azconfig.io'
    KeyFilter: '*'
    Label: 'production'

# Configuration loaded as pipeline variables
- script: |
    echo "Database: $(ConnectionStrings--DefaultConnection)"
    echo "API URL: $(ApiSettings--BaseUrl)"
```

#### Azure Key Vault

**Pipeline Integration**:
```yaml
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Production-ServiceConnection'
    KeyVaultName: 'prod-keyvault'
    SecretsFilter: '*'

# Secrets loaded as masked pipeline variables
- script: |
    echo "Deploying with database password: $(DatabasePassword)"  # Masked
```

---

## Inner vs Outer Loop Comparison

| Aspect | Inner Loop | Outer Loop |
|--------|-----------|----------|
| **Scope** | Individual developer | Entire team |
| **Environment** | Local machine | Cloud (CI/CD agents) |
| **Configuration** | Local files, environment variables | Azure App Configuration, Key Vault |
| **Secrets** | Dummy/local values | Production secrets |
| **Data** | Local database, mock data | Staging/production databases |
| **Speed Priority** | Fastest feedback (seconds) | Thorough validation (minutes) |
| **Testing** | Unit tests, limited integration | Full test suite, E2E tests |
| **Deployment** | No deployment (local run) | Automated deployment to environments |
| **Monitoring** | Console logs, debugger | Application Insights, Log Analytics |

---

## Configuration Patterns by Loop

### Inner Loop: User Secrets (Development)

**ASP.NET Core User Secrets**:
```bash
# Initialize user secrets
dotnet user-secrets init --project MyApp.csproj

# Set secrets (stored outside project directory)
dotnet user-secrets set "ApiSettings:ApiKey" "local-dev-key-123"
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=localhost;Database=myapp;..."

# List secrets
dotnet user-secrets list
```

**Storage Location**: 
- Windows: `%APPDATA%\Microsoft\UserSecrets\{guid}\secrets.json`
- Linux/macOS: `~/.microsoft/usersecrets/{guid}/secrets.json`

**Access in Code**:
```csharp
// Automatically loaded in Development environment
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }
    
    public IConfiguration Configuration { get; }
    
    public void ConfigureServices(IServiceCollection services)
    {
        var apiKey = Configuration["ApiSettings:ApiKey"];  // From user secrets
    }
}
```

### Inner Loop: Environment Variables

```bash
# Set environment variables (session-scoped)
export ASPNETCORE_ENVIRONMENT=Development
export ApiSettings__ApiKey=local-dev-key
export ConnectionStrings__DefaultConnection="Server=localhost;..."

# Run application
dotnet run
```

**Docker Compose**:
```yaml
services:
  app:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ApiSettings__ApiKey=local-dev-key
```

### Outer Loop: Azure App Configuration

**Multi-Environment Strategy**:
```bash
# Staging configuration
az appconfig kv set \
  --name myapp-config \
  --key "ApiSettings:BaseUrl" \
  --value "https://staging-api.myapp.com" \
  --label staging

# Production configuration
az appconfig kv set \
  --name myapp-config \
  --key "ApiSettings:BaseUrl" \
  --value "https://api.myapp.com" \
  --label production
```

**Pipeline Selection**:
```yaml
# Staging pipeline
- task: AzureAppConfiguration@5
  inputs:
    Label: 'staging'

# Production pipeline
- task: AzureAppConfiguration@5
  inputs:
    Label: 'production'
```

### Outer Loop: Key Vault References

**Configuration with Key Vault References**:
```bash
# Store secret in Key Vault
az keyvault secret set \
  --vault-name prod-keyvault \
  --name DatabasePassword \
  --value "P@ssw0rd123!"

# Reference in App Configuration
az appconfig kv set \
  --name myapp-config \
  --key "ConnectionStrings:DefaultConnection" \
  --value '{"uri":"https://prod-keyvault.vault.azure.net/secrets/DatabasePassword"}' \
  --content-type "application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8"
```

---

## Transitioning Between Loops

### Developer Pushes Code (Inner → Outer)

```
Inner Loop                                   Outer Loop
┌─────────────────┐                         ┌─────────────────┐
│ Local Dev       │                         │ CI/CD Pipeline  │
│                 │                         │                 │
│ 1. Code change  │                         │                 │
│ 2. Local test   │                         │                 │
│ 3. Commit code  │────git push───────────▶│ 4. Build        │
└─────────────────┘                         │ 5. Test         │
                                             │ 6. Deploy       │
                                             └─────────────────┘
```

**Configuration Transition**:
```csharp
// Automatically switches based on environment
public class Startup
{
    public Startup(IConfiguration configuration, IWebHostEnvironment env)
    {
        var builder = new ConfigurationBuilder()
            .SetBasePath(env.ContentRootPath)
            .AddJsonFile("appsettings.json", optional: false)
            .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true);  // Inner loop
        
        if (env.IsDevelopment())
        {
            builder.AddUserSecrets<Startup>();  // Inner loop only
        }
        else
        {
            // Outer loop: Connect to Azure App Configuration
            builder.AddAzureAppConfiguration(options =>
            {
                options.Connect(Environment.GetEnvironmentVariable("APP_CONFIG_CONNECTION"))
                       .ConfigureKeyVault(kv =>
                       {
                           kv.SetCredential(new DefaultAzureCredential());
                       });
            });
        }
        
        builder.AddEnvironmentVariables();
        Configuration = builder.Build();
    }
}
```

---

## Best Practices

### Inner Loop Optimization

1. **Fast Feedback**: Use hot reload and incremental builds
```bash
# .NET hot reload
dotnet watch run  # Auto-restart on code changes

# Node.js hot reload
npm run dev  # Nodemon auto-restart
```

2. **Local Service Dependencies**: Use Docker Compose
```yaml
# One command to start all services
docker-compose up -d
```

3. **Dummy Secrets**: Never use production secrets locally
```json
{
  "ApiKey": "local-dev-key",  // ✅ Fake key for local testing
  "DatabasePassword": "dev"   // ✅ Local database password
}
```

### Outer Loop Optimization

1. **Centralized Configuration**: Azure App Configuration
```csharp
// Configuration automatically refreshes every 30 seconds
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .ConfigureRefresh(refresh =>
           {
               refresh.Register("Sentinel", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromSeconds(30));
           });
});
```

2. **Secret Management**: Azure Key Vault
```yaml
# All secrets retrieved at pipeline runtime (never committed)
- task: AzureKeyVault@2
  inputs:
    SecretsFilter: '*'
```

3. **Environment Parity**: Consistent configuration structure
```bash
# Same key structure across environments (different values)
# Staging
ApiSettings:BaseUrl = https://staging-api.myapp.com

# Production
ApiSettings:BaseUrl = https://api.myapp.com
```

---

## Real-World Example

### Scenario: Microservices Development

**Inner Loop (Developer)**:
```bash
# Developer working on Order Service
cd order-service
docker-compose up -d  # Start dependencies: PostgreSQL, RabbitMQ, Redis

# appsettings.Development.json
{
  "ConnectionStrings": {
    "OrderDb": "Host=localhost;Database=orders;Username=dev;Password=dev"
  },
  "MessageQueue": {
    "Host": "localhost",
    "Username": "guest",
    "Password": "guest"
  }
}

# Code → Build → Test → Debug (30 seconds per iteration)
dotnet watch run
```

**Outer Loop (CI/CD)**:
```yaml
# Azure Pipelines
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - order-service/*

stages:
- stage: Build
  jobs:
  - job: BuildTest
    steps:
    - task: DotNetCoreCLI@2
      inputs:
        command: 'build'
        projects: 'order-service/**/*.csproj'
    
    - task: DotNetCoreCLI@2
      inputs:
        command: 'test'

- stage: DeployProduction
  jobs:
  - deployment: Deploy
    environment: 'Production'
    steps:
    # Retrieve production configuration
    - task: AzureAppConfiguration@5
      inputs:
        AppConfigurationEndpoint: 'https://myapp-config.azconfig.io'
        Label: 'production'
    
    # Retrieve production secrets
    - task: AzureKeyVault@2
      inputs:
        KeyVaultName: 'prod-keyvault'
        SecretsFilter: 'OrderDbPassword,RabbitMQPassword'
    
    # Deploy with production config
    - task: AzureWebApp@1
      inputs:
        appName: 'order-service-prod'
        appSettings: |
          -ConnectionStrings__OrderDb "Host=prod-db.postgres.database.azure.com;Database=orders;Username=admin;Password=$(OrderDbPassword)" 
          -MessageQueue__Host "prod-rabbitmq.servicebus.windows.net"
          -MessageQueue__Password "$(RabbitMQPassword)"
```

**Result**:
- Developer: Fast local iteration (30s cycles) with dummy data
- Production: Secure deployment (15min pipeline) with real secrets

---

## Key Takeaways

✅ **Inner loop**: Individual developer iteration (minutes), local configuration  
✅ **Outer loop**: Team-wide CI/CD (minutes to hours), centralized configuration  
✅ **Inner loop config**: Local files, user secrets, Docker Compose  
✅ **Outer loop config**: Azure App Configuration, Key Vault  
✅ **Optimization**: Fast feedback (inner), thorough validation (outer)  
✅ **Security**: Dummy secrets locally, real secrets in pipelines  
✅ **Environment parity**: Docker containers match production architecture  

---

**Next**: Learn comprehensive Azure Key Vault integration with Azure DevOps →

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-application-configuration-data/11-examine-devops-inner-outer-loop)
