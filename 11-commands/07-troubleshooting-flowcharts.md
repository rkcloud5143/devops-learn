# Troubleshooting Flowcharts 🔍

Step-by-step decision trees for common production incidents.

---

## 🔴 Website/App is Down

```
Is the DNS resolving?  (nslookup myapp.example.com / dig myapp.example.com)
  │
  ├─ NO → Is it a PUBLIC or PRIVATE DNS record?
  │        │
  │        ├─ PUBLIC (internet-facing app)
  │        │    Check Route 53 public hosted zone
  │        │    dig myapp.example.com @8.8.8.8     ← Query public DNS
  │        │    Is the A/CNAME record correct?
  │        │    Is the domain expired? (whois myapp.example.com)
  │        │    Did DNS propagation complete? (TTL still caching old value?)
  │        │    Try: dig myapp.example.com +trace   ← Trace full DNS path
  │        │
  │        └─ PRIVATE (internal app, VPC only)
  │             Check Route 53 private hosted zone
  │             Is the private hosted zone associated with the VPC?
  │             dig myapp.internal @<VPC-DNS-IP>    ← VPC DNS = VPC CIDR base +2
  │             Check: Are you resolving FROM inside the VPC?
  │             Private DNS won't resolve from internet or other VPCs
  │             Need cross-VPC? → VPC peering + hosted zone association
  │             Check DHCP options set → DNS resolution enabled?
  │
  └─ YES → Can you reach the load balancer?
              │
              ├─ NO → Check ALB/NLB
              │        Is it provisioned? (aws elbv2 describe-load-balancers)
              │        Security group allows inbound 80/443?
              │        Is it in public subnets with internet gateway?
              │        Is it internal LB? (scheme: internal won't be reachable from internet)
              │
              └─ YES → Are targets healthy?
                         │
                         ├─ NO → Check target group health
                         │        EC2/ECS health checks failing?
                         │        App not responding on health check path?
                         │        Security group between ALB → targets?
                         │
                         └─ YES → App is reachable but returning errors
                                    Check application logs
                                    Check database connectivity
                                    Check external dependencies
                                    Recent deployment? → Rollback
```

---

## 🟡 App Works for Some Users, Not All

```
Who is it working for vs not?
  │
  ├─ By region/location
  │    → CDN issue (CloudFront caching stale/bad content)
  │      Invalidate cache: aws cloudfront create-invalidation
  │    → DNS propagation (recent change, old TTL still cached)
  │    → Regional service outage (check AWS Health Dashboard)
  │
  ├─ By percentage (random — some requests fail, some don't)
  │    → Load balancer routing to mix of healthy + unhealthy targets
  │      Check: aws elbv2 describe-target-health
  │      One bad pod/instance? Kill it, let ASG/K8s replace
  │    → Canary/weighted deployment in progress?
  │      New version broken? Rollback
  │    → Database connection pool exhausted (some requests get connection, some don't)
  │      Check: max connections, active connections
  │
  ├─ By device/browser
  │    → Frontend issue (CSS/JS not loading, CORS error)
  │      Check browser console (F12)
  │    → TLS/certificate issue (old devices can't negotiate)
  │      Check: openssl s_client -connect host:443
  │
  ├─ Internal users OK, external users NOT
  │    → DNS: Internal using private DNS, external using public DNS
  │      Public DNS pointing to wrong/old resource?
  │    → Firewall/WAF blocking external IPs
  │      Check WAF rules, security groups, NACLs
  │    → NAT/routing issue for external traffic
  │
  └─ Intermittent (works sometimes, fails sometimes)
       → Race condition in code
       → Connection timeouts under load
       → Pod being killed and recreated (check restarts)
         kubectl get pods → RESTARTS column
       → DNS caching returning stale IPs
```

---

## 🐌 App is Slow

