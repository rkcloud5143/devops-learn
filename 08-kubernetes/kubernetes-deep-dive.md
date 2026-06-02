# Kubernetes / EKS Deep Dive 🚢

All the key Kubernetes components explained simply — using our pizza shop!

---

## 🍕 The Pizza Shop is Now a Chain

Your pizza shop got so popular you're opening multiple locations. You need a system to manage all the kitchens, cooks, and deliveries. That system is **Kubernetes**.

---

## The Building Blocks (Smallest → Biggest)

### Container
**What it is:** Your app + everything it needs (code, libraries, settings) packed into one portable box. Built with Docker. Runs the same everywhere.

**Pizza shop:** A single pizza-making station — oven, ingredients, recipe card — all in one portable unit. Move it anywhere and it works the same.

### Pod
**What it is:** The smallest unit in Kubernetes. A wrapper around one or more containers that share the same network and storage. Usually it's just one container per pod.

**Pizza shop:** A workstation. It might be just an oven (one container), or an oven + a drink machine side by side (two containers) that share the same counter space.

**Why not just use containers directly?** Kubernetes doesn't manage containers — it manages pods. The pod gives containers a shared IP address and storage, like giving a workstation its own power outlet and counter.

### ReplicaSet (RS)
**What it is:** Makes sure a specific number of identical pods are always running. If one pod dies, the ReplicaSet creates a new one. If there are too many, it kills extras.

**Pizza shop:** A rule that says "always have 3 pizza stations running." If one breaks, the manager immediately sets up a replacement. If someone accidentally sets up a 4th, the manager shuts it down.

**Do you create ReplicaSets directly?** Almost never — Deployments create them for you.

### Deployment
**What it is:** The main way you run apps in Kubernetes. It manages ReplicaSets and handles updates. When you deploy a new version, it gradually replaces old pods with new ones (rolling update) so there's zero downtime.

**Pizza shop:** The franchise operations manual. It says "run 3 stations using recipe v2." When recipe v3 comes out, it swaps stations one at a time — customers never notice the change.

**Deployment → creates a ReplicaSet → which creates Pods → which run Containers**

### Service
**What it is:** A stable address (IP + DNS name) to reach a group of pods. Pods come and go (they get new IPs every time), but the Service stays the same. It load-balances traffic across all matching pods.

**Pizza shop:** The phone number of the shop. Cooks quit and new ones start, but customers always call the same number and reach whoever's working.

**Types:**
- **ClusterIP** — Only reachable inside the cluster. Like an internal extension number.
- **NodePort** — Opens a port on every node. Like putting a sign on every shop door.
- **LoadBalancer** — Creates an external load balancer (on AWS, this creates an ELB). Like a call center that routes to any shop.

### Ingress
**What it is:** Manages external HTTP/HTTPS traffic coming into the cluster. Routes requests to the right Service based on the URL path or hostname. One entry point, many backends.

**Pizza shop:** The front desk of the whole chain. Customer calls and says "I want delivery" → routed to delivery team. "I want catering" → routed to catering team. One phone number, smart routing.

**Example:**
- `mypizza.com/order` → goes to the **order-service**
- `mypizza.com/menu` → goes to the **menu-service**
- `mypizza.com/track` → goes to the **tracking-service**

**Ingress vs Service:** A Service exposes one app. Ingress sits in front of multiple Services and routes traffic based on rules.

### Ingress Controller
**What it is:** Ingress by itself is just rules on paper. The Ingress Controller is the actual software that reads those rules and makes them work (e.g., NGINX, ALB Ingress Controller on AWS).

**Pizza shop:** The front desk person. The routing rules are written on a sheet — the Ingress Controller is the person actually answering the phone and transferring calls.

---

## Configuration & Storage

### ConfigMap
**What it is:** Stores non-secret configuration (settings, URLs, feature flags) as key-value pairs. Pods read from it so you can change config without rebuilding the container.

**Pizza shop:** The settings board. "Oven temp: 450°F, delivery radius: 5 miles." Change the board, all cooks see the update — no need to retrain anyone.

### Secret
**What it is:** Like ConfigMap but for sensitive data (passwords, API keys, certificates). Stored encoded (base64) and can be encrypted at rest with KMS.

**Pizza shop:** The safe with the alarm code and bank account info. Only managers can access it, and it's locked up — not posted on the wall.

### PersistentVolume (PV) & PersistentVolumeClaim (PVC)
**What it is:** Pods are temporary — when they die, their data dies too. PV is a piece of storage (like an EBS volume). PVC is a request for storage ("I need 10GB"). Kubernetes matches PVCs to PVs.

