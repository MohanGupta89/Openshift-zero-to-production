# 03 — OpenShift Fundamentals
## Where Kubernetes Ends, OpenShift Begins

---

## 🔑 OpenShift vs Kubernetes — Key Differences

| Feature | Kubernetes | OpenShift |
|---------|-----------|-----------|
| CLI | kubectl | oc (superset of kubectl) |
| Namespaces | Namespace | Project (Namespace + extras) |
| Security | Permissive by default | Strict by default |
| Ingress | Ingress object | Route object |
| Image Builds | External tools | Built-in BuildConfig + S2I |
| CI/CD | External (Jenkins etc.) | Built-in Pipelines (Tekton) |
| Registry | External | Built-in Internal Registry |
| Users | External LDAP etc. | Built-in OAuth + LDAP + more |
| Web Console | Basic dashboard | Rich developer + admin console |

---

## 📁 Projects — OpenShift's Namespace

```bash
# Project = Namespace + Default RBAC + Annotations

# Create project
oc new-project myapp \
  --display-name="My Application" \
  --description="Production app for team Alpha"

# List projects
oc get projects
oc projects              # Shows current + switch

# Switch project
oc project myapp
oc project default

# Delete project (deletes EVERYTHING inside)
oc delete project myapp

# See project details
oc describe project myapp
```

---

## 🖥️ Web Console — Your Visual Dashboard

```
Access Web Console:
- Developer Sandbox: console-openshift-console.apps.sandbox...
- Local CRC: https://console-openshift-console.apps-crc.testing

Two Views:
1. Developer View  → Deploy apps, see logs, manage workloads
2. Administrator View → Cluster management, nodes, storage, security
```

---

## 🚀 Deploy Apps in OpenShift — Multiple Ways

### Way 1: From Docker Image
```bash
# Deploy from public image
oc new-app nginx:latest --name=my-web

# Deploy from private registry
oc new-app myregistry.com/myteam/myapp:v1 --name=my-app

# See what was created
oc get all
```

### Way 2: From Source Code (S2I Magic!)
```bash
# Deploy directly from Git repo — OpenShift detects language!
oc new-app https://github.com/openshift/nodejs-ex.git \
  --name=my-node-app

# OpenShift will:
# 1. Detect it's Node.js
# 2. Pull Node.js builder image
# 3. Clone your code
# 4. Build it
# 5. Deploy it
# All automatically!

# Watch the build
oc logs -f buildconfig/my-node-app

# See all resources created
oc get all
```

### Way 3: From Dockerfile
```bash
oc new-app https://github.com/myrepo/myapp.git \
  --strategy=docker \
  --name=my-app
```

### Way 4: From YAML (like kubectl)
```bash
oc apply -f deployment.yaml
oc apply -f service.yaml
```

---

## 🌐 Routes — Expose Apps to Internet

```
Kubernetes has Ingress.
OpenShift has Routes. (Easier to use!)

Route = Public URL → Service → Pods
```

```bash
# Expose a service as a route
oc expose service my-web
# Creates URL like: my-web-myproject.apps.cluster.example.com

# See routes
oc get routes

# Create route with custom hostname
oc expose service my-web \
  --hostname=myapp.example.com

# Create secure (HTTPS) route
oc create route edge my-web-secure \
  --service=my-web \
  --port=80
```

```yaml
# route.yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: my-app-route
spec:
  host: myapp.apps.cluster.example.com
  to:
    kind: Service
    name: my-app-service
  port:
    targetPort: 8080
  tls:
    termination: edge          # HTTPS termination at router
    insecureEdgeTerminationPolicy: Redirect
```

---

## 🔨 ImageStreams — Smart Image Management

```
Problem: You update image on registry.
         How does OpenShift know to redeploy?

Solution: ImageStream!
          ImageStream watches for image changes.
          New image pushed → Auto trigger new deployment!
```

```bash
# See imagestreams
oc get imagestreams
oc get is

# See what's in an imagestream
oc describe imagestream nodejs

# Import external image into imagestream
oc import-image my-nginx \
  --from=docker.io/nginx:latest \
  --confirm

# Tag an image
oc tag nginx:latest my-nginx:production
```

---

## 🔨 BuildConfig — Build Your App Inside OpenShift

```yaml
# buildconfig.yaml
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: my-app-build
spec:
  source:
    type: Git
    git:
      uri: https://github.com/myteam/myapp.git
      ref: main
  strategy:
    type: Source          # S2I strategy
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: nodejs:18
  output:
    to:
      kind: ImageStreamTag
      name: my-app:latest
  triggers:
  - type: GitHub          # Auto build on git push
    github:
      secret: my-webhook-secret
  - type: ImageChange     # Auto build when base image updates
```

