# Examine Multi-Stage Dockerfiles

Multi-stage Dockerfiles revolutionize container builds by enabling complex build processes within a single file while keeping final images lean and secure. This advanced technique is essential for production-ready containerized applications.

## What Are Multi-Stage Builds?

Multi-stage builds use multiple `FROM` statements in a single Dockerfile, where each `FROM` instruction starts a new build stage. You can selectively copy artifacts from one stage to another, leaving behind everything you don't need in the final image.

**Problem Without Multi-Stage Builds**:
```
Build Container (Full SDK):
├── Build tools (compilers, SDKs)
├── Development dependencies
├── Source code
├── Build artifacts
└── Total size: 2 GB

Deploy this entire 2 GB image to production! ❌
- Includes unnecessary build tools
- Large attack surface
- Slow deployments
```

**Solution With Multi-Stage Builds**:
```
Build Stage (Full SDK):      Production Stage (Runtime Only):
├── Build tools              ├── Compiled artifacts
├── Dev dependencies         ├── Runtime dependencies
├── Source code              ├── No build tools
├── Compile application      └── Total size: 100 MB ✅
└── Size: 2 GB (discarded)
```

## Multi-Stage Build Example

### Complete Multi-Stage Dockerfile (ASP.NET Core)

```dockerfile
# ============================================
# Stage 1: BASE - Runtime foundation
# ============================================
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

# ============================================
# Stage 2: BUILD - Build environment
# ============================================
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy project file and restore dependencies
COPY ["WebApplication1.csproj", ""]
RUN dotnet restore "./WebApplication1.csproj"

# Copy source code and build
COPY . .
WORKDIR "/src/."
RUN dotnet build "WebApplication1.csproj" -c Release -o /app/build

# ============================================
# Stage 3: PUBLISH - Prepare deployment artifacts
# ============================================
FROM build AS publish
RUN dotnet publish "WebApplication1.csproj" -c Release -o /app/publish

# ============================================
# Stage 4: FINAL - Production image
# ============================================
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WebApplication1.dll"]
```

### Build Commands

```bash
# Build entire multi-stage Dockerfile (final stage)
docker build -t webapp:1.0 .

# Build specific stage (development)
docker build --target base -t webapp:dev .

# Build with build arguments
docker build --build-arg BUILD_CONFIGURATION=Debug -t webapp:debug .
```

## Multi-Stage Build Mechanics

### Named Stages

Stages can be named using `AS` keyword and reference each other:

```dockerfile
# Create named stage "builder"
FROM golang:1.21 AS builder
WORKDIR /src
COPY . .
RUN go build -o app

# Reference "builder" stage in final stage
FROM alpine:latest
COPY --from=builder /src/app /app
CMD ["/app"]
```

**Stage Naming Best Practices**:
```dockerfile
# ✅ Good: Descriptive names
FROM node:18 AS dependencies
FROM node:18 AS builder
FROM node:18 AS tester
FROM nginx:alpine AS final

# ❌ Bad: Generic names
FROM node:18 AS stage1
FROM node:18 AS stage2
```

### Cross-Stage Copying

Use `COPY --from=stage_name` to copy files between stages:

```dockerfile
# Stage 1: Build application
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build  # Creates /app/dist

# Stage 2: Production image
FROM nginx:alpine
# Copy only built artifacts from build stage
COPY --from=build /app/dist /usr/share/nginx/html
```

**Multiple Source Stages**:
```dockerfile
FROM node:18 AS frontend-build
WORKDIR /app/frontend
COPY frontend/ .
RUN npm install && npm run build

FROM golang:1.21 AS backend-build
WORKDIR /app/backend
COPY backend/ .
RUN go build -o server

# Final stage combines both
FROM alpine:latest
COPY --from=frontend-build /app/frontend/dist /var/www
COPY --from=backend-build /app/backend/server /usr/local/bin/
CMD ["server"]
```

### Optimized Image Size

Compare image sizes with and without multi-stage builds:

```
Without Multi-Stage (Deploy build environment):
├── Base: golang:1.21 (800 MB)
├── Build tools: 400 MB
├── Dependencies: 300 MB
├── Source code: 50 MB
├── Compiled binary: 15 MB
└── Total: 1,565 MB ❌

With Multi-Stage (Deploy only binary):
├── Base: alpine:latest (5 MB)
├── Compiled binary: 15 MB
└── Total: 20 MB ✅

Size Reduction: 98.7%! 🚀
```

