<h3>Guided Exercise: Manage Non-shared Storage with Stateful Sets</h3>


**Instructions**

1. Create a web server deployment named web-server. Use the registry.lab.example.com:8443/redhattraining/hello-world-nginx:latest container image.

1.1 Log in to the OpenShift cluster as the developer user with developer as the password.

```
[student@workstation]$ oc login -u developer -p developer \
https://api.lab.example.com:6443

```

1.2 Set the storage-statefulsets project as the active project.

```
[student@workstation ~]$ oc project storage-statefulsets

```

1.3 Create the web-server deployment.

```
[student@workstation ~]$ oc create deployment web-server \
--image registry.lab.example.com:8443/redhattraining/hello-world-nginx:latest

```

1.4 Verify the deployment status.

```
[student@workstation ~]$ oc get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-server   1/1     1            1           50s

```
---

2. Add the PVC named web-pv to the web-server deployment. Use the default storage class and the following information to create the persistent volume claim:


```
**Field**	                  **Value**
Name	                        web-pv
Type	                        persistentVolumeClaim
Claim mode	                  rwo
Claim size	                  5Gi
Mount path	                  /usr/share/nginx/html
Claim name	                  web-pv-claim

```

2.1 Use the oc set volumes command to create and add the web-pv persistent volume to the web-server deployment.

```
[student@workstation ~]$ oc set volumes deployment/web-server \
--add --name web-pv --type persistentVolumeClaim --claim-mode rwo \
--claim-size 5Gi --mount-path /usr/share/nginx/html --claim-name web-pv-claim

```

Because a storage class was not specified with the --claim-class option, the command uses the default storage class to create the persistent volume.

2.2 Verify the deployment pod status. Notice that a new pod is created and in the running status. The pod name is different in your cluster.

```
[student@workstation ~]$ oc get pods -l app=web-server
NAME                          READY   STATUS   RESTARTS  AGE
web-server-5fb94fd568-48n8t   1/1     Running  0         5s

```

2.3 Store the pod name in the POD_NAME environment variable, and display the pod name by running the following command. You use the POD_NAME environment variable in later steps.

```
[student@workstation ~]$ POD_NAME=$(oc get pods -l app=web-server \
-o jsonpath='{.items[0].metadata.name}') && echo $POD_NAME

```

2.4 Check that the web-server pod mounts the web-pv PVC.

```
[student@workstation ~]$ oc get pod $POD_NAME -o json | \
jq .spec.volumes[0].persistentVolumeClaim.claimName

```
2.5 Verify that the web-pv PVC shows the bound status.

```
[student@workstation ~]$ oc get pvc
NAME           STATUS   VOLUME     CAPACITY  ACCESS MODES  STORAGECLASS ...
web-pv-claim   Bound    pvc-42...  5Gi       RWO           nfs-storage  ...

```

The default storage class, nfs-storage, provisioned the persistent volume.

---

3. Add data to the web-pv PVC by using the exec command.

3.1 Use the exec command to add the web-server pod name to the /usr/share/nginx/html/index.html file on the web-server pod. You use the POD_NAME environment variable from a previous step.

```
[student@workstation ~]$ oc exec -it pod/$POD_NAME \
-- /bin/bash -c \
'echo "Hello, World from ${HOSTNAME}" > /usr/share/nginx/html/index.html'

```

3.2 Retrieve the content of the /usr/share/nginx/html/index.html file to confirm that the file includes the web-server pod name. You use the POD_NAME environment variable from a previous step.

```
[student@workstation ~]$ oc exec -it pod/$POD_NAME \
-- cat /usr/share/nginx/html/index.html

```

---

4. Scale the web-server deployment to two replicas and confirm that an additional pod is created.

4.1 Scale the web-server deployment to two replicas.

```
[student@workstation ~]$ oc scale deployment web-server --replicas 2

```
4.2 Verify that the cluster created another pod. The pod names are different in your cluster. You use the new pod name in a later step.


```
[student@workstation ~]$ oc get pods
NAME                          READY   STATUS    RESTARTS   AGE
web-server-5fb94fd568-48n8t   1/1     Running   0          10m
web-server-5fb94fd568-rqvpb   1/1     Running   0          6s

```

5. Retrieve the content of the /usr/share/nginx/html/index.html file on the web-server pods by using the oc exec command to verify that the file is the same in both pods.

5.1 Verify the content in the /usr/share/nginx/html/index.html file in the first pod by using the POD_NAME environment variable.

```
Verify the content in the /usr/share/nginx/html/index.html file in the first pod by using the POD_NAME environment variable.


```

5.2 Verify the content of the /usr/share/nginx/html/index.html file in the new pod.

```
[student@workstation ~]$ oc exec -it pod/web-server-5fb94fd568-rqvpb \
-- cat /usr/share/nginx/html/index.html

```

Notice that both files show the name of the first instance pod, because they share the persistent volume.


6. Create a database server by using a stateful set and a headless service.

6.1 Edit the ~/DO180/labs/storage-statefulsets/statefulset-db.yml file and modify the placeholder text to match the following content:

