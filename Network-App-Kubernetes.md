<h3>Deploy Managed and Networked Applications on Kubernetes</h3>

**Login**
1. Login to OCP

```
oc login -u developer -p developer https://api.lab.example.com:6443

```

1.2 Change to the database-applications project.

```
oc project database-applications

```

2. Create a MySQL database deployment named mysql-app by using the registry.lab.example.com:8443/redhattraining/mysql-app:v1 image, and identify the root cause of the failure.


2.1 

```
Create the MySQL database deployment.

```

2.2 

```
Verify the deployment status. The pod name might differ in your output.

```

2.3 Identify the root cause of the deployment failure.


```
oc logs mysql-app-75dfd58f99-5xfqc

```

3. Configure the environment variables for the mysql-app deployment by using the following information:

```
**Field**          **Value**
MYSQL_USER          redhat
MYSQL_PASSWORD      redhat123
MYSQL_DATABASE      world_x
```
---

Then, execute the following command in the mysql-app deployment pod to load the world_x database:

```
/bin/bash -c "mysql -uredhat -predhat123 </tmp/world_x.sql"

```

3.1 Update the environment variables for the mysql-app deployment.

```
[student@workstation ~]$ oc set env deployment/mysql-app \
MYSQL_USER=redhat MYSQL_PASSWORD=redhat123 MYSQL_DATABASE=world_x

```

3.2 Verify that the mysql-app application pod is in the RUNNING state. The pod name might differ in your output.

```
[student@workstation ~]$ oc get pods

```

3.3 Load the world_x database.

```

[student@workstation ~]$ oc exec -it mysql-app-57c44f646-5qt2k \
-- /bin/bash -c "mysql -uredhat -predhat123 </tmp/world_x.sql"

```

3.4 Confirm that you can access the MySQL database.

```
[student@workstation ~]$ oc rsh mysql-app-57c44f646-5qt2k

```

```
sh-4.4$ mysql -uredhat -predhat123 world_x

```

3.5 Exit the MySQL database, and then exit the container.

```
exit

and

exit

```
---





