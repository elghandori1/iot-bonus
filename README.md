# IoT Bonus: GitOps Infrastructure with Kubernetes, ArgoCD & GitLab

Inception of Things (IoT) is a DevOps-focused project that explores modern cloud-native infrastructure using Kubernetes and GitOps principles. The project demonstrates how applications and services can be automatically deployed, synchronized, and managed through declarative configuration stored in Git repositories.

## Project Overview

This project demonstrates a complete **GitOps workflow** using modern cloud-native technologies:

- **Kubernetes Cluster**: Lightweight k3s/k3d for development and testing
- **GitLab**: In-cluster Git repository serving as the source of truth
- **ArgoCD**: Continuous deployment tool that watches GitLab and automatically syncs Kubernetes manifests
- **Automated Infrastructure**: One-command setup that provisions everything automatically


## Learning Objectives

Through this project, you will gain practical experience with:

- Kubernetes fundamentals and cluster management

- Container orchestration and service discovery

- GitOps workflows and infrastructure as code

- Continuous deployment with ArgoCD

- GitLab integration for deployment automation

- Application lifecycle management

- Declarative infrastructure configuration

- Monitoring and synchronization of Kubernetes resources

### Key Features

✅ **Automated Infrastructure-as-Code**: Single-command cluster and application deployment  
✅ **GitOps Workflow**: Changes to Git automatically trigger Kubernetes deployments  
✅ **In-Cluster GitLab**: GitLab runs inside Kubernetes, no external dependencies  
✅ **Lightweight Setup**: Uses k3s/k3d for minimal resource requirements  
✅ **Reproducible**: Scripted setup ensures consistent deployments  
✅ **Automated Testing**: Includes smoke tests and validation  

---

## Architecture

### High-Level Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Development Machine                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          k3d Kubernetes Cluster (Docker)            │  │
│  │  ┌──────────────────────────────────────────────┐  │  │
│  │  │        Kubernetes Namespaces                 │  │  │
│  │  │                                              │  │  │
│  │  │  ┌─────────────────────────────────────┐   │  │  │
│  │  │  │ GitLab Namespace                    │   │  │  │
│  │  │  │ ├─ GitLab WebService               │   │  │  │
│  │  │  │ ├─ GitLab PostgreSQL               │   │  │  │
│  │  │  │ └─ Git Repository: iot-gitops      │   │  │  │
│  │  │  └─────────────────────────────────────┘   │  │  │
│  │  │                                              │  │  │
│  │  │  ┌─────────────────────────────────────┐   │  │  │
│  │  │  │ ArgoCD Namespace                    │   │  │  │
│  │  │  │ ├─ ArgoCD Server (UI)              │   │  │  │
│  │  │  │ ├─ Repo Server                     │   │  │  │
│  │  │  │ ├─ Controller                      │   │  │  │
│  │  │  │ └─ Application: wil-playground    │   │  │  │
│  │  │  └─────────────────────────────────────┘   │  │  │
│  │  │                                              │  │  │
│  │  │  ┌─────────────────────────────────────┐   │  │  │
│  │  │  │ Dev Namespace                       │   │  │  │
│  │  │  │ ├─ wil-playground Deployment       │   │  │  │
│  │  │  │ └─ wil-playground Service          │   │  │  │
│  │  │  └─────────────────────────────────────┘   │  │  │
│  │  │                                              │  │  │
│  │  └──────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          Local Port-Forward Tunnels                 │  │
│  │  ├─ localhost:9080 → GitLab UI                     │  │
│  │  ├─ localhost:8080 → ArgoCD UI                     │  │
│  │  └─ localhost:8888 → wil-playground App            │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### GitOps Workflow

```
Developer Makes Change
        ↓
  Edit in GitLab UI
   (or git clone → edit → push)
        ↓
  Commit to main branch
        ↓
  Git Repository Updated
        ↓
  ArgoCD Detects Change
   (polls every ~3 minutes)
        ↓
  ArgoCD Syncs Manifest
        ↓
  Kubernetes Deployment Rolls Out
        ↓
  New Application Version Live
```

