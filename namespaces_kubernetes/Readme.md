# Kubernetes Organization: Namespaces and Cross-Namespace Services

This repository covers **Namespaces**, the fundamental way Kubernetes isolates resources, and how **Services** communicate across these boundaries. Mastering namespace management and Fully Qualified Domain Names (FQDN) is absolutely critical for the **Certified Kubernetes Administrator (CKA)** exam.

---

## 📋 Table of Contents
1. [What is a Namespace?](#1-what-is-a-namespace)
2. [The Built-in Namespaces](#2-the-built-in-namespaces)
3. [Creating and Managing Namespaces](#3-creating-and-managing-namespaces)
4. [Services & Namespaces (Cross-Namespace Routing)](#4-services--namespaces-cross-namespace-routing)
5. [CKA Exam Quick-Reference Cheat Sheet](#5-cka-exam-quick-reference-cheat-sheet)

---

## 1. What is a Namespace?

A **Namespace** is a virtual cluster backed by the same physical cluster hardware. It allows you to partition cluster resources among multiple users, teams, or environments.

**Why use them?**
* **Isolation:** You can have a `frontend` pod in the `dev` namespace and a completely different `frontend` pod in the `prod` namespace. They will not conflict.
* **Security & Quotas:** You can restrict CPU/Memory limits and Role-Based Access Control (RBAC) on a per-namespace basis.
* **Organization:** Prevents your cluster from becoming a chaotic mess of thousands of unorganized Pods and Services.

---

## 2. The Built-in Namespaces

When you spin up a fresh Kubernetes cluster, it comes with four initial namespaces:

1. **`default`:** The standard namespace where your resources go if you don't specify one.
2. **`kube-system`:** Where Kubernetes runs its own critical control-plane components (e.g., `kube-apiserver`, `CoreDNS`, `etcd`). *Never mess with this in production unless you know what you are doing!*
3. **`kube-public`:** A readable namespace available to all users (even unauthenticated ones). Rarely used for custom workloads.
4. **`kube-node-lease`:** Holds heartbeat data for each node so the control plane knows the nodes are alive.

---

## 3. Creating and Managing Namespaces

Managing namespaces imperatively is the fastest method for the exam.

### Create a Namespace
    kubectl create namespace dev-env

### List all Namespaces
    kubectl get namespaces
    # OR the shorthand:
    kubectl get ns

### Deploying Resources into a Specific Namespace
By default, `kubectl` always targets the `default` namespace. To deploy a pod into our new namespace, you must use the `-n` or `--namespace` flag:

    kubectl run dev-nginx --image=nginx --namespace=dev-env

### Finding Resources in a Namespace
If you forget the `-n` flag, you won't see your pod!

    # This will return "No resources found":
    kubectl get pods 

    # This will show your pod:
    kubectl get pods -n dev-env

    # This will show pods across the ENTIRE cluster:
    kubectl get pods --all-namespaces
    # Shorthand:
    kubectl get pods -A

---

## 4. Services & Namespaces (Cross-Namespace Routing)

This is a heavily tested concept in the CKA exam. 

**The Golden Rule:** Namespaces isolate *names*, but they **DO NOT** isolate *network traffic* (unless you explicitly create NetworkPolicies). A Pod in the `dev` namespace can absolutely talk to a Service in the `prod` namespace, but it has to use the correct DNS address.

### Scenario: The FQDN (Fully Qualified Domain Name)
You have a Frontend Pod in the `frontend-ns` namespace, and it needs to hit a Database Service (named `db-service`) in the `database-ns` namespace.

If the Frontend Pod simply curls `http://db-service`, it will fail. CoreDNS will only look for `db-service` inside the local `frontend-ns` namespace.

To cross the namespace boundary, the Frontend Pod must use the **FQDN**:

    http://<service-name>.<namespace>.svc.cluster.local

**Example:**
    http://db-service.database-ns.svc.cluster.local

### The Cross-Namespace Architecture

    [Frontend Pod] (Namespace: frontend-ns)
          |
          | (Looks up: db-service.database-ns.svc.cluster.local)
          v
    [CoreDNS] (Resolves FQDN to Service IP: 10.96.5.5)
          |
          v
    [Service: db-service] (Namespace: database-ns)
          |
          v
    [Database Pod] (Namespace: database-ns)

---

## 5. CKA Exam Quick-Reference Cheat Sheet

Managing contexts and namespaces quickly is the #1 way to save time on the CKA exam.

    # 1. PERMANENTLY change your default namespace. 
    # If a question asks you to work exclusively in 'qa-env', run this so you don't have to type '-n qa-env' every time!
    kubectl config set-context --current --namespace=qa-env

    # 2. Find a missing pod when you don't know what namespace it is in
    kubectl get pods -A | grep <pod-name>

    # 3. Create a namespace declaratively (Dry Run)
    kubectl create ns web-prod --dry-run=client -o yaml > prod-ns.yaml

    # 4. Get all resources (Pods, Services, Deployments, ReplicaSets) in a namespace
    kubectl get all -n <namespace-name>

    # 5. Delete a namespace (WARNING: This deletes EVERYTHING inside it!)
    kubectl delete ns <namespace-name> --force --grace-period=0