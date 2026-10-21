<h3>Guided Exercise: Scale and Expose Applications to External Access</h3>

Example Procedure command, papaliwanag sa baba ang buong processo

---

1. Create two web application deployments, named satir-app and sakila-app. Use the registry.lab.example.com:8443/redhattraining/do180-httpd-app:v1 container image for both deployments.

1.1 Log in to the OpenShift cluster as the developer user with developer as the password.

```
oc login -u developer -p developer \
https://api.lab.example.com:6443

```

1.2 Change to the web-applications project.

```
 oc project web-applications

```

1.3 Create the satir-app web application deployment by using the registry.lab.example.com:8443/redhattraining/do180-httpd-app:v1 container image.

```
oc create deployment satir-app \
--image registry.lab.example.com:8443/redhattraining/do180-httpd-app:v1

```

1.4 After a few moments, verify that the deployment is successful. A successful deployment shows the Running status.

```
oc get pods
```

```
oc get deploy/satir-app

```

1.5 Create the sakila-app web application deployment by using the registry.lab.example.com:8443/redhattraining/do180-httpd-app:v1 image.


```
oc create deployment sakila-app \
--image registry.lab.example.com:8443/redhattraining/do180-httpd-app:v1

```

1.6 Wait a few moments and then verify that the deployment is successful.

```
 oc get pods

```

```
oc get deploy
```

---

<h2>2. Create services for the web application deployments.</h2>

2.1 Expose the satir-app deployment. Name the service satir-svc, and specify port 8080 as the port and target port.

```
oc expose deployment satir-app --name satir-svc \
--port 8080 --target-port 8080

```

2.2 Expose the sakila-app deployment to create the sakila-svc service.

```
oc expose deployment sakila-app --name sakila-svc \
--port 8080 --target-port 8080

```

2.3 Verify the status of the services.

```
oc get services

```

2.4 Verify the status of the endpointslices resources.

```
oc get endpointslices

```

2.5 Verify the status of the pods.

```
oc get pods -o wide

```

---

<h2>3. Expose the satir-svc service and create the ingr-sakila ingress object.</h2>

3.1 Expose the satir-svc service to create a route named satir for the satir-app web application.

```
oc expose service satir-svc --name satir

```

```
oc get routes

```

3.2 Create an ingress object named ingr-sakila for the sakila-svc service. Configure the --rule option with the following values:

```
**Field**        **Value**
Host             ingr-sakila.apps.lab.example.com
Service name     sakila-svc
Port number      8080
```
```
oc create ingress ingr-sakila \
--rule "ingr-sakila.apps.lab.example.com/*=sakila-svc:8080"
```
```
oc get ingress
```

3.3 Confirm that a route exists for the ingr-sakila ingress object.

```
oc get routes

```
A specific port is not assigned to routes that ingress objects created. By contrast, a route that an exposed service created is assigned the same ports as the service.

3.4 Use the curl command to access the ingr-sakila ingress object and the satir route. The output states the name of the pod that is servicing the request.


```
 curl ingr-sakila.apps.lab.example.com
```

```
curl satir-web-applications.apps.lab.example.com
```


4. Scale the web application deployments to load-balance their services. Scale the sakila-app deployment to two replicas, and scale the satir-app deployment to three replicas.

4.1 Scale the sakila-app deployment with two replicas.

```
oc scale deployment sakila-app --replicas 2

```

4.2 Wait a few moments and then verify the status of the replica pods. The output confirms that the sakila-app deployment is scaled to two replicas.


```
oc get pods
```

4.3 Scale the satir-app deployment with three replicas.

```
oc scale deployment satir-app --replicas 3

```

4.4 Wait a few moments and then verify the status of the replica pods. The output confirms that the satir-app deployment is scaled to three replicas.


```
oc get pods -o wide

```


4.5 Retrieve the service endpoints to confirm that the services are load-balanced between the additional replica pods.

```
oc get endpointslices

```

<h3>5. Enable the sticky sessions for the sakila-app web application. Then, use the curl command to confirm that the sticky sessions are working for the ingr-sakila object.</h3>

5.1 Configure a cookie for the ingr-sakila ingress object.

```
 oc annotate ingress ingr-sakila \
ingress.kubernetes.io/affinity=cookie
```
5.2 Use the curl command to access the ingr-sakila ingress object. The output states the name of the pod that is servicing the request. Notice that the connection is load-balanced between the two replicas.

```
for i in {1..10}; do \
curl ingr-sakila.apps.lab.example.com ; done

```

5.3 Use the curl command to save the ingr-sakila ingress object cookie to the /tmp/cookie_jar file.

```
curl ingr-sakila.apps.lab.example.com \
-c /tmp/cookie_jar
```

5.4 Confirm that the cookie exists in the /tmp/cookie_jar file.

```
cat /tmp/cookie_jar

```

5.5 The cookie provides session stickiness for connections to the ingr-sakila route. Use the curl command and the cookie in the /tmp/cookie_jar file to connect to the ingr-sakila route again. Confirm that you are connected to the same pod that handled the request in the previous step.


```
for i in {1..10}; do \
curl ingr-sakila.apps.lab.example.com -b /tmp/cookie_jar; done

```

5.6 Use the curl command to connect to the ingr-sakila route without the cookie. Observe that session stickiness occurs only when you supply the stored cookie.

