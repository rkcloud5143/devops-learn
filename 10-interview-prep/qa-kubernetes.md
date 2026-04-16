# Kubernetes Q&A (200+ Questions)

Basic, Advanced, and Scenario-based questions for Kubernetes and EKS.

---

## Basic Concepts

**Q1: What is Kubernetes?**
A: Container orchestration platform. Automates deployment, scaling, and management of containerized applications.

**Q2: What is K8s?**
A: Abbreviation for Kubernetes. K + 8 letters + s.

**Q3: What problem does Kubernetes solve?**
A: Managing containers at scale. Handles scheduling, scaling, self-healing, service discovery, load balancing.

**Q4: What is a container?**
A: Lightweight, standalone package containing application and dependencies. Runs consistently anywhere.

**Q5: What is Docker?**
A: Container runtime and tooling. Builds and runs containers. Kubernetes can use Docker (or containerd, CRI-O).

**Q6: What is a Kubernetes cluster?**
A: Set of nodes running containerized applications. Consists of control plane and worker nodes.

**Q7: What is the control plane?**
A: Brain of the cluster. API server, scheduler, controller manager, etcd.

**Q8: What is a worker node?**
A: Machine that runs pods. Has kubelet, container runtime, kube-proxy.

**Q9: What is the API server?**
A: Frontend for Kubernetes. All communication goes through it. RESTful API.

**Q10: What is etcd?**
A: Distributed key-value store. Stores all cluster state and configuration.

**Q11: What is the scheduler?**
A: Assigns pods to nodes. Considers resources, constraints, affinity.

**Q12: What is the controller manager?**
A: Runs controllers that maintain desired state. Node controller, replication controller, etc.

**Q13: What is kubelet?**
A: Agent on each node. Ensures containers are running in pods.

**Q14: What is kube-proxy?**
A: Network proxy on each node. Maintains network rules for pod communication.

**Q15: What is kubectl?**
A: Command-line tool for Kubernetes. Communicates with API server.

---

## Pods

**Q16: What is a Pod?**
A: Smallest deployable unit. One or more containers sharing network and storage.

**Q17: Why not just run containers directly?**
A: Pods provide shared networking (same IP), shared storage, and lifecycle management.

**Q18: Can a pod have multiple containers?**
A: Yes. Sidecar pattern. Containers share localhost and volumes.

**Q19: What is a sidecar container?**
A: Helper container in same pod. Logging, proxying, syncing. Example: Envoy proxy.

**Q20: What is an init container?**
A: Runs before main containers. For setup tasks. Must complete successfully.

**Q21: What is a pod's lifecycle?**
A: Pending → Running → Succeeded/Failed. Can also be Unknown.

**Q22: What is a pod restart policy?**
A: Always (default), OnFailure, Never. Determines when kubelet restarts containers.

**Q23: How do you view pod logs?**
A: `kubectl logs pod-name` or `kubectl logs pod-name -c container-name`

**Q24: How do you exec into a pod?**
A: `kubectl exec -it pod-name -- /bin/sh`

**Q25: What is a static pod?**
A: Pod managed directly by kubelet, not API server. Defined in manifest file on node.

**Q26: What is a pod's IP address?**
A: Unique IP within cluster. Changes when pod restarts. That's why we use Services.

**Q27: What is pod affinity?**
A: Schedule pods together. Example: frontend pods near backend pods.

**Q28: What is pod anti-affinity?**
A: Schedule pods apart. Example: spread replicas across nodes.

**Q29: What is a taint?**
A: Mark on a node that repels pods. Pods need matching toleration to schedule.

**Q30: What is a toleration?**
A: Allows pod to schedule on tainted node.

---

## Workloads

**Q31: What is a ReplicaSet?**
A: Ensures specified number of pod replicas are running. Replaces failed pods.

**Q32: Should you create ReplicaSets directly?**
A: No. Use Deployments, which manage ReplicaSets.

**Q33: What is a Deployment?**
A: Manages ReplicaSets and provides declarative updates. Rolling updates, rollbacks.

**Q34: How do you create a Deployment?**
A: `kubectl create deployment name --image=image` or apply YAML manifest.

**Q35: What is a rolling update?**
A: Gradually replaces old pods with new ones. Zero downtime.

**Q36: What is maxUnavailable in a Deployment?**
A: Maximum pods that can be unavailable during update. Number or percentage.

**Q37: What is maxSurge in a Deployment?**
A: Maximum pods that can be created above desired count during update.

**Q38: How do you rollback a Deployment?**
A: `kubectl rollout undo deployment/name`

**Q39: How do you check rollout status?**
A: `kubectl rollout status deployment/name`

