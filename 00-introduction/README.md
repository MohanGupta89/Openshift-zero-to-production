# 00 — Introduction & Setup
## Before You Write a Single Command — Understand WHY

---

## 🤔 What is OpenShift? (Explain Like I'm 5)

Imagine you run a restaurant.

- **Your food** = Your application (website, app, software)
- **The kitchen** = The server where your app runs
- **Docker** = A lunchbox that packs your food neatly so it works anywhere
- **Kubernetes** = The restaurant manager who decides which kitchen makes which food
- **OpenShift** = The ENTIRE restaurant chain — manager + security + delivery + billing + everything built-in

That's it. OpenShift = Kubernetes + everything enterprise needs, already configured.

---

## 🤔 Why Should YOU Learn OpenShift?

You have 15 years of Linux experience. You already know:
- How servers work
- How networking works
- How storage works
- How processes work

OpenShift is just these same things — but for containers. **You have 80% of the knowledge already.** You just need to learn the container layer on top.

---

## 📊 OpenShift vs Kubernetes — Simple Comparison

| Feature | Kubernetes | OpenShift |
|---------|-----------|-----------|
| Installation | Complex, manual | Automated, easy |
| Security | You configure it | Built-in, strict by default |
| CI/CD | You add it yourself | Built-in pipelines |
| Image Registry | You set it up | Built-in registry |
| Web Console | Basic | Rich, developer-friendly |
| Support | Community | Red Hat Enterprise Support |
| Used by | Startups, tech companies | Banks, Governments, Oracle, IBM |

**Bottom line:** OpenShift = Kubernetes with everything already done for you.

---

## 🏗️ OpenShift Architecture (Human Language)

```
┌─────────────────────────────────────────┐
│           YOUR APPLICATIONS             │
│        (Pods running containers)        │
├─────────────────────────────────────────┤
│              WORKER NODES               │
│     (Servers that run your apps)        │
├─────────────────────────────────────────┤
│              MASTER NODES               │
│   (Brain of the cluster — decides       │
│    where to run what)                   │
│                                         │
│  • API Server   → Receives all commands │
│  • etcd         → Stores all data       │
│  • Scheduler    → Decides placement     │
│  • Controller   → Keeps things running  │
├─────────────────────────────────────────┤
│           OPENSHIFT EXTRAS              │
│  • Web Console  → Pretty UI             │
│  • Registry     → Stores your images   │
│  • Router       → Routes traffic in    │
│  • OAuth        → Login/Auth           │
└─────────────────────────────────────────┘
```

---

## 🔧 Setup Your Free Environment (Do This Now)

### Option 1 — Red Hat Developer Sandbox (EASIEST — Recommended)

```
1. Go to: developers.redhat.com/developer-sandbox
2. Click "Start your sandbox for free"
3. Create free Red Hat account
4. Get real OpenShift cluster in 2 minutes
5. No credit card. No installation.
```

### Option 2 — OpenShift Local on Your Laptop

```bash
# Download CRC (CodeReady Containers)
# From: console.redhat.com/openshift/create/local

# After download:
crc setup          # First time setup (takes 10 mins)
crc start          # Start your local cluster
crc console        # Open web console in browser

# Get login credentials
crc console --credentials
```

### Option 3 — KillerCoda (Zero Setup)

```
Go to: killercoda.com/openshift
Pick any scenario
Start practicing immediately — no account needed
```

---

## 🖥️ Install oc CLI (Your Main Tool)

```bash
# On Linux (your home ground brother!)
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz
tar -xvf openshift-client-linux.tar.gz
sudo mv oc kubectl /usr/local/bin/

# Verify installation
oc version
kubectl version
```

---

## 🔑 Your First Login

```bash
# Login with token (from Developer Sandbox)
oc login --token=YOUR_TOKEN --server=YOUR_SERVER_URL

# OR login with username/password (local cluster)
oc login -u developer -p developer https://api.crc.testing:6443

# Check who you are
oc whoami

# See your projects
oc get projects

# See cluster info
oc cluster-info
```

---

## 📁 Key Concepts Glossary (Simple English)

| Term | What It Means in Human Language |
|------|---------------------------------|
| **Pod** | Smallest unit — one or more containers running together |
| **Node** | A server (physical or VM) in your cluster |
| **Project** | Like a folder — groups your apps and resources |
| **Namespace** | Same as Project (Kubernetes name for it) |
| **Service** | Stable address to reach your pods |
| **Route** | Public URL to access your app from internet |
| **Deployment** | Instructions for how to run your app |
| **Image** | Packaged version of your app (like a zip file) |
| **Registry** | Warehouse that stores your images |
| **Operator** | Robot that manages complex apps automatically |
| **RBAC** | Who can do what — permissions system |
| **PVC** | Request for storage space |
| **ConfigMap** | Store configuration (non-secret) |
| **Secret** | Store passwords and sensitive data |

---

## ✅ Setup Checklist

```
□ Red Hat account created
□ Developer Sandbox active
□ oc CLI installed
□ Successfully logged in with: oc login
□ oc whoami returns your username
□ oc get projects shows your projects
□ Web console accessible in browser
```

---

## 🎯 Lab 00 — Your First Commands

```bash
# 1. Login
oc login --token=YOUR_TOKEN --server=YOUR_SERVER

# 2. Who am I?
oc whoami

# 3. What projects exist?
oc get projects

# 4. Create your first project
oc new-project my-first-project

# 5. Switch to your project
oc project my-first-project

# 6. Deploy your first app (one command!)
oc new-app httpd --name=my-first-app

# 7. Watch it come up
oc get pods -w

# 8. Expose it to internet
oc expose service my-first-app

# 9. Get the URL
oc get route

# 10. Open in browser!
# Copy the HOST/PORT value and open in browser
```

**If you see a webpage — you just deployed your first OpenShift app! 🎉**

---

## ❓ Interview Questions — Introduction Level

**Q1: What is OpenShift?**
> OpenShift is Red Hat's enterprise container platform built on top of Kubernetes. It adds built-in CI/CD, security, image registry, web console, and developer tools that you would otherwise have to configure yourself in plain Kubernetes.

**Q2: How is OpenShift different from Kubernetes?**
> Kubernetes is the engine. OpenShift is the full car. OpenShift adds security policies, integrated pipelines, developer console, built-in registry, and enterprise support on top of Kubernetes.

**Q3: What are the types of OpenShift?**
> - OpenShift Container Platform (OCP) — On-premise, self-managed
> - OpenShift Dedicated — Managed by Red Hat on cloud
> - OpenShift Online — Public cloud version
> - OpenShift Local (CRC) — For local development
> - Azure Red Hat OpenShift (ARO) — Managed on Azure
> - Red Hat OpenShift on AWS (ROSA) — Managed on AWS

**Q4: What is the oc command?**
> `oc` is the OpenShift CLI tool. It is like `kubectl` but with extra OpenShift-specific commands. Almost everything you can do in the web console, you can do with `oc`.

---

## ➡️ Next Step

Go to [01 — Containers & Docker](../01-containers-docker/README.md)
