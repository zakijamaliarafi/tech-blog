---
heroImage: '/kubernetes-cluster-setup-kubeadm.svg'
title: 'Kubernetes Cluster Setup Using Kubeadm'
description: 'A comprehensive guide to bootstrapping a production-ready Kubernetes cluster from scratch using kubeadm.'
pubDate: 'May 10 2026'
---

In the modern era of cloud-native infrastructure, Kubernetes has cemented its position as the undisputed king of container orchestration. However, the vast majority of developers and organizations interact with Kubernetes through managed cloud services like Amazon EKS, Google GKE, or Azure AKS. While these managed services are incredibly convenient, abstracting away the complex control plane, they also obscure the fundamental mechanics of how Kubernetes actually operates.

To truly understand the architecture of Kubernetes—the intricate dance between the API server, the scheduler, the kubelet, and the underlying container runtime—there is no better exercise than building a cluster entirely from scratch on bare-metal servers or unmanaged virtual machines. 

The official, community-supported tool for bootstrapping a Kubernetes cluster is **`kubeadm`**. 

`kubeadm` is designed to be the lowest common denominator for cluster creation. It handles the agonizingly complex cryptographic work—generating Certificate Authorities (CAs), signing certificates for the API server, and generating secure bootstrap tokens—while leaving the higher-level provisioning of infrastructure entirely up to you. 

This comprehensive guide will walk you through the precise steps required to construct a highly functional, multi-node Kubernetes cluster using `kubeadm` on Ubuntu Linux.

## Phase 1: Architectural Prerequisites and System Preparation

Before installing a single Kubernetes package, your underlying Linux nodes must be strictly configured to meet the stringent requirements of the Kubernetes engine. For this guide, assume we have two Ubuntu 22.04 virtual machines:
*   `k8s-master` (IP: 192.168.1.10)
*   `k8s-worker-1` (IP: 192.168.1.11)

**Crucial System Configurations (Execute on ALL Nodes):**

1.  **Disable Swap:** Kubernetes is highly aggressively protective of its memory management. If a Linux system begins swapping memory to disk, performance plummets. The `kubelet` will categorically refuse to start if swap is enabled.
    ```bash
    # Disable swap immediately
    sudo swapoff -a
    
    # Comment out the swap entry in fstab to ensure it remains disabled after reboot
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
    ```

2.  **Enable Kernel Modules for Networking:** Kubernetes requires specific networking configurations to route traffic efficiently between pods across different nodes.
    ```bash
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    
    sudo modprobe overlay
    sudo modprobe br_netfilter
    ```

3.  **Configure IP Tables (sysctl):** You must instruct the Linux kernel to allow iptables to see bridged traffic, which is essential for the Container Network Interface (CNI) to function.
    ```bash
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF
    
    # Apply the sysctl params without rebooting
    sudo sysctl --system
    ```

## Phase 2: Installing the Container Runtime (Containerd)

Kubernetes no longer uses Docker as its default container runtime (due to the deprecation of `dockershim`). Instead, the industry standard is to use **containerd**, a lightweight, robust daemon that manages the complete container lifecycle.

**Execute on ALL Nodes:**

1.  Install the `containerd` package from the standard Ubuntu repositories:
    ```bash
    sudo apt-get update
    sudo apt-get install -y containerd
    ```
2.  Generate the default configuration file for containerd:
    ```bash
    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    ```
3.  **Critical Step:** Kubernetes uses `systemd` to manage cgroups (control groups) for resource allocation. You must configure `containerd` to integrate with `systemd`. Open `/etc/containerd/config.toml` in your editor, find the `[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]` section, and change `SystemdCgroup = false` to `SystemdCgroup = true`.
4.  Restart the runtime:
    ```bash
    sudo systemctl restart containerd
    sudo systemctl enable containerd
    ```

## Phase 3: Installing the Kubernetes Core Components

Now that the OS and container runtime are primed, we can install the holy trinity of Kubernetes components.

**Execute on ALL Nodes:**

1.  Add the official Google Cloud Kubernetes GPG key and APT repository:
    ```bash
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl
    
    # Download the Google Cloud public signing key
    curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
    
    # Add the Kubernetes apt repository
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```
2.  Install the components:
    *   `kubeadm`: The command to bootstrap the cluster.
    *   `kubelet`: The primary "node agent" that runs on every machine and ensures containers are running in a Pod.
    *   `kubectl`: The command-line utility used to talk to the cluster API.
    ```bash
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    
    # Prevent these packages from being automatically updated by the OS, 
    # as Kubernetes upgrades require a specific, manual sequence.
    sudo apt-mark hold kubelet kubeadm kubectl
    ```

## Phase 4: Initializing the Control Plane (Master Node)

Everything up to this point was preparation. Now, we ignite the cluster. 

**Execute ONLY on the Master Node (`k8s-master`):**

Run the `kubeadm init` command. We must pass the `--pod-network-cidr` flag to tell Kubernetes what IP address range it should assign to the internal Pods. (We use `10.244.0.0/16` because it is the default requirement for the Flannel network add-on we will install later).

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

This command will take a few minutes. It is pulling the core control plane container images (etcd, kube-apiserver, kube-controller-manager, kube-scheduler) from the internet, generating certificates, and starting the services.

When it finishes, the terminal will output a massive success message. **Pay close attention to the end of this message.**

First, it will tell you how to configure your local user to use `kubectl`:
```bash
# Run these commands as your regular, non-root user on the master node
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Second, it will output the exact `kubeadm join` command required to attach worker nodes to this new master. Copy this command and save it in a notepad immediately.

## Phase 5: Deploying the Container Network Interface (CNI)

If you run `kubectl get nodes` on the master right now, you will see the master node is listed as `NotReady`. This is because Kubernetes relies on third-party plugins to handle networking between pods. Until a CNI is installed, the cluster is paralyzed.

We will deploy **Flannel**, a highly reliable and simple Layer 3 network fabric.

**Execute ONLY on the Master Node:**
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
Wait a few moments, and run `kubectl get nodes` again. The master node should now transition to the `Ready` state.

## Phase 6: Joining Worker Nodes to the Cluster

The control plane is operational. Now, we provide it with compute resources.

**Execute ONLY on the Worker Nodes (`k8s-worker-1`):**

Paste the `kubeadm join` command that you copied from the output of the master node's initialization phase. It will require `sudo` privileges.

```bash
sudo kubeadm join 192.168.1.10:6443 --token abcdef.1234567890abcdef \
        --discovery-token-ca-cert-hash sha256:abcd1234efgh5678ijkl9012mnop...
```

The worker node will authenticate with the master using the token, download the necessary certificates, and the local `kubelet` will begin communicating with the API server.

## Conclusion and Verification

Return to your **Master Node** and execute:
```bash
kubectl get nodes
```

You should see both `k8s-master` and `k8s-worker-1` listed, both displaying a `Ready` status. 

Congratulations. You have successfully bootstrapped a distributed, bare-metal Kubernetes cluster. You have navigated the complexities of container runtimes, system-level networking, cryptographic initialization, and CNI deployment. From this solid foundation, you are now ready to begin deploying highly available deployments, managing persistent storage volumes, and mastering the true power of cloud-native orchestration.
