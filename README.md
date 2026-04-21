---
## 🎙️ bitfid Content Series
This repository serves as the companion source code for the technical podcast and video series. If you are following along with the "Kueue vs. Scheduler" episode, all demo files are located in /manifests.

Built for the CNCF Community.
If you find this useful, please ⭐ the repo!
---

# 🚦 Kueue Batch Orchestration: The "Restaurant" Guide

Welcome to the bidfid resource hub for understanding and deploying **Kueue**—the Kubernetes-native job queueing controller. This project bridges the gap between standard Pod scheduling and complex Batch/AI workload management using a simple, relatable **Restaurant Analogy**.

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

<br>
<img width="1408" height="768" alt="restaurantAnalagy" src="https://github.com/user-attachments/assets/2ede1fa3-a2a8-4cf0-8b73-866d4b23fdd5" />
<br>
<img width="1408" height="768" alt="pov_vs_job" src="https://github.com/user-attachments/assets/d491c11a-d7dd-43af-a86c-5198480c5d0f" />
<br>

---

## 🚀 Quick Start: Local Laptop Demo

You can run this entire demo on your laptop using **Minikube** or **Kind**.

### 1. Prerequisites
* [Minikube](https://minikube.sigs.k8s.io/docs/start/) or [Kind](https://kind.sigs.k8s.io/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
* [Kueue installed](https://kueue.sigs.k8s.io/docs/installation/) on your cluster.
```Plaintext
https://kueue.sigs.k8s.io/docs/installation/
```

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
<img width="1408" height="768" alt="tinyRestaurantJob" src="https://github.com/user-attachments/assets/2dd50ca2-8482-4805-ae58-7ecffe354992" />

### 4. Scenario B: The "Small Job" (Success Path)
Submit a job that fits (0.5 CPU) and watch Kueue admit it immediately:
```bash
kubectl create -f ./manifests/nginx-job.yaml
# Watch the 'suspend' flag flip from true to false
kubectl get jobs -w
```
<img width="1408" height="768" alt="nginxJob" src="https://github.com/user-attachments/assets/654426e3-a629-4509-80ac-4fec72ad4069" /> 

---

## 🛡️ Enterprise & Security
Kueue is officially a Kubernetes SIG (Special Interest Group) project, which means it follows the same rigorous development and stability standards as Kubernetes itself.

* Production Adoption: Major organizations (e.g., CyberAgent, CoreWeave) and cloud providers use it in production to manage expensive GPU clusters.
* Scalability: It is specifically designed to handle "Batch" scale—managing thousands of jobs without overwhelming the Kubernetes API server, which is a common breaking point for enterprises.
* Multi-Tenancy: It is built for the enterprise "Shared Cluster" model. Its ClusterQueue and ResourceFlavor abstractions allow platform teams to set hard quotas for different departments (e.g., "Research" vs. "Production") while allowing for safe resource borrowing.
* Integration: It integrates natively with popular enterprise tools like Kubeflow, Ray, JobSet, and Cluster Autoscaler.

## 📁 Project Structure
```Plaintext
├── diagrams/            # Visual aids for the Restaurant Analogy
├── manifests/           
│   ├── tiny-restaurant/ # ClusterQueue & LocalQueue setup
│   └── jobs/            # Sample Nginx and Batch jobs
└── docs/                # Detailed technical deep-dives
```
---
## The Standard Scheduler "Partial Allocation" Crisis
This scenario illustrates the exact failure point discussed in the podcast outline: how thousands of unrelated pod scheduling decisions create gridlock.

A large, distributed job (like the 100-node training session) requires "All-or-Nothing" scheduling. The default scheduler doesn't understand "All," only "One."

<br>
<img width="1408" height="768" alt="schedularFailure" src="https://github.com/user-attachments/assets/453b68f4-6353-4b7b-b474-7c9bd3665903" />
<br>

What this diagram shows:

1. Atomic Pod Decisions: The Scheduler is looking only at immediate space. It seats Pods from Job A and Job B interleaved on available nodes (Nodes 1, 2, 4, 5).
2. Resource Fragmentation: By trying to be efficient in the short term, the scheduler fragments the cluster.
3. The Crisis: Job B (green) has only 4 of its 5 required pods scheduled (Node 5 is full). Job A (blue) has many pods scattered, but not nearly enough to run its 100-node training. Both large, critical jobs are stalled, and the cluster is locked down by pending pods.


## The Kueue Solution—The Admission Gate
This is the central metaphor of Kueue. It changes the sequence of events. Instead of trying to schedule the chaos, Kueue creates a waiting room upstream of the scheduler.
<br>
<img width="1408" height="768" alt="kueueAdmissionGate" src="https://github.com/user-attachments/assets/115f7045-c9a5-492b-a4a5-7521213c0aba" />
<br>
What this diagram shows:

1. Job Suspension: The two large jobs (now called Workloads) arrive at Kueue but are immediately suspended. They enter a LocalQueue.
2. The Waiting Room: Instead of flooding the API server with 105 Pending pods, Kueue holds the entire intent of the Job in the Waiting Room.
3. The Gate: The "Admission Gate" is locked. It only opens when the Cluster-Wide Quota Check validates that resources (e.g., A100 GPUs) are available.
4. Orderly Admission: Job B (5 GPUs) is smaller than the available quota (20 GPUs), so the gate opens (green arrow). The entire Job B is admitted at once. Job A remains blocked, as it needs 100 GPUs and only 20 are available. No resource fragmentation occurs.


## Borrowing and Preemption (The Advanced Logic)
This final diagram visualizes the powerful concepts of ResourceFlavors and Fair Sharing that you’ll discuss in later episodes. It shows how Kueue maximizes utilization without compromising access.
<br>
<img width="1408" height="768" alt="kueuePreemption" src="https://github.com/user-attachments/assets/57a8d620-1e3f-4ada-999e-252190362585" />
<br>
What this diagram shows:

1. Time T0 (Resource Borrowing): There are two teams, Namespace A and Namespace B, each with a ClusterQueue of 10 GPUs and 20 CPUs. Currently, only Workload A1 (which needs 30 CPUs) is running. It consumes all 20 of Namespace A’s CPUs, but because Namespace B is not using its resources (Active Workload B1 is small), Workload A1 can borrow 10 CPUs. The cluster utilization is maximized.
2. Time T1 (The High-Priority Request): The timelines advance. A new, high-priority request (Job B) arrives in Namespace B.
3. The Preemption Logic: Kueue sees that Namespace B now requires its full quota. The Preemption Logic is triggered. It points back to Workload A1, which is now crossed out with a red X and labeled PREEMPTION TRIGGER.
4. Reclaimed Quota: Workload A1 is immediately stopped. This releases the borrowed 10 CPUs back to Namespace B.
5. Order Restored: Reclaimed quota (20 CPUs/10 GPUs) is now available for the high-priority Job B. The fair sharing policy is enforced.
   
---

## The Kueue Internal Components
<br>
<img width="1408" height="768" alt="kueueWorkflow" src="https://github.com/user-attachments/assets/ee878b22-01cd-41fe-847e-14ddad4c6a54" />
<br>

