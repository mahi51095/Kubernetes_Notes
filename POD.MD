
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

# 📘 Assignment

Please create the following Pods using YAML:

- ✅ `tomcat`
- ✅ `jenkins`
- ✅ `mongo`

---
