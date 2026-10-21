para saan ang ingress at scale ung sa ingress ba ay neded ito para lang sa connection?


Sa OpenShift/Kubernetes, kadalasan nalilito ang mga nag-aaral sa Service, Ingress, Route, at Scale.


**1. Ano ang Ingress?**

Ito yung command sa screenshot:

```
oc create ingress ingr-sakila \
 --rule="ingr-sakila.apps.lab.example.com/*=sakila-svc:8080"

```

Ano ang ginagawa nito?

Gumagawa ito ng Ingress object na nagsasabi:

"Kapag may dumating na request sa hostname na ito, ipadala mo sa service na ito."

Parang traffic director.

```
Client
   |
   v
ingr-sakila.apps.lab.example.com
   |
   v
Ingress
   |
   v
sakila-svc
   |
   v
Pod

```

---

**2. Para saan ang Ingress?**

Hindi siya application.

Hindi rin siya Pod.

Ang trabaho niya ay routing ng external connections papasok sa cluster.

Halimbawa:

May service ka:

```
sakila-svc

```

Internal lang ito sa cluster.

Kapag gusto mong ma-access ng browser:

```
http://ingr-sakila.apps.lab.example.com

```

kailangan mo ng Ingress o Route.

---

**3. Ano ang pagkakaiba ng Service at Ingress?**
**Service**
```
ClusterIP
172.30.x.x

```

Pang-loob lang ng cluster.

```
Pod A ---> Service ---> Pod B
```




















