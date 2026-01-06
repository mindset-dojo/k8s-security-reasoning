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
3. Create and inspect a Kubernetes cluster
4. Apply the manifest
5. Observe Kubernetes resources
6. Observe related cloud control-plane resources
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

## A Simple Threat Reasoning Approach

This exercise is not a formal threat model.
It is a **way of orienting your thinking** when working with Kubernetes as an
abstraction layer over cloud infrastructure.

The goal is to reason clearly about **defaults, failure modes, and attacker
opportunity**, even when you do not yet know every detail of the system.

---

### Defaults and Failure Modes

A useful starting question in any security discussion is:

> *When something is missing or misconfigured, does the system fail open or fail closed?*

Examples you will encounter in this exercise:

- **Kubernetes RBAC**: default **closed**
- **Cloud IAM**: default **closed**
- **Kubernetes networking**: default **open**
- **NetworkPolicy**: once applied, default becomes **closed**

---

### Reasoning from a Foothold

Assume a simple starting condition:

> *An attacker has execution inside a container.*

From there, reasoning can unfold in a predictable sequence.

---

### 1. Identity

What identity does the workload already have?

Consider:
- ServiceAccount tokens
- Mounted credentials or secrets
- Cloud workload identity

---

### 2. Network

What can the workload reach?

Consider:
- other pods and namespaces
- cluster services
- external endpoints

---

### 3. Data

What data is accessible?

Consider:
- persistent volumes
- databases or object storage
- data that outlives the pod

---

### 4. Escape and Expansion

Can containment be broken?

Consider:
- privileged containers
- node access
- cloud API access

---

### Visibility as a Multiplier

Logs, metrics, traces, and higher-level CNAPP platforms do not prevent compromise.
They change how quickly and clearly activity can be understood.

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

Inspect the kubeconfig created by this command:

```bash
ls ~/.kube
cat ~/.kube/config
```

Verify cluster connectivity and explore the API surface:

```bash
kubectl cluster-info
kubectl api-resources
kubectl get namespaces
kubectl get nodes
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

## Visibility & Reachability Exercises

This section grounds the threat reasoning model in **direct observation**.

Rather than assuming behavior, you will:
- inspect logs
- execute inside running containers
- observe different network access paths
- compare control-plane vs data-plane traffic
- see how node-level visibility differs from pod-level visibility

These exercises intentionally mirror the question:

> *“If I had execution here, what could I see?”*

---

### 1. Pod-Level Visibility (Logs and Exec)

Inspect application logs from the running NGINX pod:

```bash
kubectl logs deploy/nginx -n demo
```

Then execute inside the container:

```bash
kubectl exec -it deploy/nginx -n demo -- /bin/sh
```

Notice:
- execution is real, not theoretical
- logs are scoped to the workload
- identity is inherited automatically by the pod

This is the concrete form of “attacker has execution.”

---

### 2. In-Cluster Network Reachability (BusyBox)

Create a temporary pod to test in-cluster access:

```bash
kubectl run bb --image=busybox -n demo -it --restart=Never -- sh
```

From inside the BusyBox shell:

```sh
wget -qO- http://nginx.demo.svc.cluster.local
```

This tests **data-plane networking** and NetworkPolicy behavior.

Reason about:
- what is reachable by default
- what changes when NetworkPolicy is modified
- how lateral movement might begin

---

### 3. Control-Plane Mediated Access (Port Forward)

From your local environment, forward a port to the Service:

```bash
kubectl port-forward svc/nginx -n demo 8080:80
```

Then:

```bash
curl http://localhost:8080
```

Notice:
- access succeeds even if NetworkPolicy would block pod-to-pod ingress
- kubectl is acting as a privileged control-plane proxy

This highlights the difference between **control-plane access**
and **data-plane access**.

---

### 4. Public Access Path (Cloud Load Balancer)

If the Service is of type `LoadBalancer`, retrieve the external IP:

```bash
kubectl get svc nginx -n demo
```

Once an `EXTERNAL-IP` is assigned:

```bash
curl http://<EXTERNAL-IP>
```

Compare:
- port-forward traffic path
- in-cluster traffic path
- public ingress via cloud-managed load balancer

Each path crosses **different boundaries**
and is governed by different controls.

---

## Node-Level Visibility with a DaemonSet (Advanced)

A DaemonSet runs **one pod per node**.

This exercise is not about exploitation.
It is about **making the node boundary visible**.

Inspect DaemonSet pods:

```bash
kubectl get pods -n demo -l app=node-logger
```

Exec into one of the DaemonSet pods:

```bash
kubectl exec -it <node-logger-pod> -n demo -- sh
```

Inspect the log file written on the node filesystem:

```sh
cat /var/log/node-logger/heartbeat.log
```

Reason about:
- how pod visibility differs from node visibility
- what persists beyond individual pod lifecycles
- why DaemonSets are powerful — and potentially risky

This is a concrete example of **crossing abstraction boundaries**.

## Break and Reason

Introduce failure deliberately:
- remove the NetworkPolicy
- change the Service type
- delete and recreate pods
- delete the namespace but not the cluster

Observe both Kubernetes behavior and cloud-side effects.

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

## Reflection

After teardown, reflect:

- Which layers cleaned up automatically?
- Which required human verification?
- Where does Kubernetes abstraction hide risk?

---

## Contributing & Discussion

This repository is intentionally minimal and exploratory.

It is not a best-practices guide or production reference.
It exists as a **shared reasoning surface**.

If you notice inaccuracies, missing considerations, or alternative ways to
reason about these boundaries, you’re invited to:

- open an issue
- submit a pull request
- or reach out directly to the maintainers and contributors

Thoughtful discussion and real-world counterexamples are welcome.
