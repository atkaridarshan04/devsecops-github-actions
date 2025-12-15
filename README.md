# 🚀 DevSecOps CI/CD Pipeline with GitHub Actions

<div align="center">

[![SonarQube](https://img.shields.io/badge/SonarQube-4E9BCD?logo=sonar&logoColor=white)](https://sonarcloud.io/)
[![Trivy](https://img.shields.io/badge/Trivy-1904DA?logo=trivy&logoColor=white)](https://trivy.dev/)
[![Gitleaks](https://img.shields.io/badge/Gitleaks-FF6B6B?logo=git&logoColor=white)](https://gitleaks.io/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![GHCR](https://img.shields.io/badge/GHCR-181717?logo=github&logoColor=white)](https://ghcr.io/)
[![npm](https://img.shields.io/badge/npm-CB3837?logo=npm&logoColor=white)](https://www.npmjs.com/)
[![Vite](https://img.shields.io/badge/Vite-6E9F18?logo=vite&logoColor=white)](https://vite.dev/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?logo=argo&logoColor=white)](https://argoproj.github.io/cd/)

</div>

A comprehensive DevSecOps implementation showcasing a complete CI/CD pipeline with integrated security scanning, code quality analysis, and automated deployment using GitHub Actions, SonarCloud, and Kubernetes.

## 🏗️ Pipeline Architecture

![DevSecOps Pipeline Architecture](./assets/github-actions-devsecops-pipeline-light.png)

This DevSecOps pipeline implements a comprehensive security-first approach with the following stages. For comprehensive pipeline documentation and workflow details **[.github/README.md](./.github/README.md)**

### 🔄 Pipeline Flow
1. **Source Control** - Developer commits trigger the pipeline
2. **Pre-Build Security** - Secrets scanning (Gitleaks) and dependency/IaC scanning (Trivy FS)
3. **Code Validation** - Unit testing with comprehensive test coverage
4. **Static Code Quality** - SonarCloud analysis with quality gate enforcement
5. **Build & CI Artifacts** - Application build and artifact upload
6. **Container Build & Security** - Docker image creation with Trivy image scanning and pushing to GHCR
7. **Continuous Delivery** - Automated Kubernetes manifest updates for GitOps deployment

## 🎮 Application Overview

![Tic-Tac-Toe Application](https://github.com/user-attachments/assets/5b2813a5-f493-4665-8964-77359b5be93a)

<details>
<summary>🚀 <strong>Getting Started</strong></summary>

## Tech Stack

This project features a modern **React-based Tic-Tac-Toe game** built with:
- **React 18** with TypeScript for type safety
- **Tailwind CSS** for responsive styling
- **Vite** for fast development and optimized builds
- **Vitest** for comprehensive unit testing
- **ESLint** for code quality enforcement

## 🔧 Getting Started

### Prerequisites

- **Node.js** (v18 or higher)
- **npm** or **yarn**
- **Docker** (for containerization)
- **kubectl** (for Kubernetes deployment)

### 📦 Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/atkaridarshan04/devsecops-github-actions.git
   cd devsecops-github-actions
   ```

2. **Install dependencies:**
   ```bash
   npm install
   # or
   yarn install
   ```

3. **Start the development server:**
   ```bash
   npm run dev
   # or
   yarn dev
   ```

4. **Open your browser and navigate to:**
   ```
   http://localhost:5173
   ```

### 🏗️ Building for Production

To create a production build:

```bash
npm run build
# or
yarn build
```

The build artifacts will be stored in the `dist/` directory.

### 🧪 Running Tests

Execute the test suite:

```bash
npm test
# or
yarn test
```

### 🐳 Docker Deployment

Build and run the application using Docker:

```bash
# Build the Docker image
docker build -t tic-tac-toe-app .

# Run the container
docker run -p 80:80 tic-tac-toe-app
```

</details>

## Implementation Steps

### 1. Clone Repository

```bash
git clone https://github.com/atkaridarshan04/devsecops-github-actions.git
cd devsecops-github-actions
```
---

<details>
<summary><strong>2. SonarCloud Setup</strong></summary>

### Step 1: Create SonarCloud Project

1. **Navigate to SonarCloud** and create a new project:

![Project Creation 1](./assets/create-project-1.png)

2. **Configure new code definition** - Select "Previous version" for projects following regular versions or releases.

![Project Creation 2](./assets/create-project-2.png)

---

### Step 2: Generate Authentication Token

1. **Go to Security settings** in your SonarCloud account:

2. **Generate a new token** with appropriate permissions for your CI/CD pipeline.

![Token Generation](./assets/sonar-token.png)

---

### Step 3: Configure Project and Organization Keys

#### Project Key Configuration:
![Project Key Setup](./assets/project-key.png)

#### Organization Key Setup:
![Organization Key Setup](./assets/org-key-setup.png)

### Step 4: Project Information Overview

![Project Information](./assets/project-info.png)

The project information page shows:
- **Quality Gate**: Sonar way (default)
- **Quality Profiles**: CSS and TypeScript using Sonar way
- **Project Key**: `atkaridarshan04-devsecops-github-actions`
- **Organization Key**: `atkaridarshan04-devsecops-github-actions`

---

### Step 5: Configure sonar-project.properties

![Sonar Properties Configuration](./assets/sonar-properties.png)

Ensure to update your `sonar-project.properties` file with the correct project and organization configurations.

---

### Step 6: Disable Automatic Analysis for CI/CD

**Important**: For CI/CD integration, disable automatic analysis:

![Disable Automatic Analysis](./assets/disable-automatic-analysis.png)

1. Go to **Administration** → **Analysis Method**
2. **Disable Automatic Analysis** 
3. Select **"With GitHub Actions"** for CI integration

> **Note**: Let the project run with automatic analysis initially to establish a baseline, then disable it for CI/CD integration to avoid conflicts.

</details>

---

<details>
<summary><strong>3. GitHub Token & Secrets Setup</strong></summary>

### Create GitHub Personal Access Token
- Go to **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
- Generate new token with permissions:
  - `contents: write` (for updating Kubernetes manifests)
  - `packages: write` (for pushing to GitHub Container Registry)
  - `actions: read` (for workflow access)

![GitHub Token Creation](./assets/github-token-permissions.png)

### Add Repository Secrets
- Go to your repository **Settings** → **Secrets and variables** → **Actions**
- Add the following secrets:
  ```
  SONAR_TOKEN=your_sonarcloud_token_here
  GH_TOKEN=your_github_personal_access_token
  ```

![GitHub Secrets Setup](./assets/github-secrets.png)

</details>

---

### 4. Run Pipeline

**Make a change and push to main branch or start the workflow manually**

Monitor execution in GitHub Actions tab and verify all security scans pass.
![GitHub Actions Tab](./assets/actions-tab.png)

![Detailes Workflow View](./assets/workflow-details.png)
---

### 5. Verify Analysis Results

Head over to SonarCloud to review the analysis results. 
![SonarCloud Analysis Report](./assets/sonar-analysis-report.png)

---

### 6. Kubernetes Deployment

You can further configure GitOps deployment using ArgoCD or FluxCD to automate application deployment to your Kubernetes cluster.


## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Built with ❤️ for DevSecOps Excellence**
<br>
**⭐ Star this repository if you find it helpful!**
</div>
