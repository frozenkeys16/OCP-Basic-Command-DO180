<h3>Guided Exercise: Provision Persistent Data Volumes</h3>

**Instructions**

1. Log in to the OpenShift web console as the developer user.

1.1 Open a web browser and go to https://console-openshift-console.apps.lab.example.com

1.2 Click Red Hat Identity Management and log in as the developer user with developer as the password. If the Welcome to the new OpenShift experience! message is displayed, then click Skip tour.

2. Identify the default storage class by going to Storage → StorageClasses. The nfs-storage storage class has the Default label and uses the k8s-sigs.io/nfs-subdir-external-provisioner provisioner.

---
<img width="964" height="598" alt="image" src="https://github.com/user-attachments/assets/e48fa6b5-d8e7-47fe-8c02-f0957bb7ef8f" />
---

3. Use the registry.lab.example.com:8443/rhel10/mariadb-118 container image to create a MariaDB deployment named db-pod. Use the storage-volumes project that the lab command created. Add a service for the database.

3.1 Go to Workloads → Deployments and select the storage-volumes project.

---

<img width="966" height="581" alt="image" src="https://github.com/user-attachments/assets/b1683756-9596-4510-a3e4-c9e7ffbb43a9" />

---


3.2 Click Create Deployment.
3.3 Use db-pod for the deployment name and change the image name to registry.lab.example.com:8443/rhel10/mariadb-118.
3.4 Add the required environment variables for the database.

**Table 5.5. MariaDB Environment Variables**

```
**Name**                 **Value**
MYSQL_USER	               user1
MYSQL_PASSWORD	           redhat123
MYSQL_DATABASE	           items

```

---


3.5 In the Advanced options section, click Scaling and set the number of replicas to 1.
3.6 Click Create. Wait for the blue circle to indicate that a single pod is running.
3.7 Create a service for the database. Go to Networking → Services and click Create Service.
3.8 Update the resource file with the following values:

**Table 5.6. Service Fields**

```
**Field name**                   **Value**

Name	                            db-pod
Selector	                        app: db-pod
Port	                            3306
Target                            port	3306


```

---

<img width="1031" height="502" alt="image" src="https://github.com/user-attachments/assets/147ec156-dd25-42e5-a562-d7dda9ea7745" />

---

3.9 Click Create.

4. Add to the deployment a 1 GiB, ReadWriteOnce (RWO) PVC named db-pod-pvc. Set the /var/lib/mysql directory as the mount path.

4.1 Go to Workloads → Deployments.
4.2 Click the vertical ellipsis icon (⋮) in the row for the db-pod deployment, and select Add storage.
4.3 In the Add Storage form, click Create new claim.
4.4 Complete the form with the following values:

**Table 5.7. Add PVC Storage Fields**

```
**Field name**                     	**Value**
PersistentVolumeClaim name	        db-pod-pvc
Access mode	                        Single user (RWO)
Size                              	1 GiB
Volume mode	                        Filesystem
Mount path	                        /var/lib/mysql
```

----


4.5 Click Save.

4.6 On the Deployment details page, scroll to the Volumes section, and click the db-pod-pvc link to view the PVC details.

---

<img width="1097" height="385" alt="image" src="https://github.com/user-attachments/assets/2a4a8728-571b-4d1c-8f62-37778f2f96d8" />

---

4.7 To retrieve the name of the PV that is associated with the PVC, go to the YAML tab. The name of the PV is available in the .spec.volumeName field. The name starts with pvc- even though the resource is a PV.


---

<img width="1116" height="597" alt="image" src="https://github.com/user-attachments/assets/46320fb2-6eab-47b8-9fe5-3b24fec8a4a3" />


---

The name of the PV is different on your system.


**5. Verify that configuring storage changes the deployment resource.**

5.1 Go to Workloads → Deployments, click the db-pod deployment name, and go to the YAML tab.
5.2 Verify that configuring storage for the deployment adds the volumes and volumeMounts fields.


```

kind: Deployment
apiVersion: apps/v1
...output omitted...
      volumes:
        - name: db-pod-pvc
          persistentVolumeClaim:
            claimName: db-pod-pvc
...output omitted...
        volumeMounts:
          - name: db-pod-pvc
            mountPath: /var/lib/mysql
...output omitted...

```


6. Use a configuration map resource to add initialization data to the database.

6.1 Review the ~/DO180/labs/storage-volumes/configmap/init-db.sql script that initializes the database. You do not have to change its contents.

```
DROP TABLE IF EXISTS `Item`;
CREATE TABLE `Item` (`id` BIGINT not null auto_increment primary key, `description` VARCHAR(100), `done` BOOLEAN);
INSERT INTO `Item` (`id`,`description`,`done`) VALUES (1,'Pick up newspaper', 0);
INSERT INTO `Item` (`id`,`description`,`done`) VALUES (2,'Buy groceries', 1);


```

6.2 Log in to the RHOCP cluster as the developer user with developer as the password.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443


```

6.3 Set the storage-volumes project as the active project.

```
[student@workstation ~]$ oc project storage-volumes

```

6.4 Use the contents of the init-db.sql file to create a configuration map named init-db-cm.

```
[student@workstation ~]$ oc create configmap init-db-cm \
--from-file ~/DO180/labs/storage-volumes/configmap/init-db.sql

```

6.5 Add the init-db-cm configuration map as a volume named init-db-volume to the deployment. Specify the volume type as configmap, and set the /var/db/config directory as the mount path.

```
[student@workstation ~]$ oc set volumes deployment/db-pod \
--add --name init-db-volume --type configmap \
--configmap-name init-db-cm --mount-path /var/db/config

```

6.6 Start a remote shell session inside the container.

```
[student@workstation ~]$ oc rsh deployment/db-pod

```

6.7 Use the mariadb client to execute the database script in the /var/db/config/init-db volume.

```
sh-5.2$ mariadb -uuser1 -predhat123 items < /var/db/config/init-db.sql


```

6.8 Execute a query to verify the database contents.


```
sh-5.2$ mariadb -uuser1 -predhat123 items -e 'select * from Item;'
+----+-------------------+------+
| id | description       | done |
+----+-------------------+------+
|  1 | Pick up newspaper |    0 |
|  2 | Buy groceries     |    1 |
+----+-------------------+------+
sh-5.2$


```















