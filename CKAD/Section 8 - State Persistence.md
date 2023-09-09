# Volumes
Docker containers are meant to be transient in nature - means they are meant to only last for short period of time. Same is with data - data is destroyed along with container.

To persist data, we attack volume to the container. So if we container is destroyed, data persists.

## Volumes and Mounts

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh", "-c"]
    # This is generating random number between 0 and 100
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    # Mounting the volume
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
```

![](images/Pasted%20image%2020230909175625.png)

So number. out in pod is `/opt/number.out`, in host it is at `/data/number.out`

## Volume Types
It is not recommended to use Directory mount on multi-node clusters as data may not be same across multiple nodes unless data replication is enabled.
![](images/Pasted%20image%2020230909180031.png)

Kubernetes offers multiple volume types like AWS EBS, Azure Disk, etc. So data from all nodes can be stored at same place.

Example of AWS EBS:
``` yaml
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

---
# Persistent Volume

Earlier we were configuring storage for the volumes in pod definition file. But in large environment, lot of users have to deploy multiple pods along with its storage. Whatever storage solution we use, users have to configure storage in the pods. Everytime a change has been made, it has to be updated in all of the pods.

How to configure it centrally, such that an admin can configure large pool of storage and then users carve out pieces from it when required?

![](images/Pasted%20image%2020230909180821.png)
So admin can create PVs and each pod can claim using PVC

``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  # Below hostPath should not be used as we read in volumes (in case pods are deployed in multiple pods)
  # hostPath:
  #   path: /tmp/data
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

---
# Persistent Volume Claim

PV and PVC are 2 separate objects in K8s. Admin creates PV and users create PVC to use the storage.

Once the PVC are created K8s bind the PV to claims based on the request and properties set on the volume.

Every PVC is bound to single PV. During the binding process, K8s tries to find the PV that has sufficient capacity as requested by the claim and any other request properties
* Sufficient Capacity
* Access Modes
* Volume Modes
* Storage Class
* etc

If there are multiple possible matches for a single claim and you would like to use particular volume, you can still use labels and selectors to bind to the right volume.
![](images/Pasted%20image%2020230909182049.png)

A smaller claim can bound to large volume if all the other criteria matches and there are no better options. There is a 1 to 1 mapping between PV and PVC, so no other claims can utilise the remaining capacity in the volume.
![](images/Pasted%20image%2020230909182300.png)

If no volumes are available, PVC will remain in pending state until newer volumes are made available to the cluster. One new volume are available, it will be automatically bound to the newly available PV.

![](images/Pasted%20image%2020230909182621.png)
As it can be seen, criteria is matching  between PV and PVC. PVC needs 500Mi storage while PV has 1Gi, it will bound to the claim because there are no other PV.

What if you delete the PVC, what will happen to PV? You can choose what happens to the volume. By default it is set to retain - means PV will be there until manually deleted by admin (but it will be unavailable for other claims)
``` yaml
persistentVOlumeReclaimPolicy: Retain # Other option is to Delete or Recycle
```
Once it is set to `Delete`, PV will be deleted once PVC is deleted, thus freeing up storage on end storage device.
With `Recycle`, data in data volume is removed before making it available to other claims.

## Using PVCs in Pods
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```
Same is true for Replicasets or Deployments. Add this section to the pod template section of a Deployment on Replicaset.

> `emptyDir` volume type is used to store data in a Pod only as long as that Pod is running on that node. When the pod is deleted the files are to be deleted as well.

> If you try to delete a PVC which is attached to a pod, it will remain in terminating state but will not terminate until pod is deleted

---
# Storage Classes (Not in Curriculum)

Used to dynamically create EBS volumes or disks in GCP when creating PV. Otherwise, disk had to be created before creating PV.

---
# Stateful Sets (Not in Curriculum)

Consider example of master-slave architecture of MySQL:
* Setup master first and then slaves
* Clone data from the master to slave-1
* Enable continous replication from master to slave-1
* Wait for slave-1 to be ready
* Clone data from slave-1 to slave-2
* Enable continous replication from master to slave-2
* Configure master address on slave

So these applications has to start sequentially. Below is the scenario in non-Kubenertes world, like VMs, servers:
![](images/Pasted%20image%2020230909220748.png)

Coming back to Kubernetes world:
* In deployments, we can't guarantee the sequence of pods to start, it will all come together, so above kind of setup cannot be done.
	* We want to dedicate a hostname for master. We can't rely on IP address as they may change as pod restarts.
	* Host names also wont work as pods comes with random names once restarted.

Statefulsets are similar to deployments. Here pods are created in sequential order. After the first pod is created, it should be up and in running state, before next pod is deployed.

* This helps master can be deployed first.
* Statefulsets assign a unique original index to each pod - starting from 0 and increments by 1.
* Each pod comes with a unique name combined with index, so no more random names.
* So we can always a pod as master, for now lets say mysql-0 is master. So slave pods can be pointed to masters host which is mysql-0.
* If mysql-0 is destroyed and recreated, it will still come up with same name.
* Stateful sets maintains a sticky identity for each of their pods.
![](images/Pasted%20image%2020230909221742.png)

---
# Stateful Sets Introduction (Not in Curriculum)

Note that you might not need a stateful set, it depends on the type of application.

Similar to deployments, stateful sets can be scaled up and pods will be created one by one. If you scale down, pods will be deleted in the sequence such that pod which was created last.

Example, there is a stateful set with 3 replicas and it has mysql-1, mysql-2 and mysql-3. Once you scale up lets say mysql-4, mysql-5 is created. If you scale down, mysql-5 will be deleted first and then mysql-4.

``` yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  template:
    metadata;
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  # Name of the headless service
  serviceName: mysql-h
  # This is optional, if you want to use unique names but you don't have to worry about sequence of pod creation, this will be used. If this is removed, pods will be created in a sequence.
  podManagementPolicy: Parallel
```

# Headless Services (Not in Curriculum)

Headless services is similar to service except that it doesn't have its own IP (like clusterIP), doesn't do load balancing. It only helps in providing the domain name for the pod.

`<podname>.<headless-servicename>.<namespace>.svc.<cluster-domain>.example`

`mysql-0.mysql-h.default.svc.cluster.local`

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-h
spec:
  ports:
    - port: 3306
  selector:
    app: mysql
  # This makes the service headless
  clusterIP: None
```

In pod-definition, under spec, subdomain and hostname is required.
``` yaml
apiVersion:
kind: Pod
metadata:
  ...
spec:
  containers:
  - name: mysql
    image: mysql

  # This has to match the headless service name
  subdomain: mysql-h
  hostname: mysql-pod
```

For above: `mysql-pod.mysql-h.default.svc.cluster.local`

If above pod definition is added to a deployment along with subdomain and hostname, all the pods created by deployments will have same domain `mysql-pod.mysql-h.default.svc.cluster.local`.

In Statefulset, we don't need subdomain and hostname as created in one of the YAML file above.
So this will give unique domain names to the pod: `mysql-0.mysql-h.default.svc.cluster.local`, `mysql-1.mysql-h.default.svc.cluster.local`

---
# Storage in StatefulSets

If you define a PVC in the pod within deployment or stateful sets, all the pods will try to use same PVC, which is fine but is not supported by all storage classes.

Instead, lets say we want to create PV and PVC for each pod that is created. For this, we can use `volumeClaimTemplates` within Deployments and StatefulSets definition file which is same as PVC definition file (but within Deployments and StatefulSets definition file)