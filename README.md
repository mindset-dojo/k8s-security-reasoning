# k8s-security-reasoning — A Minimal Learning Asset

## Purpose

This repository is a **minimal learning asset** designed to explore cloud security
*reasoning* through a small, concrete Kubernetes exercise.

It focuses on understanding **security boundaries**, **failure modes**, and
**risk tradeoffs** that emerge at the intersection of:

- the Kubernetes control plane
- the cloud provider control plane (IAM, compute, network, storage)
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
- Identify where cloud compute, network, and storage intersect with Kubernetes workloads
- Understand why security and cost risk often exists *between* systems
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
- compute placement
- network exposure
- storage persistence
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
6. Observe cloud **compute, storage, and networking** artifacts created indirectly by Kubernetes
7. Deliberately break things and reason about what fails and why
8. Tear everything down and verify cleanup across layers

---

## Kubernetes as an Abstraction Layer

Kubernetes is an abstraction layer for:

- **Compute**
- **Network**
- **Storage**

It provides a consistent control plane across cloud providers and infrastructure types.

From a security perspective, Kubernetes is a **coordination system**, not a complete
security solution.

Many critical controls live outside Kubernetes — particularly in cloud compute,
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
Security requires the ability to observe and reason about system state and change
across **all layers**, not just Kubernetes.

---

## What `main.yaml` Contains

The single manifest intentionally includes:

- **Namespace** — isolation boundary
- **ServiceAccount** — workload identity
- **Role & RoleBinding** — least-privilege Kubernetes API access
- **Deployment (NGINX)** — workload lifecycle and compute placement
- **PersistentVolumeClaim** — state and data persistence
- **Service** — network exposure
- **NetworkPolicy** — explicit network allow rules

Each resource exists to make a security-relevant boundary visible.

---

## Quick Start: GKE (Google Kubernetes Engine)

### Assumptions

- You have a GCP project
- You can create GKE clusters
- You will use **Google Cloud Shell**
- You will operate from the `us-central1` region

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

## Observing Cloud-Side Resources (Compute, Storage, Network)

Kubernetes declares *intent*.  
The cloud provider control plane implements *infrastructure*.

Explicitly observe resources created indirectly by Kubernetes.

---

### Compute (Nodes as VMs)

Kubernetes schedules pods onto nodes.  
The cloud provider runs **virtual machines**.

```bash
gcloud compute instances list
```

Reason about:
- How many VMs exist?
- Which zone and subnet are they in?
- What permissions do they inherit?

---

### Storage (PVCs as Disks)

PersistentVolumeClaims map to **cloud disks**.

```bash
gcloud compute disks list
```

Reason about:
- Which disks back Kubernetes PVCs?
- What happens to disks if pods or namespaces are deleted?
- What data persists beyond workload lifetime?

---

### Networking (Services as Infrastructure)

Kubernetes networking intent becomes cloud networking primitives.

```bash
gcloud compute firewall-rules list
gcloud compute forwarding-rules list
gcloud compute routes list
```

Reason about:
- Which rules allow ingress?
- How broadly are they scoped?
- Which paths exist outside Kubernetes visibility?

---

## Break and Reason

Introduce failure or loosen controls:

- Remove the `NetworkPolicy`
- Change the `Service` type
- Delete and recreate pods
- Delete the namespace but not the cluster

Observe **Kubernetes behavior** and **cloud-side consequences**.

---

## Teardown and Verification

Teardown is part of the learning exercise.

Deleting Kubernetes resources does **not** guarantee that all cloud resources
are removed.

Verification must happen **per layer**.

---

### Step 1: Delete Kubernetes resources

```bash
kubectl delete namespace demo
```

---

### Step 2: Delete the GKE cluster

```bash
gcloud container clusters delete sec-dojo-cluster --region us-central1
```

---

## Layered Teardown Verification

### Compute Verification

Ensure no cluster-related VMs remain:

```bash
gcloud compute instances list
```

Lingering instances represent:
- cost leakage
- unmanaged compute
- unexpected attack surface

---

### Storage Verification

Ensure no disks remain:

```bash
gcloud compute disks list
```

Lingering disks represent:
- persistent data exposure
- silent cost accumulation
- orphaned state outside Kubernetes control

---

### Network Verification

Ensure no networking artifacts remain:

```bash
gcloud compute firewall-rules list
gcloud compute forwarding-rules list
gcloud compute routes list
```

Lingering networking resources represent:
- unintended access paths
- security group drift
- infrastructure-level exposure

---

## Reflection: What Required Human Verification?

After teardown, reflect:

- Which layers cleaned up automatically?
- Which required explicit verification?
- Which resources were easiest to forget?
- Where does Kubernetes abstraction *help* — and where does it hide risk?

The goal is not perfect cleanup.
The goal is **clear reasoning about responsibility and residual risk**.

---

## Contributing & Discussion

This repository is intentionally **minimal, incomplete, and opinionated**.

It is not meant to represent “the right way” to secure Kubernetes or cloud
infrastructure. Instead, it exists as a **shared reasoning surface** — a small,
concrete system that makes boundaries, assumptions, and failure modes visible.

If you notice:

- inaccuracies or oversimplifications
- missing edge cases or important caveats
- alternative interpretations of responsibility boundaries
- places where your real-world experience contradicts the narrative

You are encouraged to engage.

Useful contributions include:
- opening issues with questions or counterexamples
- submitting pull requests that clarify or deepen the reasoning
- adding small experiments or variations that reveal different failure modes
- sharing stories where these abstractions broke down in practice

Thoughtful disagreement is welcome.
So is uncertainty.

The goal is not consensus or completeness, but **clearer thinking about where
security responsibility actually lives — and where it does not**.

