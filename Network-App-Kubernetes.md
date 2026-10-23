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

4. Create a service for the mysql-app deployment by using the following information:

**Field**    **Value**
Name         mysql-service
Port         3306
Target port  3306


4.1 Expose the mysql-app deployment.

```
[student@workstation ~]$ oc expose deployment mysql-app --name mysql-service \
--port 3306 --target-port 3306

```

4.2 Verify the service configuration. The endpoint IP address might differ in your output.

```
[student@workstation ~]$ oc get services

[student@workstation ~]$ oc get endpointslices

```

5. Create a web application deployment named php-app by using the registry.lab.example.com:8443/redhattraining/php-webapp:v1 image.


5.1 Create the web application deployment.

```
[student@workstation ~]$ oc create deployment php-app \
--image registry.lab.example.com:8443/redhattraining/php-webapp:v1

```

5.2 Verify the deployment status. Verify that the php-app application pod is in the RUNNING state.

```
[student@workstation ~]$ oc get pods

[student@workstation ~]$ oc status

```

---

6. Create a service for the php-app deployment by using the following information:

**Field**        **Value**
Name              php-svc
Port              8080
Target port       8080

Then, create a route named phpapp to expose the web application to external access.

6.1 Expose the php-app deployment.

```
[student@workstation ~]$ oc expose deployment php-app --name php-svc \
--port 8080 --target-port 8080

```

6.2 Verify the service configuration. The endpoint IP address might differ in your output.

```
**[student@workstation ~]$ oc get services**
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
mysql-service   ClusterIP   172.30.146.213   <none>        3306/TCP   7m47s
php-svc         ClusterIP   172.30.228.80    <none>        8080/TCP   4m34s
**[student@workstation ~]$ oc get endpointslices**
NAME                    ADDRESSTYPE   PORTS   ENDPOINTS      AGE
mysql-service-j9r64     IPv4          3306    10.8.0.132     7m47s
php-svc-69b57           IPv4          8080    10.8.0.133     12s

```

6.3 Expose the php-svc service.

```
[student@workstation ~]$ oc expose service/php-svc --name phpapp

```

```
[student@workstation ~]$ oc get routes

```

---

7. Test the connectivity between the web application and the MySQL database. In a web browser, go to the phpapp-database-applications.apps.lab.example.com route, and verify that the application retrieves data from the MySQL database.

7.1 Go to the phpapp-database-applications.apps.lab.example.com route in the web browser.

---















