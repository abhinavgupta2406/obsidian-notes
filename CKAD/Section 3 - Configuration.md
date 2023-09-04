# Commands and Arguments in Docker
```
# To create a container (this will create and exits immediately)
docker run ubuntu

# List running containers
docker ps

# List all containers including stopped
docker ps -a

# To override default command mentioned in Docker image
docker run ubuntu [COMMAND]
docker run ubuntu sleep 5
```

```
# Correct commands
CMD sleep 5
CMD ["sleep", "5"]

# Incorrect commands
CMD ["sleep 5"]
```

```
# Dockerfile
FROM ubuntu

CMD sleep 5
```

```
# Build the image
docker build -t ubuntu-sleeper .

docker run ubuntu-sleeper
```

What if I want to change the number of seconds to sleep?
```
# One way is
docker run ubuntu-sleeper sleep 10

# But we want to pass this value is argument
docker run ubuntu-sleeper 10
```

That is where ENTRYPOINT instructions come into play
```
# Docker file
FROM ubuntu

ENTRYPOINT ["sleep"]
```

```
docker run ubuntu-sleeper 10

# The value which we are appending (here 10) will be added in front of the ENTRYPOINT and will work as whole command. Means, command will be sleep 10
```

If no value is passed (like here 10), there will be no arguments passed to the command. So how to configure default value if no value is passed?
So we will use combination of ENTRYPOINT and CMD

```
FROM ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```
Here default value is 5. So if you do not pass any value in the docker command, this will be executed `sleep 5`

If you pass a variable, that will override the CMD value. For example, if you write `docker run ubuntu-sleeper 10`, it will execute `sleep 10`

What if you want to update ENTRYPOINT at runtime?
```
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```
Above command will execute `sleep2.0 10`

---
# Command and Arguments in Kubernetes pod

`args` in pod definition file is used to override CMD parameters in Docker image. 
`command` in pod definition file is used to override ENTRYPOINT in Docker image.

Continuing from last example:

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"] # This is replacing ENTRYPOINT in Docker
      args: ["10"] #This is replacing CMD parameters. This will run `sleep2.0 10`
```

---
# Editing pods and deployments
## Editing pod
We CANNOT edit specification of an existing POD. For example, you cannot edit environment variables, service accounts, resource limits of a running pod. So you have 2 options:

### Option 1
Run `kubectl edit pod <pod-name>` command which will give option to edit. Once you try to save it, you will get error because you are trying to edit a field that is not editable. But a copy of the file with your changes is saved in temporary location which will be shown in the error.

Once you get the file, delete existing pod and create a new pod with your changes in temporary file.

### Option 2
Extract pod definition in YAML format using `-o yaml` and add in a new YAML file. Edit the YAML file as per the requirement. Delete the existing pod and use new YAML file to create the pod.

## Editing deployments
With deployments you can easily edit any field or property of the POD template. Since pod template is child of the deployment specification, with every change the deployment  will automatically delete and create a new pod with new changes.
So if we have to edit a property of pod which is part of deployment, we can do that simply by running: `kubectl edit deployment my-deployment`

---
# Environment Variables

```
# Docker command to add environment variables
docker run -e APP_COLOR=pink simple-webapp-color
```

To add in pod definition file:
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: pink
```

Different ways to add environment variables:

Plain Key Value:
```
env:
  - name: APP_COLOR
    value: pink
```

ConfigMap:
```
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
```

Secrets:
```
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
```

---
# ConfigMaps

When you have lot of pod definition files, it is difficult to manage many environment variables. So we can take environment variables out of the pod definition and manage it centrally using configuration map or ConfigMap

## Imperative Approach
```
# Imperative command to create multiple environment variables
kubectl create configmap \
	app-config --from-literal=APP_COLOR=blue \
			   --from-literal=APP_MOD=prod

# Or from file
kubectl create configmap \
	app-config --from-file=app_config.properties
```

## Declarative Approach 

Step 1: Create configMap
``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MOD: prod
```

Step 2: Inject into Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
	envFrom:
	  - configMapRef:
	      name: app-config
```

To fetch a particular environment variable (here APP_COLOR) from configMap:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
	env:
	  - name: APP_COLOR
	    valueFrom:
	      configMapRef:
	        name: app-config
	        key: APP_COLOR
