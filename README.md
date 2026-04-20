# 🚦 Kueue Batch Orchestration: The "Restaurant" Guide

Welcome to the ultimate resource hub for understanding and deploying **Kueue**—the Kubernetes-native job queueing controller. This project bridges the gap between standard Pod scheduling and complex Batch/AI workload management using a simple, relatable **Restaurant Analogy**.

---

## 📖 The Core Concept: Why Kueue?

Standard Kubernetes is like a restaurant with no host. Groups of diners (**Pods**) try to grab any available chair. If a family of 20 (**a Job**) arrives and only 5 chairs are free, the first 5 sit down and wait forever for the other 15. The restaurant is full, but no one is actually eating. This is known as **Partial Allocation**.

**Kueue is the Restaurant Host.** It keeps the family in the "Waiting Room" (**LocalQueue**) until a table for 20 is ready. 

### Key Benefits:
* ✅ **No Partial Allocation:** Jobs only start when they have *all* required resources.
* ✅ **Fair Sharing:** Different teams (Namespaces) get their fair share of the "Dining Room" (**ClusterQuota**).
* ✅ **Resource Borrowing:** If one team isn't "hungry," another team can borrow their seats until they return.

---

## 🏗️ Architecture & Components

This project explores the internal mechanics of the Kueue Manager:

* **LocalQueue:** The team-specific waitlist (Namespace scoped).
* **ClusterQueue:** The total "Dining Room" capacity and global resource rules (Cluster scoped).
* **ResourceFlavor:** Different "table types" (e.g., standard nodes, GPU-enabled nodes).
* **Admission Manager:** The decision engine that flips the `suspend: true` flag to `false`.



---

## 🚀 Quick Start: Local Laptop Demo

You can run this entire demo on your laptop using **Minikube** or **Kind**.

### 1. Prerequisites
* [Minikube](https://minikube.sigs.k8s.io/docs/start/) or [Kind](https://kind.sigs.k8s.io/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
* [Kueue installed](https://kueue.sigs.k8s.io/docs/installation/) on your cluster.

### 2. Setup the "Tiny Restaurant"
Apply the configuration to limit your cluster to a 1 CPU quota:
```bash
kubectl apply -f ./manifests/tiny-restaurant.yaml
```
### 3. Scenario A: The "Giant Job" (Waitlist Trigger)
Submit a job that requests 2 CPUs. Since the quota is only 1 CPU, Kueue will hold it in the waitlist:
```bash
kubectl create -f ./manifests/giant-job.yaml
```
### 4. Scenario B: The "Small Job" (Success Path)
Submit a job that fits (0.5 CPU) and watch Kueue admit it immediately:
```bash
kubectl create -f ./manifests/nginx-job.yaml

# Watch the 'suspend' flag flip from true to false
kubectl get jobs -w
```
---
## 🛡️ Enterprise & Security
For production environments, this repository includes an assessment covering:

RBAC: Limiting who can modify ClusterQueues.

Multi-tenancy: Using Cohorts for resource sharing between departments.

Admission Checks: Integrating external security/data scans before a job is allowed to run.

## 📁 Project Structure
```Plaintext
├── diagrams/            # Visual aids for the Restaurant Analogy
├── manifests/           
│   ├── tiny-restaurant/ # ClusterQueue & LocalQueue setup
│   └── jobs/            # Sample Nginx and Batch jobs
└── docs/                # Detailed technical deep-dives
```
---
##🎙️ bitfid Content Series
This repository serves as the companion source code for the technical podcast and video series. If you are following along with the "Kueue vs. Scheduler" episode, all demo files are located in /manifests.

Built for the CNCF Community.
If you find this useful, please ⭐ the repo!
