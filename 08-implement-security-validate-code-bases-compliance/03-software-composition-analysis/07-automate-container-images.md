# Automate Container Image Scanning

## Key Concepts
- **Container Security Risks**: Base image vulnerabilities (OS packages), application dependency vulnerabilities, image layer accumulation, configuration issues
- **Scanning Approaches**: Registry scanning (continuous monitoring), build-time scanning (CI/CD), runtime scanning (deployed containers)
- **Container Tools**: Trivy (open-source), Snyk Container, Aqua Security, Anchore Engine, GitHub Advanced Security
- **Best Practices**: Multi-stage builds, minimal base images, scan early and often, security gates, automated remediation

## Understanding Container Security Risks

### Base Image Vulnerabilities
```yaml
Operating System Packages:
  - System libraries: glibc, OpenSSL, zlib may contain vulnerabilities
  - Package managers: apt, yum, apk and their packages have security issues
  - Shell utilities: bash, curl, wget can have vulnerabilities
  - Update lag: Official base images may include outdated packages

Base Image Selection Impact:
  Alpine vs Debian:
    - Alpine: Smaller (~5 MB), uses musl instead of glibc (different vulnerability profile)
    - Debian: Larger, more packages, glibc-based
  
  Distroless Images:
    - Google's distroless: Only application runtime dependencies
    - No shell or package manager
    - Dramatically reduced attack surface
  
  Version Tags:
    - "latest": Can introduce unexpected changes (avoid)
    - Specific versions: Stability but require manual updates (recommended)
```

### Application Dependency Vulnerabilities
- **Language-specific packages**: npm (Node.js), PyPI (Python), Maven/Gradle (Java), NuGet (.NET)
- **Transitive dependencies**: Deep dependency trees (200-500 packages), hidden vulnerabilities, complex updates
- **Update complexity**: Requires understanding compatibility across entire chain

### Image Layer Accumulation
```yaml
Layered Filesystem Issues:
  - Layer inheritance: Each Dockerfile instruction creates layer; vulnerabilities in any layer affect final image
  - Deleted files persist: Files deleted in later layers still exist in earlier layers
  - Secrets in history: Secrets in early layers remain even if removed later
  - Build-time dependencies: Compilers, build tools shouldn't appear in runtime images
```

### Configuration Vulnerabilities
- **Running as root**: Containers with unnecessary root privileges
- **Exposed ports**: Unnecessary ports expand attack surface
- **SUID binaries**: Enable privilege escalation attacks
- **Insecure defaults**: Default configurations may not follow security best practices

## Container Scanning Approaches

### 1. Registry Scanning
**Continuous registry monitoring for centralized scanning and policy enforcement**

**Azure Container Registry with Microsoft Defender:**
```yaml
Automatic Scanning:
  - Microsoft Defender for Containers: Automatically scans ACR images
  - Trigger-based scanning: Push, import, pull operations
  - Continuous re-scanning: Periodically rescanned for new vulnerabilities
  - Recommendations: Security Center provides remediation guidance

Registry Scan Triggers:
  - Push triggers: New images scan on push
  - Import triggers: External registry imports scanned
  - Pull triggers: Images scanned within 24 hours after pull
  - Periodic rescanning:
    * Last 90 days for pushed images (daily)
    * Last 30 days for pulled images (daily)
```

### 2. Build-Time Scanning
**CI/CD pipeline integration catches vulnerabilities before images reach registries**

**Azure Pipelines Container Scanning:**
```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: "ubuntu-latest"

variables:
  imageName: "myapp"
  dockerfilePath: "$(Build.SourcesDirectory)/Dockerfile"

steps:
  - task: Docker@2
    displayName: "Build container image"
    inputs:
      command: "build"
      repository: "$(imageName)"
      dockerfile: "$(dockerfilePath)"
      tags: "$(Build.BuildNumber)"

  - task: AquaScannerCLI@4
    displayName: "Scan image with Aqua Security"
    inputs:
      image: "$(imageName):$(Build.BuildNumber)"
      scanType: "local"
      register: false
      hideBase: false
      showNegligible: false

  - script: |
      docker run --rm \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -v $(Build.SourcesDirectory):/src \
        aquasec/trivy image \
        --severity HIGH,CRITICAL \
        --exit-code 1 \
        $(imageName):$(Build.BuildNumber)
    displayName: "Scan with Trivy (fail on high/critical)"

  - task: Docker@2
    displayName: "Push image to registry"
    condition: succeeded()
    inputs:
      command: "push"
      repository: "$(imageName)"
      tags: "$(Build.BuildNumber)"
```

