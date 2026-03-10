# 🛤️ Jerney — Blog Platform

A Gen-Z vibe blog platform built with a 3-tier architecture, fully containerized with a complete DevSecOps pipeline.

![Tech Stack](https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react)
![Tech Stack](https://img.shields.io/badge/Node.js-20-339933?style=flat-square&logo=node.js)
![Tech Stack](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql)
![Tech Stack](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker)
![Tech Stack](https://img.shields.io/badge/Kubernetes-EKS-326CE5?style=flat-square&logo=kubernetes)
![Tech Stack](https://img.shields.io/badge/Terraform-EKS_Auto-844FBA?style=flat-square&logo=terraform)
![Tech Stack](https://img.shields.io/badge/GitHub_Actions-DevSecOps-2088FF?style=flat-square&logo=githubactions)

## ✨ Features

- 📝 Create blog posts with emoji vibes
- ✏️ Edit your existing posts
- 🗑️ Delete posts you're not feeling anymore
- 💬 Comment on posts
- 🎨 Gen-Z dark UI with glassmorphism and gradients

## 🏗️ Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Frontend   │────▶│   Backend    │────▶│  PostgreSQL   │
│   (React +   │◀────│  (Node.js +  │◀────│  (EBS Volume) │
│    Nginx)    │     │   Express)   │     │              │
│   Port 80    │     │  Port 5000   │     │  Port 5432   │
└──────────────┘     └──────────────┘     └──────────────┘
  Container 1          Container 2          Container 3
```

## 📁 Project Structure

```
Jerney/
├── .github/
│   └── workflows/
│       └── ci-cd.yml              # DevSecOps CI/CD pipeline
├── frontend/                      # React (Vite) frontend
│   ├── src/                       # React components & pages
│   ├── Dockerfile                 # Multi-stage: build + Nginx
│   ├── nginx.conf                 # Nginx config for container
│   ├── .dockerignore
│   ├── .eslintrc.json
│   └── package.json
├── backend/                       # Node.js Express API
│   ├── src/                       # Routes, DB connection
│   ├── Dockerfile                 # Multi-stage: build + runtime
│   ├── .dockerignore
│   ├── .eslintrc.json
│   └── package.json
├── k8s/
│   └── jerney.yaml                # All K8s resources (single file)
├── terraform/                     # EKS Auto Mode infrastructure
│   ├── provider.tf
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── terraform.tfvars
├── deploy/                        # EC2 bare-metal deploy (main branch)
│   ├── setup.sh
│   └── jerney-nginx.conf
├── docker-compose.yml             # Local dev with Docker
├── .checkov.yml                   # IaC scanning config
├── .gitignore
└── README.md
```

---

## 🐳 Quick Start with Docker Compose

The fastest way to run Jerney locally:

```bash
# Clone the repo
git clone <YOUR_REPO_URL>
cd Jerney

# Switch to devops branch
git checkout devops

# Start everything (builds + runs)
docker compose up --build

# Access the blog
open http://localhost
```

**That's it!** Three containers spin up:
- `jerney-frontend` on port **80** (Nginx + React)
- `jerney-backend` on port **5000** (Node.js API)
- `jerney-db` on port **5432** (PostgreSQL)

To stop:
```bash
docker compose down          # Stop containers
docker compose down -v       # Stop + delete database volume
```

---

## 🔐 DevSecOps Pipeline (GitHub Actions)

The CI/CD pipeline runs on **every push and pull request** with these stages:

```
┌──────────┐   ┌──────────┐   ┌──────────────┐   ┌───────────────┐   ┌────────────┐   ┌──────────────┐   ┌──────────────────┐
│  Lint    │──▶│   SCA    │──▶│ Build & Push │──▶│  Image Scan   │──▶│  IaC Scan  │──▶│ Dockerfile   │──▶│ Update K8s       │
│ (ESLint) │   │(npm audit)│   │   (GHCR)     │   │   (Trivy)     │   │ (Checkov)  │   │ Lint         │   │ Manifest         │
└──────────┘   └──────────┘   └──────────────┘   └───────────────┘   └────────────┘   │ (Hadolint)   │   │ (on main only)   │
                                                                                       └──────────────┘   └──────────────────┘
```

### Pipeline Stages

| Stage | Tool | What it does |
|-------|------|--------------|
| **Lint** | ESLint | Catches code quality issues in JS/JSX |
| **SCA** | npm audit | Scans dependencies for known CVEs |
| **Build** | Docker + GHCR | Builds container images, pushes to GitHub Container Registry |
| **Image Scan** | Trivy | Scans container images for OS & library vulnerabilities |
| **IaC Scan** | Checkov | Scans Terraform & K8s manifests for security misconfigurations |
| **Dockerfile Lint** | Hadolint | Lints Dockerfiles for best practices |
| **Update Manifest** | sed + git | Updates `k8s/jerney.yaml` with new image tags (main branch only) |

### Security Best Practices in the Pipeline

- ✅ **Least-privilege permissions** — `contents: read` by default, `write` only where needed
- ✅ **GHCR authentication** — uses `GITHUB_TOKEN`, no external secrets
- ✅ **Image provenance & SBOM** — build attestation enabled
- ✅ **[skip ci]** on manifest commits — prevents infinite loops
- ✅ **PR-safe** — images are NOT pushed on pull requests (build-only)
- ✅ **Dependency caching** — npm & Docker layer caching for speed

---

## ☸️ Deploy to Kubernetes (EKS)

### Step 1: Provision EKS with Terraform

```bash
cd terraform

# Initialize Terraform
terraform init

# Preview changes
terraform plan

# Apply (creates VPC + EKS cluster)
terraform apply
```

> ⏱️ EKS cluster creation takes ~15 minutes.

### Step 2: Configure kubectl

```bash
aws eks update-kubeconfig --region us-east-1 --name jerney-eks
kubectl get nodes  # Verify connection
```

### Step 3: Deploy the App

```bash
# Apply the single manifest file
kubectl apply -f k8s/jerney.yaml

# Check everything is running
kubectl get all -n jerney

# Get the LoadBalancer URL
kubectl get svc jerney-frontend -n jerney
```

The `EXTERNAL-IP` from the frontend service is your app's public URL.

### Useful kubectl Commands

```bash
kubectl get pods -n jerney                    # List pods
kubectl logs -f deploy/jerney-backend -n jerney  # Backend logs
kubectl logs -f deploy/jerney-frontend -n jerney # Frontend logs
kubectl describe pod <pod-name> -n jerney     # Debug a pod
kubectl exec -it deploy/jerney-db -n jerney -- psql -U jerney_user -d jerney_db  # DB shell
```

---

## 🔒 Security Practices Across the Stack

### Container Security
| Practice | Where |
|----------|-------|
| Non-root users | Backend Dockerfile (`appuser`), Frontend (nginx user UID 101) |
| Multi-stage builds | Both Dockerfiles — no build tools in production image |
| Alpine base images | Minimal attack surface |
| `dumb-init` | Backend — proper PID 1 signal handling |
| `no-new-privileges` | Docker Compose `security_opt` |
| Read-only filesystem | Backend container in Compose + K8s |
| `.dockerignore` | Prevent secrets/node_modules from leaking into images |

### Kubernetes Security
| Practice | Where |
|----------|-------|
| NetworkPolicies | DB only accepts traffic from Backend; Backend only from Frontend |
| `automountServiceAccountToken: false` | All pods — no K8s API access unless needed |
| `allowPrivilegeEscalation: false` | All containers |
| `capabilities.drop: [ALL]` | Backend and Frontend pods |
| `runAsNonRoot: true` | Backend and Frontend pods |
| Resource limits | CPU and memory limits on all containers |
| Secrets in K8s Secrets | DB credentials via `secretKeyRef`, not env literals |
| EBS encryption | StorageClass has `encrypted: "true"` |
| Liveness & Readiness probes | All three components |

### Terraform / Infrastructure Security
| Practice | Where |
|----------|-------|
| Secrets-at-rest encryption | `cluster_encryption_config` for K8s secrets |
| Full audit logging | All EKS log types enabled |
| Private endpoint enabled | `cluster_endpoint_private_access = true` |
| Encrypted EBS volumes | StorageClass with `encrypted: "true"` |

### CI/CD Security
| Practice | Where |
|----------|-------|
| Least-privilege `permissions` | `contents: read` default, `write` only for specific jobs |
| No external secrets | Uses built-in `GITHUB_TOKEN` for GHCR |
| Image provenance + SBOM | Enabled in Docker build step |
| Dependency auditing | `npm audit` for both frontend and backend |
| Container scanning | Trivy scans for CRITICAL and HIGH CVEs |
| IaC scanning | Checkov on Terraform + K8s manifests |
| Dockerfile linting | Hadolint for best practices |

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check |
| GET | `/api/posts` | Get all posts |
| GET | `/api/posts/:id` | Get single post with comments |
| POST | `/api/posts` | Create a new post |
| PUT | `/api/posts/:id` | Update a post |
| DELETE | `/api/posts/:id` | Delete a post |
| GET | `/api/comments/post/:postId` | Get comments for a post |
| POST | `/api/comments` | Create a comment |
| DELETE | `/api/comments/:id` | Delete a comment |

---

## 🐛 Troubleshooting

### Docker Compose Issues

```bash
# View logs
docker compose logs -f

# Rebuild from scratch
docker compose down -v && docker compose up --build

# Check if backend can reach DB
docker compose exec backend sh -c "wget -qO- http://localhost:5000/api/health"
```

### Kubernetes Issues

```bash
# Pods stuck in Pending — check events
kubectl describe pod <pod-name> -n jerney

# ImagePullBackOff — check GHCR access
kubectl get events -n jerney --sort-by='.lastTimestamp'

# DB not starting — check PVC
kubectl get pvc -n jerney
kubectl describe pvc jerney-db-pvc -n jerney
```

---

## 🌿 Branch Strategy

| Branch | Purpose |
|--------|---------|
| `main` | Bare-metal EC2 deployment (original setup) |
| `devops` | Containerized with full DevSecOps pipeline, K8s, Terraform |

---

Built with 💜 by the Jerney team. No cap, this blog platform hits different. 🛤️
