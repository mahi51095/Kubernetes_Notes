# 🧪 Health Checks in Kubernetes: Liveness & Readiness Probes

### 🏨 Real-World Metaphor: Hotel Room and Guest Health Checks

Imagine a **Pod** as a **hotel room**, and the **container(s)** inside it as **guests**.

* **Liveness probe** = Is the guest still alive inside the room? Should hotel management knock or call emergency?
* **Readiness probe** = Is the guest ready to receive room service or visitors?

---

## 📌 What is a Liveness Probe?

* **Liveness probe checks if the container is alive**.
* If the probe fails, K8s will **restart the container**.
* It's like a hotel receptionist checking if the guest in the room is unresponsive. If they are, security intervenes and restarts the stay (restarts the container).

## 📌 What is a Readiness Probe?

* **Readiness probe checks if the container is ready to serve traffic**.
* If the probe fails, the Pod is **temporarily removed from the service endpoint**.
* It's like checking if the guest has finished settling in, is dressed, and ready for room service or guests. If not, hotel staff wait before interacting.

---

## 🎯 5W1H Breakdown

### ✅ **What?**

* Liveness: Checks container's survival
* Readiness: Checks container's availability to serve

### ✅ **Why?**

* Avoid sending traffic to apps that are booting or unhealthy.
* Prevent unnecessary downtimes with automatic restarts.

### ✅ **When?**

* Liveness: Used **after** app is up to monitor crash/recovery.
* Readiness: Used **during** app startup or after config reloads.

### ✅ **Where?**

* Defined in Pod specs (usually at the container level)

### ✅ **Who?**

* Kubernetes **Kubelet agent** performs the checks regularly.

### ✅ **How?**

* Using HTTP/HTTPS endpoints
* TCP port checks
* Exec command (e.g., checking if a process/file exists)

---

## 🧪 Liveness & Readiness Probe Example

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 3
```

### 📝 Parameters

* `initialDelaySeconds`: Time to wait before starting the first probe
* `periodSeconds`: Frequency of checks
* `timeoutSeconds`: Max time for a probe to return
* `failureThreshold`: How many failures before action is taken
* `successThreshold`: Minimum consecutive successes for readiness

---

## 📦 Sample Pod with Probes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-demo
spec:
  containers:
  - name: myapp
    image: myapp:latest
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 3
```

---

## 🔍 When Things Go Wrong

| Scenario                | Likely Issue    | K8s Response               |
| ----------------------- | --------------- | -------------------------- |
| App crashes silently    | Liveness fails  | Container restarted        |
| App is slow to boot     | Readiness fails | Traffic not routed yet     |
| App can't connect to DB | Readiness fails | Wait until DB is available |

---

## ✅ Realistic Java Web App Example

### 🧾 Deployment with Liveness & Readiness Probes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: javawebappdep
  namespace: prod
spec:
  replicas: 2
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: javawebapp
  template:
    metadata:
      name: javawebapp
      labels:
        app: javawebapp
    spec:
      containers:
      - name: javawebapp
        image: kkeducation123456/maven-web-app:1.2
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health  # Health check endpoint
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health  # Readiness endpoint
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 5
          failureThreshold: 3
```

### 📡 Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
  namespace: prod
spec:
  type: NodePort
  selector:
    app: javawebapp
  ports:
  - port: 80
    targetPort: 8080
```

Let me know if you'd like to add a startupProbe example or troubleshoot failing probes!
