# Kubernetes CRD & Operator Basics

## What is a CRD?

**CRD (Custom Resource Definition)** lets you extend Kubernetes with your own resource types.

Kubernetes knows built-in resources like:

* Pod
* Deployment
* Service
* Secret

A CRD adds new ones such as:

* Certificate
* Database
* KafkaCluster
* BackupPolicy

---

## What Problem Does a CRD Solve?

Without CRDs, users must manage low-level Kubernetes resources manually.

Example:

To manage a database manually you may need:

* StatefulSet
* PVC
* Service
* Secret
* Backup Job
* Monitoring Rules

With a CRD:

```yaml
apiVersion: database.mycompany.com/v1
kind: Database
metadata:
  name: mydb
spec:
  engine: postgres
  version: "15"
  storage: 20Gi
```

Now users declare intent instead of wiring infrastructure manually.

---

## CRD vs Controller

### CRD

Registers a new resource type with Kubernetes.

Example:

```yaml
kind: Certificate
```

After installing the CRD, Kubernetes accepts this resource.

**CRD only defines the type/schema.**
It does **NOT** perform any actions.

---

### Controller

The controller watches custom resources and performs automation.

It:

1. Watches for resource changes
2. Compares desired vs actual state
3. Takes action to reconcile

---

## Mental Model

```text
CRD        = Defines WHAT exists
Controller = Defines WHAT HAPPENS
```

---

## Example: cert-manager Certificate

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: my-cert
spec:
  dnsNames:
    - example.com
```

Flow:

```text
1. Certificate CRD lets Kubernetes accept kind: Certificate
2. cert-manager controller watches Certificate objects
3. Controller requests/renews certificate
4. Stores cert in Secret
```

---

## Installing an Operator / Extension

Usually installs:

```text
CRDs
+ Controller
+ RBAC
+ Webhooks
+ Services
```

---

## cert-manager Components

### cert-manager Controller

Main reconciliation engine.

Responsibilities:

* Watches Certificate resources
* Requests/Renews certificates
* Creates Secrets
* Updates Status

---

### cert-manager Webhook

Admission webhook for validation/defaulting.

Responsibilities:

* Validates CRs before storage
* Rejects invalid configs
* Applies defaults
* Handles conversion between API versions

---

### cert-manager CA Injector

Maintains CA bundles for webhook trust.

Responsibilities:

* Injects CA certificates into webhook configs
* Keeps webhook TLS trust updated

---

## End-to-End Request Flow

```text
kubectl apply Certificate
        ↓
API Server
        ↓
Webhook validates/defaults request
        ↓
Stored in etcd
        ↓
Controller sees Certificate
        ↓
Controller performs reconciliation
        ↓
Secret created/updated with certificate
```

---

## Key Takeaways

```text
CRD = New Kubernetes API Type
Custom Resource = Instance of That Type
Controller = Logic That Reconciles It
Webhook = Validates/Mutates API Requests
CA Injector = Maintains Webhook Trust Certificates
```

---

## Debugging Mindset for New CRDs

When you encounter a new CRD:

1. Find the CRD:

   ```bash
   kubectl get crd
   ```

2. Inspect the CRD:

   ```bash
   kubectl describe crd <name>
   ```

3. Find the controller/operator:

   ```bash
   kubectl get pods -A | grep <operator-name>
   ```

4. Check controller logs:

   ```bash
   kubectl logs <pod> -n <namespace>
   ```

5. Read CRD schema/docs:

   ```bash
   kubectl explain <resource>
   kubectl explain <resource>.spec
   ```

---

## Final Summary

```text
CRD extends Kubernetes API
Controller gives CRD behavior
Together they form an Operator/Extension
```
