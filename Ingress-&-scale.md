para saan ang ingress at scale ung sa ingress ba ay neded ito para lang sa connection?


Sa OpenShift/Kubernetes, kadalasan nalilito ang mga nag-aaral sa Service, Ingress, Route, at Scale.


**1. Ano ang Ingress?**

Ito yung command sa screenshot:

```
oc create ingress ingr-sakila \
 --rule="ingr-sakila.apps.lab.example.com/*=sakila-svc:8080"

```
