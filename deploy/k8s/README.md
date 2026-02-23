# Artifact Keeper — Kubernetes Deployment

Kubernetes manifests for deploying Artifact Keeper using [Kustomize](https://kustomize.io/).

## Quick Start

```bash
# Create namespace
kubectl create namespace artifact-keeper

# Configure secrets (CHANGE the values!)
kubectl create secret generic artifact-keeper-secrets \
  --from-literal=db-password=$(openssl rand -base64 32) \
  --from-literal=jwt-secret=$(openssl rand -base64 64) \
  --namespace artifact-keeper

# Deploy with Kustomize (choose an overlay)
kubectl apply -k overlays/development   # low resources, single replica
kubectl apply -k overlays/staging       # moderate resources
kubectl apply -k overlays/production    # HA, ingress, autoscaling
```

## Structure

```
deploy/k8s/
├── base/                          # Shared base manifests
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secrets.yaml
│   ├── postgres-statefulset.yaml
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   ├── pvc.yaml
│   ├── hpa.yaml
│   ├── backup-cronjob.yaml
│   ├── servicemonitor.yaml
│   └── storage-classes.yaml       # Cloud provider examples (commented)
└── overlays/
    ├── development/               # Single replica, low resources
    ├── staging/                   # Moderate replicas
    └── production/                # HA, ingress, TLS, autoscaling
        ├── ingress.yaml
        ├── patches/
        │   ├── replica-count.yaml
        │   └── resource-limits.yaml
        └── secrets/               # Replace placeholder values!
            ├── db-password.txt
            └── jwt-secret.txt
```

## Production Setup

1. **Replace secret placeholders** in `overlays/production/secrets/`.
2. **Edit** `overlays/production/ingress.yaml` — set your domain instead of `registry.example.com`.
3. **Choose a StorageClass** — see `base/storage-classes.yaml` for AWS/GCP/Azure examples.
4. Deploy:
   ```bash
   kubectl apply -k overlays/production
   ```

## Verify

```bash
kubectl get pods -n artifact-keeper
kubectl get svc -n artifact-keeper
kubectl get ingress -n artifact-keeper
```

## Full Documentation

See <https://artifactkeeper.com/docs/deployment/kubernetes/> for the complete guide
including HA setup, monitoring, backup strategy, and troubleshooting.