```bash
# Trigger a build manually
oc start-build my-app-build

# Watch build logs
oc logs -f buildconfig/my-app-build

# See build history
oc get builds

# Cancel a running build
oc cancel-build my-app-build-3
```

---

## 👤 Users and Authentication

```bash
# See current user
oc whoami

# See all users (admin only)
oc get users

# Add user to project
oc adm policy add-role-to-user view john -n myproject
oc adm policy add-role-to-user edit jane -n myproject
oc adm policy add-role-to-user admin bob -n myproject

# Give cluster-admin (be careful!)
oc adm policy add-cluster-role-to-user cluster-admin alice

# Remove role
oc adm policy remove-role-from-user view john -n myproject

# See who has what role
oc get rolebindings -n myproject
```

---

## 📋 DeploymentConfig vs Deployment

```
OpenShift originally had DeploymentConfig (DC).
Now standard Kubernetes Deployment is preferred.

DeploymentConfig (old way):
- OpenShift specific
- Has lifecycle hooks
- Has triggers built-in

Deployment (new way - preferred):
- Standard Kubernetes
- More portable
- Use this for new apps
```

```bash
# Old way (DeploymentConfig)
oc get dc
oc rollout latest dc/my-app
oc rollout status dc/my-app

# New way (Deployment)
oc get deployment
oc rollout status deployment/my-app
oc rollout undo deployment/my-app
```

---

## 🔧 oc vs kubectl — Differences

```bash
# These are identical:
kubectl get pods    =   oc get pods
kubectl apply -f    =   oc apply -f
kubectl logs        =   oc logs
kubectl exec        =   oc exec

# These are OpenShift only:
oc new-app          # Deploy from image/git/template
oc new-project      # Create project with extras
oc expose           # Create route from service
oc start-build      # Trigger a build
oc rollout          # Manage rollouts
oc adm              # Admin commands
oc get routes       # OpenShift Routes
oc get bc           # BuildConfigs
oc get builds       # Build history
oc get is           # ImageStreams
```

---

## 🧪 Lab 03 — Full OpenShift App Deployment

```bash
# Step 1: Login
oc login --token=YOUR_TOKEN --server=YOUR_SERVER

# Step 2: Create project
oc new-project lab03-demo

# Step 3: Deploy from image
oc new-app --image=registry.access.redhat.com/ubi8/httpd-24:latest \
  --name=my-httpd

# Step 4: Watch deployment
oc rollout status deployment/my-httpd

# Step 5: See all resources
oc get all

# Step 6: Expose as route
oc expose service my-httpd

# Step 7: Get URL and test
oc get route
curl http://$(oc get route my-httpd -o jsonpath='{.spec.host}')

# Step 8: Scale up
oc scale deployment my-httpd --replicas=3
oc get pods

# Step 9: Check resource usage
oc adm top pods

# Step 10: Update image
oc set image deployment/my-httpd \
  httpd-24=registry.access.redhat.com/ubi8/httpd-24:1-222

# Step 11: Watch rolling update
oc rollout status deployment/my-httpd

# Step 12: Rollback
oc rollout undo deployment/my-httpd

# Cleanup
oc delete project lab03-demo
```

---

## ❓ Interview Questions — OpenShift Fundamentals

**Q1: What is a Route in OpenShift?**
> Route exposes a service externally with a URL. It maps a hostname to a service. OpenShift's alternative to Kubernetes Ingress, easier to configure and with built-in TLS support.

**Q2: What is S2I (Source-to-Image)?**
> S2I builds container images directly from source code. You give it your git repo, it detects the language, injects your code into a builder image, compiles/builds it, and produces a ready-to-run container image. No Dockerfile needed.

**Q3: What is an ImageStream?**
> An OpenShift object that tracks container images and their versions. When a new image is pushed, ImageStream detects the change and can trigger automatic rebuilds or redeployments.

**Q4: What is BuildConfig?**
> A blueprint that tells OpenShift how to build your application. Defines source (git), build strategy (S2I/Docker/Custom), output image, and triggers (webhook, image change, manual).

**Q5: What is the difference between Project and Namespace?**
> Namespace is Kubernetes concept. Project is OpenShift's extension of Namespace. Project adds default RBAC, display name, description, and project request templates. All projects are namespaces, not all namespaces are projects.

**Q6: What are the Route termination types?**
> - Edge: TLS terminates at the Router. Traffic from router to pod is unencrypted.
> - Passthrough: TLS goes all the way to the pod. Router doesn't decrypt.
> - Re-encrypt: TLS terminates at router, new TLS created to pod.

**Q7: How do you do zero-downtime deployment in OpenShift?**
> Use Rolling Update strategy (default). OpenShift starts new pods, waits for readiness, then removes old pods. Combined with readiness probes ensures no traffic goes to unready pods.

---

## ➡️ Next Step

Go to [04 — Builds & Deployments](../04-builds-deployments/README.md)
