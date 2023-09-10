# Define, build and Modify Container Images

``` Dockerfile
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flash run
```

```
# To build Docker image, this will create image locally
# -t is used to tag the image
docker build Dockerfile -t abhinavgupta2406/my-custom-app

# To push image to Docker registry
docker push abhinavgupta2406/my-custom-app

# To see images in local
docker images

# Run docker image `webapp-color`, publish port 8080 on container to 8282 on host
docker run -p 8282:8080 webapp-color
```

>When Docker builds the images, it build in the layered architecture and keeps it as cache so that it can be reused in the next build step.

---
# Authentication, Authorisation and Admission Control

## Securing Hosts
* Password based authentication disables
* SSH key based authentication

If hosts is compromised everything is compromised.

## Securing Kubernetes
**kube-apiserver** is centre of all operations within K8s. We access it using kubectl or accessing the API directly - and through that you can perform any operation on the cluster. So that is the first line of defence - controlling access to the apiserver itself.

Authentication
* Files - Username and Passwords
* Files - Username and Tokens
* Certificates
* External Authentication providers - LDAP
* Service Accounts (for applications)

Authorization
* RBAC Authorization
* ABAC Authorization (Attribute Based Access Control)
* Node Authorization
* Webhook Mode

All communications between different components is secured using TLS encryption
![](images/Pasted%20image%2020230909235001.png)

All pods can talk to each other and we can improve security between these communication using Network Policies.

---
# Authentication

All user access is managed by kube-apiserver. We cannot create or list users (unlike service accounts where we can create or list SAs)
![](images/Pasted%20image%2020230909235457.png)

![](images/Pasted%20image%2020230909235550.png)

Lets say we have users in CSV file which we can pass in kube-apiserver
![](images/Pasted%20image%2020230909235748.png)

If you're using kubeadm tool, update pod and it will automatically restart with your changes.
![](images/Pasted%20image%2020230909235900.png)

Use username and password during curl command to authenticate
![](images/Pasted%20image%2020230910000000.png)

You can optionally have group names in 4th column of CSV to add in same group. Similarly for token, you can pass it as Bearer token.
![](images/Pasted%20image%2020230910000041.png)

![](images/Pasted%20image%2020230910000222.png)

---
# KubeConfig

![](images/Pasted%20image%2020230910000604.png)
These users may have different priveleges on different clusters.
Context marry both (Clusters and Users) together - this will define which user account will be used to access which cluster.
Remember through contexts, you are not creating new user or adding any authorization through this process. You are using existing users with their existing privileges and defining what user you are going to use to access what cluster. Through this you don't have to specify user certificates and server address in each and every kubectl command you run.

![](images/Pasted%20image%2020230910001535.png)

``` yaml
apiVersion: v1
kind: Config
# You can specify default context
current-context: dev-user@google
clusters:
- name: production
  cluster:
    # It is better to give full path from root for certificate file
    certificate-authority: ca.crt
    server: https://17.17.0.51:6443
- name: development
  ...
- name: google
  ...

contexts:
- name: admin@production
  context:
    cluster: production
    user: admin
    # You can set context with namespace (this is optional)
    namespace: finance
- name: dev-user@google
  ...
- name: prod-user@production
  ...

users:
- name: admin
  user:
    # It is better to give full path from root for certificate file
    client-certificate: admin.crt
    # It is better to give full path from root for key file
    client-key: admin.key
- name: dev-user
  ...
- name: prod-user
  ...
```

If you don't specify which kubeconfig file to use, it ends up locating the default file located in the folder .kube located in users home directory. So above config can be moved to the home directory in .kube so that it becomes our default config file.

```
# To view current kubeconfig file
kubectl config view

# Alternatively you can specify a kubeconfig file in the command line
kubectl config view --kubeconfig=my-custom-config

# Update current-context in kubeconfig file
kubectl config use-context prod-user@production
```

---
# API Groups

Example of using APIs to get version and pods
![](images/Pasted%20image%2020230910003909.png)

These are the other endpoints available
![](images/Pasted%20image%2020230910004036.png)

We will focus on APIs responsible for cluster functionality which is categorised into 2 - core and named
![](images/Pasted%20image%2020230910004218.png)

![](images/Pasted%20image%2020230910004354.png)

named category is more organised
![](images/Pasted%20image%2020230910004444.png)

These can be viewed in Kubernetes cluster
![](images/Pasted%20image%2020230910004540.png)

