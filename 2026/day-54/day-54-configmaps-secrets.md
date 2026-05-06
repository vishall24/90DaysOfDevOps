# Day 54 – Kubernetes ConfigMaps and Secrets

## Task
Your application needs configuration — database URLs, feature flags, API keys. Hardcoding these into container images means rebuilding every time a value changes. Kubernetes solves this with ConfigMaps for non-sensitive config and Secrets for sensitive data.

---

## Challenge Tasks

### Task 1: Create a ConfigMap from Literals
1. Use `kubectl create configmap` with `--from-literal` to create a ConfigMap called `app-config` with keys `APP_ENV=production`, `APP_DEBUG=false`, and `APP_PORT=8080`
2. Inspect it with `kubectl describe configmap app-config` and `kubectl get configmap app-config -o yaml`
3. Notice the data is stored as plain text — no encoding, no encryption

**Verify:** Can you see all three key-value pairs?

---

Created and verified from literal( literal means creating the service direct from command line not yaml file).

<img width="2860" height="318" alt="image" src="https://github.com/user-attachments/assets/7b25d567-b63b-4cf1-b0ba-717fbc08a309" />

---

did describe configmap and found this info(everything is visible):

<img width="1942" height="1100" alt="image" src="https://github.com/user-attachments/assets/3fa59462-c3f2-44ba-a574-fd342c232bca" />

---

config map yaml output:

<img width="2024" height="578" alt="image" src="https://github.com/user-attachments/assets/a0d3cc9b-7fdd-4122-9b88-78b6e33f465a" />

---

### Task 2: Create a ConfigMap from a File
1. Write a custom Nginx config file that adds a `/health` endpoint returning "healthy"
2. Create a ConfigMap from this file using `kubectl create configmap nginx-config --from-file=default.conf=<your-file>`
3. The key name (`default.conf`) becomes the filename when mounted into a Pod

**Verify:** Does `kubectl get configmap nginx-config -o yaml` show the file contents?

---

created default.conf file:

<img width="771" height="176" alt="image" src="https://github.com/user-attachments/assets/d6b733b8-d58a-4faa-879b-bfb7f3431970" />

---

created configmap named nginx-config from the file named default.conf:

<img width="1244" height="445" alt="image" src="https://github.com/user-attachments/assets/08c6ba07-a24f-4ee0-9b97-87da82188211" />

---

### Task 3: Use ConfigMaps in a Pod
1. Write a Pod manifest that uses `envFrom` with `configMapRef` to inject all keys from `app-config` as environment variables. Use a busybox container that prints the values.
2. Write a second Pod manifest that mounts `nginx-config` as a volume at `/etc/nginx/conf.d`. Use the nginx image.
3. Test that the mounted config works: `kubectl exec <pod> -- curl -s http://localhost/health`

Use environment variables for simple key-value settings. Use volume mounts for full config files.

**Verify:** Does the `/health` endpoint respond?

---
Use ConfigMap in Pod

Two ways:

Method	      | Use case
Environment   | variables	simple values
Volume mount	| full config files

PART A — Environment variables:

created env-pod yaml file:

<img width="740" height="292" alt="image" src="https://github.com/user-attachments/assets/f4ac4027-568e-49a6-9d6f-3f4e6913e4a5" />


by this:

Kubernetes injects:

APP_ENV
APP_DEBUG
APP_PORT

inside container

---

verified the variables got injected from app-config:

<img width="792" height="370" alt="image" src="https://github.com/user-attachments/assets/7fb802e1-0eb5-421d-9299-c4a293ae2225" />

---

created nginx-pod and mounted the configMap : nginx-config

<img width="890" height="507" alt="image" src="https://github.com/user-attachments/assets/cb76ca12-82f4-4ac2-b51f-ed700e177f0e" />

What happens?:

ConfigMap file becomes REAL file inside container

<img width="900" height="506" alt="image" src="https://github.com/user-attachments/assets/b3587c34-771a-4e8d-888e-a7e82e34ecd9" />

Verified and its showing healthy:

