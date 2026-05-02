# Kubernetes — Command Reference ☸️

Every kubectl command you'll use, organized by task.

---

## Cluster & Context

```bash
# ─── Context (which cluster you're talking to) ───
kubectl config get-contexts                        # List all contexts
kubectl config current-context                     # Current context
kubectl config use-context my-cluster              # Switch context
kubectl config set-context --current --namespace=production  # Set default namespace

# ─── Cluster info ───
kubectl cluster-info                               # Cluster endpoint
kubectl get nodes                                  # List nodes
kubectl get nodes -o wide                          # With IPs and OS
kubectl describe node node-name                    # Node details
kubectl top nodes                                  # Node resource usage

# ─── EKS specific ───
aws eks update-kubeconfig --name my-cluster --region ca-central-1
```

---

## Pods

```bash
# ─── List ───
kubectl get pods                                   # Current namespace
kubectl get pods -n production                     # Specific namespace
kubectl get pods -A                                # All namespaces
kubectl get pods -o wide                           # With node and IP
kubectl get pods -w                                # Watch (live updates)
kubectl get pods --show-labels                     # With labels
kubectl get pods -l app=my-app                     # Filter by label
kubectl get pods --field-selector status.phase=Running  # Filter by status
kubectl get pods --sort-by='.status.startTime'     # Sort by start time
kubectl get pods -o json                           # JSON output
kubectl get pods -o yaml                           # YAML output
kubectl get pods -o name                           # Names only

# ─── Describe ───
kubectl describe pod pod-name                      # Full details + events
kubectl describe pod pod-name -n production

# ─── Logs ───
kubectl logs pod-name                              # Current logs
kubectl logs pod-name -f                           # Follow (tail -f)
kubectl logs pod-name --tail=100                   # Last 100 lines
kubectl logs pod-name --since=1h                   # Last hour
kubectl logs pod-name -c container-name            # Specific container
kubectl logs pod-name --previous                   # Previous crash logs
kubectl logs pod-name --all-containers             # All containers
kubectl logs -l app=my-app                         # By label (all pods)
kubectl logs -l app=my-app --max-log-requests=10   # Multiple pods

# ─── Exec ───
kubectl exec -it pod-name -- /bin/sh               # Shell
kubectl exec -it pod-name -- bash                  # Bash
kubectl exec -it pod-name -c container -- sh       # Specific container
kubectl exec pod-name -- ls /app                   # Run command
kubectl exec pod-name -- env                       # View env vars
kubectl exec pod-name -- cat /etc/resolv.conf      # Check DNS config
kubectl exec pod-name -- curl http://service:80    # Test connectivity

# ─── Copy ───
kubectl cp pod-name:/path/file ./local             # From pod
kubectl cp ./local pod-name:/path/file             # To pod
kubectl cp pod-name:/path/file ./local -c container  # Specific container

# ─── Port Forward ───
kubectl port-forward pod-name 8080:80              # Pod port forward
kubectl port-forward svc/service-name 8080:80      # Service port forward
kubectl port-forward deploy/my-app 8080:80         # Deployment port forward

# ─── Create / Delete ───
kubectl run debug --image=busybox -it --rm -- sh   # Temp debug pod
kubectl run nginx --image=nginx --port=80          # Quick pod
kubectl delete pod pod-name                        # Delete pod
kubectl delete pod pod-name --force --grace-period=0  # Force delete
kubectl delete pods --all -n namespace             # Delete all in namespace
```

---

## Deployments

