# Senior DevOps Engineer — Questions & Answers 🎯

Real-world questions covering the depth expected at a senior level.

---

## AWS & Cloud Architecture

**Q: How would you design a multi-region active-active architecture on AWS?**

A: Key components:
- **Route 53** with latency-based or geolocation routing to direct users to the nearest region
- **DynamoDB Global Tables** or **Aurora Global Database** for multi-region data replication
- **S3 Cross-Region Replication** for static assets
- **Application deployed in both regions** behind regional ALBs
- **CloudFront** in front for caching and faster failover
- **Health checks** at Route 53 level to remove unhealthy regions

Challenges to address: data consistency (eventual vs strong), conflict resolution, cost (data transfer between regions), and deployment coordination.

---

**Q: Explain the difference between a NAT Gateway and a NAT Instance. When would you use each?**

A: 
| Aspect | NAT Gateway | NAT Instance |
|--------|-------------|--------------|
| Managed by | AWS (fully managed) | You (EC2 instance) |
| Availability | Highly available in AZ | Single point of failure unless you script HA |
| Bandwidth | Up to 100 Gbps | Depends on instance type |
| Cost | Higher hourly + data processing | Lower (just EC2 cost) |
| Maintenance | None | Patching, monitoring, scaling |

**Use NAT Gateway** for production — it's managed, scalable, and highly available.
**Use NAT Instance** for dev/test environments where cost matters more than reliability, or when you need to customize (e.g., run a proxy, do packet inspection).

---

**Q: A Lambda function is timing out when connecting to RDS in a private subnet. What's wrong?**

A: Common causes:
1. **No NAT Gateway** — Lambda in a VPC needs a NAT Gateway to reach the internet (for AWS APIs) or VPC endpoints
2. **Security group misconfiguration** — RDS security group doesn't allow inbound from Lambda's security group
3. **Lambda not in the same VPC/subnets** — Lambda must be configured with the VPC, subnets, and security group
4. **RDS not in private subnet accessible to Lambda** — Check route tables
5. **Connection pool exhaustion** — Lambda scales fast; RDS connections max out. Solution: use **RDS Proxy**

Debugging: Check VPC Flow Logs, Lambda execution role permissions, and RDS connection limits.

---

**Q: How do you handle secrets in a CI/CD pipeline?**

A: Never store secrets in code or environment variables in plain text.

Options:
- **AWS Secrets Manager** or **SSM Parameter Store (SecureString)** — fetch at runtime
- **HashiCorp Vault** — centralized secrets management
- **CI/CD native secrets** (GitHub Secrets, GitLab CI Variables) — encrypted, injected at build time

Best practices:
- Rotate secrets automatically (Secrets Manager supports this)
- Use IAM roles instead of access keys where possible
- Audit access via CloudTrail
- Use short-lived credentials (STS AssumeRole)

---

## Kubernetes & Containers

**Q: A pod is stuck in CrashLoopBackOff. Walk me through your debugging process.**

A:
```bash
# 1. Check pod status and events
kubectl describe pod <pod-name>

# 2. Check logs (current and previous crash)
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# 3. Check if it's a resource issue
kubectl top pod <pod-name>

# 4. Check if it's a config/secret issue
kubectl get pod <pod-name> -o yaml | grep -A5 env

# 5. Exec into the container (if it stays up long enough)
kubectl exec -it <pod-name> -- /bin/sh
```

Common causes:
- Application error (check logs)
- Missing ConfigMap/Secret
- Liveness probe failing too aggressively
- OOMKilled (check `kubectl describe` for "Last State")
- Image pull error (wrong tag, no auth)

---

**Q: Explain the difference between a Deployment, StatefulSet, and DaemonSet. When do you use each?**

A:
| Type | Use Case | Key Feature |
|------|----------|-------------|
| **Deployment** | Stateless apps (web servers, APIs) | Rolling updates, easy scaling, pods are interchangeable |
| **StatefulSet** | Stateful apps (databases, Kafka, Zookeeper) | Stable pod names (pod-0, pod-1), persistent storage per pod, ordered startup/shutdown |
| **DaemonSet** | One pod per node (log collectors, monitoring agents) | Automatically runs on every node (or selected nodes) |

Example:
- NGINX frontend → Deployment
- PostgreSQL cluster → StatefulSet
- Fluentd log shipper → DaemonSet

---

**Q: How does Kubernetes networking work? Explain pod-to-pod communication.**

A: Kubernetes networking model:
1. **Every pod gets its own IP** — no NAT between pods
2. **All pods can reach all other pods** across nodes without NAT
3. **Agents on a node can reach all pods on that node**