<img width="1116" height="93" alt="image" src="https://github.com/user-attachments/assets/e2332cd4-3619-42b3-9b45-a95ddab03900" />

whats inside nginx-config ?? this:

<img width="1012" height="400" alt="image" src="https://github.com/user-attachments/assets/d83d2bf2-7eeb-49c4-b747-bdf214171cd3" />

---

### Task 4: Create a Secret
1. Use `kubectl create secret generic db-credentials` with `--from-literal` to store `DB_USER=admin` and `DB_PASSWORD=s3cureP@ssw0rd`
2. Inspect with `kubectl get secret db-credentials -o yaml` — the values are base64-encoded
3. Decode a value: `echo '<base64-value>' | base64 --decode`

**base64 is encoding, not encryption.** Anyone with cluster access can decode Secrets. The real advantages are RBAC separation, tmpfs storage on nodes, and optional encryption at rest.

**Verify:** Can you decode the password back to plaintext?

---
created db-credentials and verified:

<img width="1432" height="357" alt="image" src="https://github.com/user-attachments/assets/994e6d33-f666-4910-bd19-e56d4af34558" />

---

decoded:

<img width="1000" height="51" alt="image" src="https://github.com/user-attachments/assets/4a001310-d904-4fe0-a0dd-02b741786639" />

---

### Task 5: Use Secrets in a Pod
1. Write a Pod manifest that injects `DB_USER` as an environment variable using `secretKeyRef`
2. In the same Pod, mount the entire `db-credentials` Secret as a volume at `/etc/db-credentials` with `readOnly: true`
3. Verify: each Secret key becomes a file, and the content is the decoded plaintext value

**Verify:** Are the mounted file values plaintext or base64?

---

Created secret-pod yaml and applied: one DB_USER getting injected using env and whole secrets are getting injected using volumes

<img width="910" height="663" alt="image" src="https://github.com/user-attachments/assets/d26480a1-71a5-4fca-a863-3da771769478" />

Volume mount:
Secret becomes files:

/etc/db-credentials/DB_USER
/etc/db-credentials/DB_PASSWORD

Checked by going inside the pod :

<img width="920" height="165" alt="image" src="https://github.com/user-attachments/assets/55fb9b05-b7cb-4cf1-87eb-8a9acba97662" />

---

### Task 6: Update a ConfigMap and Observe Propagation
1. Create a ConfigMap `live-config` with a key `message=hello`
2. Write a Pod that mounts this ConfigMap as a volume and reads the file in a loop every 5 seconds
3. Update the ConfigMap: `kubectl patch configmap live-config --type merge -p '{"data":{"message":"world"}}'`
4. Wait 30-60 seconds — the volume-mounted value updates automatically
5. Environment variables from earlier tasks do NOT update — they are set at pod startup only

**Verify:** Did the volume-mounted value change without a pod restart?

---

Created live-config using literals, then created live-pod and in volumes gave live-config:

<img width="1268" height="639" alt="image" src="https://github.com/user-attachments/assets/3da3a4b2-7e4b-4ba6-aa07-24a87318698b" />

check using logs command:

<img width="810" height="289" alt="image" src="https://github.com/user-attachments/assets/0856058f-6b40-4423-bef5-fa251f995575" />

after change now showing world:
    
    kubectl patch configmap live-config \
     --type merge \
     -p '{"data":{"message":"world"}}'
     
<img width="401" height="269" alt="image" src="https://github.com/user-attachments/assets/8d3ca59f-e1d9-4387-a6e4-ac3640e644fb" />

Type	                 |  Auto updates?
Volume mount           |	YES
Environment variable	 |  NO

---

Task 6:

<img width="1165" height="294" alt="image" src="https://github.com/user-attachments/assets/71dc3116-d1fa-4a36-9006-13baa3144169" />

---

FINAL UNDERSTANDING
ConfigMap

= Non-sensitive config

Secret

= Sensitive config

Environment variable

= Simple values

Volume mount

= Full config files

Base64

 = Encoding only
 = NOT encryption

 