**Q40: How do you view rollout history?**
A: `kubectl rollout history deployment/name`

**Q41: What is a StatefulSet?**
A: For stateful applications. Stable pod names, persistent storage, ordered deployment.

**Q42: When would you use StatefulSet?**
A: Databases, message queues, anything needing stable identity or persistent storage.

**Q43: What is a DaemonSet?**
A: Runs one pod on every node (or selected nodes). For node-level services.

**Q44: What are DaemonSet use cases?**
A: Log collectors (Fluentd), monitoring agents (Prometheus node exporter), network plugins.

**Q45: What is a Job?**
A: Runs pods to completion. For batch processing. Retries on failure.

**Q46: What is a CronJob?**
A: Runs Jobs on a schedule. Cron syntax.

**Q47: What is a ReplicationController?**
A: Legacy. Replaced by ReplicaSet. Ensures pod count.

---

## Services & Networking

**Q48: What is a Service?**
A: Stable network endpoint for pods. Abstracts pod IPs. Load balances.

**Q49: Why do we need Services?**
A: Pod IPs change. Services provide stable DNS name and IP.

**Q50: What is ClusterIP?**
A: Default service type. Internal IP only. Reachable within cluster.

**Q51: What is NodePort?**
A: Exposes service on each node's IP at a static port (30000-32767).

**Q52: What is LoadBalancer?**
A: Provisions external load balancer (cloud provider). Gets external IP.

**Q53: What is ExternalName?**
A: Maps service to external DNS name. Returns CNAME record.

**Q54: What is a headless Service?**
A: ClusterIP: None. Returns pod IPs directly. For StatefulSets.

**Q55: How does service discovery work?**
A: DNS. service-name.namespace.svc.cluster.local resolves to service IP.

**Q56: What is kube-dns / CoreDNS?**
A: Cluster DNS server. Resolves service names to IPs.

**Q57: What is an Endpoint?**
A: List of pod IPs backing a service. Auto-managed by Kubernetes.

**Q58: What is an EndpointSlice?**
A: Scalable replacement for Endpoints. Better for large clusters.

**Q59: What is Ingress?**
A: Manages external HTTP/HTTPS access. Routes by host/path to services.

**Q60: What is an Ingress Controller?**
A: Implements Ingress rules. NGINX, Traefik, AWS ALB Ingress Controller.

**Q61: What's the difference between Ingress and LoadBalancer service?**
A: LoadBalancer = one LB per service. Ingress = one LB, multiple services via routing.

**Q62: What is a NetworkPolicy?**
A: Firewall rules for pods. Controls ingress/egress traffic.

**Q63: What is the default network policy?**
A: Allow all. Pods can communicate freely until you create policies.

**Q64: What is a CNI?**
A: Container Network Interface. Plugin that provides pod networking. Calico, Cilium, Flannel.

**Q65: What is pod-to-pod communication?**
A: All pods can reach all pods without NAT. Flat network.

**Q66: What is the pause container?**
A: Infrastructure container in every pod. Holds network namespace.

---

## Storage

**Q67: What is a Volume?**
A: Directory accessible to containers in a pod. Lifecycle tied to pod.

**Q68: What is emptyDir?**
A: Temporary volume. Created when pod starts, deleted when pod ends.

**Q69: What is hostPath?**
A: Mounts file/directory from host node. Use carefully (security, portability).

**Q70: What is a PersistentVolume (PV)?**
A: Cluster-wide storage resource. Provisioned by admin or dynamically.

**Q71: What is a PersistentVolumeClaim (PVC)?**
A: Request for storage by user. Binds to a PV.

**Q72: What is dynamic provisioning?**
A: Automatically creates PV when PVC is created. Uses StorageClass.

**Q73: What is a StorageClass?**
A: Defines storage "classes" (fast, slow). Specifies provisioner and parameters.

**Q74: What is the reclaim policy?**
A: What happens to PV when PVC is deleted. Retain, Delete, or Recycle (deprecated).

**Q75: What is access mode?**
A: ReadWriteOnce (single node), ReadOnlyMany (multiple nodes), ReadWriteMany (multiple nodes read-write).

**Q76: What is a CSI driver?**
A: Container Storage Interface. Standard for storage plugins. EBS CSI, EFS CSI.

---

## Configuration

**Q77: What is a ConfigMap?**
A: Stores non-sensitive configuration. Key-value pairs or files.

**Q78: How do you use a ConfigMap?**
A: Environment variables, command arguments, or mounted as files.

**Q79: What is a Secret?**
A: Stores sensitive data. Base64 encoded (not encrypted by default).

