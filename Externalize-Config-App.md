**Guided Exercise: Externalize the Configuration of Applications**

---

**Instructions**

1. Log in to the OpenShift web console as the developer user.

 1.1 Open a web browser and go to https://console-openshift-console.apps.lab.example.com
 1.2 Click Red Hat Identity Management and log in as the developer user with developer as the password. If the Welcome to the new OpenShift experience message is displayed, then click Skip tour.

2. Create a web application deployment named webconfig from the web console. Use the registry.lab.example.com:8443/ubi9/httpd-24:latest container image.

 2.1 Go to Workloads → Deployments.
 2.2 Select the storage-configs project from the project drop-down list. Ignore the restricted access error, which appears because the developer user cannot list deployments across all projects.

---

<img width="914" height="606" alt="image" src="https://github.com/user-attachments/assets/74d6f94b-9dfa-4328-b7a2-adc297382671" />

---


2.3 Click Create Deployment.
2.4 Use webconfig for the deployment name.
2. 5 Change the image name to registry.lab.example.com:8443/ubi9/httpd-24:latest.

---

<img width="918" height="607" alt="image" src="https://github.com/user-attachments/assets/6240326e-d9fa-4630-9954-e9f17b8605c5" />

---

2.6 Click Create. Wait for the blue circle to indicate that three pods are running.

---
<img width="912" height="623" alt="image" src="https://github.com/user-attachments/assets/9ca1718e-9ffe-4575-8e6c-5c00f356c9c9" />

---

3. Expose the web application to external access from the web console. Create a service and a route for the web application. Use the values from the following tables to create the service and the route.

---
<img width="1205" height="368" alt="image" src="https://github.com/user-attachments/assets/8397080d-bf6d-4ee7-97fb-73eafcef1763" />

---


3.1 Go to Networking → Services and click Create Service.

3.2 Update the service values from the service table.

---

<img width="936" height="607" alt="image" src="https://github.com/user-attachments/assets/c17431ab-11f4-4580-b0f7-dc658f949286" />

---

3.3 Click Create.

**Note**
Alternatively, you can create the service from the CLI by using the following command:

```

[student@workstation ~]$ oc expose deploy/webconfig \
--namespace storage-configs --name webconfig-svc \
--selector app=webconfig \
--target-port 8080 --port 8080


```

---

3.4 Go to Networking → Routes and click Create Route.
3.5   Use webconfig-rt for the route name.

---

<img width="917" height="632" alt="image" src="https://github.com/user-attachments/assets/a035eeaa-0f6c-4d5d-872c-4ea62aa020b1" />

---


3.6 Select webconfig-svc for the service and 8080 -→ 8080 for the target port.


---

<img width="949" height="615" alt="image" src="https://github.com/user-attachments/assets/f33e5e6c-0d50-4263-9990-fb1c048c6d44" />


---


3.7 Click Create.




