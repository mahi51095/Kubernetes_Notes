
# 🌐 Kubernetes Cluster Setup Guide

---

## 🧱 Kubernetes Cluster Types

### 1. Self-Managed Kubernetes Cluster

- **Minikube** – Single-node Kubernetes cluster (ideal for local testing).
- **Kubeadm** – Multi-node Kubernetes cluster.

> 💡 **Note:**  
If a **Pod fails**, Kubernetes will handle it.  
If a **Node fails**, **manual intervention** is required.

---

### 2. Cloud-Managed Kubernetes Cluster

- **AWS** – EKS (Elastic Kubernetes Service)
- **Azure** – AKS (Azure Kubernetes Service)
- **Google Cloud** – GKE (Google Kubernetes Engine)

> 💡 **Note:**  
Pod failures are managed by Kubernetes.  
Node failures are handled by the **Cloud Provider**.

---

## 🖥️ Cluster Architecture Example

- 1 Master Node
- 2 Worker Nodes

---

# 📌 Agenda: Kubernetes Setup Using Kubeadm in AWS EC2 with containerd

---

## 🔧 Prerequisites

- **3 Ubuntu Servers**
  - **1 Master Node** – 4GB RAM, 2 vCPU (`t2.medium`)
  - **2 Worker Nodes** – 1GB RAM, 1 vCPU (`t2.micro`)
- Open all traffic in AWS Security Group (for testing only)

---

# ⚙️ Common Setup (Master & Workers)

### 👤 Step 1: Switch to root

```bash
sudo su -
```

---

### 🚫 Step 2: Disable Swap

```bash
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#/g' /etc/fstab
```

---

### 🌉 Step 3: Load Kernel Modules & Sysctl Settings

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```

---

### 📦 Step 4: Install containerd

```bash
apt-get update -y 
apt-get install -y ca-certificates curl gnupg lsb-release

mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt-get update -y
apt-get install -y containerd.io

containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

systemctl restart containerd
systemctl enable containerd
```

---

### 🧰 Step 5: Install Kubernetes Tools

```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet
```

---

# 👑 Master Node Configuration

### 🔧 Initialize Kubernetes

```bash
sudo su -
kubeadm init --cri-socket /run/containerd/containerd.sock
```

### 📁 Configure kubectl

```bash
exit
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 📡 Apply CNI Plugin (Choose one)

#### Option 1: Weave Net

```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

#### Option 2: Calico

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

### 🧩 Verify Cluster

```bash
kubectl get nodes
kubectl get pods -o wide -n kube-system
```

---

### 🔑 Get Join Token for Workers

```bash
kubeadm token create --print-join-command
```

---

# 👷 Worker Node: Join Cluster

Paste the token command from Master node.

---

# 🧪 Sample App Deployment

```bash
kubectl run nginx-demo --image=nginx --port=80
kubectl expose pod nginx-demo --port=80 --target-port=80 --type=NodePort --namespace=prod
kubectl get services
```

---

# 📚 Kubernetes Resources (API Objects)

- Pod
- ReplicaSet / ReplicationController
- DaemonSet
- Deployment
- StatefulSet
- Namespace
- ConfigMap / Secret
- Service / Ingress / NetworkPolicy
- PersistentVolumeClaim / PersistentVolume

---

## 🧾 Useful kubectl Commands

```bash
kubectl api-resources
kubectl get nodes
kubectl get pods --all-namespaces
```

---

# 🧩 Kubernetes Namespaces

Namespaces allow logical partitioning in a cluster.

### 🔧 Create Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    name: my-namespace
```

```bash
kubectl apply -f namespace.yaml
kubectl create namespace my-namespace
```

### 📋 List and Describe

```bash
kubectl get namespaces
kubectl describe ns my-namespace
```

### ❌ Delete Namespace

```bash
kubectl delete namespace my-namespace
```

### 🛠️ Set Namespace Context

```bash
kubectl config set-context --current --namespace=prod
```

---

## 📊 Namespace Resource Quotas

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
  namespace: prod
spec:
  hard:
    pods: "10"
```

```bash
kubectl apply -f resource-quota.yaml
```

---

> 🧠 **Tip:** Always test configurations on a development cluster before moving to production!

---

### 🙋 Support  
**KK FUNDA**  
📞 863980177 / 9676831734
