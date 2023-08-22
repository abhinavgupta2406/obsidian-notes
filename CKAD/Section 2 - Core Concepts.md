
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
