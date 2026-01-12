# Work with Docker Containers

Docker is the most popular container platform, providing tools to build, ship, and run containers. Understanding the Docker container lifecycle and essential commands is fundamental for effective container management.

## Container Lifecycle Overview

The Docker workflow follows a clear pattern from building images to running containers:

```
┌─────────────────────────────────────────────────────────────┐
│                 Docker Container Lifecycle                  │
└─────────────────────────────────────────────────────────────┘

1. BUILD                2. PUSH              3. PULL
┌──────────┐           ┌──────────┐         ┌──────────┐
│Dockerfile│──build──→ │  Image   │──push─→ │ Registry │
└──────────┘           └──────────┘         └──────────┘
                             ↑                     │
                             └──────pull───────────┘
                                    ↓
4. RUN                     ┌──────────────┐
┌──────────────┐          │    Image     │
│   Container  │←──run────│  (Local)     │
│   (Running)  │          └──────────────┘
└──────────────┘
       │
       ↓
5. STOP / REMOVE
┌──────────────┐
│   Stopped    │──remove──→ [Deleted]
│  Container   │
└──────────────┘
```

### Lifecycle Phases Explained

| Phase | Command | Purpose | Result |
|-------|---------|---------|--------|
| **BUILD** | `docker build` | Create image from Dockerfile | Image stored locally |
| **PUSH** | `docker push` | Upload image to registry | Image available remotely |
| **PULL** | `docker pull` | Download image from registry | Image stored locally |
| **RUN** | `docker run` | Create and start container from image | Running container |
| **STOP** | `docker stop` | Gracefully stop running container | Stopped container |
| **REMOVE** | `docker rm` | Delete stopped container | Container deleted |

## Core Docker Operations

### 1. Docker Build - Creating Images

**Purpose**: Transform Dockerfile into executable container image

**Basic Syntax**:
```bash
docker build [OPTIONS] PATH

# Common pattern
docker build -t IMAGE_NAME:TAG .
```

**Essential Build Examples**:

```bash
# Build with tag
docker build -t myapp:1.0 .

# Build with custom Dockerfile name
docker build -t myapp:1.0 -f Dockerfile.prod .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t myapp:1.0 .

# Build without cache (force rebuild)
docker build --no-cache -t myapp:1.0 .

# Build and show progress
docker build -t myapp:1.0 --progress=plain .
```

**Build Options**:

| Option | Purpose | Example |
|--------|---------|---------|
| `-t, --tag` | Name and tag image | `-t myapp:1.0` |
| `-f, --file` | Specify Dockerfile | `-f Dockerfile.dev` |
| `--build-arg` | Set build-time variables | `--build-arg ENV=prod` |
| `--no-cache` | Build without cache | `--no-cache` |
| `--target` | Build specific stage (multi-stage) | `--target production` |
| `--platform` | Build for specific platform | `--platform linux/amd64` |

**Build Process**:
```
Step 1: FROM ubuntu:22.04
 ---> Using cached layer (if available)
Step 2: RUN apt-get update
 ---> Running in temporary container
 ---> Layer created and cached
Step 3: COPY app.py /app/
 ---> Layer created
Step 4: CMD ["python", "/app/app.py"]
 ---> Final layer created
Successfully built abc123def456
Successfully tagged myapp:1.0
```

### 2. Docker Pull - Downloading Images

**Purpose**: Download images from container registries

**Basic Syntax**:
```bash
docker pull [OPTIONS] IMAGE_NAME[:TAG]
```

**Common Pull Examples**:

```bash
# Pull latest tag (default)
docker pull nginx
# Equivalent to: docker pull nginx:latest

# Pull specific tag
docker pull nginx:1.25-alpine

# Pull from specific registry
docker pull mcr.microsoft.com/dotnet/aspnet:8.0

# Pull all tags of an image
docker pull --all-tags ubuntu

# Pull from private Azure Container Registry
docker pull myregistry.azurecr.io/myapp:1.0
```

**Registry URLs**:

