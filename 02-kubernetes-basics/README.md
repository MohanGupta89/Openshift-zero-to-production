# 02 — Kubernetes Basics
## The Engine Under OpenShift's Hood

---

## 🤔 Why Kubernetes? The Problem It Solves

Imagine you have 100 containers running your app.

**Without Kubernetes:**
```
Container 5 crashed at 3 AM → Nobody knows
Container 8 is overloaded → Others are idle
Server 2 is full → Server 3 is empty
New version deploy → Everything down for 10 mins
```

**With Kubernetes:**
```
Container 5 crashed → Auto-restarted in 10 seconds
Container 8 overloaded → Auto-scaled to 3 containers
Server 2 full → Auto-moved containers to Server 3
New version → Zero downtime rolling update
```

Kubernetes = Self-healing, auto-scaling container manager.

---

## 🏗️ Kubernetes Architecture

```
                    ┌──────────────────┐
    You type        │   MASTER NODE    │
    kubectl ──────► │                  │
                    │  • API Server    │ ← Receives all commands
                    │  • Scheduler     │ ← Decides where to run
                    │  • Controller    │ ← Keeps desired state
                    │  • etcd          │ ← Database of everything
                    └────────┬─────────┘
                             │ tells workers what to do
              ┌──────────────┼──────────────┐
              │              │              │
    ┌─────────▼──┐  ┌────────▼───┐  ┌──────▼─────┐
    │ WORKER 1   │  │ WORKER 2   │  │ WORKER 3   │
    │            │  │            │  │            │
    │ Pod Pod    │  │ Pod Pod    │  │ Pod        │
    │ Pod        │  │            │  │ Pod Pod    │
    └────────────┘  └────────────┘  └────────────┘
```

---

## 🔑 Core Kubernetes Objects

### 1. Pod — Smallest Unit
```yaml
# pod.yaml — Simplest possible pod
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
  labels:
    app: myapp
spec:
  containers:
  - name: web-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

```bash
# Create it
kubectl apply -f pod.yaml

# See it
kubectl get pods
kubectl get pods -o wide    # See which node it's on

# Describe it (details + events)
kubectl describe pod my-first-pod

# Logs
kubectl logs my-first-pod

# Go inside
kubectl exec -it my-first-pod -- bash

# Delete it
kubectl delete pod my-first-pod
```

### 2. Deployment — Manages Pods
```yaml
# deployment.yaml — Real way to run apps
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3          # Run 3 copies
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

```bash
# Deploy
kubectl apply -f deployment.yaml

# See deployments
kubectl get deployments
kubectl get deploy

# See rollout status
kubectl rollout status deployment/my-app

# Scale up
kubectl scale deployment my-app --replicas=5

# Update image (rolling update)
kubectl set image deployment/my-app my-app=nginx:1.21

# Rollback if something goes wrong
kubectl rollout undo deployment/my-app

# See rollout history
kubectl rollout history deployment/my-app
```

### 3. Service — Stable Network Address
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app           # Finds pods with this label
  ports:
  - port: 80              # Service port
    targetPort: 80        # Pod port
  type: ClusterIP         # Only accessible inside cluster
```

```
Service Types:
ClusterIP   → Internal only (default)
NodePort    → Accessible on each node's IP + port
LoadBalancer→ Creates external load balancer (cloud)
```

### 4. ConfigMap — Store Configuration
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: "mysql-service"
  DATABASE_PORT: "3306"
  LOG_LEVEL: "info"
  app.properties: |
    color=blue
    theme=dark
```

```bash
# Use in pod
env:
- name: DB_HOST
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: DATABASE_HOST
```

### 5. Secret — Store Sensitive Data
```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  # Values must be base64 encoded
  DB_PASSWORD: c2VjcmV0cGFzcw==   # "secretpass"
  API_KEY: bXlhcGlrZXkxMjM=       # "myapikey123"
```

```bash
# Create secret from command line (easier)
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=secretpass \
  --from-literal=API_KEY=myapikey123

# Use in pod
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: DB_PASSWORD
```

### 6. Namespace — Isolation
```bash
# Create namespace
kubectl create namespace development
kubectl create namespace production

# Work in a namespace
kubectl get pods -n development
kubectl apply -f app.yaml -n development

# Set default namespace
kubectl config set-context --current --namespace=development
```

---

## 🔄 Kubernetes Workflow

```
You write YAML → kubectl apply → API Server stores in etcd
                                      ↓
                              Scheduler assigns to node
                                      ↓
                              kubelet on node creates container
                                      ↓
                              Container runs!
                                      ↓
                              Controller watches continuously
                                      ↓
                              If pod dies → creates new one
```

---

## 📊 Labels and Selectors — The Glue of Kubernetes

```yaml
# Labels on pods
metadata:
  labels:
    app: frontend
    version: v2
    environment: production
    team: platform

# Selector finds pods by labels
selector:
  matchLabels:
    app: frontend
    environment: production
```

