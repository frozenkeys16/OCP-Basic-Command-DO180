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

<h2<PALIWANAG sa mga Command ng  oc expose deployment, service</h2>

**Maraming nalilito sa Deployment at Service dahil parehong tungkol sila sa application, pero magkaiba ang trabaho nila.**

**Una: Ano ang nangyayari sa oc create deployment?**

Halimbawa:

```
oc create deployment db-pod --image=mysql

```

Gagawa ito ng:

```

Deployment
   |
ReplicaSet
   |
Pod

```

Maaari mong tingnan:

```
oc get deployment
oc get rs
oc get pods

```
Output:

```
NAME
db-pod

```

Ang Deployment ang bahala sa:

- Pag-create ng Pods
- Self-healing
- Scaling
- Rolling updates

Pero tandaan:

✅ May Pod na
❌ Wala pang Service

---


**Bakit kailangan pa ng Service?**

Problema:

Kapag namatay ang Pod:

```
db-pod-12345
IP: 10.128.0.5

```
at ni-recreate ni Deployment:

```
db-pod-67890
IP: 10.128.0.8

```

Nagbago ang IP.

Kung application mo ay kumokonekta sa:

```
10.128.0.5

```

babagsak ito pagkatapos ma-recreate ang Pod.

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

**Query the label on the pod.**

```
oc get pod $PODNAME --show-labels

```
The label list includes the app=db-pod key-value pair, which matches the service selector.



5.3 Retrieve the endpoints for the db-pod service by listing the endpointslices resources.

```
oc get endpointslices

```
The endpoint IP address is different in your cluster.

5.4 Verify that the service endpoint matches the pod IP address. Use the oc get pods command with the -o wide option.

```

oc get pods -o wide

```

The service endpoint resolves to the pod's IP address.


6. Delete and re-create the db-pod deployment. Confirm that the db-pod service endpoint automatically resolves to the new pod's IP address.


6.1 Delete the db-pod deployment.

```
 oc delete deployment.apps/db-pod

```

6.2 Verify that the service still exists without the deployment.

```
oc get service

```
6.3 Confirm that the endpointslices resource does not set any endpoints.

```
oc get endpointslices
```

6.4 Re-create the db-pod deployment.

```
oc create deployment db-pod --port 3306 \
--image registry.lab.example.com:8443/rhel10/mariadb-118

```
6.5 Add the environment variables.
```
 oc set env deployment/db-pod \
MYSQL_USER=user1 \
MYSQL_PASSWORD=mypa55w0rd \
MYSQL_DATABASE=items
```

6.6 Confirm that the new pod has the app=db-pod selector. The pod name and the IP address might differ in your cluster.

```
oc get pods --selector app=db-pod -o wide

```

6.7 Confirm that the endpointslices resource includes the new pod.

```
 oc get endpointslices

```
7. Create a pod to identify the available DNS name assignments for the service.

7.1 Create a pod named shell for troubleshooting. Use the oc run command with the registry.lab.example.com:8443/openshift4/network-tools-rhel8 image. The command might take a few seconds to return the prompt.

```
oc run shell -it \
--image registry.lab.example.com:8443/openshift4/network-tools-rhel8

```
7.2 View the /etc/resolv.conf file from the shell pod to identify the cluster-domain name.

```
cat /etc/resolv.conf
```
The container uses the search directive values as suffixes for DNS queries. The cluster-domain name follows svc.

7.3 Test the available DNS names with the nc and echo commands.

```
nc -z db-pod.deploy-services 3306 && \
echo "Connection success to db-pod.deploy-services:3306" || \
echo "Connection failed"

```
7.4 Exit the interactive session.

```
exit

```
7.5 Delete the shell pod.

```
oc delete pod shell
```

8. Test pod communications across namespaces with a new project.

8.1 Create a second namespace with the oc new-project command.

```
oc new-project deploy-services-2

```

