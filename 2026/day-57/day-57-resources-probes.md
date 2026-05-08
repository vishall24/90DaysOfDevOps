# Day 57 – Resource Requests, Limits, and Probes

## Task
Your Pods are running, but Kubernetes has no idea how much CPU or memory they need — and no way to tell if they are actually healthy. Today you set resource requests and limits for smart scheduling, then add probes so Kubernetes can detect and recover from failures automatically.

---

## Challenge Tasks

### Task 1: Resource Requests and Limits
1. Write a Pod manifest with `resources.requests` (cpu: 100m, memory: 128Mi) and `resources.limits` (cpu: 250m, memory: 256Mi)
2. Apply and inspect with `kubectl describe pod` — look for the Requests, Limits, and QoS Class sections
3. Since requests and limits differ, the QoS class is `Burstable`. If equal, it would be `Guaranteed`. If missing, `BestEffort`.

CPU is in millicores: `100m` = 0.1 CPU. Memory is in mebibytes: `128Mi`.

**Requests** = guaranteed minimum (scheduler uses this for placement). **Limits** = maximum allowed (kubelet enforces at runtime).

**Verify:** What QoS class does your Pod have?

---
CONCEPTS:

CPU units:

1 = 1 full CPU core
100m = 100 millicores = 0.1 CPU
500m = 500 millicores = 0.5 CPU

QoS Classes (Quality of Service):

ClassCondition:

Guaranteed  | requests == limits for all containers
Burstable   | requests < limits (what we're doing)
BestEffort  | no requests or limits set at all

created and applied pod-recources.yaml:

<img width="943" height="400" alt="image" src="https://github.com/user-attachments/assets/2849c699-0aeb-4543-8e66-5aeaadebbf4e" />

executed:

kubectl describe pod pod-resources:

<img width="606" height="564" alt="image" src="https://github.com/user-attachments/assets/b5777b47-eb59-4d8e-bd0f-acc4e64222a6" />

Qos class is burstable, means requests < limits.

---

### Task 2: OOMKilled — Exceeding Memory Limits
1. Write a Pod manifest using the `polinux/stress` image with a memory limit of `100Mi`
2. Set the stress command to allocate 200M of memory: `command: ["stress"] args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]`
3. Apply and watch — the container gets killed immediately

CPU is throttled when over limit. Memory is killed — no mercy.

Check `kubectl describe pod` for `Reason: OOMKilled` and `Exit Code: 137` (128 + SIGKILL).

**Verify:** What exit code does an OOMKilled container have?

---

Concept First

CPU and memory behave very differently when a pod exceeds its limit:

Resource   |  What happens when limit exceeded
CPU        |  Throttled — slowed down, not killed
Memory     |  OOMKilled — container is immediately killed

OOM = Out Of Memory. Exit code 137 = 128 + SIGKILL signal.

We're going to deliberately make a pod ask for more memory than its limit allows and watch it get killed.

created pod-oom.yaml:

<img width="805" height="438" alt="image" src="https://github.com/user-attachments/assets/d8faa372-9c88-46ef-9652-1bb35803d9b6" />

applied and error:

<img width="866" height="226" alt="image" src="https://github.com/user-attachments/assets/be79dcb9-e5d4-4361-b5cd-c065eb9c0249" />

kubectl describe pod pod-oom:

<img width="579" height="274" alt="image" src="https://github.com/user-attachments/assets/244d659f-1f98-4bbe-a408-73740dfd1743" />

---

### Task 3: Pending Pod — Requesting Too Much
1. Write a Pod manifest requesting `cpu: 100` and `memory: 128Gi`
2. Apply and check — STATUS stays `Pending` forever
3. Run `kubectl describe pod` and read the Events — the scheduler says exactly why: insufficient resources

**Verify:** What event message does the scheduler produce?

---

Concept First:

If you request more resources than ANY node in your cluster has, the scheduler can't place the pod anywhere. It stays Pending forever — the scheduler tells you exactly why.

created and applied pod-pending.yaml and checked the status:

<img width="853" height="490" alt="image" src="https://github.com/user-attachments/assets/8e7adc52-743a-4d01-a9a0-0948b0a5bbc5" />

Status will be Pending and stay that way forever.

kubectl describe pod pod-pending:

<img width="1432" height="231" alt="image" src="https://github.com/user-attachments/assets/32ec4fea-80ca-4e18-a7d9-b6c0a616af0b" />

Scheduler event says Insufficient cpu and Insufficient memory.

---

### Task 4: Liveness Probe
A liveness probe detects stuck containers. If it fails, Kubernetes restarts the container.

1. Write a Pod manifest with a busybox container that creates `/tmp/healthy` on startup, then deletes it after 30 seconds
2. Add a liveness probe using `exec` that runs `cat /tmp/healthy`, with `periodSeconds: 5` and `failureThreshold: 3`
3. After the file is deleted, 3 consecutive failures trigger a restart. Watch with `kubectl get pod -w`

**Verify:** How many times has the container restarted?

---

Concept First:

Liveness probe = "Is this container still alive and working?"
If the liveness probe fails X times in a row → Kubernetes restarts the container.

Use case: A container that gets stuck in a deadlock. It's "running" but not doing anything useful. Liveness probe detects this and restarts it.

We'll simulate this by:

1) Creating a file /tmp/healthy on startup
2) Deleting it after 30 seconds
3) The liveness probe checks for this file every 5 seconds
4) After 3 failures (15 seconds) → container restarts

