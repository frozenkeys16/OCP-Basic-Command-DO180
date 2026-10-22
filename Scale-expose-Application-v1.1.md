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





















