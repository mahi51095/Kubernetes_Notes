# Static Pods in Kubernetes

## Table of Contents
- [What is a Static Pod?](#what-is-a-static-pod)
- [Key Characteristics](#key-characteristics)
- [When to Use Static Pods](#when-to-use-static-pods)
- [How Static Pods Work — Behind the Scenes](#how-static-pods-work--behind-the-scenes)
- [Hands‑On Example : Nginx Static Pod](#handson-example--nginx-static-pod)
- [Quick Facts to Memorise](#quick-facts-to-memorise)
- [Advantages](#advantages)
- [Disadvantages](#disadvantages)

## What is a Static Pod?
A **Static Pod** in Kubernetes is a pod that bypasses the Kubernetes API server and is managed **directly by the Kubelet** on a particular node.  
Its manifest is a plain YAML (or JSON) file sitting on the node’s filesystem. Whenever the Kubelet sees the file, it spins the pod up and keeps it alive.

---

## Key Characteristics

- **Not Managed by the Kubernetes Scheduler**  
  Static Pods are created on the node immediately; they never enter the scheduler’s queue.

- **Node‑Specific**  
  Because the manifest lives on disk **inside a single node**, the pod is tied to that node and cannot be moved automatically.

- **Self‑Healing**  
  If the container dies, the Kubelet restarts it as long as the manifest is still present.

- **Invisible to the API Server**  
  They do *not* appear in `kubectl get pods` by default.  
  You can see them via `kubectl describe node <node-name>` or by inspecting Kubelet logs.

---

## When to Use Static Pods

| Scenario | Why Static Pods Help |
|----------|----------------------|
| **Essential system services** (logging agents, node‑exporter, etc.) | Must run on *every* node, survive API outages |
| **Node‑pinned workloads** | A workload that must stay on specific hardware |
| **Core Kubernetes components** | `kube‑apiserver`, `kube‑controller‑manager`, `kube‑scheduler` themselves often run as static pods on control‑plane nodes |

---

## How Static Pods Work — Behind the Scenes

1. **Manifest on Disk** – Save a file under a directory the Kubelet watches (default `/etc/kubernetes/manifests/`).
2. **Kubelet File‑Watcher** – The Kubelet polls / inotify‑watches the directory for additions, changes, or deletions.
3. **Lifecycle Management** –  
   * create → start container(s)  
   * crash → restart  
   * delete manifest → terminate pod

---

## Hands‑On Example : Nginx Static Pod

### 1  Create `nginx-static-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### 2  Move the file into the manifests directory
```bash
sudo mv nginx-static-pod.yaml /etc/kubernetes/manifests/
```

### 3  Watch Kubelet spin up the pod
```bash
# May not list static pods
kubectl get pods --all-namespaces

# But they show up on the node description
kubectl describe node <node-name>
```

### 4  Lifecycle in action  
Delete the file → Kubelet deletes the pod.  
Crash the container → Kubelet restarts it.

---

## Quick Facts to Memorise

- Static Pods **do not** participate in scheduling.
- The manifest’s *directory* acts like a poor‑man’s “controller”.
- Great for *per‑node* daemons; bad for scalable apps.

---

## Advantages

1. **Self‑Healing** – Automatic restarts without external controllers.  
2. **Node‑Specific** – Perfect for node‑level agents.  
3. **Simplicity** – Plain files, no API objects to manage.

## Disadvantages

1. **No Scheduler / No Scaling** – You cannot `kubectl scale` a static pod.  
2. **Operational Overhead at Scale** – Managing files on hundreds of nodes quickly becomes painful.

---

> **Tip:** If you need the same behaviour but via the API, use a *DaemonSet* — it gives you one pod *per node* while remaining fully managed by Kubernetes.
