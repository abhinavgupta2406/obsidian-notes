# Services

Services enable connectivity between group of pods.
![](images/Pasted%20image%2020230905195118.png)

![](images/Pasted%20image%2020230905195601.png)
## Service - NodePort

![](images/Pasted%20image%2020230905195342.png)

Maps a port on the node to the port on the pod.
![](images/Pasted%20image%2020230905195833.png)

* These terms (like TargetPort, Port, NodePort) are from the viewpoint of the service.
* Inside the service, service has its own IP which is cluster IP of the service.
* NodePort range is between 30000 - 32767

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  # Remember that ports is array
  # As there can be multiple ports to map
  ports:
  - targetPort: 80 # Optional - defaults to the value of `port`
    port: 80 # Mandatory
    nodePort: 30008 # Optional - defaults to any available port between 30000 - 32767
  # And this is how service is mapped to pod
  selector:
    app: myapp
    type: front-end
```

Example pod for above service
``` yaml
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

### Scenario #2
In above scenario, we saw there was 1 pod. What if there is more than 1 pod with same labels?
![](images/Pasted%20image%2020230905201128.png)

There is no addition configuration required. In this case, service will behave as built-in load balancer and distribute load across different pods. For load balancing, it uses Random algorithm

### Scenario #3
What if pod is spread across multiple nodes?
![](images/Pasted%20image%2020230905201555.png)

There is no additional configuration required. Service will spread across all the nodes.

> In any scenarios above, service is created exactly same without you having to do any additional steps during the service creation.
> When the pods are removed/added, service is automatically updated making it highly flexible and adaptive. Once created, you typically don't have to make any additional changes.

## Service - ClusterIP

![](images/Pasted%20image%2020230905222119.png)

ClusterIP service can be used by frontend pods to connect with backend. If backend pods are added/removed, there is no worry for frontend to find out new IP address of new pods, it will just connect to "backend" service and further service will take care of addition/deletion of backend pods.

Same for backend and redis.

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
  - targetPort: 80 # This is backend application port
    port: 80 # This is service port
    
  selector:
    app: myapp
    type: back-end
```

Example pod for above service
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: back-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

> While describing the service, `Endpoints` can be useful to identify how many pods service is forwarding the load. This can be useful to identify the mistake during the use of selector - if extra pod is mapped or none of the pods are mapped to the service.

---
# Ingress Networking

Typical flow of the service
![](images/Pasted%20image%2020230906233417.png)

As node port can range from 30000 and we don't want that user should add this port in the URL while opening the website. So proxy server is added to forward the request on this port, and DNS is pointed to proxy-server
![](images/Pasted%20image%2020230906233518.png)

Above situation is generally in the data centre. What can we do on cloud?

Below, we can make wear-service as GCP load balancer and this load balancer can be mapped with DNS to access the website.
![](images/Pasted%20image%2020230906233751.png)

What if company expands and more features are added in the website? Below we can see, there are additional load balancers for more feature, and another load balancer to balance the feature load balancers. SSL is also enabled on the website.
![](images/Pasted%20image%2020230906234019.png)

We want to manage this configuration at one place and it will be difficult to manage many configurations as applications scale. Wouldn't it be nice if all the configuration is handled inside Kubernetes as part of definition file.

## Ingress
Think of ingress as layer 7 load balancer built in to the Kubernetes cluster that can be configured using native Kubernetes primitives just like any other object we have been working in Kubernetes.

![](images/Pasted%20image%2020230906234541.png)

With ingress we still need to configure to expose it to accessible outside the cluster, but that is one time configuration. So either you need to publish as NodePort or Cloud native load balancer (as shown below). Going forward you are going to perform all your load balancing, auth, SSL and URL based routing configuration on the ingress controller.
![](images/Pasted%20image%2020230906234642.png)

Solution we deploy is called **ingress controller** (like Ningx, HAProxy, etc) and set of rules are called **ingress resources**

Ingress resources are created using definition file.

Kubernetes cluster does not comes with ingress controller by default. So if you simply create ingress resources and expect them to work, it won't.
![](images/Pasted%20image%2020230906235038.png)


## Ingress Controller

Few examples of ingress controllers. Below highlighted in blue boxes are maintained by Kubernetes.
![](images/Pasted%20image%2020230906235719.png)

Below configurations are required to create ingress controller
![](images/Pasted%20image%2020230906235652.png)

``` yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controllers
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
	        # Below is the configMap we created
	        # At this point, configMap need not have entries, a blank object will do. Making one will be easy to update the configuration by easily editing the configMap and not have worry about modifying nginx configuration files.
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
          env:
            # Nginx configuration requires these 2 environment variables to read within the pods.
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            # Ports used by the ingress controller
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
```

``` yaml
# These nginx configurations includes err-log-path, keep-alive, ssl-protocols
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configuration
```

``` yaml
# We need a service to expose the controller to the external world
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    # To link the service to the deployment
    name: nginx-ingress
