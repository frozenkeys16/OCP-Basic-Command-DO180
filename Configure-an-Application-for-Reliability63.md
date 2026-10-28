**Lab: Configure an Application for Reliability**

---

**Instructions**

In this exercise, you work in the reliability-review project that the lab command created. The command also deployed the longload application in the reliability-review project.

The API URL of your OpenShift cluster is https://api.lab.example.com:6443, and the oc command is already installed on your workstation machine.

1. Log in to the OpenShift cluster as the developer user with developer as the password, and change to the reliability-review project. You use this project for all your work in this exercise.

1.1 Log in to the OpenShift cluster.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443

````

1.2 Change to the reliability-review project.

```
[student@workstation ~]$ oc project reliability-review

```

2. The longload application in the reliability-review project fails to start. Diagnose and then fix the issue. The application needs 512 MiB of memory to work.

After you fix the issue, you can confirm that the application works by running the ~/DO180/labs/reliability-review/curl_loop.sh script that the lab command prepared. The script sends requests to the application in a loop. For each request, the script displays the pod name and the application status. Press Ctrl+C to end the script.


2.1 List the pods in the project. The pod is in the Pending status. The name of the pod is different in your cluster.

```
[student@workstation ~]$ oc get pods

```

2.2 Retrieve the events for the pod. No compute node has enough memory to accommodate the pod.

```
[student@workstation ~]$ oc describe pod longload-64bf8dd776-b6rkz

```

2.3 Verify that the longload deployment requests 8 GiB of memory by running the following command:

```
[student@workstation ~]$ oc get deployment longload -o \
jsonpath='{.spec.template.spec.containers[0].resources.requests.memory}{"\n"}'

```
2.4 Set the memory requests for the longload deployment to 512 MiB.

```
[student@workstation ~]$ oc set resources deployment/longload \
--requests memory=512Mi

```
2.5 Verify that the longload pod is running. The name of the pod is different in your cluster.

```
[student@workstation ~]$ oc get pods

```
2.6 Run the ~/DO180/labs/reliability-review/curl_loop.sh script to confirm that the application works. You might get the curl: (7) Failed to connect …​ message until the application is ready.

```
[student@workstation ~]$ ~/DO180/labs/reliability-review/curl_loop.sh

```
Press Ctrl+C to end the script after the application displays the Ok message.



