Created pod-liveness.yaml:

<img width="793" height="558" alt="image" src="https://github.com/user-attachments/assets/5e756883-87b6-480a-a8f6-85f585ad5118" />

checked and the pod got restarted:

<img width="927" height="162" alt="image" src="https://github.com/user-attachments/assets/8b00b81a-6661-47c5-9468-006b2470761c" />

kubectl describe pod pod-liveness:

<img width="1423" height="674" alt="image" src="https://github.com/user-attachments/assets/adf424a7-eba0-480f-ba95-246cd297776d" />

Verify: RESTARTS column shows 1 or more after ~45 seconds.

---

### Task 5: Readiness Probe
A readiness probe controls traffic. Failure removes the Pod from Service endpoints but does NOT restart it.

1. Write a Pod manifest with nginx and a `readinessProbe` using `httpGet` on path `/` port `80`
2. Expose it as a Service: `kubectl expose pod <name> --port=80 --name=readiness-svc`
3. Check `kubectl get endpoints readiness-svc` — the Pod IP is listed
4. Break the probe: `kubectl exec <pod> -- rm /usr/share/nginx/html/index.html`
5. Wait 15 seconds — Pod shows `0/1` READY, endpoints are empty, but the container is NOT restarted

**Verify:** When readiness failed, was the container restarted?

---

Concept First:

Readiness probe = "Is this container ready to receive traffic?"

If readiness fails → Pod is removed from Service endpoints (no traffic sent to it) but the container is NOT restarted.

| | Liveness | Readiness |
|---|---|---|
| Failure action | Restart container | Remove from endpoints |
| Use case | Detect stuck/crashed app | Detect app not ready for traffic |

Created applied and checked the pod: pod-readiness.yaml:

<img width="867" height="467" alt="image" src="https://github.com/user-attachments/assets/11aecdb7-ffe8-473d-8daf-71e25e3f9fa9" />

Exposed it as a Service, 
Checked endpoints — the pod's IP listed — traffic is flowing to it, 
Broke the readiness probe,
Waited for 15 sec then checked, 
saw 0/1 in the READY column — pod is running but NOT ready.
Endpoints will be empty — no traffic goes to this pod now.

<img width="1156" height="343" alt="image" src="https://github.com/user-attachments/assets/3f0b0bef-11b8-41a1-8601-e5e469c0b500" />

kubectl describe pod pod-readiness:

<img width="1437" height="688" alt="image" src="https://github.com/user-attachments/assets/a1272373-c3aa-44ae-badf-224caabb8062" />

Container was NOT restarted (RESTARTS = 0). Only removed from endpoints.

---

### Task 6: Startup Probe
A startup probe gives slow-starting containers extra time. While it runs, liveness and readiness probes are disabled.

