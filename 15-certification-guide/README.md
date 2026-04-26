# 15 — EX280 Certification Guide
## Red Hat Certified Specialist in OpenShift Administration

---

## 🎯 What is EX280?

EX280 = Red Hat's OpenShift Administration exam.

**This ONE certification can take you from 30 LPA to 50-70 LPA.**

- 100% hands-on exam (no multiple choice — real cluster, real tasks)
- 3 hours duration
- Passing score: 210/300 (70%)
- Valid for 3 years

---

## 📋 Official Exam Topics

### 1. Manage OpenShift Container Platform
```bash
# Login and basic navigation
oc login --token=TOKEN --server=SERVER
oc whoami
oc get nodes
oc get clusterversion
oc describe clusterversion

# Check cluster operators
oc get clusteroperators
oc get co

# Check cluster health
oc adm top nodes
oc get events -A
```

### 2. Deploy Applications
```bash
# Deploy from image
oc new-app --image=nginx:latest --name=myapp -n myproject

# Deploy from git
oc new-app https://github.com/repo/app.git --name=myapp

# From YAML
oc apply -f app.yaml

# Expose
oc expose service myapp
oc get routes

# Scale
oc scale deployment myapp --replicas=3

# Set resources
oc set resources deployment myapp \
  --requests=cpu=100m,memory=128Mi \
  --limits=cpu=500m,memory=256Mi
```

### 3. Manage Storage
```bash
# Create PVC
cat <<EOF | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: gp2
EOF

# See PVCs
oc get pvc
oc get pv

# Mount PVC in deployment
oc set volume deployment/myapp \
  --add \
  --name=data \
  --type=persistentVolumeClaim \
  --claim-name=my-pvc \
  --mount-path=/data
```

### 4. Configure Networking
```bash
# Create route
oc expose service myapp --hostname=myapp.example.com

# Secure route (edge TLS)
oc create route edge myapp-secure \
  --service=myapp \
  --hostname=myapp.example.com

# Network policy - deny all
cat <<EOF | oc apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF

# Network policy - allow specific
cat <<EOF | oc apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
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
```

### 5. Configure Pod Scheduling
```bash
# Node selector - run on specific node
oc patch deployment myapp -p \
  '{"spec":{"template":{"spec":{"nodeSelector":{"disktype":"ssd"}}}}}'

# Taint a node
oc adm taint nodes node1 special=true:NoSchedule

# Toleration in pod
spec:
  tolerations:
  - key: "special"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"

# Resource limits
oc set resources deployment myapp \
  --limits=cpu=200m,memory=256Mi \
  --requests=cpu=100m,memory=128Mi

# LimitRange for namespace
cat <<EOF | oc apply -f -
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-cpu-limits
spec:
  limits:
  - default:
      memory: 256Mi
      cpu: 200m
    defaultRequest:
      memory: 128Mi
      cpu: 100m
    type: Container
EOF

# ResourceQuota
cat <<EOF | oc apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: project-quota
spec:
  hard:
    pods: "20"
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
EOF
```

### 6. Configure OpenShift Builds
```bash
# Create build from git
oc new-build https://github.com/repo/app.git \
  --name=myapp \
  --strategy=source \
  --image-stream=nodejs:16

# Start build
oc start-build myapp

# Watch logs
oc logs -f buildconfig/myapp

# List builds
oc get builds

# Cancel build
oc cancel-build myapp-3
```

### 7. Manage Users and Policies
```bash
# Add user to project
oc adm policy add-role-to-user view user1 -n myproject
oc adm policy add-role-to-user edit user2 -n myproject
oc adm policy add-role-to-user admin user3 -n myproject

# Add cluster role
oc adm policy add-cluster-role-to-user cluster-admin admin-user

# Create custom role
cat <<EOF | oc apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: myproject
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
EOF

# Bind role
oc create rolebinding pod-reader-binding \
  --role=pod-reader \
  --user=user1 \
  -n myproject

# See who has access
oc get rolebindings -n myproject
oc get clusterrolebindings

# Check permissions (can-i)
oc auth can-i get pods
oc auth can-i get pods --as=user1 -n myproject
```

