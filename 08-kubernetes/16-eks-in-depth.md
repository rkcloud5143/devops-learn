# EKS — In-Depth Guide 🚀

Beyond basics: how EKS actually works, networking, IRSA, add-ons, upgrades, and production patterns.

---

## EKS Architecture (What AWS Manages vs You)

```
┌─── AWS Manages (Control Plane — $0.10/hr) ──────────────────────┐
│                                                                    │
│  API Server (multi-AZ, auto-scaled)                               │
│  etcd (encrypted, backed up, multi-AZ)                            │
│  Scheduler + Controller Manager                                    │
│  Cloud Controller Manager (integrates with AWS APIs)              │
│                                                                    │
│  You NEVER SSH into this. You interact via kubectl / API only.    │
└────────────────────────────────────────────────────────────────────┘
         │
         │ kubelet communicates with API server
         │
┌─── YOU Manage (Data Plane — Worker Nodes) ─────────────────────────┐
│                                                                      │
│  Option A: Managed Node Groups                                      │
│    EC2 instances managed by EKS (AMI updates, scaling, draining)   │
│    You choose: instance type, min/max size, AMI                     │
│                                                                      │
│  Option B: Karpenter                                                │
│    Auto-provisions right-sized nodes based on pending pods          │
│    Picks cheapest instance type, mixes Spot/On-Demand              │
│                                                                      │
│  Option C: Fargate                                                  │
│    Serverless — no nodes at all. AWS runs each pod in microVM      │
│    Pay per pod (vCPU + memory per second)                           │
│    Limitations: no DaemonSets, no GPU, no privileged containers    │
│                                                                      │
│  Option D: Self-managed nodes (full control, most work)            │
└──────────────────────────────────────────────────────────────────────┘
```

---

## EKS Networking (VPC CNI)

```
Traditional K8s: Pod gets VIRTUAL IP → NAT to reach outside cluster
EKS with VPC CNI: Pod gets REAL VPC IP → directly routable

Pod IP = VPC IP from subnet CIDR
  → Pods can talk to RDS, ElastiCache, etc. directly
  → No NAT, no overlay network
  → Security groups work at pod level

Consequence:
  Subnet size limits pod count!
  /24 subnet = 256 IPs = limited pods
  Solution: Use secondary CIDR for pods (100.64.0.0/16)
```

### Service Networking
```
ClusterIP service → kube-proxy handles routing (iptables or IPVS)
NodePort → Opens port on every node
LoadBalancer → AWS Load Balancer Controller creates ALB/NLB

Ingress with ALB:
  Ingress resource → AWS LB Controller → Creates ALB → Routes to pods
  Annotations control: scheme (internal/internet-facing), SSL, WAF

Ingress with NLB:
  Service type: LoadBalancer → Creates NLB
  Use for: TCP/UDP, extreme performance, static IPs
```

---

## IRSA (IAM Roles for Service Accounts)

```
Problem: How does a pod access AWS services (S3, Secrets Manager)?

Old way: Instance role on the node → EVERY pod on that node gets same permissions 😬
New way: IRSA → Each pod gets its own IAM role via ServiceAccount

How it works:
  1. EKS has an OIDC provider (identity bridge)
  2. IAM Role trusts the OIDC provider
  3. ServiceAccount annotated with IAM Role ARN
  4. Pod uses ServiceAccount → gets temporary credentials for THAT role only

Setup:
  aws eks describe-cluster --name my-cluster --query "cluster.identity.oidc"
  # Get OIDC provider URL

  # Create IAM Role with trust policy:
  {
    "Effect": "Allow",
    "Principal": {
      "Federated": "arn:aws:iam::ACCOUNT:oidc-provider/oidc.eks.REGION.amazonaws.com/id/CLUSTER_ID"
    },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": {
        "oidc.eks.REGION.amazonaws.com/id/CLUSTER_ID:sub": "system:serviceaccount:NAMESPACE:SA_NAME"
      }
    }
  }

  # Annotate ServiceAccount:
  kubectl annotate serviceaccount my-sa \
    eks.amazonaws.com/role-arn=arn:aws:iam::123456789012:role/MyPodRole
```

---

## EKS Add-ons (Managed Components)

| Add-on | What it does | Managed? |
|--------|-------------|----------|
| VPC CNI | Pod networking (real VPC IPs) | Yes (EKS add-on) |
| CoreDNS | Cluster DNS | Yes (EKS add-on) |
| kube-proxy | Service routing | Yes (EKS add-on) |
| EBS CSI Driver | EBS PersistentVolumes | Yes (EKS add-on) |
| EFS CSI Driver | EFS shared volumes | Self-managed |
| AWS LB Controller | ALB/NLB from Ingress/Service | Self-managed (Helm) |
| External Secrets | AWS Secrets → K8s Secrets | Self-managed (Helm) |
| Karpenter | Node auto-provisioning | Self-managed (Helm) |
| Metrics Server | kubectl top pods/nodes | Self-managed |
| Cluster Autoscaler | Node scaling (legacy) | Self-managed |

---

## EKS Upgrades

```
EKS supports N-3 Kubernetes versions. You MUST upgrade regularly.

Upgrade order:
  1. Control plane (AWS does this — takes ~25 min)
  2. Add-ons (VPC CNI, CoreDNS, kube-proxy)
  3. Worker nodes (new AMI with matching kubelet version)

Strategy:
  - Blue/green node groups (safest):
    Create new node group → drain old → delete old
  - In-place rolling update:
    Update launch template → instance refresh → nodes replaced

Commands:
  aws eks update-cluster-version --name my-cluster --kubernetes-version 1.29
  aws eks update-addon --cluster-name my-cluster --addon-name vpc-cni --addon-version v1.16.0
  eksctl upgrade nodegroup --cluster=my-cluster --name=workers
```

---

## Production EKS Patterns

```
1. Node Strategy:
   Managed node group (system/infra) + Karpenter (workloads)
   System nodes: tainted CriticalAddonsOnly → only infra pods land here

2. Multi-AZ:
   Nodes spread across 3 AZs
   Pod topology constraints to spread replicas

3. Secrets:
   External Secrets Operator → pulls from AWS Secrets Manager
   SOPS for GitOps secrets in Git

4. Monitoring:
   Datadog DaemonSet (or Prometheus) on every node
   CloudWatch Container Insights for AWS-native

5. Autoscaling:
   HPA for pods (CPU/memory/custom metrics)
   Karpenter for nodes (automatic right-sizing)

6. Security:
   IRSA for pod-level AWS access
   Network Policies (Calico or Cilium)
   Pod Security Standards: Restricted
   No exec into production pods
```

---

*EKS = Kubernetes without managing the control plane. Master the AWS integrations! 🚀*
