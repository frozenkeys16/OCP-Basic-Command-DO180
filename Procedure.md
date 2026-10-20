<h1>Flow of Procedure of using</h1>

```
oc create deployment db-pod --port 3306 \
--image registry.lab.example.com:8443/rhel10/mariadb-118

```
Expose the db-pod deployment to create a service.



```
oc expose deployment/db-pod

```


---
<img width="1580" height="842" alt="image" src="https://github.com/user-attachments/assets/63632b4c-84bd-4a62-ad9e-876fb8bc533c" />

---


---

**First Proceudure:**

1. Log in to the OpenShift cluster.

```
oc login -u developer -p developer \
https://api.lab.example.com:6443

```

---

2. Set the deploy-services project as the active project.

```
oc project deploy-services
```

3. Use the registry.lab.example.com:8443/rhel10/mariadb-118 container image to create a MariaDB deployment named db-pod. Add the missing environment variables for the pod to run.

3.1 Create the db-pod deployment.


```
oc create deployment db-pod --port 3306 \
--image registry.lab.example.com:8443/rhel10/mariadb-118

```


3.2 Add the environment variables.

```
oc set env deployment/db-pod \
MYSQL_USER=user1 \
MYSQL_PASSWORD=mypa55w0rd \
MYSQL_DATABASE=items

```

Confirm that the db-pod is running. The pod name might differ in your cluster.

```
oc get pods
```

---


4. Expose the db-pod deployment to create a ClusterIP service.

4.1 View the deployment for the pod.

```
oc get deployment
```

4.2 Expose the db-pod deployment to create a service.

```
oc expose deployment/db-pod

```

5. Validate the service. Verify that the service selector matches the pod label. Then, confirm that the db-pod service endpoint matches the pod IP address.

5.1 Identify the selector for the db-pod service. Use the oc get service command with the -o wide option.

```
oc get service db-pod -o wide

```
The selector shows an app=db-pod key:value pair.


5.2 Capture the pod name in the PODNAME variable, and print the variable value.


```
PODNAME=$(oc get pods \
-o jsonpath='{.items[0].metadata.name}') && echo $PODNAME

```



















