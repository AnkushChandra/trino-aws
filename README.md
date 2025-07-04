# Trino on AWS EKS

This repository contains the Kubernetes manifests for deploying Trino on AWS EKS using Helm charts and Argo CD.

## Components

- **trino-values.yaml**: Helm values file for Trino configuration
- **trino-manifests.yaml**: Generated Kubernetes manifests from Helm chart
- **namespace.yaml**: Namespace definition for Trino
- **argocd-application.yaml**: Argo CD Application manifest for GitOps deployment

## Trino Configuration

The Trino deployment includes:
- 1 Coordinator node with 8GB heap memory
- 2 Worker nodes with 8GB heap memory each
- LoadBalancer service for external access
- Production-ready configuration with G1GC

## Deployment Options

### Option 1: Direct kubectl deployment
```bash
kubectl apply -f namespace.yaml
kubectl apply -f trino-manifests.yaml
```

### Option 2: Argo CD GitOps deployment
```bash
kubectl apply -f argocd-application.yaml
```

### Option 3: Helm deployment
```bash
helm repo add trino https://trinodb.github.io/charts
helm repo update
helm install trino trino/trino --values trino-values.yaml --namespace trino --create-namespace
```

## Access Trino

After deployment, get the LoadBalancer URL:
```bash
kubectl get svc -n trino trino
```

Access the Trino Web UI at: `http://<EXTERNAL-IP>:8080`

## Resource Requirements

- **Coordinator**: 2 CPU cores, 8GB memory
- **Workers**: 2 CPU cores each, 8GB memory each
- **Total**: 6 CPU cores, 24GB memory

## Monitoring

Trino exposes metrics at `/v1/info` and `/v1/status` endpoints for monitoring integration.

## Scaling

To scale workers:
```bash
kubectl scale deployment trino-worker --replicas=<desired-count> -n trino
```

Or update the `server.workers` value in `trino-values.yaml` and redeploy.
