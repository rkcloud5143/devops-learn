# Scenario-Based Q&A (100+ Questions)

Real-world scenarios combining multiple technologies. These test practical problem-solving skills.

---

## Production Incidents

**Q1: Your website is down. Walk me through your troubleshooting process.**
A:
1. **Verify the issue** — Can you reproduce? Check from multiple locations
2. **Check monitoring** — CloudWatch, Datadog alerts
3. **Check recent changes** — Deployments, config changes
4. **Check infrastructure** — EC2 status, ELB health, RDS status
5. **Check application** — Logs, error rates, response times
6. **Check dependencies** — External APIs, databases, caches
7. **Communicate** — Update status page, notify stakeholders
8. **Fix and document** — Resolve, write post-mortem

**Q2: Database CPU is at 100%. What do you do?**
A:
1. **Immediate** — Check slow query log, identify expensive queries
2. **Check connections** — Too many? Connection pool exhausted?
3. **Check for locks** — Blocking queries?
4. **Scale if needed** — Increase instance size (brief downtime)
5. **Long-term** — Add read replicas, optimize queries, add caching

**Q3: A deployment went out and errors are spiking. What's your process?**
A:
1. **Assess severity** — How many users affected?
2. **Decide: rollback or fix forward** — If unclear, rollback
3. **Rollback** — `kubectl rollout undo` or redeploy previous version
4. **Communicate** — Update status page
5. **Investigate** — Compare metrics, check logs, review diff
6. **Post-mortem** — Document and prevent recurrence

**Q4: Users report the app is slow. How do you investigate?**
A:
1. **Reproduce** — Is it slow for you? Specific pages?
2. **Check metrics** — Response times, error rates, throughput
3. **Check infrastructure** — CPU, memory, disk I/O, network
4. **Check application** — APM traces, slow queries, external calls
5. **Check dependencies** — Database, cache, third-party APIs
6. **Narrow down** — Which component is the bottleneck?

**Q5: You're getting paged at 3am for the same alert repeatedly. How do you fix this?**
A:
1. **Investigate root cause** — Why does this keep happening?
2. **Fix the underlying issue** — Not just the symptom
3. **Improve alerting** — Is the threshold right? Is it actionable?
4. **Add automation** — Can it self-heal?
5. **Document** — Runbook for future incidents
6. **Review** — Should this page at 3am or wait until morning?

**Q6: A server is running out of disk space. What do you do?**
A:
```bash
df -h                           # Which filesystem?
du -sh /* | sort -h             # What's using space?
find / -size +100M -type f      # Large files
journalctl --vacuum-time=7d     # Clean old logs
docker system prune             # Clean Docker
```
Long-term: Add monitoring, set up log rotation, increase disk size.

**Q7: An EC2 instance is unreachable. How do you troubleshoot?**
A:
1. **Check instance status** — AWS Console, instance state
2. **Check system status checks** — Hardware issues?
3. **Check security groups** — Port 22 open?
4. **Check NACLs** — Subnet-level blocking?
5. **Check route tables** — Can traffic reach the instance?
6. **Use EC2 Serial Console** — If SSH doesn't work
7. **Check instance logs** — System Log in console

**Q8: A Lambda function is timing out. What do you check?**
A:
1. **Timeout setting** — Is it long enough?
2. **VPC configuration** — Needs NAT Gateway for internet
3. **Cold starts** — First invocation slower
4. **Dependencies** — External API slow? Database connection?
5. **Code** — Inefficient loops? Large payloads?
6. **Memory** — More memory = more CPU

**Q9: Kubernetes pods are in CrashLoopBackOff. How do you debug?**
A:
```bash
kubectl describe pod <name>     # Events, state
kubectl logs <name> --previous  # Previous crash logs
kubectl get events              # Cluster events
```
Common causes: App error, missing config/secret, failed health check, OOMKilled.

**Q10: A service can't connect to RDS. What do you check?**
A:
1. **Security groups** — RDS SG allows inbound from app SG?
2. **Network** — Same VPC? Correct subnets?
3. **Credentials** — Correct username/password?
4. **Endpoint** — Using correct RDS endpoint?
5. **Database exists** — Correct database name?
6. **Connection limits** — Max connections reached?

---

## Architecture & Design

**Q11: How would you design a highly available web application on AWS?**
A:
- **Multi-AZ** — Resources in at least 2 AZs
- **Load balancer** — ALB distributing traffic
- **Auto Scaling** — Handle traffic spikes
- **RDS Multi-AZ** — Database failover
- **ElastiCache** — Reduce database load
- **CloudFront** — Cache static content
- **Route 53** — DNS with health checks
- **S3** — Static assets, backups

**Q12: How would you design a CI/CD pipeline for microservices?**
A:
- **Trigger** — On push to main or PR
- **Build** — Build only changed services (path filters)
- **Test** — Unit tests, integration tests
- **Security** — SAST, dependency scanning, image scanning
- **Artifact** — Build and push Docker image
- **Deploy to dev** — Automatic
- **Deploy to staging** — Automatic with integration tests
- **Deploy to prod** — Manual approval or canary

