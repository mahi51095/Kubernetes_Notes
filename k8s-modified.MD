
# 🌐 Kubernetes Cluster Setup Guide  
*A DevOps‑oriented annotated version*  

---

## 🧱 Kubernetes Cluster Types  

### 1. Self‑Managed Kubernetes Cluster  

| Item | **Definition** | **DevOps ➜ What & Why** |
|------|----------------|--------------------------|
| **Minikube** | A single‑node Kubernetes distribution that runs locally inside a VM or container. | • Fast spin‑up for POCs and demos. <br>• Lets DevOps engineers prototype manifests exactly as they would run in production—no “it‑works‑on‑my‑laptop” surprises. |
| **Kubeadm** | The CNCF‑supported CLI tool that bootstraps a multi‑node cluster by generating certificates, kube‑configs, and control‑plane manifests. | • Declarative, version‑consistent cluster creation (immutable infra). <br>• Guarantees repeatability when rebuilding or scaling clusters in lab or edge environments. |

> 💡 **Self‑managed:** *You* own the control plane. Great for fine‑grained control and air‑gapped or cost‑conscious setups, but you must patch, upgrade, and back up etcd yourself.

---

### 2. Cloud‑Managed Kubernetes Cluster  

| Cloud | **Managed Service** | **Definition & DevOps Benefits** |
|-------|--------------------|----------------------------------|
| AWS   | **EKS** | Fully managed control plane; IAM‑integrated RBAC and built‑in scalability tooling (Karpenter/Fargate). DevOps teams focus on app delivery, not cluster care. |
| Azure | **AKS** | Azure AD authentication, automated patching, plus built‑in CI/CD hooks via Azure DevOps and GitHub Actions. Simplifies governance for enterprises already on Microsoft stack. |
| GCP   | **GKE** | Google‑hardened master, auto‑upgrades, and binary authorization. Popular for SRE teams adopting Google’s same production practices. |

> 💡 **Managed:** Cloud provider monitors the nodes, auto‑heals them, and rolls out zero‑downtime upgrades—reducing toil and on‑call pages.

---

## 🖥️ Cluster Architecture Example  

| Role | **Definition** | **Why DevOps Cares** |
|------|----------------|----------------------|
| **Master / Control Plane** | Runs API server, scheduler, controller‑manager, etcd. | Central point for GitOps pipelines—everything declared here is the single source of truth. |
| **Worker Node** | Hosts application Pods and kubelet. | Scaling unit for horizontal growth; observability and resource quotas are applied per‑node. |

Example layout: **1 master + 2 workers**—small yet production‑realistic lab for chaos‑testing and upgrade rehearsals.

---

# 📌 Agenda: *Kubeadm* ‑‑> *AWS EC2* ‑‑> *containerd*  

The steps below are annotated with the *what* (definition) and *why* (DevOps rationale).

---

## 🔧 Prerequisites  

| Resource | **Definition** | **Why (DevOps)** |
|----------|----------------|------------------|
| **Ubuntu 20.04/22.04** | LTS OS, predictable kernel & systemd. | Long‑term support means fewer CVEs mid‑project. |
| **Master t2.medium** | 4 GB RAM, 2 vCPU. | Control‑plane components are memory‑sensitive; under‑provisioning = etcd crashes. |
| **Worker t2.micro** | 1 GB RAM, 1 vCPU (lab). | Mirrors low‑spec edge nodes; validates resource requests/limits. |
| **Open All Ports (lab)** | 0.0.0.0/0 in SG. | *Only* for throw‑away demos—ease of setup outweighs security. In prod you’d lock this down with least‑privilege rules and private subnets. |

---

# ⚙️ Common Setup (Master & Workers)  

| Step | Snippet | **What / Why** |
|------|---------|----------------|
| 1️⃣ `sudo su -` | Become root. | Reduces “permission denied” friction during automation scripts. |
| 2️⃣ Disable swap | `swapoff ‑a` | K8s scheduler assumes no swapping; prevents unpredictable latency. |
| 3️⃣ Kernel tuning | Enable `br_netfilter`, `ip_forward`. | Lets kube‑proxy & CNI manipulate iptables for Pod‑to‑Pod traffic. |
| 4️⃣ Install **containerd** | OCI‑compliant CRI runtime. | Lightweight vs. Docker Engine; endorsed by Kubernetes v1.24+. |
| 5️⃣ Install **kubelet, kubeadm, kubectl** | Core binaries pinned with `apt-mark hold`. | Version skew policy: nodes ±1 minor from control plane; pinning avoids accidental upgrades that might break compatibility. |

---

