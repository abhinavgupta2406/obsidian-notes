# Readiness and Liveness Probe

Pod Lifecycle: Pending -> ContainerCreating -> Running

These are the different pod conditions which can be seen in describe command.
![](images/Pasted%20image%2020230912220851.png)

> Service starts forwarding the traffic to pod when the pod is ready. So once we start the pod, it goes to ready state (even though internally application is taking time to come up) - and this is a problem as traffic should not come to this pod.
## Readiness Probe

Types of readiness probe
![](images/Pasted%20image%2020230912221337.png)

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
        # Default is 3
        failureThreshold: 8
```

![](images/Pasted%20image%2020230912221836.png)

## Liveness Probe

![](images/Pasted%20image%2020230912222109.png)

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      livenessProbe:
        httpGet:
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
        # Default is 3
        failureThreshold: 8
```

![](images/Pasted%20image%2020230912222240.png)

---
# Container Logging

```
kubectl logs -f <pod-name> <container-name>
```

---
# Monitor and Debug Applications

```
kubectl top node

kubectl top pod
```