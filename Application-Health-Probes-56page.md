<h3>Guided Exercise: Application Health Probes</h3>

**Instructions**

1. As the developer user, deploy the long-load application in the reliability-probes project.

1.1 Log in to the OpenShift cluster as the developer user with developer as the password.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443

```
1.2 Ensure that the reliability-probes project is the active project.

```
[student@workstation ~]$ oc project reliability-probes

```

1.3 Apply the long-load-deploy.yaml file to create the pod. Move to the next step within one minute.

```
[student@workstation ~]$ oc apply -f \
~/DO180/labs/reliability-probes/long-load-deploy.yaml

```

---


1.4 Verify that the pods take several minutes to start by sending a request to a pod in the deployment.

```
[student@workstation ~]$ oc exec deploy/long-load -- \
curl -s localhost:3000/health


```

1.5 Observe that the pods are ready even though the application is not ready.

```
[student@workstation ~]$ oc get pods
NAME                         READY   STATUS    RESTARTS   AGE
long-load-5dbd695f8b-7hjjv   1/1     Running   0          37s
long-load-5dbd695f8b-q86kc   1/1     Running   0          37s
long-load-5dbd695f8b-xmtw9   1/1     Running   0          37s


```


2. Add a startup probe to the pods so that the cluster knows when the pods are ready.

2.1 Modify the deployment in the ~/DO180/labs/reliability-probes/long-load-deploy.yaml file by defining a startup probe. The probe runs every three seconds and triggers a pod as failed after 30 failed attempts. The file should match the following excerpt:

```
apiVersion: apps/v1
kind: Deployment
...output omitted...
spec:
  ...output omitted...
  template:
    ...output omitted...
    spec:
      containers:
      - image: registry.lab.example.com:8443/redhattraining/long-load:v1
        imagePullPolicy: Always
        name: long-load
        startupProbe:
          failureThreshold: 30
          periodSeconds: 3
          httpGet:
            path: /health
            port: 3000
        env:
```
2.2 Apply the updated long-load-deploy.yaml file.

```
[student@workstation ~] oc apply -f \
~/DO180/labs/reliability-probes/long-load-deploy.yaml

```

2.3 Run the watch command to observe that OpenShift replaces the pods one at a time during the rolling update. The new pod shows as ready only when the startup probe succeeds, and OpenShift replaces the next pod only when the current one is ready.

```
[student@workstation ~]$ watch oc get pods
Every 2.0s: oc get pods                workstation: Thu Jul 16 08:19:22 2026

NAME                         READY   STATUS    RESTARTS   AGE
long-load-596f8d547f-jnz99   0/1     Pending   0          12s
long-load-5dbd695f8b-7hjjv   1/1     Running   0          75s
long-load-5dbd695f8b-q86kc   1/1     Running   0          75s
long-load-5dbd695f8b-xmtw9   1/1     Running   0          75s
```

2.4 Wait for the three pods to reach the ready state. Press Ctrl+C to stop the watch command.

```
[student@workstation ~]$ watch oc get pods
Every 2.0s: oc get pods                workstation: Thu Jul 16 08:23:06 2026

NAME                         READY   STATUS    RESTARTS   AGE
long-load-596f8d547f-5mv4v   1/1     Running   0          109s
long-load-596f8d547f-jgzfb   1/1     Running   0          2m52s
long-load-596f8d547f-jnz99   1/1     Running   0          3m56s

```

3. Add a liveness probe so that the cluster restarts broken instances of the application.

3.1 Start the load test script.

```
[student@workstation ~]$ ~/DO180/labs/reliability-probes/load-test.sh

```
Keep the script running in a visible window.

3.2 In a new terminal window, use the /togglesick endpoint to make one of the pods unhealthy.

```
[student@workstation ~]$ oc exec \
deploy/long-load -- curl -s localhost:3000/togglesick

```

The load test window begins to show app is unhealthy. Because only one pod is unhealthy, the remaining pods still respond with Ok.

3.3 Update the deployment in the ~/DO180/labs/reliability-probes/long-load-deploy.yaml file to add a liveness probe. The probe runs every three seconds and triggers the pod as failed after three failed attempts. Modify the spec.template.spec.containers object in the file to match the following excerpt.

```
apiVersion: apps/v1
kind: Deployment
...output omitted...
spec:
  ...output omitted...
  template:
    ...output omitted...
    spec:
      containers:
      - image: registry.lab.example.com:8443/redhattraining/long-load:v1
        ...output omitted...
        startupProbe:
          failureThreshold: 30
          periodSeconds: 3
          httpGet:
            path: /health
            port: 3000
        livenessProbe:
          failureThreshold: 3
          periodSeconds: 3
          httpGet:
            path: /health
            port: 3000
        env:
