# EKS Troubleshooting — CrashLoopBackOff, Evictions, Cordon & Drain 🔧

Everything you need to debug pod failures on EKS.

---

## Pod Status Reference

```
Status              Meaning                           Panic Level
──────────          ─────────────────────────         ───────────
Running             All good                          😊
Pending             Waiting to be scheduled           ⏳
ContainerCreating   Pulling image / setting up        ⏳
CrashLoopBackOff    App keeps crashing and restarting 🔴
Error               Container exited with error       🔴
OOMKilled           Ran out of memory                 🔴
ImagePullBackOff    Can't pull container image        🟡
Evicted             Node kicked the pod out           🟡
Terminating         Being deleted (stuck = problem)   🟡
Unknown             Lost contact with node            🔴
```

---

## CrashLoopBackOff — Deep Dive

### What It Means
Container starts → crashes → Kubernetes restarts it → crashes again → waits longer → restarts... (exponential backoff: 10s, 20s, 40s, 80s... up to 5 minutes)

### Debug Flowchart
```
kubectl describe pod <name>
  │
  ├── Check "Last State" section
  │   │
  │   ├── Reason: OOMKilled (Exit Code 137)
  │   │   → Container exceeded memory limit
  │   │   → Fix: increase resources.limits.memory
  │   │   → Or: fix memory leak in application
  │   │
  │   ├── Reason: Error (Exit Code 1)
  │   │   → Application error at startup
  │   │   → Check logs: kubectl logs <pod> --previous
  │   │   → Common: missing env var, bad config, DB not reachable
  │   │
  │   ├── Reason: Error (Exit Code 126)
  │   │   → Command not executable (permission denied)
  │   │   → Check: Dockerfile CMD/ENTRYPOINT, file permissions
  │   │
  │   ├── Reason: Error (Exit Code 127)
  │   │   → Command not found
  │   │   → Check: binary exists in image, PATH correct
  │   │
  │   └── Reason: Error (Exit Code 143)
  │       → SIGTERM received (killed by K8s)
  │       → Liveness probe failing? Check probe config
  │       → preStop hook timing out?
  │
  └── Check "Events" section
      │
      ├── "Back-off restarting failed container"
      │   → Normal for CrashLoop — shows the backoff
      │
      ├── "Liveness probe failed"
      │   → App starts but health check fails
      │   → Increase initialDelaySeconds
      │   → Check probe path/port is correct
      │
      └── "Failed to pull image"
          → Not CrashLoop but can look similar
          → Check image name, tag, registry auth
```

### Exit Code Reference
```
Exit Code    Meaning                    Common Cause
─────────    ────────────────────       ──────────────────────────────
0            Success                    Normal exit (shouldn't restart)
1            General error              Application exception/panic
126          Permission denied          Can't execute CMD
127          Command not found          Wrong binary path, missing dep
130          SIGINT (Ctrl+C)            Manual termination
137          SIGKILL (OOMKilled)        Memory limit exceeded
                                        OR: killed by liveness probe after failureThreshold
143          SIGTERM                    Graceful shutdown requested
255          Exit status out of range   Undefined error
```

### Debug Commands
```bash
# 1. See why it's crashing
kubectl logs <pod> --previous              # Logs from LAST crash
kubectl logs <pod> --previous --tail=50    # Last 50 lines of crash

# 2. Check events
kubectl describe pod <pod> | grep -A 20 "Events:"
kubectl get events --field-selector involvedObject.name=<pod> --sort-by='.lastTimestamp'

# 3. Check resource limits vs actual usage
kubectl describe pod <pod> | grep -A 5 "Limits:"
kubectl top pod <pod>                      # Current usage (if it stays up long enough)

# 4. Check if it's OOMKilled
kubectl describe pod <pod> | grep -i "oom\|killed\|reason"

# 5. Exec into container (if it stays up briefly)
kubectl exec -it <pod> -- /bin/sh          # Quick — it might crash again

# 6. Run a debug container with same image
kubectl debug <pod> -it --image=busybox --target=<container>
# Or run same image without the crashing CMD:
kubectl run debug --image=<same-image> --command -- sleep 3600
kubectl exec -it debug -- /bin/sh

# 7. Check if config/secrets are mounted correctly
kubectl exec <pod> -- env                  # Check env vars
kubectl exec <pod> -- cat /path/to/config  # Check mounted files
```

### Common Fixes
```yaml
# Fix 1: OOMKilled — increase memory
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"     # Was 128Mi, container needs more

# Fix 2: Liveness probe too aggressive
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30    # Give app time to start (was 5)
  periodSeconds: 10
  failureThreshold: 5        # Allow 5 failures before killing (was 3)
  timeoutSeconds: 5          # Allow 5s for response (was 1)

# Fix 3: App needs env var from Secret
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: my-secret      # Does this Secret exist?
        key: password         # Does this key exist in the Secret?

# Fix 4: Readiness probe (don't send traffic until ready)
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

---

## Evictions — When Nodes Kick Pods Out

### What Causes Eviction
```
Node is under pressure → kubelet evicts pods to survive

Types:
  Memory pressure    → Node running out of RAM
  Disk pressure      → Node running out of disk
  PID pressure       → Too many processes on node

Priority of eviction:
  1. BestEffort pods (no requests/limits)     ← Evicted FIRST
  2. Burstable pods (requests < limits)
  3. Guaranteed pods (requests == limits)      ← Evicted LAST
```

### Investigating Evictions
```bash
# Find evicted pods
kubectl get pods --field-selector=status.phase=Failed | grep Evicted
kubectl get pods -A | grep Evicted