8.2 Test DNS name access from a pod in the new namespace to the db-pod service in the deploy-services namespace.

```
oc run shell -it --rm \
--image registry.lab.example.com:8443/openshift4/network-tools-rhel8 \
--restart Never -- nc -z db-pod.deploy-services.svc.cluster.local 3306 && \
echo "Connection success to db-pod.deploy-services.svc.cluster.local:3306" || \
echo "Connection failed"

```

8.3 Return to the deploy-services project.

```
oc project deploy-services

```

9. Use a Kubernetes job to add initialization data to the database.

9.1 Create a job named mariadb-init by using the registry.lab.example.com:8443/redhattraining/do180-dbinit:v2 image. This image, which is based on mariadb-118, includes a script to add initial records.

```
oc create job mariadb-init \
--image registry.lab.example.com:8443/redhattraining/do180-dbinit:v2 \
-- /bin/bash -c "mariadb -uuser1 -pmypa55w0rd --protocol tcp \
-h db-pod -P3306 items </tmp/db-init.sql"

```
The -h option directs the command to the db-pod service short name. The -- option separates oc arguments from the pod command. The /tmp/db-init.sql file, which is included in the image, contains the following queries:



9.2 Confirm the mariadb-init job status, and wait for its completion.

```
oc get job

```
9.3 Verify that the mariadb-init pod shows Completed in the STATUS column.

```
 oc get pods
```
9.4 Delete the mariadb-init job.

```
oc delete job mariadb-init

```
9.5 Verify that the mariadb-init pod is deleted.

```
 oc get pods
```

10. Create a query-db pod to query the database service.

10.1 Create the query-db pod. Use the MariaDB client to query the db-pod service.

```
oc run query-db -it --rm \
--image registry.lab.example.com:8443/redhattraining/do180-dbinit:v2 \
--restart Never \
-- mariadb -uuser1 -pmypa55w0rd --protocol tcp \
-h db-pod -P3306 items -e 'select * from Item;'

```

11. Use pod-to-pod communication for troubleshooting.

11.1 Confirm the IP address of the MariaDB database pod. The IP address might differ in your cluster.

```
oc get pods -o wide

```

11.2 Capture the IP address in the POD_IP environment variable, and print the variable value.

11.2
```
POD_IP=$(oc get pod -l app=db-pod \
-o jsonpath='{.items[0].status.podIP}') && echo $POD_IP

```

11.3 Create a test pod named shell. Use the nc command to test the connection to the $POD_IP IP address and to port 3306.

```
oc run shell --env POD_IP=$POD_IP -it --rm \
--image registry.lab.example.com:8443/openshift4/network-tools-rhel8 \
--restart Never \
-- nc -z $POD_IP 3306 && echo "Connection success to $POD_IP:3306" || \
echo "Connection failed"

```
---

**Kaya ginawa ang Service**

Service = Permanent DNS Name + Load Balancer

Example:

```
Service Name: db-pod
ClusterIP: 172.30.108.92

```
Application:

```
mysql://db-pod:3306
```

Hindi na kailangang alamin ang Pod IP.

---

**Ano ang ginagawa ng command na ito?**

```
oc expose deployment/db-pod

```

Sinasabi nito:

```
Gumawa ng Service para sa Deployment na db-pod.
```
Results:

```
Deployment ---> Pod

Service -----> Pod

```

Makikita:

```
oc get svc
```
Output:

```
NAME    TYPE        CLUSTER-IP
db-pod  ClusterIP   172.30.108.92

```

---

**Hindi ba iisa lang sila?**

Hindi.

**Deployment**

Nagma-manage ng Pods.

```
Deployment
    |
    +-- Pod

```
---

**Service**

Nagbibigay ng stable network access.

```
Service
    |
    +-- Pod

```
---

**Example sa totoong buhay**
**Walang Service**
```
App ----> 10.128.0.5

```

Kapag nag-restart:

```
10.128.0.5 ❌

new IP:
10.128.0.8

```
Connection failed.