If you're not able to access these APIs because of authentication error, you may need to pass user key and certificate file along with command
![](images/Pasted%20image%2020230910004815.png)

Alternative to above is to start kubectl proxy command which launches a proxy service locally on port 8001 and uses credentials and certificates from your kubeconfig file to access the cluster. That way you don't have to specify those in the curl command.
![](images/Pasted%20image%2020230910005042.png)

![](images/Pasted%20image%2020230910005219.png)

Kube proxy: It is used to enable connectivity between pods and services across different nodes and clusters.
Kubectl proxy: It is an HTTP proxy service created by kubectl utility to access the kube-apiserver

---
# Authorization

We want each type of user minimum level of permissions to do operations.
![](images/Pasted%20image%2020230910112611.png)

## Authorization Mechanisms
![](images/Pasted%20image%2020230910112656.png)

## Node Authorizer
This is to authorise kubelets to access Kube API to take and provide information like services, endpoints, nodes, node status etc as mentioned below
![](images/Pasted%20image%2020230910113033.png)

## ABAC - Attribute Based
They are difficult to manage as policies have to be updated manually everytime we are making changes to security for the user and restart the Kube API server
![](images/Pasted%20image%2020230910113234.png)

## RBAC
This makes much easier compared to ABAC. Instead of setting up permissions to a user or a group, we define a role.
For example, we create a developer role with set of permissions and we give this role to the users and groups.
![](images/Pasted%20image%2020230910113530.png)

Going forward we can modify the roles and it will update the permissions for all the users/groups who have this role.

## Webhook
What if you want to out source authorisation mechanisms?
![](images/Pasted%20image%2020230910113723.png)

## Authorization Mode

There are 2 more modes which are there: AlwaysAllow or AlwaysDeny
![](images/Pasted%20image%2020230910113813.png)

These 2 modes are set on Kube API server. If you don't mention, they are always default to AlwaysAllow

If you provide other authorisation modes, it is used in the order which it is specified.
![](images/Pasted%20image%2020230910114117.png)
So in above scenario, if Node authorizer denies the request, it will forward to RBAC. If RBAC is fine, it will grant permissions to the user.

> To inspect and identify the authorisation modes. Describe kube-apiserver configured in kube-system namespace

---
# RBAC in Detail

Step 1: Create the role
``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
# These are the permissiosn for pods
- apiGroups: [""] # For core group, you can leave this blank, for any other group you specify the group name. For example, to give permissions for deployments, here value will be "apps" as deployment is part apiGroups.
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
# These are the permissions for configMaps
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```

Step 2: Link the role to the user
``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```


> Both Role and RoleBinding falls into namespace. So if you want to create a role and role binding within a namespace, mention namespace in metadata section.

## Check Access

```
# Output will be yes/no

kubectl auth can-i create deployments 

kubectl auth can-i delete nodes
```

If you are admin, you can impersonate another user (say you want to test user permissions you recently created)

```
# Output will be yes/no

kubectl auth can-i create deployments --as dev-user

kubectl auth can-i create pods --as dev-user

# To check for a namespace
kubectl auth can-i create pods --as dev-user --namespace test

# Or you can try to perform as a user
kubectl get pods --as dev-user
```

## Resource names

Say you want to restrict permissions for particular set of pods, not for all the pods
``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "create", "update"]
  resourceNames: ["blue", "orange"]
```

![](images/Pasted%20image%2020230910115423.png)

---
# Cluster Roles

Nodes cannot be associated to a namespaces, they are cluster wide scoped resources. So resources are categorised as either **Namespaced or Cluster scoped**

![](images/Pasted%20image%2020230910121938.png)
This is not complete list. To get full list, use commands mentioned above.

Similar to namespaces roles and rolesbindings:

Step 1: Create cluster role
``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
```

Step 2: Create cluster role binding
``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

Cluster roles and roles binding is not limited to cluster scoped resources, you can create cluster role for namespaced resources as well. When you do that a user will have access to those resources across all namespaces. For example, earlier when namespaced role was created for pod within a namespace, but if you define clusterrole for pods, users will be able to access pods across all namespaces. Kubernetes cluster create number of clusters roles during setup.

---
# Admission Controllers

![](images/Pasted%20image%2020230910132721.png)
When we make a request through kubectl it send the request to Kube ApiServer. Kube ApiServer authenticates using user certificate file and then authorize.

![](images/Pasted%20image%2020230910132831.png)

![](images/Pasted%20image%2020230910132844.png)

