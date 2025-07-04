# Trino on AWS EKS

This repository contains the configuration for deploying Trino on AWS EKS using Argo CD with native Helm support.

## Components

- **trino-values.yaml**: Helm values file for Trino configuration
- **argocd-application.yaml**: Argo CD Application manifest for GitOps (static manifests)
- **argocd-helm-application.yaml**: Argo CD Application manifest using Helm plugin (recommended)
- **namespace.yaml**: Namespace definition for Trino

## Trino Configuration

The Trino deployment includes:
- 1 Coordinator node with 8GB heap memory
- 5 Worker nodes with 8GB heap memory each
- LoadBalancer service for external access
- Production-ready configuration with G1GC

## Deployment Options

### Option 1: Argo CD with Helm Plugin (Recommended)
```bash
kubectl apply -f argocd-helm-application.yaml
```

This approach:
- Uses Trino Helm chart directly from https://trinodb.github.io/charts
- No need to generate/commit static manifests
- Cleaner repository with only configuration files
- Native Helm features and lifecycle management

### Option 2: Argo CD with Static Manifests
```bash
kubectl apply -f argocd-application.yaml
```

### Option 3: Direct Helm deployment
```bash
helm repo add trino https://trinodb.github.io/charts
helm repo update
helm install trino trino/trino --values trino-values.yaml --namespace trino --create-namespace
```

## Making Changes

### With Helm Plugin Approach:
1. Edit `trino-values.yaml` or update values in `argocd-helm-application.yaml`
2. Commit and push changes
3. Argo CD automatically syncs the changes

### With Static Manifests Approach:
1. Edit `trino-values.yaml`
2. Regenerate manifests: `helm template trino trino/trino --values trino-values.yaml --namespace trino > trino-manifests.yaml`
3. Commit and push all changes
4. Argo CD automatically syncs the changes

## Access Trino

After deployment, get the LoadBalancer URL:
```bash
kubectl get svc -n trino trino
```

Access the Trino Web UI at: `http://<EXTERNAL-IP>:8080`

## Resource Requirements

- **Coordinator**: 2 CPU cores, 8GB memory
- **Workers**: 2 CPU cores each, 8GB memory each (5 workers)
- **Total**: 12 CPU cores, 48GB memory

## Monitoring

Trino exposes metrics at `/v1/info` and `/v1/status` endpoints for monitoring integration.

## Scaling

### With Helm Plugin:
Update the `server.workers` value in `argocd-helm-application.yaml` or `trino-values.yaml`

### With Static Manifests:
```bash
kubectl scale deployment trino-worker --replicas=<desired-count> -n trino
```

Or update the `server.workers` value in `trino-values.yaml` and redeploy.
