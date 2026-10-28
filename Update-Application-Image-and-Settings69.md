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
5.5 Confirm that the version deployment includes a readiness probe. The probe performs an HTTP GET request on port 8080.

```
[student@workstation ~]$ oc get deployment version \
-o jsonpath='{.spec.template.spec.containers[0].readinessProbe}' | jq .

```

6. To watch the rolling update that you cause in a following step, open a new terminal window and then run the ~/DO180/labs/updates-rollout/curl_loop.sh script that the lab command prepared. The script sends web requests to the application in a loop.

6.1 Open a new terminal and run the /home/student/DO180/labs/updates-rollout/curl_loop.sh script. Leave the script running to monitor the rolling update.

```
[student@workstation ~]$ /home/student/DO180/labs/updates-rollout/curl_loop.sh

```

7. Change the container image of the version deployment. The new application version creates a web page with a different message.

7.1 Change to the first terminal window, and use the oc set image command to update the deployment.

```
[student@workstation ~]$ oc set image deployment/version \
versioned-hello=registry.lab.example.com:8443/redhattraining/versioned-hello:v1.1

```

7.2 Changing the image caused a rolling update. Watch the output of the curl_loop.sh script in the second terminal. It takes a few seconds for the update to start. If the output of the curl_loop.sh is not updating, then press Ctrl+C to stop the script in the second terminal. Then, restart the script.

Before the update, only pods that run the v1.0 version of the application reply. During the rolling updates, both old and new pods are responding. After the update, only pods that run the v1.1 version of the application reply. The following output probably differs on your system.

```

Hi!
Hi!
Hi!
Hi!
Hi! v1.1
Hi! v1.1
Hi!
Hi! v1.1
Hi!
Hi! v1.1
Hi! v1.1
Hi! v1.1
Hi! v1.1

```

Do not stop the script.

8. Confirm that the rollout process is successful. List the ReplicaSet objects and verify that the new object uses the new image version.

8.1 Use the oc rollout status command to confirm that the rollout process is successful.

```
[student@workstation ~]$ oc rollout status deployment/version

```

8.2 List the ReplicaSet objects. The initial object scaled down to zero pods. The new ReplicaSet object scaled up to 10 pods. The names of the ReplicaSet objects on your system probably differ.

```
[student@workstation ~]$ oc get replicaset

```

8.3 Retrieve the name and version of the image that the new ReplicaSet object uses. This image provides the new version of the application.

```
[student@workstation ~]$ oc get replicaset version-b7fddfc8c \
-o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'

```

9. Roll back the version deployment.

9.1 Use the oc rollout undo command to roll back to the initial application version.

```
[student@workstation ~]$ oc rollout undo deployment/version

```

9.2 Watch the output of the curl_loop.sh script in the second terminal. After approximately one minute, the pods that run the v1.0 version of the application are responding again. The following output probably differs on your system.

```
Hi! v1.1
Hi! v1.1
Hi! v1.1
Hi! v1.1
Hi!
Hi! v1.1
Hi!
Hi! v1.1
Hi! v1.1
Hi!
Hi!
Hi!

```
Press Ctrl+C to quit the script. Close that second terminal when done.


9.3 List the ReplicaSet objects. The initial object scaled up to 10 pods. The object for the new application version scaled down to zero pods.


```
[student@workstation ~]$ oc get replicaset

```

---
