```bash
# Filter by label
kubectl get pods -l app=frontend
kubectl get pods -l environment=production
kubectl get pods -l app=frontend,version=v2

# Add label to existing pod
kubectl label pod my-pod tier=backend

# Remove label
kubectl label pod my-pod tier-
```

---

## 🏥 Health Checks — Liveness and Readiness

```yaml
containers:
- name: my-app
  image: my-app:v1
  
  # Liveness Probe: Is the container alive?
  # If fails → restart the container
  livenessProbe:
    httpGet:
      path: /health
      port: 8080
    initialDelaySeconds: 30
    periodSeconds: 10
    failureThreshold: 3

  # Readiness Probe: Is the container ready to serve traffic?
  # If fails → remove from service (no traffic sent)
  readinessProbe:
    httpGet:
      path: /ready
      port: 8080
    initialDelaySeconds: 5
    periodSeconds: 5
```

---

## 📈 Horizontal Pod Autoscaler (HPA)

```yaml
# hpa.yaml — Auto scale based on CPU
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70    # Scale when CPU > 70%
```

```bash
# Apply HPA
kubectl apply -f hpa.yaml

# Watch it scale
kubectl get hpa -w
```

---

## 🔍 Essential kubectl Commands Cheatsheet

```bash
# ===== GET (VIEW) =====
kubectl get pods
kubectl get pods -o wide          # More details
kubectl get pods -o yaml          # Full YAML output
kubectl get pods --all-namespaces # All namespaces
kubectl get all                   # Everything

# ===== DESCRIBE (DETAILS) =====
kubectl describe pod my-pod
kubectl describe node worker-1
kubectl describe deployment my-app

# ===== LOGS =====
kubectl logs my-pod
kubectl logs my-pod -c container-name   # Multi-container pod
kubectl logs -f my-pod                  # Follow live
kubectl logs --previous my-pod          # Previous crash logs

# ===== EXEC (GO INSIDE) =====
kubectl exec -it my-pod -- bash
kubectl exec -it my-pod -- sh
kubectl exec my-pod -- ls /app

# ===== APPLY / DELETE =====
kubectl apply -f file.yaml
kubectl delete -f file.yaml
kubectl delete pod my-pod
kubectl delete deployment my-app

# ===== EDIT LIVE =====
kubectl edit deployment my-app

# ===== DEBUG =====
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl top pods              # CPU/Memory usage
kubectl top nodes
```

---

## 🧪 Lab 02 — Deploy a Complete App

```bash
# Step 1: Create namespace
kubectl create namespace mylab

# Step 2: Deploy nginx
kubectl create deployment web --image=nginx:latest -n mylab
kubectl scale deployment web --replicas=3 -n mylab

# Step 3: Expose it
kubectl expose deployment web --port=80 -n mylab

# Step 4: Create ConfigMap
kubectl create configmap web-config \
  --from-literal=APP_COLOR=blue \
  -n mylab

# Step 5: Check everything
kubectl get all -n mylab

# Step 6: Watch pods
kubectl get pods -n mylab -w

# Step 7: Kill a pod and watch it recreate!
kubectl delete pod <pod-name> -n mylab
# Watch it come back automatically!

# Step 8: Scale to 0 and back
kubectl scale deployment web --replicas=0 -n mylab
kubectl scale deployment web --replicas=3 -n mylab

# Step 9: Rolling update
kubectl set image deployment/web nginx=nginx:1.21 -n mylab
kubectl rollout status deployment/web -n mylab

# Step 10: Rollback
kubectl rollout undo deployment/web -n mylab

# Cleanup
kubectl delete namespace mylab
```

---

## ❓ Interview Questions — Kubernetes

**Q1: What is a Pod?**
> Smallest deployable unit in Kubernetes. Contains one or more containers that share network and storage. Usually one container per pod.

**Q2: Difference between Deployment and Pod?**
> Pod = Single instance running. Deployment = Manager that ensures N pods are always running. If pod dies, Deployment creates new one. Always use Deployments, not bare Pods.

**Q3: What is a Service?**
> Stable network endpoint for pods. Pods get new IPs when recreated — Service provides a fixed IP/DNS name that always routes to healthy pods.

**Q4: What is etcd?**
> Distributed key-value database that stores entire cluster state. All Kubernetes data lives here. If etcd dies, cluster loses its brain.

**Q5: What is a Namespace?**
> Virtual cluster within a cluster. Used to isolate teams, environments, or projects. Resources in different namespaces don't see each other by default.

**Q6: What is the difference between liveness and readiness probe?**
> Liveness = Is container alive? Fails → restart container.
> Readiness = Is container ready for traffic? Fails → remove from load balancer, but don't restart.

**Q7: What is a DaemonSet?**
> Ensures one pod runs on every node. Used for log collectors, monitoring agents, network plugins.

**Q8: What is a StatefulSet?**
> Like Deployment but for stateful apps (databases). Pods get stable names (pod-0, pod-1) and stable storage. Used for MySQL, MongoDB, Kafka etc.

---

## ➡️ Next Step

Go to [03 — OpenShift Fundamentals](../03-openshift-fundamentals/README.md)
