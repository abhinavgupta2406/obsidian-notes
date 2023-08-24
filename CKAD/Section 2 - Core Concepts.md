
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