**GitHub Actions Container Scanning:**
```yaml
name: Container Build and Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  IMAGE_NAME: myapp
  REGISTRY: ghcr.io

jobs:
  build-and-scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      security-events: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Build container image
        uses: docker/build-push-action@v4
        with:
          context: .
          load: true
          tags: ${{ env.IMAGE_NAME }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Scan image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: "${{ env.IMAGE_NAME }}:${{ github.sha }}"
          format: "sarif"
          output: "trivy-results.sarif"
          severity: "CRITICAL,HIGH"

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: "trivy-results.sarif"

      - name: Scan with Snyk
        uses: snyk/actions/docker@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          image: ${{ env.IMAGE_NAME }}:${{ github.sha }}
          args: --severity-threshold=high --file=Dockerfile

      - name: Log in to GitHub Container Registry
        if: github.event_name == 'push'
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Push image to registry
        if: github.event_name == 'push'
        run: |
          docker tag ${{ env.IMAGE_NAME }}:${{ github.sha }} \
            ${{ env.REGISTRY }}/${{ github.repository }}:${{ github.sha }}
          docker push ${{ env.REGISTRY }}/${{ github.repository }}:${{ github.sha }}
```

**Build-Time Scanning Benefits:**
- Fail fast: Prevent vulnerable images from reaching registries
- Developer feedback: Immediate feedback during build process
- Policy enforcement: Enforce security policies before production
- Build artifact validation: Only compliant images progress

### 3. Runtime Scanning
**Deployed container monitoring detects vulnerabilities in production**

**Kubernetes Admission Controllers (OPA Gatekeeper):**
```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredscanstatus
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredScanStatus
      validation:
        openAPIV3Schema:
          type: object
          properties:
            maxSeverity:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredscanstatus

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not scan_clean(container.image)
          msg := sprintf("Image %v has not been scanned or has vulnerabilities", [container.image])
        }

        scan_clean(image) {
          # Query registry for scan status
          scan_result := data.scans[image]
          scan_result.status == "passed"
        }
```

**Azure Kubernetes Service Runtime Protection:**
- Microsoft Defender for Containers provides:
  - **Behavioral analytics**: Monitor container behavior for anomalies
  - **Threat intelligence**: Compare against known attack patterns
  - **MITRE ATT&CK mapping**: Map threats to MITRE framework
  - **Alert generation**: Security alerts for suspicious activities

## Container Scanning Tools

### Trivy (Aqua Security)
**Comprehensive open-source container vulnerability scanner**

```yaml
Key Features:
  - Comprehensive scanning: OS packages, app dependencies, IaC configs, secrets
  - Multi-format support: Container images, filesystems, git repos, Kubernetes clusters
  - Offline scanning: Operates in air-gapped environments
  - Fast performance: Lightweight with fast scan times
  - SBOM generation: CycloneDX and SPDX formats

Azure Pipelines Trivy Integration:
```

```bash
- script: |
    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
    echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
    sudo apt-get update
    sudo apt-get install trivy

    trivy image \
      --severity HIGH,CRITICAL \
      --exit-code 1 \
      --no-progress \
      --format json \
      --output trivy-results.json \
      $(imageName):$(Build.BuildNumber)
  displayName: "Scan image with Trivy"
```

### Snyk Container
| Feature | Description |
|---------|-------------|
| **Base Image Recommendations** | Suggests alternative base images with fewer vulnerabilities |
| **Prioritization** | Based on exploitability and business impact |
| **Fix Guidance** | Specific remediation steps for discovered vulnerabilities |
| **Kubernetes Integration** | Scans images deployed to Kubernetes clusters |

### Aqua Security
- **Image Assurance**: Comprehensive scanning with customizable policies
- **Runtime Protection**: Monitors running containers for suspicious behavior
- **Compliance Checking**: CIS Docker Benchmark validation, custom policies
- **Supply Chain Security**: Verifies image provenance, detects supply chain attacks

### Anchore Engine
- **Policy-Driven**: Flexible policy engine for security and compliance rules
- **Deep Inspection**: Analyzes image layers, packages, configuration
- **Custom Policies**: Organization-specific security/compliance policies
- **API-Driven**: REST API for integration with custom tooling

### GitHub Advanced Security
**Enterprise-grade security for repositories including container scanning**

