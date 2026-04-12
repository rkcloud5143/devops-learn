# Kubernetes — Objects (All Types) — Deep Dive

---

## HOW OBJECTS RELATE

```
┌─── Kubernetes Object Hierarchy ──────────────────────────────┐
│                                                               │
│  Namespace (logical isolation)                               │
│  └── Deployment (manages ReplicaSets)                        │
│      └── ReplicaSet (ensures N pods running)                 │
│          └── Pod (runs containers)                           │
│              └── Container (your app)                        │
│                                                               │
│  Service (exposes pods to network)                           │
│  Ingress (HTTP routing from outside)                         │
│  ConfigMap (config data)                                     │
│  Secret (sensitive data)                                     │
│  PersistentVolume (storage)                                  │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 1. POD (Smallest Unit)

```yaml
# A pod = one or more containers sharing network and storage
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  containers:
    - name: my-app
      image: nginx:latest
      ports:
        - containerPort: 80
      resources:
        requests:              # Minimum guaranteed
          memory: "128Mi"
          cpu: "250m"          # 250 millicores = 0.25 CPU
        limits:                # Maximum allowed
          memory: "256Mi"
          cpu: "500m"
```

```
Key points:
- Pods are ephemeral (they die and get replaced)
- Never create pods directly — use Deployments
- Pods get their own IP address (changes on restart)
- Containers in same pod share localhost
- One container per pod is the common pattern
```

---

## 2. REPLICASET (Ensures N Pods Running)

```
┌─── ReplicaSet: replicas=3 ───────────────────────────────────┐
│                                                               │
│  ┌─────┐  ┌─────┐  ┌─────┐                                  │
│  │Pod 1│  │Pod 2│  │Pod 3│  ← always maintains 3 pods       │
│  └─────┘  └─────┘  └─────┘                                  │
│                                                               │
│  Pod 3 dies? → ReplicaSet creates Pod 4 automatically        │
│                                                               │
│  You almost NEVER create ReplicaSets directly.               │
│  Deployments create and manage ReplicaSets for you.          │
└───────────────────────────────────────────────────────────────┘
```

---

## 3. DEPLOYMENT (Most Common Object)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate        # Zero-downtime deploys
    rollingUpdate:
      maxSurge: 1              # 1 extra pod during update
      maxUnavailable: 0        # Never have fewer than 3
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:v2.1.0
          ports:
            - containerPort: 3000
          env:
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: DB_HOST
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: DB_PASSWORD
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "1000m"
          readinessProbe:        # Is pod ready for traffic?
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:         # Is pod alive?
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 20
```

```
Deployment strategies:
┌──────────────────┬───────────────────────────────────────────┐
│ RollingUpdate    │ Replace pods one by one (zero downtime)   │
│ (default)        │ Old and new versions run simultaneously   │
├──────────────────┼───────────────────────────────────────────┤
│ Recreate         │ Kill all old pods, then create new ones   │
│                  │ Brief downtime, but no version mixing     │
└──────────────────┴───────────────────────────────────────────┘

Health checks:
- readinessProbe: "is this pod ready to receive traffic?"
  Failed → removed from Service (no traffic sent)
- livenessProbe: "is this pod still alive?"
  Failed → pod is killed and restarted
- startupProbe: "has this pod finished starting?"
  For slow-starting apps
```

---

## 4. SERVICE (Expose Pods to Network)

```
Pods get random IPs that change. Services give a stable endpoint.

┌─── Service: my-app-service ──────────────────────────────────┐
│  Stable IP: 10.96.50.100                                     │
│  DNS: my-app-service.production.svc.cluster.local            │
│                                                               │
│  Routes to pods with label: app=my-app                       │
│  ┌─────┐  ┌─────┐  ┌─────┐                                  │
│  │Pod 1│  │Pod 2│  │Pod 3│  ← load balanced                 │
│  └─────┘  └─────┘  └─────┘                                  │
└───────────────────────────────────────────────────────────────┘
```

### Service Types
```yaml
# ClusterIP (default) — internal only
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: ClusterIP              # Only accessible inside cluster
  selector:
    app: my-app
  ports:
    - port: 80                 # Service port
      targetPort: 3000         # Container port
```

```
┌──────────────────┬───────────────────────────────────────────┐
│ ClusterIP        │ Internal only (default)                   │
│                  │ Other pods call: my-app-service:80        │
│                  │ Use for: internal microservices            │
├──────────────────┼───────────────────────────────────────────┤
│ NodePort         │ Exposes on each node's IP at a static port│
│                  │ Range: 30000-32767                        │
│                  │ Access: <NodeIP>:30080                    │
│                  │ Use for: dev/testing                      │
├──────────────────┼───────────────────────────────────────────┤
│ LoadBalancer     │ Creates cloud provider load balancer      │
│                  │ On AWS: creates an ELB/NLB automatically  │
│                  │ Use for: exposing to internet             │
├──────────────────┼───────────────────────────────────────────┤
│ ExternalName     │ Maps to an external DNS name              │
│                  │ Use for: pointing to external services    │
└──────────────────┴───────────────────────────────────────────┘
```