```
Is it slow for everyone or specific users?
  │
  ├─ Specific users → CDN issue? Regional issue? Client-side?
  │                    Check CloudFront, check from multiple locations
  │
  └─ Everyone → Check application metrics
                  │
                  ├─ High CPU? → Which process?
                  │               top / kubectl top pods
                  │               Scale up or optimize code
                  │
                  ├─ High Memory? → Memory leak? OOMKilled?
                  │                  Check pod restarts
                  │                  Increase limits or fix leak
                  │
                  ├─ High Disk I/O? → Database queries?
                  │                    iostat -x 1
                  │                    Check slow query log
                  │
                  ├─ Network latency? → External API slow?
                  │                      Check traces (X-Ray/Jaeger)
                  │                      Which service is the bottleneck?
                  │
                  └─ Database slow? → Check connections
                                      Check slow query log
                                      Check CPU/memory on RDS
                                      Missing index?
                                      Add read replica or cache
```

---

## 🔄 Pod CrashLoopBackOff

```
kubectl describe pod <name> → Check Events section
  │
  ├─ OOMKilled → Container ran out of memory
  │               Increase memory limits
  │               Fix memory leak in app
  │
  ├─ Error / CrashLoopBackOff → Check logs
  │   kubectl logs <pod> --previous
  │     │
  │     ├─ Application error → Fix code, check config
  │     ├─ Missing env var / secret → Check ConfigMap/Secret exists
  │     ├─ Can't connect to DB → Check endpoint, credentials, security group
  │     └─ Permission denied → Check RBAC, file permissions, IRSA
  │
  ├─ ImagePullBackOff → Can't pull container image
  │                      Wrong image name/tag?
  │                      Registry auth (imagePullSecrets)?
  │                      ECR login expired?
  │
  └─ Pending (not starting) → Check resources
      kubectl describe pod → Events
        │
        ├─ Insufficient CPU/memory → Scale nodes or reduce requests
        ├─ No matching node (selector/affinity) → Check node labels
        ├─ PVC pending → StorageClass exists? CSI driver running?
        └─ Taint/toleration mismatch → Check node taints
```

---

## 🔒 Can't Connect to Service

```
From where to where?
  │
  ├─ Internet → ALB
  │    Check: DNS, ALB security group (inbound 80/443), ALB in public subnet
  │
  ├─ ALB → Pod
  │    Check: Target group health, pod security group, pod listening on right port
  │
  ├─ Pod → Pod (same cluster)
  │    Check: Service exists? kubectl get endpoints <svc>
  │    DNS working? kubectl exec <pod> -- nslookup <service>
  │    NetworkPolicy blocking?
  │
  ├─ Pod → RDS
  │    Check: RDS security group allows pod SG
  │    Correct endpoint? (not localhost)
  │    Correct credentials?
  │    Max connections reached?
  │
  ├─ Pod → External API
  │    Check: NAT Gateway exists in private subnet route table
  │    Security group allows outbound
  │    DNS resolving? (check CoreDNS)
  │
  └─ SSH → EC2
       Check: Security group allows port 22 from your IP
       Key pair correct?
       Instance running? (not stopped/terminated)
       Disk full? (can prevent SSH)
       Use SSM Session Manager as alternative
```

---

## 💾 Disk Full

```
df -h → Which filesystem is full?
  │
  ├─ /var/log → Logs filling up
  │              journalctl --vacuum-time=3d
  │              find /var/log -name "*.log" -mtime +7 -delete
  │              Set up log rotation
  │
  ├─ /var/lib/docker → Docker eating space
  │                     docker system prune -a
  │                     ECR lifecycle policy (delete old images)
  │                     Check container logs: docker system df
  │
  ├─ / (root) → Find what's using space
  │              du -sh /* | sort -h
  │              find / -size +100M -type f
  │              Old kernels? yum autoremove
  │
  └─ EBS volume → Extend it
                   AWS Console: Modify volume size
                   growpart /dev/xvda 1
                   xfs_growfs / (or resize2fs)
```

---

## 🚀 Deployment Failed

```
Where did it fail?
  │
  ├─ CI (Build/Test) → Check CI logs
  │    Dependency install failed? → Lock file out of sync
  │    Tests failed? → Fix tests or code
  │    Docker build failed? → Check Dockerfile, base image
  │
  ├─ CD (Deploy) → Check deployment logs
  │    Helm upgrade failed? → helm history, check values
  │    kubectl apply failed? → RBAC? Resource quota?
  │    ArgoCD sync failed? → Check ArgoCD UI, Git diff
  │
  ├─ App won't start → Check pod logs
  │    kubectl logs <pod>
  │    kubectl describe pod <pod>
  │    Missing config/secret?
  │    Database migration failed?
  │
  └─ App starts but errors → Check metrics/logs
       Error rate spiking?
       Compare with previous version
       Rollback: kubectl rollout undo deployment/<name>
       Or: helm rollback <release> <revision>
```

