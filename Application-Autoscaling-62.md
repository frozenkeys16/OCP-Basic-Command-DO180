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