---

## Technology Stack

| Component | Purpose | Version/Notes |
|-----------|---------|---------------|
| **Docker** | Container runtime for k3d nodes | Latest stable |
| **Kubernetes (k3s)** | Lightweight Kubernetes distribution | Latest via k3d |
| **k3d** | K3s running in Docker containers | Latest stable |
| **kubectl** | Kubernetes command-line interface | Stable release |
| **Helm** | Package manager for Kubernetes | v3+ |
| **ArgoCD** | GitOps continuous deployment | Stable manifest |
| **GitLab** | Git repository & UI | v8.8.1 (Helm chart) |
| **git** | Version control client | System default |

---

## Prerequisites

- **Linux machine** (or WSL2 on Windows, similar on macOS)
- **CPU**: At least 2 vCPU (4+ recommended)
- **RAM**: 6-8 GB free (GitLab is resource-heavy even when minimized)
- **Storage**: 20+ GB free disk space
- **Network**: Internet access to download container images
- **Permissions**: Ability to run Docker (may require sudo or docker group membership)

---

## Quick Start

### Step 1: Clone the Repository
```bash
git clone <repository-url>
cd iot-bonus
```

### Step 2: Install Tools (One-Time)
```bash
bash bonus/scripts/install.sh
```

This script installs all required tools:
- Docker
- kubectl
- k3d
- Helm
- ArgoCD CLI
- git

**Note**: You may need to log out/in if your Docker group was added, or run with `sudo`.

### Step 3: Setup Infrastructure (Full Automation)
```bash
bash bonus/scripts/setup.sh
```

This script automatically:
1. Creates a k3d Kubernetes cluster named `mycluster`
2. Creates namespaces: `argocd`, `dev`, `gitlab`
3. Deploys ArgoCD
4. Deploys GitLab via Helm
5. Creates the GitLab project `root/iot-gitops`
6. Pushes Kubernetes manifests to GitLab
7. Configures ArgoCD to watch GitLab
8. Creates the ArgoCD Application with automated sync
9. Sets up port-forwards for UI and app access

**Expected Runtime**: 10-15 minutes (first boot slower on small VMs)

### Step 4: Access the Interfaces

After setup completes, you have:

**GitLab UI:**
```bash
# Open browser: http://localhost:9080
# Password file: cat ~/gitlabrootpass
```

**ArgoCD UI:**
```bash
# Open browser: https://localhost:8080
# Password file: cat ~/argocdpass
```

**Application:**
```bash
# Test with: curl http://localhost:8888
# Or open browser: http://localhost:8888
```

---

## Directory Structure

```
iot-bonus/
├── README.md                          # This file
├── bonus/                             # Bonus Part: GitLab + ArgoCD extension
│   ├── scripts/
│   │   ├── install.sh                # Install host dependencies + Helm
│   │   └── setup.sh                  # Full infrastructure automation
│   ├── confs/
│   │   ├── values.yaml               # GitLab Helm chart configuration
│   │   └── manifests/
│   │       └── deployment.yaml       # Kubernetes Deployment & Service
│   └── reference-commands.txt        # Manual commands reference
└── p3/                               # Part 3: Base Kubernetes setup
    ├── scripts/
    │   ├── install.sh                # Install base tools
    │   └── setup.sh                  # Setup base cluster + ArgoCD
    └── confs/
        ├── deployment.yaml
        └── argocd-app.yaml
```

---

## Setup Details

### Install Script (`bonus/scripts/install.sh`)

Installs required host tools:

```bash
✓ Docker Engine       - Container runtime (required for k3d)
✓ kubectl            - Kubernetes CLI tool
✓ k3d                - Lightweight Kubernetes distribution
✓ Helm               - Package manager (for GitLab chart)
✓ ArgoCD CLI         - ArgoCD command-line interface
✓ git                - Version control client
```

**Idempotent**: Safe to run multiple times; skips already-installed tools.

### Setup Script (`bonus/scripts/setup.sh`)

Full infrastructure automation in 10 steps:

**1. Cluster Creation**
```bash
k3d cluster create mycluster
```
Creates a lightweight Kubernetes cluster running in Docker containers.

**2. Namespace Creation**
```bash
kubectl create namespace {argocd,dev,gitlab}
```
Isolates system components:
- `argocd`: ArgoCD deployment controller
- `gitlab`: GitLab services
- `dev`: Your application deployment

**3. ArgoCD Installation**
```bash
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Deploys ArgoCD (GitOps continuous deployment system).

**4. GitLab Installation via Helm**
```bash
helm install gitlab gitlab/gitlab -f bonus/confs/values.yaml --version 8.8.1
```
Deploys GitLab with lightweight configuration:
- No ingress controller
- No cert-manager
- No Prometheus monitoring
- No GitLab Runner
- Minimized resource requests

**5. GitLab Access**
```bash
kubectl port-forward -n gitlab svc/gitlab-webservice-default 9080:8181
```
Creates temporary local tunnel to GitLab UI.

**6. GitLab Project Creation**
```bash
gitlab-rails runner "Project.create!(...)"
```
Automatically creates `root/iot-gitops` project in GitLab.

**7. Push Manifests to GitLab**
Copies `bonus/confs/manifests/deployment.yaml` to GitLab repository as source of truth.

**8. ArgoCD Repository Configuration**
```bash
argocd repo add http://gitlab-webservice-default.gitlab.svc.cluster.local:8181/root/iot-gitops.git
```
Registers GitLab repository with ArgoCD (repo-server runs in-cluster and clones via Kubernetes DNS).

**9. ArgoCD Application Creation**
```bash
argocd app create wil-playground \
  --repo http://gitlab-webservice-default.gitlab.svc.cluster.local:8181/root/iot-gitops.git \
  --path manifests \
  --revision main \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace dev \
  --sync-policy automated \
  --auto-prune \
  --self-heal
```
Creates ArgoCD application that watches GitLab and auto-syncs Kubernetes manifests.

**10. Port-Forward Setup**
```bash
kubectl port-forward svc/wil-playground -n dev 8888:8888
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Creates local tunnels for UI and app access (runs in background with nohup).

### GitLab Helm Configuration (`bonus/confs/values.yaml`)

Lightweight profile for small VMs:

```yaml
global:
  edition: ce                # Community Edition
  kas.enabled: false         # No Agent Server
  registry.enabled: false    # No Container Registry
  ingress.enabled: false     # No Ingress Controller
  certmanager: false         # No Certificate Manager

gitlab:
  webservice:
    minReplicas: 1           # Single instance
    workerProcesses: 2       # Reduced workers
  sidekiq:
    minReplicas: 1           # Single worker
    
prometheus.install: false     # No monitoring
gitlab-runner.install: false  # No CI/CD runners
```

**Resource Requirements**: ~3 vCPU / 6-8 GiB RAM

### Deployment Manifest (`bonus/confs/manifests/deployment.yaml`)

The Kubernetes manifest watched by ArgoCD:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wil-playground
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wil-playground
  template:
    metadata:
      labels:
        app: wil-playground
    spec:
      containers:
      - name: wil-playground
        image: wil42/playground:v1    # ← Change this to v2 to test GitOps
        ports:
        - containerPort: 8888
---
apiVersion: v1
kind: Service
metadata:
  name: wil-playground
  namespace: dev
spec:
  selector:
    app: wil-playground
  ports:
  - port: 8888
    targetPort: 8888
```

---

## Testing the Deployment

### Smoke Tests (Post-Setup)

After setup completes, verify all components are working:

```bash
# 1. Check cluster
k3d cluster list

# 2. Check all pods
kubectl get pods -A

# 3. Check GitLab pods
kubectl get pods -n gitlab

# 4. Check ArgoCD application status
argocd app get wil-playground

# 5. Check Kubernetes deployment
kubectl -n dev get pods

# 6. Test application
curl http://localhost:8888

