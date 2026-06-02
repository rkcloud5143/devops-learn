# Real-World Troubleshooting Guide 🔥

Actual production incidents and how they were resolved. These are interview gold.

> **Note:** Company/service names replaced with placeholders.

---

## 1. Database Stored Procedure Running 6+ Hours (Normally 5-20 min)

### Symptom
Batch job that normally completes in 5-20 minutes ran for 6.8 hours.

### Investigation
```bash
# Check RDS performance (can't ping RDS — use mysql client or network tools)
# Test RDS latency:
mysqladmin -h rds-endpoint.region.rds.amazonaws.com ping
mysql -h rds-endpoint -e "SELECT 1" --connect-timeout=5

# On the app server, check what's running:
mysql -e "SHOW FULL PROCESSLIST;"
mysql -e "SELECT * FROM information_schema.INNODB_TRX;"
```

### Root Cause
A stored procedure (`SP_container_hourly`) examined **4.9 billion rows**. The `UPDATE` statement had:
- A correlated subquery with `LIKE CONCAT(search_text, '%')` pattern
- This pattern can't use an index efficiently
- Query did a full table scan for every row

### Resolution
- Identified the specific problematic query within the stored procedure
- Recommended index optimization and query rewrite
- Monitored using RDS Performance Insights

### Lesson
```
When a job is slow:
1. Check SHOW PROCESSLIST — find the long-running query
2. Check EXPLAIN plan — is it using indexes?
3. Look for: full table scans, correlated subqueries, LIKE with leading wildcard
4. RDS Performance Insights shows top SQL by wait time
```

---

## 2. EFS Ownership Changing Unexpectedly

### Symptom
`/data/efs` directory ownership kept changing from `tomcat:tomcat` to individual usernames (`rbhupathiraju:tomcat`, `asharma:tomcat`).

### Investigation
```bash
# Check current ownership
ls -ld /data/efs

# Check how it's mounted
mount | grep /data/efs
# Result: NFS4 mount via EFS helper (127.0.0.1:/)

grep efs /etc/fstab
# fs-0f8e33731449b63d1.efs.us-east-1.amazonaws.com:/ /data/efs efs _netdev,tls,nofail 0 0

# Check audit logs for who changed ownership
sudo ausearch -f /data/efs -i | grep -i chown

# Check if any process is writing to EFS
lsof +D /data/efs 2>/dev/null | head -20

# Check cron jobs that might change permissions
crontab -l
cat /etc/crontab | grep -i efs
```

### Root Cause
When users SSH in and write files to the EFS mount, files inherit the user's UID. NFS4 doesn't enforce ownership the same way local filesystems do.

### Resolution
- Set proper permissions at mount point level
- Added cron job to periodically reset ownership: `chown -R tomcat:tomcat /data/efs`
- Investigated if applications needed write access and fixed with proper group permissions

### Lesson
```
EFS/NFS ownership issues:
1. NFS maps UIDs — different servers with same UID = same user
2. Check mount options (root_squash, all_squash)
3. Use audit logs: ausearch -f /path
4. Consider using POSIX ACLs or specific mount options
```

---

## 3. SSM Session Manager Timeout (DBA Team Complaint)

### Symptom
Database administrators reporting their SSM sessions disconnecting after 20 minutes, despite 40-minute idle timeout configured.

### Investigation
```bash
# Check SSM document settings
aws ssm describe-document --name "SSM-SessionManagerRunShell" --query 'Document.Content'

# Check session logs in S3
aws s3 ls s3://ssm-session-logs-bucket/

# Check CloudTrail for session events
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ResumeSession \
  --start-time "2026-05-01"

# Check actual timeout in session preferences
aws ssm get-document --name "SSM-SessionManagerRunShell" --document-version '$LATEST'
```

### Root Cause
Two types of timeout:
- **Idle timeout** (configured: 40 min) — no keyboard input
- **Session duration timeout** — maximum session length regardless of activity

The sessions were hitting the session duration limit, not the idle timeout.

### Resolution
- Documented the difference between idle and session duration timeouts
- Adjusted session preferences in SSM Document
- Enabled S3 session logging for audit trail
- Communicated clear timeout expectations to DBA team