---


**May Service**

```
App ----> db-pod

```

Kahit magbago:

```
10.128.0.5
10.128.0.8
10.128.0.15
```
gumagana pa rin dahil si Service ang naghahanap ng Pod.

---

**Ano ang ginawa ng oc expose sa ilalim?**

Kapag ginawa mo:

```
oc expose deployment/db-pod
```
Parang gumawa ito ng YAML:

```
apiVersion: v1
kind: Service
metadata:
  name: db-pod
spec:
  selector:
    app: db-pod
  ports:
  - port: 3306
    targetPort: 3306


```

Pansinin ang:

```
selector:
  app: db-pod

```

Ito ang ginagamit para hanapin ang tamang Pod.

---


<h3>Ano naman ang selector?</h3>h3>

Sa screenshot mo:

```
oc get service db-pod -o wide

```
Output:

```
SELECTOR
app=db-pod

```

Ibig sabihin:

```
Service
   |
   | hanapin lahat ng Pods
   | na may label na
   |
app=db-pod

```
Check mo:

```
oc get pods --show-labels

```

Makikita mo:

```
NAME                   LABELS
db-pod-xxxxxx          app=db-pod

```

Kaya alam ni Service kung aling Pod ang papadalhan ng traffic.

---

**Paano makita ang endpoint?**

```
oc get endpoints db-pod

```

Halimbawa:

```
NAME     ENDPOINTS
db-pod   10.128.0.15:3306

```

Ibig sabihin:

```
db-pod Service
      |
      +--> 10.128.0.15:3306

```

---

**Mga madalas gamitin na expose**
**Expose Deployment**

```
oc expose deployment myapp

```
**Gagawa ng Service.**

---

**Expose Service**

```

oc expose service myapp

```

Sa OpenShift, madalas itong gumagawa ng:

```
Route

```

para ma-access mula browser.

Makikita:

```
oc get routes

```

---


<h2>Madaling Tandaan</h2>

```
Deployment
    |
    +--> Creates Pods

Service
    |
    +--> Creates stable IP/DNS to Pods

Route (OpenShift)
    |
    +--> Access application from outside cluster

```
Karaniwang flow sa OpenShift:

```
oc create deployment
        |
        v
      Pod

oc expose deployment
        |
        v
     Service

oc expose service
        |
        v
      Route

```

Kaya ang oc expose deployment/db-pod ay hindi gumagawa ng panibagong Pod o Deployment. Ang ginagawa lang nito ay gumawa ng Service (ClusterIP) para magkaroon ng permanenteng network name ang Deployment at mga Pods nito.

---



**Dito madalas nalilito ang mga nagsisimula sa OpenShift/Kubernetes.**

<h2>Flow ng Application sa OpenShift</h2>

```
Deployment
    |
    v
   Pod
    |
    v
 Service (ClusterIP)
    |
    v
 Route (OpenShift)


```

**Step 1 - Create Deployment**

```

oc create deployment myapp --image=myimage

```

Result:

```
Deployment
   |
   +-- Pod

```

May running application ka na, pero sa loob lang ng Pod.

---

**Step 2 - Expose Deployment**

```
oc expose deployment myapp

```
Result:

```
Service

```

Makikita:

```
oc get svc

```
Output:

```
NAME    TYPE
myapp   ClusterIP

```

Ngayon, pwede nang tawagin ng ibang Pods ang application gamit ang:

```
Ngayon, pwede nang tawagin ng ibang Pods ang application gamit ang:

```
myapp:8080

```
sa halip na Pod IP.

---

**Step 3 - Expose Service**

```
oc expose service myapp

```

Result:

```
Route

```

Makikita:

```
oc get routes

```
Output:

```
NAME    HOST/PORT
myapp   myapp.apps.ocp.lab.local

```
Ngayon pwede nang i-access mula browser:

```
http://myapp.apps.ocp.lab.local

```
---













































