```yaml
Container Security Features:
  - Dependency scanning: Detect vulnerable dependencies in container images
  - Secret scanning: Identify exposed secrets (API keys, tokens) in layers/Dockerfiles
  - Code scanning: CodeQL analysis of Dockerfiles and application code
  - Security advisories: Integration with GitHub Advisory Database
  - Supply chain security: Dependency graph and Dependabot for containers

Enabling GitHub Advanced Security:
  Organization:
    1. Settings → Code security and analysis
    2. Enable GitHub Advanced Security
    3. Configure Dependabot alerts, Secret scanning, Code scanning

  Repository:
    1. Settings → Code security and analysis
    2. Enable Dependency graph (free for public repos)
    3. Enable Dependabot alerts and security updates
    4. Enable Secret scanning (requires GH Advanced Security for private repos)
    5. Enable Code scanning with CodeQL
```

## Best Practices for Container Scanning

### 1. Implement Multi-Stage Builds
**Separate build and runtime dependencies**

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM node:18-slim
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Benefits:**
- Smaller images (no build tools in runtime)
- Fewer vulnerabilities (build dependencies excluded)
- Better performance (smaller = faster push/pull/start)

### 2. Use Minimal Base Images

| Base Image Type | Size | Attack Surface | Use Case |
|----------------|------|----------------|----------|
| **Alpine** | ~5 MB | Minimal (musl libc) | Small footprint applications |
| **Distroless** | Varies | Minimal (no shell/package manager) | Production applications |
| **Slim Variants** | Medium | Reduced (no unnecessary utilities) | General purpose |
| **Specific Versions** | Varies | Consistent | Reproducibility (avoid "latest") |

**Example Distroless Image:**
```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o myapp .

FROM gcr.io/distroless/static-debian11
COPY --from=builder /app/myapp /
USER nonroot:nonroot
CMD ["/myapp"]
```

### 3. Scan Early and Often

```yaml
Scanning Frequency:
  - Developer workstation: Scan locally before committing
  - Pull request validation: Scan in CI on every PR
  - Main branch builds: Comprehensive scans on main merges
  - Registry admission: Scan before accepting into registries
  - Scheduled rescanning: Periodically rescan stored images for new vulnerabilities
  - Pre-deployment: Final validation before production deployment
```

### 4. Implement Security Gates

```yaml
- task: Trivy@1
  inputs:
    image: "$(imageName):$(Build.BuildNumber)"
    severityThreshold: "HIGH"
    exitCode: 1
  displayName: "Security gate: block high/critical vulnerabilities"
```

**Gate Examples:**
- Vulnerability severity: Fail builds with critical/high vulnerabilities
- License compliance: Block images with prohibited licenses
- Configuration issues: Prevent deployment of images running as root
- Secret detection: Fail if secrets detected in image layers

### 5. Automate Remediation

**Dependabot for Dockerfiles:**
```yaml
version: 2
updates:
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
```

**Automated Updates:**
- Enable Dependabot to update base image versions and dependencies in Dockerfiles
- Configure pipelines to automatically rebuild images when base images updated
- Use tools to automatically patch base images or dependencies
- Ensure automated updates trigger comprehensive testing

### 6. Maintain Scan Result History
- **Centralized reporting**: Aggregate scan results in dashboards
- **Trend analysis**: Track vulnerability counts over time
- **Compliance audits**: Maintain history for compliance evidence
- **SBOM archiving**: Archive Software Bill of Materials for deployed images

### 7. Implement Image Signing

```yaml
- task: Docker@2
  inputs:
    command: "sign"
    arguments: "--key $(signingKey) $(imageName):$(Build.BuildNumber)"
  displayName: "Sign container image"

- script: |
    docker trust inspect --pretty $(imageName):$(Build.BuildNumber)
  displayName: "Verify image signature"
```

**Image Signing Benefits:**
- **Provenance verification**: Confirm images from trusted sources
- **Tampering detection**: Detect if images modified after signing
- **Policy enforcement**: Deploy only signed images to production

## Critical Notes
- ⚠️ **Multi-Layer Risk**: Vulnerabilities can exist in base images, OS packages, application dependencies, and configuration
- 💡 **Three Scanning Points**: Registry (continuous) + Build-time (prevention) + Runtime (production monitoring)
- 🎯 **Minimal Base Images**: Distroless and Alpine images dramatically reduce attack surface
- 📊 **Multi-Stage Builds**: Separate build and runtime dependencies (smaller, fewer vulnerabilities)
- 🔒 **Fail Fast**: Build-time scanning prevents vulnerable images from reaching registries/production
- 🚀 **Automate Everything**: Automated scanning, remediation, and image signing for comprehensive security

[Learn More](https://learn.microsoft.com/en-us/training/modules/software-composition-analysis/7-automate-container-images)