```

``` yaml
# Ingress controller has additional intelligence to monitor the Kubernetes cluster for ingress resources and configuring the underlying nginx server when something is changed. For ingress controller to do this, it required a service account with right set of permissions.
# For this we create SA with correct roles and role bindings
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
```

> So remember that ingress controller will be a deployment, this is not Kubernetes object to look for.
## Ingress Resource

![](images/Pasted%20image%2020230907001830.png)

### Scenario #1

![](images/Pasted%20image%2020230907002653.png)

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
    # Connecting service to Nginx controller
    service
      name: wear-service
      port:
        number: 80
```

```
kubectl get ingress
```

![](images/Pasted%20image%2020230907002951.png)

### Scenario #2

![](images/Pasted%20image%2020230907003029.png)

![](images/Pasted%20image%2020230907003124.png)

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          service:
            name: wear-service
            port:
              number: 80
      - path: /watch
        backend:
          service:
            name: watch-service
            port:
              number: 80
```

```
$ kubectl describe ingress ingress-wear-watch
...
...
Default backend: default-http-backend:80 (<none>)
...
```
In above output, it can be seen that a default backend is setup, why? This is for the paths which are not configured in definition file. This had to be updated so that users can actually see the error page as mentioned below
![](images/Pasted%20image%2020230907003632.png)

### Scenario #3
![](images/Pasted%20image%2020230907003704.png)

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths:
      - backend:
          service
            name: wear-service
            port:
              number: 80
  - host: watch.my-online-store.com
    http:
      paths:
      - backend:
          service
            name: watch-service
            port:
              number: 80
```

> Services/applications running in namespace1 will have ingress controller in namespace1, this ingress controller should not expose applications from another namespace. So you need to create ingress controller in another namespace.
> This is the best practice. Teams who have access to their namespaces should create ingress resource in their namespace instead of exposing application via another namespace.

```
# Imperative command in examples to create ingress
kubectl create ingress ingree-pay -n critical-space --rule="foo.bar/pay=pay-service:8282"

# If you are using imperative command and want hosts to be '*'
kubectl create ingress ingree-pay -n critical-space --rule="/pay=pay-service:8282"

# As you can see in rule, there is nothing before /path, which means hosts is *
# Another way is to add dummy host and create YML to update it.
```

## Rewrite Target

> This is particular to NGINX controller

The path which is mentioned in ingress will be passed to the applications it is calling.
For example, if you are adding path "/eat"  in ingress, it will call the application with path "/eat"

But what if this path is not defined in the application? For this, rewrite-target annotation is used.

The applications don't have this URL/Path configured on them:  
  
`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/`
`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/`

Without the `rewrite-target` option, this is what would happen:

`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/watch`
`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/wear`

This annotation rewrites the URL by replacing whatever is under `rules->http->paths->path` which happens to be `/pay` in this case with the value in `rewrite-target`. This works just like a search and replace function.

For example: `replace(path, rewrite-target)`  
In our case: `replace("/path","/")`

``` yaml
# In this example, ingress endpoint will be /pay, but application will be called at /
apiVersion:
kind: Ingress
metadata
  ...
  annotations:
    # Here request inside application will be /
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: pay-service
            port:
              number: 8282
        path: /pay
```

What if there are more than 1 path at different path values? See below
`replace("/something(/|$)(.*)", "/$2")

``` yaml
apiVersion:
kind: Ingress
metadata
  ...
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: http-svc
            port:
              number: 80
        path: /something(/|$)(.*)
