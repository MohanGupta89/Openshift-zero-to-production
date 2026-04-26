# 13 — Complete Interview Questions
## Zero to Architect Level — All Questions in One Place

---

## 🟢 BEGINNER LEVEL (Fresher / 0-2 years)

### Containers & Docker

**Q: What is a container?**
> Lightweight, standalone package that includes app + all dependencies + OS libraries needed to run it. Runs consistently across any environment.

**Q: Docker image vs Docker container?**
> Image = blueprint (static, stored on disk). Container = running instance of image (active, has its own process).

**Q: What is Docker Hub?**
> Public registry for Docker images. Anyone can push/pull images. Like GitHub but for containers.

**Q: What is a Dockerfile?**
> Text file with instructions to build Docker image. Each instruction creates a layer. Cached for fast rebuilds.

**Q: How do you persist data in Docker?**
> Use Volumes. Container filesystem is ephemeral — volumes exist outside container lifecycle. Mount volume at container path.

---

### Kubernetes Basics

**Q: What is Kubernetes?**
> Container orchestration platform. Manages deployment, scaling, and health of containers across multiple servers. Self-heals, auto-scales, rolling updates.

**Q: What is a Pod?**
> Smallest deployable unit. One or more containers sharing network namespace and storage. Usually one app container per pod.

**Q: Deployment vs Pod?**
> Pod = just a running container. Deployment = controller that manages pods, ensures replicas, handles updates/rollbacks. Always use Deployment not bare Pod.

**Q: What is a Service?**
> Stable network endpoint for pods. Pods get new IPs on restart — Service provides fixed DNS/IP that load-balances to healthy pods.

**Q: Service types?**
> ClusterIP (internal only), NodePort (accessible on node IP:port), LoadBalancer (external LB on cloud), ExternalName (DNS alias).

**Q: What is a Namespace?**
> Virtual cluster within cluster. Isolates resources. Teams/environments get separate namespaces.

**Q: What is etcd?**
> Distributed key-value store. Brain of Kubernetes. Stores entire cluster state. If etcd goes down, cluster can't make decisions.

**Q: Liveness vs Readiness probe?**
> Liveness = is container alive? Fails → restart it. Readiness = is container ready for traffic? Fails → remove from service endpoints.

**Q: What is a ConfigMap?**
> Store non-sensitive config as key-value. Mount as env vars or volume files. Decouples config from image.

**Q: What is a Secret?**
> Store sensitive data (passwords, tokens). Base64 encoded (NOT encrypted by default). Mount as env vars or files.

---

### OpenShift Basics

**Q: What is OpenShift?**
> Red Hat's enterprise Kubernetes platform. Adds security, CI/CD, image registry, web console, S2I builds, OAuth. More opinionated and production-ready than vanilla Kubernetes.

**Q: OpenShift vs Kubernetes?**
> Kubernetes = engine. OpenShift = full car. OpenShift adds security policies, developer tools, built-in registry, integrated pipelines on top of Kubernetes.

**Q: What is oc?**
> OpenShift CLI tool. Superset of kubectl. Has all kubectl commands plus OpenShift-specific ones (new-app, expose, start-build, etc.)

**Q: What is a Route?**
> OpenShift object that exposes service externally with a URL/hostname. Alternative to Kubernetes Ingress. Simpler to use, built-in TLS options.

**Q: What is S2I?**
> Source-to-Image. Builds container image directly from source code. Detects language, uses builder image, injects code, compiles, produces ready image. No Dockerfile needed.

**Q: What is an ImageStream?**
> Tracks container images and versions. Watches for image changes. Can trigger automatic builds/deployments when new image is pushed.

**Q: Project vs Namespace?**
> Namespace = Kubernetes concept. Project = OpenShift extension of namespace with RBAC defaults, display name, description. All projects are namespaces.

---

## 🟡 INTERMEDIATE LEVEL (2-5 years)

