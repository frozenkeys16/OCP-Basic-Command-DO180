<h3>Guided Exercise: Container Image Identity and Tags</h3>


**Instructions**

1. Log in to the OpenShift cluster as the developer user with developer as the password. Use the updates-ids project.

1.1 Log in to the OpenShift cluster.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443

```

1.2 Set the updates-ids project that the lab start command created as the active project.

```
[student@workstation ~]$ oc project updates-ids

```

2. Inspect the two versions of the registry.lab.example.com:8443/ubi8/httpd-24 image from the classroom container registry. The classroom setup copied that image from the Red Hat Ecosystem Catalog. The original image is registry.access.redhat.com/ubi8/httpd-24.

2.1 Use the oc image info command to inspect the image version that the 1-209 tag references. Notice the unique SHA ID that identifies the image version.

---

**Note**
To improve readability, the instructions truncate the SHA-256 strings.

On your system, the commands return the full SHA-256 strings. Also, you must type the full SHA-256 string, to provide such a parameter to a command.

---

```
[student@workstation ~]$ oc image info \
registry.lab.example.com:8443/ubi8/httpd-24:1-209 | head

```

2.2 Inspect the image version that the 1-215 tag references. Notice that the SHA ID, or digest, differs from the preceding image version.

```
[student@workstation ~]$ oc image info \
registry.lab.example.com:8443/ubi8/httpd-24:1-215 | head

```
2.3 Use the skopeo login command to log in to the registry as the developer user with developer as the password.

```
[student@workstation ~]$ skopeo login registry.lab.example.com:8443 -u developer
Password: developer

```

2.4 Use the skopeo inspect command to inspect the 1-215 image tag. The output format differs from the oc image info command, although both commands report similar data.

```
[student@workstation ~]$ skopeo inspect \
docker://registry.lab.example.com:8443/ubi8/httpd-24:1-215 | head

```
The skopeo inspect command also shows other existing image tags.


3. Deploy an application from the image version that the 1-209 tag references.

3.1 Use the oc create deployment command to deploy the application. Set the name of the deployment to httpd1.

```
[student@workstation ~]$ oc create deployment httpd1 \
--image registry.lab.example.com:8443/ubi8/httpd-24:1-209

```

3.2 Wait for the pod to start, and then retrieve the name of the cluster node that runs it. You might have to rerun the command several times for the pod to report the Running status. The name of the pod might differ on your cluster.

```
[student@workstation ~]$ oc get pods -o wide

```
3.3 Retrieve the name of the container that is running inside the pod. The crictl ps command that you run in a following step takes the container name as an argument.

```
[student@workstation ~]$ oc get deployment httpd1 -o wide

```

4. Access the cluster node and retrieve the image that the container is using.

4.1 Log in to the OpenShift cluster as the admin user with redhatocp as the password.

```
[student@workstation ~]$ oc login -u admin -p redhatocp \
https://api.lab.example.com:6443

```

4.2 Use the oc debug node command to access the cluster node.

```
[student@workstation ~]$ oc debug node/master01

```

4.3 In the remote shell, run the chroot /﻿host command to use host binaries.

```
sh-5.1# chroot /host

```
4.4 Use the crictl ps command to confirm that the httpd-24 container is running. Add the -o yaml option to display the container details in YAML format.

```
sh-5.1# crictl ps --name httpd-24 -o yaml
containers:
- annotations:
...output omitted...
  image:
    annotations: {}
    image: registry.lab.example.com:8443/ubi8/httpd-24@sha256:b1e3...f876
...output omitted...
  imageRef: registry.lab.example.com:8443/ubi8/httpd-24@sha256:b1e3...f876
  labels:
...output omitted...
  state: CONTAINER_RUNNING

```
Notice that the command refers to the image by its SHA ID, and not by the tag that you specified when you created the deployment resource.

4.5 Use the crictl images command to list the locally available images on the node. The registry.lab.example.com:8443/ubi8/httpd-24:1-209 image is in that list, because the local container engine pulled it when you deployed the httpd1 application.

---

**Note**
The IMAGE ID column displays the local image identifier that the container engine assigns to the image. This identifier is not related to the SHA image ID that the container registry assigned to the image.

Most crictl commands, such as crictl images or crictl rmi, accept a local image identifier instead of the full image name. For example, you can run the crictl images 8ee59251acc93 command as a short version of the crictl images registry.lab.example.com:8443/ubi8/httpd-24:1-209 command.

---

```
sh-5.1# crictl images
IMAGE                                           TAG     IMAGE ID       SIZE
...output omitted...
quay.io/openshift-release-dev/ocp-release       <none>  d29424caa281a  517MB
quay.io/openshift-release-dev/ocp-v4.0-art-dev  <none>  42a50bb73e2fb  588MB
quay.io/openshift-release-dev/ocp-v4.0-art-dev  <none>  df18dddad77e7  510MB
quay.io/openshift-release-dev/ocp-v4.0-art-dev  <none>  388587ea693e1  447MB
...output omitted...
registry.lab.example.com:8443/ubi8/httpd-24     1-209   8ee59251acc93  461MB
...output omitted...

```

4.6 The crictl images command does not display the SHA image IDs by default. Rerun the command and add the --digests option to display the SHA IDs. Also, add the local image ID to the command to limit the output to the registry.lab.example.com:8443/ubi8/httpd-24:1-209 image.

The command reports only the first characters of the SHA image ID. These characters match the SHA ID of the image that the httpd-24 container is using. Therefore, the httpd-24 container is using the expected image.

```
sh-5.1# crictl images --digests 8ee59251acc93

```
4.7 Disconnect from the cluster node by exiting the chroot environment and the remote shell.

```
sh-5.1# exit
exit
sh-5.1# exit
exit

```

---





















