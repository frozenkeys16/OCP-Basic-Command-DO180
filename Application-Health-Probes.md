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























