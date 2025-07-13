# 🛡️ Network Policies in Kubernetes (Hotel Metaphor)

In Kubernetes, **Network Policies** are like hotel communication rules — defining which rooms (pods) can talk to each other. Without them, it's like a hotel where every room has an open intercom line with every other room!

---

## 📘 What are Network Policies?

Network Policies are Kubernetes resources that allow you to control **how groups of pods communicate** with each other and with the outside world.

They define:

* **Who can talk to whom** (Ingress)
* **Who can talk out** (Egress)

By default, **all pods can communicate freely**. Once a policy is applied to a pod, **all communication not explicitly allowed is denied**.

---

## 🧠 Key Concepts

* **Namespaces**: Policies are applied within a namespace.
* **Pod Selectors**: Define which pods the policy applies to.
* **Namespace Selectors**: Define which namespaces are allowed communication.
* **Ingress Rules**: Control incoming traffic to pods.
* **Egress Rules**: Control outgoing traffic from pods.

---

## 🏨 Hotel Metaphor

* **Pod** = Hotel Room 🏨
* **Container** = Guest 👤
* **Network Policy** = Security rule about which guests can visit or call each other 📲
* **No Network Policy** = All guests can freely visit each other
* **With Network Policy** = Only authorized room visits allowed

---

## 🛠️ What is FQDN?

**FQDN** stands for **Fully Qualified Domain Name**. It is the complete domain name used to access a service or pod across the cluster network.

In Kubernetes:

```
<ServiceName>.<Namespace>.svc.cluster.local
```

### Example:

```bash
curl -v telnet://mongosvc.prod.svc.cluster.local:27017
```

This FQDN resolves to the internal IP of the MongoDB service running in the `prod` namespace.

---

## ✨ How FQDN is Related to Network Policies

* **Network Policies** filter traffic based on the **Pod's labels**, **Namespace labels**, or **IP blocks**.
* **FQDN** helps in testing connectivity to services/pods using their full domain name.
* Even if the FQDN resolves to the right IP, **Network Policy can still block the request** if the source pod is not explicitly allowed.

> Think of FQDN as the **room number with extension** in the hotel, while Network Policy is the **rule allowing or denying calls to that number**.

---

## 📌 What is a NetworkPolicy in YAML?

```yaml
apiVersion: networking.k8s.io/v1  # Specifies the API version for NetworkPolicy
kind: NetworkPolicy               # The type of resource
metadata:
  name: mongodb-allow-springapp   # Name of the policy
  namespace: prod                 # Namespace where the policy applies
spec:
  podSelector:                    # Targets the pods with specific labels
    matchLabels:
      app: mongodb                # Only applies to pods labeled with app: mongodb
  policyTypes:
  - Ingress                      # This policy defines Ingress traffic rules
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: springapp         # Allow traffic from pods labeled app: springapp
    ports:
    - protocol: TCP
      port: 27017                # Only allow traffic on port 27017 (MongoDB)
```

This policy only allows the `springapp` pod to communicate with the `mongodb` pod on port 27017.

---

## 🔍 Test Connectivity (Before Policy)

```bash
kubectl exec -it <javawebapp-pod> -n prod -- curl -v telnet://mongosvc:27017  # ✅ Connected
kubectl exec -it <springapp-pod> -n prod -- curl -v telnet://mongosvc:27017   # ✅ Connected
```

---

## 🚧 After Applying NetworkPolicy

```bash
kubectl exec -it <springapp-pod> -n prod -- curl -v telnet://mongosvc:27017   # ✅ Allowed
kubectl exec -it <javawebapp-pod> -n prod -- curl -v telnet://mongosvc:27017  # ❌ Blocked
```

---

## ➕ Extend Policy to More Pods

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mongodb-allow-multiple-apps
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: mongodb
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: springapp
    - podSelector:
        matchLabels:
          app: javawebapp
    ports:
    - protocol: TCP
      port: 27017
```

---

## 🌐 Add Namespace Selector

```yaml
- namespaceSelector:
    matchLabels:
      ns: test-ns  # Allow traffic from pods in this labeled namespace
```

Use this inside `from:` block to allow pods from another namespace.

---

## 🛠️ Troubleshooting Tips

```bash
kubectl get netpol -n prod                          # List network policies
kubectl describe netpol mongodb-allow-springapp -n prod  # View details
kubectl exec -it <pod> -n prod -- curl -v telnet://mongosvc:27017  # Test connectivity
```
