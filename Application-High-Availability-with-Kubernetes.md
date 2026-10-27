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



















