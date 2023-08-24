```
# To get explaination of resource type and their API version
kubectl explain replicaset
```

```
# Multiple pods can be deleted together
kubectl delete pod pod-1 pod-2 pod-3
```

```
# Get all resources from all namespaces
kubectl get all --all-namespaces

# Or
kubectl get all -A
```

```
# Imperative commands to generate YAML manifest files
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

