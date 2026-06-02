# Cloud Cost Optimization — FinOps 💰

Save money without sacrificing performance.

---

## The FinOps Mindset

```
"Cloud is not cheaper than on-prem by default.
 Cloud is cheaper ONLY if you optimize."

FinOps = Financial Operations
  = Engineering + Finance + Business working together on cloud costs
```

---

## AWS Cost Optimization — Quick Wins

### 1. Right-Sizing (Biggest Savings)
```
Problem: Running t3.xlarge when t3.medium is enough.

How to find:
  AWS Cost Explorer → Right Sizing Recommendations
  CloudWatch → CPU/Memory utilization < 40% = oversized

Action:
  Downsize instances
  Use AWS Compute Optimizer recommendations
```

### 2. Reserved Instances & Savings Plans
```
On-Demand:  $0.0416/hr (t3.medium)
1yr RI:     $0.0260/hr (37% savings)
3yr RI:     $0.0166/hr (60% savings)
Savings Plan: Flexible commitment across instance families

Rule: If it runs 24/7, buy Reserved or Savings Plan.
```

### 3. Spot Instances
```
Up to 90% cheaper than On-Demand.
Can be interrupted with 2-minute warning.

Good for:
  ✅ Batch processing, CI/CD runners
  ✅ Kubernetes worker nodes (with fallback)
  ✅ Data processing, ML training
  ✅ Dev/test environments

Bad for:
  ❌ Databases, stateful services
  ❌ Single-instance production
```

### 4. Storage Optimization
```
S3:
  - Lifecycle policies (move to IA after 30 days, Glacier after 90)
  - Delete incomplete multipart uploads
  - Use S3 Intelligent-Tiering for unknown access patterns

EBS:
  - Delete unattached volumes (check: aws ec2 describe-volumes --filters Name=status,Values=available)
  - Use gp3 instead of gp2 (20% cheaper, better performance)
  - Snapshot old volumes, delete the volume

ECR:
  - Lifecycle policies to delete old images (keep last 10)
```

### 5. Unused Resources (Low-Hanging Fruit)
```bash
# Find unattached EBS volumes
aws ec2 describe-volumes --filters Name=status,Values=available

# Find unused Elastic IPs
aws ec2 describe-addresses --filters Name=association-id,Values=""

# Find old snapshots (> 90 days)
aws ec2 describe-snapshots --owner-ids self --query 'Snapshots[?StartTime<`2024-01-01`]'

# Find idle load balancers
aws elbv2 describe-target-health --target-group-arn <arn>
```

### 6. Auto Scaling
```
Scale DOWN when not needed:
  - Dev/staging: shut down nights and weekends (save 65%)
  - Auto Scaling: scale to 0 at night
  - RDS: stop dev databases when not in use

Schedule:
  cron(0 8 ? * MON-FRI *)   # Start at 8am weekdays
  cron(0 20 ? * MON-FRI *)  # Stop at 8pm weekdays
```

### 7. Data Transfer
```
Often the hidden cost surprise.

Reduce:
  - Use VPC endpoints (free for S3/DynamoDB gateway endpoints)
  - Keep traffic in same AZ when possible
  - Use CloudFront for content delivery
  - Compress data before transfer
  - Use PrivateLink instead of NAT Gateway for AWS services
```

---

## Kubernetes Cost Optimization

```
1. Right-size pods
   - Set requests = actual usage (not 2x)
   - Use VPA recommendations
   - kubectl top pods → compare with requests

2. Use Karpenter (not Cluster Autoscaler)
   - Picks cheapest instance type automatically
   - Consolidates underutilized nodes

3. Spot instances for worker nodes
   - Use mixed instance types for availability
   - Keep critical pods on On-Demand

4. Namespace resource quotas
   - Prevent teams from over-provisioning
   - Set limits per namespace

5. Scale to zero
   - KEDA for event-driven scaling
   - Scale dev namespaces to 0 at night
```

---

## Cost Monitoring

```
AWS Tools:
  Cost Explorer          — Visualize spending trends
  AWS Budgets            — Set alerts when spending exceeds threshold
  Cost Anomaly Detection — ML-based unusual spend alerts
  Compute Optimizer      — Right-sizing recommendations

Third-party:
  Kubecost               — Kubernetes cost allocation
  Infracost              — Terraform cost estimation in PRs
  Spot.io                — Automated Spot management
```

### Infracost in CI/CD
```bash
# Show cost impact of Terraform changes in PR
infracost diff --path=. --format=json
# "This change will increase monthly cost by $45"
```

---

## Cost Allocation Tags

```
Tag everything:
  Environment: dev/staging/prod
  Team: platform/backend/frontend
  Project: my-app
  CostCenter: 12345

Then filter in Cost Explorer by tag to see:
  "Team backend spent $5,000 last month"
  "Dev environment costs $2,000 — do we need it 24/7?"
```

---

## Quick Savings Checklist

```
□ Delete unattached EBS volumes
□ Delete unused Elastic IPs
□ Delete old snapshots (> 90 days)
□ Right-size EC2 instances (CPU < 40%)
□ Use gp3 instead of gp2
□ S3 lifecycle policies
□ ECR image lifecycle policies
□ Stop dev/staging at night
□ Buy Savings Plans for steady workloads
□ Use Spot for fault-tolerant workloads
□ Set up AWS Budgets alerts
□ Tag all resources for cost allocation
□ Review Cost Explorer monthly
```

---

*The cheapest resource is the one you don't run! 💰*
