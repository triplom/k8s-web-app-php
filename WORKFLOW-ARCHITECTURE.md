# CI/CD Workflow Architecture

## Overview

This repository contains two GitOps CI/CD workflows designed for **master's thesis research** comparing pull-based vs push-based GitOps efficiency.

## Active Workflow

### 🟢 `ci-pipeline.yaml` - **ArgoCD Pull-Based GitOps** (ACTIVE)
- **Purpose**: Primary workflow for pull-based GitOps thesis evaluation
- **Target Infrastructure**: `triplom/infrastructure-repo-argocd` (ArgoCD-managed)
- **Container Strategy**: Single unified container image
- **GitOps Pattern**: Pull-based (ArgoCD polls and syncs)
- **Trigger**: Push to `main`, PRs, manual dispatch

**Flow:**
1. **Test** → PHP 7.4 compatibility testing with Sylius/Symfony
2. **Build** → Unified container image → Push to GHCR  
3. **Update Config** → Updates ArgoCD manifests in `infrastructure-repo-argocd`
4. **ArgoCD Sync** → ArgoCD detects changes and deploys automatically

## Disabled Workflow

### 🔴 `trigger-deploy.yaml` - **Push-Based GitOps** (DISABLED)
- **Purpose**: Legacy workflow for push-based GitOps comparison
- **Target Infrastructure**: `triplom/infrastructure-repo` (traditional push-based)
- **Container Strategy**: Separate PHP-FPM + Nginx images
- **GitOps Pattern**: Push-based (repository dispatch triggers)
- **Status**: **Temporarily disabled** to prevent conflicts

**Flow (when enabled):**
1. **Build** → Separate PHP-FPM & Nginx images → Push to GHCR
2. **Repository Dispatch** → Triggers deployment in `infrastructure-repo`
3. **Push Deploy** → Direct Kubernetes deployment via CI/CD

## Thesis Research Context

### Pull-Based (ArgoCD) Advantages
- ✅ **Continuous Reconciliation**: Detects and corrects drift
- ✅ **Declarative State**: Git as single source of truth
- ✅ **Security**: No cluster credentials in CI/CD
- ✅ **Multi-Environment**: App-of-apps pattern management

### Push-Based Comparison Points
- ✅ **Immediate Deployment**: Event-driven, faster initial response
- ✅ **Simple Architecture**: Direct CI/CD → Kubernetes
- ❌ **Drift Detection**: Manual intervention required
- ❌ **Security**: Requires cluster access tokens in CI/CD

## Switching Between Workflows

### To Enable Pull-Based (ArgoCD) - Current State ✅
```yaml
# In ci-pipeline.yaml
on:
  push:
    branches: [main, 'feature/**']  # ENABLED
```

### To Enable Push-Based (for comparison testing)
```yaml
# In trigger-deploy.yaml
on:
  push:
    branches:
      - main  # UNCOMMENT these lines
    paths-ignore:
      - 'README.md'
      - 'docs/**'
```

### To Enable Both (NOT RECOMMENDED)
- Both workflows will run simultaneously
- May cause resource conflicts and confusion
- Only use for direct A/B testing scenarios

## Infrastructure Repositories

| Workflow | Target Repository | GitOps Pattern | ArgoCD |
|----------|------------------|----------------|---------|
| `ci-pipeline.yaml` | `infrastructure-repo-argocd` | Pull-based | Yes ✅ |
| `trigger-deploy.yaml` | `infrastructure-repo` | Push-based | No ❌ |

## Secrets Required

### For Pull-Based (Active)
- `GITHUB_TOKEN`: GHCR authentication (auto-provided)
- `CONFIG_REPO_PAT`: Updates ArgoCD config repository

### For Push-Based (Disabled)
- `GHCR_TOKEN`: Manual GHCR authentication token
- `INFRA_REPO_PAT`: Repository dispatch to infrastructure repo

## Monitoring Deployments

### Pull-Based (ArgoCD)
- **ArgoCD UI**: `https://argocd.your-domain.com`
- **Config Changes**: Monitor `infrastructure-repo-argocd` commits
- **Kubernetes**: `kubectl get applications -n argocd`

### Push-Based (When Enabled)
- **GitHub Actions**: `infrastructure-repo` workflow runs
- **Direct Monitoring**: Kubernetes cluster directly

## Troubleshooting

### Both Workflows Running
If you see both workflows running simultaneously:
1. Check which one you actually need for your current thesis phase
2. Disable the unwanted workflow by commenting out the `on:` triggers
3. Commit the changes to stop future dual executions

### Current Status
- ✅ **ci-pipeline.yaml**: ACTIVE - Pull-based GitOps for ArgoCD
- ❌ **trigger-deploy.yaml**: DISABLED - Push-based GitOps workflow

This architecture supports comprehensive comparison of GitOps patterns for academic research while preventing conflicts during active development.