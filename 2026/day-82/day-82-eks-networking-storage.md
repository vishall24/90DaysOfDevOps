# Day 82 -- EKS Networking with Gateway API and Persistent Storage

## Task
Your EKS cluster is running and the AI-BankApp deployed with raw manifests. But production needs proper ingress, HTTPS, session persistence, and reliable storage. The AI-BankApp project uses the Kubernetes Gateway API with Envoy Gateway instead of traditional Ingress -- the next generation of Kubernetes traffic management.

Today you set up the Gateway API, configure TLS with cert-manager, understand EBS storage in action, and explore the AI-BankApp's production networking setup.

Reference: https://github.com/TrainWithShubham/AI-BankApp-DevOps (branch `feat/gitops`) -- `k8s/gateway.yml`, `k8s/cert-manager.yml`, `k8s/pv.yml`, `k8s/pvc.yml`

---

What Are We Learning Today?:

Yesterday we deployed the AI-BankApp on EKS. But we used port-forward to access it — not production. Today we add proper networking.

Three Things today:

| Topic | What you're setting up |
|---|---|
| Gateway API | Next-gen Kubernetes ingress — replaces Ingress resource |
| Envoy Gateway | The implementation that handles Gateway API for the AI-BankApp |
| EBS Storage | Deep dive into how persistent volumes actually work on EKS |

Why Gateway API instead of Ingress?:

| Feature | Old Ingress | Gateway API |
|---|---|---|
| Traffic splitting |  Not supported |  Built-in |
| Session affinity |  Not standardized |  BackendTrafficPolicy |
| Role separation | One resource does all | GatewayClass → Gateway → HTTPRoute |
| TLS management | Annotation-based | Native in Gateway listener |
| API status | Stable but limited | GA since K8s 1.26 |

The traffic flow you're building:

    Internet
       ↓
    AWS NLB (created automatically by Envoy Gateway)
       ↓
    Gateway (bankapp-gateway) — listens on port 80 and 443
       ↓
    HTTPRoute (bankapp-route) — routes to the BankApp service
       ↓
    Service (bankapp-service:8080)
       ↓
    BankApp Pods (with cookie session affinity)

---

## Challenge Tasks

### Task 1: Understand Gateway API vs Ingress
The AI-BankApp uses the Gateway API instead of the traditional Ingress resource. Research the differences:

| Feature | Ingress | Gateway API |
|---------|---------|-------------|
| API maturity | Stable but limited | GA since Kubernetes 1.26 |
| Traffic splitting | Not supported | Built-in (weighted backends) |
| Header matching | Annotation-dependent | Native HTTPRoute rules |
| Role separation | Single resource | GatewayClass (infra) -> Gateway (ops) -> HTTPRoute (dev) |
| TLS management | Annotation-based | Native TLS config in Gateway listeners |
| Session affinity | Not standardized | BackendTrafficPolicy (with Envoy) |

**The AI-BankApp's Gateway architecture:**
```
[Internet]
    |
[AWS NLB] (created by Envoy Gateway)
    |
[Gateway: bankapp-gateway]
  |-- Listener: HTTP (port 80)
  |-- Listener: HTTPS (port 443, TLS terminated)
    |
[HTTPRoute: bankapp-route]
    |
[Service: bankapp-service:8080]
    |
[Pods: bankapp x2-4] (with session affinity via cookie)
```

---

Role separation is the key innovation:

    GatewayClass → Infrastructure team manages
       "Which controller handles our gateways?"
    
    Gateway → Platform/Ops team manages
       "Create a load balancer on port 80 and 443"
    
    HTTPRoute → Developer team manages
       "Route /api/* to my backend service"

Why cookie session affinity for AI-BankApp?:

The Spring Boot BankApp uses Spring Security with form-based login. Your session is stored in the server's memory. 

Without session affinity:

    Login request → Pod 1 (session created here)
    Next request  → Pod 2 (no session! logged out!)

With cookie affinity:

    Login request  → Pod 1 (creates BANKAPP_AFFINITY cookie)
    Next request   → Pod 1 (cookie routes you back to same pod)
    Next request   → Pod 1 (always same pod, never logged out)

---

### Task 2: Install Envoy Gateway
Envoy Gateway is the Gateway API implementation the AI-BankApp uses.