**Q13: How would you migrate a monolith to microservices?**
A:
1. **Strangler pattern** — Gradually extract services
2. **Start with boundaries** — Identify loosely coupled components
3. **API gateway** — Route traffic to monolith or microservice
4. **Database per service** — Eventually, but can share initially
5. **Incremental** — One service at a time
6. **Monitoring** — Distributed tracing essential

**Q14: How would you implement disaster recovery for a critical application?**
A:
- **RPO/RTO** — Define acceptable data loss and downtime
- **Backup strategy** — Regular backups, test restores
- **Multi-region** — Pilot light, warm standby, or active-active
- **Data replication** — RDS cross-region replicas, S3 CRR
- **DNS failover** — Route 53 health checks
- **Runbooks** — Documented procedures
- **Regular drills** — Test the DR process

**Q15: How would you secure a Kubernetes cluster?**
A:
- **RBAC** — Least privilege access
- **Network policies** — Restrict pod communication
- **Pod security** — Non-root, read-only filesystem
- **Secrets** — External secrets manager, encryption at rest
- **Image security** — Scan images, use trusted registries
- **Audit logging** — Track API calls
- **Updates** — Keep cluster and nodes patched

**Q16: How would you handle secrets in a microservices architecture?**
A:
- **Centralized** — AWS Secrets Manager, HashiCorp Vault
- **Rotation** — Automatic rotation where possible
- **Access control** — IAM policies, Vault policies
- **Injection** — Environment variables or mounted files
- **Audit** — Log access to secrets
- **Never in code** — No secrets in Git

**Q17: How would you implement blue-green deployment?**
A:
1. **Two environments** — Blue (current), Green (new)
2. **Deploy to green** — New version
3. **Test green** — Smoke tests, health checks
4. **Switch traffic** — Update load balancer or DNS
5. **Monitor** — Watch for errors
6. **Rollback** — Switch back to blue if issues
7. **Clean up** — Blue becomes next green

**Q18: How would you design a logging and monitoring strategy?**
A:
- **Metrics** — CloudWatch, Prometheus, Datadog
- **Logs** — Centralized (CloudWatch Logs, ELK, Loki)
- **Traces** — Distributed tracing (X-Ray, Jaeger)
- **Correlation** — Trace IDs across all three
- **Dashboards** — Key metrics visible
- **Alerts** — Actionable, tiered severity
- **Runbooks** — What to do when alert fires

**Q19: How would you handle database migrations in a zero-downtime deployment?**
A:
1. **Backward compatible** — New code works with old schema
2. **Expand** — Add new columns/tables (nullable or with defaults)
3. **Migrate** — Backfill data if needed
4. **Deploy** — New code uses new schema
5. **Contract** — Remove old columns/tables (later)
6. **Never** — Rename columns, change types in place

**Q20: How would you implement auto-scaling for unpredictable traffic?**
A:
- **Target tracking** — Maintain CPU at 50%
- **Step scaling** — More aggressive at higher thresholds
- **Predictive scaling** — ML-based for patterns
- **Warm pools** — Pre-initialized instances
- **Scale out fast, scale in slow** — Avoid thrashing
- **Load testing** — Verify scaling works

---

## Security

**Q21: How do you secure an AWS account?**
A:
- **Root account** — MFA, no access keys, rarely used
- **IAM** — Least privilege, MFA for users
- **SCPs** — Guardrails in Organizations
- **CloudTrail** — Audit all API calls
- **GuardDuty** — Threat detection
- **Config** — Compliance rules
- **Security Hub** — Centralized view

**Q22: A developer accidentally committed secrets to Git. What do you do?**
A:
1. **Rotate immediately** — Assume compromised
2. **Remove from history** — `git filter-branch` or BFG
3. **Force push** — Update remote
4. **Audit** — Check if secrets were used
5. **Prevent** — Pre-commit hooks, secret scanning

**Q23: How do you implement least privilege for a Lambda function?**
A:
- **Specific resources** — Not `*`, use ARNs
- **Specific actions** — Only what's needed
- **Conditions** — Restrict by IP, time, etc.
- **Review regularly** — Remove unused permissions
- **Use IAM Access Analyzer** — Find overly permissive policies

**Q24: How do you secure container images?**
A:
- **Base images** — Use official, minimal images
- **Scan** — Trivy, ECR scanning, Snyk
- **No secrets** — Never bake secrets into images
- **Non-root** — Run as non-root user
- **Read-only** — Read-only filesystem where possible
- **Sign** — Image signing for verification

**Q25: How do you handle a security incident?**
A:
1. **Contain** — Isolate affected systems
2. **Assess** — Scope of breach
3. **Preserve** — Evidence for investigation
4. **Eradicate** — Remove threat
5. **Recover** — Restore systems
6. **Learn** — Post-incident review
7. **Report** — Notify stakeholders, regulators if required

