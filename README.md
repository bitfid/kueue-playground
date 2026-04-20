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