Install via Helm:
```bash
helm install envoy-gateway oci://docker.io/envoyproxy/gateway-helm \
  --version v1.4.0 \
  -n envoy-gateway-system --create-namespace \
  --wait
```

Verify:
```bash
kubectl get pods -n envoy-gateway-system
kubectl get gatewayclass
```

You should see the `envoy-gateway` GatewayClass registered.

Now install the Gateway API CRDs if not already present:
```bash
kubectl get crd gateways.gateway.networking.k8s.io 2>/dev/null || \
  kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.1/standard-install.yaml
```

---

Concept First:

Envoy Gateway is what actually reads your Gateway API resources and creates AWS infrastructure. When you apply a Gateway resource, 
Envoy Gateway sees it and creates an AWS NLB automatically.

<img width="918" height="142" alt="image" src="https://github.com/user-attachments/assets/7fe55016-cab7-4178-a67c-f775048b03bb" />

<img width="1291" height="676" alt="image" src="https://github.com/user-attachments/assets/f617eb2e-c615-4fec-b681-74ff93c51bbb" />

<img width="865" height="70" alt="image" src="https://github.com/user-attachments/assets/b3db3f08-aea0-4b17-a341-35f005e2f555" />

Created gatewayclass.yaml:

<img width="674" height="140" alt="image" src="https://github.com/user-attachments/assets/ea5b525c-8186-4294-8efa-f1279562de58" />

<img width="817" height="56" alt="image" src="https://github.com/user-attachments/assets/ac7e7932-da54-4a23-8096-d0fc5226000b" />

Helm chart we installed does not create a GatewayClass. Why? ::

Starting with Envoy Gateway v1.4, the Helm chart was changed.
The chart only installs the controller.
It does not automatically create:
- GatewayClass
- Gateway
- HTTPRoute

Those are expected to be created by the user.

<img width="842" height="130" alt="image" src="https://github.com/user-attachments/assets/1571b93e-429b-4ca5-978d-97fa66871580" />

<img width="737" height="708" alt="image" src="https://github.com/user-attachments/assets/e8e315e8-12df-46e3-8d2b-f16c76efd3c0" />

---

### Task 3: Deploy the AI-BankApp with Gateway API
Make sure the app is deployed (from Day 81):
```bash
kubectl get pods -n bankapp
```

If not running, redeploy the core manifests:
```bash
cd AI-BankApp-DevOps
kubectl apply -f k8s/namespace.yml
kubectl apply -f k8s/pv.yml
kubectl apply -f k8s/pvc.yml
kubectl apply -f k8s/configmap.yml
kubectl apply -f k8s/secrets.yml
kubectl apply -f k8s/mysql-deployment.yml
kubectl apply -f k8s/service.yml
kubectl apply -f k8s/ollama-deployment.yml
kubectl apply -f k8s/bankapp-deployment.yml
kubectl apply -f k8s/hpa.yml
```

**Now study and apply the Gateway configuration.**

Open `k8s/gateway.yml` and understand each resource:

**1. GatewayClass** -- defines which controller handles Gateways:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy-gateway
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
```

**2. Gateway** -- creates the actual load balancer with listeners:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: bankapp-gateway
  namespace: bankapp
spec:
  gatewayClassName: envoy-gateway
  listeners:
    - name: http
      protocol: HTTP
      port: 80
    - name: https
      protocol: HTTPS
      port: 443
      hostname: <your-ip>.nip.io
      tls:
        mode: Terminate
        certificateRefs:
          - name: bankapp-tls
```

When this is applied, Envoy Gateway creates an AWS NLB (Network Load Balancer) automatically.

**3. HTTPRoute** -- routes traffic to the BankApp service:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bankapp-route
  namespace: bankapp
spec:
  parentRefs:
    - name: bankapp-gateway
      sectionName: https
    - name: bankapp-gateway
      sectionName: http
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: bankapp-service
          port: 8080
```

**4. BackendTrafficPolicy** -- session persistence via cookies:
```yaml
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: BackendTrafficPolicy
metadata:
  name: bankapp-session
  namespace: bankapp
