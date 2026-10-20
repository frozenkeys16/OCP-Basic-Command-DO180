magkaiba ang Job at CronJob sa OpenShift/Kubernetes.

Yung command sa image

```
oc create job date-loop \
  --image registry.lab.example.com:8443/ubi9/ubi \
  -- /bin/bash -c "for i in {1..30}; do date; done"

```

Hatiin natin
1. oc create job date-loop

Gagawa ito ng Job na ang pangalan ay date-loop.

```
oc create job date-loop
```

Ang Job ay tatakbo isang beses lang hanggang matapos ang task.

Parang:

```
run once -> complete -> stop

```

Hindi ito katulad ng CronJob na paulit-ulit ayon sa schedule.

---

2. --image

```
--image registry.lab.example.com:8443/ubi9/ubi

```

Ito ang image na gagamitin ng Pod.

Parang katumbas ng:

```
containers:
- image: registry.lab.example.com:8443/ubi9/ubi

```

---

3. Ano ang ibig sabihin ng --

```
-- /bin/bash -c ...

```
Ang -- ay separator.

Sinasabi nito kay oc:

```
"Tapos na ang options ng oc create job. Ang susunod ay command na ipapasa sa container."

```


---

4. Bakit may /bin/bash -c?

```
/bin/bash -c "for i in {1..30}; do date; done"

```




































