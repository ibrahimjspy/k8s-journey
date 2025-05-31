# 📦 Step 03: Services & Networking Internals

This step covers how Pods communicate inside a Kubernetes cluster and how to expose your applications to the outside world. You'll learn the difference between **ClusterIP**, **NodePort**, and **Port Forwarding**, and you'll use `curlpod` to verify internal service networking.

---

## 🎯 Goals of This Step

- Understand Kubernetes **networking primitives**
- Learn how to expose applications internally (ClusterIP) and externally (NodePort)
- Practice using `kubectl` and `curl` for verification
- Prepare for Ingress and advanced routing in future steps

---

## 🧠 Key Definitions & Use Cases

### 🟦 Pod
> **Definition**: The smallest deployable unit in Kubernetes. It wraps one or more containers (usually just one).

🔧 **Use case**: Your Node.js app or NGINX server runs inside a Pod.

---

### 🟦 Deployment
> **Definition**: A controller that manages the lifecycle of Pods — creation, updates, scaling, rollbacks.

🔧 **Use case**: You define "run 3 replicas of my web app" — the Deployment ensures that.

---

### 🟦 ReplicaSet
> **Definition**: Ensures the desired number of Pod replicas are always running.

🔧 **Use case**: Deployment tells ReplicaSet "keep 3 Pods running."

---

### 🟦 Service
> **Definition**: A stable IP and DNS name that exposes a set of Pods.

🔧 **Use case**: Your frontend app talks to `http://backend-service`, not specific Pod IPs.

---

### 🟦 ClusterIP (Default Service)
> **Definition**: Exposes the service only within the cluster. Not accessible from outside.

🔧 **Use case**: Internal microservice-to-microservice communication.

---

### 🟦 NodePort
> **Definition**: Exposes a service on a static port on every cluster node (default range: 30000–32767).

🔧 **Use case**: Access an app from your browser using `http://<minikube-ip>:<nodePort>`.  
⚠️ On macOS (Docker driver), this may **not work** due to network isolation.

---

### 🟦 Port Forwarding
> **Definition**: Forwards traffic from your machine to a Pod or Service inside the cluster.

🔧 **Use case**: Access internal service using `http://localhost:8080` via `kubectl port-forward`.

---

### 🟦 curlpod
> **Definition**: A helper Pod that runs the `curl` tool, allowing you to test internal service access.

🔧 **Use case**: Verify that `nginx-service-clusterip` is accessible inside the cluster.

---

## 📂 YAML Files in This Step

### 1. `deployment.yaml` — NGINX App

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
          image: nginx:latest
          ports:
            - containerPort: 80
````

🧠 Runs 2 NGINX Pods. This is your backend app.

---

### 2. `service-clusterip.yaml` — Internal ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

🧠 Provides an internal stable endpoint (`nginx-service-clusterip`) that can be accessed by other Pods.

---

### 3. `service-nodeport.yaml` — External NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

🧠 Opens port `30080` on Minikube. Access via `http://<minikube-ip>:30080` (may not work on macOS Docker).

---

### 4. `curl-pod.yaml` — Internal Testing Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curlpod
spec:
  containers:
    - name: curl
      image: curlimages/curl:latest
      command: ['sleep', '3600']
      tty: true
```

🧠 Used to test internal access to `nginx-service-clusterip` via:

```bash
kubectl exec -it curlpod -- sh
curl http://nginx-service-clusterip
```

---

## 🛠 Practical Commands

### 🔹 Start Minikube

```bash
minikube start --driver=docker
```

---

### 🔹 Apply All YAMLs

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service-clusterip.yaml
kubectl apply -f service-nodeport.yaml
kubectl apply -f curl-pod.yaml
```

---

### 🔹 Test Internal Access (Inside Cluster)

```bash
kubectl exec -it curlpod -- sh
curl http://nginx-service-clusterip
```

✅ If successful, you’ll see NGINX HTML output.

---

### 🔹 Test External Access (May fail on macOS)

```bash
minikube ip
# Example output: 192.168.49.2

# Then open in browser:
http://192.168.49.2:30080
```

If this hangs or fails, continue with...

---

### 🔹 Reliable Access via Port Forwarding

```bash
kubectl port-forward service/nginx-service-clusterip 8080:80
```

Then open:

```
http://localhost:8080
```

✅ This works reliably on macOS.

---

### 🧹 Cleanup Resources

```bash
kubectl delete -f .
```

---

## 📘 Summary Table

| Component    | Description                    | Use Case                                     |
| ------------ | ------------------------------ | -------------------------------------------- |
| Pod          | Runs containers                | Your actual app runtime                      |
| Deployment   | Manages Pods, scaling, updates | Desired state declaration                    |
| ReplicaSet   | Maintains number of replicas   | Created/managed by Deployment                |
| Service      | Stable endpoint for Pods       | Internal/external networking                 |
| ClusterIP    | Default service, internal only | App-to-app communication                     |
| NodePort     | Opens port on all nodes        | Testing apps from browser (limited on macOS) |
| Port Forward | Local port to internal service | Reliable local testing                       |
| curlpod      | Pod with curl tool             | Test internal networking                     |

---

## ✅ Next Step

> Move on to [Step 04: ConfigMaps & Secrets](../step-04-config-secrets)
> Learn how to inject environment configs and secure values into your containers.