**Pizza shop:** PV = a storage locker in the building. PVC = a cook saying "I need a locker." The manager assigns one. Even if the cook leaves, the locker (and its contents) stays.

---

## Scheduling & Scaling

### Namespace
**What it is:** A virtual divider inside a cluster. Separates environments (dev, staging, prod) or teams so they don't step on each other. Like folders for your Kubernetes resources.

**Pizza shop:** Separate floors in the building. Floor 1 = development kitchen (experiments). Floor 2 = production kitchen (real orders). They share the building but don't interfere.

### Node
**What it is:** A machine (physical or virtual) that runs pods. In EKS, nodes are usually EC2 instances. The cluster is made up of multiple nodes.

**Pizza shop:** A physical kitchen location. You have multiple kitchens (nodes) across town, and Kubernetes decides which kitchen each workstation (pod) goes in.

### DaemonSet
**What it is:** Ensures one copy of a pod runs on every node. Used for things that need to be everywhere — log collectors, monitoring agents, security tools.

**Pizza shop:** A security camera in every kitchen. You don't choose which kitchens get cameras — every single one gets one automatically. New kitchen opens? Camera installed immediately.

### StatefulSet
**What it is:** Like a Deployment, but for apps that need stable identity and persistent storage (databases, message queues). Each pod gets a permanent name (pod-0, pod-1) and its own storage that survives restarts.

**Pizza shop:** Numbered ovens. Oven-0 always has its own recipe book and temperature log. If oven-0 breaks and gets replaced, the new one gets the same number, same recipe book, same log. Identity matters.

### HPA (Horizontal Pod Autoscaler)
**What it is:** Automatically adds or removes pods based on CPU, memory, or custom metrics. "If CPU > 80%, add more pods. If it drops, remove extras."

**Pizza shop:** The manager watching the order board. Rush hour? Open more stations. Slow afternoon? Close some down. Automatically.

### Job & CronJob
**What it is:** Job = run a task once and stop (like a database migration). CronJob = run a task on a schedule (like a nightly backup at 2am).

**Pizza shop:** Job = "deep clean the oven today." CronJob = "deep clean the oven every Sunday at midnight."

---

## How It All Fits Together

```
                    Internet
                       │
                   [ Ingress ]          ← Routes traffic by URL
                       │
              ┌────────┼────────┐
              │        │        │
          [Service] [Service] [Service]  ← Stable addresses
              │        │        │
          [Deploy]  [Deploy]  [StatefulSet]
              │        │        │
          [ReplicaSet] [RS]    │
              │        │        │
           [Pods]   [Pods]   [Pods]      ← Running containers
              │        │        │
    ┌─────────┴────────┴────────┴─────────┐
    │           Nodes (EC2 instances)       │
    │         managed by EKS               │
    │                                      │
    │  [DaemonSet pods on every node]      │
    │  [ConfigMaps & Secrets mounted]      │
    │  [PVs attached where needed]         │
    └──────────────────────────────────────┘
```

---

## Quick Reference

| Component | One-Liner |
|-----------|-----------|
| Container | Your app in a portable box |
| Pod | Smallest deployable unit (wraps containers) |
| ReplicaSet | Keeps N identical pods running |
| Deployment | Manages ReplicaSets + rolling updates |
| Service | Stable address to reach pods |
| Ingress | Routes external HTTP traffic to Services |
| Ingress Controller | The software that makes Ingress rules work |
| ConfigMap | Non-secret config for pods |
| Secret | Sensitive config (passwords, keys) |
| PV / PVC | Persistent storage for pods |
| Namespace | Virtual divider (dev/staging/prod) |
| Node | A machine that runs pods (EC2 in EKS) |
| DaemonSet | One pod on every node |
| StatefulSet | Deployment with stable identity + storage |
| HPA | Auto-scale pods by CPU/memory |
| Job | Run once and done |
| CronJob | Run on a schedule |

---

## EKS-Specific Extras

| EKS Feature | What It Does |
|-------------|-------------|
| Managed Node Groups | AWS manages the EC2 instances (patching, updates) |
| Fargate Profiles | Run pods serverless — no EC2 to manage at all |
| ALB Ingress Controller | Uses AWS ALB as the Ingress Controller |
| IAM Roles for Service Accounts (IRSA) | Give pods specific AWS permissions via IAM |
| EBS CSI Driver | Lets pods use EBS volumes as persistent storage |
| EFS CSI Driver | Lets pods use EFS for shared persistent storage |
| Cluster Autoscaler | Adds/removes EC2 nodes when pods need more room |

---

*Master the building blocks first, then the rest clicks into place! 🧱*