```
for i in {1..10}; do \
curl ingr-sakila.apps.lab.example.com ; done

```

6. Enable the sticky sessions for the satir-app web application. Then, use the curl command to confirm that sticky sessions are active for the satir route.

 6.1 Configure a cookie with the hello value for the satir route.

```
oc annotate route satir \
router.openshift.io/cookie_name="hello"

```

6.2 Use the curl command to access the satir route. The output states the name of the pod that is servicing the request. Notice that the connection is load-balanced between the three replica pods.

```
for i in {1..10}; do \
curl satir-web-applications.apps.lab.example.com; done

```


6.3 Use the curl command to save the hello cookie to the /tmp/cookie_jar file.

```
curl satir-web-applications.apps.lab.example.com \
-c /tmp/cookie_jar
```

6.4 Confirm that the hello cookie exists in the /tmp/cookie_jar file.

```
 cat /tmp/cookie_jar

```

6.5 The hello cookie provides session stickiness for connections to the satir route. Use the curl command and the hello cookie in the /tmp/cookie_jar file to connect to the satir route again. Confirm that you are connected to the same pod that handled the request in the previous step.

```
for i in {1..10}; do \
curl satir-web-applications.apps.lab.example.com -b /tmp/cookie_jar; done
```

6.6 Use the curl command to connect to the satir route without the hello cookie. Observe that session stickiness occurs only with the cookie.

```
for i in {1..10}; do \
curl satir-web-applications.apps.lab.example.com; done

```

---

Tanong: maari mo bang ipaliwanag lahat ng command na ito at para saan ito lalo na rin ang endpoint saan ito nakatutok? pag katapos ma expose at naging services meaning ang endpointslices is na ka point na sa pods palabas?


Answer:

Oo, tama ang direction ng pag-intindi mo. Ipaliwanag natin step-by-step kung paano nagkakabit-kabit ang Deployment → Pod → Service → EndpointSlice.


---

<h3> 1. Expose Deployment</h3>

Command:

```
oc expose deployment satir-app \
  --name satir-svc \
  --port 8080 \
  --target-port 8080

```

**Ano ang ginagawa nito?**

May existing ka nang Deployment:

```
satir-app
```
at may pod na ginawa nito:

```
satir-app-787b7-xxxx

```
Kapag nag:

```
oc expose deployment satir-app

```

gumagawa si OpenShift ng Service para sa deployment.

Parang sinasabi mo:

```
"Gawan mo ng permanenteng address ang mga pod ng deployment na ito."

```

---
<h3>Resulta</h3>

Nagkaroon ng Service:

```
satir-svc

```
---

**2. Service**

Pag nag:

```
oc get svc

```

Makikita

```
NAME        TYPE       CLUSTER-IP      PORT
satir-svc   ClusterIP  172.30.27.59    8080

```
---

**Ano ang ClusterIP?**

Ito ang virtual IP ng service.

Halimbawa:

```
172.30.27.59

```

Hindi ito IP ng pod.

Ito ay IP ng Service.

---

**Flow**

```
User
  |
  v
Service IP
172.30.27.59
  |
  v
Pod
10.8.0.126

```
Kapag kumonekta ka sa:

```
172.30.27.59:8080

```
ireredirect ka ng Service papunta sa Pod.

---

**3. Port vs TargetPort**

Command

```
--port 8080
--target-port 8080

```

---

**Port**

Ito ang port ng Service.

```
Service
172.30.27.59:8080

```
---

**Target Port**

Ito ang port sa loob ng container.

```
Pod
10.8.0.126:8080

```

---

**Example**

Puwedeng magkaiba:

```
oc expose deployment satir-app \
 --port=80 \
 --target-port=8080

```

Result:

```
Service :80
   |
   v
Pod :8080

```

---

**4. EndpointSlice**

Command:

```
oc get endpointslices

```

Output:

```
NAME              PORTS   ENDPOINTS
satir-svc-c5h59   8080    10.8.0.126

```
---

**Ito ang pinakaimportanteng bahagi**

Ang EndpointSlice ang nagsasabi:

```
"Anong pod ang dadaanan ng service?"

```

Sa halimbawa:

```
Service
satir-svc

```
nakaturo sa:

```
10.8.0.126
```

na siyang Pod IP.

---


**Diagram**

```
Service
satir-svc
172.30.27.59
      |
      |
      v
EndpointSlice
10.8.0.126:8080
      |
      |
      v
Pod
satir-app-787b7-xxxx

```

---

**5. Saan nakatutok ang EndpointSlice?**

Makikita rin sa:

```
oc get pods -o wide

```
Output:

```
NAME                    IP
satir-app-787b7-dfsh    10.8.0.126

```

At EndpointSlice:

```
10.8.0.126

```
Pareho.

Kaya masasabi nating:

✅ EndpointSlice → Pod IP

---

**6. Tama ba na "pag na-expose at naging service, ang EndpointSlice ay nakapoint sa Pods"?**

Sagot:

✅ OO.

Ganito ang relationship:

```
Deployment
      |
      v
Pods
      |
      v
Service
      |
      v
EndpointSlice
      |
      v
Pod IPs

```
Mas tamang isipin:

```

Service
   |
   v
EndpointSlice
   |
   v
Pods

```

Dahil ginagamit ng Service ang EndpointSlice para malaman kung saan niya ipapadala ang traffic.

---





















































































































































