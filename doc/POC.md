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
- **Access to the Argo CD UI**: via `kubectl port-forward` and `https://localhost:8080`.

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

## 2. ArgoCD installation process

**2.1 Create cluster:**

```bash
k3d cluster create argocd
```
Verification of the created cluster:

```bash
kubectl cluster-info
```

**2.2 Create namespace:**

```bash
kubectl create namespace argocd
```
Verification:
```bash
kubectl get ns
```

**2.3 Run ArgoCD in container:**

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Verification containers:
```bash
kubectl get pod -n argocd -w
```

**2.4 Get access to GUI interface:**

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**2.5 Get password:**

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d;echo
```

**2.6 First enter:**

Go to https://127.0.0.1:8080.</br>
ArgoCD GUI runs on port 443 (HTTPS). You must accept the self-signed certificate and continue.</br>
- Login name: admin</br>
- Password: in step 2.5

## 3. Create project

**3.1 **