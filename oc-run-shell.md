**Checking for Network**

---
<img width="1279" height="668" alt="image" src="https://github.com/user-attachments/assets/9ead1c9a-4921-4928-8a78-151975f06c41" />
---

Ang section na yan ay para i-verify na tama ang pagkakakonekta ng Service sa bagong Pod at para subukan ang DNS name ng Service sa loob ng cluster.

**1. oc get pods --selector app=db-pod -o wide**

```
oc get pods --selector app=db-pod -o wide

```
**Ano ang ginagawa nito?**

Hinahanap lahat ng Pods na may label na:

```
app=db-pod

```

Katumbas nito:

```
oc get pods -l app=db-pod -o wide

```

**Bakit kailangan ito?**

Para makita kung may Pod na tinutukoy ang Service.

Halimbawa:

```
Service Selector:
app=db-pod

```
Dapat may Pod na ganito rin:

```
Pod Labels:
app=db-pod

```

Kung walang matching label:

```
Service -> walang makitang Pod

```
Hindi gagana ang application.

---



**2. oc get endpointslices**

```
oc get endpointslices
```
Ano ang EndpointSlice?

Ito ang listahan ng mga Pod IP na ginagamit ng Service.

Halimbawa:

```
Service: db-pod
```
may Pod na:

```
10.8.0.64

```
Makikita:

```
ENDPOINTS
10.8.0.64

```
---


**FLOW**

```

Service
   |
Selector app=db-pod
   |
Pod
IP: 10.8.0.64

```

Makikita ni EndpointSlice:

```
10.8.0.64:3306
```
---

Bakit mahalaga ito?

Kapag may problema:

```
oc get svc

```

nakikita mo ang Service

pero hindi gumagana.

Tapos:

```
oc get endpointslices

```


at walang IP na lumabas.

Ibig sabihin:

```
Service ❌ hindi nakakakita ng Pod

```

kadalasan label mismatch ang problema.

---

**3. oc run shell -it**

```
oc run shell -it \
--image registry.lab.example.com:8443/openshift4/network-tools-rhel8

```
Ano ang ginagawa nito?

Gumagawa ng temporary Pod.

```
shell
```
na may mga network tools gaya ng:

```
nslookup
host
dig
curl
ping

```

Parang gumawa ka ng maliit na Linux server sa loob ng OpenShift.


---

**Ano ang meaning ng -it?**
-i

Interactive

```
mananatiling bukas ang session

```
-t
TTY

```
magkakaroon ka ng terminal

```

---

Pagkatapos:
```
bash-4.4$
```
nasa loob ka na ng Pod.


---

Para saan ito ginagamit?

Halimbawa may Service kang:

```
db-pod

```
Pwede mong itest:

```
nslookup db-pod
```
Output:

```
Name: db-pod
Address: 172.30.x.x
```
---

Pwede din:

```
curl http://myapp

```

or
```
nc -zv db-pod 3306
```

upang makita kung open ang MySQL port.

---

**Buong Purpose ng Section**

Ang ginagawa ng training na ito ay:

Hakbang 1

Suriin ang Pod

```
oc get pods --selector app=db-pod -o wide
```
Result:

```
May Pod ba?
```
---

Hakbang 2

Suriin kung nakita ni Service ang Pod

```
oc get endpointslices
```
Result:
```
Nasa Endpoint ba ang Pod IP?

```

---

Hakbang 3

Gumawa ng test Pod

```
oc run shell -it ...

```
Result
```
May test machine ka sa loob ng cluster

```
---















































































