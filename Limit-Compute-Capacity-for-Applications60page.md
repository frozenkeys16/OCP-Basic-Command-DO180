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





