How it works:
- **CNI plugin** (Calico, Cilium, AWS VPC CNI) sets up the network
- Each node gets a CIDR range for pods
- Pod IPs are routable within the cluster
- **Services** provide stable IPs and DNS names (ClusterIP)
- **kube-proxy** (or eBPF in Cilium) handles routing to pod backends

On EKS with AWS VPC CNI: Pods get real VPC IPs from the subnet, so they're directly routable from other AWS resources.

---

**Q: How would you implement zero-downtime deployments in Kubernetes?**

A:
1. **Rolling update strategy** (default in Deployments)
   ```yaml
   strategy:
     type: RollingUpdate
     rollingUpdate:
       maxUnavailable: 0
       maxSurge: 1
   ```

2. **Readiness probes** — new pods only receive traffic when ready
   ```yaml
   readinessProbe:
     httpGet:
       path: /health
       port: 8080
     initialDelaySeconds: 5
     periodSeconds: 5
   ```

3. **PodDisruptionBudget** — ensure minimum available during voluntary disruptions
   ```yaml
   apiVersion: policy/v1
   kind: PodDisruptionBudget
   spec:
     minAvailable: 2
     selector:
       matchLabels:
         app: myapp
   ```

4. **Graceful shutdown** — handle SIGTERM in your app, use `preStop` hooks if needed

5. **Blue-green or canary** — use Argo Rollouts or Flagger for advanced strategies

---

## CI/CD & Automation

**Q: How would you design a CI/CD pipeline for a microservices architecture?**

A:
```
Code Push → Build → Test → Security Scan → Build Image → Push to Registry → Deploy to Dev → Integration Tests → Deploy to Staging → Approval → Deploy to Prod
```

Key principles:
- **Mono-repo or multi-repo** — affects pipeline triggers
- **Build once, deploy many** — same artifact goes dev → staging → prod
- **Infrastructure as code** — Terraform/CloudFormation in the same pipeline
- **Feature flags** — decouple deployment from release
- **Automated rollback** — if health checks fail post-deploy

Tools: GitHub Actions, GitLab CI, ArgoCD (GitOps), Jenkins, Tekton

For microservices specifically:
- Only build/deploy services that changed (path filters)
- Shared libraries versioned and published separately
- Contract testing between services (Pact)
- Centralized secrets management

---

**Q: Explain GitOps. How is it different from traditional CI/CD?**

A:
**Traditional CI/CD:** Pipeline pushes changes to the cluster (push-based)
```
CI builds image → CD runs kubectl apply → Cluster updated
```

**GitOps:** Cluster pulls changes from Git (pull-based)
```
CI builds image → Updates Git manifest → ArgoCD detects change → Cluster syncs
```

Key differences:
| Aspect | Traditional | GitOps |
|--------|-------------|--------|
| Source of truth | CI/CD pipeline | Git repository |
| Deployment | Push (CI has cluster access) | Pull (agent in cluster pulls) |
| Drift detection | Manual | Automatic (agent reconciles) |
| Rollback | Re-run pipeline | Git revert |
| Audit trail | CI logs | Git history |

Tools: ArgoCD, Flux

