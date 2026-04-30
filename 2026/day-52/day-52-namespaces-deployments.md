# Day 52 – Kubernetes Namespaces and Deployments

---

## Challenge Tasks

### Task 1: Explore Default Namespaces
Kubernetes comes with built-in namespaces. List them:

```bash
kubectl get namespaces
```

You should see at least:
- `default` — where your resources go if you do not specify a namespace
- `kube-system` — Kubernetes internal components (API server, scheduler, etc.)
- `kube-public` — publicly readable resources
- `kube-node-lease` — node heartbeat tracking

Check what is running inside `kube-system`:
```bash
kubectl get pods -n kube-system
```

These are the control plane components keeping your cluster alive. Do not touch them.

**Verify:** How many pods are running in `kube-system`?

---
Namespaces available:

<img width="1632" height="310" alt="image" src="https://github.com/user-attachments/assets/c662ed9e-1900-4961-baf7-493e555b0b1e" />

all pods in system namespace: (like api-server, kube-proxy, etcd, controller manager, scheduler) (only control plane components)

<img width="2128" height="712" alt="image" src="https://github.com/user-attachments/assets/8c841947-616f-4dea-9783-a8d569d5142e" />

---

### Task 2: Create and Use Custom Namespaces
Create two namespaces — one for a development environment and one for staging:

```bash
kubectl create namespace dev
kubectl create namespace staging
```

Verify they exist:
```bash
kubectl get namespaces
```

You can also create a namespace from a manifest:
```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

```bash
kubectl apply -f namespace.yaml
```

Now run a pod in a specific namespace:
```bash
kubectl run nginx-dev --image=nginx:latest -n dev
kubectl run nginx-staging --image=nginx:latest -n staging
```

List pods across all namespaces:
```bash
kubectl get pods -A
```

Notice that `kubectl get pods` without `-n` only shows the `default` namespace. You must specify `-n <namespace>` or use `-A` to see everything.

**Verify:** Does `kubectl get pods` show these pods? What about `kubectl get pods -A`?

---

created and verified namespaces:

<img width="1818" height="598" alt="image" src="https://github.com/user-attachments/assets/05b1f474-ca32-49ff-a63a-458091f8b569" />

created:

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

created using yaml:

<img width="1606" height="472" alt="image" src="https://github.com/user-attachments/assets/f1b9a624-45d0-46a3-aa00-e317bd171763" />

Created pod in namespaces:

<img width="2860" height="1108" alt="image" src="https://github.com/user-attachments/assets/2a988e62-a74e-46da-ace9-222d65187cab" />

---

### Task 3: Create Your First Deployment
A Deployment tells Kubernetes: "I want X replicas of this Pod running at all times." If a Pod crashes, the Deployment controller recreates it automatically.

Create a file `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
```

Key differences from a standalone Pod:
- `kind: Deployment` instead of `kind: Pod`
- `apiVersion: apps/v1` instead of `v1`
- `replicas: 3` tells Kubernetes to maintain 3 identical pods
- `selector.matchLabels` connects the Deployment to its Pods
- `template` is the Pod template — the Deployment creates Pods using this blueprint

Apply it:
```bash
kubectl apply -f nginx-deployment.yaml
```

Check the result:
```bash
kubectl get deployments -n dev
kubectl get pods -n dev
```

You should see 3 pods with names like `nginx-deployment-xxxxx-yyyyy`.

**Verify:** What do the READY, UP-TO-DATE, and AVAILABLE columns mean in the deployment output?

---

2 replicas running:

<img width="1612" height="190" alt="image" src="https://github.com/user-attachments/assets/4fd4dbe6-d01e-45bf-91fd-9d8a76e7065f" />

<img width="1650" height="238" alt="image" src="https://github.com/user-attachments/assets/2ff2051f-581a-4445-a7d1-6589f6eebde3" />

READY	 = running pods
UP-TO-DATE  = 	updated pods
AVAILABLE	 = healthy pods


---

### Task 4: Self-Healing — Delete a Pod and Watch It Come Back
This is the key difference between a Deployment and a standalone Pod.

```bash
# List pods
kubectl get pods -n dev

