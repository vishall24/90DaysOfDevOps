# Day 58 – Metrics Server and Horizontal Pod Autoscaler (HPA)

## Task
Yesterday you set resource requests and limits. Today you put that to work. Install the Metrics Server so Kubernetes can see actual resource usage, then set up a Horizontal Pod Autoscaler that scales your app up under load and back down when things calm down.


### CONCEPT:

Yesterday (Day 57) — You set resource requests and limits (what a pod is allowed to use).
Today (Day 58) — You make Kubernetes actually watch those resources and automatically add or remove pods based on real traffic.
Two things:

Metrics Server — Collects real-time CPU/memory data from all nodes and pods. Without this, Kubernetes is blind to actual usage.
HPA (Horizontal Pod Autoscaler) — Watches CPU/memory usage and automatically scales pods up or down based on rules you define.

Real world example:

9 AM → low traffic → 1 pod running
12 PM → lunch rush → HPA scales to 8 pods automatically
3 PM → traffic drops → HPA scales back to 1 pod automatically

You pay less, your app never crashes. That's HPA.

---

## Challenge Tasks

### Task 1: Install the Metrics Server
1. Check if it is already running: `kubectl get pods -n kube-system | grep metrics-server`
2. If not, install it:
   - Minikube: `minikube addons enable metrics-server`
   - Kind/kubeadm: apply the official manifest from the metrics-server GitHub releases
3. On local clusters, you may need the `--kubelet-insecure-tls` flag (never in production)
4. Wait 60 seconds, then verify: `kubectl top nodes` and `kubectl top pods -A`

**Verify:** What is the current CPU and memory usage of your node?

---

CONCEPT:

Concept First

'Metrics Server' sits inside your cluster and constantly polls each node's kubelet every 15 seconds asking "how much CPU and memory are 
your pods using right now?" Without it, kubectl top doesn't work and HPA can't function.

configured metrics server:

<img width="1431" height="390" alt="image" src="https://github.com/user-attachments/assets/780d9d4d-6844-4e11-abc7-db7b4ee6aaa1" />

checked:

<img width="1049" height="484" alt="image" src="https://github.com/user-attachments/assets/cf36c586-6219-4132-9e6a-0791dd4e31f7" />

---

### Task 2: Explore kubectl top
1. Run `kubectl top nodes`, `kubectl top pods -A`, `kubectl top pods -A --sort-by=cpu`
2. `kubectl top` shows real-time usage, not requests or limits — these are different things
3. Data comes from the Metrics Server, which polls kubelets every 15 seconds

**Verify:** Which pod is using the most CPU right now?

---

Concept First:

kubectl top shows actual real-time usage.
kubectl describe pod shows configured requests/limits.
These are completely different things:

A pod might request 200m CPU but only use 5m right now
HPA works on actual usage, not requests

Checked node usage, Checked all pods sorted by CPU, Checked all pods sorted by memory:

<img width="845" height="701" alt="image" src="https://github.com/user-attachments/assets/7471d7ac-adcd-4834-bc91-032fdec9a60a" />

---

### Task 3: Create a Deployment with CPU Requests
1. Write a Deployment manifest using the `registry.k8s.io/hpa-example` image (a CPU-intensive PHP-Apache server)
2. Set `resources.requests.cpu: 200m` — HPA needs this to calculate utilization percentages
3. Expose it as a Service: `kubectl expose deployment php-apache --port=80`

Without CPU requests, HPA cannot work — this is the most common HPA setup mistake.

**Verify:** What is the current CPU usage of the Pod?

---

IMPORTANT:

Concept First:

HPA calculates scaling using this formula:

    desiredReplicas = ceil(currentReplicas × (currentUsage / targetUsage))

Example:

1 pod running, using 100m CPU
Request is 200m, target is 50% = 100m

    100/100 = 1.0 → stay at 1 replica

But if load increases to 300m:

    300/100 = 3.0 → scale to 3 replicas

This only works if resources.requests is set. Without it HPA shows <unknown> and never scales.

Created deployment-php.yaml:

<img width="814" height="596" alt="image" src="https://github.com/user-attachments/assets/9c1a8cb1-436b-4446-8446-2032ae942b79" />

<img width="759" height="160" alt="image" src="https://github.com/user-attachments/assets/756ae09c-48d5-4594-aad0-4f91722b1e91" />

1m-5m since no traffic is hitting it yet.

Pod is running and CPU usage is very low at idle.

---

### Task 4: Create an HPA (Imperative)
1. Run: `kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`
2. Check: `kubectl get hpa` and `kubectl describe hpa php-apache`
3. TARGETS may show `<unknown>` initially — wait 30 seconds for metrics to arrive

This scales up when average CPU exceeds 50% of requests, and down when it drops below.

**Verify:** What does the TARGETS column show?

---

Concept First

This command creates an HPA that says:

Keep average CPU below 50% of requests (50% of 200m = 100m)
Never go below 1 replica
Never go above 10 replicas

1%/50% means: currently using 1% of requests, target is 50%.:

<img width="964" height="271" alt="image" src="https://github.com/user-attachments/assets/20b70c1a-52df-41a6-a8f9-462a7e980525" />

kubectl describe hpa php-apache:

<img width="1425" height="465" alt="image" src="https://github.com/user-attachments/assets/f61f84d3-bcec-4ef7-b15b-1e54f4e5ad39" />

