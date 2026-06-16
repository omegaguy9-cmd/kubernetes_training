# Kubernetes Networking: Services and Traffic Routing

This repository covers the networking backbone of Kubernetes: **Services**. Understanding how to route internal traffic, expose applications to the outside world, and link frontend workloads to backend databases is critical for both real-world production environments and the **Certified Kubernetes Administrator (CKA)** exam.

---

## 📋 Table of Contents
1. [Why Do We Need Services?](#1-why-do-we-need-services)
2. [The Architecture of a Service](#2-the-architecture-of-a-service)
3. [The Three Core Service Types](#3-the-three-core-service-types)
4. [Imperative Service Creation](#4-imperative-service-creation)
5. [Declarative Service Generation](#5-declarative-service-generation)
6. [Troubleshooting Endpoints](#6-troubleshooting-endpoints)
7. [CKA Exam Quick-Reference Cheat Sheet](#7-cka-exam-quick-reference-cheat-sheet)

---

## 1. Why Do We Need Services?

In Kubernetes, **Pods are ephemeral**. They die, scale up, scale down, and get recreated constantly. Every time a Pod is created, it gets a new, completely random IP address. 

**The Problem:** If your Frontend Pod needs to talk to your Backend Pod, how does it know where to send traffic if the Backend Pod's IP address changes every 5 minutes?

**The Solution:** A **Service**. A Service acts as a static, permanent IP address and DNS name for a group of Pods. Even if the Pods behind the Service die and are replaced 100 times, the Service IP remains exactly the same. The Service continuously acts as a load balancer, forwarding traffic to the healthy, currently active Pods based on **Labels**.

---

## 2. The Architecture of a Service

The Service relies entirely on **Selectors** and **Labels** to know where to send traffic. 

```text
    [External User] OR [Internal Frontend Pod]
           |
           v
    +---------------------------------------------------+
    |                   SERVICE                         |
    |  Static IP: 10.96.0.10     Selector: app=backend  |
    +------------------------+--------------------------+
                             |
         (Continuously discovers Pods with app=backend)
                             |
           +-----------------+-----------------+
           |                 |                 |
    +------v------+   +------v------+   +------v------+
    |     POD     |   |     POD     |   |     POD     |
    | app=backend |   | app=backend |   | app=backend |
    | IP: 1.1.1.1 |   | IP: 1.1.1.2 |   | IP: 1.1.1.3 |
    +-------------+   +-------------+   +-------------+
```

---

## 3. The Three Core Service Types

Kubernetes provides three main ways to expose your application, depending on where the traffic is coming from.

### 1. ClusterIP (The Default)
* **What it does:** Exposes the Service on an internal IP address inside the cluster. 
* **Accessibility:** The application is **only** reachable from *within* the cluster.
* **Use Case:** A backend database (like Redis or MySQL) that should only be accessed by your internal frontend applications, never by the public internet.

### 2. NodePort
* **What it does:** Exposes the Service on each Worker Node's physical IP address at a specific port.
* **Accessibility:** External. Anyone who hits `http://<Node-IP>:<NodePort>` will be routed to the Service.
* **Port Range:** Kubernetes strictly enforces NodePorts to be between **30000 - 32767**.
* **Use Case:** Local development, testing, or environments where you don't have a cloud load balancer.

### 3. LoadBalancer
* **What it does:** Automatically integrates with your Cloud Provider (AWS, GCP, Azure) to provision a physical, external Load Balancer (like an AWS ALB).
* **Accessibility:** External. Traffic hits the Cloud Load Balancer -> NodePort -> Service -> Pod.
* **Use Case:** Production applications exposed to the public internet.

---

## 4. Imperative Service Creation

The fastest way to create Services during the CKA exam is using the `kubectl expose` command. This automatically grabs the labels from the target Pod or Deployment and creates the Service linking to them.

### Create a ClusterIP Service
Assume you have a deployment named `redis-backend` running on port 6379:

```bash
kubectl expose deployment redis-backend --port=6379 --target-port=6379 --type=ClusterIP
```

### Create a NodePort Service
Assume you have a deployment named `nginx-frontend` running on port 80. You want to expose it outside the cluster:

```bash
kubectl expose deployment nginx-frontend --port=80 --target-port=80 --type=NodePort
```
*(Note: If you don't specify the exact NodePort, Kubernetes will randomly assign one between 30000-32767).*

---

## 5. Declarative Service Generation

To generate the YAML for your repository or to specify an exact NodePort (which is often required in the exam), use the dry-run flag.

**Step 1: Generate the YAML**
```bash
kubectl expose deployment nginx-frontend --port=80 --type=NodePort --dry-run=client -o yaml > svc.yaml
```

**Step 2: Edit the YAML (to enforce a specific NodePort)**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-frontend
spec:
  type: NodePort
  selector:
    app: nginx-frontend
  ports:
  - port: 80           # The port the Service listens on
    targetPort: 80     # The port the Container is listening on
    nodePort: 30080    # <--- ADD THIS manually to specify the exact port
```

**Step 3: Apply the YAML**
```bash
kubectl apply -f svc.yaml
```

---

## 6. Troubleshooting Endpoints

A Service is useless if it isn't actually routing traffic to your Pods. The mechanism that links a Service to a Pod is called an **Endpoint**.

**The Golden Rule of Services:** If your Service isn't working, check the Endpoints! If the Service's `selector` doesn't perfectly match the Pod's `labels`, the Endpoint list will be empty.

### Check the Service Status
```bash
kubectl get svc
```

### Check the Endpoints (Crucial for CKA Diagnostics)
```bash
kubectl get endpoints
```
*If you see `<none>` under the endpoints column, your Service's labels are wrong, or your Pods are dead.*

### Deep-Dive Inspection
```bash
kubectl describe svc nginx-frontend
```
*(Look closely at the `Selector:` and `Endpoints:` rows in the output. They must align).*

---

## 7. CKA Exam Quick-Reference Cheat Sheet

```bash
# 1. Create a pod and expose it simultaneously (Huge time saver)
kubectl run web --image=nginx --port=80 --expose

# 2. List all services and their assigned NodePorts/ClusterIPs
kubectl get svc -o wide

# 3. Quickly test if an internal ClusterIP service is responding
# Run a temporary busybox pod to curl the service name
kubectl run test-curl --image=busybox:1.28 --rm -it -- sh
/ # wget -O- http://<service-name>:<port>

# 4. Find which Endpoints are mapped to a specific service
kubectl get ep <service-name>
```