---

## Cost Optimization

**Q26: How do you reduce AWS costs?**
A:
- **Right-sizing** — Match instance size to workload
- **Reserved/Savings Plans** — Commit for discounts
- **Spot instances** — For fault-tolerant workloads
- **Auto Scaling** — Scale down when not needed
- **Storage tiering** — S3 lifecycle policies
- **Delete unused** — Old snapshots, unattached EBS
- **Cost Explorer** — Analyze spending

**Q27: Your AWS bill spiked unexpectedly. How do you investigate?**
A:
1. **Cost Explorer** — Filter by service, region, tag
2. **Billing dashboard** — Top services
3. **CloudWatch** — Usage metrics
4. **Recent changes** — New resources deployed?
5. **Data transfer** — Often overlooked cost
6. **Set up budgets** — Alerts for future

**Q28: How do you optimize Kubernetes costs?**
A:
- **Right-size pods** — Requests match actual usage
- **Cluster Autoscaler** — Remove unused nodes
- **Spot instances** — For non-critical workloads
- **Resource quotas** — Prevent over-provisioning
- **Vertical Pod Autoscaler** — Recommend right sizes
- **Karpenter** — Efficient node provisioning

---

## DevOps Culture & Process

**Q29: How do you handle on-call?**
A:
- **Rotation** — Fair distribution
- **Runbooks** — Documented procedures
- **Escalation** — Clear path when stuck
- **Blameless** — Focus on systems, not people
- **Reduce toil** — Automate repetitive tasks
- **Review** — Weekly review of incidents

**Q30: How do you approach a post-mortem?**
A:
1. **Timeline** — What happened when
2. **Impact** — Users affected, duration
3. **Root cause** — 5 whys analysis
4. **What went well** — Detection, response
5. **What went wrong** — Gaps in process
6. **Action items** — Specific, assigned, deadlined
7. **Share** — Organization-wide learning

**Q31: How do you introduce DevOps practices to a traditional team?**
A:
- **Start small** — One project, one team
- **Quick wins** — Automate painful manual tasks
- **Measure** — Show improvement with metrics
- **Training** — Invest in skills
- **Culture** — Blameless, collaborative
- **Iterate** — Continuous improvement

**Q32: How do you balance speed and stability?**
A:
- **Automate testing** — Catch bugs early
- **Feature flags** — Decouple deploy from release
- **Canary deployments** — Gradual rollout
- **Monitoring** — Detect issues quickly
- **Rollback** — Fast recovery
- **Error budgets** — SLOs guide risk tolerance

**Q33: How do you handle technical debt?**
A:
- **Track it** — Document known issues
- **Prioritize** — Impact vs effort
- **Allocate time** — Regular percentage of sprints
- **Boy scout rule** — Leave code better than found
- **Refactor incrementally** — Not big rewrites
- **Measure** — Track improvement

---

## Troubleshooting Commands Cheat Sheet

**Q34: How do you check system resources?**
A:
```bash
top / htop          # CPU, memory, processes
free -h             # Memory
df -h               # Disk space
iostat -x 1         # Disk I/O
vmstat 1            # System stats
```

**Q35: How do you check network connectivity?**
A:
```bash
ping host           # Basic connectivity
traceroute host     # Path to host
curl -v url         # HTTP request
telnet host port    # TCP connection
ss -tuln            # Listening ports
```

**Q36: How do you check logs?**
A:
```bash
tail -f /var/log/syslog     # Follow log
journalctl -u service       # Systemd service logs
kubectl logs pod            # Kubernetes pod logs
docker logs container       # Docker container logs
```

**Q37: How do you check processes?**
A:
```bash
ps aux              # All processes
pgrep -a name       # Find by name
lsof -p PID         # Files opened by process
strace -p PID       # System calls
```

**Q38: How do you check Kubernetes resources?**
A:
```bash
kubectl get pods -A                 # All pods
kubectl describe pod name           # Pod details
kubectl logs pod -f                 # Follow logs
kubectl top pods                    # Resource usage
kubectl get events --sort-by='.lastTimestamp'
```

---

## Quick Decision Trees

**Q39: Service is slow — where do you start?**
A:
```
Is it slow for everyone? 
  → Yes: Check server metrics (CPU, memory, I/O)
  → No: Check user's network, CDN, regional issues

Server metrics normal?
  → No: Scale up/out, optimize resource usage
  → Yes: Check application (APM, traces, slow queries)

Application normal?
  → No: Optimize code, queries, caching
  → Yes: Check dependencies (database, external APIs)
```

**Q40: Deployment failed — what do you check?**
A:
```
Build failed?
  → Check build logs, dependencies, tests

Deploy failed?
  → Check deployment logs, permissions, resources

App won't start?
  → Check app logs, config, secrets, health checks

App starts but errors?
  → Check app logs, compare with previous version, rollback
```

---

*These scenarios test real-world problem-solving. Practice thinking through them out loud! 🎯*
