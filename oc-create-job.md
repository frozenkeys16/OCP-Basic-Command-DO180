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
✅ Tama ka.

Ang -c ay nangangahulugang:

"bash, paki-execute itong command string."

Parang:

```
bash -c "command_here"

```

Example:

```
bash -c "date"

```
Output:


```
Mon Oct 20 09:00:00 PST 2026

```
---

5. Ano naman ang ginagawa ng script?

```
for i in {1..30}; do date; done

```
Equivalent nito:

```
for i in 1 2 3 4 5 ... 30
do
  date
done

```
Magpi-print ng date nang 30 beses.

Sample:

```
Mon Oct 20 09:00:01
Mon Oct 20 09:00:01
Mon Oct 20 09:00:01
...
30 times

```

Mabilis itong tatakbo dahil walang sleep.

---

Resulting YAML

Kapag ginawa mo:

```
oc get job date-loop -o yaml
```

Makikita mo:

```
containers:
- command:
  - /bin/bash
  - -c
  - for i in {1..30}; do date; done

```
Makikita mo na:
```
/bin/bash

```

ay command

at

```
-c

```
ay parameter ni bash.

---

**Job vs CronJob**

<h3<Job</h3>


```
oc create job backup-job \
  --image=ubi9 \
  -- /bin/bash -c "backup.sh"

```
Mangyayari:

```
Run once
Complete
Stop

```

---

<h3>CronJob</h3>
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "0 2 * * *"

```

Mangyayari:

```
02:00 AM -> Run
02:00 AM next day -> Run
02:00 AM next day -> Run

```
Ito ang katumbas ng Linux cron.

---


Madaling Tandaan

**Resource**    **Purpose**
Pod              Isang running workload
Job              Isang beses na task
CronJob          Scheduled na task
Deployment       Long-running application
DaemonSet        Isang Pod bawat node

---

Sa example mo, ang date-loop ay Job lang, kaya hindi ito scheduled tulad ng CronJob. Magki-create ito ng Pod, tatakbo ang:

```
for i in {1..30}; do date; done

```
pagkatapos ay magiging status na:

```
Completed
```
at tapos na ang Job.

---




























































