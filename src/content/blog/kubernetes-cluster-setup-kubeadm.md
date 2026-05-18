---
heroImage: '/kubernetes-cluster-setup-kubeadm.svg'
title: 'Kubernetes Cluster Setup Using Kubeadm'
description: 'A comprehensive guide to bootstrapping a production-ready Kubernetes cluster from scratch using kubeadm.'
pubDate: 'May 10 2026'
---

Setting up a Kubernetes cluster manually is an excellent way to understand its underlying architecture. `kubeadm` is the official tool recommended for bootstrapping Kubernetes clusters.

## Prerequisites

Before starting, ensure you have:
- At least two Linux machines (Ubuntu/Debian or CentOS/RHEL).
- A static IP address for the master node.
- Swap disabled on all nodes (`sudo swapoff -a`).
- Container runtime installed (e.g., containerd or Docker).

## 1. Install Kubernetes Components

On all nodes, you must install `kubeadm`, `kubelet`, and `kubectl`.

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## 2. Initialize the Control Plane

On the master node, initialize the cluster:

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

This command will output a `kubeadm join` command. Save it securely; you will need it to add worker nodes.

Configure `kubectl` for your regular user:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 3. Install a Pod Network Add-on

To allow pods to communicate, you must deploy a Container Network Interface (CNI). We will use Flannel:

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

## 4. Join Worker Nodes

On each worker node, run the `kubeadm join` command you saved earlier. It will look something like this:

```bash
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Verify your cluster is running properly by executing `kubectl get nodes` on the master node. All nodes should eventually transition to the `Ready` status.

