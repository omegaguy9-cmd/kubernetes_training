# Kubernetes Workload Controllers: ReplicaSets, Deployments, and Rollouts

This repository covers the fundamental lifecycle managers in Kubernetes. While Pods are the atomic unit of scheduling, they are ephemeral. To ensure high availability, scalability, and seamless updates, Kubernetes uses higher-level controllers. 

This guide is heavily optimized for the **Certified Kubernetes Administrator (CKA)** exam, focusing on architectural understanding and imperative/declarative commands.

---

## 📋 Table of Contents
1. [The Evolution: ReplicationController vs. ReplicaSet](#1-the-evolution-replicationcontroller-vs-replicaset)
2. [Deployments: The Production Standard](#2-deployments-the-production-standard)
3. [Creating & Scaling Deployments](#3-creating--scaling-deployments)
4. [Performing Rolling Updates](#4-performing-rolling-updates)
5. [Performing Rollbacks (Undo)](#5-performing-rollbacks-undo)
6. [CKA Exam Quick-Reference Cheat Sheet](#6-cka-exam-quick-reference-cheat-sheet)

---

## 1. The Evolution: ReplicationController vs. ReplicaSet

Both of these controllers serve the same primary purpose: **Ensure that a specified number of Pod replicas are running at any given time.** If a node crashes or a pod is deleted, the controller immediately spins up a replacement.

### ReplicationController (Legacy)
* **Status:** Deprecated / Obsolete.
* **Selector Type:** **Equality-based** only (e.g., `app = frontend`).
* **Note:** You will rarely see this in modern Kubernetes, but it is important to know it was the predecessor to the ReplicaSet.

### ReplicaSet (Modern)
* **Status:** Active (but rarely created directly).
* **Selector Type:** **Set-based** (e.g., `app in (frontend, backend)` or `tier doesNotExist`).
* **How it works:** It uses `matchLabels` and `matchExpressions` to track which pods it owns. If the count drops below the `replicas` specification, it creates more. If it goes above, it terminates the excess.

> ⚠️ **CKA Exam Tip:** You should **never** create a ReplicaSet directly in production or in the exam unless explicitly asked to. Instead, you create a **Deployment**, which automatically creates and manages the ReplicaSet for you.

---

## 2. Deployments: The Production Standard

A **Deployment** is a higher-order controller that manages ReplicaSets and provides declarative updates to Pods. 

### The Architecture Hierarchy

```text
+-------------------------------------------------------------+
|                        DEPLOYMENT                           |
|                 (Manages rollout strategy)                  |
+------------------------------+------------------------------+
                               |
               +---------------v---------------+
               |          REPLICASET           |
               |     (Ensures replica count)   |
               +---------------+---------------+
                               |
          +--------------------+--------------------+
          |                    |                    |
  +-------v-------+    +-------v-------+    +-------v-------+
  |      POD      |    |      POD      |    |      POD      |
  | (Running app) |    | (Running app) |    | (Running app) |
  +---------------+    +---------------+    +---------------+
```

### Why use Deployments?
1. **Self-Healing:** Automatically replaces failed pods.
2. **Scaling:** Easily scale up or down based on traffic.
3. **Zero-Downtime Updates:** Seamlessly roll out new application versions.
4. **Rollbacks:** Instantly revert to a previous version if an update crashes.

---

## 3. Creating & Scaling Deployments

Mastering imperative commands for Deployments will save you massive amounts of time on the CKA exam.

### Imperative Creation
Create a deployment named `web-app` using the `nginx:1.14` image with 3 replicas:
```bash
kubectl create deployment web-app --image=nginx:1.14 --replicas=3
```

### Declarative Generation (The CKA Way)
Always use `--dry-run=client -o yaml` to generate your base template:
```bash
kubectl create deployment web-app --image=nginx:1.14 --replicas=3 --dry-run=client -o yaml > deploy.yaml
```

### Scaling the Deployment
If traffic spikes, you need to increase the replica count.

**Imperative Scaling (Instant):**
```bash
kubectl scale deployment web-app --replicas=5
```

**Declarative Scaling:**
Edit the `deploy.yaml` file, change `replicas: 5`, and apply:
```bash
kubectl apply -f deploy.yaml
```

---

## 4. Performing Rolling Updates

A **Rolling Update** is the default deployment strategy in Kubernetes. Instead of taking down all old pods at once (which causes downtime), it incrementally replaces old pods with new ones.

### The Update Logic
* **maxSurge:** How many extra pods can be created over the desired number during the update (default 25%).
* **maxUnavailable:** How many pods can be offline during the update (default 25%).

### Triggering an Update (Changing the Image)
Suppose we want to upgrade our `web-app` from `nginx:1.14` to `nginx:1.16`. Use the `set image` command:
```bash
kubectl set image deployment/web-app nginx=nginx:1.16
```
*(Note: The format is `container_name=new_image_name`)*

### Monitoring the Rollout
Check the live status of the update as pods are swapped out:
```bash
kubectl rollout status deployment/web-app
```

### Viewing Upgrade History
To see all previous versions (revisions) of your deployment:
```bash
kubectl rollout history deployment/web-app
```

> **Pro-Tip:** To make the history readable, always use `--record` when making changes (though this flag is being deprecated, it is still heavily referenced in legacy documentation), or rely on proper GitOps annotations.

---

## 5. Performing Rollbacks (Undo)

If you accidentally deploy a broken image (e.g., `nginx:1.99-typo`), the pods will crash, and the rollout will halt. You must roll back to the previous stable state.

### Instantly Undo the Last Update
This will revert the deployment to the immediately preceding revision:
```bash
kubectl rollout undo deployment/web-app
```

### Rollback to a Specific Revision
If you need to go back to a version from three days ago, first check the history to find the Revision number:
```bash
kubectl rollout history deployment/web-app
```

Then rollback to that exact revision (e.g., revision 2):
```bash
kubectl rollout undo deployment/web-app --to-revision=2
```

---

## 6. CKA Exam Quick-Reference Cheat Sheet

Accelerate your workflow with these essential workload controller commands:

```bash
# 1. Create a deployment and expose it simultaneously (Rare but powerful)
kubectl create deployment web --image=nginx --port=80
kubectl expose deployment web --port=80 --type=NodePort

# 2. Fast Scaling
kubectl scale deploy <deployment-name> --replicas=10

# 3. Update an image on the fly
kubectl set image deploy/<deployment-name> <container-name>=<new-image>

# 4. Check rollout status
kubectl rollout status deploy/<deployment-name>

# 5. Rollback to safety
kubectl rollout undo deploy/<deployment-name>

# 6. Extract the YAML of a running deployment without cluster-specific junk
kubectl get deploy <deployment-name> -o yaml > current-deploy.yaml
```