# ⚙️ Requests and Limits in Kubernetes

### 🎭 Real-time Metaphor

> Imagine you are organizing a **marriage buffet**.
>
> * Each food stall (Pod) asks for a **minimum space and electricity** to function well. (Request)
> * But to prevent overuse or monopoly of resources, you **set a maximum limit** for each stall. (Limit)
>
> ⚡ This avoids some stalls taking all the electricity or space and affecting others.

---

## ✅ 5W1H: Requests and Limits

### 1️⃣ What are Requests and Limits?

* **Request**: Minimum amount of CPU/Memory guaranteed to a container
* **Limit**: Maximum amount the container is allowed to use

These are **resource management rules** inside Kubernetes.

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

---

### 2️⃣ Why do we use Requests and Limits?

* To ensure **fair distribution** of cluster resources
* To **avoid resource hogging** by one container
* To **prevent crashes** due to memory starvation
* To help scheduler place Pods accurately

---

### 3️⃣ When should you define them?

* In **production workloads** where stability matters
* When multiple teams/applications share a cluster
* When you want to **predict cluster capacity**

---

### 4️⃣ Where are they used?

* In the **`spec.containers`** section of any Pod/Deployment YAML
* Applied at container-level, not whole Pod

---

### 5️⃣ Who benefits from it?

* **Cluster Admins**: manage resources
* **SRE/DevOps**: maintain stability
* **Developers**: get predictable performance

---

### 6️⃣ How do they work?

* **Request** is used by the scheduler to decide which node has enough capacity
* **Limit** is enforced during runtime

  * If memory crosses limit → Pod is killed (OOMKilled)
  * If CPU crosses limit → Throttled (not killed)

---

## 📈 Real-time Example

You run 3 backend microservices on your cluster. One service has a memory leak. Without limits, it **consumes all memory**, crashes the node, and brings down other apps.

✅ Solution: Apply requests and limits:

```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "300m"
  limits:
    memory: "1Gi"
    cpu: "700m"
```

---

## 🧠 Summary Table

| Resource Type | Acts As           | Purpose                         | Enforced At     | Example  |
| ------------- | ----------------- | ------------------------------- | --------------- | -------- |
| Request       | Minimum Guarantee | Scheduler uses it for placement | Scheduling Time | 250m CPU |
| Limit         | Upper Bound       | Prevents overuse of resources   | Runtime         | 500m CPU |

Would you like a YAML with auto-scaling based on requests/limits too?
