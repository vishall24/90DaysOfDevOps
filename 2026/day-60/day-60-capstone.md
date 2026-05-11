# Day 60 – Capstone: Deploy WordPress + MySQL on Kubernetes

## Task
Ten days of Kubernetes — clusters, Pods, Deployments, Services, ConfigMaps, Secrets, storage, StatefulSets, resource management, autoscaling, and Helm. Today you put it all together. Deploy a real WordPress + MySQL application using every major concept you have learned.


Architecture:

Internet → NodePort Service (port 30080)
               ↓
    WordPress Deployment (2 pods)
               ↓
    Headless Service (mysql)
               ↓
    MySQL StatefulSet (mysql-0)
               ↓
    PVC (persistent storage)

concepts:

| Concept | Learned on |
|---|---|
| Namespace | Day 52 |
| Deployments | Day 52 |
| Services (NodePort, Headless) | Day 53 |
| ConfigMaps | Day 54 |
| Secrets | Day 54 |
| PersistentVolumeClaims | Day 55 |
| StatefulSets | Day 56 |
| Resource Requests & Limits | Day 57 |
| Liveness & Readiness Probes | Day 57 |
| HPA | Day 58 |
| Helm | Day 59 |

---

## Challenge Tasks

### Task 1: Create the Namespace (Day 52)
1. Create a `capstone` namespace
2. Set it as your default: `kubectl config set-context --current --namespace=capstone`

---

WordPress needs MySQL to store blog posts, users, settings. MySQL needs persistent storage so data survives pod restarts. Everything lives in its own capstone namespace — clean and isolated.

Concept First
A namespace is like a folder inside your cluster. Everything we create today goes into capstone so it's all isolated and easy to clean up later with one command.

Done:

<img width="1078" height="408" alt="image" src="https://github.com/user-attachments/assets/f1b735ea-e362-4a88-bea7-4558c7102112" />

---

