# 🧑‍💼 RBAC in Kubernetes – Office Metaphor

Think of Kubernetes like an office building. **RBAC (Role-Based Access Control)** is the security system that controls **who can access what rooms and do what activities** inside the office.

---

## 🔐 What is RBAC?

RBAC lets you control **what users can do** in your Kubernetes cluster. It defines:

* **Who** (subject: person, team, app)
* **Can do what** (verbs: get, list, create, delete)
* **On what resources** (pods, services, secrets, etc.)

---

## 🏢 Office Metaphor Mapping

| RBAC Concept       | Office Analogy                               |
| ------------------ | -------------------------------------------- |
| Role               | Job Description (e.g., Receptionist)         |
| ClusterRole        | Job Description that works across all floors |
| RoleBinding        | Assigning a person to a specific floor job   |
| ClusterRoleBinding | Assigning a person to building-wide job      |
| Subject            | Actual employee (user or app)                |

---

## 📂 Role (Floor Access Rules)

A Role defines what a person can do on a **specific floor (namespace)**.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: test-ns            # Namespace where Role is defined
  name: pod-reader              # Name of the Role
rules:
- apiGroups: [""]               # Core API group for basic resources like pods
  resources: ["pods"]           # Resource type
  verbs: ["get", "watch", "list"]  # Allowed operations
- apiGroups: ["apps"]          # API group for deployments
  resources: ["deployments"]   # Additional resource
  verbs: ["get", "watch", "list"]
```

## 📌 RoleBinding (Give the Job to Someone)

This binds the Role to a user or team member in that namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods               # Name of the binding
  namespace: test-ns            # Namespace where it's active
subjects:
- kind: User                    # Binding type (User, Group, or ServiceAccount)
  name: kkfunda                 # IAM or Kubernetes username
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role                    # Refers to Role kind
  name: pod-reader              # Name of Role to bind
  apiGroup: rbac.authorization.k8s.io
```

---

## 🏢 ClusterRole (Building-wide Access Rules)

ClusterRole is like a job that lets someone work on **all floors or global systems**.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fullaccesscr            # Name of the ClusterRole
rules:
- apiGroups: ["*"]              # Wildcard for all API groups
  resources: ["*"]              # Wildcard for all resource types
  verbs: ["*"]                  # Wildcard for all actions (get, create, delete...)
```

## 🗂 ClusterRoleBinding (Assign It Building-Wide)

Assign a ClusterRole to someone who needs access across all floors.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: fullaccess              # Name of the ClusterRoleBinding
subjects:
- kind: User                    # Binding a User
  name: kkfunda                 # IAM or Kubernetes username
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole             # Refers to ClusterRole kind
  name: fullaccesscr            # Name of the ClusterRole
  apiGroup: rbac.authorization.k8s.io
```

---

## 👩‍💻 Practical Setup for AWS EKS Users

### Step 1: Install `kubectl`

Download using:

```bash
curl.exe -LO "https://dl.k8s.io/release/v1.32.0/bin/windows/amd64/kubectl.exe"
```

Add path to environment variable: `C:\Users\<your-name>\Desktop\rbac-practise`

### Step 2: Install AWS CLI

```bash
https://awscli.amazonaws.com/AWSCLIV2.msi
```

### Step 3: Configure IAM Credentials

```bash
aws configure
```

Enter Access key, Secret key, Region: `ap-south-1`

### Step 4: Update kubeconfig for EKS cluster

```bash
aws eks update-kubeconfig --name EKS-Demo123 --region ap-southeast-1
```

### Step 5: Add IAM user to cluster

Edit `aws-auth` ConfigMap in kube-system:

```yaml
mapUsers: |
  - userarn: arn:aws:iam::<account-id>:user/prasanth
    username: prasanth
```

Then:

```bash
kubectl edit cm aws-auth -n kube-system
```

Now, bind RBAC roles to that user using Role/RoleBinding or ClusterRole/ClusterRoleBinding.

---

## 🔍 Useful Commands

```bash
kubectl get roles -n test-ns
kubectl get rolebindings -n test-ns
kubectl auth can-i list pods --as kkfunda --namespace=test-ns
```

---

## ✅ Summary

RBAC is like giving specific office roles to employees:

* Define the job (Role / ClusterRole)
* Assign the job (RoleBinding / ClusterRoleBinding)
* Control access to resources based on real needs

Let me know if you want to:

* Add **ServiceAccounts** with RBAC
* Explore **Jenkins or CI/CD integration**
* Troubleshoot with `kubectl auth can-i`
