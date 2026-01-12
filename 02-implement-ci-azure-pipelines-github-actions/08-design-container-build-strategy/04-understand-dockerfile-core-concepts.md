# Understand Dockerfile Core Concepts

Dockerfiles are text-based instruction files that define how to build container images. Mastering Dockerfile creation is essential for building efficient, secure, and maintainable containerized applications.

## What is a Dockerfile?

A **Dockerfile** is a script containing a series of instructions that Docker executes sequentially to create a layered container image. Each instruction creates a new layer in the image, enabling efficient caching and reuse.

**Key Characteristics**:
- ✅ Text file (usually named `Dockerfile`)
- ✅ Contains step-by-step build instructions
- ✅ Each instruction creates an image layer
- ✅ Executed from top to bottom
- ✅ Cached for faster subsequent builds

### Simple Dockerfile Example

```dockerfile
FROM ubuntu:22.04
LABEL maintainer="devops@example.com"
ADD appsetup /
RUN /bin/bash -c 'source $HOME/.bashrc; \
echo $HOME'
CMD ["echo", "Hello World from within the container"]
```

**Build and Run**:
```bash
# Build image from Dockerfile
docker build -t myapp:1.0 .

# Run container
docker run myapp:1.0
# Output: Hello World from within the container
```

## Key Dockerfile Instructions

### FROM - Base Image Selection

**Purpose**: Specifies the base image for the new container image

**Syntax**:
```dockerfile
FROM image:tag
FROM image:tag AS stage_name  # For multi-stage builds
```

**Examples**:
```dockerfile
# Use official Ubuntu image
FROM ubuntu:22.04

# Use official Python image
FROM python:3.11-slim

# Use official Node.js image
FROM node:18-alpine

# Use .NET runtime image
FROM mcr.microsoft.com/dotnet/aspnet:8.0

# Build from scratch (for base images)
FROM scratch
```

**Best Practices**:
```dockerfile
# ✅ Good: Use specific tags
FROM node:18.16-alpine3.17

# ❌ Bad: Use latest tag (unpredictable)
FROM node:latest

# ✅ Good: Use minimal base images (Alpine)
FROM python:3.11-alpine  # ~50 MB

# ❌ Bad: Use full OS images
FROM python:3.11  # ~900 MB
```

**Base Image Selection Guide**:

| Base Image | Size | Use Case |
|------------|------|----------|
| `alpine` | 5-10 MB | Minimal, production-ready |
| `slim` | 50-100 MB | Smaller than full, common libraries included |
| `full` (default) | 500+ MB | Development, all dependencies included |
| `scratch` | 0 MB | Statically compiled binaries only |

### LABEL - Image Metadata

**Purpose**: Adds metadata to images, including maintainer information

**Syntax**:
```dockerfile
LABEL key="value"
LABEL key1="value1" key2="value2"
```

**Examples**:
```dockerfile
# Maintainer information (replaces deprecated MAINTAINER)
LABEL maintainer="devops@example.com"

# Version information
LABEL version="1.0.0"
LABEL release-date="2024-01-15"

# Application metadata
LABEL app.name="Web API"
LABEL app.description="RESTful API for customer management"
LABEL app.vendor="Contoso Inc."

# Multiple labels in one instruction (efficient)
LABEL version="1.0.0" \
      maintainer="devops@example.com" \
      description="Production API"
```

**Viewing Labels**:
```bash
# Inspect image labels
docker inspect myapp:1.0 | jq '.[0].Config.Labels'

# Output:
{
  "maintainer": "devops@example.com",
  "version": "1.0.0",
  "description": "Production API"
}
```

### ADD - Copy Files with Advanced Features

**Purpose**: Copies files, directories, or URLs into the image filesystem

**Syntax**:
```dockerfile
ADD source destination
ADD --chown=user:group source destination
```

**Examples**:
```dockerfile
# Copy local file
ADD config.json /app/config.json

# Copy directory
ADD ./src /app/src

# Copy and extract tar archive (automatic)
ADD application.tar.gz /app/
# Automatically extracts to /app/

# Download from URL
ADD https://example.com/config.json /app/config.json

# Copy with ownership
ADD --chown=appuser:appgroup app.py /app/
```

**ADD vs COPY**:

| Feature | ADD | COPY |
|---------|-----|------|
| **Copy local files** | ✅ | ✅ |
| **Copy directories** | ✅ | ✅ |
| **URL download** | ✅ | ❌ |
| **Auto-extract archives** | ✅ | ❌ |
| **Transparency** | Less (magic extraction) | More (explicit) |
| **Recommended for** | Archives, URLs | Regular file copies |

**Best Practice**:
```dockerfile
# ✅ Use COPY for simple file copies (more explicit)
COPY app.py /app/

# ✅ Use ADD for archives (automatic extraction)
ADD dist.tar.gz /app/

# ❌ Avoid ADD for simple copies
ADD app.py /app/  # Use COPY instead
```

### RUN - Execute Build-Time Commands

**Purpose**: Executes commands during image build time

**Syntax**:
```dockerfile
RUN command                          # Shell form
RUN ["executable", "param1", "param2"]  # Exec form
```

**Shell Form** (Most Common):
```dockerfile
# Install packages
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    curl

# Create directories
RUN mkdir -p /app/data /app/logs

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Multi-line command with backslashes
RUN /bin/bash -c 'source $HOME/.bashrc; \
    echo $HOME; \
    pip install flask'
```

**Exec Form** (No Shell Processing):
```dockerfile
# Direct execution (no shell)
RUN ["apt-get", "update"]
RUN ["pip", "install", "flask"]

# With variables (won't work without shell)
RUN ["echo", "$HOME"]  # Outputs literal "$HOME"
```

**Common RUN Patterns**:

```dockerfile
# Package installation (Ubuntu/Debian)
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*  # Clean up

# Package installation (Alpine)
RUN apk add --no-cache \
    package1 \
    package2

# Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Node.js dependencies
RUN npm ci --only=production

# Create non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
```

**Optimization Tips**:
```dockerfile
# ❌ Bad: Multiple RUN commands (multiple layers)
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y curl
# Creates 3 layers

# ✅ Good: Single RUN command (one layer)
RUN apt-get update && \
    apt-get install -y python3 curl && \
    rm -rf /var/lib/apt/lists/*
# Creates 1 layer, smaller image
```

### CMD - Default Container Command

**Purpose**: Defines the default command executed when starting a container

**Syntax**:
```dockerfile
CMD ["executable", "param1", "param2"]     # Exec form (preferred)
CMD command param1 param2                   # Shell form
CMD ["param1", "param2"]                    # Parameters to ENTRYPOINT
```

**Key Characteristics**:
- ⚡ Executes at **container runtime** (not build time)
- ⚡ Can be overridden by `docker run` arguments
- ⚡ Only the **last CMD** in Dockerfile takes effect

**Examples**:
```dockerfile
# Python application
CMD ["python", "app.py"]

# Node.js application
CMD ["node", "server.js"]

# Shell script
CMD ["sh", "start.sh"]

# Shell form (with variable expansion)
CMD python /app/server.py --port=$PORT
```

**CMD vs RUN**:

| Instruction | Execution Time | Purpose | Example |
|-------------|----------------|---------|---------|
| **RUN** | Build time | Install dependencies, setup | `RUN apt-get install nginx` |
| **CMD** | Runtime | Start application | `CMD ["nginx", "-g", "daemon off;"]` |

**Overriding CMD**:
```dockerfile
# Dockerfile
CMD ["echo", "Hello"]

# Run with default CMD
docker run myimage
# Output: Hello

# Override CMD
docker run myimage echo "Goodbye"
# Output: Goodbye
```

## Complete Dockerfile Examples

### Example 1: Python Flask Application

```dockerfile
# Base image
FROM python:3.11-slim

# Metadata
LABEL maintainer="devops@example.com"
LABEL version="1.0.0"
LABEL description="Flask API"

# Set working directory
WORKDIR /app

# Copy dependency file
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 5000

# Set environment variable
ENV FLASK_APP=app.py

# Run application
CMD ["flask", "run", "--host=0.0.0.0"]
```

**Build and Run**:
```bash
docker build -t flask-app:1.0 .
docker run -d -p 5000:5000 flask-app:1.0
```

### Example 2: Node.js Application

```dockerfile
FROM node:18-alpine

LABEL maintainer="devops@example.com"

# Create app directory
WORKDIR /usr/src/app

# Copy package files
COPY package*.json ./

# Install production dependencies
RUN npm ci --only=production

# Copy application source
COPY . .

# Expose port
EXPOSE 3000

# Start application
CMD ["node", "server.js"]
```

