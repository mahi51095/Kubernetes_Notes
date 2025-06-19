<!-- Static Pods in Kubernetes -->
# 📦 Static Pods in Kubernetes

> A deep dive into one of the most foundational — yet less known — Kubernetes concepts.

---

## 📚 Table of Contents
- [📌 What is a Static Pod?](#-what-is-a-static-pod)
- [✨ Key Characteristics](#-key-characteristics)
- [🧰 Use Cases](#-use-cases)
- [⚙️ How Static Pods Work](#-how-static-pods-work)
- [🧪 Example: Deploying Nginx as a Static Pod](#-example-deploying-nginx-as-a-static-pod)
- [📝 Key Points to Remember](#-key-points-to-remember)
- [✅ Advantages](#-advantages)
- [⚠️ Disadvantages](#-disadvantages)

---

## 📌 What is a Static Pod?

A **Static Pod** in Kubernetes is a pod directly managed by the **Kubelet** on a node — not through the Kubernetes control plane (API server and scheduler).

- 🗂️ Defined using plain YAML or JSON files
- 📂 Stored in a specific folder on the node
- 🔁 Automatically managed and restarted by Kubelet

---

## ✨ Key Characteristics

### 🔸 Not Managed by Kubernetes Scheduler
> Static pods skip the scheduler queue. They're created by the Kubelet directly on the node.

### 🔸 Node-Specific
> Tied to a particular node because the configuration file resides on that node.

### 🔸 Self-Healing
> The Kubelet will restart them if they crash, as long as the manifest file is present.

### 🔸 Invisible to the API Server
> Static pods won’t appear in `kubectl get pods` by default.

- Check via:  
  ```bash
  kubectl describe node <node-name>
  ```

---

## 🧰 Use Cases

| 🏷️ Use Case | 🔍 Description |
|-------------|----------------|
| **Node-specific agents** | Logging, monitoring tools like Fluentd or Prometheus Node Exporter |
| **Kubernetes Core Components** | Like `kube-apiserver`, `controller-manager`, and `scheduler` |
| **Single-node apps** | Daemons that must run only on one machine |

---

## ⚙️ How Static Pods Work

1. **Manifest File Created**  
   Create a YAML/JSON pod definition.

2. **Placed in Watch Directory**  
   Default: `/etc/kubernetes/manifests/`

3. **Kubelet Watches and Manages**  
   - Detects the file
   - Starts the pod
   - Restarts on failure
   - Deletes pod if the file is removed

---

## 🧪 Example: Deploying Nginx as a Static Pod

### 🔹 Step 1: Create a YAML File
`nginx-static-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### 🔹 Step 2: Move File to Watch Directory
```bash
sudo mv nginx-static-pod.yaml /etc/kubernetes/manifests/
```

### 🔹 Step 3: Verify Pod Status
```bash
kubectl get pods --all-namespaces
kubectl describe node <node-name>
```

### 🔹 Step 4: Observe Lifecycle Behavior
- Remove the file → Pod is removed
- Pod crashes → Kubelet restarts it

---

## 📝 Key Points to Remember

- ❌ Not scheduled or controlled by API server  
- 🧠 Best for node-local workloads  
- 🛠️ Ideal for essential infrastructure services  
- 📁 Lives in a specific manifest directory

---

## ✅ Advantages

- 🔄 **Self-Healing**  
  Automatically restarted by Kubelet if it fails

- 📍 **Node-specific Deployment**  
  Useful for per-node agents or local system services

- 🧾 **Simple Configuration**  
  Just drop a YAML file in a folder — no complex resources

---

## ⚠️ Disadvantages

- 🚫 **No High-Level Scheduling**  
  Can't use features like Deployments, ReplicaSets, etc.

- 📉 **Not Scalable**  
  Managing files across many nodes is tough

- ❓ **Harder to Monitor**  
  Visibility and control are limited via `kubectl`

---

> 💡 **Pro Tip**: Use **DaemonSets** if you want Kubernetes-native management of per-node pods with API server integration.

