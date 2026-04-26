# 07 — Security & RBAC
## Who Can Do What — The Complete Guide

---

## 🔐 Why Security Matters in OpenShift

OpenShift is **strict by default**. Unlike Kubernetes which is permissive, OpenShift:
- Runs containers as non-root by default
- Blocks privileged containers
- Enforces network isolation
- Requires explicit permission grants

This is WHY enterprises choose OpenShift. And why knowing security = architect level.

---

## 👥 RBAC — Role Based Access Control

```
RBAC = Who (Subject) can do What (Verb) on Which (Resource)

Subject = User / Group / ServiceAccount
Verb    = get, list, create, update, delete, watch, patch
Resource = pods, deployments, services, secrets, etc.
```

### Core RBAC Objects

```
Role          → Permissions in ONE namespace
ClusterRole   → Permissions across ALL namespaces

RoleBinding        → Assigns Role to Subject in ONE namespace
ClusterRoleBinding → Assigns ClusterRole to Subject cluster-wide
```

### Built-in OpenShift Roles

| Role | What They Can Do |
|------|-----------------|
| `view` | Read-only. See everything, change nothing |
| `edit` | Create/modify resources. Cannot change RBAC |
| `admin` | Full project control including RBAC |
| `cluster-admin` | God mode. Full cluster control |

---

## 🔧 RBAC Commands

```bash
# ===== GRANT ACCESS =====

# Give user view access to a project
oc adm policy add-role-to-user view alice -n myproject

# Give user edit access
oc adm policy add-role-to-user edit bob -n myproject

# Give user admin of a project
oc adm policy add-role-to-user admin carol -n myproject

# Give cluster-admin (CAREFUL!)
oc adm policy add-cluster-role-to-user cluster-admin dave

# Give group access
oc adm policy add-role-to-group view developers -n myproject


# ===== REMOVE ACCESS =====

oc adm policy remove-role-from-user view alice -n myproject
oc adm policy remove-cluster-role-from-user cluster-admin dave


# ===== VIEW ACCESS =====

# See role bindings in project
oc get rolebindings -n myproject
oc describe rolebinding -n myproject

# See cluster role bindings
oc get clusterrolebindings

# Check if user can do something
oc auth can-i get pods -n myproject
oc auth can-i get pods --as=alice -n myproject
oc auth can-i delete secrets --as=bob -n myproject
```

---

## 📝 Create Custom Roles

```yaml
# custom-role.yaml — Read pods and logs only
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: myproject
rules:
- apiGroups: [""]                    # "" = core API group
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list"]
```

```bash
# Create the role
oc apply -f custom-role.yaml

# Bind to user
oc create rolebinding pod-reader-binding \
  --role=pod-reader \
  --user=alice \
  -n myproject

# Bind to service account
oc create rolebinding pod-reader-sa-binding \
  --role=pod-reader \
  --serviceaccount=myproject:mysa \
  -n myproject
```

---

## 🔑 Service Accounts

```bash
# Create service account
oc create serviceaccount myapp-sa -n myproject

# Give it permissions
oc adm policy add-role-to-user view \
  -z myapp-sa \          # -z = service account in current project
  -n myproject

# Use in deployment
oc set serviceaccount deployment/myapp myapp-sa

# Get service account token
oc create token myapp-sa -n myproject
```

---

## 🛡️ SCC — Security Context Constraints

```
SCC = OpenShift's way to control what security powers a pod can use.
Like RBAC but for pod-level security.

Kubernetes has Pod Security Admission — OpenShift extends with SCC.
```

### Built-in SCCs (from most to least restrictive)

| SCC | Description |
|-----|-------------|
| `restricted` | Default. No root. No host resources. |
| `restricted-v2` | Same as restricted, newer version |
| `nonroot` | Must run as non-root user |
| `anyuid` | Can run as any UID including root |
| `hostnetwork` | Can use host network namespace |
| `hostmount-anyuid` | Can mount host paths |
| `privileged` | Everything allowed. DANGEROUS. |