## Stage Breakdown Explained

### Stage 1: BASE - Runtime Foundation

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
```

**Purpose**: Establish the minimal runtime environment
- Uses optimized runtime image (no SDK)
- Sets up working directory
- Documents exposed ports
- Foundation for final production image

**Size**: ~200 MB (ASP.NET runtime only)

### Stage 2: BUILD - Build Environment

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["WebApplication1.csproj", ""]
RUN dotnet restore "./WebApplication1.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "WebApplication1.csproj" -c Release -o /app/build
```

**Purpose**: Compile the application
- Uses complete SDK image (includes compilers, tools)
- Restores NuGet packages
- Compiles source code
- Outputs built assemblies

**Size**: ~2 GB (full .NET SDK)
**Note**: This stage is discarded in final image!

### Stage 3: PUBLISH - Deployment Preparation

```dockerfile
FROM build AS publish
RUN dotnet publish "WebApplication1.csproj" -c Release -o /app/publish
```

**Purpose**: Prepare deployment artifacts
- Inherits from build stage
- Creates optimized production build
- Collects all required binaries and dependencies
- Outputs ready-to-deploy package

**Output**: `/app/publish` directory with production-ready files

### Stage 4: FINAL - Production Image

```dockerfile
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WebApplication1.dll"]
```

**Purpose**: Create minimal production image
- Starts from lightweight runtime base
- Copies ONLY published artifacts
- No build tools or source code
- Configures startup command

**Size**: ~250 MB (runtime + application)
**Result**: 87% smaller than including full SDK!

## Testing Integration in Multi-Stage Builds

Add test stage between build and publish:

```dockerfile
# ============================================
# Stage 1: Build application
# ============================================
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["App/App.csproj", "App/"]
COPY ["App.Tests/App.Tests.csproj", "App.Tests/"]
RUN dotnet restore "App/App.csproj"
RUN dotnet restore "App.Tests/App.Tests.csproj"
COPY . .
RUN dotnet build "App/App.csproj" -c Release

# ============================================
# Stage 2: Run tests (BLOCKS BUILD IF FAIL)
# ============================================
FROM build AS test
WORKDIR /src/App.Tests
RUN dotnet test --no-build -c Release --logger "trx;LogFileName=test_results.xml"
# Build fails if tests fail! ✅

# ============================================
# Stage 3: Publish (only if tests pass)
# ============================================
FROM build AS publish
WORKDIR /src/App
RUN dotnet publish -c Release -o /app/publish

# ============================================
# Stage 4: Final production image
# ============================================
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "App.dll"]
```

**Testing Benefits**:
- ✅ Tests run as part of build process
- ✅ Failed tests prevent deployment
- ✅ No separate test infrastructure needed
- ✅ Consistent test environment

**Build with Tests**:
```bash
# Build and run tests (fails if tests fail)
docker build -t app:1.0 .

# Build without running tests (skip test stage)
docker build --target publish -t app:1.0-untested .

# Build only tests (for debugging)
docker build --target test -t app:test .
```

## Stage Ordering Strategy

### Optimal Stage Order: BASE First

```dockerfile
# ✅ Recommended: Base stage first
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base  # 1. Runtime
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build    # 2. Build
FROM build AS publish                              # 3. Publish
FROM base AS final                                 # 4. Final
```

**Advantages**:

**1. Development Efficiency**:
```bash
# Target base stage for rapid development iteration
docker build --target base -t app:dev .
# Skips build/publish stages (seconds instead of minutes)
```

**2. Debug Optimization with Visual Studio**:
```
Visual Studio Container Tools workflow:
1. Build --target base (fast)
2. Mount compiled binaries into base stage
3. Run application with debugger attached
4. No need to rebuild entire image on code change

Result: Hot-reload in containers! 🔥
```

**3. Layered Development**:
```bash
# Different targets for different purposes
docker build --target base -t app:runtime .      # Runtime only
docker build --target build -t app:builder .     # Build environment
docker build --target test -t app:test .         # Test runner
docker build -t app:prod .                       # Full production build
```

### Stage Order Example

```dockerfile
# BASE: Minimal runtime (for local dev and final prod)
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

# BUILD: Full SDK for compilation
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.csproj ./
RUN dotnet restore
COPY . .
RUN dotnet build -c Release

# TEST: Run unit tests
FROM build AS test
RUN dotnet test --no-build -c Release

# PUBLISH: Prepare artifacts
FROM build AS publish
RUN dotnet publish -c Release -o /app/publish

# FINAL: Combine base + published artifacts
FROM base AS final
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "app.dll"]
```

