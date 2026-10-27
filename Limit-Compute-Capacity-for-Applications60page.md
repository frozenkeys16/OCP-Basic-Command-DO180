<h3>Guided Exercise: Limit Compute Capacity for Applications</h3>

**Instructions**

1. Log in to the OpenShift cluster as the developer user with developer as the password. Use the reliability-limits project.
1.1 Log in to the OpenShift cluster as the developer user with developer as the password.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443

```
1.2 Use the reliability-limits project.

```
[student@workstation ~]$ oc project reliability-limits

```

2. Create the leakapp deployment from the ~/DO180/labs/reliability-limits/leakapp.yml file that the lab command prepared. The application has a bug, and leaks 1 MiB of memory every second.

2.1 Review the ~/DO180/labs/reliability-limits/leakapp.yml resource file. The memory limit is set to 35 MiB.

```
...output omitted...
        resources:
          requests:
            memory: 20Mi
          limits:
            memory: 35Mi

```

2.2 Use the oc apply command to create the application.

```
[student@workstation ~]$ oc apply -f \
~/DO180/labs/reliability-limits/leakapp.yml

```

2.3 Wait for the pod to start. You might have to rerun the command several times for the pod to report a Running status. The name of the pod on your system probably differs.

```
[student@workstation ~]$ oc get pods
NAME                      READY   STATUS    RESTARTS   AGE
leakapp-99bb64c8d-hk26k   1/1     Running   0          12s
```

2.4 Retrieve the container status to verify that OpenShift restarted the pod due to an Out-Of-Memory (OOM) event.

```
[student@workstation ~]$ oc get pods leakapp-99bb64c8d-hk26k \
-o jsonpath='{.status.containerStatuses[0].lastState}' | jq .
{
  "terminated": {
    "containerID": "cri-o://5800...1d04",
    "exitCode": 137,
    "finishedAt": "2026-07-16T09:04:47Z",
    "reason": "OOMKilled",
    "startedAt": "2026-07-16T09:04:17Z"
  }
}
```

---

3. Observe the status of the leakapp pod until the OOMKilled status is displayed.

3.1 Use the watch oc get pods command to observe the status of the leakapp pod.

```
[student@workstation ~]$ watch oc get pods

```

OpenShift restarts the pod several times because of the memory leak. Between each restart, OpenShift sets the pod status to OOMKilled and then CrashLoopBackOff, waits an increasing amount of time between retries, and restarts the pod. During this wait time, the application is not available to your customers.

Press Ctrl+C to quit the watch command.

4. Update the leakapp deployment to set the memory limit to 600 MiB.

4.1 Use the oc set resources command to set the new limit.

```
[student@workstation ~]$ oc set resources deployment/leakapp \
--limits memory=600Mi

```

4.2 Use the watch oc get pods command to verify the status of the leakapp pod. The pod no longer restarts every 30 seconds.

```
[student@workstation ~]$ watch oc get pods

```
Press Ctrl+C to quit the watch command.

---

5. Optional. Wait about 10 minutes from the creation time until the application reaches the out of memory error. After this period, OpenShift restarts the pod, because it reached the 600 MiB memory limit.

5.1 Run the watch command to monitor the oc adm top pods command.

```
[student@workstation ~]$ watch oc adm top pods

```
Leave the command running and do not interrupt it.

---

**Note**
You might see a message that metrics are not yet available. If so, wait some time and try again.

5.2 Open a second terminal window and use the watch command to monitor the oc get pods command.

```
[student@workstation ~]$ watch oc get pods

```
Watch the output of the oc adm top pods command in the second terminal. When the memory usage reaches 600 MiB, the OOM subsystem kills the process inside the container, and OpenShift restarts the pod.














