# Expected response: {"status":"ok","message":"v1"}
```

### GitOps Workflow Test: Image Bump (v1 → v2)

This demonstrates the complete GitOps workflow:

#### Option A: Via GitLab UI

1. Open GitLab: `http://localhost:9080`
2. Login with `root` / (password in `~/gitlabrootpass`)
3. Navigate to: `root/iot-gitops` project
4. Open file: `manifests/deployment.yaml`
5. Edit the image line:
   ```yaml
   FROM: image: wil42/playground:v1
   TO:   image: wil42/playground:v2
   ```
6. Commit directly to `main` branch
7. Wait 3-5 minutes for ArgoCD to detect and sync
8. Check status:
   ```bash
   argocd app get wil-playground
   kubectl -n dev get pods
   ```
9. Test updated app:
   ```bash
   curl http://localhost:8888
   # Expected: {"status":"ok","message":"v2"}
   ```

#### Option B: Via Git CLI

```bash
# Get credentials
ROOT_PASS="$(cat ~/gitlabrootpass)"
PASS_ENC="$(printf '%s' "$ROOT_PASS" | python3 -c "import sys,urllib.parse; print(urllib.parse.quote(sys.stdin.read().strip(), safe=''))")"

# Clone repository
git clone "http://root:${PASS_ENC}@127.0.0.1:9080/root/iot-gitops.git"
cd iot-gitops

# Configure git
git config user.name "iot-tester"
git config user.email "test@local"

# Modify deployment
sed -i 's|image: wil42/playground:v1|image: wil42/playground:v2|' manifests/deployment.yaml

# Commit and push
git add manifests/deployment.yaml
git commit -m "bump playground image to v2"
git push origin main

# Wait for sync and test
sleep 180  # ArgoCD default polling ~3 minutes
curl http://localhost:8888
```

### Continuous Monitoring

**Watch ArgoCD synchronization:**
```bash
watch argocd app get wil-playground
```

**Watch Kubernetes rollout:**
```bash
kubectl -n dev get pods -w
```

**Watch all pods:**
```bash
watch kubectl get pods -A
```

**Check ArgoCD UI:**
```
https://localhost:8080  (admin / cat ~/argocdpass)
```

---

## GitOps Workflow

### Important Concepts

#### Source of Truth Hierarchy

```
bonus/confs/manifests/
    ↓ (copied once during setup)
GitLab Repository (iot-gitops)
    ↓ (watched by ArgoCD)
Kubernetes Cluster (current state)
```

**Critical**: After initial setup, NEVER edit `bonus/confs/manifests/` directly.
- It's only the bootstrap seed
- ArgoCD watches GitLab, NOT local files
- All changes must be made in GitLab (UI or git push)

#### Why This Matters

| Action | Effect |
|--------|--------|
| Edit `bonus/confs/manifests/deployment.yaml` | **Nothing** - ArgoCD sees no change |
| Edit in GitLab UI + Commit | ✓ ArgoCD detects and syncs |
| git clone → edit → git push | ✓ ArgoCD detects and syncs |

#### GitOps Principles

1. **Declarative**: Describe desired state in Git (not imperative kubectl apply)
2. **Version Controlled**: All changes tracked with audit trail
3. **Automated**: ArgoCD automatically enforces desired state
4. **Self-Healing**: If pods crash/deleted, ArgoCD restores them
5. **Rollback**: Revert commits in Git to rollback deployments

---

## Troubleshooting

### Common Issues

#### 1. `localhost:8888` Not Responding

**Problem**: Application service isn't accessible
```bash
# Check if port-forward is running
ps aux | grep "port-forward.*wil-playground"

# Check if deployment exists
kubectl -n dev get pods

# Check pod logs
kubectl -n dev logs -l app=wil-playground
```

**Solution**: Restart port-forward
```bash
kubectl port-forward -n dev svc/wil-playground 8888:8888
```

#### 2. GitLab UI Won't Load (`localhost:9080`)

**Problem**: GitLab port-forward isn't working
```bash
# Check if port-forward is running
ps aux | grep "port-forward.*gitlab"

# Check GitLab pods
kubectl get pods -n gitlab
```