```

3.4 Scale down the deployment to zero replicas.

```
[student@workstation ~]$ oc scale deploy/long-load --replicas 0

```

The load test script shows that the application is not available.


---

**Note**
Red Hat recommends scaling down an application to zero replicas before making multiple changes to a deployment, such as adding health probes, resource requests, or environment variables. Scaling down to zero replicas prevents creating any pods, and enables pods to terminate gracefully after finishing all current requests.

For multiple deployment updates, scaling down to zero prevents a noisy event log from Kubernetes responding to every change, and conserves cluster resources.


3.5 Apply the updated long-load-deploy.yaml file to update the deployment, which triggers the deployment to re-create its pods.

```
[student@workstation ~]$ oc apply -f \
~/DO180/labs/reliability-probes/long-load-deploy.yaml

```

3.6 Wait for the three new pods to reach the ready state. Press Ctrl+c to stop the watch command.

```
[student@workstation ~]$ watch oc get pods
Every 2.0s: oc get pods                workstation: Thu Jul 16 08:28:41 2026

NAME                        READY   STATUS    RESTARTS   AGE
long-load-c6bbcdf66-gnkp2   1/1     Running   0          91s
long-load-c6bbcdf66-pwf6g   1/1     Running   0          91s
long-load-c6bbcdf66-sbwqg   1/1     Running   0          91s

```

3.7 Wait for the load test window to show Ok for all responses, and then toggle one of the pods to be unhealthy.

```
[student@workstation ~]$ oc exec \
deploy/long-load -- curl -s localhost:3000/togglesick

```
The load test window might show app is unhealthy several times before the cluster runs the liveness probe and excludes the pod from the service resource.


3.8 Observe that the cluster restarts the unhealthy pod after the liveness probe fails. After the pod is restarted, the load test window shows only Ok.

```
[student@workstation ~]$ watch oc get pods
Every 2.0s: oc get pods                workstation: Thu Jul 16 08:29:38 2026

NAME                        READY   STATUS    RESTARTS      AGE
long-load-c6bbcdf66-gnkp2   0/1     Running   1 (12s ago)   2m28s
long-load-c6bbcdf66-pwf6g   1/1     Running   0             2m28s
long-load-c6bbcdf66-sbwqg   1/1     Running   0             2m28s

```

4. Add a readiness probe so that traffic goes only to pods that are ready and healthy.

4.1 Scale down the deployment to zero replicas.

```
[student@workstation ~]$ oc scale deploy/long-load --replicas 0
```

4.2 Use the oc set probe command to add the readiness probe.

```
[student@workstation ~]$ oc set probe deploy/long-load \
--readiness --failure-threshold 1 --period-seconds 3 \
--get-url http://:3000/health

```
4.3 Scale up the deployment to three replicas.

```
[student@workstation ~]$ oc scale deploy/long-load --replicas 3

```

4.4 Observe the status of the pods by using the watch command. Leave this command running in a visible window.

```
[student@workstation ~]$ watch oc get pods
Every 2.0s: oc get pods                workstation: Thu Jul 16 08:31:10 2026

NAME                         READY   STATUS    RESTARTS   AGE
long-load-77795fbd45-4t4tb   0/1     Running   0          23s
long-load-77795fbd45-56tw5   0/1     Running   0          23s
long-load-77795fbd45-nns4g   0/1     Running   0          23s

```

4.5 Wait for the pods to show as ready. Then, open a new terminal window, and make one of the pods unhealthy for five seconds by using the /hiccup endpoint.

```
[student@workstation ~]$ oc exec \
deploy/long-load -- curl -s localhost:3000/hiccup?time=5

```

The pod status window shows that one of the pods is no longer ready.

```
[student@workstation ~]$ watch oc get pods
Every 2.0s: oc get pods                workstation: Thu Jul 16 08:32:41 2026

NAME                         READY   STATUS    RESTARTS   AGE
long-load-77795fbd45-4t4tb   0/1     Running   0          115s
long-load-77795fbd45-56tw5   1/1     Running   0          115s
long-load-77795fbd45-nns4g   1/1     Running   0          115s

```
After five seconds, the pod is healthy again and shows as ready.

The load test window might show app is unhealthy one time before the pod is set as not ready. After the cluster determines that the pod is no longer ready, it stops sending traffic to the pod until either the pod recovers or the liveness probe fails. Because the pod is sick for only five seconds, it is enough time for the readiness probe to fail, but not the liveness probe.

---

**Note**
Optionally, repeat this step and observe as the temporarily sick pod's status changes.


4.6 Stop the load test and status commands by pressing Ctrl+C in their respective windows, and close all the terminal windows.

---




























