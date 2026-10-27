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

---

**Note**
Alternatively, you can create the route from the CLI by using the following command:

```
[student@workstation ~]$ oc expose svc/webconfig-svc \
--namespace storage-configs --name webconfig-rt

```

3.8 On the Details page of the route, click the Location link.


---

<img width="937" height="622" alt="image" src="https://github.com/user-attachments/assets/a39926e9-1652-45db-97d4-ca4c4fec8949" />


---

3.9  The route link opens a browser tab that displays the default test page of the web server. Do not close the browser tab, because you use it in another step.



---

<img width="1090" height="679" alt="image" src="https://github.com/user-attachments/assets/27314332-4d9d-4bbb-9b2f-4cb5911173ed" />


---

4. Create a configuration map that contains the files for the web application.


4.1 Change to the web console browser tab. Go to Workloads → ConfigMaps and click Create ConfigMap.
4.2 Set the name of the configuration map to webfiles.
4.3 In the Data section, define the index.html name as the Key value, and click Browse in the Value field to select the /home/student/DO180/labs/storage-configs/index.html file.

---

<img width="961" height="628" alt="image" src="https://github.com/user-attachments/assets/4a65e1ce-f17b-4b11-8713-c24b2b81e51f" />

---

4.4 Add a binary data key. In the Binary Data section, click Add key/value.

---

<img width="968" height="625" alt="image" src="https://github.com/user-attachments/assets/dcb94270-3a23-4ab5-9ead-63b79e2b9f4e" />

---

4.5 Define the redhatlogo.png name as the Key value, and click Browse in the Value field to select the /home/student/DO180/labs/storage-configs/redhatlogo.png file.


---

<img width="937" height="625" alt="image" src="https://github.com/user-attachments/assets/a36391f2-bb83-457a-a71c-048b0f440c6f" />


---

4.6 Click Create.

---


**5. Mount the webfiles configuration map as a volume in the webconfig deployment by using the command line.**

5.1 Open a terminal window and log in to the RHOCP cluster as the developer user with developer as the password.

```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.lab.example.com:6443


```

5.2 Ensure that you are in the storage-configs project.

```
[student@workstation ~]$ oc project storage-configs

```

5.3 Mount the webfiles configuration map as a volume.

```
[student@workstation ~]$ oc set volume deployment/webconfig \
--add --type configmap --configmap-name webfiles \
--name webfiles-vol --mount-path /var/www/html/

```

5.4 Verify that the deployment has three available replicas.

```
[student@workstation ~]$ oc get deployment

```

---

6. Verify that the web application shows the content from the configuration map.

6.1 Switch to the web browser, and go to the web server tab. Click the reload icon in the web browser.
6.2 Click the Click me link that the web page displays. The web page shows the Red Hat logo.

---

<img width="1075" height="255" alt="image" src="https://github.com/user-attachments/assets/50c726ed-19e8-44ff-9714-dc93c445dba2" />

---
