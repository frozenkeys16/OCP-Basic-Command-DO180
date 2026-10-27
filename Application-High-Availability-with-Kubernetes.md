<h3>Guided Exercise: Application High Availability with Kubernetes</h3>


**Instructions**

1. As the developer user, create a pod from a YAML manifest in the reliability-ha project.

1.1 Log in to the OpenShift cluster as the developer user with developer as the password.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443

```

1.2 Select the reliability-ha project.

```
[student@workstation ~]$ oc project reliability-ha

```

1.3 Go to the lab materials directory and view the contents of the pod definition. In particular, the manifest sets restartPolicy to Always.

```
[student@workstation ~]$ cd DO180/labs/reliability-ha
[student@workstation reliability-ha]$ cat long-load.yaml
apiVersion: v1
kind: Pod
metadata:
  name: long-load
spec:
  containers:
  - image: registry.lab.example.com:8443/redhattraining/long-load:v1
    imagePullPolicy: Always
    securityContext:
      allowPrivilegeEscalation: false
    name: long-load
  restartPolicy: Always

```

1.4 Create a pod by using the oc apply command.

```
[student@workstation reliability-ha]$ oc apply -f long-load.yaml

```

1.5 Send a request to the pod to confirm that it is running and responding.

```
[student@workstation reliability-ha]$ oc exec long-load -- \
curl -s localhost:3000/health

```

---

2. Trigger the pod to crash, and observe that the restartPolicy instructs the cluster to re-create the pod.

2.1 Observe that the pod is running and did not restart.

```
[student@workstation reliability-ha]$ oc get pods
NAME        READY   STATUS    RESTARTS   AGE
long-load   1/1     Running   0          1m

```

2.2 Send a request to the /destruct endpoint in the application. This request triggers the process to crash.

```
[student@workstation reliability-ha]$ oc exec long-load -- \
curl -s localhost:3000/destruct
command terminated with exit code 52

```

2.3 Observe that the pod is running and restarted one time.

```
[student@workstation reliability-ha]$ oc get pods
NAME        READY   STATUS    RESTARTS      AGE
long-load   1/1     Running   1 (34s ago)   4m16s

```

2.4 Delete the long-load pod.

```
[student@workstation reliability-ha]$ oc delete pod long-load

```

The cluster does not re-create the pod, because you manually created the pod outside a workload resource, such as a deployment.



3. Use a restart policy of Never to create the pod, and observe that the cluster does not re-create the pod on crashing.

3.1 Modify the long-load.yaml file so that the restartPolicy is set to Never.

```
apiVersion: v1
kind: Pod
metadata:
  name: long-load
spec:
...output omitted...
  restartPolicy: Never

```

3.2 Create the pod with the updated YAML file.

```
[student@workstation reliability-ha]$ oc apply -f long-load.yaml

```

3.3 Send a request to the pod to confirm that the pod is running and that the application is responding.

```
[student@workstation reliability-ha]$ oc exec long-load -- \
curl -s localhost:3000/health

```

3.4 Send a request to the /destruct endpoint in the application to crash it.

```
[student@workstation reliability-ha]$ oc exec long-load -- \
curl -s localhost:3000/destruct

```

3.5 Observe that the cluster does not restart the pod, and that the pod is in an error state.

```
[student@workstation reliability-ha]$ oc get pods
NAME        READY   STATUS   RESTARTS   AGE
long-load   0/1     Error    0          2m36s

```

3.6 Delete the long-load pod.

```
[student@workstation reliability-ha]$ oc delete pod long-load
```

4. Because the cluster does not know when the application is ready to receive requests, you must add a startup delay. A later exercise covers adding this capability by using probes.

4.1 Update the long-load.yaml file by adding a startup delay and by setting the restart policy to Always. Set the START_DELAY variable to 60,000 milliseconds (one minute) so that the file exactly matches the following excerpt:

```
...output omitted...
spec:
  containers:
  - image: registry.lab.example.com:8443/redhattraining/long-load:v1
    imagePullPolicy: Always
    securityContext:
      allowPrivilegeEscalation: false
    name: long-load
    env:
      - name: START_DELAY
        value: "60000"
  restartPolicy: Always
```
---

```
**Note**
Although numbers are a valid YAML type, you must pass environment variables as strings. YAML syntax is also indentation-sensitive.

For these reasons, ensure that your file appears exactly as in the preceding example.

```

4.2 Apply the YAML file to create the pod and proceed within one minute to the next step.

```
[student@workstation reliability-ha]$ oc apply -f long-load.yaml

```

4.3 Within one minute of pod creation, verify the status of the pod. The status shows as Running even though the application is not yet ready. Try to send a request to the application, and observe that it fails.

```
[student@workstation reliability-ha]$ oc get pods
NAME        READY   STATUS    RESTARTS   AGE
long-load   1/1     Running   0          16s
```

```
[student@workstation reliability-ha]$ oc get pods
NAME        READY   STATUS    RESTARTS   AGE
long-load   1/1     Running   0          16s

```

4.4 After waiting one minute for the application to start, send another request to the pod to confirm that it is running and responding.

```
[student@workstation reliability-ha]$ oc exec long-load -- \
curl -s localhost:3000/health

```

---

5. Use a deployment to scale up the number of deployed pods. Observe that OpenShift dispatches the client requests between the replicated pods, even if the application in one of the pods returns errors.

5.1 Review the long-load-deploy.yaml file, which defines a deployment, service, and route. The deployment creates three replicas of the application pod.

The deployment file sets a START_DELAY environment variable to 15,000 milliseconds (15 seconds) in each pod. The application responds that it is not ready until after the delay.


```
student@workstation reliability-ha]$ cat long-load-deploy.yaml
...output omitted...
    spec:
      containers:
      - image: registry.lab.example.com:8443/redhattraining/long-load:v1
        imagePullPolicy: Always
        name: long-load
        env:
        - name: START_DELAY
          value: "15000"
...output omitted...
```

5.2 Start the load test script, which sends a request to the /health API endpoint of the application every two seconds. Leave the script running in a visible terminal window.

```
[student@workstation reliability-ha]$ ./load-test.sh
<html>
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1">

```

5.3 In a new terminal window, apply the ~/DO180/labs/reliability-ha/long-load-deploy.yaml file.

```
[student@workstation ~]$ oc apply -f \
~/DO180/labs/reliability-ha/long-load-deploy.yaml

```
5.4 Watch the output of the load test script in the other terminal window as the pods and the application instances start. After a delay, the requests succeed.

```
app is still starting
...output omitted...
Ok
Ok
Ok

```

5.5 Go to the second terminal window and send a request to the /togglesick API endpoint to put one of the three pods into a broken state.

```
[student@workstation ~]$ curl \
long-load-reliability-ha.apps.lab.example.com/togglesick

```

5.6 Go to the first terminal window and notice the output of the load test script during the failure of some requests. Because the load balancer distributes requests across pods, the order of the output varies.

```

...output omitted...
Ok
app is unhealthy
app is unhealthy
Ok
Ok

```

Press Ctrl+C to end the load test script, and close the second terminal window.



5.7 Return to the /home/student/ directory.

```
[student@workstation reliability-ha]$ cd /home/student/

```

---

































