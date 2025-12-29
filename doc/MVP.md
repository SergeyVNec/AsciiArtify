# AsciiArtify MVP Deployment with ArgoCD

## Overview

This document describes the deployment of the **AsciiArtify MVP** application using ArgoCD GitOps approach. The MVP demonstrates the core functionality of converting images to ASCII art using Machine Learning.

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Git Repository │────▶│     ArgoCD      │────▶│   Kubernetes    │
│  (go-demo-app)  │     │  (GitOps CD)    │     │    Cluster      │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                              │                        │
                              ▼                        ▼
                        Auto-Sync              ┌──────────────┐
                        Detection              │  Ambassador  │
                                               │  (Ingress)   │
                                               └──────┬───────┘
                                                      │
                              ┌────────────────┬──────┴───────┬────────────────┐
                              ▼                ▼              ▼                ▼
                        ┌──────────┐    ┌──────────┐   ┌──────────┐    ┌──────────┐
                        │   API    │    │   Data   │   │   Image  │    │   NATS   │
                        │ Service  │    │ Service  │   │ Service  │    │  (Queue) │
                        └──────────┘    └──────────┘   └──────────┘    └──────────┘
```

## Prerequisites

- Running k3d cluster with ArgoCD installed (see [POC.md](POC.md))
- kubectl configured
- Access to ArgoCD UI (port-forward on 8080)

## Deployment Steps

### 1. Create Namespace

```bash
kubectl create namespace demo
```

### 2. Create ArgoCD Application

#### Option A: Via ArgoCD UI

1. Open ArgoCD UI at `https://localhost:8080`
2. Click **+ NEW APP**
3. Fill in the form:

| Field | Value |
|-------|-------|
| Application Name | `demo` |
| Project | `default` |
| Sync Policy | `Automatic` |
| Repository URL | `https://github.com/den-vasyliev/go-demo-app` |
| Revision | `HEAD` |
| Path | `helm` |
| Cluster URL | `https://kubernetes.default.svc` |
| Namespace | `demo` |

4. Enable **Auto-Sync** options:
   - ✅ Prune Resources
   - ✅ Self Heal

5. Click **CREATE**

#### Option B: Via YAML Manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/den-vasyliev/go-demo-app
    targetRevision: HEAD
    path: helm
  destination:
    server: https://kubernetes.default.svc
    namespace: demo
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Apply with:
```bash
kubectl apply -f demo-application.yaml
```

### 3. Verify Deployment

```bash
# Check ArgoCD application status
kubectl get applications -n argocd

# Check deployed pods
kubectl get pods -n demo

# Check services
kubectl get svc -n demo
```

Expected output:
```
NAME                      READY   STATUS    RESTARTS   AGE
ambassador-xxx            1/1     Running   0          5m
demo-api-xxx              1/1     Running   0          5m
demo-ascii-xxx            1/1     Running   0          5m
demo-data-xxx             1/1     Running   0          5m
demo-front-xxx            1/1     Running   0          5m
demo-img-xxx              1/1     Running   0          5m
demo-nats-xxx             3/3     Running   0          5m
```

### 4. Access the Application

```bash
# Port-forward to Ambassador service
kubectl port-forward -n demo svc/ambassador 8088:80
```

## Testing the Application

### Convert Image to ASCII Art

```bash
# Using curl (replace with your image path)
curl -F 'image=@/path/to/your/image.png' localhost:8088/img/
```

### Example with wget

```bash
# Download a test image
wget -O test.png https://www.google.com/images/branding/googlelogo/2x/googlelogo_color_272x92dp.png

# Convert to ASCII
curl -F 'image=@test.png' localhost:8088/img/
```

## Auto-Sync Demonstration

ArgoCD automatically detects changes in the Git repository and syncs them to the cluster.

### How Auto-Sync Works

1. **Polling**: ArgoCD polls the Git repository every 3 minutes (default)
2. **Detection**: When changes are detected, ArgoCD compares desired state with live state
3. **Sync**: If differences found, ArgoCD automatically applies changes
4. **Self-Heal**: If someone manually changes resources, ArgoCD reverts them

### Verify Auto-Sync Status

```bash
# Check sync status
kubectl get applications demo -n argocd -o jsonpath='{.status.sync.status}'

# Watch for changes
kubectl get applications demo -n argocd -w
```

In ArgoCD UI:
- **Sync Status**: Shows `Synced` when in sync with Git
- **Health Status**: Shows `Healthy` when all resources are running
- **Last Sync**: Shows timestamp of last synchronization

## Application Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /` | Health check |
| `POST /img/` | Convert image to ASCII art |
| `GET /ascii/` | Get ASCII result |

## Troubleshooting

### Pods not starting

```bash
# Check pod events
kubectl describe pod <pod-name> -n demo

# Check logs
kubectl logs <pod-name> -n demo
```

### Ambassador service stuck in Progressing

If using k3d, the LoadBalancer might not get external IP. Use port-forward instead:

```bash
kubectl port-forward -n demo svc/ambassador 8088:80
```

### Sync failed

```bash
# Check ArgoCD application status
kubectl describe application demo -n argocd

# Force sync
argocd app sync demo --force
```

## Demo

### Application Demo

[![AsciiArtify Demo](https://img.youtube.com/vi/YOUR_VIDEO_ID/hqdefault.jpg)](https://youtu.be/YOUR_VIDEO_ID)

▶️ *Click on the image to watch the demo*

### What the demo shows:

1. ArgoCD UI with deployed application
2. All pods running in `demo` namespace
3. Port-forward to access the application
4. Converting an image to ASCII art using curl
5. Auto-sync in action when Git repository changes

---

## Summary

| Component | Status |
|-----------|--------|
| ArgoCD Application | ✅ Configured |
| Auto-Sync | ✅ Enabled |
| Self-Heal | ✅ Enabled |
| Application | ✅ Deployed |

The MVP is now deployed using GitOps approach. Any changes pushed to the `go-demo-app` repository will be automatically synchronized to the Kubernetes cluster.

## References

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [go-demo-app Repository](https://github.com/den-vasyliev/go-demo-app)
- [k3d Documentation](https://k3d.io/)
