# 14 — Hands-On Labs
## Practice Makes Architect — 20 Real Labs

---

## 🔧 Setup Before All Labs

```bash
# Login to Developer Sandbox
oc login --token=YOUR_TOKEN --server=YOUR_SERVER

# Verify
oc whoami
oc get projects
```

---

## Lab 1 — Your First App (Beginner)
**Goal:** Deploy, expose, access an app

```bash
oc new-project lab01
oc new-app --image=nginx:latest --name=web
oc expose service web
oc get route web
curl http://$(oc get route web -o jsonpath='{.spec.host}')
# ✅ See nginx welcome page = SUCCESS
oc delete project lab01
```

---

## Lab 2 — Scale and Self-Heal (Beginner)
**Goal:** See auto-recovery in action

```bash
oc new-project lab02
oc new-app --image=nginx:latest --name=web
oc scale deployment web --replicas=5
oc get pods -w

# Kill a pod — watch it come back!
POD=$(oc get pods -o name | head -1)
oc delete $POD
oc get pods -w
# ✅ New pod created automatically = SUCCESS

oc delete project lab02
```

---

## Lab 3 — Rolling Update (Beginner)
**Goal:** Zero downtime deployment

```bash
oc new-project lab03
oc new-app --image=nginx:1.20 --name=web
oc scale deployment web --replicas=3
oc expose service web

# Update to new version
oc set image deployment/web nginx=nginx:1.21
oc rollout status deployment/web

# Rollback
oc rollout undo deployment/web
oc rollout status deployment/web
# ✅ Rollback complete = SUCCESS

oc delete project lab03
```

---

## Lab 4 — ConfigMap and Secrets (Beginner)
**Goal:** Pass config to your app

```bash
oc new-project lab04

# Create ConfigMap
oc create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MODE=production

# Create Secret
oc create secret generic app-secret \
  --from-literal=DB_PASSWORD=mysecretpassword

# Deploy app
oc new-app --image=nginx:latest --name=web

# Inject ConfigMap as env vars
oc set env deployment/web --from=configmap/app-config

# Inject Secret as env vars
oc set env deployment/web --from=secret/app-secret

# Verify inside pod
POD=$(oc get pods -l app=web -o name | head -1)
oc exec $POD -- env | grep -E "APP_|DB_"
# ✅ See APP_COLOR=blue etc = SUCCESS

oc delete project lab04
```

---

## Lab 5 — Persistent Storage (Intermediate)
**Goal:** Data survives pod restart

```bash
oc new-project lab05

# Create PVC
cat <<EOF | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mydata
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

oc get pvc

# Deploy with volume
oc new-app --image=nginx:latest --name=web
oc set volume deployment/web \
  --add --name=mydata \
  --type=pvc --claim-name=mydata \
  --mount-path=/data

# Write data
POD=$(oc get pods -l app=web -o name | head -1)
oc exec $POD -- bash -c "echo 'My important data' > /data/test.txt"

# Delete pod
oc delete $POD

# New pod comes up, check data still there!
NEW_POD=$(oc get pods -l app=web -o name | head -1)
oc exec $NEW_POD -- cat /data/test.txt
# ✅ See "My important data" = SUCCESS

oc delete project lab05
```

---

## Lab 6 — RBAC and Users (Intermediate)
**Goal:** Control who can do what

```bash
oc new-project lab06-admin
oc new-project lab06-readonly

# Deploy an app
oc new-app --image=nginx:latest --name=web -n lab06-admin

# Create a service account for read-only access
oc create serviceaccount readonly-sa -n lab06-admin

# Give view access
oc adm policy add-role-to-user view \
  system:serviceaccount:lab06-admin:readonly-sa \
  -n lab06-admin

# Test - can readonly-sa get pods?
oc auth can-i get pods \
  --as=system:serviceaccount:lab06-admin:readonly-sa \
  -n lab06-admin
# Should return: yes

# Test - can readonly-sa delete pods?
oc auth can-i delete pods \
  --as=system:serviceaccount:lab06-admin:readonly-sa \
  -n lab06-admin
# Should return: no
# ✅ Access control working = SUCCESS

oc delete project lab06-admin lab06-readonly
```