| Registry | URL Pattern | Example |
|----------|-------------|---------|
| **Docker Hub** | `docker.io/IMAGE` | `nginx:latest` |
| **Azure Container Registry** | `REGISTRY.azurecr.io/IMAGE` | `myacr.azurecr.io/app:1.0` |
| **GitHub Container Registry** | `ghcr.io/OWNER/IMAGE` | `ghcr.io/myorg/app:1.0` |
| **Amazon ECR** | `ACCOUNT.dkr.ecr.REGION.amazonaws.com/IMAGE` | `123456.dkr.ecr.us-east-1.amazonaws.com/app:1.0` |
| **Google Container Registry** | `gcr.io/PROJECT/IMAGE` | `gcr.io/myproject/app:1.0` |

**Pull Process**:
```
$ docker pull nginx:alpine

alpine: Pulling from library/nginx
96526aa774ef: Downloading  [=====>     ]  5.2MB/15.6MB
0e1c8e2e85e5: Download complete
f28a5e4a68f8: Download complete
2a7df4f6ed45: Waiting
9c6d18d37b28: Waiting

alpine: Pull complete
Digest: sha256:abc123...
Status: Downloaded newer image for nginx:alpine
docker.io/library/nginx:alpine
```

### 3. Docker Run - Starting Containers

**Purpose**: Create and start container from an image

**Basic Syntax**:
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARGS]
```

**Essential Run Examples**:

```bash
# Simple run (foreground)
docker run nginx

# Run in detached mode (background)
docker run -d nginx

# Run with name
docker run --name my-nginx nginx

# Run with port mapping
docker run -p 8080:80 nginx
# Host port 8080 → Container port 80

# Run with environment variables
docker run -e APP_ENV=production -e API_KEY=secret myapp

# Run with volume mount
docker run -v /host/data:/container/data myapp

# Run with resource limits
docker run --memory="512m" --cpus="1.0" myapp

# Run interactively with terminal
docker run -it ubuntu bash
```

**Run Options Reference**:

| Option | Purpose | Example |
|--------|---------|---------|
| `-d, --detach` | Run in background | `-d` |
| `-p, --publish` | Port mapping | `-p 8080:80` |
| `--name` | Container name | `--name web-server` |
| `-e, --env` | Environment variable | `-e NODE_ENV=production` |
| `-v, --volume` | Mount volume | `-v /data:/app/data` |
| `-it` | Interactive terminal | `-it` (combination) |
| `--rm` | Remove container on exit | `--rm` |
| `--memory` | Memory limit | `--memory="512m"` |
| `--cpus` | CPU limit | `--cpus="2.0"` |
| `--network` | Network mode | `--network host` |
| `--restart` | Restart policy | `--restart always` |

**Auto-Pull Behavior**:

```bash
# If image doesn't exist locally, Docker automatically pulls it
docker run nginx:alpine

# Output shows automatic pull:
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
...
alpine: Pull complete
Starting container...
```

**No need for explicit pull in most cases**:
```bash
# ❌ Unnecessary (manual pull)
docker pull nginx
docker run nginx

# ✅ Simpler (auto-pull)
docker run nginx
```

## Common Docker Container Commands

### Container Management

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List container IDs only
docker ps -q

# View container details
docker inspect <container-id>

# View container logs
docker logs <container-id>

# Follow logs in real-time
docker logs -f <container-id>

# View last 100 log lines
docker logs --tail 100 <container-id>
```

### Starting and Stopping Containers

```bash
# Stop container gracefully (SIGTERM, then SIGKILL after 10s)
docker stop <container-id>

# Stop immediately (SIGKILL)
docker kill <container-id>

# Start stopped container
docker start <container-id>

# Restart container
docker restart <container-id>

# Pause container (freeze processes)
docker pause <container-id>

# Unpause container
docker unpause <container-id>
```

### Executing Commands in Containers

```bash
# Execute command in running container
docker exec <container-id> COMMAND

# Interactive shell in running container
docker exec -it <container-id> bash

# Run command as specific user
docker exec -u root <container-id> apt-get update

# Execute with environment variable
docker exec -e VAR=value <container-id> printenv VAR
```

### Container Cleanup

```bash
# Remove stopped container
docker rm <container-id>

# Force remove running container
docker rm -f <container-id>

# Remove all stopped containers
docker container prune

# Remove container when it exits (--rm flag)
docker run --rm nginx
```

## Image Management Commands

