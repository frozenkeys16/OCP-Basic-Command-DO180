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




















































































