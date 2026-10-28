<h3>Guided Exercise: Application Autoscaling</h3>


**Instructions**

1. Log in to the OpenShift cluster as the developer user with developer as the password. Use the reliability-autoscaling project.

1.1 Log in to the OpenShift cluster.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443

```
1.2 Set the reliability-autoscaling project as the active project.

```
[student@workstation ~]$ oc project reliability-autoscaling

```
2. Create the loadtest deployment, service, and route. The deployment uses the registry.lab.example.com:8443/redhattraining/loadtest:v1.0 container image, which provides a web application. The web application exposes an API endpoint that creates a CPU-intensive task when queried.

```
apiVersion: v1
kind: List
metadata: {}
items:
  - apiVersion: apps/v1
    kind: Deployment
...output omitted...
        spec:
          containers:
          - image: registry.lab.example.com:8443/redhattraining/loadtest:v1.0
            name: loadtest
            readinessProbe:
              failureThreshold: 3
              httpGet:
                path: /api/loadtest/v1/healthz
                port: 8080
                scheme: HTTP
              periodSeconds: 10
              successThreshold: 1
              timeoutSeconds: 1

  - apiVersion: v1
    kind: Service
...output omitted...

  - apiVersion: route.openshift.io/v1
    kind: Route

```

2.2 Use the oc apply command to create the application.

```
[student@workstation ~]$ oc apply -f \
~/DO180/labs/reliability-autoscaling/loadtest.yml

```

2.3 Run the watch oc get pods command and wait for the pod to report the Running status. The name of the pod is different in your cluster.

```
[student@workstation ~]$ watch oc get pods

```
Press Ctrl+C to end the watch command.


3. Configure an HPA resource for the loadtest deployment. Set the minimum number of replicas to 2 and the maximum to 20. Set the average CPU usage to 50% of the CPU requests attribute.

The HPA does not work, because the loadtest deployment does not specify requests for CPU usage.

3.1 Use the oc autoscale command to create the HPA resource.

```
[student@workstation ~]$ oc autoscale deployment/loadtest --min 2 --max 20 \
--cpu 50%

```

3.2 Retrieve the status of the loadtest HPA resource. The unknown value in the TARGETS column indicates that OpenShift cannot compute the current CPU usage of the loadtest deployment. The deployment must include the CPU requests attribute for OpenShift to be able to compute the CPU usage.

```
[student@workstation ~]$ oc get hpa loadtest
NAME      REFERENCE            TARGETS              MINPODS  MAXPODS   REPLICAS   AGE
loadtest  Deployment/loadtest  cpu: <unknown>/50%   2        20        2          74s

```
3.3 Get more details about the resource status. You might have to rerun the command several times.

```
[student@workstation ~]$ oc describe hpa loadtest
Name:                                             loadtest
Namespace:                                        reliability-autoscaling
...output omitted...
Conditions:
  Type           Status  Reason                   Message
  ----           ------  ------                   -------
  AbleToScale    True    SucceededGetScale        the HPA controller was able to get the target's current scale
  ScalingActive  False   FailedGetResourceMetric  the HPA was unable to compute the replica count: failed to get cpu utilization: missing request for cpu in container loadtest of Pod loadtest-86d987b9bf-w66x9
Events:
  Type     Reason                    Age     From                        Message
  ----     ------                    ---     ----                        -------
...output omitted...
  Warning  FailedGetResourceMetric   2m24s   horizontal-pod-autoscaler   failed to get cpu utilization: missing request for cpu in container loadtest of Pod loadtest-86d987b9bf-w66x9
```

3.4 Delete the HPA resource.

```
[student@workstation ~]$ oc delete hpa loadtest

```

3.5 Delete the loadtest application.

```
[student@workstation ~]$ oc delete -f \
~/DO180/labs/reliability-autoscaling/loadtest.yml

