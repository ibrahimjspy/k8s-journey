## Step 2: Pods, ReplicaSets, and Deployments (Advanced)

This step demonstrates how Kubernetes runs and manages containerized apps using **Pods**, **ReplicaSets**, and **Deployments** — the core of Kubernetes workload management.

---

## 📂 Files in This Directory

| File                    | Purpose                                  |
| ----------------------- | ---------------------------------------- |
| `nginx-pod.yaml`        | A simple Pod with health probes          |
| `nginx-rs.yaml`         | A ReplicaSet to maintain multiple Pods   |
| `nginx-deployment.yaml` | A Deployment with rolling update support |

---

## 🧱 `nginx-pod.yaml` – A Single Pod with Probes

### 🎯 Purpose:

Demonstrates how to define a Pod manually and apply **liveness** and **readiness probes** for health checking.

### 🔍 YAML Breakdown:

```yaml
apiVersion: v1                   # Specifies this is a v1 Pod resource
kind: Pod                        # Declares the resource type
metadata:
  name: nginx-probed             # Pod name
  labels:
    app: nginx                   # Label used for selection/matching
spec:
  containers:
    - name: nginx                # Container name
      image: nginx:1.25          # Container image
      ports:
        - containerPort: 80      # Port the app listens on
      livenessProbe:            # Probe to check if app is still alive
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 5
      readinessProbe:           # Probe to check if app is ready to serve traffic
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 5
```

### 🔧 Commands:

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl describe pod nginx-probed
kubectl logs nginx-probed
```

---

## 🔁 `nginx-rs.yaml` – A ReplicaSet to Maintain Replicas

### 🎯 Purpose:

Ensures that **multiple identical Pods** (3 in this case) are always running.

### 🔍 YAML Breakdown:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3                    # Desired number of Pods
  selector:
    matchLabels:
      app: nginx                 # Label selector to match pods
  template:                      # Pod template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```

### 🔧 Commands:

```bash
kubectl apply -f nginx-rs.yaml
kubectl get rs
kubectl get pods -l app=nginx
kubectl scale rs nginx-rs --replicas=5
```

---

## 🚀 `nginx-deployment.yaml` – Rolling Updates with Deployment

### 🎯 Purpose:

Provides declarative management of Pods using a Deployment. Supports **updates**, **rollbacks**, and **version control**.

### 🔍 YAML Breakdown:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 2                             # Desired number of pods
  strategy:                               # Rolling update strategy
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1                         # Extra pod during update
      maxUnavailable: 1                  # Max pods unavailable during update
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```

### 🔧 Commands:

```bash
# Deploy
kubectl apply -f nginx-deployment.yaml
kubectl get deployments
kubectl get pods -l app=nginx

# Rolling Update
kubectl set image deployment/nginx-deploy nginx=nginx:1.26
kubectl rollout status deployment nginx-deploy

# Verify Image
kubectl describe pod <pod-name>

# Rollback (if needed)
kubectl rollout undo deployment nginx-deploy
```

---

## ✅ Key Concepts Practiced

| Concept         | YAML File               | Purpose                             |
| --------------- | ----------------------- | ----------------------------------- |
| Pod             | `nginx-pod.yaml`        | Smallest unit; runs container       |
| ReplicaSet      | `nginx-rs.yaml`         | Maintains desired pod count         |
| Deployment      | `nginx-deployment.yaml` | Manages rollout/rollback/update     |
| Liveness Probe  | `nginx-pod.yaml`        | Checks if container is still alive  |
| Readiness Probe | `nginx-pod.yaml`        | Controls traffic only to ready pods |
| Rolling Update  | Deployment              | Zero-downtime updates of containers |

---

## 🛠 Troubleshooting Tips

### 🔄 Pods Stuck in `Pending` or `ContainerCreating`?

```bash
kubectl describe pod <pod-name>
```

### 💣 Disk Pressure?

```bash
kubectl describe node docker-desktop | grep Taint
# If 'disk-pressure:NoSchedule' exists:
docker system prune -a --volumes
```

### 🧼 Full Cleanup:

```bash
kubectl delete -f nginx-pod.yaml
kubectl delete -f nginx-rs.yaml
kubectl delete -f nginx-deployment.yaml
```

Or:

```bash
kubectl delete all --all
```

---

## 📘 Notes for Learners

* **Pods** are mostly used for experimentation or tight control.
* **Deployments** are the production-standard for stateless apps.
* Always use **probes** in real applications for better resilience.
* Use `kubectl rollout` commands to manage deployment lifecycle safely.

> Move on to [Step 03: ConfigMaps & Secrets](../step-03-services-networking)