```bash
# ─── List ───
kubectl get deployments                            # List
kubectl get deploy -n production                   # Specific namespace
kubectl describe deployment my-app                 # Details

# ─── Create / Update ───
kubectl create deployment my-app --image=nginx     # Create
kubectl apply -f deployment.yaml                   # Apply manifest
kubectl set image deployment/my-app app=nginx:1.25 # Update image
kubectl edit deployment my-app                     # Edit live (opens editor)

# ─── Scale ───
kubectl scale deployment my-app --replicas=5       # Scale
kubectl autoscale deployment my-app --min=2 --max=10 --cpu-percent=70  # HPA

# ─── Rollout ───
kubectl rollout status deployment/my-app           # Rollout status
kubectl rollout history deployment/my-app          # Revision history
kubectl rollout undo deployment/my-app             # Rollback to previous
kubectl rollout undo deployment/my-app --to-revision=2  # Rollback to specific
kubectl rollout restart deployment/my-app          # Rolling restart
kubectl rollout pause deployment/my-app            # Pause rollout
kubectl rollout resume deployment/my-app           # Resume rollout

# ─── Delete ───
kubectl delete deployment my-app
```

---

## Services

```bash
kubectl get svc                                    # List services
kubectl get svc -A                                 # All namespaces
kubectl describe svc service-name                  # Details
kubectl get endpoints service-name                 # Pod IPs behind service

# Create
kubectl expose deployment my-app --port=80 --target-port=8080  # ClusterIP
kubectl expose deployment my-app --port=80 --type=NodePort     # NodePort
kubectl expose deployment my-app --port=80 --type=LoadBalancer # LoadBalancer

kubectl delete svc service-name                    # Delete
```

---

## ConfigMaps & Secrets

```bash
# ─── ConfigMaps ───
kubectl create configmap my-config --from-literal=key=value
kubectl create configmap my-config --from-file=config.properties
kubectl create configmap my-config --from-env-file=.env
kubectl get configmaps
kubectl describe configmap my-config
kubectl get configmap my-config -o yaml            # View contents
kubectl delete configmap my-config

# ─── Secrets ───
kubectl create secret generic my-secret --from-literal=password=s3cret
kubectl create secret generic my-secret --from-file=ssh-key=~/.ssh/id_rsa
kubectl create secret tls my-tls --cert=cert.pem --key=key.pem
kubectl create secret docker-registry my-reg \
    --docker-server=registry.com \
    --docker-username=user \
    --docker-password=pass
kubectl get secrets
kubectl describe secret my-secret
kubectl get secret my-secret -o jsonpath='{.data.password}' | base64 -d  # Decode
kubectl delete secret my-secret
```

---

## Namespaces

```bash
kubectl get namespaces                             # List
kubectl create namespace production                # Create
kubectl delete namespace production                # Delete (deletes EVERYTHING in it)
kubectl get all -n production                      # All resources in namespace
kubectl config set-context --current --namespace=production  # Set default
```

---

## Apply & Delete Manifests

```bash
# ─── Apply ───
kubectl apply -f manifest.yaml                     # Apply single file
kubectl apply -f dir/                              # Apply all in directory
kubectl apply -f https://url/manifest.yaml         # Apply from URL
kubectl apply -k ./kustomize-dir/                  # Apply Kustomize
kubectl apply -f manifest.yaml --dry-run=client    # Dry run (client-side)
kubectl apply -f manifest.yaml --dry-run=server    # Dry run (server-side)
kubectl diff -f manifest.yaml                      # Show diff before apply

# ─── Delete ───
kubectl delete -f manifest.yaml                    # Delete from file
kubectl delete -f dir/                             # Delete all in directory
kubectl delete all --all -n namespace              # Delete everything
```

---

## Resource Usage

```bash
kubectl top pods                                   # Pod CPU/memory
kubectl top pods -A                                # All namespaces
kubectl top pods --sort-by=cpu                     # Sort by CPU
kubectl top pods --sort-by=memory                  # Sort by memory
kubectl top nodes                                  # Node CPU/memory
kubectl top pods -l app=my-app                     # By label
```

---

## Events & Debugging