### 8. Security Context Constraints
```bash
# See all SCCs
oc get scc

# See which SCC a pod uses
oc get pod mypod -o yaml | grep scc

# Add SCC to service account
oc adm policy add-scc-to-user anyuid -z myserviceaccount -n myproject

# Remove SCC
oc adm policy remove-scc-from-user anyuid -z myserviceaccount -n myproject

# Create service account
oc create serviceaccount mysa -n myproject

# Use service account in deployment
oc set serviceaccount deployment/myapp mysa
```

---

## ⏱️ Exam Tips — How to Pass

### Time Management
```
3 hours for ~20 tasks.
= 9 minutes per task average.

Easy tasks (deploy app, create route) → 3-5 mins
Medium tasks (RBAC, NetworkPolicy) → 10-15 mins
Hard tasks (builds, storage) → 15-20 mins

Do easy tasks first. Mark hard ones, come back.
```

### Must Know by Heart
```bash
# These will definitely be in exam:

# 1. Deploy an app
oc new-app --image=IMAGE --name=NAME -n PROJECT

# 2. Create and expose route
oc expose service NAME
oc create route edge NAME --service=SERVICE

# 3. Scale
oc scale deployment NAME --replicas=N

# 4. Set resources
oc set resources deployment NAME \
  --requests=cpu=Xm,memory=XMi \
  --limits=cpu=Xm,memory=XMi

# 5. Add user to project
oc adm policy add-role-to-user ROLE USER -n PROJECT

# 6. Add SCC to service account
oc adm policy add-scc-to-user SCC -z SA -n PROJECT

# 7. Create PVC and mount
oc set volume deployment/NAME --add --name=VOL \
  --type=pvc --claim-name=PVC --mount-path=/PATH

# 8. Network policy
# Know YAML by heart

# 9. ResourceQuota and LimitRange
# Know YAML structure

# 10. Troubleshoot
oc get pods
oc describe pod NAME
oc logs NAME
oc get events
```

### Common Mistakes to Avoid
```
❌ Wrong namespace — always check/set namespace
   oc project myproject  OR  -n myproject

❌ Forgetting to verify — always check after doing
   oc get pods
   oc get routes
   curl the URL

❌ YAML indentation errors — use spaces not tabs

❌ Not reading question fully — read twice before doing

❌ Spending too long on one question — skip and return
```

---

## 🗓️ 8-Week Study Plan

```
Week 1: Containers + Docker (Lab every day)
Week 2: Kubernetes basics (Lab every day)
Week 3: OpenShift fundamentals (Developer Sandbox)
Week 4: Builds + Deployments (Do 10 builds)
Week 5: Networking + Storage (Create 20 PVCs, 20 routes)
Week 6: Security RBAC + SCC (Create 30 role bindings)
Week 7: CI/CD + Monitoring (Set up full pipeline)
Week 8: Mock exam + weak areas (Time yourself)

Daily: 2 hours study + 30 mins practice commands
```

---

## 🆓 Free Practice Resources

```
1. Red Hat Developer Sandbox
   developers.redhat.com/developer-sandbox
   Free real OpenShift cluster — 30 days renewable

2. KillerCoda OpenShift Labs
   killercoda.com/openshift
   Free browser-based labs, no setup

3. Red Hat Learning Portal (Free tier)
   learn.redhat.com
   Free intro courses

4. OpenShift Interactive Learning
   learn.openshift.com
   Structured free scenarios

5. Official Docs
   docs.openshift.com
   Best reference for exam
```

---

## 💰 Cost

```
Exam fee: ~$400 USD (₹33,000 approx)

Worth it? YES.
30 LPA → 50-70 LPA = ₹20-40 LPA increase per year
Exam cost = ROI in first month of new salary
```

---

*Start with Developer Sandbox today. Practice every command in this guide. You will be ready in 8 weeks.*