**Q80: What types of Secrets exist?**
A: Opaque (generic), kubernetes.io/tls, kubernetes.io/dockerconfigjson, etc.

**Q81: How do you create a Secret?**
A: `kubectl create secret generic name --from-literal=key=value` or YAML.

**Q82: Are Secrets encrypted?**
A: Base64 encoded, not encrypted by default. Enable encryption at rest in etcd.

**Q83: What is a ServiceAccount?**
A: Identity for pods. Used for API authentication. Each namespace has default.

**Q84: What is RBAC?**
A: Role-Based Access Control. Controls who can do what in the cluster.

**Q85: What is a Role?**
A: Set of permissions within a namespace.

**Q86: What is a ClusterRole?**
A: Set of permissions cluster-wide.

**Q87: What is a RoleBinding?**
A: Grants Role to users/groups/service accounts in a namespace.

**Q88: What is a ClusterRoleBinding?**
A: Grants ClusterRole cluster-wide.

**Q89: What is a ResourceQuota?**
A: Limits resource consumption per namespace. CPU, memory, object count.

**Q90: What is a LimitRange?**
A: Default and limits for containers in a namespace.

---

## Advanced

**Q91: What is a Namespace?**
A: Virtual cluster within a cluster. Isolates resources. Default: default, kube-system, kube-public.

**Q92: What is Helm?**
A: Package manager for Kubernetes. Charts are packages of K8s resources.

**Q93: What is a Helm chart?**
A: Collection of files describing K8s resources. Templates with values.

**Q94: What is a Helm release?**
A: Instance of a chart running in cluster.

**Q95: What is Kustomize?**
A: Template-free customization. Overlays for different environments.

**Q96: What is a Custom Resource Definition (CRD)?**
A: Extends Kubernetes API with custom resources.

**Q97: What is an Operator?**
A: Custom controller that manages complex applications. Uses CRDs.

**Q98: What is the Horizontal Pod Autoscaler (HPA)?**
A: Scales pods based on CPU, memory, or custom metrics.

**Q99: What is the Vertical Pod Autoscaler (VPA)?**
A: Adjusts pod resource requests/limits automatically.

**Q100: What is the Cluster Autoscaler?**
A: Scales nodes based on pending pods. Adds/removes nodes.

**Q101: What is a PodDisruptionBudget (PDB)?**
A: Limits voluntary disruptions. Ensures minimum available during maintenance.

**Q102: What is a PriorityClass?**
A: Assigns priority to pods. Higher priority pods preempt lower.

**Q103: What is pod preemption?**
A: Evicting lower-priority pods to schedule higher-priority ones.

**Q104: What is a readiness probe?**
A: Checks if pod is ready to receive traffic. Fails = removed from service.

**Q105: What is a liveness probe?**
A: Checks if pod is alive. Fails = container restarted.

**Q106: What is a startup probe?**
A: Checks if application has started. Disables liveness/readiness until success.

**Q107: What probe types exist?**
A: HTTP GET, TCP socket, exec command, gRPC.

**Q108: What is a pod security policy (deprecated)?**
A: Controlled security-sensitive pod settings. Replaced by Pod Security Standards.

**Q109: What are Pod Security Standards?**
A: Privileged, Baseline, Restricted. Enforced via admission controller.

**Q110: What is admission control?**
A: Intercepts requests to API server. Validates and mutates. Webhooks.

---

## EKS Specific

**Q111: What is Amazon EKS?**
A: Managed Kubernetes service. AWS runs control plane.

**Q112: What does EKS manage?**
A: Control plane (API server, etcd, scheduler, controllers). You manage worker nodes.

**Q113: What are EKS node types?**
A: Managed node groups, self-managed nodes, Fargate.

**Q114: What is a managed node group?**
A: AWS manages EC2 instances. Automatic updates, scaling.

**Q115: What is EKS Fargate?**
A: Serverless pods. No nodes to manage. Pay per pod.

**Q116: When would you use Fargate vs managed nodes?**
A: Fargate = no node management, pay per pod. Managed nodes = more control, potentially cheaper at scale.

**Q117: What is the AWS Load Balancer Controller?**
A: Provisions ALB/NLB for Ingress and Services.

**Q118: What is IRSA?**
A: IAM Roles for Service Accounts. Gives pods specific AWS permissions.

**Q119: How does IRSA work?**
A: ServiceAccount annotated with IAM role ARN. Pod assumes role via OIDC.

**Q120: What is the EBS CSI driver?**
A: Enables EBS volumes as PersistentVolumes.

**Q121: What is the EFS CSI driver?**
A: Enables EFS as shared PersistentVolumes.

**Q122: What is eksctl?**
A: CLI tool for creating and managing EKS clusters.

