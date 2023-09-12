# Labels, Selectors and Annotations

```
kubectl get pods --selector app=App1
# OR
kubectl get pods -l app=App1

kubectl get po -l key1=value1,key2=value2
```

Labels and selectors are used to group and select objects. Annotations are used to record other details for informatory purpose.

``` yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: myapp
    function: front-end
  annotations:
    buildVersion: 1.34
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: front-end
    spec:
      containers:
      - name: simple-webapp
        image: simple-webapp
```

---
# Rolling Updates & Rollbacks in Deployments

## Rollout and Versioning in a Deployment

* When you first create a deployment, it triggers a rollout.
* A new rollout creates a new deployment revision. Lets say revision 1
* When the image is upgraded and a new deployment revision is created. Lets say revision 2
* This helps us to keep track the changes made to our deployment and enables us to rollback to previous version if necessary.

```
# To get the rollout status
kubectl rollout status deployment/myapp-deployment

# To get history of our deployments
kubectl rollout history deployment/myapp-deployment
```

## Deployment Strategy

* Recreate (Explained in Section 9 - Part 2)
* Rolling Update (Explained in Section 9 - Part 2) - This is default deployment strategy.

* For example, if we want to update the image of deployment, we can update in definition file.
* Other way is to use `kubectl set image deployment/myapp-deployment nginx-container=nginx:1.9.1`
* Note that, this image is not updated in definition file and it has to be updated manually.

Difference id deployment strategy can be seen:
![](images/Pasted%20image%2020230911212726.png)

## Upgrade
How does deployment perform the upgrade?

When you update the deployment, new replicaset is created. One pod is added in new RS, while one pod is deleted from old RS, and further for all the pods. This can be seen if we try to get the RS
![](images/Pasted%20image%2020230911212954.png)

## Rollback
When you request rollback to previous version, new pods will be deleted and old pods will be back
```
kubectl rollout undo deployment/myapp-deployment
```

![](images/Pasted%20image%2020230911213153.png)

![](images/Pasted%20image%2020230911213244.png)

```
# To get the history of the rollout by revision number
kubectl rollout history deployment nginx --revision=1

# To record what was changed in the next revision
# This will save the command used during the update
kubectl set image deployment nginx nginx=nginx:1.17 --record
# OR
# Below command will record just the below command, not the actual thing saved in file.
kubectl edit deployments nginx --record

# Rolling back to a revision
kubectl rollout undo deployment nginx --to-revision=1
```

---
# Jobs

A job is used to perform set of task to completion.

``` yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ["expr", "3", "+", "2"]
      # By default pod will try to recreate once the job is complete.
      # So this is required so that no new pod is created.
      restartPolicy: Never
```

![](images/Pasted%20image%2020230911221427.png)

## Completions

``` yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  # By default pods will be created sequentially.
  # Next pod will wait for the current pod to complete.
  # If there are errors, it will try to re-create the same pod again
  completions: 3
  # This is so the job does not quit before it succeeds
  backoffLimit: 15
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ["expr", "3", "+", "2"]
      restartPolicy: Never
```

![](images/Pasted%20image%2020230911221603.png)

In below screenshot, docker image randomly gives error
![](images/Pasted%20image%2020230911221658.png)

## Parallelism

``` yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  # To set 3 pods in parallel
  parallelism: 3
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ["expr", "3", "+", "2"]
      restartPolicy: Never
```
![](images/Pasted%20image%2020230911221849.png)

---
# CronJobs

``` yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: reporting-cron-job
spec:
  schedule: "*/1 * * * *"
  
  jobTemplate:
    # This is the exact copy of spec we had for Job
    # Note that there are 3 specs now
    spec:
      completions: 3
      parallelism: 3
      template:
        spec:
          containers:
            - name: reporting-tool
              image: reporting-tool
           restartPolicy: Never
```