```

---
# Secrets
## Imperative
```
kubectl create secret generic \
	app-secret --from-literal=DB_HOST=mysql
			   --from-literal=DB_USER=root
			   --from-literal=DB_PASSWORD=passwd

kubectl create secret generic \
	app-secret --from-file=app_secret.properties
```

## Declarative
Before writing the secret values in definition file, encode the values using `echo -n 'passwd' | base64` (on Linux machines). To decode use `echo -n '<....>' | base64 --decode`

``` yaml
apiVersion: v1
kind: Secret
metadata:
	name: app-secret
data:
	DB_HOST: val1 # Values are base64 encoded
	DB_USER: val2 # Values are base64 encoded
	DB_PASSWORD: val3 # Values are base64 encoded
```

``` yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
	    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
	envFrom:
	  - secretRef:
	      name: app-secret
```

## Ways to inject secrets
Please note that similar ways can be done for configMap

All secrets:
```
envFrom:
  - secretRef:
      name: app-config
```

Single environment
```
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_PASSWORD
```

Volume
```
volumes:
- name: app-secret-volume
  secret:
    secretName: app-secret
```

## Important Points
* Secrets are not encrypted, they are encoded.
	* So do not check in secret objects to SCM in your code.
* Secrets are not encrypted in ETCD
	* So enable encryption at rest
* Pods/deployments in same namespace can access secrets
	* Configure least-privelege access to secrets - RBAC
* Consider third party secret store providers like AWS, Azure, GCP, Vault

---
# Docker Security

* Containers and hosts share the same kernel.
* Containers are isolated using namespaces in linux
* Host has namespace and container has their own namespace.
* All the processes run by the containers are run on the hosts itself but in their own namespace.
* As far as Docker container is concerned it is in its own namespace and it can see its own processes only. It cannot see anything outside of it or any other namespace.
* When you list processes using `ps` command inside container, you will see container processes running. But on hosts, it will show all the processes including the one running in container.
* So above is called process isolation.

## User Security
* By default, processes inside containers run as root user. So both inside and outside the container i.e. hosts, process will run as root.
* If you don't want to run the process as root, you may set the user using docker run command and specify the new user id: `docker run --user=1000 ubuntu sleep 3600`
	* Here user id is 1000
* Another way is to use user id in the image
``` Dockerfile
FROM ubuntu

USER 1000
```

```
docker build -t my-ubuntu-image .

docker run my-ubuntu-image sleep 3600
```

## Is root user within container is same as root user on hosts?

Docker implements set of features that limits the ability of root user within container, so the root user within the container isn't like the root user on host. Docker uses Linux capabilities.

Root user capabilities can be added in the container: `docker run --cap-add MAC_ADMIN ubuntu`

---
# Security Contexts

Container are encapsulated in pods. You may choose to set the security settings at container level or at pod level. If you change at pod level, it will carry over to all the containers inside pods.

If you configure at both container and pod level, settings on the container will override the settings on the pod.

To enable security context at pod level
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: web-pod
spec:
	securityContext:
		runAsUser: 1000
	containers:
		- name: ubuntu
		  image: ubuntu
		  command: ["sleep", "3600"]
```

To enable security context at container level, bring the whole securityContext block under container
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: web-pod
spec:
	containers:
		- name: ubuntu
		  image: ubuntu
		  command: ["sleep", "3600"]
		  securityContext:
			  runAsUser: 1000
			  # NOTE: Capabilities are only supported at the container level and not at the pod level
			  capabilities:
				  add: ["MAC_ADMIN"]
```

---
# Service Account (REVIEW VIDEO)

There are two types of account:
* User account
	* Used by humans
	* Example, a user account can be for administrator for administrative tasks or a developer to deploy the applications
* Service account
	* Used by machines
	* Used by applications to interact with Kubernetes cluster
	* Example, Prometheus uses service account to pull metrics for performance metrics or Jenkins uses service account to deploy applications on Kubernetes cluster.

When we create a SA, token is generated and stored in a secret. Then secret is linked to SA which can tell the token. To view the token, describe the secret. (Token is bearer token)
```
kubectl create serviceaccount dashboard-sa

kubectl get serviceaccount

