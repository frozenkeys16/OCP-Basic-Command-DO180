simula Ingress pababa, ito ang importanteng maintindihan mo dahil madalas lumalabas ito sa actual cluster administration.
Gagamit tayo ng mga command mula sa iyong PDF.

<h3>1. Create Ingress</h3>

```
oc create ingress ingr-sakila \
--rule="ingr-sakila.apps.lab.example.com/*=sakila-svc:8080"

```

Ano ang ginagawa nito?

Gumagawa ito ng Ingress object na magruroute ng traffic mula sa URL papunta sa Service.

Diagram:

```
User
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
sakila-app pod
``

```


**Breakdown**

```
ingr-sakila.apps.lab.example.com

```
Host name na gagamitin ng user.

```
sakila-svc

```

Target Service.

```
:8080
```
Port ng service.

Kapag may nag-access ng:

```
http://ingr-sakila.apps.lab.example.com

```

automatically ipapasa sa:

```
Service = sakila-svc
Port = 8080

```

---

<h3>2. Verify Ingress</h3>

```
oc get ingress

```

Halimbawa:

```
NAME          HOSTS
ingr-sakila   ingr-sakila.apps.lab.example.com

```

**Para saan?**

Para makita kung:

Existing ang ingress
Tama ang hostname
May assigned address na

Parang:

```
oc get route

```

pero para sa Kubernetes Ingress resources.


---


<h3>3. Verify Route</h3>

```
oc get routes

```

Output:

```
NAME
ingr-sakila
satir

```
**Bakit may route kahit ingress ang ginawa?**

Sa OpenShift:

```
Ingress
   |
   v
Auto-generated Route

```

Kapag gumawa ka ng Ingress, si OpenShift router gagawa rin ng Route internally.

Ito ang isa sa malaking pagkakaiba ng OpenShift kumpara sa pure Kubernetes.


---


<h3>4. Test Connectivity</h3>

```
curl ingr-sakila.apps.lab.example.com
```

Output

```
Welcome to Red Hat Training,
from sakila-app-66947cdd78-x5kpd
```

**Bakit lumabas yung pod name?**

Para makita mo kung aling pod ang sumagot sa request.

Ito ay pang-test ng:

✅ Ingress
✅ Service
✅ Pod
Lahat gumagana.

---

<h3>5. Scale Deployment</h3>
Dalawang Replica

```
oc scale deployment sakila-app --replicas 2

```

Before

```
sakila-app-xxxxx

```

1 pod lang.

After:

```
sakila-app-xxxxx
sakila-app-yyyyy

```
2 pods na.

---

<h3>Tatlong Replica</h3>

```
oc scale deployment satir-app --replicas 3

```
After

```
satir-app-a
satir-app-b
satir-app-c

```
Tatlong pods na tumatakbo.

---


<h3>Bakit tayo nag-scale?</h3>

Para sa:

<h3>High Availability</h3>

Kapag namatay ang isang pod:

```
Pod A = Down
Pod B = Running

```

Available pa rin application.

---

<h3>Load Balancing</h3>

Kung maraming users:

```
User1 -> PodA
User2 -> PodB
User3 -> PodC

```
nahahati ang traffic.

---

<h3>6. Check EndpointSlices</h3>

```
oc get endpointslices

```

Example:

```
sakila-svc-dcv28

```

Output:

```
10.8.0.127
10.8.0.128

```

**Ano ang EndpointSlice?**

Listahan ito ng mga pod IP na dinadalhan ng service.

Parang:

```
Service
 |
 +--> Pod1
 |
 +--> Pod2
 |
 +--> Pod3

```

Kapag may bagong replica:

```
oc scale

```

dadagdag din sa EndpointSlice.


----

<h3>7. Enable Sticky Session sa Ingress</h3>

```
oc annotate ingress ingr-sakila \
ingress.kubernetes.io/affinity=cookie

```

<h3>Ano ang Sticky Session?</h3>

Normally:

```
Request1 -> PodA
Request2 -> PodB
Request3 -> PodA
Request4 -> PodB

```

Paiba-iba.

---


Kapag sticky:

```
User A -> PodA
User A -> PodA
User A -> PodA

```
Laging sa parehong pod.

Ginagamit ito sa:

Login session
Shopping cart
Banking application

Para hindi mawala ang user session.

---


<h3>8. Save Cookie</h3>

```
curl ingr-sakila.apps.lab.example.com \
-c /tmp/cookie_jar

```

**Ano ang ginagawa?**

Kumuha ng cookie mula sa server at sine-save.

Example:
```
cookie_jar

```

lalagyan ng:

```
session id

```

---

**9. Reuse Cookie**

```
curl ingr-sakila.apps.lab.example.com \
-b /tmp/cookie_jar

```

**Ano ginagawa nito?**

Ipinapadala ulit ang cookie.

Result:

```
same pod
same session
```
Hindi na ililipat sa ibang pod.

---

<h3>10. Sticky Session sa Route</h3>

Para naman sa OpenShift Route:

```
oc annotate route satir \
router.openshift.io/cookie_name="hello"

```