---

## Lab 7 — Network Policy (Intermediate)
**Goal:** Control pod-to-pod traffic

```bash
oc new-project lab07

# Deploy frontend and backend
oc new-app --image=nginx:latest --name=frontend
oc new-app --image=nginx:latest --name=backend
oc label deployment backend tier=backend
oc label deployment frontend tier=frontend

# Test connectivity BEFORE policy (should work)
FRONTEND=$(oc get pods -l app=frontend -o name | head -1)
BACKEND_IP=$(oc get pod -l app=backend -o jsonpath='{.items[0].status.podIP}')
oc exec $FRONTEND -- curl -s --max-time 3 http://$BACKEND_IP
# Should return nginx page

# Apply deny-all policy
cat <<EOF | oc apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF

# Allow only frontend to backend
cat <<EOF | oc apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
EOF

# Test - frontend to backend should work
oc exec $FRONTEND -- curl -s --max-time 3 http://$BACKEND_IP
# ✅ Should return nginx page = SUCCESS

oc delete project lab07
```

---

## Lab 8 — Resource Limits and Quotas (Intermediate)
**Goal:** Control resource usage

```bash
oc new-project lab08

# Create ResourceQuota
cat <<EOF | oc apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: lab-quota
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
EOF

# Create LimitRange (default limits)
cat <<EOF | oc apply -f -
apiVersion: v1
kind: LimitRange
metadata:
  name: lab-limits
spec:
  limits:
  - default:
      cpu: 200m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
EOF

# Deploy app - will auto-get limits from LimitRange
oc new-app --image=nginx:latest --name=web

# Check quota usage
oc describe quota lab-quota
# ✅ See quota consumption = SUCCESS

oc delete project lab08
```

---

## Lab 9 — Build from Source (Intermediate)
**Goal:** S2I build from Git

```bash
oc new-project lab09

# Build from public git repo
oc new-app \
  https://github.com/openshift/nodejs-ex.git \
  --name=nodejs-app

# Watch build
oc logs -f buildconfig/nodejs-app

# Once built, expose
oc expose service nodejs-app

# Get URL
oc get route nodejs-app
curl http://$(oc get route nodejs-app -o jsonpath='{.spec.host}')
# ✅ See Node.js app running = SUCCESS

# Trigger rebuild manually
oc start-build nodejs-app
oc logs -f buildconfig/nodejs-app

oc delete project lab09
```

---

## Lab 10 — Health Checks (Intermediate)
**Goal:** Liveness and readiness probes

```bash
oc new-project lab10

# Deploy app
oc new-app --image=nginx:latest --name=web

# Add liveness probe
oc set probe deployment/web \
  --liveness \
  --get-url=http://:80/ \
  --initial-delay-seconds=30 \
  --period-seconds=10 \
  --failure-threshold=3

# Add readiness probe
oc set probe deployment/web \
  --readiness \
  --get-url=http://:80/ \
  --initial-delay-seconds=5 \
  --period-seconds=5

# Verify probes are set
oc describe deployment web | grep -A 10 "Liveness\|Readiness"
# ✅ See probe configuration = SUCCESS

oc delete project lab10
```

---

## Lab 11 — Secure Routes (HTTPS) (Intermediate)
**Goal:** HTTPS for your app

```bash
oc new-project lab11

# Deploy app
oc new-app --image=nginx:latest --name=web

# Create edge TLS route (HTTP → OpenShift → HTTPS → pod gets HTTP)
oc create route edge web-secure \
  --service=web \
  --port=80

# See the route
oc get route web-secure
# Note the HOST — it already has HTTPS!

# Test
curl https://$(oc get route web-secure -o jsonpath='{.spec.host}') -k
# ✅ HTTPS working = SUCCESS

oc delete project lab11
```

---

## Lab 12 — HPA Auto Scaling (Advanced)
**Goal:** Auto-scale based on CPU

