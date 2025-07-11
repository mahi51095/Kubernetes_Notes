# 📘 Kubernetes: Labels, Selectors, and Services

---

## 🔖 Labels

- Labels are **key/value pairs** attached to Kubernetes objects like pods.
- They are used to **identify**, **group**, or **categorize** objects.
- You can create custom labels.

**Example:**
```yaml
labels:
  app: nginx
  role: web
  env: dev
```

---

## 🔍 Selectors

- Selectors **query/filter** objects using labels.
- Used by controllers (like ReplicaSets, Deployments, Services) to **find and manage** matching pods.

**Example:**
```yaml
selectors:
  env = dev
  app != db
  release in (1.3,1.4)
```

---

## 🔄 Labels & Selectors in Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: javawebapptest
  labels:
    app: javawebappTest
  namespace: test-ns
spec:
  containers:
  - name: javawebapp1
    image: kkeducationb2/java-webapp:1.1
    ports:
    - containerPort: 8080
```
**Commands:**
```bash
kubectl apply -f javawebapp.yaml --dry-run=client
kubectl apply -f javawebapp.yaml
```

**Mongo Pod Example:**
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

**Commands:**
```bash
kubectl apply -f mongo.yaml
kubectl describe pod <pod-name> -n <namespace>
kubectl get po <pod-name> -n <namespace> --show-labels
kubectl get po -n test-ns -o wide
kubectl exec -it -n test-ns javawebapptest -- sh
ping <another-pod-ip>
```

---

## ❓ Why Use a Kubernetes Service Object?

Pods can talk using IPs, but not ideal because:
1. Pod IPs can **change when a pod crashes**.
2. Multiple replicas = multiple IPs = hard to manage.

💡 Solution: Use a **Service** object.

---

## 📦 Kubernetes Service

- A **Service** is an abstraction that defines a logical set of pods and a policy to access them.
- It uses **label selectors** to find and route traffic to pods.

---

## 🔗 Service Types

1. **ClusterIP** (default)
2. **NodePort**
3. **LoadBalancer**

---

### 1. ClusterIP

- Exposes the service on an internal IP.
- Only reachable **within the cluster**.
- Other pods can access the service using its **name** (via kube-dns).

**Example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: javawebapp
  labels:
    app: javawebapp
  namespace: test-ns
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
  type: ClusterIP
  selector:
    app: javawebapp
  ports:
  - port: 80
    targetPort: 8080
```

**Commands:**
```bash
kubectl apply -f pod1.yaml
kubectl get all -n test-ns -o wide --show-labels
kubectl describe svc javawebappsvc
```

**Another example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-web-app
  labels:
    app: simple-web-app
spec:
  containers:
  - name: simple-web-app
    image: nginx:latest
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: simple-web-service
spec:
  selector:
    app: simple-web-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

```bash
curl http://<ClusterIP>:80
```

---

### 2. NodePort

- Exposes the service on **each node's IP** at a **static port** (default range: 30000–32767).
- Can access the service from outside the cluster using:  
  `http://<NodeIP>:<NodePort>`

**Example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: javawebapp
  labels:
    app: javawebapp
  namespace: test-ns
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
  name: javawebappsvcnp
  namespace: test-ns
spec:
  type: NodePort
  selector:
    app: javawebapp
  ports:
  - port: 80
    targetPort: 8080
```

---
