# ⚙️ Step 07: Helm Charts & Package Management

Welcome to **Step 7** of the Kubernetes Mastery Program. In this step, you'll learn to use **Helm**, the Kubernetes package manager, to manage complex deployments in a scalable and maintainable way.

---

## 🎯 What You’ll Learn

* What Helm is and how it works
* How to install public and custom Helm charts
* Helm chart directory structure and templating
* How to pass values and override configurations
* How to handle secrets securely in Helm
* How to upgrade, rollback, and uninstall releases

---

## 🧠 Core Concepts

### 🛠 What is Helm?

> Helm is to Kubernetes what apt/yum is to Linux. It simplifies deploying applications via **charts** — collections of templated YAMLs.

**Key features:**

* Templating of Kubernetes YAML
* Version-controlled, repeatable deployments
* Easy overrides per environment
* Application packaging and reuse

---

### 📦 What is a Helm Chart?

A Helm **chart** is a directory with:

| File/Folder   | Description                               |
| ------------- | ----------------------------------------- |
| `Chart.yaml`  | Metadata about the chart                  |
| `values.yaml` | Default config values                     |
| `templates/`  | YAML templates with Go-style placeholders |
| `.helmignore` | Ignores files when packaging the chart    |

---

## 🛠 Helm Setup & Installation

### ✅ Install Helm

```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### ✅ Confirm Version

```bash
helm version
```

---

## 🚀 PART 1: Install a Public Helm Chart

### ✅ Step 1: Add the Bitnami Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### ✅ Step 2: Install NGINX Using Helm

```bash
helm install my-nginx bitnami/nginx
```

This creates a **release** named `my-nginx`.

### 🔍 Verify

```bash
helm list
kubectl get pods
kubectl get svc
```

---

## ⚙️ PART 2: Create and Deploy Your Own Chart

### ✅ Step 1: Scaffold a Chart

```bash
helm create myapp
cd myapp
```

This creates:

```
myapp/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    └── ...
```

---

### ✅ Step 2: Customize `values.yaml`

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "latest"

service:
  type: ClusterIP
  port: 80
```

---

### ✅ Step 3: Install the Chart

```bash
helm install demo-app .
```

To override values:

```bash
helm install demo-app . --set replicaCount=3
```

Or with a file:

```bash
cp values.yaml my-values.yaml
# (edit the file)
helm install demo-app . -f my-values.yaml
```

---

## 🔄 PART 3: Upgrade, Rollback, and Uninstall

### 🔁 Upgrade the release

```bash
helm upgrade demo-app . --set replicaCount=4
```

### ⏪ Rollback to previous version

```bash
helm rollback demo-app 1
```

### ❌ Uninstall

```bash
helm uninstall demo-app
```

---

## 🔐 PART 4: Secrets Management in Helm

### Option A: Inline secrets (for testing only)

```yaml
env:
  DB_USER: nest_user
  DB_PASSWORD: supersecret
```

In `deployment.yaml`:

```yaml
env:
  - name: DB_USER
    value: "{{ .Values.env.DB_USER }}"
```

---

### Option B: Use a Kubernetes Secret

**`templates/secret.yaml`**:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-secret
type: Opaque
stringData:
  DB_USER: {{ .Values.secret.DB_USER | quote }}
  DB_PASSWORD: {{ .Values.secret.DB_PASSWORD | quote }}
```

In `deployment.yaml`:

```yaml
env:
  - name: DB_USER
    valueFrom:
      secretKeyRef:
        name: {{ .Release.Name }}-secret
        key: DB_USER
```

In `values.yaml`:

```yaml
secret:
  DB_USER: nest_user
  DB_PASSWORD: s3cr3t123
```

---

## 📂 Helm Chart File Breakdown

| File                        | Purpose                                         |
| --------------------------- | ----------------------------------------------- |
| `Chart.yaml`                | Metadata (name, version, app version)           |
| `values.yaml`               | Default configurable parameters for templates   |
| `templates/deployment.yaml` | Templated Kubernetes Deployment using `.Values` |
| `templates/service.yaml`    | Templated Kubernetes Service                    |
| `templates/secret.yaml`     | (Optional) Creates Secrets from values          |
| `templates/_helpers.tpl`    | Shared template logic, e.g., full names         |
| `.helmignore`               | Files to exclude during packaging               |

---

## 🧠 Best Practices

✅ Use `values.yaml` for defaults, and override per environment
✅ Use `helm lint` to validate charts before install
✅ Avoid committing secrets — pass them with `--set` or use Vault/External Secrets
✅ Version charts with `Chart.yaml` and `helm package`
✅ Use `helm test` for post-deployment health checks

---

## ✅ Step 7 Complete!

You’ve now mastered:

* Installing public Helm charts
* Creating and templating your own charts
* Configuring and overriding values
* Managing secrets securely
* Upgrading, rolling back, and deleting releases

➡️ Ready for [Step 08: Monitoring & Logging](../step-08-monitoring-logging)
