# kubectl-kubeplugin

A kubectl plugin that displays resource usage statistics (CPU and Memory) in a convenient format.

## Installation

### 1. Download the script

```bash
curl -O https://raw.githubusercontent.com/SergeyVNec/AsciiArtify/refs/heads/main/scripts/kubectl-kubeplugin
```

### 2. Make the file executable and move it to PATH

```bash
chmod +x kubectl-kubeplugin
sudo mv kubectl-kubeplugin /usr/local/bin/kubectl-kubeplugin
```

Or install to your home directory:

```bash
mkdir -p ~/bin
mv kubectl-kubeplugin ~/bin/kubectl-kubeplugin
chmod +x ~/bin/kubectl-kubeplugin

# Add ~/bin to PATH (if not already added)
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### 3. Verify installation

```bash
kubectl plugin list
```

Expected output:

```
The following compatible plugins are available:

/usr/local/bin/kubectl-kubeplugin
```

## Usage

```bash
kubectl kubeplugin <resource_type> [namespace]
```

### Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resource_type` | Resource type: `pod` or `node` | `pod` |
| `namespace` | Kubernetes namespace | `default` |

### Examples

```bash
# Pod statistics in kube-system namespace
kubectl kubeplugin pod kube-system

# Pod statistics in default namespace
kubectl kubeplugin pod

# Cluster node statistics
kubectl kubeplugin node
```

### Output format

```
<resource_type>, <namespace>, <name>, <cpu>, <memory>
```

### Example output

```bash
$ kubectl kubeplugin pod kube-system

pod, kube-system, coredns-5dd5756b68-abcde, 3m, 18Mi
pod, kube-system, etcd-master, 25m, 64Mi
pod, kube-system, kube-apiserver-master, 45m, 256Mi
pod, kube-system, kube-controller-manager-master, 15m, 48Mi
pod, kube-system, kube-proxy-xyz12, 1m, 15Mi
pod, kube-system, kube-scheduler-master, 4m, 25Mi
```

## Requirements

- Kubernetes cluster with [Metrics Server](https://github.com/kubernetes-sigs/metrics-server) installed
- kubectl configured to work with the cluster
- Bash

### Installing Metrics Server (if not installed)

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

For local clusters (minikube, k3d, kind) additional configuration may be required.

## Help

```bash
kubectl kubeplugin --help
```

## License

MIT
