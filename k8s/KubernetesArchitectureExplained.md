# Kubernetes Architecture Explained

## Overview Diagram

> Insert your Kubernetes architecture diagram here for reference.

---

## High-Level Architecture

Kubernetes has **two major parts**:

1. **Control Plane** → The brain of the cluster
2. **Worker Nodes** → The machines that run your applications

```text
Cluster
├── Control Plane
│   ├── kube-apiserver
│   ├── etcd
│   ├── scheduler
│   ├── controller-manager
│   └── cloud-controller-manager
│
└── Worker Nodes
    ├── kubelet
    ├── kube-proxy
    ├── CRI / Container Runtime
    └── Pods
```

---

## Control Plane Components

### kube-apiserver

The front door of Kubernetes.

**Responsibilities:**

* Receives requests from `kubectl`
* Validates manifests
* Authenticates / Authorizes users
* Reads/Writes cluster state
* Exposes Kubernetes REST API

**Think of it as:** The central gateway of the cluster.

---

### etcd

The database of Kubernetes.

**Stores:**

* Deployments
* Pods
* Services
* Secrets
* ConfigMaps
* CRDs
* Node info

**Think of it as:** The source of truth for the cluster.

---

### scheduler

Chooses which node a Pod should run on.

**Looks at:**

* CPU / Memory availability
* Node affinity / anti-affinity
* Taints / tolerations
* Scheduling constraints

**Think of it as:** Pod placement engine.

---

### controller-manager

Runs built-in Kubernetes controllers.

**Examples:**

* Deployment Controller
* ReplicaSet Controller
* Job Controller
* Node Controller

**Think of it as:** The automation engine.

---

### cloud-controller-manager

Connects Kubernetes to cloud provider APIs.

**Handles:**

* Load Balancers
* Cloud Volumes
* Node lifecycle in cloud

**Think of it as:** Cloud integration layer.

---

## Worker Node Components

### kubelet

Agent running on every node.

**Responsibilities:**

* Watches for Pods assigned to its node
* Pulls container images
* Starts/stops containers
* Reports Pod status back

**Think of it as:** Node manager.

---

### kube-proxy

Handles Kubernetes networking.

**Responsibilities:**

* Implements Service networking
* Routes traffic to correct Pods
* Maintains iptables/ipvs rules

**Think of it as:** Network router for Services.

---

### CRI / Container Runtime

The software that actually runs containers.

**Examples:**

* containerd
* CRI-O

**Think of it as:** Container engine.

---

### Pods

The smallest deployable unit in Kubernetes.

Contains one or more containers.

---

## What Happens After `kubectl apply`?

### Example Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
```

---

### Flow

```text
1. kubectl apply
      ↓
2. API Server receives Deployment
      ↓
3. API Server stores it in etcd
      ↓
4. Deployment Controller notices new Deployment
      ↓
5. Creates ReplicaSet
      ↓
6. ReplicaSet creates Pod objects
      ↓
7. Scheduler assigns Pods to Nodes
      ↓
8. Kubelet starts containers via CRI
      ↓
9. Pods become Running
```

---

## Why Controllers Matter

Controllers continuously reconcile:

```text
Desired State vs Actual State
```

Example:

```text
Desired: 3 Pods
Actual: 2 Pods
```

Controller fixes it by creating another Pod.

---

## Why This Matters for CRDs

Custom Resources use the same architecture:

```text
Custom Resource Applied
      ↓
Stored in API Server / etcd
      ↓
Custom Controller Watches It
      ↓
Controller Reconciles Desired State
      ↓
Creates/Updates Resources
```

---

## Mental Model Summary

```text
kubectl             = Client / CLI Tool
kube-apiserver      = Front Door / API Gateway
etcd                = Database / Source of Truth
scheduler           = Pod Placement Engine
controller-manager  = Automation / Reconciliation Brain
kubelet             = Node Agent
CRI                 = Runs Containers
kube-proxy          = Networking Layer
Pods                = Running Applications
```

---

## Final Key Insight

Most Kubernetes components do **not** talk directly to each other.

They communicate through the **API Server**:

```text
Read State from API Server
Write Changes to API Server
```

This is what makes Kubernetes extensible and allows CRDs/operators to integrate naturally.