### Builds & Deployments

**Q: What are build strategies in OpenShift?**
> Source (S2I) — from source code using builder image. Docker — using Dockerfile. Custom — custom build process. Pipeline — Tekton/Jenkins pipeline.

**Q: What is a BuildConfig?**
> Blueprint for building apps. Defines: source (git), strategy (S2I/Docker), output image location, triggers (webhook/image change/manual).

**Q: What are build triggers?**
> Webhook — triggered by git push (GitHub/GitLab webhook). ImageChange — triggered when base image updates. ConfigChange — triggered when BuildConfig itself changes.

**Q: Rolling vs Recreate deployment?**
> Rolling = gradual replacement (zero downtime, requires double resources momentarily). Recreate = kill all old, create all new (downtime, simpler).

**Q: Blue-Green deployment?**
> Two identical environments. Blue = current live. Green = new version. Test green, then switch route from blue to green. Instant switch, easy rollback.

**Q: Canary deployment?**
> Route small % of traffic to new version. Monitor. If healthy, gradually increase %. OpenShift can split traffic using route weights.

```bash
# Traffic splitting in OpenShift
oc set route-backends myroute \
  my-app-v1=90 my-app-v2=10
```

**Q: What are resource requests and limits?**
> Request = minimum guaranteed resources (scheduler uses this). Limit = maximum allowed resources (container killed if exceeded). Always set both for production.

**Q: What is a ResourceQuota?**
> Namespace-level limits on total resources. Prevents one team from consuming all cluster resources. Limits total pods, CPU, memory, PVCs etc.

**Q: What is a LimitRange?**
> Sets default and max resource limits per pod/container in namespace. Applied automatically if pod doesn't specify limits.

---

### Networking

**Q: How does pod networking work?**
> Every pod gets unique IP. All pods can communicate with all other pods (no NAT). CNI plugin handles network setup. OpenShift uses OVN-Kubernetes by default.

**Q: What is a NetworkPolicy?**
> Controls pod-to-pod traffic. By default all pods can talk to all pods. NetworkPolicy adds rules: allow only frontend to talk to backend, deny all else.

**Q: Route termination types?**
> Edge: TLS ends at router, HTTP to pod. Passthrough: TLS goes to pod, router doesn't decrypt. Re-encrypt: TLS to router, new TLS to pod.

**Q: What is OVN-Kubernetes?**
> OpenShift's default CNI (Container Network Interface). Handles pod networking, network policies, load balancing.

**Q: What is a Service Mesh?**
> Network of microservices + tools for managing communication between them. Handles: traffic management, mTLS security, observability, retries. OpenShift uses Istio-based Service Mesh.

---

### Storage

**Q: PV vs PVC?**
> PersistentVolume (PV) = actual storage resource created by admin. PersistentVolumeClaim (PVC) = request for storage by developer. Like hotel room (PV) vs booking (PVC).

**Q: Static vs Dynamic provisioning?**
> Static: Admin creates PVs manually. Developer creates PVC, K8s finds matching PV. Dynamic: StorageClass automatically creates PV when PVC is created. Dynamic is preferred.

**Q: What is a StorageClass?**
> Template for dynamically creating storage. Defines: provisioner (AWS EBS, GCP PD, NFS), parameters, reclaim policy. Different classes for SSD vs HDD, fast vs slow.

**Q: Storage access modes?**
> ReadWriteOnce (RWO) = mounted read-write by single node. ReadOnlyMany (ROX) = mounted read-only by many nodes. ReadWriteMany (RWX) = mounted read-write by many nodes.

**Q: Reclaim policy?**
> Retain = keep PV and data after PVC deleted (manual cleanup). Delete = automatically delete PV and underlying storage. Recycle = deprecated.

---

### Security

**Q: What is RBAC?**
> Role-Based Access Control. Controls who can do what. Role = set of permissions. RoleBinding = assigns Role to User/Group. ClusterRole = cluster-wide. ClusterRoleBinding = binds ClusterRole.

