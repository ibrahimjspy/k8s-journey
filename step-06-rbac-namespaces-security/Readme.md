# 🔐 Step 06: Namespaces, RBAC & Security

Welcome to **Step 6** of the Kubernetes Mastery Journey. In this step, you’ll learn how to **isolate environments**, **define fine-grained access controls**, and follow **security best practices** using **Namespaces** and **RBAC** (Role-Based Access Control).

---

## 🎯 What You’ll Learn

* Why and how to use Namespaces to isolate workloads
* The building blocks of RBAC: Roles, RoleBindings, ClusterRoles
* How to use ServiceAccounts to control Pod access
* How to enforce least-privilege permissions
* How to test and verify RBAC rules

---

## 🧠 Key Concepts

---

### 🗂️ Namespaces

> Logical partitions in a cluster to separate environments or teams.

| Use Case               | Example Namespace     |
| ---------------------- | --------------------- |
| Environment separation | `dev`, `test`, `prod` |
| Team separation        | `frontend`, `backend` |
| CI/CD pipelines        | `staging`, `release`  |

Namespaces help with **organizational clarity**, **resource isolation**, and **clean resource cleanup**.

---

### 🛡️ Role-Based Access Control (RBAC)

> Kubernetes’ permission system for defining **who can do what** on **which resources**.

| Object               | Description                               | Scope        |
| -------------------- | ----------------------------------------- | ------------ |
| `Role`               | Set of permissions within a namespace     | Namespaced   |
| `RoleBinding`        | Grants Role to user/group/serviceAccount  | Namespaced   |
| `ClusterRole`        | Global Role, usable across all namespaces | Cluster-wide |
| `ClusterRoleBinding` | Grants ClusterRole globally               | Cluster-wide |

RBAC controls **verbs** like `get`, `list`, `create`, `delete` over resources like `pods`, `secrets`, `deployments`, etc.

---

### 👤 ServiceAccount

Each Pod in Kubernetes runs as a **ServiceAccount** that can access the API. You can create custom ServiceAccounts and control their permissions via RBAC.

---

## 📁 Files and YAML Explained

---

### 1️⃣ `namespace.yaml`

Creates a Namespace called `dev`.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

📌 **Usage**:

```bash
kubectl apply -f namespace.yaml
```

---

### 2️⃣ `serviceaccount.yaml`

Creates a ServiceAccount named `dev-user` in the `dev` namespace.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-user
  namespace: dev
```

📌 **Usage**:

```bash
kubectl apply -f serviceaccount.yaml
```

---

### 3️⃣ `role.yaml`

Defines a Role that allows **read-only access to Pods** within the `dev` namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

🧠 **Explanation**:

* `apiGroups: [""]` → for core resources like `pods`
* Only allows read access to pods (`get`, `list`, `watch`)

📌 **Usage**:

```bash
kubectl apply -f role.yaml
```

---

### 4️⃣ `rolebinding.yaml`

Binds the Role to the `dev-user` ServiceAccount.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
  - kind: ServiceAccount
    name: dev-user
    namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

📌 **Usage**:

```bash
kubectl apply -f rolebinding.yaml
```

---

## 🧪 Testing Access

Use the following command to simulate actions **as the dev-user ServiceAccount**:

### ✅ Allowed (list pods in `dev`):

```bash
kubectl auth can-i list pods --as=system:serviceaccount:dev:dev-user --namespace=dev
```

✅ Output:

```
yes
```

### ❌ Denied (list pods in `default`):

```bash
kubectl auth can-i list pods --as=system:serviceaccount:dev:dev-user --namespace=default
```

❌ Output:

```
no
```

This confirms:

* RBAC scoping works as expected
* Access is **limited to `dev` namespace** only

---

## 🧹 Cleanup

To remove all resources created in this step:

```bash
kubectl delete -f rolebinding.yaml
kubectl delete -f role.yaml
kubectl delete -f serviceaccount.yaml
kubectl delete -f namespace.yaml
```

---

## 🧠 Best Practices

* 🔐 **Follow least privilege**: Only grant the exact permissions needed.
* 📂 **Use namespaces for environment isolation**.
* 🔄 **Rotate ServiceAccount tokens** and avoid using `default`.
* 📜 **Audit RBAC policies** regularly.
* ✅ Use `kubectl auth can-i` to verify RBAC logic before applying.

---

## ✅ Step 6 Complete!

You now understand:

* How to use Namespaces for clean multi-environment isolation
* How to create and bind Roles to restrict access
* How RBAC controls user and Pod permissions
* How to enforce and test least-privilege security

➡️ Ready for [Step 07: Helm Charts & Package Management](../step-07-helm-charts)