## Real-World Multi-Stage Examples

### Example 1: Go Application (Extreme Size Reduction)

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

# Production stage (from scratch!)
FROM scratch
COPY --from=builder /app/app /app
ENTRYPOINT ["/app"]
```

**Result**:
- Build stage: ~300 MB
- Final image: ~10 MB (just the binary!)

### Example 2: Node.js Application

```dockerfile
# Dependencies stage
FROM node:18-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
# Save production dependencies

# Build stage
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci  # Install all dependencies (including dev)
COPY . .
RUN npm run build  # TypeScript compilation, bundling, etc.

# Test stage
FROM build AS test
RUN npm run test
RUN npm run lint

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
# Copy production dependencies from first stage
COPY --from=dependencies /app/node_modules ./node_modules
# Copy built application from build stage
COPY --from=build /app/dist ./dist
COPY package*.json ./
USER node
CMD ["node", "dist/server.js"]
```

### Example 3: Python Application

```dockerfile
# Build stage: Install dependencies
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Production stage: Minimal runtime
FROM python:3.11-alpine AS production
WORKDIR /app
# Copy installed packages from builder
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

## Advanced Multi-Stage Patterns

### Pattern 1: Parallel Build Stages

```dockerfile
# Frontend build (parallel)
FROM node:18 AS frontend
WORKDIR /frontend
COPY frontend/ .
RUN npm install && npm run build

# Backend build (parallel)
FROM golang:1.21 AS backend
WORKDIR /backend
COPY backend/ .
RUN go build -o server

# Combine both in final image
FROM alpine:latest
COPY --from=frontend /frontend/dist /var/www
COPY --from=backend /backend/server /usr/local/bin/
CMD ["server"]
```

### Pattern 2: External Image as Stage

```dockerfile
# Use external image as source
FROM nginx:alpine AS nginx-source

# Custom image
FROM alpine:latest
# Copy nginx binary from official image
COPY --from=nginx-source /usr/sbin/nginx /usr/sbin/nginx
COPY --from=nginx-source /etc/nginx /etc/nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Pattern 3: Build Argument Selection

```dockerfile
ARG BUILD_ENV=production

FROM node:18 AS base
WORKDIR /app

FROM base AS development
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

FROM base AS production
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "server.js"]

# Select stage based on BUILD_ENV
FROM ${BUILD_ENV} AS final
```

**Build with Selection**:
```bash
# Build development image
docker build --build-arg BUILD_ENV=development -t app:dev .

# Build production image
docker build --build-arg BUILD_ENV=production -t app:prod .
```

## Critical Notes

🎯 **Size Reduction**: Multi-stage builds can reduce final image size by 80-95%—essential for production deployments.

💡 **Named Stages**: Always name stages with `AS` keyword—makes Dockerfile self-documenting and enables selective builds.

⚠️ **Cross-Stage Copy**: Use `COPY --from=stage` to transfer only necessary artifacts—everything else is discarded.

📊 **Test Integration**: Add test stage between build and publish—failed tests block deployment automatically.

🔄 **Stage Order**: Place base stage first for optimal development workflow—enables fast iteration with `--target base`.

✨ **From Scratch**: For compiled languages (Go, Rust), use `FROM scratch` as final base—creates smallest possible images.

## Quick Reference

### Multi-Stage Build Template

```dockerfile
# Stage 1: Build
FROM sdk-image:tag AS build
WORKDIR /src
COPY . .
RUN build-command

# Stage 2: Test (optional)
FROM build AS test
RUN test-command

# Stage 3: Production
FROM runtime-image:tag AS final
COPY --from=build /src/output /app
CMD ["run-command"]
```

### Build Commands

```bash
# Build final stage (default)
docker build -t app:prod .

# Build specific stage
docker build --target test -t app:test .

# Build with arguments
docker build --build-arg ENV=prod -t app:prod .

# View image size
docker images app:prod
```

### Size Comparison Formula

```
Size Reduction % = ((Build Image - Final Image) / Build Image) × 100

Example:
Build Image: 2000 MB
Final Image: 250 MB
Reduction: ((2000 - 250) / 2000) × 100 = 87.5%
```

[Learn More](https://learn.microsoft.com/en-us/training/modules/design-container-build-strategy/5-examine-multi-stage-dockerfiles)