### Example 3: Multi-Command Setup

```dockerfile
FROM ubuntu:22.04

# Install multiple packages
RUN apt-get update && apt-get install -y \
    curl \
    git \
    vim \
    && rm -rf /var/lib/apt/lists/*

# Create directories
RUN mkdir -p /app/data /app/logs

# Add application files
ADD application.tar.gz /app/

# Set permissions
RUN chown -R nobody:nogroup /app

# Set working directory
WORKDIR /app

# Run startup script
CMD ["bash", "startup.sh"]
```

## Dockerfile Best Practices

### 1. Order Instructions for Optimal Caching

```dockerfile
# ✅ Good: Stable layers first, changing layers last
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./          # Changes rarely
RUN npm ci                     # Cached if package.json unchanged
COPY . .                       # Changes frequently (last)
CMD ["node", "server.js"]

# ❌ Bad: Copying all files first
FROM node:18-alpine
COPY . .                       # Changes frequently (first)
RUN npm install                # Re-runs on every code change
CMD ["node", "server.js"]
```

### 2. Combine Commands to Reduce Layers

```dockerfile
# ❌ Bad: Multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean

# ✅ Good: Single layer
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Use .dockerignore File

```
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.vscode
*.md
```

### 4. Don't Run as Root

```dockerfile
# Create non-root user
RUN useradd -m -u 1000 appuser

# Change ownership
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Run application
CMD ["python", "app.py"]
```

### 5. Use Explicit Tags

```dockerfile
# ❌ Bad: Latest tag (unpredictable)
FROM node:latest

# ✅ Good: Specific version
FROM node:18.16-alpine3.17
```

## Layer Caching Explained

Each Dockerfile instruction creates a layer that Docker caches:

```dockerfile
FROM node:18-alpine           # Layer 1: Base image (cached)
WORKDIR /app                  # Layer 2: Set workdir (cached)
COPY package*.json ./         # Layer 3: Copy package files (cached if unchanged)
RUN npm install               # Layer 4: Install deps (cached if Layer 3 unchanged)
COPY . .                      # Layer 5: Copy source (changes frequently)
CMD ["node", "server.js"]     # Layer 6: Set command (cached)
```

**Cache Behavior**:
```
Build #1 (Fresh):
Layer 1: Downloaded
Layer 2: Executed
Layer 3: Executed
Layer 4: Executed (npm install)
Layer 5: Executed
Layer 6: Executed
Total time: 2 minutes

Build #2 (Code changed):
Layer 1: CACHED ⚡
Layer 2: CACHED ⚡
Layer 3: CACHED ⚡ (package.json unchanged)
Layer 4: CACHED ⚡ (dependencies unchanged)
Layer 5: EXECUTED (source code changed)
Layer 6: CACHED ⚡
Total time: 5 seconds
```

## Critical Notes

🎯 **Execution Time**: RUN executes at build time, CMD executes at runtime—understand this distinction for effective Dockerfiles.

💡 **Layer Optimization**: Order instructions from least to most frequently changing—maximizes cache efficiency.

⚠️ **ADD vs COPY**: Use COPY for simple file copies (explicit), ADD only for archives/URLs (implicit features).

📊 **Single RUN**: Combine related commands in single RUN instruction—reduces layers and image size.

🔄 **Last CMD Wins**: Only the last CMD instruction takes effect—previous CMD instructions are ignored.

✨ **Cache Invalidation**: Any change to an instruction invalidates cache for that layer and all subsequent layers.

## Quick Reference

### Essential Instructions

```dockerfile
FROM image:tag              # Base image
LABEL key="value"           # Metadata
WORKDIR /path               # Set working directory
COPY source dest            # Copy files
ADD source dest             # Copy files (with extras)
RUN command                 # Execute command (build time)
EXPOSE port                 # Document port
ENV KEY=value               # Environment variable
USER username               # Switch user
CMD ["executable"]          # Default command (runtime)
ENTRYPOINT ["executable"]   # Configure as executable
```

### Common Patterns

```dockerfile
# Python app
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]

# Node.js app
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "server.js"]

# Go app (multi-stage)
FROM golang:1.21 AS build
WORKDIR /src
COPY . .
RUN go build -o app

FROM alpine:latest
COPY --from=build /src/app /app
CMD ["/app"]
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/design-container-build-strategy/4-understand-dockerfile-core-concepts)