```bash
oc new-project lab12

# Deploy app with resource requests (required for HPA)
oc new-app --image=nginx:latest --name=web
oc set resources deployment web \
  --requests=cpu=100m,memory=128Mi \
  --limits=cpu=200m,memory=256Mi

# Create HPA
cat <<EOF | oc apply -f -
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
EOF

# Check HPA
oc get hpa
oc describe hpa web-hpa
# ✅ HPA configured and watching = SUCCESS

oc delete project lab12
```

---

## Lab 13 — SCC and Service Accounts (Advanced)
**Goal:** Run privileged containers safely

```bash
oc new-project lab13

# Create dedicated service account
oc create serviceaccount privileged-sa

# Assign anyuid SCC (allows running as any user including root)
oc adm policy add-scc-to-user anyuid -z privileged-sa

# Deploy app with service account
oc new-app --image=nginx:latest --name=web
oc set serviceaccount deployment/web privileged-sa

# Verify
oc get pod -l app=web -o yaml | grep serviceAccount
# ✅ Shows privileged-sa = SUCCESS

oc delete project lab13
```

---

## Lab 14 — Complete Multi-Tier App (Advanced)
**Goal:** Deploy frontend + backend + database

```bash
oc new-project lab14

# ===== DATABASE =====
# Create secret for MySQL
oc create secret generic mysql-secret \
  --from-literal=MYSQL_ROOT_PASSWORD=rootpass \
  --from-literal=MYSQL_DATABASE=appdb \
  --from-literal=MYSQL_USER=appuser \
  --from-literal=MYSQL_PASSWORD=apppass

# Create PVC for MySQL
cat <<EOF | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

# Deploy MySQL
oc new-app --image=mysql:8 --name=mysql
oc set env deployment/mysql --from=secret/mysql-secret
oc set volume deployment/mysql \
  --add --name=data \
  --type=pvc --claim-name=mysql-pvc \
  --mount-path=/var/lib/mysql

# ===== BACKEND =====
oc new-app --image=nginx:latest --name=backend
oc set env deployment/backend \
  DB_HOST=mysql \
  DB_PORT=3306

# ===== FRONTEND =====
oc new-app --image=nginx:latest --name=frontend
oc expose service frontend

# ===== VERIFY =====
oc get all
oc get route
# ✅ All running = SUCCESS

oc delete project lab14
```

---

## Lab 15 — Troubleshooting (Advanced)
**Goal:** Fix broken deployments

```bash
oc new-project lab15

# Deploy broken app (wrong image name)
cat <<EOF | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: broken-app
  template:
    metadata:
      labels:
        app: broken-app
    spec:
      containers:
      - name: app
        image: nginx:doesnotexist999
        ports:
        - containerPort: 80
EOF

# Troubleshoot!
oc get pods
# See: ErrImagePull or ImagePullBackOff

oc describe pod $(oc get pods -l app=broken-app -o name | head -1)
# See: Failed to pull image error

# Fix it
oc set image deployment/broken-app app=nginx:latest

oc get pods -w
# ✅ Pods running = SUCCESS

oc delete project lab15
```

---

## 🏆 Final Lab — Full Production Setup
**Goal:** Everything in one lab

```bash
oc new-project production

# 1. Create quota
cat <<EOF | oc apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: prod-quota
spec:
  hard:
    pods: "20"
    requests.cpu: "4"
    requests.memory: 8Gi
EOF

# 2. Deploy app with all best practices
cat <<EOF | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
EOF

# 3. Expose with HTTPS
oc expose service myapp
oc create route edge myapp-secure --service=myapp --port=80

# 4. Add HPA
oc autoscale deployment myapp \
  --min=3 --max=10 \
  --cpu-percent=70

# 5. Verify everything
oc get all
oc get route
oc get hpa
oc describe quota prod-quota

# ✅ Production-ready deployment = SUCCESS!
oc delete project production
```

---

*Complete all 20 labs and you are ready for EX280 exam.*
*Practice each lab 3 times until you can do it from memory.*