```

---
# Network Policies
![](images/Pasted%20image%2020230909122751.png)
In above diagram, response represented by dotted lines doesn't really matters

![](images/Pasted%20image%2020230909122814.png)

## Network Security
Each pod in multiple namespace and multiple pods can reach to each other using IPs or pod names or service configured for that purpose
![](images/Pasted%20image%2020230909123011.png)

Kubernetes cluster by default is enabled "All Allow" rule that allows traffic from any pod to any other pod or services in the cluster.

This means by default all pods can communicate with each other
![](images/Pasted%20image%2020230909123330.png)

What if we don't want web pod to not communicate with DB pod directly? So we will need Network Policy for this which is another object in Kubernetes.
You link Network Policy to one or more pods and you can define rules within the network policy.

![](images/Pasted%20image%2020230909123557.png)
These rules are only applicable on the pods where network policies are applied - not to every pod. In above screenshot, API pod can communicate with DB Pod on 3306, but Web Pod cannot communicate with DB Pod.

![](images/Pasted%20image%2020230909123759.png)
So how can we map pods and network policies - using labels and selectors

> In policy type, you have to specify whether you want to allow ingress or egress or both.

![](images/Pasted%20image%2020230909123932.png)
Mapping rules in network policy

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metdata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  # IMPORTANT: Isolation only comes into effect if you have ingress or egress is defined in the policy types. For example here only Ingress is there in policy type which means only ingress traffic is isolated and all egress traffic is unaffected - meaning pod is able to make any egress calls and is not blocked. So you need to mention the type here (ingress, egress or both), otherwise there is no isolation.
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

![](images/Pasted%20image%2020230909124541.png)
For clusters which does not support network policies, you will be able to create network policies but they will not be enforced

---
# Developing Network Policies

This is more detailed in terms of Egress and Ingress

![](images/Pasted%20image%2020230909125640.png)
In above image, once you allow the rule for Ingress, response to API pod is automatically added and we don't need to add separate rule for that (example we don't need Egress to API pod). But if DB pod had to make API call to API pod separately, then we had to add Egress policy type.

> Ingress and Egress should be termed based on the pod where network policy is attaching. For example, DB pod has to allow ingress from API pod. So ingress/egress terms are from the perspective of the pod where network policy has to attach.


![](images/Pasted%20image%2020230909130203.png)
So this is a simple rule where DB pod will accept traffic from API pod

But what if there are multiple API pods with same labels in different namespace?
![](images/Pasted%20image%2020230909130312.png)
Current policy will allow this

![](images/Pasted%20image%2020230909130418.png)
So we can add namespaceSelector in addition to podSelector (Please note that a label has to be added to namespace before doing this)

Please note, namespaceSelector is not part of array instead it is combined with podSelector - this means both selectors should match.

What if you have just the namespaceSelector? This will allow web pod in prod namespace to communicate with DB Pod
![](images/Pasted%20image%2020230909130629.png)

What if there is a backup server which is outside Kubernetes cluster which needs to connect with DB Pod? As this is outside K8s cluster, we can't use selectors and labels, but we know IP of the server, so we will allow that
![](images/Pasted%20image%2020230909130755.png)

> These are the three selectors available: podSelector, namespaceSelector and ipBlock

So these are 2 rules defined:
![](images/Pasted%20image%2020230909131028.png)

This 2 rules works as OR condition. First one is to select API pods from prod namespace and second one is to select IP address.
In first one, both the criteria should meet as they are part of same `from` array.

So if we have all 3 rules separately, this means that any pod in prod namespace is allowed including Web Pod, and IP block is allowed
![](images/Pasted%20image%2020230909131317.png)

> So this small change can have big effect

Above we say Ingress, lets look at Egress. What if there is an agent in DB Pod which pushes backup to Backup server?
![](images/Pasted%20image%2020230909131457.png)

Please note below YAML file. As there is no `policyType`, neither Egress or Ingress is enabled which means NO ISOLATION is done here even though `spec.ingress` is there.
``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metdata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

```
kubectl get networkpolicies
#OR
kubectl get netpol
```