Ano ginagawa nito?

Lumilikha ng cookie name:

```
hello

```
at ginagamit iyon para sa sticky session.

Diagram:

```
User
 |
 | hello cookie
 v
Route
 |
 v
Pod A

```

Lahat ng susunod na request:

```
Pod A
Pod A
Pod A

```
Hangga't valid ang cookie.

---


<h3>Madaling Tandaan</h3>
Flow ng Exposure

```
Deployment
      |
      v
   Service
      |
      v
 Route/Ingress
      |
      v
    User
```

Commands

```
oc expose deployment

```
Create Service

```
oc expose service
```
Create Route

```
oc create ingress
```

Create Ingress

```
oc scale deployment --replicas=N

```

Dagdag bawas Pod

```
oc get endpointslices

```
Tingnan kung anong Pods ang nasa likod ng Service

```
oc annotate ingress ... affinity=cookie

```
Sticky Session sa Ingress

```
oc annotate route ... cookie_name=hello

```
Sticky Session sa Route

---


**Tandaan ang pinakaimportanteng exam/interview question**

Ano ang pagkakaiba ng Route at Ingress?

**Route**

OpenShift specific
oc expose service
Mas madaling gamitin
May built-in OpenShift router

**Ingress**

Standard Kubernetes object
Portable sa ibang Kubernetes distribution
Kailangan ng Ingress Controller
Sa OpenShift, automatic na kino-convert sa Route

Ito ang pinakamadalas na tanong kapag pinag-uusapan ang networking sa OpenShift.


---


Ang pinakamahalagang maintindihan dito ay ang --rule dahil ito ang nagsasabi sa Ingress kung saan ipapadala ang traffic.


Command:

```
oc create ingress ingr-sakila \
--rule="ingr-sakila.apps.lab.example.com/*=sakila-svc:8080"

```

Hatiin natin:

```
ingr-sakila.apps.lab.example.com/*=sakila-svc:8080

```

**Structure**

```
HOST/PATH=SERVICE:PORT

```
Sa example:

```
ingr-sakila.apps.lab.example.com/*=sakila-svc:8080
│
├─ Host
│
├─ Path
│
├─ Service
│
└─ Port


```

---

**1. Host**

```
ingr-sakila.apps.lab.example.com

```

Ito ang URL na ita-type ng user sa browser.

Example:

```
http://ingr-sakila.apps.lab.example.com

```

Kapag may request na dumating sa hostname na ito, saka gagana ang rule.

---

**2. Path**

```
/*

```

Ibig sabihin:

```
lahat ng path

```
Examples:

```
/
```

```
/index.html

```
```
/products
```

```
/login

```
lahat ipapadala sa parehong service.

Parang:

```
if host = ingr-sakila.apps.lab.example.com
 then send all traffic to sakila-svc

```

---

**3. Service**

```
sakila-svc

```

Ito ang Kubernetes/OpenShift Service.

Makikita mo ito sa:

```
oc get svc

```

Halimbawa:

```
NAME         TYPE
sakila-svc   ClusterIP

```

Ang Service ang naglo-load balance papunta sa mga Pod.

```
Ingress
   |
   v
sakila-svc
   |
   +--> Pod1
   +--> Pod2
   +--> Pod3
``

```

---

**4. Port**

```
8080
```
Ito ang port ng Service.

Makikita mo sa:

```
oc describe svc sakila-svc

```
Example

```
Port: 8080/TCP
TargetPort: 8080

```

Kaya alam ng Ingress kung saan ipapasa ang request.

---


**Buong Flow**

Kapag may user:

```
Browser
   |
   v
http://ingr-sakila.apps.lab.example.com
   |
   v
Ingress
   |
   v
sakila-svc:8080
   |
   v
Pod


```

---

**Katumbas nito sa YAML**

Ang command na ito:

```
oc create ingress ingr-sakila \
--rule="ingr-sakila.apps.lab.example.com/*=sakila-svc:8080"

```

ay parang gumawa ka ng:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingr-sakila
spec:
  rules:
  - host: ingr-sakila.apps.lab.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: sakila-svc
            port:
              number: 8080


```

Mapapansin mo na ang --rule ay shortcut lamang para hindi ka na gumawa ng buong YAML.

---

**Bakit may --rule?**

Para mabilis makagawa ng Ingress gamit lang ang command line.

Sa halip na:

```
vi ingress.yaml
oc apply -f ingress.yaml

```

isang command lang:

```
oc create ingress ... --rule=...

```

---

**Madaling Tandaan**

Kapag nakita mo ang:

```
--rule="HOST/PATH=SERVICE:PORT"

```
basahin mo ito bilang:

"Kapag may pumasok na request sa HOST at PATH na ito, ipadala sa SERVICE na ito gamit ang PORT na ito."

Sa exam o actual work, kadalasan ang tanong ay:

"Saan napupunta ang traffic?"

Sagot:

```
Host
  ↓
Ingress Rule
  ↓
Service
  ↓
Pod

```

Kaya ang --rule ang nag-uugnay sa URL ng user papunta sa tamang Service/Pod.


























 
























