There are few things which cannot be achieved using RBAC like:
![](images/Pasted%20image%2020230910133114.png)

That is where Admission Controller comes in:
![](images/Pasted%20image%2020230910133230.png)

Admission controller help us better security measures enforce how a cluster is used. Apart from simply validating configuration, it can do lot more such as change the request itself or perform additional operations before the pod gets created.

Pre-built admissions controllers
![](images/Pasted%20image%2020230910133534.png)

`NamespaceExists` is built-in admission controller which is enabled by default. This checks if namespace exists or not.
![](images/Pasted%20image%2020230910133703.png)

`NamespaceAutoProvision` is not automatically enabled by default. This creates the namespace if namespaces doesn't exists.
![](images/Pasted%20image%2020230910133829.png)

```
# To view enabled admission controller
kube-apiserver -h | grep enable-admission-plugin
```

![](images/Pasted%20image%2020230910134004.png)
In above screenshot, below one is if there is kubeadm based setup

To enable add admission controller in kube-apiserver configuration:
![](images/Pasted%20image%2020230910134102.png)

To disable admission controller use `--disable-admission-plugins` flag
![](images/Pasted%20image%2020230910134219.png)

Once updated, admission controller will take steps. For example, creating a namespace using `NamespaceAutoProvision`

`NamespaceAutoProvision` and `NamespaceExists` is deprecated with `NamespaceLifecycle` which will make sure that request to non-existent namespace is rejected and default namespaces like default, kube-system cannot be deleted.

---
# Validating and Mutating Admission Controllers

`NamespaceExists` is a type of **validating admission controller** as it validates whether namespace exists

`DefaultStorageClass` is a type of **mutating admission controller** as it adds `storageClassName: default` in PVC if storage class is not mentioned in the definition file. So mutating admission controller can change or mutate the object itself before it is created.

Generally mutating AC are called first then validating AC so that any change done by mutating AC can be validated.

To write custom AC, there is **MutatingAdmission Webhook** and **ValidatingAdmission Webhook**. For this we have to setup a Admission Webhook Server either inside the cluster or outside the cluster.

* Step 1: Deploy own webhook server which can be an API server which can be built on any platform - for example Go.
* Step 2: Host the code. It can be deployed as deployment along with service in K8s cluster.
![](images/Pasted%20image%2020230910141311.png)

---
# API Versions

When API version is at v1, it means it is at GA version (Generally Available) or stable version
![](images/Pasted%20image%2020230910145201.png)

![](images/Pasted%20image%2020230910145529.png)

An API Group can support multiple version at same time as shown below in Deployment. This means you can create same object using any of these versions of YML file.
![](images/Pasted%20image%2020230910145744.png)

Even though there are multiple versions, only one can be preferred or storage version.

Preferred version
* When you have multiple versions enabled and you run the `kubectl get deployment` command, which version is command going to query - thats defined by the preferred version. Or when you run `kubectl explain deployment`, the version you get is preferred version.

Storage version
* If objects are created with for example v1alpha1, etc, this will be converted to storage version for example v1 and then stored in etcd datatabase.

Way to check preferred version:
![](images/Pasted%20image%2020230910150541.png)

There is no direct way to check storage version.

## Enable/Disable API Groups
![](images/Pasted%20image%2020230910150707.png)

---
# API Deprecations

## Rule #1
![](images/Pasted%20image%2020230910151243.png)

![](images/Pasted%20image%2020230910150955.png)
Lets say we create a Kubernetes resource and PR is accepted and merged into Kubernetes as `v1alpha1`. For some reason, one of the resource has to be removed as users don't like it. So it will not be removed from `v1alpha1`, instead it should be removed in `v1alpha2`

## Rule #2
![](images/Pasted%20image%2020230910151337.png)

![](images/Pasted%20image%2020230910151522.png)

## Rule \#4a
![](images/Pasted%20image%2020230910151713.png)

As we do not have to keep alpha releases, below in X+1 `v1alpha1` can be removed. But for beta or GA versions, above rules should be followed.
![](images/Pasted%20image%2020230910151820.png)

![](images/Pasted%20image%2020230910152120.png)

![](images/Pasted%20image%2020230910152315.png)

![](images/Pasted%20image%2020230910152408.png)

So v1 version cannot be deprecated based on Rule #3
![](images/Pasted%20image%2020230910152426.png)

## Kubectl convert

![](images/Pasted%20image%2020230910152609.png)

