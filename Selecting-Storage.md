<h3>Guided Exercise: Selecting a Storage Class for an Application</h3>

**Instructions**

1. In a terminal window, log in to the RHOCP cluster as the developer user with developer as the password, and switch to the storage-classes project.

1.1 Log in to the OpenShift cluster.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443

```

1.2 Set the storage-classes project as the active project.

```
[student@workstation ~]$ oc project storage-classes

```

2. Examine the available storage classes on the cluster. Identify an appropriate storage class to use for a database application.

2.1 Use the oc get command to retrieve a list of storage classes in the cluster. You can use the storageclass short name, sc, in the command.


```
[student@workstation ~]$ oc get sc
NAME                   PROVISIONER   ...
lvms-vg1               topolvm.io    ...
nfs-storage (default)  k8s-sigs.io/nfs-subdir-external-provisioner ...

```

Because an administrator can change the default storage class, applications must specify a storage class that meets the application requirements.



2.2 Use the oc describe command to view the details of the lvms-vg1 storage class.

```
[student@workstation ~]$ oc describe sc lvms-vg1
Name:                  lvms-vg1
IsDefaultClass:        No
Annotations:           description=Provides RWO and RWOP Filesystem & Block volumes,storageclass.kubernetes.io/is-default-class=false
Provisioner:           topolvm.io
Parameters:            csi.storage.k8s.io/fstype=xfs,topolvm.io/device-class=vg1
AllowVolumeExpansion:  True
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>

```
The description annotation states that the storage class provides support for block volumes. For some applications, such as databases, block volumes can provide a performance advantage over file system volumes. In the lvms-vg1 storage class, the AllowVolumeExpansion field is set to True. With volume expansion, cluster users can edit their PVC objects and specify a new size for the PVC. Kubernetes then uses the storage back end to automatically expand the volume to the requested size. Kubernetes also expands the file system of pods that use the PVC. Enabling volume expansion can help to protect an application from failing due to the data growing too fast. With these features, the lvms-vg1 storage class is a good choice for the database application.

---

3. Use the registry.lab.example.com:8443/rhel10/mariadb-118 container image to create a MariaDB deployment named db-pod.

The MariaDB container image requires the MYSQL_USER, MYSQL_PASSWORD, and MYSQL_DATABASE environment variables to initialize the database. Configure the environment variables for the pod to run.

3.1 Create the db-pod deployment.

```
[student@workstation ~]$ oc create deployment db-pod \
--port 3306 \
--image registry.lab.example.com:8443/rhel10/mariadb-118
deployment.apps/db-pod created


```

3.2 Add the environment variables.

```
[student@workstation ~]$ oc set env deployment/db-pod \
MYSQL_USER=user1 \
MYSQL_PASSWORD=redhat123 \
MYSQL_DATABASE=items
deployment.apps/db-pod updated


```

3.3 Verify that the pod is running.

```
[student@workstation ~]$ oc get pods

```
3.4 Expose the db-pod deployment to create a service.

```
[student@workstation ~]$ oc expose deployment/db-pod

```

4. Add a 1 Gi, RWO PVC named db-pod-pvc to the deployment. Specify the volume name as lvm-storage, and set the /var/lib/mysql directory as the mount path. Use the lvms-vg1 storage class.

4.1 Use the oc set volume command to create a PVC for the deployment.

```
[student@workstation ~]$ oc set volumes deployment/db-pod \
--add --name lvm-storage --type pvc --claim-mode rwo \
--claim-size 1Gi --mount-path /var/lib/mysql \
--claim-class lvms-vg1 --claim-name db-pod-pvc

```

The claim-class option specifies a non-default storage class.


---

**Note**
The oc set volumes command does not have an option to set the volume mode property. Thus, the oc set volumes command results in using the default Filesystem volume mode. Use a declarative manifest to set the volume mode to Block. The use of declarative manifests is out of scope for this course.


---


4.2 Use the oc get pvc command to view the status of the PVC. Identify the name of the PV, and confirm that the PVC uses the lvms-vg1 non-default storage class.

```
[student@workstation ~]$ oc get pvc

```

4.3 Use the oc describe pvc command to inspect the details of the db-pod-pvc PVC.

```
[student@workstation ~]$ oc describe pvc db-pod-pvc
Name:          db-pod-pvc
Namespace:     storage-classes
StorageClass:  lvms-vg1
Status:        Bound
Volume:        pvc-72084a46-3f32-435e-987b-4ad3b9026021
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: topolvm.io
               volume.kubernetes.io/selected-node: master01
               volume.kubernetes.io/storage-provisioner: topolvm.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       db-pod-568888457d-qmxn9
Events:
...output omitted...

```
The Used By attribute confirms that the PVC is bound to a pod.


---
5. Connect to the database to verify that it is working.

```
[student@workstation ~]$ oc run query-db -it --rm \
--image registry.lab.example.com:8443/rhel10/mariadb-118 \
--restart Never --command \
-- /bin/bash -c "mariadb -uuser1 -predhat123 \
-h db-pod -P3306 items -e 'show databases;'"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| items              |
+--------------------+
pod "query-db" deleted from storage-classes namespace

```

6. Delete the db-pod deployment and the db-pod-pvc PVC.

6.1 Delete the db-pod deployment and service.

```
[student@workstation ~]$ oc delete deployment db-pod
deployment.apps "db-pod" deleted from storage-classes namespace
[student@workstation ~]$ oc delete service db-pod

```

6.2 Verify that the PVC still exists without the deployment.

```
[student@workstation ~]$ oc get pvc

```




