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




