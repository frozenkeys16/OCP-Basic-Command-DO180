<h3>Guided Exercise: Update Application Image and Settings</h3>

**Instructions**

1. Log in to the OpenShift cluster as the developer user with developer as the password. Use the updates-rollout-db project.

1.1 Log in to the OpenShift cluster.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443

```

1.2 Set the updates-rollout-db project as the active project.

```
[student@workstation ~]$ oc project updates-rollout-db

```

2. Review the resources that the lab command created. Confirm that you can connect to the database. The MariaDB database uses ephemeral storage.

2.1 List the Deployment object and confirm that the pod is available. Retrieve the name of the container. You use that information when you update the container image in another step.

```
[student@workstation ~]$ oc get deployment -o wide
NAME   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS ...
mydb   1/1     1            1           17m   mariadb-118   ...

```
2.2 List the pods and confirm that the pod is running. The name of the pod on your system probably differs.

```
[student@workstation ~]$ oc get pod

```

2.3 Retrieve the name of the image that the pod is using. The pod is using the rhel10/mariadb-118 image with the 1783945307 tag. Use the pod name from the previous step.

```
[student@workstation ~]$ oc get pod mydb-5c79866d48-5xzkk \
-o jsonpath='{.spec.containers[0].image}{"\n"}'

```
The classroom setup copied that image from the Red Hat Ecosystem Catalog. The original image is registry.redhat.io/rhel10/mariadb-118.

2.4 Confirm that you can connect to the database system by listing the available databases. Run the mariadb command from inside the pod and connect as the operator1 user with test as the password. Use the pod name from the previous step.

```
[student@workstation ~]$ oc rsh mydb-5c79866d48-5xzkk \
mariadb --user=operator1 --password=test -e "SHOW DATABASES"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| quotes             |
+--------------------+

```

3. You must implement several updates to the Deployment object. Pause the deployment to prevent OpenShift from rolling out the application for each modification that you make. After you pause the deployment, change the password for the operator1 database user, update the container image, and then resume the deployment.

3.1 Pause the mydb deployment.

```
[student@workstation ~]$ oc rollout pause deployment/mydb

```
3.2 Change the password of the operator1 database user to redhat123. To change the password, update the MYSQL_PASSWORD environment variable in the pod template of the Deployment object.

```
[student@workstation ~]$ oc set env deployment/mydb MYSQL_PASSWORD=redhat123

```

3.3 Because the Deployment object is paused, confirm that the new password is not yet active. To do so, rerun the mariadb command by using the current password. The database connection succeeds.

```
[student@workstation ~]$ oc rsh mydb-5c79866d48-5xzkk \
mariadb --user=operator1 --password=test -e "SHOW DATABASES"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| quotes             |
+--------------------+

```

3.4 Update the MariaDB container image to the 1784149182 version.

```
[student@workstation ~]$ oc set image deployment/mydb \
mariadb-118=registry.lab.example.com:8443/rhel10/mariadb-118:1784149182
```

3.5 Because the Deployment object is paused, confirm that the pod still uses the 1783945307 image version.

```
[student@workstation ~]$ oc get pod mydb-5c79866d48-5xzkk \
-o jsonpath='{.spec.containers[0].image}{"\n"}'

```

3.6 Resume the mydb deployment.

```
[student@workstation ~]$ oc rollout resume deployment/mydb

```

3.7 Confirm that the new rollout completes by waiting for the new pod to be running. The name of the pod on your system probably differs.

```
[student@workstation ~]$ oc get pods
```

4. Verify that OpenShift applied all your modifications to the Deployment object.

4.1 Retrieve the name of the image that the new pod is using. In the following command, use the name of the new pod as a parameter to the oc get pod command. The pod is now using the 1784149182 image version.

```
[student@workstation ~]$ oc get pod mydb-dd5dcbddb-rmf85 \
-o jsonpath='{.spec.containers[0].image}{"\n"}'

```

4.2 Confirm that you can connect to the database system by using the new password, redhat123, for the operator1 database user.

```
[student@workstation ~]$ oc rsh mydb-dd5dcbddb-rmf85 \
mariadb --user=operator1 --password=redhat123 -e "SHOW DATABASES"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| quotes             |
+--------------------+

```

5. In the second part of the exercise, you perform a rolling update of a replicated web application. Use the updates-rollout-web project and review the resources that the lab command created.

5.1 Set the updates-rollout-web project as the active project.

```
[student@workstation ~]$ oc project updates-rollout-web

```
5.2 List the Deployment object and confirm that the pods are available. Retrieve the name of the containers. You use that information when you update the container image in another step.

```
[student@workstation ~]$ oc get deployment -o wide

```

5.3 List the ReplicaSet objects. Because OpenShift did not yet perform rolling updates, only one ReplicaSet object exists. The name of the ReplicaSet object on your system probably differs.

```
[student@workstation ~]$ oc get replicaset

```

5.4 Retrieve the name and version of the image that the ReplicaSet object uses to deploy the pods. The pods are using the redhattraining/versioned-hello image version v1.0.

```
[student@workstation ~]$ oc get replicaset version-7bfff6b5b4 \
-o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'

```




