---

## 5. INGRESS (HTTP Routing)

```
Ingress = smart HTTP router that sits in front of Services

                    Internet
                       │
              ┌────────▼────────┐
              │ Ingress Controller│  (nginx, ALB, traefik)
              │                  │
              │ Rules:           │
              │ /api/*  → api-svc│
              │ /web/*  → web-svc│
              │ /*      → default│
              └───┬──────────┬───┘
                  │          │
         ┌────────▼──┐  ┌───▼────────┐
         │ api-svc   │  │ web-svc    │
         │ (pods)    │  │ (pods)     │
         └───────────┘  └────────────┘
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
    - hosts:
        - myapp.com
      secretName: myapp-tls
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```

```
Ingress vs LoadBalancer Service:
- LoadBalancer: one ELB per service ($$$)
- Ingress: one ELB for ALL services, routes by path/host ($)

On AWS EKS:
- AWS Load Balancer Controller creates ALB from Ingress
- Annotation: kubernetes.io/ingress.class: alb
```

---

## 6. CONFIGMAP & SECRET

```yaml
# ConfigMap — non-sensitive configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: "my-rds.amazonaws.com"
  DB_PORT: "5432"
  LOG_LEVEL: "info"
  APP_ENV: "production"

---
# Secret — sensitive data (base64 encoded)
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=      # base64 of "password123"
  API_KEY: bXlzZWNyZXRrZXk=          # base64 of "mysecretkey"
```

```
Using in pods:
- Environment variables (valueFrom: configMapKeyRef / secretKeyRef)
- Volume mounts (mount as files inside container)

Secrets are NOT encrypted by default — just base64 encoded!
For real security: enable encryption at rest in etcd,
or use AWS Secrets Manager with External Secrets Operator
```

---

## 7. NAMESPACE (Logical Isolation)

```
┌─── Cluster ──────────────────────────────────────────────────┐
│                                                               │
│  ┌─── namespace: production ──────────────────────────────┐  │
│  │  Deployments, Services, ConfigMaps for prod            │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌─── namespace: staging ─────────────────────────────────┐  │
│  │  Same app names, different config                      │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌─── namespace: kube-system ─────────────────────────────┐  │
│  │  K8s internal components (DNS, metrics, etc.)          │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                               │
└───────────────────────────────────────────────────────────────┘

kubectl get pods -n production       # Pods in production namespace
kubectl get pods --all-namespaces    # Pods in ALL namespaces
kubectl create namespace staging
```

---

## 8. PERSISTENT VOLUMES (Storage)

```
Containers are ephemeral — data is lost when pod dies.
PersistentVolumes solve this.

┌─── PersistentVolume (PV) ────────────────────────────────────┐
│  Actual storage (EBS volume, EFS, etc.)                      │
│  Created by admin or dynamically by StorageClass             │
└──────────────────────────────────────────────────────────────┘
         ▲ bound
┌─── PersistentVolumeClaim (PVC) ──────────────────────────────┐
│  Request for storage by a pod ("I need 10Gi")               │
│  Created by developer                                        │
└──────────────────────────────────────────────────────────────┘
         ▲ mounted
┌─── Pod ──────────────────────────────────────────────────────┐
│  Uses PVC as a volume mount                                  │
└──────────────────────────────────────────────────────────────┘
```

```yaml
# PVC — request storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce            # One node can mount read-write
  storageClassName: gp3
  resources:
    requests:
      storage: 20Gi

---
# Pod using PVC
apiVersion: v1
kind: Pod
metadata:
  name: my-db
spec:
  containers:
    - name: postgres
      image: postgres:15
      volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: db-storage
  volumes:
    - name: db-storage
      persistentVolumeClaim:
        claimName: data-pvc
```

```
Access Modes:
- ReadWriteOnce (RWO): one node read-write (EBS)
- ReadOnlyMany (ROX): many nodes read-only
- ReadWriteMany (RWX): many nodes read-write (EFS)

On AWS EKS:
- EBS CSI Driver → provisions EBS volumes (RWO)
- EFS CSI Driver → provisions EFS volumes (RWX)
```

---

## 9. DAEMONSET (One Pod Per Node)