**Solution**: Restart GitLab port-forward
```bash
kubectl port-forward -n gitlab svc/gitlab-webservice-default 9080:8181
```

#### 3. ArgoCD Not Syncing Changes

**Problem**: Pushed changes to GitLab but ArgoCD hasn't synced
```bash
# Check ArgoCD app status
argocd app get wil-playground

# Check ArgoCD repo connection
argocd repo list

# Check ArgoCD logs
kubectl logs -n argocd deploy/argocd-repo-server
```

**Solution**: 
- ArgoCD polls every ~3 minutes (default)
- Force sync:
  ```bash
  argocd app sync wil-playground
  ```

#### 4. Cluster Creation Fails

**Problem**: `k3d cluster create mycluster` fails
```bash
# Check Docker is running
docker ps

# Check Docker has enough resources
docker system df
```

**Solution**: 
- Ensure Docker daemon is running: `systemctl start docker`
- Clean up old containers: `docker system prune`
- Add more resources if needed

#### 5. Setup Script Hangs

**Problem**: Script stops at GitLab installation
```bash
# Check GitLab pod status
kubectl get pods -n gitlab

# Check resource availability
kubectl top nodes
free -h
```

**Solution**: 
- GitLab initial boot takes 5-10 minutes
- VM may need more RAM (recommend 8 GB+)
- Monitor with: `watch kubectl get pods -n gitlab`

#### 6. Git Push Authentication Fails

**Problem**: 
```
fatal: Authentication failed for 'http://...'
```

**Solution**: 
```bash
# Re-read password
ROOT_PASS="$(cat ~/gitlabrootpass)"

# Verify password is correct
echo "$ROOT_PASS"

# Re-encode for URL
PASS_ENC="$(printf '%s' "$ROOT_PASS" | python3 -c "import sys,urllib.parse; print(urllib.parse.quote(sys.stdin.read().strip(), safe=''))")"

# Update git remote
git remote set-url origin "http://root:${PASS_ENC}@127.0.0.1:9080/root/iot-gitops.git"
```

### Getting Help

**Check cluster status:**
```bash
k3d cluster list
kubectl cluster-info
kubectl get nodes
```

**View all pods:**
```bash
kubectl get pods -A
```

**Inspect specific resources:**
```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
```

**Check events:**
```bash
kubectl get events -A --sort-by='.lastTimestamp'
```

---

## Manual Commands Reference

### Cluster Management

```bash
# List clusters
k3d cluster list

# Start cluster
k3d cluster start mycluster

# Stop cluster
k3d cluster stop mycluster

# Delete cluster (DESTRUCTIVE)
k3d cluster delete mycluster

# Recreate from scratch after deletion
bash bonus/scripts/setup.sh
```

### Kubernetes Operations

```bash
# Get all pods
kubectl get pods -A

# Get pods in specific namespace
kubectl get pods -n dev
kubectl get pods -n gitlab
kubectl get pods -n argocd

# Watch pods in real-time
kubectl get pods -n dev -w

# Check pod logs
kubectl logs <pod-name> -n <namespace>

# Get pod details
kubectl describe pod <pod-name> -n <namespace>

# Execute command in pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh

# Get services
kubectl get svc -A

# Port-forward to pod
kubectl port-forward pod/<pod-name> <local-port>:<pod-port> -n <namespace>

# Port-forward to service
kubectl port-forward svc/<service-name> <local-port>:<svc-port> -n <namespace>
```

### GitLab Operations

```bash
# Access GitLab UI
kubectl port-forward -n gitlab svc/gitlab-webservice-default 9080:8181

# Get initial root password
kubectl -n gitlab get secret gitlab-initial-root-password-* -o jsonpath="{.data.password}" | base64 -d

# SSH into GitLab pod
kubectl -n gitlab exec -it deployment/gitlab-toolbox -- /bin/bash

# View GitLab logs
kubectl logs -n gitlab deployment/gitlab-webservice-default
kubectl logs -n gitlab deployment/gitlab-sidekiq
```

