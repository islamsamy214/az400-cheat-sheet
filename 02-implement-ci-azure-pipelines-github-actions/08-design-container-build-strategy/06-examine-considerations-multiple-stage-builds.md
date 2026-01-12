# Examine Considerations for Multiple Stage Builds

Designing effective multi-stage builds requires understanding container design principles, image optimization strategies, and best practices for production deployments. This unit covers essential considerations for building maintainable, secure, and efficient containerized applications.

## Container Design Principles

### 1. Single Responsibility Principle

**Design containers with single-purpose applications**:

```
❌ Bad: Monolithic Container
┌───────────────────────────────────┐
│  Single Container                 │
│  ├── Web Server (Nginx)           │
│  ├── Application Server (Node.js) │
│  ├── Database (PostgreSQL)        │
│  └── Cache (Redis)                │
└───────────────────────────────────┘
Problems:
- Hard to scale individual components
- Tight coupling
- Complex updates
- Resource contention

✅ Good: Separate Containers
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  Nginx   │→ │ Node.js  │→ │PostgreSQL│  │  Redis   │
│Container │  │Container │  │Container │  │Container │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
Benefits:
- Independent scaling
- Loose coupling
- Easy updates
- Resource optimization
```

**Single Responsibility Examples**:

```dockerfile
# ✅ Good: Web server only
FROM nginx:alpine
COPY nginx.conf /etc/nginx/
COPY static/ /usr/share/nginx/html/
CMD ["nginx", "-g", "daemon off;"]

# ✅ Good: Application server only
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "server.js"]

# ❌ Bad: Multiple responsibilities
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y \
    nginx \
    nodejs \
    postgresql \
    redis-server
# Don't do this!
```

### 2. Service Separation

**Deploy application tiers in separate containers**:

```yaml
# docker-compose.yml - Proper service separation
version: '3.8'
services:
  frontend:
    image: myapp-frontend:1.0
    ports:
      - "80:80"
    depends_on:
      - backend
  
  backend:
    image: myapp-backend:1.0
    ports:
      - "3000:3000"
    depends_on:
      - database
      - cache
    environment:
      - DATABASE_URL=postgresql://db:5432/myapp
      - REDIS_URL=redis://cache:6379
  
  database:
    image: postgres:15-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=secret
  
  cache:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:
```

**Benefits of Service Separation**:

| Benefit | Description | Example |
|---------|-------------|---------|
| **Independent Scaling** | Scale services based on demand | Scale web tier to 10 instances, database to 2 |
| **Technology Diversity** | Use best tool for each job | Python API, Node.js frontend, Rust workers |
| **Fault Isolation** | Failures contained to single service | API crash doesn't affect database |
| **Easy Updates** | Update services independently | Update frontend without touching backend |
| **Resource Optimization** | Allocate resources per service | More CPU for API, more memory for database |

### 3. Horizontal Scaling

**Design containers for horizontal scaling**:

```
Vertical Scaling (Limited):
┌────────────────┐
│  Single        │
│  Container     │
│  (4 CPU,       │
│   16 GB RAM)   │
└────────────────┘
Limit: Single machine capacity

Horizontal Scaling (Unlimited):
┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
│ API  │  │ API  │  │ API  │  │ API  │  │ API  │
│ (1   │  │ (1   │  │ (1   │  │ (1   │  │ (1   │
│ CPU) │  │ CPU) │  │ CPU) │  │ CPU) │  │ CPU) │
└──────┘  └──────┘  └──────┘  └──────┘  └──────┘
Limit: Add more instances as needed
```

**Stateless Design for Scaling**:

```dockerfile
# ✅ Good: Stateless application
FROM node:18-alpine
WORKDIR /app
COPY . .
# No local state stored
# Session data in Redis
# Files in object storage (S3, Azure Blob)
CMD ["node", "server.js"]

# ❌ Bad: Stateful application
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN mkdir /app/uploads  # Local file storage
RUN mkdir /app/sessions # Local session storage
# Can't scale horizontally!
CMD ["node", "server.js"]
```

**Kubernetes Deployment (Horizontal Scaling)**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-api
spec:
  replicas: 5  # 5 identical containers
  selector:
    matchLabels:
      app: web-api
  template:
    metadata:
      labels:
        app: web-api
    spec:
      containers:
      - name: api
        image: myapi:1.0
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 1Gi
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## Image Optimization

### 1. Minimal Dependencies

**Include only verified required packages**:

```dockerfile
# ❌ Bad: Install everything "just in case"
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    build-essential \
    git \
    vim \
    curl \
    wget \
    netcat \
    telnet \
    htop \
    screen \
    tmux
# Result: 800 MB+ image

# ✅ Good: Install only what's needed
FROM python:3.11-alpine
RUN apk add --no-cache \
    libpq  # Only PostgreSQL client library
# Result: 50 MB image
```

**Dependency Audit Checklist**:

| Question | Action |
|----------|--------|
| Is this package required at runtime? | Keep only runtime dependencies |
| Can I use a smaller alternative? | Use `curl` instead of `wget + curl` |
| Is this a build-only dependency? | Use multi-stage builds to exclude from final image |
| Does base image already include this? | Check before installing |

### 2. Lean Base Images

**Choose minimal base images**:

```dockerfile
# ❌ Bloated: Full Ubuntu (500+ MB)
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y python3
# Size: ~550 MB

# ✅ Better: Ubuntu slim (80 MB)
FROM ubuntu:22.04-slim
RUN apt-get update && apt-get install -y python3
# Size: ~100 MB

# ✅ Best: Alpine Linux (5 MB)
FROM python:3.11-alpine
# Size: ~50 MB

# ✅ Extreme: Distroless or scratch (< 5 MB)
FROM gcr.io/distroless/python3
# Size: ~30 MB
```

**Base Image Comparison**:

| Base Image | Size | Use Case | Pros | Cons |
|------------|------|----------|------|------|
| **Full (ubuntu:22.04)** | 500+ MB | Development, legacy apps | All tools included | Very large |
| **Slim (ubuntu:22.04-slim)** | 100 MB | General purpose | Smaller, common tools | Still relatively large |
| **Alpine** | 5-50 MB | Production | Minimal, fast | Some compatibility issues |
| **Distroless** | 5-30 MB | Secure production | No shell, minimal attack surface | Harder to debug |
| **Scratch** | 0 MB | Static binaries only | Smallest possible | Requires static compilation |

### 3. Layer Optimization

**Minimize layers and optimize caching**:

```dockerfile
# ❌ Bad: Multiple layers, poor caching
FROM node:18-alpine
COPY . .                          # Copies everything (invalidates cache often)
RUN npm install                   # Reinstalls on every code change
RUN npm run build
RUN npm prune --production
# Result: Slow builds, large image

# ✅ Good: Optimized layers, smart caching
FROM node:18-alpine AS build
WORKDIR /app

# Layer 1: Dependencies (changes rarely)
COPY package*.json ./
RUN npm ci

# Layer 2: Source code (changes frequently)
COPY . .
RUN npm run build

# Layer 3: Production dependencies only
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Layer 4: Built application
COPY --from=build /app/dist ./dist

CMD ["node", "dist/server.js"]
# Result: Fast builds, small image
```

**Layer Caching Strategy**:

```
Least frequently changed → Most frequently changed

1. Base image (FROM)
2. System packages (RUN apt-get)
3. Application dependencies (COPY package.json, RUN npm install)
4. Application code (COPY . .)
5. Build artifacts (RUN npm run build)
6. Runtime configuration (CMD, ENTRYPOINT)
```

## Base Image Selection

### Decision Matrix

```
Application Type → Base Image Recommendation

Python Application:
  Development:   python:3.11
  Production:    python:3.11-slim
  Minimal:       python:3.11-alpine
  Ultra-minimal: gcr.io/distroless/python3

Node.js Application:
  Development:   node:18
  Production:    node:18-slim
  Minimal:       node:18-alpine
  LTS:           node:lts-alpine

.NET Application:
  Build:         mcr.microsoft.com/dotnet/sdk:8.0
  Runtime:       mcr.microsoft.com/dotnet/aspnet:8.0
  Minimal:       mcr.microsoft.com/dotnet/aspnet:8.0-alpine

Go Application:
  Build:         golang:1.21
  Production:    alpine:latest (with binary)
  Minimal:       scratch (static binary only)

Java Application:
  Development:   openjdk:17
  Production:    openjdk:17-jre-slim
  Minimal:       gcr.io/distroless/java17
```

### Production Base Image Best Practices

```dockerfile
# ✅ Best Practice: Use specific, minimal base images

# Specify exact version (not "latest")
FROM node:18.16.0-alpine3.17

# Use official images from trusted registries
FROM mcr.microsoft.com/dotnet/aspnet:8.0

# Use minimal variants
FROM python:3.11-slim  # Not python:3.11 (full)

# Use LTS versions for stability
FROM node:lts-alpine

# Consider security updates
FROM ubuntu:22.04  # Gets security patches
```

## Data Management

### 1. External Storage

**Use Docker volumes or bind mounts for persistent data**:

```dockerfile
# ❌ Bad: Store data inside container
FROM postgres:15
# Data stored in container filesystem
# Lost when container is removed!

# ✅ Good: Use volumes for persistent data
FROM postgres:15
VOLUME /var/lib/postgresql/data
# Data persists independently of container lifecycle
```

**Volume Usage Examples**:

```bash
# Named volume (Docker-managed)
docker run -d \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15

# Bind mount (host directory)
docker run -d \
  -v /host/data:/container/data \
  myapp:1.0

# Anonymous volume
docker run -d \
  -v /container/data \
  myapp:1.0
```

**Docker Compose with Volumes**:

```yaml
version: '3.8'
services:
  database:
    image: postgres:15
    volumes:
      - postgres-data:/var/lib/postgresql/data  # Named volume
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # Bind mount
    environment:
      POSTGRES_PASSWORD: secret

  application:
    image: myapp:1.0
    volumes:
      - app-uploads:/app/uploads  # Persistent uploads
      - ./config.yml:/app/config.yml:ro  # Read-only config

volumes:
  postgres-data:  # Docker-managed volume
  app-uploads:    # Docker-managed volume
```

### 2. Stateless Design

**Design containers as immutable and stateless**:

```
Stateful (Bad):
┌─────────────────────┐
│  Container          │
│  ├── Application    │
│  ├── Session data   │ ← Stored locally (problem!)
│  ├── Uploaded files │ ← Stored locally (problem!)
│  └── User data      │ ← Stored locally (problem!)
└─────────────────────┘
Problems:
- Can't scale horizontally
- Data lost when container restarts
- Can't do rolling updates

Stateless (Good):
┌─────────────────────┐     ┌─────────────┐
│  Container          │────→│   Redis     │
│  ├── Application    │     │  (Sessions) │
│  └── No local state │     └─────────────┘
└─────────────────────┘
         │                   ┌─────────────┐
         └──────────────────→│  S3/Blob    │
                             │   (Files)   │
                             └─────────────┘
         │                   ┌─────────────┐
         └──────────────────→│  PostgreSQL │
                             │  (User Data)│
                             └─────────────┘
Benefits:
- Horizontal scaling
- Zero-downtime updates
- Fault tolerance
```

**Stateless Application Example**:

```javascript
// ❌ Bad: Store session in memory
const sessions = {};  // Lost when container restarts

app.post('/login', (req, res) => {
  sessions[req.body.userId] = { ...data };
});

// ✅ Good: Store session in Redis
const redis = require('redis').createClient();

app.post('/login', async (req, res) => {
  await redis.set(`session:${req.body.userId}`, JSON.stringify(data));
});
```

**External Storage Patterns**:

| Data Type | Storage Solution | Example |
|-----------|------------------|---------|
| **Sessions** | Redis, Memcached | User login sessions |
| **Files** | S3, Azure Blob Storage, NFS | Uploaded images, documents |
| **Databases** | External database service | PostgreSQL, MongoDB |
| **Logs** | Centralized logging (ELK, Splunk) | Application logs |
| **Metrics** | Prometheus, CloudWatch | Performance metrics |
| **Configuration** | ConfigMaps, etcd, Consul | Application settings |

## Multi-Stage Build Considerations

### Production-Ready Checklist

```dockerfile
# ✅ Complete multi-stage build template
FROM node:18 AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
# Save production dependencies

FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci  # All dependencies
COPY . .
RUN npm run build
RUN npm run test  # Tests must pass!

FROM node:18-alpine AS final
WORKDIR /app

# Non-root user
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

# Copy production dependencies
COPY --from=dependencies /app/node_modules ./node_modules

# Copy built application
COPY --from=build /app/dist ./dist
COPY package*.json ./

# Set ownership
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
  CMD node healthcheck.js || exit 1

# Run application
CMD ["node", "dist/server.js"]
```

## Critical Notes

🎯 **Single Responsibility**: One process per container—use orchestration (Docker Compose, Kubernetes) for multi-service applications.

💡 **Service Separation**: Deploy application tiers separately—enables independent scaling and updates.

⚠️ **Minimal Dependencies**: Include only runtime requirements in final image—use multi-stage builds to exclude build tools.

📊 **Stateless Design**: Never store state in containers—use external storage (databases, object storage, caches).

🔄 **Alpine Images**: Use Alpine-based images for production—dramatically smaller (5 MB vs 500 MB base).

✨ **Non-Root User**: Always run containers as non-root user—improves security and follows least-privilege principle.

## Quick Reference

### Base Image Selection

```dockerfile
# Development (include tools)
FROM python:3.11

# Production (minimal)
FROM python:3.11-slim

# Ultra-minimal (security focus)
FROM python:3.11-alpine

# Maximum security
FROM gcr.io/distroless/python3
```

### Stateless Pattern

```yaml
# Application (stateless)
services:
  app:
    image: myapp:1.0
    deploy:
      replicas: 10  # Can scale!
    depends_on:
      - redis
      - postgres

# State (external)
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

  postgres:
    image: postgres:15-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data
```

### Optimization Checklist

- [ ] Use minimal base image (Alpine or Distroless)
- [ ] Multi-stage build implemented
- [ ] Build tools excluded from final image
- [ ] Layers ordered for optimal caching
- [ ] Only production dependencies included
- [ ] Container runs as non-root user
- [ ] Persistent data stored externally
- [ ] Health check configured
- [ ] Resource limits defined

[Learn More](https://learn.microsoft.com/en-us/training/modules/design-container-build-strategy/6-examine-considerations-multiple-stage-builds)