**Q123: How do you update kubeconfig for EKS?**
A: `aws eks update-kubeconfig --name cluster-name`

**Q124: What is the EKS add-ons feature?**
A: Managed installation of common components (CoreDNS, kube-proxy, VPC CNI).

**Q125: What is the VPC CNI plugin?**
A: AWS CNI for EKS. Pods get VPC IPs. Native VPC networking.

---

## Scenario-Based

**Q126: A pod is stuck in Pending. How do you troubleshoot?**
A:
```bash
kubectl describe pod <name>  # Check events
kubectl get events           # Cluster events
kubectl describe nodes       # Check resources
```
Common causes: insufficient resources, node selector/affinity not matching, PVC not bound.

**Q127: A pod is in CrashLoopBackOff. What do you do?**
A:
```bash
kubectl logs <pod> --previous  # Previous container logs
kubectl describe pod <pod>     # Events and state
```
Common causes: application error, missing config/secret, failed health check, OOMKilled.

**Q128: A pod is in ImagePullBackOff. What's wrong?**
A: Can't pull container image. Check: image name/tag correct, registry accessible, imagePullSecrets configured.

**Q129: How do you debug a pod that won't start?**
A:
```bash
kubectl describe pod <name>
kubectl logs <name>
kubectl get events --field-selector involvedObject.name=<name>
```

**Q130: A service isn't routing traffic to pods. What do you check?**
A:
1. Selector matches pod labels: `kubectl get endpoints <service>`
2. Pods are Ready: `kubectl get pods`
3. Pod is listening on correct port
4. NetworkPolicy not blocking

**Q131: How do you perform a zero-downtime deployment?**
A:
- Use Deployment with rolling update strategy
- Set maxUnavailable: 0
- Configure readiness probes
- Use PodDisruptionBudget

**Q132: How do you rollback a bad deployment?**
A: `kubectl rollout undo deployment/<name>` or `kubectl rollout undo deployment/<name> --to-revision=2`

**Q133: A node is NotReady. How do you investigate?**
A:
```bash
kubectl describe node <name>  # Check conditions
ssh to node
systemctl status kubelet
journalctl -u kubelet
```
Common causes: kubelet not running, network issues, disk pressure, memory pressure.

**Q134: How do you drain a node for maintenance?**
A: `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data`

**Q135: How do you scale a deployment?**
A: `kubectl scale deployment/<name> --replicas=5` or update YAML and apply.

**Q136: How do you expose a deployment?**
A: `kubectl expose deployment/<name> --port=80 --target-port=8080 --type=LoadBalancer`

**Q137: How do you check resource usage?**
A: `kubectl top pods` and `kubectl top nodes` (requires metrics-server).

**Q138: How do you get a shell in a running container?**
A: `kubectl exec -it <pod> -- /bin/sh`

**Q139: How do you copy files to/from a pod?**
A: `kubectl cp <pod>:/path/file ./local` or `kubectl cp ./local <pod>:/path/`

**Q140: How do you view all resources in a namespace?**
A: `kubectl get all -n <namespace>` (note: doesn't show everything, use `kubectl api-resources` for full list)

**Q141: How do you delete all pods in a namespace?**
A: `kubectl delete pods --all -n <namespace>`

**Q142: How do you force delete a stuck pod?**
A: `kubectl delete pod <name> --grace-period=0 --force`

**Q143: How do you check which pods are using the most resources?**
A: `kubectl top pods --sort-by=cpu` or `--sort-by=memory`

**Q144: How do you debug DNS issues in a cluster?**
A:
```bash
kubectl run debug --image=busybox -it --rm -- nslookup kubernetes
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

**Q145: How do you check if a pod can reach another service?**
A: `kubectl exec <pod> -- curl http://service-name:port`

**Q146: A PVC is stuck in Pending. What do you check?**
A:
- StorageClass exists and is default
- PV available (if static provisioning)
- CSI driver installed and running
- Storage quota not exceeded

**Q147: How do you update a ConfigMap and have pods pick up changes?**
A: Pods don't auto-reload. Options: restart pods, use sidecar that watches for changes, or mount as subPath (doesn't update).

**Q148: How do you implement blue-green deployment?**
A: Two deployments (blue, green). Switch service selector between them.

**Q149: How do you implement canary deployment?**
A: Two deployments with same labels. Adjust replica counts. Or use Argo Rollouts/Flagger.

**Q150: How do you secure secrets in Kubernetes?**
A: Enable encryption at rest, use external secrets (AWS Secrets Manager, Vault), RBAC, avoid mounting unnecessary secrets.

---

*Master these and you'll handle any Kubernetes challenge! ☸️*
