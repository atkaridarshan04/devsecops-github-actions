# 🔄 GitHub Actions Workflows

This directory contains the GitHub Actions workflows details that implement our comprehensive DevSecOps CI/CD pipeline.

## 🚀 Pipeline  Workflow Jobs

### 1. 🔍 Security Pre-checks (`security-prechecks`)
**Purpose**: Early security validation before code processing

**Steps**:
- **Repository Checkout**: Full history fetch for comprehensive scanning
- **Gitleaks Scan**: Detects exposed secrets, API keys, and credentials
- **Trivy FS Scan**: Scans dependencies and Infrastructure as Code for vulnerabilities

**Configuration**:
```yaml
# Secrets Scanning (Gitleaks)
- name: Gitleaks
  uses: gitleaks/gitleaks-action@v2
  env:
    GH_TOKEN: ${{ secrets.GH_TOKEN }}

- name: Trivy FS Scan
  uses: aquasecurity/trivy-action@0.33.1
  with: 
    scan-type: 'fs'
    scan-ref: '.'
    severity: CRITICAL,HIGH
    exit-code: 1
    ignore-unfixed: true
```

**Triggers Pipeline Failure**: CRITICAL and HIGH severity vulnerabilities

---

### 2. 🧪 Unit Testing (`unit-test`)
**Purpose**: Validate code functionality and quality

**Dependencies**: `security-prechecks`

**Steps**:
- **Node.js Setup**: Version 20 with npm caching
- **Dependency Installation**: Clean install with `npm ci`
- **Test Execution**: Comprehensive unit test suite with Vitest

**Configuration**:
```yaml
# Install Dependencies
- name: Install Dependencies
  run: npm ci

# Run Unit Tests
- name: Run Unit Tests
  run: npm test
```

---

### 3. 📊 Static Code Quality (`static-testing`)
**Purpose**: Code quality analysis and security scanning

**Dependencies**: `unit-test`

**Steps**:
- **SonarCloud Analysis**: Static code analysis for quality, security, and maintainability
- **Quality Gate Enforcement**: Ensures code meets quality standards

**Configuration**:
```yaml
# SonarCloud Analysis
- name: SonarCloud Scan
  uses: sonarsource/sonarcloud-github-action@v2
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

# Quality Gate Enforcement
- name: SonarCloud Quality Gate
  uses: sonarsource/sonarqube-quality-gate-action@v1
  timeout-minutes: 5
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

**Quality Metrics**:
- Code coverage analysis
- Security vulnerability detection
- Code smell identification
- Technical debt assessment

---

### 4. 🏗️ Build & Artifacts (`build-artifacts`)
**Purpose**: Create production-ready application build

**Dependencies**: `static-testing`

**Steps**:
- **Production Build**: Optimized build with Vite
- **Artifact Upload**: Store build outputs for deployment stages

**Configuration**:
```yaml
# Build Project
- name: Build Project
  run: npm run build

# Upload Artifacts
- name: Upload build artifacts
  uses: actions/upload-artifact@v4
  with:
    name: build-artifacts
    path: dist/
```

**Artifacts**:
- **Name**: `build-artifacts`
- **Path**: `dist/`
- **Retention**: Default GitHub retention policy

---

### 5. 🐳 Container Build & Security (`docker`)
**Purpose**: Containerize application with security scanning

**Dependencies**: `build-artifacts`

**Permissions**:
```yaml
permissions:
  contents: read 
  packages: write
```

**Environment Variables**:
- `REGISTRY`: `ghcr.io`
- `IMAGE_NAME`: `${{ github.repository }}`
- `IMAGE_TAG`: `${{ github.sha }}`

**Steps**:
1. **Artifact Download**: Retrieve build artifacts
2. **Docker Buildx Setup**: Multi-platform build support
3. **GHCR Login**: Authenticate with GitHub Container Registry
4. **Image Build**: Create container image (load locally first)
5. **Trivy Image Scan**: Security scan of container image
6. **Image Push**: Push secure image to registry

**Security Scanning**:
```yaml
# Download build artifacts
- name: Download build artifacts
  uses: actions/download-artifact@v4
  with:
    name: build-artifacts
    path: dist/