```bash
# List local images
docker images

# Remove image
docker rmi <image-id>

# Remove unused images
docker image prune

# Remove all unused images (not just dangling)
docker image prune -a

# View image history (layers)
docker history <image-name>

# Tag image
docker tag myapp:1.0 myapp:latest

# Save image to tar file
docker save -o myapp.tar myapp:1.0

# Load image from tar file
docker load -i myapp.tar
```

## Working with Registries

### Authentication

```bash
# Login to Docker Hub
docker login

# Login to Azure Container Registry
docker login myregistry.azurecr.io

# Login with credentials
docker login -u username -p password registry.example.com

# Logout
docker logout
```

### Pushing Images

```bash
# Tag for registry
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0

# Push to registry
docker push myregistry.azurecr.io/myapp:1.0

# Push all tags
docker push --all-tags myregistry.azurecr.io/myapp
```

## Practical Workflow Examples

### Example 1: Complete Development Workflow

```bash
# 1. Build image from Dockerfile
docker build -t myapp:dev .

# 2. Run container with development settings
docker run -d \
  --name myapp-dev \
  -p 3000:3000 \
  -v $(pwd):/app \
  -e NODE_ENV=development \
  myapp:dev

# 3. View logs
docker logs -f myapp-dev

# 4. Execute command in container
docker exec -it myapp-dev npm test

# 5. Stop and remove
docker stop myapp-dev
docker rm myapp-dev
```

### Example 2: Production Deployment Workflow

```bash
# 1. Build production image
docker build -t myapp:1.0 -f Dockerfile.prod .

# 2. Tag for registry
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0

# 3. Push to registry
docker push myregistry.azurecr.io/myapp:1.0

# 4. On production server: pull image
docker pull myregistry.azurecr.io/myapp:1.0

# 5. Run production container
docker run -d \
  --name myapp-prod \
  --restart always \
  -p 80:80 \
  -e NODE_ENV=production \
  myregistry.azurecr.io/myapp:1.0
```

### Example 3: Database Container

```bash
# Run PostgreSQL container
docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=mydb \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15-alpine

# Connect to database
docker exec -it postgres-db psql -U postgres -d mydb

# View database logs
docker logs postgres-db

# Backup database
docker exec postgres-db pg_dump -U postgres mydb > backup.sql
```

## Container Networking

```bash
# List networks
docker network ls

# Create custom network
docker network create my-network

# Run container on custom network
docker run -d --network my-network --name web nginx

# Connect running container to network
docker network connect my-network <container-id>

# Inspect network
docker network inspect my-network
```

## Volume Management

```bash
# List volumes
docker volume ls

# Create volume
docker volume create my-data

# Run container with volume
docker run -v my-data:/app/data myapp

# Inspect volume
docker volume inspect my-data

# Remove volume
docker volume rm my-data

# Remove unused volumes
docker volume prune
```

## Critical Notes

🎯 **Auto-Pull**: `docker run` automatically pulls missing images—no need for explicit `docker pull` in most workflows.

💡 **Detached Mode**: Use `-d` flag for production containers so they run in background—foreground mode blocks terminal.

⚠️ **Container Naming**: Use `--name` to give containers meaningful names—easier than working with random IDs.

📊 **Resource Limits**: Always set memory/CPU limits in production—prevents containers from consuming all host resources.

🔄 **Restart Policies**: Use `--restart always` for production services—containers restart automatically after failures or reboots.

✨ **Image Tags**: Always use specific tags (`:1.0`) in production, not `:latest`—ensures predictable deployments.

## Quick Reference

### Essential Commands

```bash
# Build, Run, Stop, Remove (BRSR)
docker build -t myapp:1.0 .
docker run -d --name myapp -p 8080:80 myapp:1.0
docker stop myapp
docker rm myapp

# List and Clean Up
docker ps                    # Running containers
docker ps -a                 # All containers
docker images                # Local images
docker system prune -a       # Clean up everything
```

### Common Run Patterns

```bash
# Development (with volume mount, interactive)
docker run -it -v $(pwd):/app -p 3000:3000 myapp:dev

# Production (detached, restart policy, resource limits)
docker run -d --restart always --memory="512m" --cpus="1.0" -p 80:80 myapp:prod

# Database (named volume, environment variables)
docker run -d --name db -v db-data:/var/lib/data -e DB_PASSWORD=secret postgres:15

# Testing (remove on exit)
docker run --rm myapp:test npm test
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/design-container-build-strategy/3-work-docker-containers)
