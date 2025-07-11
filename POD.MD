
# 📦 What is a Pod?

- A **Pod** is the smallest building block or **basic unit of scheduling** in Kubernetes.
- A Pod always **runs on a Node**.
- In a Kubernetes cluster, a pod represents a **running process**.
- Inside a pod, you can have **one or more containers**. All containers:
  - Share a **unique network IP**, storage, and other configurations.
- A **Pod is a group of containers** that are co-located and co-scheduled.
- Each Pod has its own **unique IP address** within the cluster.

---

# 🔄 Pod Lifecycle

1. ➡️ **Make a Pod request** to the API server using a YAML definition.
2. 📦 API server stores the pod definition in **ETCD** (key-value store).
3. 🧭 **Scheduler** finds the unscheduled pod and assigns it to a Node.
4. ⚙️ **Kubelet** on that node pulls the image via **container-runtime** and starts the container.
5. 📚 Entire **lifecycle state** is stored in **ETCD**.

---

# 🧠 Pod Concepts

- A Pod is **ephemeral** – once terminated, it won’t be rescheduled automatically.
- Pods should **not** be used directly in production. Use **controllers** like:
  - `ReplicaSet`, `Deployment`, `DaemonSet`, etc., to maintain pod health.

## Types of Pod Models

### 🧍 One-container-per-Pod
- Most common model.
- Pod is just a **wrapper for a single container**.
- Kubernetes manages Pods rather than containers directly.

### 🤝 Multi-container Pods (Sidecar Pattern)
- A pod has **multiple co-located containers**:
  - One **primary container**
  - One or more **utility containers** (e.g., log shippers, monitoring agents)

---

# 📋 Listing Kubernetes Resources

| Command | Description |
|--------|-------------|
| `kubectl api-resources` | Lists all Kubernetes resources |
| `kubectl api-resources --namespaced=true` | Lists **namespace-level** objects |
| `kubectl api-resources --namespaced=false` | Lists **cluster-level** objects |

---

# 📑 Pod YAML Syntax

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: <pod-name>
  labels: 
    <key>: <value>
  namespace: <namespace-name>
spec:
  containers:
    - name: <container-name>
      image: <registry/repo:tag>
      ports:
        - containerPort: <port>
      env:
        - name: <ENV_NAME>
          value: <value>
```

✅ Save this as: `app1.yaml`  
🚀 Apply using: `kubectl apply -f app1.yaml`

---

# ✨ Pod YAML Examples

### ✅ Example 1: Simple NGINX Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: test-ns
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

---

### ☕ Example 2: Java Web App Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: javawebappTest
  labels:
    app: javawebappTest
  namespace: test-ns
spec:
  containers:
    - name: javawebapp1
      image: kkeducation123456/maven-web-app:1.2
      ports:
        - containerPort: 8080
```

---

### 🗄️ Example 3: MongoDB Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mogodbpod
  namespace: db-ns
spec:
  containers:
    - name: mongodbcontainer
      image: mongo
      ports:
        - containerPort: 27017
```

---

# 🔍 Useful Pod Commands

| Command | Purpose |
|--------|---------|
| `kubectl describe po <pod-name> -n <namespace>` | View full pod info |
| `kubectl apply -f abc.yaml -v=8` | Apply pod definition with **debug (sanity)** mode |
| `kubectl get po -o wide` | Check the **node** where pod is running |

---

# ℹ️ Pod Status Meaning

| Status | Meaning |
|--------|---------|
| `1/1` | One container is inside pod and it's **Running** |
| `0/1` | One container is inside pod and it's **Not Running** |
| `2/2` | Two containers running successfully |
| `1/2` | One container running, one is not |

---


# Kubernetes Pod Lifecycle

Understanding how a Pod behaves from creation to termination helps in troubleshooting and effective cluster management.

## 🚦 Pod Lifecycle Stages

```text
Pending   -->   Scheduled   -->   Running   -->   Succeeded / Failed / CrashLoopBackOff
```

### 1. 🟡 Pending
- Pod is created but not yet scheduled to any node.
- Reasons: Image pull pending, node not available, resource constraints.

### 2. 🔵 Scheduled
- Kubernetes Scheduler assigns the Pod to a suitable node.
- Node selection based on CPU, memory, affinity rules, taints, etc.

### 3. 🟢 Running
- Pod is successfully started on the node.
- All containers in the pod are up and running.

### 4. ✅ Succeeded
- All containers in the Pod have completed successfully and will not restart.
- Usually for short-lived batch jobs.

### 5. ❌ Failed
- One or more containers exited with an error.
- Pod won’t restart by itself unless managed by a controller.

### 6. 🔁 CrashLoopBackOff
- Container starts, crashes, and Kubernetes retries it repeatedly.
- Often caused by misconfiguration, failed dependencies, etc.

## 🔄 Summary Table

| Phase               | Description                                          |
|---------------------|------------------------------------------------------|
| Pending             | Awaiting scheduling or image pull                    |
| Scheduled           | Assigned to a node                                   |
| Running             | All containers are active                            |
| Succeeded           | All containers completed successfully                |
| Failed              | Containers exited with errors                        |
| CrashLoopBackOff    | Container keeps crashing; being restarted repeatedly |

---

### 🔍 Useful Commands

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 📦 Example Pod YAML with Explanation

```yaml
apiVersion: v1                      # Kubernetes API version
kind: Pod                           # Resource type is Pod
metadata:
  name: nginx-pod                   # Name of the pod
  labels:
    app: web                        # Label to identify pod group
spec:
  containers:                       # List of containers in the pod
    - name: nginx                   # Name of the container
      image: nginx:1.21             # Docker image to use
      ports:
        - containerPort: 80         # Port exposed inside the container
```

