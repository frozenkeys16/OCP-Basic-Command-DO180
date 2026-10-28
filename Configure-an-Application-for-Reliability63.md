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



3. When the longload application scales up, your customers complain that some requests fail. To replicate the issue, manually scale up the longload application to three replicas, and run the ~/DO180/labs/reliability-review/curl_loop.sh script at the same time.

The application takes seven seconds to initialize. The application exposes the /health API endpoint on HTTP port 3000. Configure the longload deployment to use this endpoint, to ensure that the application is ready before serving client requests.

3.1 Open a second terminal window and run the ~/DO180/labs/reliability-review/curl_loop.sh script.

```
[student@workstation ~]$ ~/DO180/labs/reliability-review/curl_loop.sh

```
Leave the script running and do not interrupt it.


3.2 In the first terminal window, scale up the longload application to three replicas.

```
[student@workstation ~]$ oc scale deployment/longload --replicas 3

```
3.3 Switch back to the second terminal, and watch the output of the curl_loop.sh script. Some requests fail because OpenShift sends requests to the new pods before the application is ready.

```
22 longload-5897c9558f-cx4gt: Ok
23 longload-5897c9558f-cx4gt: Ok
24 longload-5897c9558f-cx4gt: Ok
25 curl: (7) Failed to connect to master01.lab.example.com port 30372 after 4 ms: Could not connect to server
26 curl: (7) Failed to connect to master01.lab.example.com port 30372 after 5 ms: Could not connect to server
27 longload-5897c9558f-cx4gt: Ok
28 curl: (7) Failed to connect to master01.lab.example.com port 30372 after 4 ms: Could not connect to server
29 longload-5897c9558f-cx4gt: Ok
30 curl: (7) Failed to connect to master01.lab.example.com port 30372 after 4 ms: Could not connect to server
31 longload-5897c9558f-tpssf: app is still starting
32 longload-5897c9558f-kkvm5: app is still starting
33 longload-5897c9558f-cx4gt: Ok
34 longload-5897c9558f-tpssf: app is still starting
35 longload-5897c9558f-tpssf: app is still starting
36 longload-5897c9558f-tpssf: app is still starting
37 longload-5897c9558f-cx4gt: Ok
38 longload-5897c9558f-tpssf: app is still starting
39 longload-5897c9558f-cx4gt: Ok
40 longload-5897c9558f-cx4gt: Ok

```
Leave the script running and do not interrupt it.

3.4 In the first terminal window, add a readiness probe to the longload deployment.

```
[student@workstation ~]$ oc set probe deployment/longload --readiness \
--initial-delay-seconds 7 \
--get-url http://:3000/health

```
3.5 Scale down the longload application back to one pod.

```
[student@workstation ~]$ oc scale deployment/longload --replicas 1

```
If scaling down breaks the curl_loop.sh script, then press Ctrl+C to stop the script in the second terminal. Then, restart the script.

3.6 To test your work, scale up the application to three replicas again.

```
[student@workstation ~]$ oc scale deployment/longload --replicas 3

```

3.7 Watch the output of the curl_loop.sh script in the second terminal, and observe that no requests fail.

```
92 longload-7ddcc9b7fd-72dtm: Ok
93 longload-7ddcc9b7fd-72dtm: Ok
94 longload-7ddcc9b7fd-72dtm: Ok
95 longload-7ddcc9b7fd-qln95: Ok
96 longload-7ddcc9b7fd-wrxrb: Ok
97 longload-7ddcc9b7fd-qln95: Ok
98 longload-7ddcc9b7fd-wrxrb: Ok
99 longload-7ddcc9b7fd-72dtm: Ok

```
Press Ctrl+C to end the script. Do not close the second terminal window.


---

4. Configure the longload application so that it automatically scales up when the average memory usage is above 60% of the memory requests value, and scales down when the usage is below this percentage. The minimum number of replicas must be one, and the maximum must be three. The resource that you create for scaling the application must be named longload.

The lab command provides the ~/DO180/labs/reliability-review/hpa.yaml resource file as an example. Use the oc explain command to learn the valid parameters for the hpa.spec.metrics.resource.target attribute. Because the file is incomplete, you must update it first if you choose to use it.

To test your work, use the oc exec deploy/longload -- curl localhost:3000/leak command to send an HTTP request to the application /﻿leak API endpoint. Each request consumes an additional 480 MiB of memory. To free this memory, you can use the ~/DO180/labs/reliability-review/free.sh script.

4.1 Before you create the horizontal pod autoscaler resource, scale down the application to one pod.

```
[student@workstation ~]$ oc scale deployment/longload --replicas 1

```

4.2 Edit the ~/DO180/labs/reliability-review/hpa.yaml resource file. You can retrieve the parameters for the resource attribute by using the oc explain hpa.spec.metrics.resource and oc explain hpa.spec.metrics.resource.target commands.

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: longload
  labels:
    app: longload
spec:
  maxReplicas: 3
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: longload
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 60

```

4.3 Use the oc apply command to deploy the horizontal pod autoscaler.

```
[student@workstation ~]$ oc apply -f ~/DO180/labs/reliability-review/hpa.yaml

```
4.4 In the second terminal, run the watch command to monitor the oc get hpa longload command. Wait for the longload horizontal pod autoscaler to report usage in the TARGETS column. The percentages might be different in your cluster.

```
[student@workstation ~]$ watch oc get hpa longload

```

Leave the command running and do not interrupt it.

4.5 To test your work, run the oc exec deploy/longload — curl localhost:3000/leak command in the first terminal for the application to allocate 480 MiB of memory.

```
[student@workstation ~]$ oc exec deploy/longload -- curl -s localhost:3000/leak

```

4.6 In the second terminal, after a few seconds, observe that the oc get hpa longload command shows the memory increase. The horizontal pod autoscaler scales up the application to more than one replica. The percentages might be different in your cluster.

```
NAME       REFERENCE             TARGETS             MINPODS   MAXPODS   REPLICAS   AGE
longload   Deployment/longload   memory: 145%/60%    1         3         3          5m18s

```

4.7 To test your work, run the ~/DO180/labs/reliability-review/free.sh script in the first terminal for the application to release the memory. Because the service load balances requests across the replicas, run the script multiple times to ensure that the request reaches the pod that consumed the memory.

```
[student@workstation ~]$ for i in {1..5}; \
do ~/DO180/labs/reliability-review/free.sh; done

```
4.8 In the second terminal, after ten minutes, observe that the oc get hpa longload command shows the memory decrease. The horizontal pod autoscaler scales down the application to one replica. The percentages might be different in your cluster.


```
NAME       REFERENCE             TARGETS            MINPODS   MAXPODS   REPLICAS   AGE
longload   Deployment/longload   memory: 12%/60%    1         3         1          15m28s

```
Press Ctrl+C to end the watch command. Close the second terminal window.
















































