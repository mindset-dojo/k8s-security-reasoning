# k8s-security-reasoning — A Minimal Learning Asset

## Purpose

This repository is a **minimal learning asset** designed to explore cloud security
*reasoning* through a small, concrete Kubernetes exercise.

It focuses on understanding **security boundaries**, **failure modes**, and
**risk tradeoffs** that emerge at the intersection of:

- the Kubernetes control plane
- the cloud provider control plane (IAM, network, storage)
- workload identity, networking, and persistence

Rather than presenting best-practice checklists, this repository emphasizes
**first principles**, **systems thinking**, and **hands-on exploration**.

The goal is not Kubernetes mastery, but clarity around *where security
responsibility lives* — and where it does not.

---

## Who This Is For

This repository is intended for:

- Cloud engineers, SREs, and DevOps practitioners
- Security engineers and architects
- Customer Success Engineers, TAMs, and Solutions Architects
- Anyone seeking a grounded way to reason about Kubernetes and cloud security

It assumes basic familiarity with Kubernetes concepts, but **does not require
deep operational expertise**.

This is **not** production-ready infrastructure.
It is a shared reasoning surface.

---

## Learning Goals

By working through this repository, you should be able to:

- Explain what Kubernetes *does* and *does not* secure
- Understand Kubernetes RBAC as control-plane authorization
- Reason about default failure modes (open vs. closed)
- Identify where cloud IAM, network, and storage intersect with Kubernetes workloads
- Understand why security risk often exists *between* systems
- Hold a grounded conversation about Kubernetes security using first principles

---

## Why NGINX?

This repository uses **NGINX** as the example workload intentionally.

NGINX is:
- widely understood
- operationally boring
- easy to expose over the network
- free of application-specific complexity

We are **not** teaching NGINX.

NGINX serves as a predictable execution surface so attention stays on:
- identity
- network exposure
- storage
- control-plane boundaries

---

## How to Use This Repository

This repository contains two artifacts:

- `README.md` — the reasoning guide
- `main.yaml` — a single Kubernetes manifest containing all resources

Recommended approach:

1. Read the README to understand *why* each resource exists
2. Review `main.yaml` to see how those ideas are expressed declaratively
3. Apply the manifest to a GKE cluster
4. Inspect Kubernetes resources
5. Observe related cloud control-plane resources
6. Deliberately break things and reason about what fails and why
7. Tear everything down

This works well for:
- individual learning
- pair exploration
- small group workshops

---

## Kubernetes as an Abstraction Layer

Kubernetes is an abstraction layer for:

- **Compute**
- **Network**
- **Storage**

It provides a consistent control plane across cloud providers and infrastructure types.

From a security perspective, Kubernetes is a **coordination system**, not a complete
security solution.

Many critical controls live outside Kubernetes — particularly in cloud IAM,
networking, and storage services.

Understanding where Kubernetes responsibility ends is essential.

---

## Grounding Security Principles

This exercise is grounded in three principles:

### 1. Least Privilege
Access should be narrowly scoped and explicitly granted.

In Kubernetes:
- ServiceAccounts
- Roles and RoleBindings
- Namespace scoping

In GCP:
- IAM roles
- service identities
- access to APIs, storage, and network

### 2. Defense in Depth
No single control is sufficient.

Security emerges from layering:
- identity
- network
- storage
- workload configuration

### 3. Visibility
Security requires the ability to observe and reason about system state and change.

---

## What `main.yaml` Contains

The single manifest intentionally includes:

- **Namespace** — isolation boundary
- **ServiceAccount** — workload identity
- **Role & RoleBinding** — least-privilege Kubernetes API access
- **Deployment (NGINX)** — workload lifecycle and blast radius
- **PersistentVolumeClaim** — state and data access
- **Service** — network exposure
- **NetworkPolicy** — shift from default-open to explicit allow

Each resource exists to make a security-relevant boundary visible.

---

## Quick Start: GKE (Google Kubernetes Engine)

### Assumptions

- You have a GCP project
- You can create GKE clusters
- You will use **Google Cloud Shell**
- You will operate from the us-central1 region

---

### 1. Set project and region

```bash
gcloud config set project YOUR_PROJECT_ID
gcloud config set compute/region us-central1
```

---

### 2. Create a GKE cluster

```bash
gcloud container clusters create sec-dojo-cluster \
  --num-nodes=2 \
  --enable-network-policy
```

Configure kubectl:

```bash
gcloud container clusters get-credentials sec-dojo-cluster
```

---

### 3. Apply the manifest

```bash
kubectl apply -f main.yaml
```

---

### 4. Inspect Kubernetes resources

```bash
kubectl get all -n demo
kubectl get sa,role,rolebinding,networkpolicy -n demo
```

---

### 5. Observe cloud-side resources

```bash
gcloud container clusters list
gcloud compute disks list
gcloud compute firewall-rules list
```

---

### 6. Break and reason

Examples:
- Remove the NetworkPolicy and observe traffic behavior
- Tighten RBAC and test API access
- Remove the PVC and observe pod behavior

---

## Teardown and Verification

### Step 1: Delete Kubernetes resources

```bash
kubectl delete namespace demo
```

This removes all in-cluster objects, including Pods, Services, PVCs, and NetworkPolicies.

---

### Step 2: Delete the GKE cluster

```bash
gcloud container clusters delete sec-dojo-cluster --region us-central1-a
```

Deleting the cluster **normally cleans up all underlying cloud resources**, including:
- node VMs
- attached persistent disks
- firewall rules
- routes and network tags

---

### Why resources sometimes remain

In real environments, underlying resources may remain if:
- disks were manually detached
- resources were created outside the cluster lifecycle
- deletion was interrupted or partially failed

This is an important operational failure mode to understand.

---

### Step 3: Verify cloud resources are cleared

After deletion, verify that no related resources remain:

```bash
gcloud compute instances list
gcloud compute disks list
gcloud compute firewall-rules list
```

If these commands return no resources associated with the cluster, teardown is complete.

---

## Contributing & Discussion

This repository is intentionally minimal and exploratory.

If you notice inaccuracies, missing considerations, or alternative ways to reason
about these boundaries, you’re invited to:

- open an issue
- submit a pull request
- or start a conversation directly with the maintainers and contributors

Thoughtful discussion is welcome.
