# 🗃️ Step 05: StatefulSets & Persistent Storage

Welcome to **Step 5** of the Kubernetes Mastery Program! In this step, you’ll explore how to manage **stateful applications** and **persistent storage** using Kubernetes-native resources like `StatefulSet`, `PersistentVolumeClaim`, and `StorageClass`.

---

## 🎯 What You’ll Learn

* What StatefulSets are and how they differ from Deployments
* How persistent storage is provisioned using PVCs
* How to create a MongoDB StatefulSet with dedicated volumes
* How to test persistence by deleting and restarting Pods

---

## 🧠 Key Concepts

### 📦 StatefulSet

A Kubernetes controller used for managing **stateful applications**, where each Pod needs:

* A **stable identity** (hostname, DNS name)
* A **persistent volume** that sticks with it
* **Ordered** startup/shutdown

Use cases include databases (MongoDB, PostgreSQL), Kafka, Zookeeper, etc.

---

### 💾 PersistentVolume (PV) and PersistentVolumeClaim (PVC)

* **PV**: A piece of cluster storage (disk, NFS, cloud volume)
* **PVC**: A Pod’s request for storage

Kubernetes binds a PVC to a matching PV. With StatefulSets, each Pod gets **its own PVC**, retaining data even if the Pod is rescheduled.

---

### 🕸 Headless Service

A Kubernetes Service with `clusterIP: None`, used to give Pods in a StatefulSet **stable DNS names** like:

```
mongo-0.mongo.default.svc.cluster.local
mongo-1.mongo.default.svc.cluster.local
```

---

## 📁 Files and YAML Explained

---

### 1️⃣ `service-headless.yaml`

Creates a **headless service** for stable network identity.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  clusterIP: None
  selector:
    app: mongo
  ports:
    - port: 27017
```

🧠 **Why?**

* Allows Pods like `mongo-0` and `mongo-1` to resolve each other by DNS name.

---

### 2️⃣ `statefulset-mongo.yaml`

Creates a MongoDB StatefulSet with 2 replicas, each with a **dedicated 1Gi volume**.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  serviceName: "mongo"  # Must match the headless service
  replicas: 2
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:5
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongo-storage
              mountPath: /data/db
  volumeClaimTemplates:
    - metadata:
        name: mongo-storage
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

🧠 **Key Parts:**

* `volumeMounts`: Mounts volume into container path `/data/db`
* `volumeClaimTemplates`: Tells K8s to create a separate PVC per Pod (e.g. `mongo-storage-mongo-0`)
* Pods will be named `mongo-0`, `mongo-1`, etc.

---

## 🛠 Commands to Run

---

### ✅ Apply All Resources

```bash
kubectl apply -f service-headless.yaml
kubectl apply -f statefulset-mongo.yaml
```

---

### 🔍 Inspect StatefulSet & Pods

```bash
kubectl get statefulsets
kubectl get pods -l app=mongo
```

You should see Pods like:

```
mongo-0   Running
mongo-1   Running
```

---

### 🔍 Inspect Volumes (PVCs)

```bash
kubectl get pvc
```

Expected output:

```
mongo-storage-mongo-0   Bound
mongo-storage-mongo-1   Bound
```

Each Pod has its **own unique PVC**.

---

### 🧪 Test Persistence

#### Step 1: Connect to Mongo and Insert Data

```bash
kubectl exec -it mongo-0 -- mongosh
> use mydb
> db.test.insertOne({message: "hello stateful world"})
> db.test.find()
```

#### Step 2: Delete Pod & Reconnect

```bash
kubectl delete pod mongo-0
kubectl get pods -l app=mongo  # Wait for mongo-0 to restart
kubectl exec -it mongo-0 -- mongosh
> use mydb
> db.test.find()
```

✅ Data should still be there — it’s stored on the persistent volume.

---

### 🧹 Cleanup Resources

```bash
kubectl delete -f statefulset-mongo.yaml
kubectl delete -f service-headless.yaml
```

PVCs remain by default. To clean them:

```bash
kubectl delete pvc -l app=mongo
```

---

## 🧠 Best Practices

* Use **StatefulSets** when:

  * Pod identity matters
  * Storage must persist after pod restarts
* **Avoid StatefulSets** for purely stateless apps — they add unnecessary complexity.
* Always define a **volumeClaimTemplate** in StatefulSets.
* Use **StorageClasses** to dynamically provision storage in cloud environments.
* Clean up **PVCs manually** unless you're okay with retained data.

---

## ✅ Step 5 Complete!

You now understand:

* The power of StatefulSets for running databases and other stateful apps
* How PVCs give each Pod its own persistent storage
* How Kubernetes retains data even after Pod deletion

➡️ Ready for [Step 06: Namespaces, RBAC & Security](../step-06-rbac-namespaces-security)