**Q: Built-in OpenShift roles?**
> view = read-only access. edit = create/modify resources, no RBAC changes. admin = full project control including RBAC. cluster-admin = God mode.

**Q: What is SCC (Security Context Constraint)?**
> OpenShift-specific. Controls what security privileges pods can request. Like RBAC but for pod security. anyuid, restricted, privileged are common SCCs.

**Q: What is the restricted SCC?**
> Default SCC. Pods cannot run as root. Cannot mount host paths. Cannot use host network/ports. Most secure default.

**Q: How to run a container as root in OpenShift?**
> Bad practice. If necessary: create service account, assign anyuid SCC to it, use that service account in pod. Never use default service account for privileged work.

**Q: What is a ServiceAccount?**
> Non-human identity for pods. Pods authenticate to API server as their ServiceAccount. Assign RBAC permissions to ServiceAccount.

---

## 🔴 ADVANCED LEVEL (5+ years / Senior)

### CI/CD & GitOps

**Q: What is Tekton?**
> Kubernetes-native CI/CD framework. Cloud-native, runs pipelines as pods. OpenShift Pipelines = Tekton. Defined as Kubernetes YAML (Pipeline, Task, PipelineRun).

**Q: Tekton core components?**
> Task = series of steps (each step = container). Pipeline = series of tasks. PipelineRun = one execution of pipeline. TaskRun = one execution of task. Workspace = shared storage.

**Q: What is ArgoCD / GitOps?**
> GitOps = Git is single source of truth. ArgoCD watches Git repo. Any change in Git → automatically applied to cluster. Drift detection and auto-sync.

**Q: Jenkins vs Tekton?**
> Jenkins = traditional, runs on JVM, plugin-based, not cloud-native. Tekton = cloud-native, runs as pods, Kubernetes-native, no central server, scales better.

**Q: What is a Webhook?**
> HTTP callback. Git sends POST request to OpenShift when code is pushed. OpenShift receives it and triggers build automatically.

---

### Operators

**Q: What is an Operator?**
> Software that automates management of complex apps using Kubernetes API. Encodes human operational knowledge. Handles: install, upgrade, backup, scaling of complex apps.

**Q: What is OLM (Operator Lifecycle Manager)?**
> Manages operators themselves. Handles operator installation, upgrades, dependency resolution. Ships with OpenShift.

**Q: What is a CRD (Custom Resource Definition)?**
> Extends Kubernetes API with custom objects. Operator defines CRD (e.g., Kafka, Elasticsearch). You create instances of CRD and Operator manages them.

**Q: Operator maturity levels?**
> Level 1: Basic Install. Level 2: Seamless Upgrades. Level 3: Full Lifecycle (backup/restore). Level 4: Deep Insights (monitoring). Level 5: Auto Pilot (auto-scaling, tuning).

---

### Monitoring & Logging

**Q: OpenShift monitoring stack components?**
> Prometheus = metrics collection and storage. Alertmanager = routing and sending alerts. Grafana = visualization dashboards. kube-state-metrics = Kubernetes object metrics.

**Q: What is Prometheus?**
> Pull-based monitoring system. Scrapes metrics from endpoints. Uses PromQL query language. Stores time-series data.

**Q: OpenShift logging stack?**
> Fluentd/Vector = collects logs from pods. Elasticsearch = stores and indexes logs. Kibana = visualizes and searches logs. Together called EFK/ELK stack.

**Q: What is a ServiceMonitor?**
> CRD that tells Prometheus which services to scrape for metrics. Operator pattern for configuring Prometheus monitoring.

---

### Architect Level

**Q: OpenShift installation methods?**
> IPI (Installer Provisioned Infrastructure) = automated, installer creates everything. UPI (User Provisioned Infrastructure) = manual infrastructure, then install OpenShift.