# Delete one of the deployment's pods (use an actual pod name from your output)
kubectl delete pod <pod-name> -n dev

# Immediately check again
kubectl get pods -n dev
```

The Deployment controller detects that only 2 of 3 desired replicas exist and immediately creates a new one. The deleted pod is replaced within seconds.

**Verify:** Is the replacement pod's name the same as the one you deleted, or different?

---

another pod came up:

<img width="2448" height="540" alt="image" src="https://github.com/user-attachments/assets/9c9e5431-8f27-4451-ac64-430ffd52b926" />

- Kubernetes detects missing pod
- Creates new pod automatically
- Pod name will be DIFFERENT
- Deployment creates new pod
- Not restoring old one

---

### Task 5: Scale the Deployment
Change the number of replicas:

```bash
# Scale up to 5
kubectl scale deployment nginx-deployment --replicas=5 -n dev
kubectl get pods -n dev

# Scale down to 2
kubectl scale deployment nginx-deployment --replicas=2 -n dev
kubectl get pods -n dev
```

Watch how Kubernetes creates or terminates pods to match the desired count.

You can also scale by editing the manifest — change `replicas: 4` in your YAML file and run `kubectl apply -f nginx-deployment.yaml` again.

**Verify:** When you scaled down from 5 to 2, what happened to the extra pods?

---

number of pods increased from 2 to 5:

<img width="2454" height="676" alt="image" src="https://github.com/user-attachments/assets/670bb617-3e8d-448e-a0c9-93bbdaaa0077" />

back to 2:

<img width="2486" height="306" alt="image" src="https://github.com/user-attachments/assets/7df6b64a-d9ed-49ff-b8fd-3d60afdfd7a8" />


- Extra pods are TERMINATED
- Important :: Kubernetes always tries to match desired stat


---

### Task 6: Rolling Update
Update the Nginx image version to trigger a rolling update:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev
```

Watch the rollout in real time:
```bash
kubectl rollout status deployment/nginx-deployment -n dev
```

Kubernetes replaces pods one by one — old pods are terminated only after new ones are healthy. This means zero downtime.

Check the rollout history:
```bash
kubectl rollout history deployment/nginx-deployment -n dev
```

Now roll back to the previous version:
```bash
kubectl rollout undo deployment/nginx-deployment -n dev
kubectl rollout status deployment/nginx-deployment -n dev
```

Verify the image is back to the previous version:
```bash
kubectl describe deployment nginx-deployment -n dev | grep Image
```

**Verify:** What image version is running after the rollback?

---


<img width="2610" height="808" alt="image" src="https://github.com/user-attachments/assets/7d92a86f-63d3-4ff2-8d3b-f3c0fab9f809" />

<img width="2862" height="498" alt="image" src="https://github.com/user-attachments/assets/7b07e444-ca16-4518-9a08-4b551f39efc1" />

- New pods start
- Old pods terminate gradually
- Zero downtime


---

### Task 7: Clean Up
```bash
kubectl delete deployment nginx-deployment -n dev
kubectl delete pod nginx-dev -n dev
kubectl delete pod nginx-staging -n staging
kubectl delete namespace dev staging production
```

Deleting a namespace removes everything inside it. Be very careful with this in production.

```bash
kubectl get namespaces
kubectl get pods -A
```

**Verify:** Are all your resources gone?

---

All resources deleted:

<img width="2600" height="1156" alt="image" src="https://github.com/user-attachments/assets/9e6131d5-f8a0-4396-915a-dbec805e733b" />

Deleting namespace = deletes EVERYTHING inside

## Pod vs Deployment

| Feature            | Pod | Deployment |
|------------------|-----|-----------|
| Auto heal        | ❌  | ✅        |
| Scaling          | ❌  | ✅        |
| Rolling update   | ❌  | ✅        |
| Production ready | ❌  | ✅        |
