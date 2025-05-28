# Step 1: Kubernetes Architecture Deep Dive

This step covers the **fundamental building blocks** of the Kubernetes control plane and node components.

---

## 🧠 Kubernetes Core Architecture

### 🔹 Control Plane Components

| Component            | Role                                                                 |
|----------------------|----------------------------------------------------------------------|
| **kube-apiserver**   | Central access point for all cluster operations; validates requests |
| **etcd**             | Distributed key-value store holding all cluster state               |
| **kube-scheduler**   | Assigns unscheduled pods to available nodes based on constraints    |
| **controller-manager** | Ensures cluster matches desired state via controllers (Replica, Node, etc.) |

---

### 🔹 Node Components

| Component         | Role                                                              |
|-------------------|-------------------------------------------------------------------|
| **kubelet**       | Agent on each node; ensures defined containers are running        |
| **kube-proxy**    | Manages networking rules and routes traffic to the right pods     |
| **Container Runtime** | Executes container images (e.g., containerd, CRI-O, Docker)    |

---

## 🧱 Mental Model

```

\[ User / kubectl ]
|
v
\[ kube-apiserver ]
|
-

\|             |              |
\[kube-scheduler] \[controller-mgr] \[etcd]
|
\[ kubelet on Node ]
|
\[ container runtime -> pod ]
|
\[ kube-proxy handles traffic ]

````

---

## 🔧 Commands Used

### ✅ Check Component Health
```bash
kubectl get componentstatuses
````

> Shows health of scheduler, etcd, and controller-manager (deprecated in newer versions)

Alternative (newer clusters):

```bash
kubectl get --raw='/healthz?verbose'
```

---

### ✅ Inspect Nodes

```bash
kubectl get nodes
kubectl describe node <node-name>
```

> Displays node capacity, status, taints, labels, conditions, running pods, and more.

---

### ✅ List System Pods (Control Plane Services)

```bash
kubectl get pods -n kube-system
```

> Confirms that key system components (etcd, controller-manager, apiserver, scheduler, kube-proxy) are running.

---

## 🔍 Tips for Understanding Architecture

* **Control plane makes decisions**, node components carry them out.
* **etcd is the brain** (state storage); if it goes down, your cluster loses memory.
* **kubelet = worker manager**, **kube-proxy = network router**.
* You only interact with **kube-apiserver**, directly or via `kubectl`.

---

## 🛠 Suggested Practice Tasks

| Task                        | Tool/Command                            |
| --------------------------- | --------------------------------------- |
| Check cluster health        | `kubectl get componentstatuses`         |
| Explore node specs          | `kubectl describe node <node-name>`     |
| View control plane pods     | `kubectl get pods -n kube-system`       |
| Identify running components | Observe output from `kube-system` pods  |
| Visualize architecture      | Use draw\.io or Lucidchart for diagrams |

---

## ✅ Completion Checklist

* [x] Understood all control plane components
* [x] Understood node components and their roles
* [x] Verified component health using `kubectl`
* [x] Inspected node resources and taints
* [ ] (Optional) Created a visual architecture diagram

---

## 📘 Bonus Resources

* [Kubernetes Official Architecture Docs](https://kubernetes.io/docs/concepts/overview/components/)
* [Interactive K8s Architecture Visualizer](https://learnk8s.io/kubernetes-architecture)