spec:
  targetRefs:
    - group: gateway.networking.k8s.io
      kind: HTTPRoute
      name: bankapp-route
  loadBalancer:
    type: ConsistentHash
    consistentHash:
      type: Cookie
      cookie:
        name: BANKAPP_AFFINITY
        ttl: 3600s
```

**Why cookie-based session affinity?** The AI-BankApp uses Spring Security with form-based login. Without session affinity, a user's requests could hit different pods, and they would be logged out. The `BANKAPP_AFFINITY` cookie ensures all requests from a user go to the same pod.

Apply the Gateway configuration:
```bash
kubectl apply -f k8s/gateway.yml
```

Wait for the NLB to be provisioned:
```bash
kubectl get gateway -n bankapp -w
```

Get the external IP:
```bash
export GATEWAY_IP=$(kubectl get gateway bankapp-gateway -n bankapp -o jsonpath='{.status.addresses[0].value}')
echo "App URL: http://$GATEWAY_IP"
```

Test access:
```bash
curl http://$GATEWAY_IP
```

---

we'll see 4 resources. Let me explain each:

Resource 1 — GatewayClass:

    apiVersion: gateway.networking.k8s.io/v1
    kind: GatewayClass
    metadata:
      name: envoy-gateway
    spec:
      controllerName: gateway.envoyproxy.io/gatewayclass-controller
  
Tells Kubernetes: "Envoy Gateway handles all Gateways of this class."

Resource 2 — Gateway:

    apiVersion: gateway.networking.k8s.io/v1
    kind: Gateway
    metadata:
      name: bankapp-gateway
      namespace: bankapp
    spec:
      gatewayClassName: envoy-gateway
      listeners:
        - name: http
          protocol: HTTP
          port: 80
        - name: https
          protocol: HTTPS
          port: 443
          tls:
            mode: Terminate
            certificateRefs:
              - name: bankapp-tls
          
When applied → Envoy Gateway creates an AWS NLB automatically. The Gateway = your load balancer definition.

Resource 3 — HTTPRoute:

    apiVersion: gateway.networking.k8s.io/v1
    kind: HTTPRoute
    metadata:
      name: bankapp-route
      namespace: bankapp
    spec:
      parentRefs:
        - name: bankapp-gateway
      rules:
        - matches:
            - path:
                type: PathPrefix
                value: /
          backendRefs:
            - name: bankapp-service
              port: 8080
              
Routes all traffic (/) to the BankApp service on port 8080.

Resource 4 — BackendTrafficPolicy (session affinity):

    apiVersion: gateway.envoyproxy.io/v1alpha1
    kind: BackendTrafficPolicy
    metadata:
      name: bankapp-session
      namespace: bankapp
    spec:
      targetRefs:
        - kind: HTTPRoute
          name: bankapp-route
      loadBalancer:
        type: ConsistentHash
        consistentHash:
          type: Cookie
          cookie:
            name: BANKAPP_AFFINITY
            ttl: 3600s
            
Creates a cookie that pins each user to the same pod for 1 hour.

appiled the gateway.yaml file:

<img width="713" height="116" alt="image" src="https://github.com/user-attachments/assets/d29f61e5-5d8e-4467-aa2d-268dd49c3e38" />

<img width="1139" height="303" alt="image" src="https://github.com/user-attachments/assets/fe6c7d9d-3336-4cb0-91e9-7fe3eec9ec10" />

<img width="1371" height="699" alt="image" src="https://github.com/user-attachments/assets/b5e85d48-2785-4d3f-9a04-2e4336369beb" />

<img width="1440" height="776" alt="image" src="https://github.com/user-attachments/assets/d44c352c-9f63-4f62-ae31-1c291474c81d" />

<img width="823" height="746" alt="image" src="https://github.com/user-attachments/assets/c84f55a0-0f62-4b2a-ae80-c456f1a9f7eb" />

---

### Task 4: Set Up TLS with cert-manager
The AI-BankApp uses cert-manager with Let's Encrypt for automatic HTTPS certificates.

Install cert-manager:
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  -n cert-manager --create-namespace \
  --set crds.enabled=true \
  --wait
```

Verify:
```bash
kubectl get pods -n cert-manager
```

Study and apply the ClusterIssuer from `k8s/cert-manager.yml`:
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-account-key
    solvers:
      - http01:
          gatewayHTTPRoute:
            parentRefs:
              - group: gateway.networking.k8s.io
                kind: Gateway
                name: bankapp-gateway
                namespace: bankapp