### ArgoCD Operations

```bash
# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get ArgoCD admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Login via CLI
argocd login localhost:8080 --username admin --password <password> --insecure

# List applications
argocd app list

# Check application status
argocd app get wil-playground

# Sync application (force synchronization)
argocd app sync wil-playground

# Monitor sync in real-time
argocd app wait wil-playground --timeout 300

# View application details
argocd app info wil-playground

# Get application history
argocd app history wil-playground

# Rollback to previous sync
argocd app rollback wil-playground 0
```

### Git & GitLab Repository

```bash
# Get Git credentials
GITLAB_ROOT_PASS="$(cat ~/gitlabrootpass)"

# Encode password for URL
PASS_ENC="$(printf '%s' "$GITLAB_ROOT_PASS" | python3 -c "import sys,urllib.parse; print(urllib.parse.quote(sys.stdin.read().strip(), safe=''))")"

# Clone repository
git clone "http://root:${PASS_ENC}@127.0.0.1:9080/root/iot-gitops.git"
cd iot-gitops

# Configure Git (first time)
git config user.name "DevOps Team"
git config user.email "devops@local"

# Make changes and push
nano manifests/deployment.yaml
git add manifests/deployment.yaml
git commit -m "update deployment image"
git push origin main

# View commit history
git log --oneline

# Check remote URL
git remote -v

# Pull latest changes
git pull origin main
```

### Testing & Validation

```bash
# Test application
curl http://localhost:8888

# Test with verbose output
curl -v http://localhost:8888

# Get JSON response
curl -s http://localhost:8888 | jq .

# Check Kubernetes deployment rollout
kubectl rollout status deployment/wil-playground -n dev

# View deployment details
kubectl describe deployment wil-playground -n dev

# Get deployment YAML
kubectl get deployment wil-playground -n dev -o yaml

# Check replica sets
kubectl get rs -n dev

# Check events
kubectl get events -n dev
```

### Debugging

```bash
# Check kubectl config
kubectl config view

# Switch context (if multiple clusters)
kubectl config current-context
kubectl config use-context <context-name>

# Check API server connectivity
kubectl api-resources

# Verify service account
kubectl describe sa default -n dev

# Check RBAC permissions
kubectl auth can-i get pods --as=system:serviceaccount:dev:default

# View all resources in namespace
kubectl api-resources --verbs=list --namespaced=true -o wide

# Get resource usage
kubectl top nodes
kubectl top pods -A

# Check persistent volumes
kubectl get pv
kubectl get pvc -A
```

---

## Additional Resources

### Project Structure Documentation

**Part 3 (Base)**: Kubernetes cluster with basic ArgoCD setup
- Location: `p3/`
- Purpose: Foundation for GitOps with GitHub
- Components: k3s, ArgoCD, basic manifests

**Bonus Part**: Enhanced setup with in-cluster GitLab
- Location: `bonus/`
- Purpose: Complete GitOps workflow with GitLab as source of truth
- Components: k3d, ArgoCD, in-cluster GitLab

### External Documentation

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [ArgoCD Documentation](https://argoproj.github.io/argo-cd/)
- [GitLab Documentation](https://docs.gitlab.com/)
- [k3d Documentation](https://k3d.io/)
- [Helm Documentation](https://helm.sh/docs/)
- [GitOps Principles](https://www.gitops.tech/)

---

## Summary

This project demonstrates a complete **GitOps infrastructure setup** with:

1. **Automated Provisioning**: One-command infrastructure deployment
2. **Lightweight Kubernetes**: k3s/k3d for development environments
3. **In-Cluster GitLab**: No external dependencies, all tools integrated
4. **Continuous Deployment**: ArgoCD automatically syncs Git changes to Kubernetes
5. **Reproducible Setup**: All configuration scripted and version controlled
6. **Easy Testing**: Built-in smoke tests and validation procedures

The setup is production-grade for development/testing but can be extended for larger deployments by modifying k3d cluster arguments, GitLab Helm values, and adding proper ingress/TLS configuration.
