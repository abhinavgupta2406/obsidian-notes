# Custom Resource Definition (CRD)

> CRD objects can be either namespaced or cluster scoped.

## Resource and Controller
When you create a deployment resource, k8s stores its data in ETCD datastore.

When we define replicas=3, who or what is responsible for that? - This is the job of a controller. For deployment resource, deployment controller is built-in in K8s.

So controller job is to continuously monitor the status of the resources that it suppose to manage. So when we create/update/delete the resource, it makes necessary changes to the cluster to match what we have done.
In case of deployment, controller creates a replicaset which in turn creates as many pods as we specified in the deployment definition file.
![](images/Pasted%20image%2020230910155416.png)

This is how controller looks like:
![](images/Pasted%20image%2020230910155437.png)

For every resource there is a controller which is continously monitoring the status of the objects and making the necessary changes on the cluster to maintain the state as expected.
![](images/Pasted%20image%2020230910155536.png)

## Custom Resource and CRD

``` yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Mumbai
  to: London
  number: 2
```

We want to create, get and delete this resource. So when we create/delete the resource, it will be updated in etcd datastore.

Till now, it is not actually going to book the ticket, but we want to actually book the flight ticket by calling the flight booking API.

So we will need a controller who is actually going to monitor the changes in etcd for creation/updation/deletion of the flight ticket resource and when we create/update, it is going to make a call to flight booking API. And when we delete the resource, it will make call to delete in the API.

![](images/Pasted%20image%2020230910160222.png)

With above definition file for `my-flight-ticket`, it will end up with error as resource definition for FlightTicket doesn't exists. So we have to define first what is it that we want to create, for that we need CRD.

``` yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com
spec:
  # Whether namespaced or cluster scoped
  scope: Namespaced
  # Name of the API Group - this is there in apiVersion of resource
  group: flights.com
  names:
    # This is the kind to use when creating the resource
    kind: FlightTicket
    # Used to display resource type in output of kubectl command
    singular: flightticket
    # Used by `kubectl api-resource`
    plural: flighttickets
    shortName:
      - ft
  versions:
    - name: v1
      # Out of multiple versions, we need to define which one should be served by API server
      served: true
      # If there are multiple versions, which one should be marked as storage version - there can be only one.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                from:
                  type: string
                to:
                  type: string
                number:
                  # We can add validations here from OpenAPI schema
                  type: integer
```

So now we can run kubectl create, get and delete command on FlightTicket resource. But it will only them to create them and store in etcd, but its not going to do anything about it as we don't have a controller for it.

---
# Custom Controllers

![](images/Pasted%20image%2020230910163102.png)

You can deploy the custom controller as a pod.

---
# Operator Framework

Currently CRD and controller are created and deployed separtely, but it can be packaged together to deploy as a single entity using operator framework.

![](images/Pasted%20image%2020230910163316.png)

---
# Blue/Green Deployments

We have already read Recreate and Rolling update deployment strategy.

Recreate: 
* Newer version is deployed by first destroying the existing versions and then creating newer versions of the application instances.
* Problem is, during the period when the older version is down and newer version is getting up, application is down.

Rolling Update
* We take down the older version and bring up the newer version one by one. This way application never goes down and upgrade is seamless.
* This is the default deployment strategy.

![](images/Pasted%20image%2020230910164522.png)

Blue/Green
* New version is deployed alongside the old version. So old version is called blue and new is green. 100% of the traffic is still routed to old version and tests are being run on new version.
* Once all tests are passed, we switch all the traffic to the newer version all at once.
* These strategies are best with service meshes like Istio.

![](images/Pasted%20image%2020230910164831.png)

![](images/Pasted%20image%2020230910164849.png)

How is it done in Kubernetes? It is using service and labels & selectors

Initially service is using label `version: v1` to forward traffic to old version. Then we deploy new versions with separate label `version: v2`.
![](images/Pasted%20image%2020230910165034.png)

Then label is changed in service to forward traffic to new versions.
![](images/Pasted%20image%2020230910165200.png)

With old version
![](images/Pasted%20image%2020230910165407.png)

Created new version
![](images/Pasted%20image%2020230910165439.png)

Update label in service
![](images/Pasted%20image%2020230910165514.png)

---
# Canary Updates Deployment

We deploy the new version and route only small percentage of traffic to it, so majority of traffic is routing to older version and small percentage routed to new version. At this point, we run tests and if everything looks good, we upgrade the original deployment with newer version of the application. Then we get rid of the canary deployment - old version.

First step is to send traffic to both versions. So we can create a common label (here `app: front-end`) and we will update in service, which will start sending 50% to old and 50% to new. So this is solving Point 1 in screenshot.
![](images/Pasted%20image%2020230910170211.png)

But we want to reduce the traffic on new version, this can be done by reducing number of pods and then increasing it slowly. Finally deleting the old version.
![](images/Pasted%20image%2020230910170434.png)
The traffic split is always governed by number of pods in each deployment. We cannot say route just 1% of traffic to the canary deployment. Thats why service mesh like Istio comes with better control. With Istio, you can define exact % of traffic to be distributed between each deployment and it is not dependent on the number of pods in the deployment.
![](images/Pasted%20image%2020230910173748.png)

---
# Helm

A package and release manager

```
helm install wordpress ...

helm upgrade wordpress ...

helm rollback wordpress ...

helm uninstall wordpress ...
```

---
# Helm Concepts

Helm chart have following:
![](images/Pasted%20image%2020230910175700.png)

```
# Search for helm charts
# hub - artifactory hub
helm search hub wordpress

# To search for charts in other repo, first add repo to your local helm setup
helm repo add bitnami https://charts.bitnami.com/bitnami

# To search in bitnami repo
helm search repo wordpress

# To get list of repos
helm repo list

# To run helm chart
helm install [release-name][chart-name]

# Example
# Below command will run release of multiple helm harts of wordpress.
helm install release-1 bitnami/wordpress
helm install release-2 bitnami/wordpress

# To list installed packages
helm list

# To uninstall packages
helm uninstall my-release

# If we have to download it and not run the helm chart
helm pull --untar bitnami/wordpress

# After updating values, install the above downloaded helm chart
helm install release-4 ./wordpress
```