**Q: What is Multi-cluster management?**
> Managing multiple OpenShift clusters from one place. Red Hat Advanced Cluster Management (RHACM) provides this. Deploy apps across clusters, policy enforcement, observability.

**Q: What is etcd backup and restore?**
> Critical for disaster recovery. Back up etcd database regularly. In disaster, restore etcd to recover entire cluster state.

```bash
# Backup etcd (run on master node)
/usr/local/bin/cluster-backup.sh /home/core/backup

# Files created:
# snapshot_<date>.db — etcd snapshot
# static_kuberesources_<date>.tar.gz — static pod resources
```

**Q: OpenShift upgrade strategy?**
> Cluster Version Operator (CVO) manages upgrades. Update one minor version at a time (4.12 → 4.13 → 4.14). Never skip versions. Worker nodes updated via MachineConfigOperator.

**Q: What is MCO (MachineConfig Operator)?**
> Manages OS-level configuration on nodes. Apply kernel args, systemd units, files to nodes declaratively. Nodes rebooted to apply changes.

**Q: What is cluster capacity planning?**
> Calculate required nodes based on: total pod resource requests, node capacity, overhead (OS, OpenShift components), HA requirements, growth headroom (20-30%).

**Q: What is pod disruption budget?**
> Ensures minimum pods stay running during voluntary disruptions (upgrades, drains). minAvailable or maxUnavailable setting. Prevents all pods being down at once.

**Q: What is node affinity/anti-affinity?**
> Rules that influence pod placement. Affinity = prefer/require certain nodes. Anti-affinity = avoid certain nodes (spread pods across failure domains).

**Q: How to handle certificates in OpenShift?**
> Cert Manager Operator automates certificate lifecycle. Let's Encrypt integration. Wildcard certs for apps. Custom CA for internal PKI.

---

## 🎯 Scenario-Based Questions (Senior Level)

**Q: App is crashing in OpenShift. How do you troubleshoot?**
```
1. oc get pods → see pod status (CrashLoopBackOff, OOMKilled, etc.)
2. oc describe pod <name> → see events, error messages
3. oc logs <pod> → see app logs
4. oc logs --previous <pod> → see logs from crashed container
5. oc exec -it <pod> -- bash → go inside and investigate
6. Check resource limits — OOMKilled means out of memory
7. Check liveness probe — too aggressive? Wrong path?
8. oc get events --sort-by=.metadata.creationTimestamp
```

**Q: How to debug networking issue between two pods?**
```
1. oc exec -it pod-a -- curl http://service-b:port → test service
2. oc exec -it pod-a -- nslookup service-b → test DNS
3. oc get networkpolicy → check if NetworkPolicy blocking traffic
4. oc get svc → verify service selector matches pod labels
5. oc get endpoints service-b → verify pods are in endpoints
6. Check SCC — might block network capabilities
```

**Q: Node is NotReady. What do you do?**
```
1. oc get nodes → identify node
2. oc describe node <node> → check conditions and events
3. SSH to node → check kubelet: systemctl status kubelet
4. journalctl -u kubelet → kubelet logs
5. df -h → check disk space (full disk = node issues)
6. free -m → check memory
7. oc adm drain <node> → move pods to other nodes
8. oc adm cordon <node> → prevent new pods
```

---

## 📝 Quick Reference Commands for Interview

```bash
# Most used commands — know these by heart
oc get all -n <project>
oc logs -f <pod>
oc describe pod <pod>
oc exec -it <pod> -- bash
oc get events
oc rollout status deployment/<name>
oc rollout undo deployment/<name>
oc scale deployment <name> --replicas=3
oc set image deployment/<name> <container>=<new-image>
oc expose service <name>
oc get routes
oc adm top nodes
oc adm top pods
oc get pvc
oc get pv
oc get rolebindings -n <project>
```

---

*Practice every command in Red Hat Developer Sandbox — free at developers.redhat.com/developer-sandbox*