```bash
kubectl get events                                 # All events
kubectl get events --sort-by='.lastTimestamp'       # Sorted by time
kubectl get events -n production                   # Namespace events
kubectl get events --field-selector type=Warning   # Warnings only
kubectl get events --field-selector involvedObject.name=pod-name  # For specific pod

# Debug DNS
kubectl run debug --image=busybox:1.36 -it --rm -- nslookup kubernetes
kubectl run debug --image=busybox:1.36 -it --rm -- nslookup service-name.namespace.svc.cluster.local

# Debug networking
kubectl run debug --image=curlimages/curl -it --rm -- curl http://service:80
kubectl run debug --image=nicolaka/netshoot -it --rm -- bash
```

---

## Labels & Annotations

```bash
# Labels
kubectl label pod pod-name env=prod                # Add label
kubectl label pod pod-name env-                    # Remove label
kubectl label pod pod-name env=staging --overwrite # Update label
kubectl get pods -l env=prod                       # Filter by label
kubectl get pods -l 'env in (prod,staging)'        # Multiple values
kubectl get pods -l env!=dev                       # Not equal

# Annotations
kubectl annotate pod pod-name description="my pod"
kubectl annotate pod pod-name description-          # Remove
```

---

## Ingress

```bash
kubectl get ingress                                # List
kubectl get ingress -A                             # All namespaces
kubectl describe ingress ingress-name              # Details
kubectl apply -f ingress.yaml
kubectl delete ingress ingress-name
```

---

## Helm (via kubectl context)

```bash
helm list                                          # List releases
helm list -A                                       # All namespaces
helm install my-app ./chart                        # Install
helm install my-app ./chart -f values-prod.yaml    # With values
helm install my-app ./chart -n production          # In namespace
helm upgrade my-app ./chart -f values-prod.yaml    # Upgrade
helm upgrade --install my-app ./chart              # Install or upgrade
helm rollback my-app 1                             # Rollback
helm history my-app                                # History
helm uninstall my-app                              # Uninstall
helm template my-app ./chart                       # Render locally
helm status my-app                                 # Release status
helm get values my-app                             # Current values
helm get manifest my-app                           # Rendered manifests
```

---

## Other Resources

```bash
# HPA
kubectl get hpa
kubectl describe hpa my-app-hpa

# PV / PVC
kubectl get pv                                     # PersistentVolumes
kubectl get pvc                                    # PersistentVolumeClaims
kubectl describe pvc my-pvc

# StatefulSet
kubectl get statefulsets
kubectl describe statefulset my-db

# DaemonSet
kubectl get daemonsets -A
kubectl describe daemonset fluentd

# Jobs / CronJobs
kubectl get jobs
kubectl get cronjobs
kubectl create job test-job --from=cronjob/my-cron # Manual trigger

# ServiceAccount
kubectl get serviceaccounts
kubectl describe serviceaccount my-sa

# RBAC
kubectl get roles
kubectl get rolebindings
kubectl get clusterroles
kubectl get clusterrolebindings
kubectl auth can-i create pods                     # Check permissions
kubectl auth can-i create pods --as=deploy-user    # Check for another user

# NetworkPolicy
kubectl get networkpolicies

# All resources
kubectl api-resources                              # All resource types
kubectl api-resources --namespaced=true            # Namespaced only
kubectl explain pod.spec.containers                # Field documentation
```

---

## Output Formatting

```bash
kubectl get pods -o wide                           # Extra columns
kubectl get pods -o yaml                           # YAML
kubectl get pods -o json                           # JSON
kubectl get pods -o name                           # Names only
kubectl get pods -o jsonpath='{.items[*].metadata.name}'  # JSONPath
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
kubectl get pods --no-headers                      # No header row
```

---

## Quick Aliases (add to ~/.bashrc)

```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgpa='kubectl get pods -A'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployments'
alias kgn='kubectl get nodes'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias klf='kubectl logs -f'
alias kex='kubectl exec -it'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'
alias kns='kubectl config set-context --current --namespace'
```

---

*kubectl is your daily driver in Kubernetes — muscle memory these! ☸️*