```
┌─── DaemonSet ────────────────────────────────────────────────┐
│                                                               │
│  Ensures ONE pod runs on EVERY node                          │
│                                                               │
│  Node 1: [monitoring-agent]                                  │
│  Node 2: [monitoring-agent]                                  │
│  Node 3: [monitoring-agent]                                  │
│  New Node 4 added → [monitoring-agent] auto-deployed         │
│                                                               │
│  Use for:                                                    │
│  - Log collectors (Fluentd, Filebeat)                        │
│  - Monitoring agents (Datadog, Prometheus node-exporter)     │
│  - Storage daemons                                           │
│  - kube-proxy (runs as DaemonSet by default)                 │
└───────────────────────────────────────────────────────────────┘
```

---

## 10. STATEFULSET (For Databases)

```
┌─── StatefulSet ──────────────────────────────────────────────┐
│                                                               │
│  Like Deployment but for STATEFUL apps (databases, queues)   │
│                                                               │
│  Guarantees:                                                 │
│  - Stable network identity: pod-0, pod-1, pod-2             │
│  - Ordered startup: pod-0 first, then pod-1, then pod-2     │
│  - Ordered shutdown: reverse order                           │
│  - Stable storage: each pod gets its own PVC                 │
│                                                               │
│  ┌────────┐  ┌────────┐  ┌────────┐                         │
│  │ db-0   │  │ db-1   │  │ db-2   │  (stable names)         │
│  │ PVC-0  │  │ PVC-1  │  │ PVC-2  │  (own storage each)     │
│  └────────┘  └────────┘  └────────┘                         │
│                                                               │
│  Use for: PostgreSQL, MySQL, Redis, Kafka, Elasticsearch     │
│  In practice: use managed services (RDS, ElastiCache)        │
│  instead of running databases in K8s                         │
└───────────────────────────────────────────────────────────────┘
```

---

## 11. HPA (Horizontal Pod Autoscaler)

```
┌─── HPA ──────────────────────────────────────────────────────┐
│                                                               │
│  Auto-scales pods based on CPU/memory/custom metrics         │
│                                                               │
│  CPU < 30%:  scale down to min (2 pods)                      │
│  CPU = 50%:  stay at current (3 pods)                        │
│  CPU > 70%:  scale up to max (10 pods)                       │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 12. JOBS & CRONJOBS

```yaml
# Job — run once to completion
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  template:
    spec:
      containers:
        - name: migrate
          image: my-app:v2
          command: ["python", "manage.py", "migrate"]
      restartPolicy: Never
  backoffLimit: 3              # Retry 3 times on failure

---
# CronJob — scheduled job
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  schedule: "0 2 * * *"       # Daily at 2 AM (same as Linux cron)
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup
              image: backup-tool:latest
              command: ["./backup.sh"]
          restartPolicy: OnFailure
```

---

## 13. RBAC (Role-Based Access Control)

```
WHO can do WHAT in the cluster

┌─── RBAC ─────────────────────────────────────────────────────┐
│                                                               │
│  Role / ClusterRole:  defines WHAT actions are allowed       │
│  RoleBinding / ClusterRoleBinding: WHO gets the role         │
│                                                               │
│  Role = namespace-scoped                                     │
│  ClusterRole = cluster-wide                                  │
│                                                               │
│  Example: "developers can view pods in staging namespace"    │
└───────────────────────────────────────────────────────────────┘
```

```yaml
# Role — what's allowed
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: staging
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]

---
# RoleBinding — who gets it
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: staging
  name: read-pods
subjects:
  - kind: User
    name: developer@company.com
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## OBJECT CHEAT SHEET

```
┌──────────────────┬───────────────────────────────────────────┐
│ Object           │ What it does                              │
├──────────────────┼───────────────────────────────────────────┤
│ Pod              │ Runs container(s) — smallest unit         │
│ ReplicaSet       │ Ensures N pods running (managed by Deploy)│
│ Deployment       │ Manages pods + rolling updates            │
│ StatefulSet      │ For databases (stable identity + storage) │
│ DaemonSet        │ One pod per node (monitoring, logging)    │
│ Job              │ Run once to completion                    │
│ CronJob          │ Scheduled job (like Linux cron)           │
│ Service          │ Stable network endpoint for pods          │
│ Ingress          │ HTTP routing (path/host based)            │
│ ConfigMap        │ Non-sensitive config data                 │
│ Secret           │ Sensitive data (passwords, keys)          │
│ Namespace        │ Logical isolation (prod/staging/dev)      │
│ PV / PVC         │ Persistent storage                        │
│ HPA              │ Auto-scale pods by CPU/memory             │
│ RBAC             │ Access control (who can do what)          │
└──────────────────┴───────────────────────────────────────────┘
```