# Why was it evicted?
kubectl describe pod <evicted-pod>
# Look for: "The node was low on resource: memory"
#           "The node was low on resource: ephemeral-storage"

# Check node conditions
kubectl describe node <node> | grep -A 10 "Conditions:"
# MemoryPressure: True = node is struggling

# Check node resources
kubectl top node <node>
kubectl describe node <node> | grep -A 5 "Allocated resources:"

# Clean up evicted pods (they linger as Failed)
kubectl delete pods --field-selector=status.phase=Failed -A
```

### Preventing Evictions
```yaml
# 1. Set resource requests = limits (Guaranteed QoS)
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"     # Same as request = Guaranteed
    cpu: "500m"

# 2. Use PriorityClass for critical pods
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: critical
value: 1000000
globalDefault: false
description: "Critical production services"

# In pod spec:
priorityClassName: critical

# 3. PodDisruptionBudget (for voluntary disruptions)
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2        # Always keep at least 2 running
  selector:
    matchLabels:
      app: my-app
```

---

## Cordon & Drain — Node Maintenance

### Cordon (Soft — No New Pods)
```bash
# Mark node as unschedulable (existing pods stay)
kubectl cordon <node>

# Verify
kubectl get nodes
# NAME         STATUS                     ROLES
# node-1       Ready,SchedulingDisabled   <none>    ← Cordoned

# Uncordon (allow scheduling again)
kubectl uncordon <node>
```
**When to use:** You want to stop NEW pods from landing on a node, but don't want to disrupt existing pods yet.

### Drain (Hard — Move Everything Off)
```bash
# Evict all pods from node (respects PDBs)
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

# Common flags
kubectl drain <node> \
  --ignore-daemonsets \          # Don't try to evict DaemonSet pods
  --delete-emptydir-data \       # Delete pods using emptyDir volumes
  --force \                      # Evict pods without controllers (standalone pods)
  --grace-period=60 \            # Give pods 60s to shut down
  --timeout=5m                   # Wait max 5 minutes

# If drain is stuck (pod won't evict):
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data --force --grace-period=0
```

**When to use:** Node maintenance (OS upgrade, instance type change, node rotation).

### Drain Sequence
```
kubectl drain node-1
  │
  ├── 1. Cordons node (no new pods)
  ├── 2. Finds all pods on node
  ├── 3. Checks PDBs (respects minAvailable)
  ├── 4. Evicts pods one by one
  │      → Pods get SIGTERM
  │      → Wait graceful termination period
  │      → If not stopped → SIGKILL
  ├── 5. Pods get rescheduled on other nodes
  └── 6. Node is empty (safe for maintenance)
```

### PDB Interaction with Drain
```yaml
# If PDB says minAvailable: 2 and you only have 2 pods:
# Drain will be BLOCKED — can't evict without violating PDB

# Solutions:
# 1. Scale up first: kubectl scale deploy/my-app --replicas=3
# 2. Then drain the node
# 3. Or: set PDB to minAvailable: 1 temporarily
```

---

## Node Troubleshooting

### Node NotReady
```bash
# Check node status
kubectl get nodes
kubectl describe node <node> | grep -A 20 "Conditions:"

# Common reasons for NotReady:
# 1. kubelet not running
ssh node
systemctl status kubelet
journalctl -u kubelet --since "10 min ago"

# 2. Network plugin (CNI) down
kubectl get pods -n kube-system -l k8s-app=aws-node   # VPC CNI on EKS
kubectl logs -n kube-system <aws-node-pod>

# 3. Disk full on node
ssh node
df -h

# 4. Instance health check failed (AWS)
aws ec2 describe-instance-status --instance-ids <id>
```

### Karpenter Node Management (EKS)
```bash
# See Karpenter provisioned nodes
kubectl get nodes -l karpenter.sh/nodepool

# See pending pods (Karpenter should pick these up)
kubectl get pods --field-selector=status.phase=Pending

# Check Karpenter logs
kubectl logs -n karpenter -l app.kubernetes.io/name=karpenter --tail=50

# Karpenter consolidation — why did it remove my node?
kubectl get events | grep "consolidation\|disruption"
# Karpenter removes underutilized nodes to save cost

# Force Karpenter to not touch a node (annotate):
kubectl annotate node <node> karpenter.sh/do-not-disrupt="true"
```

---

## Quick Debugging Cheat Sheet

```bash
# Pod won't start
kubectl describe pod <pod>                 # Events tell you WHY
kubectl get events --sort-by='.lastTimestamp' | tail -20

# Pod keeps crashing
kubectl logs <pod> --previous              # What happened before crash
kubectl describe pod <pod> | grep -i "exit\|reason\|oom"

# Pod is slow
kubectl top pod <pod>                      # CPU/memory usage
kubectl exec <pod> -- curl -w "\nTotal: %{time_total}s\n" -o /dev/null -s http://dependency:8080/health

# Pod can't reach something
kubectl exec <pod> -- nslookup service-name       # DNS works?
kubectl exec <pod> -- nc -zv service-name 8080    # TCP works?
kubectl get networkpolicies -n <namespace>         # Network blocked?

# Everything on a node is failing
kubectl describe node <node> | grep -i "pressure\|condition"
kubectl top node <node>
kubectl get pods --field-selector spec.nodeName=<node>

# Clean up mess
kubectl delete pods --field-selector=status.phase=Failed -A   # Dead pods
kubectl delete pods --field-selector=status.phase=Succeeded -A # Completed jobs
```

---

*Master these and you'll fix any EKS issue in minutes, not hours! 🔧*
