# Karpenter — Smart Node Autoscaling for EKS ⚡

The modern replacement for Cluster Autoscaler.

---

## The Problem with Cluster Autoscaler

```
Cluster Autoscaler:
  1. Pod is pending (no room)
  2. CA checks node groups
  3. Picks a pre-defined instance type
  4. Launches node (2-3 minutes)
  5. Pod gets scheduled

Problems:
  ❌ You must pre-define node groups with specific instance types
  ❌ Slow (minutes to provision)
  ❌ Can't mix instance types easily
  ❌ Doesn't optimize for cost
```

---

## How Karpenter is Better

```
Karpenter:
  1. Pod is pending (no room)
  2. Karpenter looks at pod requirements (CPU, memory, GPU, arch)
  3. Picks the BEST instance type from ALL available types
  4. Launches node directly via EC2 Fleet API (30-60 seconds)
  5. Pod gets scheduled

Benefits:
  ✅ No pre-defined node groups needed
  ✅ Faster provisioning (seconds, not minutes)
  ✅ Automatically picks cheapest instance that fits
  ✅ Mixes Spot and On-Demand intelligently
  ✅ Consolidates (removes underutilized nodes)
```

---

## Install Karpenter

```bash
# Prerequisites: EKS cluster, IRSA configured

helm repo add karpenter https://charts.karpenter.sh
helm repo update

helm install karpenter karpenter/karpenter \
    --namespace karpenter \
    --create-namespace \
    --set clusterName=my-cluster \
    --set clusterEndpoint=$(aws eks describe-cluster --name my-cluster --query "cluster.endpoint" --output text) \
    --set aws.defaultInstanceProfile=KarpenterNodeInstanceProfile
```

---

## NodePool (What Karpenter Provisions)

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      requirements:
        # Instance types — let Karpenter choose the best
        - key: karpenter.k8s.aws/instance-category
          operator: In
          values: ["c", "m", "r"]        # Compute, General, Memory

        - key: karpenter.k8s.aws/instance-size
          operator: In
          values: ["medium", "large", "xlarge", "2xlarge"]

        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]               # Or arm64 for Graviton

        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot", "on-demand"]   # Prefer Spot, fallback On-Demand

      nodeClassRef:
        name: default

  # Limits — max resources Karpenter can provision
  limits:
    cpu: "100"
    memory: 400Gi

  # Consolidation — remove underutilized nodes
  disruption:
    consolidationPolicy: WhenUnderutilized
    expireAfter: 720h                     # Replace nodes after 30 days
---
apiVersion: karpenter.k8s.aws/v1beta1
kind: EC2NodeClass
metadata:
  name: default
spec:
  amiFamily: AL2
  subnetSelectorTerms:
    - tags:
        karpenter.sh/discovery: my-cluster
  securityGroupSelectorTerms:
    - tags:
        karpenter.sh/discovery: my-cluster
  role: KarpenterNodeRole
```

---

## Karpenter vs Cluster Autoscaler

| Feature | Cluster Autoscaler | Karpenter |
|---------|-------------------|-----------|
| Speed | 2-5 minutes | 30-60 seconds |
| Instance selection | Pre-defined node groups | Automatic, any instance type |
| Spot support | Per node group | Mixed, automatic fallback |
| Consolidation | Basic | Smart (bin-packing) |
| Cost optimization | Manual | Automatic |
| Configuration | Node groups in ASG | NodePool CRD |
| Maintained by | Kubernetes SIG | AWS |

**Use Karpenter** for new EKS clusters. It's the future.
**Use Cluster Autoscaler** if already set up and working fine.

---

*Karpenter = faster, cheaper, smarter node scaling! ⚡*