```

**How it works:**
1. cert-manager requests a certificate from Let's Encrypt
2. Let's Encrypt sends an HTTP-01 challenge
3. cert-manager creates a temporary HTTPRoute to respond to the challenge
4. Let's Encrypt verifies and issues the certificate
5. cert-manager stores the certificate in the `bankapp-tls` Secret
6. The Gateway uses this Secret for HTTPS termination

To use this, you need a hostname that points to your NLB IP. The AI-BankApp uses `nip.io` for quick DNS:
```bash
export HOSTNAME="${GATEWAY_IP}.nip.io"
echo "HTTPS URL: https://$HOSTNAME"
```

Update the Gateway hostname and apply:
```bash
# For learning: you can skip TLS and just use HTTP
# For production: update gateway.yml with your hostname and apply cert-manager.yml
```

---

Concept First — How cert-manager works:

    cert-manager requests cert from Let's Encrypt
       ↓
    Let's Encrypt sends HTTP-01 challenge:
    "Prove you control this domain by serving
     this file at http://yourdomain/.well-known/..."
       ↓
    cert-manager creates a temporary HTTPRoute
    to respond to the challenge
       ↓
    Let's Encrypt verifies → issues certificate
       ↓
    cert-manager stores cert in Secret: bankapp-tls
       ↓
    Gateway uses bankapp-tls Secret for HTTPS


<img width="662" height="625" alt="image" src="https://github.com/user-attachments/assets/1a79b98e-8eaf-463c-a29b-15588a0d93eb" />

<img width="595" height="98" alt="image" src="https://github.com/user-attachments/assets/fc3c6478-02ea-4e9f-b37b-debdc9644d23" />

<img width="702" height="97" alt="image" src="https://github.com/user-attachments/assets/719013a5-883f-4819-841f-049485be711e" />

What is nip.io? A free wildcard DNS service. 1.2.3.4.nip.io always resolves to 1.2.3.4. You get a real domain name without buying one. Perfect for TLS testing.

<img width="612" height="353" alt="image" src="https://github.com/user-attachments/assets/7d2cdba1-86a4-4b7c-8687-32cf5fd0461e" />

<img width="1413" height="743" alt="image" src="https://github.com/user-attachments/assets/b3995199-0f4e-4e73-a586-07944841ab17" />

BankApp login page over HTTPS with a valid cert!

<img width="1440" height="788" alt="image" src="https://github.com/user-attachments/assets/54211574-ce36-4be8-a1ee-f350d057daba" />


---

### Task 5: Understand EBS Persistent Storage in Action
The AI-BankApp uses EBS volumes for MySQL (5Gi) and Ollama (10Gi). Study how they work on EKS.

Check the storage setup:
```bash
# StorageClass
kubectl get storageclass gp3

# PVCs
kubectl get pvc -n bankapp

# PVs (dynamically provisioned)
kubectl get pv
```

Output should look like:
```
NAME                      STATUS   VOLUME         CAPACITY   STORAGECLASS
mysql-pvc                 Bound    pvc-abc123...  5Gi        gp3
ollama-pvc                Bound    pvc-def456...  10Gi       gp3
```

**Find the actual EBS volumes in AWS:**
```bash
aws ec2 describe-volumes \
  --filters "Name=tag:kubernetes.io/created-by,Values=ebs.csi.aws.com" \
  --query "Volumes[*].{ID:VolumeId,Size:Size,AZ:AvailabilityZone,State:State}" \
  --output table \
  --region us-west-2
```

**Key EBS concepts on EKS:**
- `WaitForFirstConsumer` -- the volume is created in the same AZ as the pod that claims it
- `ReadWriteOnce` -- EBS can only attach to one node at a time (MySQL and Ollama use Recreate strategy because of this)
- `gp3` -- latest generation SSD, 3000 IOPS baseline, cheaper than gp2
- `allowVolumeExpansion: true` -- you can grow volumes without recreating them

**Test persistence** -- delete the MySQL pod and watch it come back with data intact:
```bash
# Check current MySQL data
kubectl exec -n bankapp deploy/mysql -- mysql -uroot -pTest@123 -e "SHOW DATABASES;"

