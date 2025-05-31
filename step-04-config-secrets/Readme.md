# 🔐 Step 04: ConfigMaps & Secrets

Welcome to Step 4 of the Kubernetes Mastery Program. In this step, you'll learn how to manage **application configuration** and **sensitive data** using native Kubernetes resources: `ConfigMap` and `Secret`.

---

## 🎯 What You’ll Learn

- The purpose and structure of ConfigMaps and Secrets
- How to inject them into Pods via environment variables
- How to mount them as files
- Best practices for decoupling config from app code

---

## 🧠 Core Concepts & Definitions

### 📦 ConfigMap

> A Kubernetes object for storing **non-sensitive** key-value configuration data.

✅ **Use cases**:
- Inject app name, logging levels, environment flags
- Mount entire configuration files into containers
- Share settings across multiple Pods

🛠 Common format: key-value pairs

---

### 🔐 Secret

> A Kubernetes object for storing **sensitive** data, such as passwords or API tokens. All values are **base64-encoded**.

✅ **Use cases**:
- Database credentials
- Access tokens and API keys
- SSH keys and TLS certificates

⚠️ **Note**: Secrets are base64-encoded, not encrypted by default — use RBAC and external vaults (like HashiCorp Vault) in production.

---

## 🧾 ConfigMap vs Secret — Summary Table

| Feature            | ConfigMap                          | Secret                                  |
|--------------------|-------------------------------------|------------------------------------------|
| Purpose            | Store non-sensitive config         | Store sensitive data                     |
| Format             | Plain text                         | Base64 encoded                           |
| Type               | `v1.ConfigMap`                     | `v1.Secret`                              |
| Access Methods     | Env vars or volume mounts          | Env vars or volume mounts                |
| Typical Usage      | Logging levels, mode, messages     | Passwords, tokens, API keys              |

---

## 📁 Files and YAML Explained

### 1️⃣ `configmap.yaml`

Stores app environment configuration.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_NAME: "my-k8s-app"
  WELCOME_MSG: "Welcome to the Kubernetes Config Demo"
````

🧠 **Explanation**:

* `APP_ENV`: sets app environment to `production`
* `APP_NAME`: name of the app
* `WELCOME_MSG`: sample display message

---

### 2️⃣ `secret.yaml`

Stores base64-encoded secrets.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_USER: c3VwZXJ1c2Vy        # base64 of 'superuser'
  DB_PASSWORD: c3VwZXJzZWNyZXQ=  # base64 of 'supersecret'
```

🧠 **Explanation**:

* `type: Opaque`: for generic key-value secrets
* Use `echo -n 'value' | base64` to encode
* Use `echo 'base64val' | base64 --decode` to decode

---

### 3️⃣ `pod-with-config.yaml`

Injects config and secret as **environment variables** into the Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod-env
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "env; sleep 3600"]
      env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_NAME
        - name: WELCOME_MSG
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: WELCOME_MSG
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_USER
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```

🧠 **Explanation**:

* This Pod will print all environment variables (including injected ones) on startup.
* It will then sleep so you can inspect it.

---

### 4️⃣ `pod-with-mounted-config.yaml`

Mounts ConfigMap and Secret **as files** inside the Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod-volume
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "ls /etc/config; cat /etc/config/APP_ENV; cat /etc/secret/DB_PASSWORD; sleep 3600"]
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
        - name: secret-volume
          mountPath: /etc/secret
  volumes:
    - name: config-volume
      configMap:
        name: app-config
    - name: secret-volume
      secret:
        secretName: app-secret
```

🧠 **Explanation**:

* Files are mounted to `/etc/config` and `/etc/secret`
* You can `cat` them like normal text files
* Ideal for mounting configuration files, certificates, etc.

---

## 🛠 Commands to Try

### ✅ Apply all files:

```bash
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f pod-with-config.yaml
kubectl apply -f pod-with-mounted-config.yaml
```

---

### 🔍 Inspect env vars:

```bash
kubectl exec -it app-pod-env -- sh
env | grep APP_
env | grep DB_
```

---

### 🔍 Inspect mounted files:

```bash
kubectl exec -it app-pod-volume -- sh
ls /etc/config
cat /etc/config/WELCOME_MSG
cat /etc/secret/DB_USER
```

---

### 🧹 Cleanup:

```bash
kubectl delete -f .
```

---

## 🧠 Best Practices

* Use ConfigMaps for values you might want to override (e.g. logging level)
* Use Secrets for sensitive values even in dev — get in the habit
* Avoid hardcoding secrets in code or YAML
* Use tools like **Sealed Secrets**, **Vault**, or **External Secrets** for production

---

## ✅ Step 4 Complete!

You're now comfortable with:

* Configuring apps via Kubernetes-native tools
* Injecting and managing both config and secrets
* Clean separation of logic, config, and security

➡️ Ready for [Step 05: StatefulSets & Persistent Storage](../step-05-statefulsets-storage)

