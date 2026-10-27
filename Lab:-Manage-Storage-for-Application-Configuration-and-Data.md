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























