# AsciiArtify GitOps PoC with Argo CD

## The purpose of PoC

The goal of this Proof of Concept (PoC) is to demonstrate that the **AsciiArtify** team can:

- deploy a local Kubernetes cluster;
- install and configure the **Argo CD** GitOps system;
- provide developers with access to the Argo CD web interface;
- prepare the environment for further deployment of the AsciiArtify MVP application.

The PoC is performed locally on the developer's machine and does not claim to be production-level, but it reproduces the basic GitOps architecture.

---

## Architecture PoC

- **Kubernetes-cluster**: local cluster based on `k3d` (K3s in Docker).
- **Git repository**: `https://github.com/<username>/AsciiArtify`.
- **GitOps-system**: `Argo CD`, set in namespace `argocd`.
- **Access to the Argo CD UI**: via `kubectl port-forward` or `https://localhost:8080`.

---

## 1. Prerequisites

The following must be installed on the work machine:

- **Docker** (or compatible runtime)  
- **k3d**  
- **kubectl**


Verification:

```bash
docker version
k3d version
kubectl version --client
```

## 2. Argo CD installation process

2.1 Create cluster:

```bash
k3d cluster create argocd
```
Verification of the created cluster:

```bash
kubectl cluster-info
```

2.2 Create namespace:

```bash
kubectl create namespace argocd
```
Verification:
```bash
kubectl get ns
```