```bash
# See all SCCs
oc get scc

# See details of an SCC
oc describe scc restricted

# See which SCC a pod uses
oc get pod mypod -o yaml | grep scc
oc describe pod mypod | grep scc

# Assign SCC to service account
oc adm policy add-scc-to-user anyuid -z mysa -n myproject

# Assign SCC to all service accounts in namespace
oc adm policy add-scc-to-group anyuid system:serviceaccounts:myproject

# Remove SCC
oc adm policy remove-scc-from-user anyuid -z mysa -n myproject

# Check which SCCs a user can use
oc adm policy who-can use scc/anyuid
```

---

## 🔒 Security Context in Pod

```yaml
spec:
  securityContext:           # Pod-level security
    runAsUser: 1000          # Run as this user ID
    runAsGroup: 1000         # Run as this group ID
    fsGroup: 1000            # Volume files owned by this group
    runAsNonRoot: true       # Must not run as root

  containers:
  - name: myapp
    image: myapp:v1
    securityContext:         # Container-level security
      runAsUser: 1000
      readOnlyRootFilesystem: true    # Can't write to /
      allowPrivilegeEscalation: false # Can't become more privileged
      capabilities:
        drop: ["ALL"]        # Drop all Linux capabilities
        add: ["NET_BIND_SERVICE"]  # Add only what needed
```

---

## 🌐 Network Policies

```bash
# By default: all pods can talk to all pods
# NetworkPolicy adds firewall rules between pods
```

### Common Patterns

```yaml
# 1. DENY ALL traffic (safest default)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: myproject
spec:
  podSelector: {}    # Apply to all pods
  policyTypes:
  - Ingress
  - Egress
```

```yaml
# 2. Allow only from specific namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend-ns
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend-namespace
```

```yaml
# 3. Allow specific port only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-http-only
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - ports:
    - port: 80
    - port: 443
```

```yaml
# 4. Allow specific pod to specific pod
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend          # This policy protects backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend     # Only frontend can reach backend
    ports:
    - port: 8080
```

---

## 🔐 Secrets Management

```bash
# Create secrets
oc create secret generic db-secret \
  --from-literal=password=mypassword \
  --from-literal=username=admin

# Create TLS secret
oc create secret tls my-tls \
  --cert=tls.crt \
  --key=tls.key

# Create Docker registry secret
oc create secret docker-registry registry-secret \
  --docker-server=registry.example.com \
  --docker-username=admin \
  --docker-password=secret \
  --docker-email=admin@example.com

# Link registry secret to service account
oc secrets link default registry-secret --for=pull

# Use in pod as env var
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password

# Use in pod as volume (mounted as file)
volumes:
- name: secret-vol
  secret:
    secretName: db-secret
volumeMounts:
- name: secret-vol
  mountPath: /etc/secrets
  readOnly: true
```

---

## ❓ Interview Questions — Security & RBAC

**Q: What is RBAC?**
> Role-Based Access Control. Controls who can perform which actions on which resources. Role = permissions. RoleBinding = assigns role to user/group/SA.

**Q: Role vs ClusterRole?**
> Role = permissions scoped to one namespace. ClusterRole = permissions across all namespaces or for cluster-level resources (nodes, PVs).

**Q: What is an SCC?**
> Security Context Constraint. OpenShift-specific. Controls security-sensitive settings for pods: can they run as root, use host network, mount host paths, etc.

**Q: Default SCC in OpenShift?**
> `restricted`. Pods cannot run as root. Cannot mount host paths. Cannot use host network. Containers get random UID assigned.

**Q: How to run an image that requires root?**
> Create service account. Assign `anyuid` SCC to it. Use that service account in the deployment. Never do this for all pods — only specific ones that need it.

**Q: What is a ServiceAccount?**
> Non-human identity for pods. Used when pods need to interact with Kubernetes/OpenShift API or need specific permissions. Assign RBAC and SCCs to service accounts.

**Q: How does NetworkPolicy work?**
> Firewall rules for pods. By default all pod traffic is allowed. NetworkPolicy adds allow/deny rules based on labels, namespaces, ports. Additive model — multiple policies combine.

---

## ➡️ Next Step

Go to [08 — CI/CD & GitOps](../08-cicd-gitops/README.md)
