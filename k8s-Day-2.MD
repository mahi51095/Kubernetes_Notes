# Kubernetes Cluster Setup using Kubeadm (One Master, Two Worker Nodes)

This document provides a complete guide for setting up a multi-node Kubernetes cluster using `kubeadm` with one master and two worker nodes. Each section answers key questions — **What**, **Why**, **When**, and **How** — and includes **real-world metaphors** and **enterprise-level insights** to deepen your practical understanding.

---

## 📌 Prerequisites

### ❓ What & Why?
Before you start, all nodes (1 master + 2 workers) must meet these system requirements:

- At least 2 CPUs and 2 GB RAM
- Internet access to install packages
- Ubuntu (or other supported Linux distro)
- Root or sudo access

🛠️ **Real World:** Think of these as your wedding venue requirements — electricity, water, chairs, lighting — all must be ready before guests (pods) arrive.

🏢 **Enterprise Note:** In production, each node usually has 4+ CPUs, 16+ GB RAM, and is deployed on private subnets for internal communication.

---

## 🧩 Step 1: Set Unique Hostnames on Each Node

### ❓ Why?
Kubernetes uses hostnames to distinguish nodes, like names on ID cards.

```bash
# Master Node
sudo hostnamectl set-hostname master

# Worker Node 1
sudo hostnamectl set-hostname worker1

# Worker Node 2
sudo hostnamectl set-hostname worker2
```

🎯 **Metaphor:** Like naming each branch office of a company — Master HQ, Worker Branch 1, Worker Branch 2.

---

## 🔗 Step 2: Add IP-to-Hostname Mappings

### ❓ Why?
So that all nodes can recognize and connect with each other by name.

```bash
sudo nano /etc/hosts
```

Add:
```bash
<Master-IP>   master
<Worker1-IP>  worker1
<Worker2-IP>  worker2
```

🚚 **Real World:** Like saving phone contacts — easier to dial “mom” than her 10-digit number.

---

## 🔒 Step 3: Disable Swap Memory

### ❓ Why?
Kubernetes nodes expect stable and predictable memory. Swap breaks that.

```bash
sudo swapoff -a
sudo nano /etc/fstab  # comment the swap line to disable on reboot
```

🧠 **Analogy:** It's like asking your team to avoid daydreaming (swapping) during a high-stakes project. Stay focused.

---

## 📶 Step 4: Load Required Kernel Modules

```bash
sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF
```

📡 **Why?** These modules enable packet routing and overlay networking, like walkie-talkies between team members.

🏭 **Enterprise:** Firewalls and network policies in production must allow these modules for network flows to work correctly.

---

## 🔧 Step 5: Set Kernel Networking Parameters

```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

🚦 **Real World:** You're setting traffic signals to green at all intersections between pods.

---

## 📦 Step 6: Install and Configure containerd

```bash
sudo apt update && sudo apt install -y containerd

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```

🧊 **Metaphor:** containerd is the chef who cooks containers. Kubernetes just tells him what dish to prepare.

🏭 **Enterprise:** containerd is chosen for simplicity, performance, and compatibility with systemd.

---

## 📥 Step 7: Add Kubernetes APT Repository

```bash
sudo apt update && sudo apt install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

📦 **Why?** This is like adding the official bakery address to your delivery app to receive fresh bread (packages).

---

## ⚙️ Step 8: Install kubelet, kubeadm, and kubectl

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

🛠️ **Tools Comparison:**
- `kubeadm` = Event planner
- `kubelet` = Event staff
- `kubectl` = You, the coordinator, giving instructions

---

## 🧠 Step 9: Initialize the Master Node

```bash
sudo kubeadm init --control-plane-endpoint=master --pod-network-cidr=192.168.0.0/16
```

🔑 **Why?** This command sets up the brain and gives out VIP entry passes (join tokens).

🏭 **Enterprise:** Always configure a static control plane endpoint or load balancer if planning for HA.

---

## 📁 Step 10: Configure kubectl on the Master

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

📳 **Why?** This connects your `kubectl` phone to the API server hotline.

---

## 🌐 Step 11: Install Pod Network Add-on (Calico)

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```

📡 **Why?** Calico sets up the roads (routes) so cars (pods) can talk to each other.

🏭 **Enterprise:** Calico supports network policies for fine-grained security between services.

---

## 🤝 Step 12: Join Worker Nodes

From the master:
```bash
kubeadm token create --print-join-command
```

Run on each worker:
```bash
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

🎟️ **Real World:** Like scanning a QR code to join a Zoom meeting.

---

## 📊 Final Check

```bash
kubectl get nodes
```
Expected output:
```
NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   Xm    vX.X.X
worker1   Ready    <none>          Xm    vX.X.X
worker2   Ready    <none>          Xm    vX.X.X
```

📈 **Why?** Verifies all your team members are present and active.

---

## 🧾 Key Terms

- **Control Plane**: The CEO + Managers
- **Worker Node**: Employees running day-to-day tasks
- **Pod Network**: Internal office LAN
- **containerd**: Kitchen in which app containers are cooked

---

## 🎯 Summary

You've now set up a Kubernetes cluster with one master and two worker nodes using `kubeadm`. Each step had clear reasoning, enterprise insight, and metaphors to help you retain the knowledge.

Want to go further?
- Set up Kubernetes Dashboard (GUI)
- Add storage provisioning with NFS
- Explore high availability (multi-master)
- Practice deploying real apps like Nginx, WordPress, or a CI/CD tool like Jenkins

Let me know — I’m here to help 🚀