# Delete the pod
kubectl delete pod -n bankapp -l app=mysql

# Watch it recreate
kubectl get pods -n bankapp -l app=mysql -w

# Verify data survived
kubectl exec -n bankapp deploy/mysql -- mysql -uroot -pTest@123 -e "SHOW DATABASES;"
```

The database is intact because the EBS volume persists independently of the pod.

---

Concept First:

    StorageClass (gp3)
       ↓ defines how volumes are created
    PVC (mysql-pvc: 5Gi)
       ↓ requests storage
    PV (auto-created by EBS CSI driver)
       ↓ the actual volume claim
    EBS Volume (real AWS block storage)
       ↓ attached to EC2 node
    Pod mounts at /var/lib/mysql


Key EBS concepts:

| Concept | Meaning |
|---|---|
| `WaitForFirstConsumer` | Volume created in same AZ as pod — prevents AZ mismatch |
| `ReadWriteOnce` | Only one node can attach this volume at a time |
| `Recreate` strategy | Old pod must die before new one starts (EBS can't attach to 2 nodes) |
| `gp3` | 3000 IOPS SSD, cheaper than gp2, expandable |
| `allowVolumeExpansion` | Can grow volume without recreating it |


Check storage setup:

<img width="1437" height="369" alt="image" src="https://github.com/user-attachments/assets/e3bac51e-f48b-41cb-85e2-7ebd455fcb7a" />

<img width="748" height="618" alt="image" src="https://github.com/user-attachments/assets/cd6c9f04-8977-4d04-9012-ffca15f6eb23" />

bankappdb is still there! The pod died but the EBS volume didn't — data is safe.

### Why not RollingUpdate? 

EBS volumes are ReadWriteOnce — only ONE node can attach them at a time. During a rolling update, the new pod starts before the old one 
dies, which means TWO pods would try to attach the same EBS volume — that would FAIL. Recreate kills the old pod first, then the new 
one starts and attaches the volume safely.


---

### Task 6: Explore HPA and Node Capacity
The AI-BankApp's HPA scales pods between 2 and 4 based on CPU.

```bash
kubectl get hpa -n bankapp
```

Check resource usage across nodes:
```bash
kubectl top nodes
kubectl top pods -n bankapp
```

**Resource budget for the AI-BankApp on 3x t3.medium nodes:**

| Component | CPU Request | Memory Request | Instances |
|-----------|-----------|---------------|-----------|
| BankApp | 250m | 256Mi | 2-4 pods |
| MySQL | 250m | 256Mi | 1 pod |
| Ollama | 900m | 2Gi | 1 pod |
| Init containers | 50m | 32Mi | temporary |
| System pods | ~500m | ~500Mi | per node |
| **Total available** | **6000m (3 nodes)** | **12Gi (3 nodes)** | |

Ollama is the heaviest consumer. If you scale BankApp to 4 pods, total CPU requests reach ~2.9 cores + system overhead.

**Clean up the workload (keep the cluster for Day 83):**
```bash
kubectl delete -f k8s/gateway.yml 2>/dev/null
kubectl delete -f k8s/hpa.yml
kubectl delete -f k8s/bankapp-deployment.yml
kubectl delete -f k8s/ollama-deployment.yml
kubectl delete -f k8s/mysql-deployment.yml
kubectl delete -f k8s/service.yml
kubectl delete -f k8s/secrets.yml
kubectl delete -f k8s/configmap.yml
kubectl delete -f k8s/pvc.yml
kubectl delete -f k8s/pv.yml
kubectl delete -f k8s/namespace.yml
```

---

<img width="1111" height="671" alt="image" src="https://github.com/user-attachments/assets/170a4ffd-2023-464b-b1da-f46e8fefdbdf" />

Understand resource budget:

| Component | CPU Request | Memory Request |
|---|---|---|
| BankApp (2 pods) | 500m | 512Mi |
| MySQL (1 pod) | 250m | 256Mi |
| Ollama (1 pod) | 900m | 2Gi |
| System pods | ~500m | ~500Mi |
| **Total used** | **~2.15 cores** | **~3.3Gi** |
| **Available (3x t3.medium)** | **6 cores** | **12Gi** |

Ollama is the biggest consumer at 900m CPU and 2Gi memory. This is why we needed to increase instance type on Day 81!

---

## Gateway API Architecture

    Internet
    ↓
    AWS NLB (auto-created by Envoy Gateway)
    ↓
    Gateway: bankapp-gateway
    ├── Listener: HTTP :80
    └── Listener: HTTPS :443 (TLS terminated)
    ↓
    HTTPRoute: bankapp-route
    └── / → bankapp-service:8080
    ↓
    Pods: bankapp x2-4
    (with BANKAPP_AFFINITY cookie for session persistence)

## Gateway API vs Ingress

| Feature | Ingress | Gateway API |
|---|---|---|
| Traffic splitting | ❌ | ✅ Built-in |
| Session affinity | ❌ | ✅ BackendTrafficPolicy |
| TLS management | Annotation-based | Native in listener |
| Role separation | One resource | GatewayClass → Gateway → HTTPRoute |
| API status | Stable but limited | GA since K8s 1.26 |

## Gateway API Resources

| Resource | Who manages it | What it does |
|---|---|---|
| GatewayClass | Infrastructure team | Defines which controller handles Gateways |
| Gateway | Platform/Ops team | Creates AWS NLB, defines listeners |
| HTTPRoute | Developer team | Routes paths to backend services |
| BackendTrafficPolicy | Platform team | Session affinity, load balancing |

## Why Cookie Session Affinity?
Spring Security stores sessions in pod memory.
Without affinity: Login on Pod 1 → next request Pod 2 → logged out.
With BANKAPP_AFFINITY cookie: all requests pinned to same pod for 1 hour.

## cert-manager TLS Flow
1. cert-manager requests cert from Let's Encrypt
2. Let's Encrypt sends HTTP-01 challenge
3. cert-manager creates temp HTTPRoute to respond
4. Let's Encrypt verifies → issues certificate
5. Stored in Secret: bankapp-tls
6. Gateway uses Secret for HTTPS termination
7. nip.io: free wildcard DNS (1.2.3.4.nip.io resolves to 1.2.3.4)

## EBS Storage Flow
StorageClass (gp3) → PVC (request) → PV (auto-provisioned) → EBS Volume → Pod

## Key EBS Concepts

| Concept | Why it matters |
|---|---|
| WaitForFirstConsumer | Volume created in same AZ as pod |
| ReadWriteOnce | Only 1 node can attach at a time |
| Recreate strategy | EBS can't attach to 2 nodes — old pod must die first |
| gp3 | 3000 IOPS SSD, cheaper than gp2 |
| allowVolumeExpansion | Grow volumes without recreating |

## Persistence Test
Deleted MySQL pod → pod recreated → bankappdb still exists.
EBS volume outlives the pod. Data is safe.

## Resource Budget (3x t3.medium)

| Component | CPU Request | Memory Request |
|---|---|---|
| BankApp (2 pods) | 500m | 512Mi |
| MySQL | 250m | 256Mi |
| Ollama | 900m | 2Gi |
| System | ~500m | ~500Mi |
| Available | 6000m | 12Gi |

Summary Table:

| Concept | What it means |
|---|---|
| Gateway API | Next-gen Kubernetes traffic management — replaces Ingress |
| GatewayClass | Defines which controller (Envoy) handles Gateways |
| Gateway | Creates AWS NLB + defines port/TLS listeners |
| HTTPRoute | Routes URL paths to backend services |
| BackendTrafficPolicy | Envoy-specific: session affinity, load balancing rules |
| Envoy Gateway | The controller that implements Gateway API on EKS |
| BANKAPP_AFFINITY cookie | Pins user sessions to one pod — prevents Spring logout |
| cert-manager | Automates TLS certificate lifecycle with Let's Encrypt |
| HTTP-01 challenge | Let's Encrypt validation method — serve a file at a URL |
| nip.io | Free wildcard DNS — 1.2.3.4.nip.io resolves to 1.2.3.4 |
| WaitForFirstConsumer | EBS volume created in same AZ as pod |
| ReadWriteOnce | EBS attaches to ONE node only |
| Recreate strategy | Kill old pod before new one — required for EBS |
| gp3 | AWS SSD storage — 3000 IOPS baseline, cheaper than gp2 |
| PVC Bound | Volume successfully provisioned and attached |