```

---

4. Add a CPU resource section to the ~/DO180/labs/reliability-autoscaling/loadtest.yml file. Redeploy the application from the file.

4.1 Edit the ~/DO180/labs/reliability-autoscaling/loadtest.yml file, and configure the CPU limits and requests for the loadtest deployment. The pod needs 25 millicores to operate, and must not consume more that 100 millicores.

You can compare your work with the completed ~/DO180/solutions/reliability-autoscaling/loadtest.yml file that the lab command prepared.


```
...output omitted...
        spec:
          containers:
          - image: registry.lab.example.com:8443/redhattraining/loadtest:v1.0
            name: loadtest
            readinessProbe:
              failureThreshold: 3
              httpGet:
                path: /api/loadtest/v1/healthz
                port: 8080
                scheme: HTTP
              periodSeconds: 10
              successThreshold: 1
              timeoutSeconds: 1
            resources:
              requests:
                cpu: 25m
              limits:
                cpu: 100m

```

4.2 Use the oc apply -f command to deploy the application from the file.

```
[student@workstation ~]$ oc apply -f \
~/DO180/labs/reliability-autoscaling/loadtest.yml

```
4.3 Run the watch oc get pods command and wait until the pod is running. The name of the pod is different in your cluster.

```
[student@workstation ~]$ watch oc get pods

```
Press Ctrl+C to end the watch command after the pod displays Running in the STATUS column.

5. Manually scale the loadtest deployment by first increasing and then decreasing the number of running pods.

5.1 Scale up the loadtest deployment to five pods.

```
[student@workstation ~]$ oc scale deployment/loadtest --replicas 5

```
5.2 Run the watch oc get pods command and wait until the five pods are running. The names of the pods are different in your cluster.

```
[student@workstation ~]$ watch oc get pods
...output omitted...
NAME                        READY   STATUS    RESTARTS   AGE
loadtest-667bdcdc99-5fcvh   1/1     Running   0          43s
loadtest-667bdcdc99-dpspr   1/1     Running   0          42s
loadtest-667bdcdc99-hkssk   1/1     Running   0          43s
loadtest-667bdcdc99-vhc9x   1/1     Running   0          8m11s
loadtest-667bdcdc99-z5n9q   1/1     Running   0          43s

```

Press Ctrl+C to end the watch command after the pods display Running in the STATUS column.

5.3 Scale down the loadtest deployment back to one pod.

```
[student@workstation ~]$ oc scale deployment/loadtest --replicas 1

```


5.4 Run the watch oc get pods command and wait until only one pod is running. The name of the pod is different in your cluster.

```
[student@workstation ~]$ watch oc get pods

```

Press Ctrl+C to end the watch command after the pod displays Running in the STATUS column.

6. Configure an HPA resource for the loadtest deployment. Set the minimum number of replicas to 2 and the maximum to 20. Set the average CPU usage to 50% of the CPU request attribute.

6.1 Use the oc autoscale command to create the HPA resource.

```

[student@workstation ~]$ oc autoscale deployment/loadtest --min 2 --max 20 \
--cpu 50%

```

6.2 Run the watch command to monitor the oc get hpa loadtest command. Wait up to five minutes for the loadtest HPA to report usage in the TARGETS column.

Notice that the HPA scales up the deployment to two replicas, to conform with the minimum number of pods that you configured.

```
[student@workstation ~]$ watch oc get hpa loadtest

```

Leave the command running, and do not interrupt it.


7. Increase the CPU usage by sending requests to the loadtest application API.

7.1 Open a second terminal window and use the oc get route command to retrieve the URL of the application.

```
[student@workstation ~]$ oc get route loadtest

```

7.2 Send a request to the application API to simulate additional CPU pressure on the container. Do not wait for the curl command to complete, and continue with the exercise. After one minute, the command reports a timeout error that you can ignore.

```
[student@workstation ~]$ curl \
loadtest-reliability-autoscaling.apps.lab.example.com/api/loadtest/v1/cpu/1

```

7.3 Watch the output of the oc get hpa loadtest command. After a minute, the HPA detects an increase in the CPU usage and deploys additional pods.

---

**Note**
The increased activity of the application does not immediately trigger the autoscaler. Wait a few moments if you do not see any changes to the number of replicas.

You might need to run the curl command multiple times before the application uses enough CPU to trigger the autoscaler.

---

The CPU usage and the number of replicas on your system probably differ.


```
[student@workstation ~]$ watch oc get hpa loadtest

```

7.4 After five minutes, the application in the pod stops consuming CPU resources. The CPU load decreases.

---
**Note**
Although the HPA resource can be quick to scale up, it is slower to scale down.

---

7.5 Press Ctrl+C to quit the watch command from the first terminal window, and close the second terminal window.

---