### Lesson
```
SSM Session timeout troubleshooting:
1. Check SSM-SessionManagerRunShell document (idleSessionTimeout vs maxSessionDuration)
2. S3 logs show actual session duration
3. CloudTrail shows TerminateSession events with reason
4. Different from SSH timeout (ServerAliveInterval)
```

---

## 4. TLS 1.3 Connection Failures (Vendor API)

### Symptom
Outbound HTTPS calls to vendor API (`api.vendor.com`) failing from production servers. Same calls work from dev servers and laptops.

### Investigation
```bash
# Test TLS version support
openssl s_client -connect api.vendor.com:443 -tls1_2
# Result: CONNECTED (works)

openssl s_client -connect api.vendor.com:443 -tls1_3
# Result: Error — TLS 1.3 not supported by this OpenSSL

# Check OpenSSL version
openssl version
# OpenSSL 1.0.2k — too old! TLS 1.3 requires OpenSSL 1.1.1+

# Check curl version
curl --version
# curl 7.61 with OpenSSL 1.0.2k — no TLS 1.3

# The vendor enforced TLS 1.3 minimum — our servers can't connect
```

### Root Cause
- Production servers running Amazon Linux 2 with OpenSSL 1.0.2 (no TLS 1.3)
- Vendor API enforced TLS 1.3 minimum
- Application uses `curl` for API calls (JVM also too old for TLS 1.3)

### Resolution
```bash
# Install OpenSSL 1.1 (Amazon Linux 2 extras)
yum install -y openssl11 openssl11-libs openssl11-devel

# Build curl with OpenSSL 1.1 support
# Or use the Amazon Linux 2 openssl11 package with custom curl build

# Verify
/usr/local/bin/curl --version  # Should show OpenSSL/1.1.1+
/usr/local/bin/curl -v --tlsv1.3 https://api.vendor.com/health
```

### Lesson
```
TLS troubleshooting:
1. openssl s_client -connect host:443        → Test connection
2. openssl s_client -connect host:443 -tls1_3 → Test specific version
3. openssl version                            → Check installed version
4. curl -v https://host                       → Shows TLS negotiation
5. Amazon Linux 2 = OpenSSL 1.0.2 (no TLS 1.3!)
6. Amazon Linux 2023 = OpenSSL 3.x (TLS 1.3 ✅)
7. This is a strong reason to migrate AL2 → AL2023
```

---

## 5. EC2 Instance SSM Agent Offline After Recreation

### Symptom
Newly created EC2 instance (migrated from AL2 to AL2023) shows "Offline" in SSM. Can't connect via Session Manager.

### Investigation
```bash
# From AWS Console: SSM > Fleet Manager > Instance shows "Offline"
# Ping status: Offline / Last ping: never

# Check if SSM agent is running (need console access or user-data logs)
# Cloud-init logs:
cat /var/log/cloud-init-output.log | grep ssm

# On the instance (if you can SSH):
sudo systemctl status amazon-ssm-agent
sudo journalctl -u amazon-ssm-agent

# Check instance role
aws ec2 describe-instances --instance-ids i-xxx \
  --query 'Reservations[].Instances[].IamInstanceProfile'
```

### Root Cause Checklist
```
SSM Agent Offline — check in order:
1. ✅ IAM Instance Profile attached? (needs AmazonSSMManagedInstanceCore policy)
2. ✅ SSM agent installed and running? (pre-installed on AL2023)
3. ✅ Instance in private subnet? → Needs VPC endpoint or NAT Gateway
   VPC endpoints needed: ssm, ssmmessages, ec2messages
4. ✅ Security group allows outbound HTTPS (443)?
5. ✅ Route table has path to NAT or VPC endpoint?
6. ✅ Instance metadata service (IMDS) accessible? (IMDSv2 token issues?)
```

### Resolution
- Missing VPC endpoint for SSM in the new subnet
- Added `com.amazonaws.region.ssm`, `com.amazonaws.region.ssmmessages`, `com.amazonaws.region.ec2messages` VPC endpoints
- SSM agent came online within 5 minutes

### Lesson
```
SSM requires HTTPS outbound to:
  - ssm.region.amazonaws.com
  - ssmmessages.region.amazonaws.com
  - ec2messages.region.amazonaws.com

In private subnets: VPC endpoints OR NAT Gateway
Most common cause of "Offline": missing network path to SSM endpoints
```

