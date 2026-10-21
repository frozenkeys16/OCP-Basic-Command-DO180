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

---

**Ingress**

Pang-labas.

```
Internet/User
      |
      v
Ingress
      |
      v
Service
      |
      v
Pod


```

---

**4. Sa OpenShift bakit may Route pa?**

Sa screenshot:

```
oc get routes

```

Makikita:

```
ingr-sakila.apps.lab.example.com

```

OpenShift may sariling object:

```
Route

```

Parang mas OpenShift version ng Ingress.

---

Typical OpenShift flow:

```

User Browser
      |
      v
Router
      |
      v
Route / Ingress
      |
      v
Service
      |
      v
Pod


```

---


5. Ano ang Router?

Sa output:

```
ADDRESS
router-default


```

Ito yung OpenShift Router.

Madalas HAProxy-based.

Trabaho nito:

```

Tumanggap ng HTTP/HTTPS Traffic

```

at hanapin kung saan ipapadala.

Example:

```
https://ingr-sakila.apps.lab.example.com

```

Router

```
Route ba ito?
Ingress ba ito?

```

Kapag nakita:

```
sakila-svc

```

doon niya ipapasa.


---

**6. Ano naman ang curl test?**

Screenshot:

```
curl ingr-sakila.apps.lab.example.com

```

Output:

```
Welcome to Red Hat Training,
from sakila-app-66947cdd78-x5kpd

```

Ibig sabihin:

✅ Gumagana ang DNS
✅ Gumagana ang Ingress
✅ Gumagana ang Service
✅ Nakaabot sa Pod

---

Flow:

```
curl
 |
 v
Ingress
 |
 v
Service
 |
 v
Pod

```

---

7. Ano naman ang Scale?

Command:

```
oc scale deployment sakila-app --replicas=2

```

---

Before

```
sakila-app
   |
   v
1 Pod

```

---


After

```

sakila-app
   |
   +--> Pod1
   |
   +--> Pod2
``

```

Dalawang magkaparehong application instance.

---

Bakit nag-scale?

Para sa:

High Availability

Kapag namatay:

```
Pod1
```
may

```
Pod2

```

parin

---

Load Balancing

Kung maraming users.

```
100 Users

```
Sa halip na 

```
100 -> Pod1

```
nagiging

```
50 -> Pod1
50 -> Pod2

```

---

8. Paano nakakatulong ang Service kapag nag-scale?

Halimbawa:


```

oc scale deployment sakila-app --replicas=3

```
Magiging:

```
Pod1 10.8.0.10
Pod2 10.8.0.11
Pod3 10.8.0.12

```
EndpointSlice:

```
10.8.0.10
10.8.0.11
10.8.0.12

```

Service:

```
sakila-svc
```

---

Kapag may request:

```
curl http://sakila-svc

```

Service:

```
Request1 -> Pod1
Request2 -> Pod2
Request3 -> Pod3
```
Naglo-load balance siya.9. Kailangan ba ang Ingress para mag-scale?

❌ Hindi.

Magkaibang bagay sila.

Scale

Dinadagdagan ang Pods.


```
oc scale deployment sakila-app --replicas=3

```

---

Ingress

Nagbibigay ng access mula sa labas.

```
---

Pwede kang mag-scale kahit walang Ingress.

Pwede ka ring may Ingress kahit isang Pod lang.


---

Madaling Tandaan
Internal Communication

```
Pod
 |
 v
Service
```

External Communication

```
User
 |
 v
Ingress / Route
 |
 v
Service
 |
 v
Pods

```
High Availability

```
Deployment
 |
 +--> Pod1
 +--> Pod2
 +--> Pod3

 ```
Load Balancing

```

Service
 |
 +--> Pod1
 +--> Pod2
 +--> Pod3

 ```

---

Simple rule:

 - Deployment = gumagawa ng Pods
 - Scale = dumadami o nababawasan ang Pods
 - Service = stable IP/DNS ng Pods
 - EndpointSlice = listahan ng Pod IPs
 - Ingress/Route = daan mula sa labas papunta sa Service
 - Router = tumatanggap ng request at sumusunod sa Route/Ingress
 - Pod = actual na tumatakbong application

Kaya ang sagot sa tanong mo:

✅ Oo, ang pangunahing purpose ng Ingress/Route ay connection at routing ng traffic mula sa labas papasok sa Service. Hindi ito para gumawa ng Pods o mag-load balance ng replicas. Ang pag-scale ay trabaho ng Deployment, at ang Service ang magdi-distribute ng traffic sa mga Pods na ginawa ng scale.



---















































