# 👑 Master Node Tasks  

| Task | **Definition** | **Why (DevOps)** |
|------|----------------|------------------|
| `kubeadm init` | Generates certificates & manifests under `/etc/kubernetes`. | Automates TLS, RBAC, and etcd bootstrap—error‑prone if done manually. |
| Configure `kubectl` | Copy admin kube‑config to `$HOME/.kube/config`. | Enables day‑one GitOps and CI/CD pipelines that use `kubectl apply`. |
| **CNI Plugin** | Weave Net / Calico. | Provides overlay networking + NetworkPolicy enforcement; choice depends on feature set (Calico for eBPF, Weave for simplicity). |
| Cluster verify | `kubectl get nodes/pods`. | Smoke test for observers and alerting baselines. |
| Join‑token | Printed with `kubeadm token create`. | Idempotent automation—can be fed into Terraform/Ansible worker modules. |

---

# 👷 Worker Node Tasks  

Run the join command → registers kubelet with API server, pulls Pod CIDR settings. **Why it matters:** Brings elasticity—new capacity can be auto‑provisioned by autoscaler events.

---

# 🧪 Sample App Deployment  

| Command | **What happens** | **Why (DevOps)** |
|---------|------------------|------------------|
| `kubectl run nginx-demo` | Creates a Pod with `nginx` image. | Quick sanity check that the container runtime, pull secrets, and DNS work. |
| `kubectl expose pod … --type=NodePort` | Service object gets a port on each node. | Mimics basic north‑south traffic without needing an Ingress controller. |

---

# 📚 Kubernetes Resources (API Objects)  

| Kind | **Definition** | **DevOps Role** |
|------|----------------|-----------------|
| **Pod** | Smallest deployable unit—1 or more containers. | Building block for higher abstractions (Deployments, Jobs). |
| **ReplicaSet** | Ensures a specified number of identical Pods. | Auto‑healing & scaling driver behind Deployments. |
| **DaemonSet** | Guarantees 1 Pod per node (e.g., log collectors). | Cluster‑wide observability & security agents. |
| **Deployment** | Declarative updates for ReplicaSets. | Enables canary/blue‑green via `kubectl rollout`. |
| **StatefulSet** | Ordered, sticky identifiers for Pods + PVCs. | Databases, queues—stable network IDs. |
| **Namespace** | Virtual cluster inside a cluster. | Multi‑tenancy, RBAC boundaries, cost allocation. |
| **ConfigMap / Secret** | Externalize config and credentials. | Twelve‑factor compliance & secret rotation. |
| **Service / Ingress** | Stable access endpoints + L7 routing. | Decouples service discovery from Pod lifecycles. |
| **PersistentVolumeClaim** | User request for storage with specific QoS. | Abstracts away underlying EBS, Ceph, NFS, etc. |

---

## 🧾 Handy `kubectl` Cheat‑sheet  

```bash
kubectl api-resources          # discover all Kinds
kubectl get nodes -o wide      # show OS/arch & taints
kubectl get pods -A -w         # watch all namespaces in real‑time
```

> 🧠 **Tip:** Add `--context`, `--namespace`, and `--selector` flags to target GitOps pipelines precisely.

---

# 🧩 Namespaces Deep‑Dive  

| Aspect | **Definition** | **Why (DevOps)** |
|--------|----------------|------------------|
| Isolation | Soft multi‑tenancy boundary for names, quotas, and RBAC. | Prevents “noisy neighbor” resource exhaustion and simplifies cost‑showback. |
| ResourceQuota | Limits CPU, memory, objects per NS. | Enforces SLOs and cost budgets automatically. |
| Context Switching | `kubectl config set-context …` | Avoids the classic “oops I deployed to prod” problem in CI jobs. |

Example **Namespace manifest** and **ResourceQuota** shown above—ideal for IaC repos.

---

## 🏁 Wrapping Up  

- **Self‑managed vs. Cloud‑managed**: choose based on control vs. operational overhead.  
- **containerd + kubeadm**: minimalistic, CNCF‑aligned stack—perfect for repeatable DevOps pipelines.  
- **Namespaces, RBAC, Quotas**: invisible heroes that turn “a bunch of VMs” into a multi‑tenant PaaS.  

> 🚀 **Next steps:**  
> 1. Script these commands with Ansible or Terraform.  
> 2. Add GitOps via Argo CD or Flux.  
> 3. Integrate CI triggers (e.g., GitHub Actions → `kubectl apply`).  

Happy shipping! ✨

---

### 🙋 Support  
**KK FUNDA**  
📞 863980177 / 9676831734
