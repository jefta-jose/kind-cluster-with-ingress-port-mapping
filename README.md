# 🛠 Local Kubernetes Setup with Kind + NGINX Ingress + Helm

This guide explains how to:

* Create a local Kubernetes cluster using kind
* Install NGINX Ingress Controller
* Deploy the application using Helm
* Expose the app locally via a custom domain

---

# 🚀 FIRST-TIME SETUP

---

## 1️⃣ Create the Cluster

```bash
kind create cluster --name go-api-cluster --config kind-config.yaml --wait 5m
```

### What this does

* Creates a Kubernetes cluster inside Docker
* Applies your custom Kind configuration
* Waits up to 5 minutes for readiness

---

## 2️⃣ Verify Kubernetes Context

Check current context:

```bash
kubectl config current-context
```

Expected:

```
kind-go-api-cluster
```

If not:

```bash
kubectl config use-context kind-go-api-cluster
```

---

## 3️⃣ Install NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

This enables HTTP routing into your cluster.

---

## 4️⃣ Wait for Ingress to Be Ready

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

---

## 5️⃣ Configure Local DNS (`/etc/hosts`)

Kubernetes Ingress uses hostnames like:

```
go-api.local
```

Your machine does NOT know what that domain is.

We manually map it to localhost:

```bash
echo "127.0.0.1 go-api.local" | sudo tee -a /etc/hosts
```

### Why this works

* Kind maps ports 80/443 → localhost
* Your browser resolves `go-api.local` → 127.0.0.1
* Traffic enters the cluster through Ingress

---

## 6️⃣ Build & Load Docker Image into Kind

### Build image locally

```bash
docker build -t ecom-go-api-project:latest .
```

### Load image into Kind cluster

```bash
kind load docker-image ecom-go-api-project:latest --name go-api-cluster
```

### Why this step is required

Kind runs inside Docker and cannot see your local images.

Without loading:

```
ErrImagePull
ImagePullBackOff
```

Make sure your `values.yaml` contains:

```yaml
image:
  pullPolicy: IfNotPresent
```

This prevents Kubernetes from trying to pull from Docker Hub.

---

## 7️⃣ Deploy the Helm Chart

```bash
helm install go-api ./go-api-chart
```

---

## 8️⃣ Verify Deployment

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

Test in browser or API client:

```
http://go-api.local
```

---

# 🔁 FUTURE UPDATES (Code Changes)

When you update application code:

---

## 1️⃣ Rebuild Docker Image

```bash
docker build -t ecom-go-api-project:latest .
```

---

## 2️⃣ Reload Image into Kind

```bash
kind load docker-image ecom-go-api-project:latest --name go-api-cluster
```

---

## 3️⃣ Upgrade Helm Release

```bash
helm upgrade go-api ./go-api-chart
```

If you changed values:

```bash
helm upgrade go-api ./go-api-chart -f values.yaml
```

---

## 4️⃣ Restart Pods (If Needed)

If Kubernetes doesn’t auto-redeploy:

```bash
kubectl rollout restart deployment go-api
```

---

# 🆕 Creating a New Project (Example: Node API)

Most steps remain identical.

Only change:

* Cluster name
* Image name
* Helm release name
* Domain

---

## Example: Node API Setup

### Create Cluster

```bash
kind create cluster --name node-api-cluster --config kind-config.yaml
```

### Add Domain Mapping

```bash
echo "127.0.0.1 node-api.local" | sudo tee -a /etc/hosts
```

### Build Image

```bash
docker build -t node-api:latest .
```

### Load Image

```bash
kind load docker-image node-api:latest --name node-api-cluster
```

### Deploy

```bash
helm install node-api ./node-api-chart
```

---

# 🧹 Cleanup

### Delete Helm Release

```bash
helm uninstall go-api
```

### Delete Cluster

```bash
kind delete cluster --name go-api-cluster
```

---

# 🧠 Architecture Summary

```
Browser
   ↓
/etc/hosts → 127.0.0.1
   ↓
Kind (Docker)
   ↓
NGINX Ingress
   ↓
Service
   ↓
Pod
```

---

# ⚠ Common Issues

### ❌ ImagePullBackOff

Fix:

* Ensure image is loaded
* Check image name matches values.yaml
* Confirm pullPolicy: IfNotPresent

---

### ❌ Ingress Not Working

Check:

```bash
kubectl get pods -n ingress-nginx
```

---

### ❌ Wrong Cluster Context

Always verify:

```bash
kubectl config current-context
```

---