---

### Task 5: Generate Load and Watch Autoscaling
1. Start a load generator: `kubectl run load-generator --image=busybox:1.36 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"`
2. Watch HPA: `kubectl get hpa php-apache --watch`
3. Over 1-3 minutes, CPU climbs above 50%, replicas increase, CPU stabilizes
4. Stop the load: `kubectl delete pod load-generator`
5. Scale-down is slow (5-minute stabilization window) — you do not need to wait

**Verify:** How many replicas did HPA scale to under load?

---

Concept First:

This is the most exciting task — you'll watch the HPA react to real traffic in real time. We'll run a pod that hammers the php-apache 
service with requests in a loop, watch CPU climb above 50%, and see HPA automatically add more pods.

created a pod which will send traffic to pod:

<img width="1431" height="63" alt="image" src="https://github.com/user-attachments/assets/1d79cb62-8cb8-4a51-ada7-617f1aaf330a" />

watched the pod getting traffic and HPA is autoscaling based on traffic

<img width="990" height="698" alt="image" src="https://github.com/user-attachments/assets/fabe741c-0ac4-49fd-b36c-ea637b2c567f" />

<img width="731" height="345" alt="image" src="https://github.com/user-attachments/assets/b597aa97-0f11-4b16-896c-27ea29d8dc5e" />

After deleting the load-generator pod:

<img width="1001" height="295" alt="image" src="https://github.com/user-attachments/assets/097b1161-7ad9-4c50-a1a2-9bac0bf5a7a8" />

Scale-down is intentionally slow (5 minute stabilization window) to prevent flapping. You don't need to wait — just note that it will
eventually go back to 1 replica.

---

### Task 6: Create an HPA from YAML (Declarative)
1. Delete the imperative HPA: `kubectl delete hpa php-apache`
2. Write an HPA manifest using `autoscaling/v2` API with CPU target at 50% utilization
3. Add a `behavior` section to control scale-up speed (no stabilization) and scale-down speed (300 second window)
4. Apply and verify with `kubectl describe hpa`

`autoscaling/v2` supports multiple metrics and fine-grained scaling behavior that the imperative command cannot configure.

**Verify:** What does the `behavior` section control?

---

Concept First:

| Feature | autoscaling/v1 | autoscaling/v2 |
|---|---|---|
| Metrics | CPU only | CPU + Memory + Custom |
| Behavior control | ❌ No | ✅ Yes |
| Multiple metrics | ❌ No | ✅ Yes |

The behavior section in v2 lets you control:

Scale-up — how fast to add pods (aggressive or conservative)
Scale-down — how fast to remove pods (stabilization window prevents flapping)


<img width="952" height="574" alt="image" src="https://github.com/user-attachments/assets/1dfa10e4-5b31-49c7-aba1-b5d538959ec5" />

Look for the Behavior section — it shows your scale-up and scale-down rules clearly.
Answer: What does the behavior section control?

scaleUp — how aggressively to add pods when load increases
scaleDown — how slowly to remove pods when load decreases (prevents flapping)

---

## What is the Metrics Server?
Metrics Server collects real-time CPU and memory usage from all nodes
and pods by polling kubelets every 15 seconds. It powers both
`kubectl top` and HPA. Without it, Kubernetes has no visibility
into actual resource usage.

## How HPA Calculates Desired Replicas

Formula:
desiredReplicas = ceil(currentReplicas × (currentUsage / targetUsage))

Example:
- 1 pod running, request = 200m CPU, target = 50% = 100m
- Current usage = 300m
- 1 × (300 / 100) = 3 → HPA scales to 3 replicas

## autoscaling/v1 vs autoscaling/v2

| Feature | autoscaling/v1 | autoscaling/v2 |
|---|---|---|
| CPU scaling | ✅ Yes | ✅ Yes |
| Memory scaling | ❌ No | ✅ Yes |
| Custom metrics | ❌ No | ✅ Yes |
| Behavior control | ❌ No | ✅ Yes |

## The behavior Section
Controls how fast HPA reacts:
- scaleUp.stabilizationWindowSeconds: 0 = scale up immediately
- scaleDown.stabilizationWindowSeconds: 300 = wait 5 min before scaling down
- Prevents flapping (rapidly scaling up and down)

## Key Rules
- HPA requires resources.requests — without it TARGETS shows unknown
- kubectl top = actual usage. kubectl describe pod = configured limits
- Scale-up is fast. Scale-down has a 5-minute stabilization window by default
- HPA works with Deployments, StatefulSets, and ReplicaSets

---

| Concept | What it means |
|---|---|
| Metrics Server | Collects real-time CPU/memory from nodes every 15s — powers kubectl top and HPA |
| `kubectl top` | Shows actual current usage (not requests/limits) |
| HPA | Automatically scales pod count based on CPU/memory usage |
| `resources.requests` | REQUIRED for HPA — without it TARGETS shows `<unknown>` |
| HPA formula | `ceil(currentReplicas × currentUsage / targetUsage)` |
| Scale-up | Fast — happens within seconds of threshold being crossed |
| Scale-down | Slow — 5 minute stabilization window by default to prevent flapping |
| `autoscaling/v1` | CPU scaling only |
| `autoscaling/v2` | CPU + memory + custom metrics + behavior control |
| `behavior` section | Controls how aggressively HPA scales up and how slowly it scales down |
