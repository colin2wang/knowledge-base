# Kubernetes Installation Guide

## Overview

This guide covers two approaches to installing Kubernetes on Ubuntu:
- **k3s**: Lightweight Kubernetes distribution (recommended for development)
- **kubeadm**: Full Kubernetes installation (for production environments)

## 1. System Preparation

### Common Prerequisites

```shell script
# Update system packages
sudo apt update && sudo apt upgrade -y

# Disable firewall (optional but recommended)
sudo ufw disable

# Disable system swap
sudo swapoff -a
# Make it permanent by commenting out swap in /etc/fstab
sudo sed -i '/swap/s/^/#/' /etc/fstab

# Check system resources
free -h  # Memory check
nproc    # CPU core count
df -h    # Disk space
```

### Install Container Runtime

**Option 1: Docker (Traditional)**
```shell script
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

**Option 2: Containerd (Recommended for k3s)**
```shell script
# Install containerd
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

## 2. k3s Installation (Lightweight - Recommended)

### Quick Installation

```shell script
# Install with default settings
curl -sfL https://get.k3s.io | sh -

# Enable and start service
sudo systemctl enable k3s
sudo systemctl start k3s
```

### Advanced Installation Options

**Custom Configuration:**
```shell script
# Install with specific components disabled
curl -sfL https://get.k3s.io | \
    INSTALL_K3S_EXEC="--disable=traefik,servicelb --write-kubeconfig-mode=644" \
    sh -
```

**Version Specific Installation:**
```shell script
# Install specific k3s version
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.2+k3s1 sh -
```

**Chinese Mirror (Better Connectivity):**
```shell script
# Use domestic mirror
curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | \
    INSTALL_K3S_MIRROR=cn \
    sh -
```

### Verification

```shell script
# Check service status
sudo systemctl status k3s

# Check k3s version
k3s --version

# Check Kubernetes nodes
kubectl get nodes

# Check system pods
kubectl get pods -A
```

### Cluster Management

**Access Configuration:**
```shell script
# Copy kubeconfig to user directory
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
export KUBECONFIG=~/.kube/config
```

**Join Worker Nodes:**
```shell script
# On master node - get token
sudo cat /var/lib/rancher/k3s/server/node-token

# On worker node - join cluster
curl -sfL https://get.k3s.io | \
    K3S_URL=https://<master-ip>:6443 \
    K3S_TOKEN=<node-token-from-master> \
    sh -
```

**Uninstall:**
```shell script
# Server uninstall
/usr/local/bin/k3s-uninstall.sh

# Agent uninstall (worker nodes)
/usr/local/bin/k3s-agent-uninstall.sh
```

## 3. Full Kubernetes Installation (kubeadm)

### Repository Setup

```shell script
# Install required packages
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg

# Add Kubernetes GPG key
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg

# Add Kubernetes repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | \
    sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
```

### Install Kubernetes Components

```shell script
# Install kubeadm, kubelet, kubectl
sudo apt install -y kubelet kubeadm kubectl

# Prevent automatic updates
sudo apt-mark hold kubelet kubeadm kubectl
```

### Initialize Master Node

```shell script
# Initialize cluster with Chinese mirror
sudo kubeadm init \
  --image-repository=registry.aliyuncs.com/google_containers \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=<your-master-ip> \
  --kubernetes-version=v1.28.0
```

### Post-Installation Setup

```shell script
# Configure kubectl for current user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install network plugin (Calico)
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml

# Optional: Allow master to run workloads
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

### Join Worker Nodes

```shell script
# On worker nodes, use the join command from master initialization output
kubeadm join <master-ip>:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

## 4. Useful Commands and Troubleshooting

### Cluster Status Checking

```shell script
# Basic cluster information
kubectl cluster-info
kubectl get nodes
kubectl get pods -A

# Component status
kubectl get cs

# Detailed node information
kubectl describe nodes
```

### Common Troubleshooting

```shell script
# Check service logs
sudo journalctl -xeu kubelet
sudo journalctl -u k3s -f

# Reset cluster (kubeadm)
sudo kubeadm reset
sudo systemctl restart kubelet

# Version information
kubeadm version
kubectl version --client
```

### Resource Monitoring

```shell script
# Check system resources
kubectl top nodes
kubectl top pods -A

# Check pod logs
kubectl logs <pod-name> -n <namespace>

# Describe resources
kubectl describe pod <pod-name> -n <namespace>
```

## 5. Comparison: k3s vs Full Kubernetes

| Aspect | k3s | Full Kubernetes (kubeadm) |
|--------|-----|---------------------------|
| **Resource Usage** | ~500MB RAM | ~2GB RAM |
| **Installation** | Single command | Multi-step process |
| **Components** | Embedded | Separate packages |
| **Storage** | SQLite (default) | etcd |
| **Network** | Flannel built-in | Manual setup required |
| **Load Balancer** | Klipper (built-in) | Manual setup |
| **External Dependencies** | Minimal | Docker/containerd + etcd |
| **Use Case** | Development, Edge, Testing | Production environments |
| **Complexity** | Low | High |
| **Learning Curve** | Gentle | Steep |

## 6. Best Practices

### For Development Environments
- Use **k3s** for local development
- Enable single-node mode for simplicity
- Use lightweight storage options

### For Production Environments
- Use **kubeadm** for production clusters
- Implement proper HA setup
- Configure monitoring and logging
- Plan capacity and resource allocation

### Security Considerations
- Regular security updates
- Proper RBAC configuration
- Network policy implementation
- Secure API server access

This guide provides comprehensive instructions for both lightweight and full Kubernetes installations, allowing you to choose the approach that best fits your requirements.