# Kubernetes Deployment and Strategies Explained

## ✨ What is a Deployment in Kubernetes?

**Definition**:
A **Deployment** is a Kubernetes object that helps you **manage, update, and scale your application** (Pods) automatically and safely.

It tells **Kubernetes**:

* How many copies (replicas) of your app to run
* What version/image of your app to run
* How to roll out or roll back changes

### 🎭 Real-time Metaphor

> Imagine you’re running a **food delivery service (like Zomato)** with multiple delivery agents. You want to:
>
> * Make sure **5 agents are always available** (Replicas)
> * If there's a new **app version**, update it **without stopping delivery** (Rolling update)
> * If the new update causes problems, **go back to the older app** (Rollback)

**Deployment = Manager**
who ensures the team (Pods) is always there, up-to-date, and ready to serve without downtime.

---

## ✅ 5W1H: Deployment in Kubernetes

### 1️⃣ What is a Deployment?

A controller that:

* Runs and maintains multiple copies of your app (Pods)
* Ensures high availability
* Manages updates and rollbacks

**File name**: `deployment.yaml`
**Controlled by**: `kubectl apply -f deployment.yaml`

### 2️⃣ Why do we use Deployment?

* To **scale** apps easily (up/down)
* To **update** app versions without downtime
* To **automate** healing of crashed Pods
* To **rollback** quickly if an update fails

**Goal**: Zero downtime deployment

### 3️⃣ When should you use Deployment?

* When you want **replicas** of the app for high availability
* When you need **safe, gradual updates**
* When you want to avoid manual Pod management

### 4️⃣ Where is Deployment used?

In any **production-ready environment**, such as:

* E-commerce websites
* Banking applications
* Food delivery platforms
* Microservice apps

### 5️⃣ Who uses Deployment?

* **DevOps Engineers**: for CI/CD
* **SRE Teams**: to ensure reliability
* **Developers**: to test new app versions

### 6️⃣ How is a Deployment created and used?

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1
        ports:
        - containerPort: 80
```

Apply using:

```bash
kubectl apply -f deployment.yaml
```

---

## 🌟 Deployment Strategies in Kubernetes

### 1️⃣ Rolling Update (Default)

🚚 **Metaphor**: Changing your **delivery agents one by one** with new training without stopping service.

* Gradually replaces old Pods with new ones.
* Ensures some Pods are always running.

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

```
Time →
Old Pods:  [🟢🟢🟢] → [🟢🟢🟡] → [🟢🟡🟡] → [🟡🟡🟡]
```

Use when:

* You need **zero downtime**
* Your app can handle **some new + old versions running together**

---

### 2️⃣ Recreate Strategy

🔁 **Metaphor**: **Fire all old agents**, then hire and train new ones.

* All old Pods are deleted first
* Then new Pods are created

```yaml
strategy:
  type: Recreate
```

Use when:

* App doesn’t support **old and new together**
* App has **shared resources** (e.g., DB, volumes)

```
Time →
Old Pods:  [🟢🟢🟢] → [] → [🟡🟡🟡]
```

---

### 3️⃣ Blue-Green Deployment (Manual)

🔵🟢 **Metaphor**: Two teams — one in **blue uniform** (old) and one in **green** (new). Switch traffic after testing.

* Run two environments: Blue (current) + Green (new)
* Test green
* Switch traffic using Service

Not native in K8s. Needs:

* Separate Deployments
* Switching `Service` label selectors

```
Service → [Blue Pods] → [Green Pods] (after switch)
```

Use when:

* You need **zero risk**
* You want to **test new version fully** before going live

---

### 4️⃣ Canary Deployment (Manual)

🐤 **Metaphor**: Send **1 new agent** first. If good, then send more.

* Deploy new version to a small % of users
* Gradually increase traffic

Needs:

* 2 Deployments (old + new)
* Ingress/Nginx + Traffic splitter (e.g., Istio, Flagger)

```
Traffic →
v1: 90% 🟢🟢🟢
v2: 10% 🟡

Then →
v1: 50% 🟢🟢
v2: 50% 🟡🟡
```

Use when:

* You want to **test in production**
* You want to **monitor** the impact of the new version

---

## 📊 Real-time Example: Zomato Backend App

You are a DevOps engineer at **Zomato**, managing the backend that handles orders.

* Want to run 5 instances of backend
* Want zero downtime deployment
* Rollback if error rate increases

**Steps**:

* Create a Deployment with 5 replicas
* Set strategy to `RollingUpdate`
* Monitor with Prometheus/Grafana
* Rollback if needed:

```bash
kubectl rollout undo deployment/zomato-backend
```

---

## 🧠 Summary Table

| Strategy       | Downtime | Risk     | Use Case                     | Real-Life Metaphor            |
| -------------- | -------- | -------- | ---------------------------- | ----------------------------- |
| Rolling Update | No       | Medium   | General updates              | Replace agents one-by-one     |
| Recreate       | Yes      | High     | Incompatible apps            | Fire all, then hire new       |
| Blue-Green     | No       | Low      | Test before switching        | Two teams, switch traffic     |
| Canary         | No       | Very Low | Gradual rollout & monitoring | Send 1 agent first, then more |