---

## 6. Jenkins Build Failures After Security Scanning Introduction

### Symptom
After introducing TruffleHog (secret scanning) to Jenkins pipeline, builds failing across multiple repos.

### Investigation
```groovy
// Jenkins console output showed:
// TruffleHog found secrets in git history
// Results: 1000+ issues across repos
```

### Root Cause
Historical secrets committed to Git (before .gitignore was fixed). TruffleHog scans ALL git history by default.

### Resolution
```
Phase 1: Don't break builds immediately
  - Set TruffleHog to WARNING only (don't fail build)
  - Generate report of all findings
  - Categorize: active secrets vs rotated vs false positives

Phase 2: Remediate
  - Rotate any active secrets found
  - Add to .trufflehog-ignore for known false positives
  - Fix repos with git-filter-repo where possible

Phase 3: Enforce
  - Enable build failure for NEW secrets only (--since-commit=HEAD~1)
  - Existing findings tracked in backlog
```

### Lesson
```
Introducing security scanning to existing repos:
1. NEVER go straight to "fail build" — you'll block everyone
2. Run in report-only mode first
3. Categorize findings (critical vs noise)
4. Fix critical first, create backlog for rest
5. Then enable enforcement for NEW commits only
6. Gradually increase scanning scope
```

---

## 7. Patch Manager Maintenance Window Failed

### Symptom
Slack alert: `"⚠️ Patch Manager Alert - Window: mw-xxx | Status: FAILED"`

### Investigation
```bash
# Check maintenance window execution history
aws ssm describe-maintenance-window-executions \
  --window-id mw-xxx \
  --query 'WindowExecutions[0]'

# Check specific task execution
aws ssm describe-maintenance-window-execution-task-invocations \
  --window-execution-id <execution-id> \
  --task-id <task-id>

# Check instance patch status
aws ssm describe-instance-patch-states \
  --instance-ids i-xxx

# Common: check if instance was reachable
aws ssm describe-instance-information \
  --instance-information-filter-list key=InstanceIds,valueSet=i-xxx
```

### Common Causes
```
1. Instance not reachable by SSM (agent offline, network issue)
2. Patch requires reboot but reboot option not set
3. Disk full — can't download patches
4. Patch conflicts (dependency issues)
5. Maintenance window too short for all patches
6. Instance was stopped/terminated during window
```

### Resolution
- Check which specific instances failed and why
- Pre-patch: ensure SSM agent online, disk space available
- Post-patch: Lambda function reports compliance status to Slack
- AMI backup taken before patching (rollback plan)

---

## 8. Java Application Processing Transactions Slowly

### Symptom
Batch processing application normally processes transactions quickly, but started taking hours.

### Investigation
```bash
# Check how many transactions processed
grep -c "Processing transaction" /data/app/logs/app.log
# 11,245 (normal count, but taking too long)

# Check for errors
grep -i -E "error|exception|failed|timeout" /data/app/logs/app.log | tail -20

# Check EC2 I/O (logs writing + Datadog reading simultaneously)
iostat -x 1 5
# %util near 100% = disk bottleneck

# Check if Datadog agent is consuming I/O
pidstat -d 1 5  # Per-process I/O

# Check EBS volume metrics in CloudWatch
# BurstBalance, VolumeQueueLength, VolumeReadOps
```

### Root Cause
- Application writing large log files
- Datadog agent reading same log files simultaneously
- EBS volume hitting IOPS limit (gp2 burst credits exhausted)

### Resolution
- Upgraded EBS from gp2 to gp3 (consistent IOPS, no burst dependency)
- Configured Datadog log collection with file rotation awareness
- Added log rotation to prevent single large files

### Lesson
```
EC2 I/O troubleshooting:
1. iostat -x 1 → Check %util (100% = saturated)
2. pidstat -d 1 → Which process is causing I/O
3. CloudWatch → EBS BurstBalance (gp2), VolumeQueueLength
4. gp2 burst credits run out under sustained load → switch to gp3
5. Multiple processes reading/writing same files = contention
```

---

