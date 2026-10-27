<h3>Guided Exercise: Provision Persistent Data Volumes</h3>

**Instructions**

1. Log in to the OpenShift web console as the developer user.

1.1 Open a web browser and go to https://console-openshift-console.apps.lab.example.com

1.2 Click Red Hat Identity Management and log in as the developer user with developer as the password. If the Welcome to the new OpenShift experience! message is displayed, then click Skip tour.

2. Identify the default storage class by going to Storage → StorageClasses. The nfs-storage storage class has the Default label and uses the k8s-sigs.io/nfs-subdir-external-provisioner provisioner.

---
<img width="964" height="598" alt="image" src="https://github.com/user-attachments/assets/e48fa6b5-d8e7-47fe-8c02-f0957bb7ef8f" />
---

3. Use the registry.lab.example.com:8443/rhel10/mariadb-118 container image to create a MariaDB deployment named db-pod. Use the storage-volumes project that the lab command created. Add a service for the database.

3.1 Go to Workloads → Deployments and select the storage-volumes project.

---

<img width="966" height="581" alt="image" src="https://github.com/user-attachments/assets/b1683756-9596-4510-a3e4-c9e7ffbb43a9" />

---












