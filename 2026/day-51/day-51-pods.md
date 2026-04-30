# Day 51 – Kubernetes Manifests and Your First Pods

---

## The Anatomy of a Kubernetes Manifest

Every Kubernetes resource is defined using a YAML manifest with four required top-level fields:

```yaml
apiVersion: v1          # Which API version to use
kind: Pod               # What type of resource
metadata:               # Name, labels, namespace
  name: my-pod
  labels:
    app: my-app
spec:                   # The actual specification (what you want)
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

- `apiVersion` — tells Kubernetes which API group to use. For Pods, it is `v1`.
- `kind` — the resource type. Today it is `Pod`. Later you will use `Deployment`, `Service`, etc.
- `metadata` — the identity of your resource. `name` is required. `labels` are key-value pairs used for organization and selection.
- `spec` — the desired state. For a Pod, this means which containers to run, which images, which ports, etc.

---


<img width="1206" height="216" alt="image" src="https://github.com/user-attachments/assets/b62c004d-32a8-4a56-819b-9a14ab2cceba" />

---

### Task 2: Create a Custom Pod (BusyBox)
Write a new manifest `busybox-pod.yaml` from scratch (do not copy-paste the nginx one):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
```

Apply and verify:
```bash
kubectl apply -f busybox-pod.yaml
kubectl get pods
kubectl logs busybox-pod
```

Notice the `command` field — BusyBox does not run a long-lived server like Nginx. Without a command that keeps it running, the container would exit immediately and the pod would go into `CrashLoopBackOff`.

**Verify:** Can you see "Hello from BusyBox" in the logs?

---

Describe Pod:

    kubectl describe pod nginx-pod

 Shows:

    events
    scheduling
    errors
    
Logs

    kubectl logs nginx-pod

  Shows container output

Enter container

    kubectl exec -it nginx-pod -- /bin/bash

(or /bin/sh if bash not available)

Inside:

    curl localhost:80

Pod has its own network
localhost = inside container

BusyBox Pod:

<img width="1328" height="140" alt="image" src="https://github.com/user-attachments/assets/50e7290b-7dea-4ff2-80d9-ad5b06cb147a" />

Without this:: sleep 3600 :-

 container exits
 pod → CrashLoopBackOff


---

### Task 3: Imperative vs Declarative
You have been using the declarative approach (writing YAML, then `kubectl apply`). Kubernetes also supports imperative commands:

```bash
# Create a pod without a YAML file
kubectl run redis-pod --image=redis:latest

# Check it
kubectl get pods
```

Now extract the YAML that Kubernetes generated:
```bash
kubectl get pod redis-pod -o yaml
```

Compare this output with your hand-written manifests. Notice how much extra metadata Kubernetes adds automatically (status, timestamps, uid, resource version).

You can also use dry-run to generate YAML without creating anything:
```bash
kubectl run test-pod --image=nginx --dry-run=client -o yaml
```

This is a powerful trick — use it to quickly scaffold a manifest, then customize it.

**Verify:** Save the dry-run output to a file and compare its structure with your nginx-pod.yaml. What fields are the same? What is different?

---

Imperative (quick command):

    kubectl run redis-pod --image=redis:latest

 Fast
 NOT recommended in real projects

Declarative (YAML):

    kubectl apply -f file.yaml

  version control
  repeatable
  industry standard


kubectl get pod redis-pod -o yaml:

<img width="2356" height="1442" alt="image" src="https://github.com/user-attachments/assets/d54bc6ab-eefd-48fd-8da9-231dd4c22932" />

    lots of extra fields
    status
    UID
    timestamps

kubectl run test-pod --image=nginx --dry-run=client -o yaml:

<img width="2078" height="672" alt="image" src="https://github.com/user-attachments/assets/241fd43f-5481-4990-92f8-5b23bac11752" />

---

### Task 4: Validate Before Applying
Before applying a manifest, you can validate it:

```bash
# Check if the YAML is valid without actually creating the resource
kubectl apply -f nginx-pod.yaml --dry-run=client

# Validate against the cluster's API (server-side validation)
kubectl apply -f nginx-pod.yaml --dry-run=server
```

Now intentionally break your YAML (remove the `image` field or add an invalid field) and run dry-run again. See what error you get.

**Verify:** What error does Kubernetes give when the image field is missing?

---

Client validation:

    kubectl apply -f nginx-pod.yaml --dry-run=client

Server validation:
    
    kubectl apply -f nginx-pod.yaml --dry-run=server

Tried breaking YAML:

got error like:

required field missing

---

### Task 5: Pod Labels and Filtering
Labels are how Kubernetes organizes and selects resources. You added labels in your manifests — now use them:

```bash
# List all pods with their labels
kubectl get pods --show-labels

# Filter pods by label
kubectl get pods -l app=nginx
kubectl get pods -l environment=dev

# Add a label to an existing pod
kubectl label pod nginx-pod environment=production

# Verify
kubectl get pods --show-labels

# Remove a label
kubectl label pod nginx-pod environment-
```

Write a manifest for a third pod with at least 3 labels (app, environment, team). Apply it and practice filtering.

---

<img width="1884" height="362" alt="image" src="https://github.com/user-attachments/assets/7134ca71-1550-41da-95ed-8dcb2f112da7" />


<img width="1844" height="188" alt="image" src="https://github.com/user-attachments/assets/65943686-2e11-4f34-a4ce-2936bdff48eb" />


<img width="2246" height="366" alt="image" src="https://github.com/user-attachments/assets/25a28e80-efb3-4964-b0c9-71216c8ce46b" />


<img width="2044" height="146" alt="image" src="https://github.com/user-attachments/assets/2c85fd21-4ac8-4b3f-80cc-f14efe8949af" />


WHY labels matter

  Services use labels to connect
  Deployments use labels to manage pods

Custom pod:

<img width="466" height="594" alt="image" src="https://github.com/user-attachments/assets/409dfe8e-588e-4b22-a3cc-ddf4c611f846" />


---

### Task 6: Clean Up
Delete all the pods you created:

```bash
# Delete by name
kubectl delete pod nginx-pod
kubectl delete pod busybox-pod
kubectl delete pod redis-pod

# Or delete using the manifest file
kubectl delete -f nginx-pod.yaml

# Verify everything is gone
kubectl get pods
```

Notice that when you delete a standalone Pod, it is gone forever. There is no controller to recreate it. This is why in production you use Deployments (coming on Day 52) instead of bare Pods.

---

Deleted all pods:

<img width="1668" height="264" alt="image" src="https://github.com/user-attachments/assets/6ddd3d58-475a-401b-9ede-748576c8257c" />

