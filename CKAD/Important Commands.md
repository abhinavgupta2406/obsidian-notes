```
# To list all aliases
alias

# You know what you are doing
alias k=kubectl
```

```
# In exam, it can be asked to set contexts or change namespace
kubectl config set-context mycontext --namespace=mynamespace
```

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

# Imperative command to create pod and create service of ClusterIP by same name and exposing target port of 80 by pod
kubectl run httpd --image=httpd:alpine --port=80 --expose

# Imperative command to expose a pod through service
kubectl expose pod redis --port 6379 --name redis-service
```

```
# Imperative command to override arguments and commands

# Please note that `--` is seperator between kubectl command and argument to the pods

kubectl run webapp-green --image=kodekloud/webapp-color -- --color green

# Start the nginx pod using the default command, but use custom arguments (arg1 .. argN) for that command
kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>

# Start the nginx pod using a different command and custom arguments
kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>
```

```
# To replace pod after editing the definition file
kubectl replace --force -f pod-definition.yaml
```

```
# To create ingress
kubectl create ingress <ingress-name> --rule="host/path=service:port"

kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```

```
# Get current context (this doesn't give much info)
kubectl config current-context

# Change namespace to get its resources without mentioning -n
kubectl config set-context $(kubectl config current-context) --namespace=dev

# View name of the namespace in current context
# It will be there in contexts[0].context.namespace
kubectl config view
```

```
# Get pods based on labels
kubectl get pods --selector=name=payroll
```

```
# To count number of resources (including headers)
kubectl get clusterroles -A | wc -l

# This will give line count including headers, so find value will be outputValue - 1.

# To count without headers
kubectl get clusterroles -A --no-headers | wc -l
```

```
# Get namespaced resources
kubectl api-resources --namespaced=true

# Get cluster wide resources
kubectl api-resources --namespaced=false
```

* https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/