# This will reveal the name of secret where token is stored
kubectl describe serviceaccount dashboard-sa
```

In above scenario, this token can be used by an application or service (like Prometheus) to interact with Kubernetes.

What is application or service is inside a pod in the same cluster? This is much simpler. Secret containing token will be mounted as volume in the pod hosting the application or service. This can be easily read by the application and we don't have to provide the token manually.

For every namespace created, there will be `default` SA. When a pod is created the `default` SA and its token are automatically mounted to that pod as a volume mount. `default` SA is very restricted, it can run only basic Kubernetes API queries.
To specify the different SA, modify the pod definition file:
``` yaml
apiVersion: v1
kind: Pod
metadata:
	name: my-kubernetes-dashboard
spec:
	containers:
		- name: my-kubernetes-dashboard
		  image: my-kubernetes-dashboard
	serviceAccountName: dashboard-sa
```
Make sure to create the SA before adding SA above.

In v1.22
* Before this version, there was no expiry in the token.
* `TokenRequestAPI` was added in this version which generates tokens which are (to make it more secure):
	* Audience bound
	* Time bound
	* Object bound

In v1.24
* Creation of SA will not generate the token.
* It has to be created separately: `kubectl create token dashboard-sa`
* If you want to use SA as old way, create the secret with non-expiring SA token
``` yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
	name: mysecretname
	annotations:
		kubernetes.io/service-account.name: dashboard-sa
```

In K8s documentation, it is mentioned that you should only create a SA token secret object if you can't use `TokenRequest` API to obtain a token and security exposure of persisting a non-expiring token credential in a readable API object is acceptable to you.

---
# Resource Requirements

__K8s scheduler__ decides where the pod should go in the node. It identifies the resource requirements from the pod and identifies the best node to place a pod on.

## Resource Requests (or minimum amount)
``` yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
	  image: simple-webapp-color
	  ports:
	    - containerPort: 8080
	  resources:
		  requests:
			  memory: "4Gi"
			  cpu: 2
```

## Resource - CPU
1m means 1 milli. This is the min. value which can be requested.

1 CPU is equivalent to 1 vCPU

## Resource - Memory
It can be represented in G (Gigabyte) or Gi (Gibibyte), M or Mi, K or Ki

## Resource Limits (or max amount)

``` yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
	  image: simple-webapp-color
	  ports:
	    - containerPort: 8080
	  resources:
		  requests:
			  memory: "4Gi"
			  cpu: 2
		  limits:
			  memory: "2Gi"
			  cpu: 2
```

## Exceed Limits

If CPU limit is reached, it will not let the pod consume more CPU.

But with memory limit is reached, pod is allowed to have more memory, but it will be terminated with OOM (out of memory) error.

## Default Behaviour

By default there is no request or limit defined in a pod.

## Behaviour - CPU

* NO REQUEST + NO LIMITS
	* In this case, 1st pod can use all the CPU resources and prevent the 2nd pod to get required resources.
	* Not Ideal
* NO REQUEST + LIMITS
	* In this scenario, Kubenetes sets REQUESTS = LIMITs
* REQUESTS + LIMITS
	* If pod 1 wants more CPU and pod 2 isn't consuming much CPU resource, this is not the ideal case.
	* So we should allow pod 1 to have CPU when pod 2 doesn't need the CPU.
* REQUESTS + NO LIMITS
	* Ideal
	* It will help avoid scenario in last point.

## Behaviour - Memory

* NO REQUEST + NO LIMITS
	* Not ideal
* NO REQUEST + LIMITS
	* In this scenario, Kubenetes sets REQUESTS = LIMITs
* REQUESTS + LIMITS
* REQUESTS + NO LIMITS
	* If pod 2 wants more memory, pod 1 has to be deleted.
	* Once the memory is assigned, only way to retrieve is to kill the pod.
	* Because unlike CPU we cannot throttle memory.

## Setting Default - LimitRange

If you change existing pod, it will not affect. It will only affect new pods.

``` yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      # Limit
      cpu: 500m
	defaultRequest:
	  # Request
	  cpu: 500m
	max:
	  # Limit
	  cpu: "1"
	min:
	  # Request
	  cpu: 100m
    type: Container
```

``` yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
  - default:
      # Limit
      memory: 1Gi
	defaultRequest:
	  # Request
	  memory: 1Gi
	max:
	  # Limit
	  memory: 1Gi
	min:
	  # Request
	  memory: 500Mi
    type: Container
```

Above values are examples, not recommended.

## Resource Quotas

Set hard limits at namespace level - say all pods should not consume more then x

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```