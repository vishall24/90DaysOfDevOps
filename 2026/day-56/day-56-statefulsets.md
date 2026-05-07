
# Day 56 – Kubernetes StatefulSets

## Task
Deployments work great for stateless apps, but what about databases? You need stable pod names, ordered startup, and persistent storage per replica. Today you learn StatefulSets — the workload designed for stateful applications like MySQL, PostgreSQL, and Kafka.

---

## PROBLEM STATEMENT:

    First, Understand WHY StatefulSets Exist
    Imagine you have a MySQL database cluster with 3 nodes. Each node needs:
    
    A fixed name (so node-2 always knows how to talk to node-1)
    Its own storage (node-1's data shouldn't mix with node-2's data)
    Ordered startup (primary DB must start before replicas)
    
    A regular Deployment can't do this — pod names are random like app-abc123-xyz. That's fine for a web server but a disaster for databases.
    StatefulSet solves all three problems.

## Challenge Tasks

### Task 1: Understand the Problem
1. Create a Deployment with 3 replicas using nginx
2. Check the pod names — they are random (`app-xyz-abc`)
3. Delete a pod and notice the replacement gets a different random name

This is fine for web servers but not for databases where you need stable identity.

| Feature | Deployment | StatefulSet |
|---|---|---|
| Pod names | Random | Stable, ordered (`app-0`, `app-1`) |
| Startup order | All at once | Ordered: pod-0, then pod-1, then pod-2 |
| Storage | Shared PVC | Each pod gets its own PVC |
| Network identity | No stable hostname | Stable DNS per pod |

Delete the Deployment before moving on.

**Verify:** Why would random pod names be a problem for a database cluster?

---


<img width="1113" height="141" alt="image" src="https://github.com/user-attachments/assets/1b083f6e-d241-4e89-9ab7-d2ddff4cd5b5" />

See those random suffixes? xkq2p, mnb7r — totally random every time.

<img width="936" height="147" alt="image" src="https://github.com/user-attachments/assets/0e18578f-3b87-42b0-bc57-6e51a62b63f9" />

The new pod will have a completely different random name. For a database cluster, this is a huge problem — other nodes won't know how to find it anymore.

Answer to "Why would random pod names be a problem for a database cluster?"
       
        Because database nodes need to communicate with each other using fixed addresses. 
        If node-1 is always reachable at mysql-1.mysql.default.svc.cluster.local, 
        the cluster works. If the name changes every restart, the whole cluster breaks.

---

### Task 2: Create a Headless Service
1. Write a Service manifest with `clusterIP: None` — this is a Headless Service
2. Set the selector to match the labels you will use on your StatefulSet pods
3. Apply it and confirm CLUSTER-IP shows `None`

A Headless Service creates individual DNS entries for each pod instead of load-balancing to one IP. StatefulSets require this.

**Verify:** What does the CLUSTER-IP column show?

---


What is a Headless Service?
Normally a Service gives you one IP that load-balances to all pods.
A Headless Service (clusterIP: None) gives each pod its own DNS name instead.
So instead of one IP for all pods, you get:

web-0.web-service.default.svc.cluster.local → Pod web-0's IP
web-1.web-service.default.svc.cluster.local → Pod web-1's IP
web-2.web-service.default.svc.cluster.local → Pod web-2's IP

StatefulSets require a Headless Service to work.

<img width="774" height="311" alt="image" src="https://github.com/user-attachments/assets/f594bb8d-8095-4a9f-a399-4f056ebb7d8a" />

<img width="919" height="105" alt="image" src="https://github.com/user-attachments/assets/405ae081-68e0-45ee-9f8e-0232193b8863" />

Verify: CLUSTER-IP shows None — that's what makes it headless!

---

### Task 3: Create a StatefulSet
1. Write a StatefulSet manifest with `serviceName` pointing to your Headless Service
2. Set replicas to 3, use the nginx image
3. Add a `volumeClaimTemplates` section requesting 100Mi of ReadWriteOnce storage
4. Apply and watch: `kubectl get pods -l <your-label> -w`

Observe ordered creation — `web-0` first, then `web-1` after `web-0` is Ready, then `web-2`.

Check the PVCs: `kubectl get pvc` — you should see `web-data-web-0`, `web-data-web-1`, `web-data-web-2` (names follow the pattern `<template-name>-<pod-name>`).

**Verify:** What are the exact pod names and PVC names?

---

What is volumeClaimTemplates?

    In a Deployment, all pods share one PVC.
    In a StatefulSet, volumeClaimTemplates automatically creates a separate PVC for each pod. So web-0 gets its own disk, web-1 gets its own disk, etc.



<img width="792" height="664" alt="image" src="https://github.com/user-attachments/assets/29e94a56-c8ea-44c3-95b1-3009aa9e1d3a" />

<img width="846" height="590" alt="image" src="https://github.com/user-attachments/assets/a61912db-11cd-42f1-97f1-4020d1e6ad3a" />

web-0   0/1   Pending    → Running   ← web-0 starts FIRST
web-1   0/1   Pending    → Running   ← web-1 starts only AFTER web-0 is Ready
web-2   0/1   Pending    → Running   ← web-2 starts only AFTER web-1 is Ready

Stable names! Not random. Always web-0, web-1, web-2.

Checked the pvc too:

<img width="1416" height="102" alt="image" src="https://github.com/user-attachments/assets/eaf6f4eb-9bff-4c84-a42d-036c549060b1" />

Verified: Pod names are web-0, web-1, web-2. PVC names follow pattern web-data-web-0, web-data-web-1, web-data-web-2.

---

### Task 4: Stable Network Identity
Each StatefulSet pod gets a DNS name: `<pod-name>.<service-name>.<namespace>.svc.cluster.local`

1. Run a temporary busybox pod and use `nslookup` to resolve `web-0.<your-headless-service>.default.svc.cluster.local`
2. Do the same for `web-1` and `web-2`
3. Confirm the IPs match `kubectl get pods -o wide`

**Verify:** Does the nslookup IP match the pod IP?

---

Each StatefulSet pod gets a DNS name in this format:
<pod-name>.<service-name>.<namespace>.svc.cluster.local

So for our setup:

web-0.web-service.default.svc.cluster.local
web-1.web-service.default.svc.cluster.local
web-2.web-service.default.svc.cluster.local


after executing : kubectl run dns-test --image=busybox:1.28 --rm -it --restart=Never -- sh :

<img width="720" height="396" alt="image" src="https://github.com/user-attachments/assets/4b5ec118-a17b-4283-81bf-b93789af032f" />


Verified: nslookup IP matches the pod IP from kubectl get pods -o wide.:

<img width="1416" height="102" alt="image" src="https://github.com/user-attachments/assets/69a599d0-85ae-4e38-ace1-417cac1281b9" />

<img width="1171" height="105" alt="image" src="https://github.com/user-attachments/assets/4295fa1e-2fe1-4565-9dd8-a669059112b3" />

---

### Task 5: Stable Storage — Data Survives Pod Deletion
1. Write unique data to each pod: `kubectl exec web-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"`
2. Delete `web-0`: `kubectl delete pod web-0`
3. Wait for it to come back, then check the data — it should still be "Data from web-0"

The new pod reconnected to the same PVC.

**Verify:** Is the data identical after pod recreation?

---

write data in each pod:

<img width="1378" height="68" alt="image" src="https://github.com/user-attachments/assets/54ae6e84-dad6-476b-808f-43ff43bacaf2" />

verified:

<img width="1087" height="50" alt="image" src="https://github.com/user-attachments/assets/2ecab221-ced6-4b3a-93c0-e6130855729b" />

Deleted and pod came back:

<img width="752" height="159" alt="image" src="https://github.com/user-attachments/assets/b9318870-31b6-4b52-bddb-a57d400851ff" />

Verified the data:

<img width="1085" height="53" alt="image" src="https://github.com/user-attachments/assets/40a2b324-f2d9-4573-b5c7-ab6827143e78" />

Why? Because the new web-0 pod reconnected to the same PVC (web-data-web-0). The PVC was never deleted — only the pod was.

---

### Task 6: Ordered Scaling
1. Scale up to 5: `kubectl scale statefulset web --replicas=5` — pods create in order (web-3, then web-4)
2. Scale down to 3 — pods terminate in reverse order (web-4, then web-3)
3. Check `kubectl get pvc` — all five PVCs still exist. Kubernetes keeps them on scale-down so data is preserved if you scale back up.

**Verify:** After scaling down, how many PVCs exist?

---

Scaled the statefulset & checked the pvc as well :

<img width="1411" height="593" alt="image" src="https://github.com/user-attachments/assets/d180150b-c800-4d24-9cad-f4fa92a0c551" />

Scaled it down and checked the pvc all the previous 5 pvc where there:

<img width="1412" height="307" alt="image" src="https://github.com/user-attachments/assets/b4ca27cd-9241-49b5-a93b-05388176bd77" />

All 5 PVCs still exist! Even though web-3 and web-4 pods are gone.

Verified: After scaling down to 3, there are still 5 PVCs. Kubernetes keeps them so if you scale back up, the data is still there.

---

### Task 7: Clean Up
1. Delete the StatefulSet and the Headless Service
2. Check `kubectl get pvc` — PVCs are still there (safety feature)
3. Delete PVCs manually

**Verify:** Were PVCs auto-deleted with the StatefulSet?

---

<img width="1400" height="182" alt="image" src="https://github.com/user-attachments/assets/94547b3c-8b36-4d20-b01d-e44064426ba4" />

They're still there! StatefulSet deletion does NOT delete PVCs — this is a safety feature so you don't accidentally lose data.

<img width="1427" height="182" alt="image" src="https://github.com/user-attachments/assets/2806b671-31a4-4a9e-84e4-01c981dd7b13" />

<img width="805" height="125" alt="image" src="https://github.com/user-attachments/assets/12b978da-b5ee-43e8-be19-f0c2802b3a10" />




## What are StatefulSets?
StatefulSets manage stateful applications (databases, message queues) that need:
- Stable, predictable pod names (web-0, web-1, web-2)
- Ordered startup and shutdown
- Per-pod persistent storage that survives restarts

Use Deployments for stateless apps (web servers, APIs).
Use StatefulSets for stateful apps (MySQL, PostgreSQL, Kafka, Redis clusters).

## Deployment vs StatefulSet

| Feature | Deployment | StatefulSet |
|---|---|---|
| Pod names | Random (app-xyz-abc) | Stable, ordered (app-0, app-1) |
| Startup order | All at once | Ordered: pod-0 → pod-1 → pod-2 |
| Storage | Shared PVC | Each pod gets its own PVC |
| Network identity | No stable hostname | Stable DNS per pod |

## How Headless Services Work
A normal Service load-balances to one IP.
A Headless Service (clusterIP: None) gives each pod its own DNS:
`web-0.web-service.default.svc.cluster.local`
This lets other pods talk to a specific pod by name — essential for DB clusters.

## volumeClaimTemplates
Instead of one shared PVC, StatefulSets auto-create a PVC per pod.
Pattern: `<template-name>-<statefulset-name>-<ordinal>`
Example: web-data-web-0, web-data-web-1, web-data-web-2

## Key Behaviors
- Scale up: pods create in order (0, 1, 2...)
- Scale down: pods terminate in reverse order (...2, 1, 0)
- Scale down does NOT delete PVCs — data is preserved
- Deleting a StatefulSet does NOT delete PVCs — must clean up manually
- Deleting a pod reconnects the new pod to the SAME PVC — data survives



| Concept | What it means |
|---|---|
| StatefulSet | Workload for stateful apps needing stable identity + storage |
| Headless Service | `clusterIP: None` — gives each pod its own DNS name |
| `volumeClaimTemplates` | Auto-creates a separate PVC per pod |
| Stable pod names | Always `web-0`, `web-1`, never random |
| Pod DNS format | `pod-name.service-name.namespace.svc.cluster.local` |
| Scale down | Pods delete in reverse order, PVCs are kept |
| StatefulSet delete | PVCs survive — must delete manually |