---

## ❌ Job / CronJob / Batch Process Failed

```
kubectl get jobs                       → Check status
kubectl describe job <name>            → Check events
  │
  ├─ Pod never started
  │    kubectl describe pod <job-pod>
  │    → Image pull error? Check image name, registry auth
  │    → Insufficient resources? Check node capacity
  │    → PVC not bound? Check storage class
  │
  ├─ Pod started but failed
  │    kubectl logs <job-pod>          → Application error
  │    kubectl logs <job-pod> --previous → If restarted
  │    │
  │    ├─ Exit code 1 → Application error (check logs for stack trace)
  │    ├─ Exit code 137 → OOMKilled (out of memory)
  │    │                   kubectl describe pod → Last State → Reason: OOMKilled
  │    │                   Increase memory limits
  │    ├─ Exit code 143 → SIGTERM (killed by K8s — deadline exceeded?)
  │    │                   Check activeDeadlineSeconds in job spec
  │    └─ Exit code 126/127 → Command not found / permission denied
  │                            Check entrypoint/command in container
  │
  ├─ Job keeps retrying (backoffLimit not reached)
  │    Check: spec.backoffLimit (default: 6)
  │    Each retry logs are in different pods:
  │    kubectl get pods -l job-name=<name> → Check ALL pods
  │    kubectl logs <pod-attempt-1>
  │    kubectl logs <pod-attempt-2>
  │
  ├─ CronJob didn't run at all
  │    kubectl get cronjobs → LAST SCHEDULE column
  │    Is it suspended? (spec.suspend: true)
  │    Cron expression correct? (use crontab.guru to verify)
  │    Check: spec.concurrencyPolicy (Forbid = skips if previous still running)
  │    Check: spec.startingDeadlineSeconds
  │
  └─ Job succeeded but results are wrong
       Check input data / environment variables
       Check database state before and after
       Run job manually with debug logging:
       kubectl create job test-run --from=cronjob/<name>
```

### Finding Root Cause of Any Failed Job
```bash
# 1. Find the job and its pods
kubectl get jobs
kubectl get pods -l job-name=<job-name> --sort-by='.status.startTime'

# 2. Check the latest pod
kubectl describe pod <latest-pod>     # Events, exit code, reason
kubectl logs <latest-pod>             # Application output

# 3. Check previous attempts
kubectl logs <previous-pod>

# 4. Check events timeline
kubectl get events --sort-by='.lastTimestamp' --field-selector involvedObject.name=<pod>

# 5. If OOMKilled — check actual memory usage
kubectl top pod <pod>                 # Current usage
kubectl describe pod <pod> | grep -A3 "Last State"  # How it died
```

---

## 🔗 Service-to-Service Latency / Communication Issues

