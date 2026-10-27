<h3>Guided Exercise: Reserving Compute Capacity for Applications</h3>



**Instructions**

1. As the admin user, deploy the long-load application by applying the long-load-deploy.yaml file in the reliability-requests project.

1.1 Log in to the OpenShift cluster as the admin user with redhatocp as the password.

```
[student@workstation ~]$ oc login -u admin -p redhatocp \
https://api.lab.example.com:6443

```
---

**Note**
In general, use accounts with the least required privileges to perform a task. In the classroom environment, this account is the developer user. However, you need cluster administrator privileges to view the cluster node metrics in this exercise.


1.2 View the allocated resources for the node to determine the current memory request usage.

```
[student@workstation ~]$ oc describe node master01
...output omitted...
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                2858m (51%)    10m (0%)
  memory             12201Mi (82%)  0 (0%)

```
The command output shows that the pods on the node requested a total of 12201 MiB of memory. That value might be slightly different on your system.

---
Important
Projects and objects from previous exercises can cause the memory usage from this exercise to mismatch the intended results. Delete any unrelated projects before continuing.

If you still experience issues, then re-create your classroom environment and try this exercise again.


---

1.3 Select the reliability-requests project.

```
[student@workstation ~]$ oc project reliability-requests

```

1.4 Go to the ~/DO180/labs/reliability-requests directory, and create a deployment, a service, and a route by applying the long-load-deploy.yaml file.

```
[student@workstation ~]$ cd DO180/labs/reliability-requests
[student@workstation reliability-requests]$ oc apply -f long-load-deploy.yaml

```

2. Add a resource request to the pod definition and scale the deployment beyond the cluster's capacity.

2.1 Modify the long-load-deploy.yaml file to add a memory resource request of 1 GiB for each application pod.

```
spec:
  ...output omitted...
  template:
  ...output omitted...
    spec:
      containers:
      - image: registry.lab.example.com:8443/redhattraining/long-load:v1
        resources:
          requests:
            memory: 1Gi

```

2.2 Apply the YAML file to modify the deployment with the resource request.

```
[student@workstation reliability-requests]$ oc apply -f long-load-deploy.yaml

```

2.3 Scale the deployment to have 10 replicas.

```
[student@workstation reliability-requests]$ oc scale deploy/long-load \
--replicas 10

```

2.4 Verify that the cluster cannot schedule all pods on the single node. The cluster cannot schedule pods with the Pending status.

```
[student@workstation reliability-requests]$ oc get pods
NAME                         READY   STATUS    RESTARTS   AGE
...output omitted...
long-load-86bb4b79f8-44zwd   0/1     Pending   0          58s

```

2.5 Retrieve the cluster event log to confirm that insufficient memory is the cause of the failed scheduling.

```
[student@workstation reliability-requests]$ oc get events \
--field-selector reason="FailedScheduling"

```

Alternatively, view the events for a pending pod to see the reason. In the following command, replace the pod name with one of the pending pods in your classroom.


```
[student@workstation reliability-requests]$ oc describe \
pod/long-load-86bb4b79f8-44zwd

```

2.6 Verify that the node's requested memory usage is high.

```
[student@workstation reliability-requests]$ oc describe node master01
...output omitted...
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests          Limits
  --------           --------          ------
  cpu                2858m (51%)       10m (0%)
  memory             14249Mi (95%)     0 (0%)


```
The command output shows that the pods from the long-load deployment requested most of the remaining memory from the node. However, not enough memory is available to accommodate the 10 replicas.


---


3. Reduce the requested memory per pod so that the replicas can run on the node.

3.1 Scale the deployment to zero replicas. Scaling down first prevents a stalled rolling update, because the existing pods that request 1Gi of memory cannot be replaced when the node has insufficient memory.

```
[student@workstation reliability-requests]$ oc scale deploy/long-load \
--replicas 0

```

3.2 Set the resource request to 150Mi.

```
[student@workstation reliability-requests]$ oc set resources deploy/long-load \
--requests memory=150Mi

```

3.3 Scale back the deployment to 10 replicas.

```
[student@workstation reliability-requests]$ oc scale deploy/long-load \
--replicas 10

```

3.4 Verify that all pods can start with the lowered memory request. Within one minute, the cluster marks the pods as Ready and places them in the Running state, with no pods in the Pending status.

```
[student@workstation reliability-requests]$ oc get pods
NAME                         READY   STATUS    RESTARTS   AGE
long-load-557b4d94f5-68hbb   1/1     Running   0          3m14s
long-load-557b4d94f5-bfk7c   1/1     Running   0          3m21s
long-load-557b4d94f5-bnpzh   1/1     Running   0          3m21s
long-load-557b4d94f5-chtv9   1/1     Running   0          3m21s
long-load-557b4d94f5-drg2p   1/1     Running   0          3m14s
long-load-557b4d94f5-hwsz6   1/1     Running   0          3m12s
long-load-557b4d94f5-k5vqj   1/1     Running   0          3m21s
long-load-557b4d94f5-lgstq   1/1     Running   0          3m21s
long-load-557b4d94f5-r8hq4   1/1     Running   0          3m21s
long-load-557b4d94f5-xrg7c   1/1     Running   0          3m21s

```

Observe that the memory usage of the node is lower.

```
[student@workstation reliability-requests]$ oc describe node master01
...output omitted...
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests           Limits
  --------           --------           ------
  cpu                2858m (51%)        10m (0%)
  memory             13701Mi (92%)      0 (0%)

```

3.5 Return to the /home/student/ directory.

```
[student@workstation reliability-requests]$ cd /home/student/

```

---






















