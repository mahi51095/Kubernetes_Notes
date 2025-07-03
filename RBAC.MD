
# Kubernetes RBAC (Role-Based Access Control) - Simple and Real-Time Explanation

## 🧠 What is Kubernetes RBAC?
RBAC (Role-Based Access Control) in Kubernetes is a method of regulating **who can do what** in your Kubernetes cluster.

---

## 🧩 Real-Time Metaphor: Apartment Building Security

Imagine a **luxury apartment building** (your Kubernetes Cluster) with multiple **floors** (namespaces), and **residents, staff, guests** (users or service accounts).

- The **Security Guard** (Kubernetes API server) checks:
  - **Who is this person?**
  - **What are they trying to do?**
  - **Do they have permission?**

RBAC works like a set of **visitor access cards**:
- **Some cards only allow entering the gym (read-only access)**
- **Some allow entering the maintenance room (admin access)**
- **Some allow only visiting friends (limited access)**

You control this using:
- **Roles** = Define "what actions can be performed"
- **RoleBindings** = Assign those roles to people

---

## 🔍 5W1H of Kubernetes RBAC

| Concept | Explanation |
|--------|-------------|
| **What** | RBAC is a security mechanism to **control access** to resources in your Kubernetes cluster using **roles and bindings**. |
| **Why** | To ensure that **only authorized users or apps** can access or modify resources like pods, deployments, secrets, etc. |
| **Who** | Applies to **users, groups, or service accounts** trying to interact with the cluster. |
| **Where** | RBAC rules are applied at two levels: **Namespace level (Role + RoleBinding)** and **Cluster level (ClusterRole + ClusterRoleBinding)**. |
| **When** | Every time a user or service tries to **access or modify a Kubernetes resource**, RBAC is evaluated. |
| **How** | By creating **Roles or ClusterRoles** that define permissions and **RoleBindings or ClusterRoleBindings** to assign them. |

---

## 🧪 Real-Time Scenario: Dev Team and Admin Access

### Scenario:
You are an SRE in a company with 3 environments: `dev`, `staging`, `production`.

Users:
- **Developers** – access to only **create, list, delete pods in `dev` namespace**
- **Admins** – **full access** in all namespaces.

### Developer Role
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-manager
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "get", "list", "delete"]
```

### RoleBinding for Developer
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-pod-access
  namespace: dev
subjects:
- kind: User
  name: developer1
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-manager
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRoleBinding for Admins
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-access
subjects:
- kind: User
  name: admin-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

---

## 💬 Real-Time Metaphor Mapping

| Kubernetes RBAC | Apartment Metaphor |
|------------------|-------------------|
| Cluster | Apartment building |
| Namespace | Different floors |
| Role | Access card type (e.g., Gym, Library) |
| RoleBinding | Issuing card to a person |
| Verbs (get, list, delete) | Allowed actions (enter, read book, lift weights) |
| Resources (pods, secrets) | Rooms/resources in the building |
| User/ServiceAccount | Resident or Guest |

---

## 🤔 Real-Time Interview Questions

1. **Restricting Dev Access**: How to restrict developers to manage only pods in a 'dev' namespace?
2. **Role vs ClusterRole**: Difference in their scope?
3. **Read-Only Secret Access**: How to allow only get/list on secrets in `staging`?
4. **Access ConfigMaps Across All Namespaces**: What would you use?
5. **RBAC Debugging**: Access denied error when deleting pod. How to debug?
6. **Multiple Bindings**: What happens if RoleBinding and ClusterRoleBinding both apply?
7. **Multi-Tenant Clusters**: How does RBAC help?
8. **AggregationRule**: What is its use?

---

## 📌 Summary Table

| Term | Meaning |
|------|---------|
| Role | Permissions within a namespace |
| ClusterRole | Permissions across cluster/namespaces |
| RoleBinding | Connect Role to user/service account in namespace |
| ClusterRoleBinding | Connect ClusterRole to user/service account globally |
| Verb | Action: get, list, create, delete |
| Resource | Kubernetes object: pod, deployment, secret, etc. |

---

## 🧠 Pro Tips

- Always follow **least privilege principle**.
- Use `kubectl auth can-i` to troubleshoot.
- Assign service accounts to automation tools like Jenkins securely.