```
How to find which service is slow:
  │
  ├─ TRACES (best method)
  │    Open Jaeger / X-Ray / Datadog APM / Tempo
  │    Search by: trace ID, service name, or min duration
  │    │
  │    │  Request flow:
  │    │  API Gateway (2ms) → Auth (15ms) → Order Service (1.2s) → DB (5ms)
  │    │                                     ↑
  │    │                              THIS is the bottleneck
  │    │
  │    │  Look for:
  │    │  - Spans with high duration
  │    │  - Gaps between spans (network latency)
  │    │  - Retries (same span repeated)
  │    │  - Errors in child spans
  │    │
  │    No tracing set up? → Use manual methods below
  │
  ├─ METRICS (dashboard method)
  │    Check per-service response times:
  │    - Prometheus: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket{service="order"}[5m]))
  │    - CloudWatch: Target Response Time on ALB
  │    - Datadog: APM service map
  │    Compare: Which service's latency increased?
  │
  ├─ LOGS (grep method)
  │    Add request timing to your logs:
  │    "GET /api/orders completed in 1200ms (db: 5ms, payment-api: 1150ms)"
  │    Search for slow requests:
  │    kubectl logs <pod> | grep -E "completed in [0-9]{4,}ms"  # > 1000ms
  │
  └─ MANUAL TESTING (quick & dirty)
       # From inside a pod, test latency to other services:
       kubectl exec <pod> -- curl -w "\n  DNS: %{time_namelookup}s\n  Connect: %{time_connect}s\n  TLS: %{time_appconnect}s\n  First byte: %{time_starttransfer}s\n  Total: %{time_total}s\n" -o /dev/null -s http://service-name:8080/health

       # Output:
       #   DNS: 0.002s
       #   Connect: 0.003s        ← Network latency
       #   TLS: 0.000s
       #   First byte: 0.250s    ← Server processing time
       #   Total: 0.251s

       # Test from different pods to isolate:
       # Is it slow from ALL pods? → Target service is slow
       # Is it slow from ONE pod? → Source pod or its node has issues
```

### Common Causes of Inter-Service Latency
```
Cause                          How to identify                    Fix
─────────────────────────      ──────────────────────────         ──────────────────────
DNS resolution slow            curl timing shows high DNS         Check CoreDNS pods, add ndots:1
Service overloaded             High CPU/memory on target          Scale up, optimize code
Connection pool exhausted      "connection timeout" in logs       Increase pool size, add RDS Proxy
Network policy blocking        Requests timeout completely        Check NetworkPolicy rules
Cross-AZ traffic               Consistent ~1-2ms extra latency   Keep pods in same AZ (topology)
Sidecar proxy overhead         ~1ms added per hop (Istio)        Normal for service mesh
Cold start (Lambda/Fargate)    First request slow, rest fast      Provisioned concurrency
External API slow              Trace shows long external span     Add timeout, circuit breaker, cache
Database slow                  Trace shows long DB span           Check slow query log, add index
```

### The curl Timing Cheat Sheet
```bash
# Save this as ~/curl-timing.sh
curl -w "
    DNS Lookup:   %{time_namelookup}s
    TCP Connect:  %{time_connect}s
    TLS Handshake:%{time_appconnect}s
    First Byte:   %{time_starttransfer}s
    Total Time:   %{time_total}s
    HTTP Code:    %{http_code}
" -o /dev/null -s "$1"

# Usage: bash curl-timing.sh http://service:8080/health
```

---

## 📊 High AWS Bill

```
AWS Cost Explorer → Filter by service
  │
  ├─ EC2 high → Right-size instances (CPU < 40% = oversized)
  │              Buy Savings Plans for steady workloads
  │              Use Spot for fault-tolerant
  │              Stop dev/staging at night
  │
  ├─ Data Transfer high → Check NAT Gateway costs
  │                        Use VPC endpoints for S3/DynamoDB
  │                        Keep traffic in same AZ
  │                        Use CloudFront
  │
  ├─ RDS high → Right-size instance
  │              Use Reserved Instances
  │              Stop dev databases when not in use
  │
  ├─ S3 high → Lifecycle policies (move to IA/Glacier)
  │             Delete old versions
  │             Check request costs (too many small requests?)
  │
  └─ EBS high → Delete unattached volumes
                 Delete old snapshots
                 Use gp3 instead of gp2
```

---

## 🔑 Quick Debug Commands

```bash
# ─── Is it DNS? ───
nslookup myapp.example.com
dig +short myapp.example.com

# ─── Is it network? ───
curl -v https://myapp.example.com
telnet host 443
nc -zv host 443

# ─── Is it the app? ───
kubectl logs <pod> -f
kubectl describe pod <pod>
kubectl exec <pod> -- curl localhost:8080/health

# ─── Is it resources? ───
kubectl top pods
kubectl top nodes
df -h
free -h

# ─── Is it recent? ───
kubectl rollout history deployment/<name>
helm history <release>
git log --oneline -10

# ─── Nuclear option ───
kubectl rollout undo deployment/<name>
helm rollback <release> <revision>
```

---

*Follow the flowchart, don't panic, fix the symptom first, investigate root cause second! 🔍*