## 9. Cloudflare Zero Trust — User Can't Access Resource

### Symptom
User reports: "I can't access the internal database via WARP tunnel"

### Investigation
```bash
# Check if user's device is connected to WARP
# Cloudflare Dashboard → Zero Trust → Devices

# Check gateway logs for the user
# Zero Trust → Logs → Gateway → Filter by user email

# Key info from gateway log:
# Source IP: 66.46.32.246 (external)
# Source Internal IP: 100.96.4.23 (WARP tunnel)
# Destination IP: 172.16.8.49 (internal resource)
# Action: allow/block
# Matched Policy: which rule matched?
```

### Troubleshooting Checklist
```
1. Is user's device registered and WARP connected?
   → Zero Trust > Devices > check status
   
2. Is the device posture passing?
   → Check device serial number in posture list
   → Check OS version, disk encryption requirements

3. Which gateway policy matched?
   → Logs show "Matched Policies" field
   → Is it the right policy? Or caught by a block rule above it?

4. Is the user in the correct AD group?
   → Policies reference AD groups by GUID
   → Check user's group membership in Azure AD

5. Is the tunnel route configured?
   → Zero Trust > Networks > Tunnels > Routes
   → Is 172.16.8.0/24 routed through the correct tunnel?

6. Is the tunnel healthy?
   → Check tunnel connector status (cloudflared running?)
```

### Lesson
```
Cloudflare ZT debugging order:
1. Device registered + WARP connected?
2. Gateway logs — what action was taken?
3. Which policy matched? (rules are ordered — first match wins)
4. User in correct identity group?
5. Tunnel route exists for destination?
6. Tunnel connector healthy?
```

---

## 10. GitHub Actions Self-Hosted Runner Not Picking Up Jobs

### Symptom
GitHub Actions workflows queued but not being picked up by self-hosted runners on EKS.

### Investigation
```bash
# Check runner pods
kubectl get pods -n arc-runners
kubectl describe pod <runner-pod>

# Check ARC controller
kubectl logs -n arc-systems deployment/arc-controller-manager

# Check runner scale set
kubectl get autoscalingrunnerset -A

# Check if runner is registered with GitHub
# GitHub > Repo > Settings > Actions > Runners
```

### Common Causes
```
1. Runner labels don't match workflow runs-on
   Workflow: runs-on: arc-runners-java
   Runner: labeled as arc-runners (mismatch!)

2. Runner pod can't pull image (ECR login expired)
   kubectl describe pod → ImagePullBackOff

3. Karpenter can't provision node (instance type unavailable)
   kubectl get events | grep Karpenter

4. Runner scale set at max replicas
   Check maxRunners in HelmRelease

5. Org/repo permissions not configured
   Runner group not assigned to repository
```

---

## Quick Reference: Troubleshooting Commands by Scenario

```bash
# ─── "Is it the network?" ───
nc -zv host port                       # TCP connectivity
curl -w "\nTime: %{time_total}s\nHTTP: %{http_code}\n" -o /dev/null -s URL
traceroute host                        # Where does it break?
mtr host                               # Continuous traceroute

# ─── "Is it the disk?" ───
df -h                                  # Space
iostat -x 1 3                          # I/O saturation
lsof +D /path | wc -l                  # Open files in path

# ─── "Is it the process?" ───
top -bn1 | head -20                    # Top CPU/memory consumers
ps aux --sort=-%mem | head -10         # Top memory
strace -p PID -c                       # System call summary

# ─── "Is it the database?" ───
mysql -e "SHOW FULL PROCESSLIST;"      # Active queries
mysql -e "SHOW ENGINE INNODB STATUS\G" # InnoDB status

# ─── "Is it Kubernetes?" ───
kubectl describe pod <name>            # Events + state
kubectl logs <name> --previous         # Crash logs
kubectl top pods --sort-by=cpu         # Resource hogs
kubectl get events --sort-by='.lastTimestamp'

# ─── "Is it AWS?" ───
aws health describe-events             # AWS service issues
aws ec2 describe-instance-status       # Instance health
aws cloudwatch get-metric-statistics   # Historical metrics
```

---

*These are REAL incidents, not textbook examples. Practice explaining your debugging process out loud for interviews! 🔥*
