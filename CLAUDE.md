# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a FluxCD v2 GitOps repository that manages Kubernetes deployments for a Talos-based cluster. The repository follows the standard Flux project structure and is designed to declaratively manage all cluster resources through Git.

## Essential Commands

### Flux Bootstrap
```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=$GITHUB_REPO \
  --branch=main \
  --path=./clusters/talos-cluster \
  --personal
```

### Common Flux Operations
- `flux get all` - View all Flux resources and their status
- `flux get sources git` - Check Git repository sync status
- `flux get kustomizations` - View all Kustomizations and reconciliation status
- `flux reconcile source git flux-system` - Force immediate Git sync
- `flux reconcile kustomization flux-system` - Force immediate reconciliation
- `flux logs --all-namespaces --follow` - Watch Flux controller logs
- `flux suspend kustomization <name>` - Temporarily stop reconciliation
- `flux resume kustomization <name>` - Resume reconciliation

### Validation Commands
- `kubectl apply --dry-run=client -f <manifest.yaml>` - Validate YAML syntax
- `flux diff kustomization flux-system` - Preview changes before they're applied
- `kustomize build <directory>` - Build and validate Kustomization

## Architecture and Key Patterns

### Directory Structure
```
clusters/talos-cluster/
├── flux-system/          # Flux components (DO NOT EDIT - managed by Flux)
├── infrastructure/       # Infrastructure components (cert-manager, ingress, etc.)
├── apps/                 # Application deployments
└── kustomization.yaml    # Root Kustomization that references subdirectories
```

### GitOps Workflow
1. All changes are made via Git commits to this repository
2. Flux polls the Git repository every 1 minute for changes
3. Kustomizations reconcile every 10 minutes (or on Git changes)
4. Resources are pruned if removed from Git (pruning is enabled)

### Adding New Resources
When adding new applications or infrastructure:
1. Create appropriate directory structure under `clusters/talos-cluster/`
2. Add Kubernetes manifests or Helm releases
3. Create a Kustomization resource to manage the directory
4. Reference the new Kustomization in the parent directory's kustomization.yaml

Example structure for a new app:
```yaml
# clusters/talos-cluster/apps/my-app/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
  - ingress.yaml
```

### Flux Components
- **Source Controller**: Manages Git repository polling and artifact storage
- **Kustomize Controller**: Applies Kustomization resources
- **Helm Controller**: Manages Helm releases (when used)
- **Notification Controller**: Handles alerts and notifications (when configured)

## Important Patterns

### Multi-Environment Support
To support multiple environments, use directory structure:
```
clusters/
├── staging/
│   └── flux-system/
└── production/
    └── flux-system/
```

### Secret Management
- Use Sealed Secrets or SOPS for encrypting secrets in Git
- Never commit plain-text secrets
- Flux supports automatic decryption of SOPS-encrypted files

### Dependency Management
Use `dependsOn` in Kustomizations to control deployment order:
```yaml
spec:
  dependsOn:
    - name: infrastructure
```

### Health Checks
Flux automatically performs health assessments. Check status with:
- `flux get all -A` - Overview of all resources
- `kubectl get kustomization -n flux-system` - Detailed Kustomization status