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

6.3 Delete the db-pod-pvc PVC.

```
[student@workstation ~]$ oc delete pvc db-pod-pvc

```

7. Create a PVC for an application that requires a shared storage volume from the nfs-storage storage class.

7.1 Create a nfs-pvc.yaml file with the following content:

```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-storage

```

For convenience, the ~/DO180/solutions/storage-classes/nfs-pvc.yaml file contains the YAML manifest for the PVC.


7.2 Create the PVC by using the oc create -f command and the YAML manifest file.

```
[student@workstation ~]$ oc create -f nfs-pvc.yaml

```
7.3 Use the oc describe pvc command to view the details of the PVC resource.

```
[student@workstation ~]$ oc describe pvc nfs-pvc
Name:          nfs-pvc
Namespace:     storage-classes
StorageClass:  nfs-storage
Status:        Bound 1
Volume:        pvc-9f462124-d96e-43e5-96a7-f96dd9375579 2
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
               volume.kubernetes.io/storage-provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       <none> 3

```

The Status: Bound value and the Volume attribute assignment confirm that the volume binding mode of the storage class is set to Immediate.

The Used By: <none> attribute shows that no pod is using the PVC.

8. Use the registry.lab.example.com:8443/ubi9/httpd-24:1-321 container image to create a web application deployment named web-pod.

8.1 Create the web-pod deployment.

```
[student@workstation ~]$ oc create deployment web-pod \
--port 8080 \
--image registry.lab.example.com:8443/ubi9/httpd-24:1-321

```

8.2 Create a service for the web-pod application.

```
[student@workstation ~]$ oc expose deployment web-pod

```

8.3 Expose the service to create a route for the wep-pod application. Specify web-pod.apps.lab.example.com as the hostname.

```
[student@workstation ~]$ oc expose svc web-pod \
--hostname web-pod.apps.lab.example.com

```

8.4 View the route that is assigned to the web-pod application.

```
[student@workstation ~]$ oc get routes

```
8.5 Use the curl command to view the index page of the web-pod application.

```
[student@workstation ~]$ curl -s \
http://web-pod.apps.lab.example.com/ | head

```

9. Add the nfs-pvc PVC to the web-pod deployment.

9.1 Use the oc set volume command to add the PVC to the deployment. Specify the volume name as nfs-volume, and set the mount path to the /var/www/html directory.

```
[student@workstation ~]$ oc set volumes deployment/web-pod \
--add --name nfs-volume \
--claim-name nfs-pvc \
--mount-path /var/www/html

```

The volume mount-path is set to the /var/www/html directory. The server uses this path to serve HTML content.



10. Use the registry.lab.example.com:8443/redhattraining/do180-roster container image to create a custom web application deployment named app-pod.

10.1 Create the app-pod deployment and specify port 9090 as the target port.

```
[student@workstation ~]$ oc create deployment app-pod \
--port 9090 --image \
registry.lab.example.com:8443/redhattraining/do180-roster

```

10.2 Create a service for the app-pod application.

```
[student@workstation ~]$ oc expose deployment app-pod

```

10.3 Expose the service to create a route for the app-pod application. Use app-pod.apps.lab.example.com for the hostname.

```
[student@workstation ~]$ oc expose svc app-pod \
--hostname app-pod.apps.lab.example.com

```
---

11. Add the nfs-pvc PVC to the app-pod application deployment.

11.1 Use the oc set volume command to add the PVC to the deployment. Set the volume name to nfs-volume and set the mount path to the /var/tmp directory.

```
[student@workstation ~]$ oc set volumes deployment/app-pod \
--add --name nfs-volume \
--claim-name nfs-pvc \
--mount-path /var/tmp

```

At this point, the web-pod and the app-pod applications are sharing a PVC. Kubernetes cannot prevent data conflicts between the two applications. In this case, the app-pod application is a writer and the web-pod application is a reader, and thus they do not conflict. The application implementation, not Kubernetes, prevents data corruption from the two applications that use the same PVC. The RWO access mode does not protect data integrity; a single node can mount the volume as read/write, and pods that share the volume must exist on the same node.



12. Use the app-pod application to add content to the shared volume.
12. 1 Open a web browser and go to http://app-pod.apps.lab.example.com

---


<img width="1158" height="469" alt="image" src="https://github.com/user-attachments/assets/be9cf3cf-d1b2-415c-8a4a-16b183646c45" />

---

12.2 In the form, enter your information and click save. The application adds your information to the list after the form.

12.3 Click push to create the /var/tmp/People.html file on the shared volume.


13. Verify that the web-pod application displays the People.html file from the shared PVC. Open a browser tab and go to http://web-pod.apps.lab.example.com/People.html



