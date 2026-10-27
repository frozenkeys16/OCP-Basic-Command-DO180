<h3>Lab: Manage Storage for Application Configuration and Data</h3>


Instructions

In this lab, you deploy a web application and its database that share database credentials from a secret. You also deploy a file-sharing application that runs with multiple replicas and shares its storage volume with a file-uploader application. The file-sharing and file-uploader applications take configuration files from a configuration map and must use NFS file storage for shareability. The database must use local storage for increased performance.

The API URL of your OpenShift cluster is https://api.lab.example.com:6443, and the oc command is already installed on your workstation machine.

1. Log in to the OpenShift cluster as the developer user with developer as the password, and change to the storage-review project. You use this project for all your work in this exercise.

1.1 Log in to the OpenShift cluster.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443

```

1.2 Change to the storage-review project.

```
[student@workstation ~]$ oc project storage-review
```

2. Create a secret named world-cred that contains the following data:

```
Field	        Value
user	        redhat
password    	redhat123
database	    world_x

```

2.1 Create a secret that contains the database credentials.

```
[student@workstation ~]$ oc create secret generic world-cred \
--from-literal user=redhat \
--from-literal password=redhat123 \
--from-literal database=world_x

```

2.2 Confirm the creation of the secret.

```
[student@workstation ~]$ oc get secrets world-cred
NAME         TYPE     DATA   AGE
world-cred   Opaque   3      2m34s

```

3. Create a configuration map named dbfiles by using the ~/DO180/labs/storage-review/insertdata.sql file.

3.1 Create a configuration map named dbfiles by using the insertdata.sql file in the ~/DO180/labs/storage-review directory.

```
[student@workstation ~]$ oc create configmap dbfiles \
--from-file ~/DO180/labs/storage-review/insertdata.sql

```

3.2 Verify the creation of the configuration map.

```
[student@workstation ~]$ oc get configmaps
NAME     DATA  AGE
dbfiles  1     11s

```

4. Create a database server deployment named dbserver by using the registry.lab.example.com:8443/redhattraining/mysql-app:v1 container image. Then, set the missing environment variables by using the world-cred secret. Use the MYSQL_ prefix for the variables.

4.1 Create the database server deployment.

```
[student@workstation ~]$ oc create deployment dbserver \
--image \
registry.lab.example.com:8443/redhattraining/mysql-app:v1

```

4.2 Set the missing environment variables.

```
[student@workstation ~]$ oc set env deployment/dbserver \
--from secret/world-cred --prefix MYSQL_

```

4.3 Verify that the dbserver pod is in the RUNNING state. The pod name might differ in your output.

```
[student@workstation ~]$ oc get pods

```


---

5. Add a volume to the dbserver deployment by using the following information:

```

Field	                Value
name	                dbserver-lvm
type	                persistentVolumeClaim
claim-mode	          rwo
claim-size	          1Gi
mount-path	          /var/lib/mysql
claim-class	          lvms-vg1
claim-name	          dbserver-lvm-pvc


```

6. Create a service for the dbserver deployment by using the following information:

```
**Field**         **Value**
name	              mysql-service
port	              3306
target-port	        3306

```

6.1 Expose the dbserver deployment.

```
[student@workstation ~]$ oc expose deployment dbserver \
--name mysql-service --port 3306 --target-port 3306

```

6.2 Verify the service and endpoint configuration. The endpoint IP address might differ in your output.

```
[student@workstation ~]$ oc get services
NAME           TYPE       CLUSTER-IP     EXTERNAL-IP PORT(S)
mysql-service  ClusterIP  172.30.240.100 <none>      3306/TCP
[student@workstation ~]$ oc get endpointslices
NAME                    ADDRESSTYPE   PORTS   ENDPOINTS      AGE
mysql-service-j9r64     IPv4          3306    10.8.0.156     19s

```

7. Create a web application deployment named file-sharing by using the registry.lab.example.com:8443/redhattraining/php-webapp-mysql:v1 container image. Scale the deployment to two replicas. Then, expose the deployment by using the following information:


```

**Field**	            **Value**
name	                file-sharing
port	                8080
target-port	          8080

```

Create a route named file-sharing to expose the file-sharing web application to external access. Access the file-sharing route in a web browser to test the connection between the web application and the database server.

7.1 Create a web application deployment.

```
[student@workstation ~]$ oc create deployment file-sharing \
--image registry.lab.example.com:8443/redhattraining/php-webapp-mysql:v1

```

7.2 Verify the deployment status. Verify that the file-sharing application pod is in the RUNNING state. The pod names might differ on your system.


```
[student@workstation ~]$ oc get pods

```

7.3 Scale the deployment to two replicas.

```
[student@workstation ~]$ oc scale deployment file-sharing \
--replicas 2

```

7.4 Verify the replica status and retrieve the pod name. The pod names might differ on your system.

```
[student@workstation ~]$ oc get pods

```

7.5 Expose the file-sharing deployment.

```
[student@workstation ~]$ oc expose deployment file-sharing \
--name file-sharing --port 8080 --target-port 8080

```

7.6 Verify the service configuration. The endpoint IP address might differ in your output.

```
[student@workstation ~]$ oc get services
NAME          TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)
file-sharing  ClusterIP  172.30.139.210 <none>       8080/TCP
mysql-service ClusterIP  172.30.240.100 <none>       3306/TCP
[student@workstation ~]$ oc get endpointslices
NAME                    ADDRESSTYPE   PORTS   ENDPOINTS               AGE
file-sharing-69b57      IPv4          8080    10.8.0.157,10.8.0.158   12s
mysql-service-j9r64     IPv4          3306    10.8.0.156              5m23s

```

7.7 Expose the file-sharing service.

```
[student@workstation ~]$ oc expose service/file-sharing
route.route.openshift.io/file-sharing exposed
[student@workstation ~]$ oc get routes
NAME         HOST/PORT                                         ...
file-sharing file-sharing-storage-review.apps.lab.example.com  ...

```

7.8 Test the connectivity between the web application and the database server. In a web browser, go to http://file-sharing-storage-review.apps.lab.example.com, and verify that the Connected successfully message is displayed.


---

8. Mount the dbfiles configuration map to the file-sharing deployment as a volume named config-map-pvc. Set the mount path to the /home/database-files directory. Then, verify the content of the insertdata.sql file.


8.1 Mount the dbfiles configuration map to the file-sharing deployment.

```
[student@workstation ~]$ oc set volume deployment/file-sharing \
--add --name config-map-pvc --type configmap \
--configmap-name dbfiles \
--mount-path /home/database-files

```

8.2 Verify the deployment status. It might take some time for the pods to be ready.

```
[student@workstation ~]$ oc get pods
NAME                            READY   STATUS   ...
dbserver-5bc6bd5d7b-7z7lv       1/1     Running  ...
file-sharing-7f77855b7f-949lg   1/1     Running  ...
file-sharing-7f77855b7f-9zvwq   1/1     Running  ...

```

8.3 Verify the content of the /home/database-files/insertdata.sql file by using one of the file-sharing pods.

```
[student@workstation ~]$ oc exec -it \
pod/file-sharing-7f77855b7f-949lg -- \
head /home/database-files/insertdata.sql
-- MySQL dump 10.13  Distrib 8.0.19, for osx10.14 (x86_64)
--
-- Host: 127.0.0.1    Database: world_x
-- ------------------------------------------------------
-- Server version 8.0.19-debug
...output omitted...

```

9. Add a shared volume to the file-sharing deployment. Use the following information to create the volume:









