# Setup Docker Buildx
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

# Login to GHCR 
- name: Login to GitHub Container Registry
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GH_TOKEN }}

# Build Docker Image
- name: Build Docker image
  uses: docker/build-push-action@v5
  with:
    context: .
    file: Dockerfile.ci
    load: true  # load image into the local Docker daemon of this GitHub runner
    push: false
    tags: ghcr.io/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}

# Trivy Image Scan
- name: Trivy Image Scan
  uses: aquasecurity/trivy-action@0.33.1
  with:
    image-ref: ghcr.io/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
    severity: CRITICAL,HIGH
    exit-code: 1
    ignore-unfixed: true
    vuln-type: os,library

# Push Image
- name: Push Docker image
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ghcr.io/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}

# Export image tag as job output
- name: Export image tag
  id: export-image-tag
  run: |
    echo "image_tag=${{ github.sha }}" >> $GITHUB_OUTPUT
```

**Outputs**:
- `image_tag`: SHA-based image tag for deployment

---

### 6. 🚀 Kubernetes Manifest Update (`update-k8s-manifest`)
**Purpose**: Update deployment manifests for GitOps

**Dependencies**: `docker`

**Permissions**:
```yaml
permissions:
  contents: write   # required to push commit
```

**Steps**:
1. **Repository Checkout**: Get latest manifest files
2. **Image Tag Update**: Update deployment.yaml with new image tag
3. **Commit & Push**: Automated commit for GitOps workflow

**GitOps Integration**:
```bash
- name: Update image tag
  run: |
    sed -i "s|image: .*tic-tac-toe-app:.*|image: ghcr.io/atkaridarshan04/devsecops-github-actions:${{ needs.docker.outputs.image_tag }}|" kubernetes/deployment.yaml
```

## 🔧 Workflow Triggers

### Push Events
```yaml
on:
  push:
    branches: [ "main" ]
    paths-ignore:
      - 'kubernetes/**'  # Prevent infinite loops
      - 'README.md'
      - 'assets/**'
      - '.github/README.md'
```

### Pull Request Events
```yaml
pull_request:
  branches: [ "main" ]
```

### Manual Trigger
```yaml
workflow_dispatch:  # Allows manual workflow execution
```

## 🔐 Required Secrets

Configure these secrets in your GitHub repository:

| Secret Name | Description | Required Permissions |
|-------------|-------------|---------------------|
| `SONAR_TOKEN` | SonarCloud authentication token | Project analysis |
| `GH_TOKEN` | GitHub Personal Access Token | `contents:write`, `packages:write`, `actions:read` |

## 🔒 Security Features

### 🛡️ Multi-Layer Security Approach

1. **Secrets Management**
   - Gitleaks integration prevents secret exposure
   - GitHub Secrets for secure credential storage

2. **Dependency Security**
   - Trivy FS scanning for vulnerable dependencies
   - Automated security updates and alerts

3. **Container Security**
   - Trivy image scanning for container vulnerabilities
   - Distroless base images for minimal attack surface

4. **Code Security**
   - SonarCloud SAST analysis
   - Security hotspot detection and remediation

5. **Infrastructure Security**
   - IaC scanning with Trivy
   - Kubernetes security best practices

## 📊 Pipeline Monitoring

### Success Indicators
- ✅ All security scans pass
- ✅ Unit tests complete successfully
- ✅ SonarCloud quality gate passes
- ✅ Container image builds and scans clean
- ✅ Kubernetes manifests updated
- ✅ ArgoCD deploys changes automatically

### Failure Points
- ❌ Secrets detected by Gitleaks
- ❌ Critical/High vulnerabilities in dependencies
- ❌ Unit test failures
- ❌ SonarCloud quality gate failure
- ❌ Container security vulnerabilities
- ❌ Build or deployment errors

---