```
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: dbserver
spec:
  selector:
    matchLabels:
      app: database
  replicas: 2
  serviceName: mysql-svc
  template:
    metadata:
      labels:
        app: database
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: dbserver
        image: registry.lab.example.com:8443/redhattraining/mysql-app:v1
        ports:
        - name: database
          containerPort: 3306
        env:
        - name: MYSQL_USER
          value: "redhat"
        - name: MYSQL_PASSWORD
          value: "redhat123"
        - name: MYSQL_DATABASE
          value: "sakila"
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "lvms-vg1"
      resources:
        requests:
          storage: 1Gi

```

6.2 Edit the ~/DO180/labs/storage-statefulsets/service-db.yml file and modify the placeholder text to match the following content:

```
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-svc
  labels:
    app: database
spec:
  ports:
  - port: 3306
    name: mysql
  clusterIP: None
  selector:
    app: database

```

6.3 Create the mysql-svc headless service.

```
[student@workstation ~]$ oc create -f \
/home/student/DO180/labs/storage-statefulsets/service-db.yml

```
6.4 Create the dbserver stateful set.

```
[student@workstation ~]$ oc create -f \
/home/student/DO180/labs/storage-statefulsets/statefulset-db.yml

```
6.5 Run the watch command and wait until the two stateful set pods are running. The pods' IP addresses are different in your cluster.

```
[student@workstation ~]$ watch oc get pod \
-l app=database -o wide
NAME         READY   STATUS    ...   AGE   IP          ...
dbserver-0   1/1     Running   ...   10s   10.8.0.112  ...
dbserver-1   1/1     Running   ...   5s    10.8.0.113  ...

```
Press Ctrl+C to end the watch command after the pods display Running in the STATUS column.

6.6 List the mysql-svc service endpoints, which correspond to the pods' IP addresses from the previous substep.

```
[student@workstation ~]$ oc get endpointslices
NAME              ADDRESSTYPE   PORTS   ENDPOINTS               AGE
mysql-svc-m6677   IPv4          3306    10.8.0.112,10.8.0.113   2m

```
6.7 Connect to the dbserver-1 pod by using its unique DNS name.

```
[student@workstation ~]$ oc rsh dbserver-1 \
curl -v telnet://dbserver-1.mysql-svc:3306

```
The DNS name resolves the IP address that corresponds to the dbserver-1 pod from previous substeps.

7. Confirm that each instance from the dbserver stateful set has a persistent volume claim. Add data to each stateful set pod.

7.1 Verify that the two persistent volume claims display Bound in the STATUS column.

```
[student@workstation ~]$ oc get pvc -l app=database
NAME                  STATUS  ...  CAPACITY    ACCESS MODE  ...
data-dbserver-0       Bound   ...  1Gi         RWO          ...
data-dbserver-1       Bound   ...  1Gi         RWO          ...

```

7.2 Verify that the dbserver-0 pod mounts the data-dbserver-0 PVC.

```
[student@workstation ~]$ oc get pod dbserver-0 -o json | \
jq .spec.volumes[0].persistentVolumeClaim.claimName

```

7.3 Verify that the dbserver-1 pod mounts the data-dbserver-1 PVC.

```
[student@workstation ~]$ oc get pod dbserver-1 -o json | \
jq .spec.volumes[0].persistentVolumeClaim.claimName

```

7.4 Create a table named items within the sakila database in the dbserver-0 pod by running the following command. You can safely ignore the warning message about using a password on the CLI.

```
[student@workstation ~]$ oc exec -it pod/dbserver-0 -- /bin/bash -c \
"mysql -uredhat -predhat123 sakila -e 'create table items (count INT);'"

```

7.5 Create a table named inventory within the sakila database in the dbserver-1 pod by running the following command. You can safely ignore the warning message about using a password on the CLI.


```

[student@workstation ~]$ oc exec -it pod/dbserver-1 -- /bin/bash -c \
"mysql -uredhat -predhat123 sakila -e 'create table inventory (count INT);'"

```
8. No data replication is configured at the application level for the dbserver stateful set. Verify that each database pod contains unique data.

8.1 Confirm that the dbserver-0 pod has a database named sakila with one table named items.

```
[student@workstation ~]$ oc exec -it pod/dbserver-0 -- /bin/bash -c \
"mysql -uredhat -predhat123 sakila -e 'show tables;'"
mysql: [Warning] Using a password on the command line interface can be insecure.
+------------------+
| Tables_in_sakila |
+------------------+
| items            |

```

8.2 Confirm that the dbserver-1 pod has a database named sakila with one table named inventory.

```
[student@workstation ~]$ oc exec -it pod/dbserver-1 -- /bin/bash -c \
"mysql -uredhat -predhat123 sakila -e 'show tables;'"
mysql: [Warning] Using a password on the command line interface can be insecure.
+------------------+
| Tables_in_sakila |
+------------------+
| inventory        |
+------------------+

```

9. Delete a pod in the dbserver stateful set. A new stateful set pod is created with the same ordinal index, and the pod mounts its corresponding PVC. The data remains in the PVC and it is accessible from the pod.

9.1 Delete the dbserver-0 pod in the dbserver stateful set.

```
[student@workstation ~]$ oc delete pod dbserver-0

```












