### Task 2: Deploy MySQL (Days 54-56)
1. Create a Secret with `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, and `MYSQL_PASSWORD` using `stringData`
2. Create a Headless Service (`clusterIP: None`) for MySQL on port 3306
3. Create a StatefulSet for MySQL with:
   - Image: `mysql:8.0`
   - `envFrom` referencing the Secret
   - Resource requests (cpu: 250m, memory: 512Mi) and limits (cpu: 500m, memory: 1Gi)
   - A `volumeClaimTemplates` section requesting 1Gi of storage, mounted at `/var/lib/mysql`
4. Verify MySQL works: `kubectl exec -it mysql-0 -- mysql -u <user> -p<password> -e "SHOW DATABASES;"`

**Verify:** Can you see the `wordpress` database?

---

Concept First:

MySQL is stateful — it needs:

A Secret for passwords (never hardcode these!)
A Headless Service so WordPress can reach it by a stable DNS name
A StatefulSet so it gets a stable name (mysql-0) and its own PVC that survives restarts

We use stringData in the Secret — Kubernetes auto base64-encodes it for us. Much easier than doing it manually.

created :
mysql-secret
mysql-headless-svc
mysql-statefulset

verified Databases, pod & PVC:

<img width="1436" height="725" alt="image" src="https://github.com/user-attachments/assets/486406ac-e21e-4999-9f05-43956a90982e" />

---

### Task 3: Deploy WordPress (Days 52, 54, 57)
1. Create a ConfigMap with `WORDPRESS_DB_HOST` set to `mysql-0.mysql.capstone.svc.cluster.local:3306` and `WORDPRESS_DB_NAME`
2. Create a Deployment with 2 replicas using `wordpress:latest` that:
   - Uses `envFrom` for the ConfigMap
   - Uses `secretKeyRef` for `WORDPRESS_DB_USER` and `WORDPRESS_DB_PASSWORD` from the MySQL Secret
   - Has resource requests and limits
   - Has a liveness probe and readiness probe on `/wp-login.php` port 80
3. Wait until both pods show `1/1 Running`

**Verify:** Are both WordPress pods running and ready?

---

Concept First:

WordPress is stateless — we use a Deployment (not StatefulSet) with 2 replicas for high availability.
WordPress needs to know:

WHERE is MySQL? → ConfigMap (WORDPRESS_DB_HOST)
WHAT database? → ConfigMap (WORDPRESS_DB_NAME)
WHO is connecting? → Secret (WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD)

We mix ConfigMap AND Secret in the same pod — one for non-sensitive config, one for credentials. This is the real-world pattern.
The DNS for MySQL is: mysql-0.mysql.capstone.svc.cluster.local:3306
Breaking it down:

mysql-0 = pod name
mysql = headless service name
capstone = namespace
svc.cluster.local = standard Kubernetes suffix

---



created wordpress-config & wordpress-deployment:

<img width="757" height="182" alt="image" src="https://github.com/user-attachments/assets/7f9fd55c-2e34-4893-b3e4-8b21a880ab14" />

<img width="506" height="844" alt="image" src="https://github.com/user-attachments/assets/7e5c5b67-618c-4133-bf5e-7200cee363fb" />

both pod for wordpress are running now:

<img width="706" height="116" alt="image" src="https://github.com/user-attachments/assets/f3419fa3-ea85-4e3f-bcd6-2388c5925615" />

---

### Task 4: Expose WordPress (Day 53)
1. Create a NodePort Service on port 30080 targeting the WordPress pods
2. Access WordPress in your browser:
   - Minikube: `minikube service wordpress -n capstone`
   - Kind: `kubectl port-forward svc/wordpress 8080:80 -n capstone`
3. Complete the setup wizard and create a blog post

**Verify:** Can you see the WordPress setup page?

---


<img width="1077" height="393" alt="image" src="https://github.com/user-attachments/assets/9c3c1f2e-284d-4d77-9181-60a5f25f0db4" />


able to access in browser:

<img width="1189" height="866" alt="image" src="https://github.com/user-attachments/assets/201546bc-bf73-477d-ab7b-de9b8c21aaf4" />

<img width="1440" height="608" alt="image" src="https://github.com/user-attachments/assets/5f49df79-adc9-4734-9032-556ede2521a0" />

---

### Task 5: Test Self-Healing and Persistence
1. Delete a WordPress pod — watch the Deployment recreate it within seconds. Refresh the site.
2. Delete the MySQL pod: `kubectl delete pod mysql-0 -n capstone` — watch the StatefulSet recreate it

3. After MySQL recovers, refresh Word
Press — your blog post should still be there

**Verify:** After deleting both pods, is your blog post still there?

---

Concept First:

This is the most satisfying task — you're proving that everything you built actually works under failure conditions.

<img width="984" height="318" alt="image" src="https://github.com/user-attachments/assets/f30fe5d0-9e5a-4e9e-80a6-2908aa8aa076" />

<img width="754" height="164" alt="image" src="https://github.com/user-attachments/assets/2130421a-e6d7-4b30-8a80-acf3217e4c0d" />

<img width="1440" height="647" alt="image" src="https://github.com/user-attachments/assets/f1f80830-0e0a-4ca9-a17f-cc4a2011de73" />

The Deployment sees "I need 2 replicas, I only have 1" and immediately creates a new pod.

The site still works — the other pod was serving traffic the whole time!

---

### Task 6: Set Up HPA (Day 58)
1. Write an HPA manifest targeting the WordPress Deployment with CPU at 50%, min 2, max 10 replicas
2. Apply and check: `kubectl get hpa -n capstone`
3. Run `kubectl get all -n capstone` for the complete picture

**Verify:** Does the HPA show correct min/max and target?

---

Concept First:

Now we add autoscaling to WordPress. If traffic spikes, HPA automatically adds more pods. When traffic drops, it scales back down. Remember HPA needs resources.requests set on the deployment — which we already did in Task 3!

<img width="1398" height="814" alt="image" src="https://github.com/user-attachments/assets/43675920-7b1d-4327-9f66-7828da4127b7" />

---

### Task 7: (Bonus) Compare with Helm (Day 59)
1. Install WordPress using `helm install wp-helm bitnami/wordpress` in a separate namespace
2. Compare: how many resources did each approach create? Which gives more control?
3. Clean up the Helm deployment

---

<img width="1434" height="605" alt="image" src="https://github.com/user-attachments/assets/a5486ba7-8af9-40e5-a87b-28397eb2d909" />

<img width="1287" height="808" alt="image" src="https://github.com/user-attachments/assets/229a9b00-4821-44d9-89fe-d2b3d81ecc59" />
7 - helm , 8 - manual
The difference:

Manual approach: full control, you understand every piece
Helm approach: one command, but less visibility into what's running

---

### Task 8: Clean Up and Reflect
1. Take a final look: `kubectl get all -n capstone`
2. Count the concepts you used: Namespace, Secret, ConfigMap, PVC, StatefulSet, Headless Service, Deployment, NodePort Service, Resource Limits, Probes, HPA, Helm — twelve concepts in one deployment
3. Delete the namespace: `kubectl delete namespace capstone`
4. Reset default: `kubectl config set-context --current --namespace=default`

**Verify:** Did deleting the namespace remove everything?

---

<img width="1428" height="511" alt="image" src="https://github.com/user-attachments/assets/536839a3-af4d-4cbe-b0c1-85f0aa778439" />

<img width="1026" height="152" alt="image" src="https://github.com/user-attachments/assets/0882227b-1b98-4564-a55c-10f90f963e12" />

---

## Architecture
    
    Internet
    ↓
    NodePort Service (port 30080)
    ↓
    WordPress Deployment (2 replicas)
    ↓ envFrom ConfigMap (DB_HOST, DB_NAME)
    ↓ secretKeyRef Secret (DB_USER, DB_PASSWORD)
    ↓
    Headless Service (mysql)
    ↓
    MySQL StatefulSet (mysql-0)
    ↓
    PVC (mysql-data-mysql-0) → 1Gi persistent storage


## Concepts Used

| Concept | Day Learned | Used For |
|---|---|---|
| Namespace | Day 52 | Isolated capstone environment |
| Deployment | Day 52 | WordPress (2 replicas, self-healing) |
| NodePort Service | Day 53 | Expose WordPress on port 30080 |
| Headless Service | Day 53 + 56 | Stable DNS for MySQL StatefulSet |
| ConfigMap | Day 54 | WordPress DB host and DB name |
| Secret | Day 54 | MySQL passwords |
| PVC via volumeClaimTemplates | Day 55 + 56 | MySQL persistent storage |
| StatefulSet | Day 56 | MySQL with stable name and storage |
| Resource Requests & Limits | Day 57 | CPU/memory for all containers |
| Liveness & Readiness Probes | Day 57 | WordPress health checks |
| HPA | Day 58 | Auto-scale WordPress pods |
| Helm | Day 59 | Bonus: compared to manual approach |

## Self-Healing Test Results
- Deleted WordPress pod → Deployment recreated it in seconds → site stayed up ✅
- Deleted mysql-0 → StatefulSet recreated it with same name → blog post survived ✅
- Data persisted because PVC outlives the pod ✅

## Reflection
- Hardest part: Getting WORDPRESS_DB_HOST DNS format exactly right
- What clicked: How StatefulSet + Headless Service + PVC work together for databases
- What I would add for production:
  - TLS/HTTPS with cert-manager
  - Ingress controller instead of NodePort
  - MySQL replication with multiple replicas
  - External secret management (Vault or AWS Secrets Manager)
  - Monitoring with Prometheus + Grafana


| Resource | What it does in this project |
|---|---|
| `capstone` namespace | Isolates all resources — delete namespace = delete everything |
| `mysql-secret` | Stores DB passwords securely using stringData |
| `mysql` Headless Service | Gives mysql-0 a stable DNS: mysql-0.mysql.capstone.svc.cluster.local |
| `mysql` StatefulSet | Runs MySQL with stable name (mysql-0) and its own PVC |
| `mysql-data-mysql-0` PVC | MySQL's persistent disk — survives pod deletion |
| `wordpress-config` ConfigMap | Stores DB_HOST and DB_NAME (non-sensitive config) |
| `wordpress` Deployment | Runs 2 WordPress pods — self-healing, stateless |
| `wordpress` NodePort Service | Exposes WordPress on port 30080 to your browser |
| Liveness Probe | Restarts WordPress pod if /wp-login.php stops responding |
| Readiness Probe | Removes WordPress pod from traffic if not ready |
| `wordpress-hpa` HPA | Auto-scales WordPress from 2 to 10 pods based on CPU |
| Helm (bonus) | Showed how one command replaces all the above YAML files |

