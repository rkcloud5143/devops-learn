# Kubernetes — Essential kubectl Commands

## Cluster Info
```bash
kubectl cluster-info
kubectl get nodes
```

## Deployments
```bash
kubectl apply -f deployment.yml       # Create/update
kubectl get deployments               # List
kubectl rollout status deployment/my-app  # Watch rollout
kubectl rollout undo deployment/my-app    # Rollback!
```

## Pods
```bash
kubectl get pods                      # List pods
kubectl get pods -o wide              # With more details
kubectl describe pod <pod-name>       # Detailed info
kubectl logs <pod-name>               # View logs
kubectl logs <pod-name> -f            # Follow logs
kubectl exec -it <pod-name> -- sh    # Shell into pod
```

## Services
```bash
kubectl get services
kubectl get svc
```

## Debugging
```bash
kubectl get events                    # Cluster events
kubectl top pods                      # Resource usage
kubectl top nodes
```
