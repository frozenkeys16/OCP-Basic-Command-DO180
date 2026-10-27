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



