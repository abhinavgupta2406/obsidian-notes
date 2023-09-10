# Multi-Container Pods

There are different patterns of multi-container pods:
* Ambassador
* Adapter
* Sidecar

Multi-container pods share the same lifecycle, network and storage, so that a service is not required to connect with each other.

![](../Pasted%20image%2020230910220008.png)

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
  - name: log-agent
    image: log-agent
```
Above pattern is sidecar pattern

## Design Pattern - Sidecar

![](../Pasted%20image%2020230910220405.png)

Example: A sidecar container responsible for sending logs to central logging server.

## Design Pattern - Adapter

Building on above example, if applications are generating logs in different formats, we need an application (in adapter container) to convert into common format before sending it to central server.
![](../Pasted%20image%2020230910220604.png)

## Design Pattern - Ambassador

While writing application, we need different databases at different stage of development. So we may need to change application logic to connect to different databases. We may choose to outsource such logic in ambassador container which will proxy your request to right database.
![](../Pasted%20image%2020230910220823.png)

> These are different patterns to design multi-container pod. When it comes to implementing using the pod definition file, it is always the same.

---
# Init Containers

When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts.

You can configure multiple such initContainers as well, like how we did for multi-pod containers. In that case each init container is run **one at a time in sequential order**.

If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```