Benefits: Better security (CI doesn't need cluster creds), automatic drift correction, Git as audit log.

---

## Terraform & IaC

**Q: How do you manage Terraform state in a team environment?**

A:
1. **Remote backend** — S3 + DynamoDB for locking
   ```hcl
   terraform {
     backend "s3" {
       bucket         = "my-terraform-state"
       key            = "prod/terraform.tfstate"
       region         = "us-east-1"
       dynamodb_table = "terraform-locks"
       encrypt        = true
     }
   }
   ```

2. **State locking** — DynamoDB prevents concurrent modifications

3. **Workspaces or separate state files** — isolate environments (dev/staging/prod)

4. **State file security** — encrypt at rest, restrict access via IAM

5. **Never commit state to Git** — contains secrets in plain text

6. **Use `terraform plan` in CI** — review changes before apply

7. **Import existing resources** — `terraform import` to bring unmanaged resources under control

---

**Q: How do you handle Terraform module versioning?**

A:
```hcl
module "vpc" {
  source  = "git::https://github.com/myorg/terraform-modules.git//vpc?ref=v1.2.0"
}

# Or from Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"
}
```

Best practices:
- **Pin versions** — never use `main` or `latest` in production
- **Semantic versioning** — breaking changes = major bump
- **Changelog** — document what changed in each version
- **Test modules** — use Terratest or `terraform validate`
- **Separate repo for modules** — reusable across projects

---

## Monitoring & Observability

**Q: What's the difference between metrics, logs, and traces? How do they work together?**

A: The three pillars of observability:

| Pillar | What it is | Example | Tool |
|--------|------------|---------|------|
| **Metrics** | Numeric measurements over time | CPU at 80%, 500 requests/sec | Prometheus, CloudWatch, Datadog |
| **Logs** | Timestamped text records | "Error: connection refused" | ELK, CloudWatch Logs, Loki |
| **Traces** | Request flow across services | Request took 2s: 1.5s in DB, 0.5s in API | Jaeger, X-Ray, Datadog APM |

How they work together:
1. **Alert fires** on a metric (error rate > 5%)
2. **Check traces** to see which service is slow
3. **Read logs** from that service to find the root cause

Correlation: Use a **trace ID** that appears in logs, metrics tags, and traces so you can jump between them.

---

**Q: How would you set up alerting that doesn't cause alert fatigue?**

A:
1. **Alert on symptoms, not causes** — "Users can't log in" not "CPU is high"

2. **Tiered severity:**
   - **Critical** — pages on-call, needs immediate action
   - **Warning** — Slack notification, investigate soon
   - **Info** — logged for review

3. **Actionable alerts only** — if there's no action to take, it's not an alert

4. **Include runbook links** — alert message should link to how to fix it

5. **Set proper thresholds** — use historical data, not guesses

6. **Aggregate related alerts** — don't send 100 alerts for 100 pods; group them

7. **Auto-resolve** — if the condition clears, close the alert

8. **Review regularly** — delete alerts nobody acts on

---

## Linux & Troubleshooting

**Q: A server is slow. Walk me through your debugging process.**

A:
```bash
# 1. Check load average
uptime
# Load > number of CPUs = overloaded

# 2. Check CPU, memory, I/O
top
htop
vmstat 1 5

# 3. Check disk I/O
iostat -x 1 5
# %util near 100% = disk bottleneck

# 4. Check disk space
df -h
# Full disk can cause all sorts of issues

# 5. Check memory
free -h
# High swap usage = memory pressure

# 6. Check network
ss -tuln          # listening ports
ss -s             # connection summary
iftop             # bandwidth by connection

# 7. Check what processes are doing
pidstat 1 5
strace -p <pid>   # system calls (last resort)

# 8. Check logs
dmesg | tail      # kernel messages
journalctl -xe    # systemd logs
```

Common culprits: runaway process, disk full, memory leak, network saturation, too many connections.

---

**Q: Explain what happens when you type `curl https://example.com` and press enter.**

A:
1. **DNS resolution** — OS checks `/etc/hosts`, then queries DNS resolver → gets IP address
2. **TCP handshake** — SYN → SYN-ACK → ACK (three-way handshake to port 443)
3. **TLS handshake** — Client Hello → Server Hello + Certificate → Key exchange → Encrypted session established
4. **HTTP request** — `GET / HTTP/1.1` sent over encrypted connection
5. **Server processes** — Web server receives request, generates response
6. **HTTP response** — Status code + headers + body sent back
7. **Rendering** — curl prints the body to stdout
8. **Connection close** — TCP FIN handshake (or kept alive for reuse)

If any step fails: DNS error, connection refused, certificate error, timeout, HTTP error code.

---

## Scenario-Based

**Q: Your production database is at 95% CPU. What do you do?**

A: Immediate actions:
1. **Don't panic** — check if it's a spike or sustained
2. **Check slow query log** — identify expensive queries
3. **Check connections** — too many? Connection pooling issue?
4. **Check for locks** — `SHOW PROCESSLIST` (MySQL) or `pg_stat_activity` (Postgres)
5. **Scale up temporarily** — if RDS, increase instance size (causes brief downtime)
6. **Add read replica** — offload read traffic

Root cause investigation:
- New deployment? Rollback if needed
- Missing index? Add it
- Traffic spike? Scale horizontally or add caching
- Inefficient query? Optimize or cache results

Long-term:
- Set up CloudWatch alarms before 95%
- Implement query performance monitoring
- Consider caching layer (ElastiCache)
- Review database schema and indexes

---

**Q: A deployment went out and now error rates are spiking. What's your process?**

A:
1. **Assess severity** — How many users affected? Is it total outage or degraded?

2. **Decide: rollback or fix forward**
   - If simple fix is known → fix forward
   - If unclear → rollback immediately

3. **Rollback** (if needed)
   ```bash
   kubectl rollout undo deployment/myapp
   # or
   argocd app rollback myapp
   ```

4. **Communicate** — Update status page, notify stakeholders

5. **Investigate** (after stability restored)
   - Compare metrics before/after deploy
   - Check logs for new errors
   - Review the diff of what changed
   - Check traces for latency changes

6. **Post-mortem** — Document what happened, why, and how to prevent it

Key principle: **Restore service first, investigate second.** Don't debug in production while users are down.

---

*These questions test real-world experience, not just textbook knowledge. Practice explaining your thought process out loud. 💪*
