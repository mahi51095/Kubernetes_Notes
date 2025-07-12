
# 📦 Kubernetes ReplicaSet Explained

## ✅ What is a ReplicaSet?

A **ReplicaSet (RS)** is the next-generation version of the **ReplicationController (RC)** in Kubernetes. It ensures that a specified number of identical Pods are always running at any given time.

---

## 🔁 Key Features of ReplicaSet

- Ensures **high availability** by maintaining the desired number of pod replicas.
- Supports **both equality-based** and **set-based selectors** (unlike ReplicationController which only supports equality-based).
- Commonly used by **Deployments** under the hood to manage scaling and rolling updates.

---

## 🔍 ReplicaSet vs ReplicationController

| Feature                          | ReplicationController | ReplicaSet           |
|----------------------------------|------------------------|----------------------|
| Maintains desired pod replicas   | ✅                     | ✅                   |
| Selector type                    | Equality-based only    | Equality + Set-based |
| Rolling update support           | ❌                     | ✅ (via Deployment)   |
| Used in production               | ❌ (deprecated)         | ✅                   |

---

## 🎯 Selector Types

### 🔹 Equality-based Selector (used in RC and RS)

```yaml
selector:
  matchLabels:
    app: javawebapp
```

Equivalent to:

```
<key>: <value>
app: javawebapp
```

### 🔸 Set-based Selector (only in RS)

```yaml
selector:
  matchExpressions:
    - key: app
      operator: In
      values:
        - javawebapp1
        - javawebapp2
```

| Operator     | Meaning                                     |
|--------------|---------------------------------------------|
| `In`         | Matches if the label's value is in the list |
| `NotIn`      | Matches if the label's value is not in list |

---

## ⚠️ Important Notes

- In **ReplicationController**, `selector` is **optional**.
- In **ReplicaSet**, `selector` is **mandatory** and must match pod labels.

---

## 🧾 Syntax Template

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: <RS-Name>
spec:
  replicas: 3
  selector:
    matchLabels:
      <key>: <value>
    matchExpressions:
      - key: <key>
        operator: <In | NotIn>
        values:
          - <value1>
          - <value2>
  template:
    metadata:
      name: <pod-name>
      labels:
        <key>: <value>
    spec:
      containers:
        - name: <containerName>
          image: <image>
          ports:
            - containerPort: <CP>
```

---

## 📘 Example 1: Equality-Based Selector

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: javawebapprs
spec:
  replicas: 2
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
          image: kkeducationb2/java-webapp:1.1
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
spec:
  type: NodePort
  selector:
    app: javawebapp
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

---

## 📘 Example 2: Set-Based Selector

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: javawebapprs
  namespace: test-ns
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - javawebapp1
          - javawebapp2
          - javawebapp
  template:
    metadata:
      labels:
        app: javawebapp1  # Ensure this matches the selector
    spec:
      containers:
        - name: javawebapp
          image: kkeducationb2/java-webapp:1.1
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
  namespace: test-ns
spec:
  type: NodePort
  selector:
    app: javawebapp1  # Must match one of the label values in pod
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

---

## 🧠 Summary

- **ReplicaSet** keeps your applications available by running the desired number of replicas.
- It supports advanced **label matching** via **set-based selectors**, which makes it more flexible.
- It is **commonly managed by Deployments** for production usage.