1. Write a Pod manifest where the container takes 20 seconds to start (e.g., `sleep 20 && touch /tmp/started`)
2. Add a `startupProbe` checking for `/tmp/started` with `periodSeconds: 5` and `failureThreshold: 12` (60 second budget)
3. Add a `livenessProbe` that checks the same file — it only kicks in after startup succeeds

**Verify:** What would happen if `failureThreshold` were 2 instead of 12?

---

### Concept First: 

Some apps take a long time to start (Java apps, large databases). If liveness probe starts checking immediately, it'll kill the container before it even finishes starting!

Startup probe = "Has the app finished starting yet?"

While startup probe is running → liveness and readiness are paused
Once startup probe passes → liveness and readiness take over
If startup probe fails too many times → container is killed

Think of it like this:

Startup probe = "Are you awake yet?" (asked once during bootup)
Liveness probe = "Are you still alive?" (asked forever after)

Created pod-startup.yaml:

<img width="733" height="639" alt="image" src="https://github.com/user-attachments/assets/cd4e3a5a-ee01-4435-9d2d-2fed68ea4eeb" />

checked the pod:

<img width="805" height="130" alt="image" src="https://github.com/user-attachments/assets/ffb140b1-a23f-4cf6-8cdd-d66b81768bc7" />

saw it stay 0/1 for ~20 seconds while it "starts", then flip to 1/1 Running

kubectl describe pod pod-startup:

<img width="1437" height="321" alt="image" src="https://github.com/user-attachments/assets/781eaa86-2720-46ee-8f67-3f246af855a3" />

saw startup probe failures in the early events (while sleeping 20s), then it passes and liveness takes over.

### What if failureThreshold were 2 instead of 12?
Budget would be only 2 × 5s = 10 seconds. But our app takes 20 seconds to start. So startup probe would fail and Kubernetes 
would kill the container before it even finishes starting — it would loop in CrashLoopBackOff forever.

---

### Task 7: Clean Up
Delete all pods and services you created.

---

Cleaned up:

<img width="1418" height="295" alt="image" src="https://github.com/user-attachments/assets/3f00cb6f-cce6-48e4-a958-1b69ca53e3f7" />


---

## Requests vs Limits

| | Requests | Limits |
|---|---|---|
| Purpose | Minimum guaranteed resources | Maximum allowed resources |
| Used by | Scheduler (for pod placement) | Kubelet (enforced at runtime) |
| CPU over limit | N/A | Throttled (slowed down) |
| Memory over limit | N/A | OOMKilled (exit code 137) |

## QoS Classes

| Class | Condition |
|---|---|
| Guaranteed | requests == limits for all containers |
| Burstable | requests < limits |
| BestEffort | no requests or limits set |

## What Happens When Limits Are Exceeded

- **CPU**: Compressible — container is throttled, not killed
- **Memory**: Incompressible — container is OOMKilled immediately (exit code 137)

## Liveness vs Readiness vs Startup Probes

| Probe | Purpose | On Failure |
|---|---|---|
| Liveness | Is the container still working? | Restart container |
| Readiness | Is the container ready for traffic? | Remove from Service endpoints |
| Startup | Has the container finished starting? | Kill container (during startup only) |

## Key Probe Settings
- `initialDelaySeconds` — wait before first check
- `periodSeconds` — how often to check
- `failureThreshold` — how many failures before action

## Startup Probe Tip
Always use a startup probe for slow-starting apps.
Budget = failureThreshold × periodSeconds
If budget < actual startup time → container gets killed in a loop

---

| Concept | What it means |
|---|---|
| `requests` | Minimum resources guaranteed — used by scheduler for placement |
| `limits` | Maximum resources allowed — enforced by kubelet at runtime |
| CPU over limit | Throttled — slowed down, never killed |
| Memory over limit | OOMKilled — exit code 137, no warning |
| `Guaranteed` QoS | requests == limits — highest priority, last to be evicted |
| `Burstable` QoS | requests < limits — middle priority |
| `BestEffort` QoS | nothing set — lowest priority, first to be evicted |
| Liveness probe | Fails → container restarts |
| Readiness probe | Fails → removed from endpoints, NOT restarted |
| Startup probe | Pauses liveness + readiness until app finishes starting |
| Exit code 137 | OOMKilled (128 + SIGKILL) |
