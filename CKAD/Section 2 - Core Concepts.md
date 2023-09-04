
# Pods

To get help from the CLI

```
kubectl run --help
```

To get more details of the pods like nodes, etc

```
kubectl get pods -o wide
```

To dry run before creating any resource

```
kubectl run redis --image=redis --dry-run=client -o yaml
```

To get pod definition file of existing pod

```
kubectl get pod redis -o yaml > pod-defn.yml
```

To edit pod properties

```
kubectl edit pod redis
```

Example pod definition file

```
# pod-definition.yml

apiVersion: v1
kind: Pod

metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

---
# ReplicaSets

Replication controller ensures specified number of pods are running - either 1 or more

Replication controller and replicasets have some purpose but there is a difference. Replication controller is old.

## Replication Controller

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  # Template will have exact pod definition as mentioned above.
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec:
        containers:
        - name: nginx-controller
          image: nginx
  replicas: 3
```

`kubectl create -f rc-definition.yml`

`kubectl get replicationcontroller`

## ReplicaSets

The only difference from Replication Controller is selectors.

If there are 3 pods running with same labels as selectors before ReplicaSets was created and replicas in replicasets are also 3, ReplicaSets will not create new pods. It will include already created pods in its sets.

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  # Template will have exact pod definition as mentioned above.
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec:
        containers:
        - name: nginx-controller
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

Replicaset definition file
``` yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-controller
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

```
kubectl create -f replicaset-definition.yml

kubectl get replicaset
```

To increase number of replicas there are multiple ways

```
# Update replicas in definition YML file and run
kubectl replace -f replicaset-definition.yml

# Using `scale` command (but this will not increase the number of replicas in definition YML file):
kubectl scale --replicas=6 -f replicaset-definition.yml

# Using replicaset name:
kubectl scale --replicas=6 replicaset myapp-replicaset
# or
kubectl scale --replicas=6 rs/myapp-replicaset
```

Replicaset commands:

```
# This will delete pods as well
kubectl delete replicaset myapp-replicaset

# Two rs can be deleted together
kubectl delete rs resplicaset-1 replicaset-2

# This will replace or update replicaset
kubectl replace -f replicaset-definition.yml

# Scale without modifying the file
kubectl scale --replicas=6 -f replicaset-definition.yml

# Shortcut
kubectl get rs
```

---
## Deployments

Definition will be exactly same as Replicaset except `kind` is `Deployment`

Deployment creates replicaset for replicas
![[Pasted image 20230824221438.png]]

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-controller
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

```
# Shortcut for deployments
kubectl get deploy

# Get help for creating deployment
kubectl create deployment --help

# Create deployment with command
kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
```

---
## Namespaces

Namespaces created by K8s automatically:
* default
* kube-system
* kube-public

### DNS
If calling the service within same namespace, we can just use the service name.
If calling the service in another namespace, a DNS is required.
![[Pasted image 20230824224041.png]]

![[Pasted image 20230824224125.png]]

```
# To create a definition file in a namespace
kubectl create -f pod-definition.yml --namespace=dev
```

If you want to create a resource without specifying the namespace:
```yaml
apiVersion: v1
kind: Pod

metadata:
  name: myapp-pod
  namespace: dev
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-controller
    image: nginx
```

```
# Command to create new namespace
kubectl create ns dev
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```
# To always choose a namespace when looking for resources (for example, here we are switching to dev namespace)
kubectl config set-context $(kubectl config current-context) --namespace=dev

# After above command, you need to specify namespace for default
kubectl get pods --namespace=default

# Get resources across all namespaces
kubectl get pods --all-namespaces
```

### Resource Quota for Namespace

``` yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

---
# Taints and Tolerations

Nothing to do with security

By default pod don't have any tolerations. So if we apply a taint (or spray) to a node, pods with default values cannot be created in those nodes.

TAINTS are set on NODES
TOLERATIONS are set on PODS

## Taints - Nodes

```
# Syntax
kubectl taint nodes node-name key=value:taint-effect

# taint-effect values can be NoSchedule, PreferNoSchedule and NoExecute
```

Pods that DO NOT TOLERATE this taint will have 3 effects: NoSchedule, PreferNoSchedule and NoExecute

NoSchedule: Pods will not be scheduled on the node
PreferNoSchedule: System will try to avoid placing the pod on the node, but that is not guaranteed.
NoExecute: New pods will be not be scheduled and existing pods will be evicted if they do not tolerate the taint. Existing pods may have scheduled on the node when the taint was applied

```
# Example
kubectl taint nodes node1 app=blue:NoSchedule
```

## Tolerations - Pods
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  # Values should be in double quotes
  # These values match from `taint nodes` command.
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

So a pod with toleration can go to default nodes as well (where no taint is set). If we want to move certain pods to particular nodes, it will be done using __Node Affinity__

> Scheduler never schedules pod on master node because Kubernetes cluster taints its master node. To check taint on master node: `kubectl describe node kubemaster | grep Taint`

To remove taint from node: `kubectl edit node node01`
Or another way is to remove by adding - at command like below:
```
# There's a hyphen sign at the end
kubectl taint node controlplane <taint-value-copied-from-describe>-
```

---
# Node Selectors

If we want a pod that should be assigned to a particular node (for example data processing pod requires high resource, so it should go to a node with higher resources)
``` yaml
apiVersion: v1
kind: Pod
metadata
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  nodeSelector:
    # This label will be added to node
    size: Large
```

## Label Nodes

`kubectl label nodes <node-name> <label-key>=<label-value>`

For above example: `kubectl label nodes node-1 size=Large`

## Limitation of Node Selectors
* What if we want to add pod based on multiple labels? Like Large or Medium
* Or exclude a node. For example, add the pod where NOT Small.

So Node Affinity and Anti Affinity is used

---
# Node Affinity

This is to ensure that pods are hosted on particular nodes.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            # These values are matching the labels on nodes
            - Large
            - Medium
```


``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: NotIn
            values:
            # These values are matching the labels on nodes
            - Small
```

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            # This is checking whether size label is there in any node.
            operator: Exists
```

**requiredDuringSchedulingIgnoredDuringExecution**
* As name suggests a `nodeSelectorTerms` should work when creating/scheduling.
* But doesn't evict the existing running pod in the node if changes are there in node labels - example if `size` label is removed.

**preferredDuringSchedulingIgnoredDuringExecution**
* As name suggests a `nodeSelectorTerms` may/may not work when creating/scheduling.
* But doesn't evict the existing running pod in the node if changes are there in node labels - example if `size` label is removed.

**requiredDuringSchedulingRequiredDuringExecution**
* As name suggests a `nodeSelectorTerms` may/may not work when creating/scheduling.
* It will evict the existing running pod in the node if changes are there in node labels - example if `size` label is removed.

