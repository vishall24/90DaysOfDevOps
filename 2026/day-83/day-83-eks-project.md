# Day 83 -- EKS Project: Production Deployment of AI-BankApp

## Task
Three days of EKS -- cluster provisioning with Terraform, Gateway API networking, EBS storage, and TLS. Today you put it all together and deploy the AI-BankApp as a production-grade application on EKS. Full stack: Spring Boot app with MySQL and Ollama AI, persistent storage, autoscaling, monitoring, and the complete end-to-end validation.

This is the kind of deployment you would do on the job.

Reference: https://github.com/TrainWithShubham/AI-BankApp-DevOps (branch: `feat/gitops`)

---

What Are We Doing Today?

    Day   | Focus
    81    |  Provisioned EKS cluster with Terraform
    82    |  Gateway API, TLS, EBS storage deep dive
    83    |  Full production deployment + monitoring + validation + teardown

Today = everything from Days 81-82 combined, plus Prometheus/Grafana monitoring, a formal validation checklist, and a clean, 
complete teardown.

full picture:
    
    Terraform (EKS cluster)
    ↓
    MySQL + Ollama (with EBS storage)
    ↓
    BankApp (Spring Boot, HPA-managed)
    ↓
    Gateway API (Envoy → AWS NLB)
    ↓
    Prometheus + Grafana (watching everything)
    ↓
    Validation checklist
    ↓
    Full teardown

Architecture

        Internet
           ↓
        AWS NLB (Envoy Gateway)
           ↓
        EKS Cluster (3x t3.medium nodes)
           ├── bankapp namespace
           │    ├── BankApp (2-4 pods, HPA managed)
           │    ├── MySQL (1 pod, 5Gi EBS)
           │    └── Ollama AI (1 pod, 10Gi EBS)
           └── monitoring namespace
                ├── Prometheus (scrapes all metrics)
                └── Grafana (dashboards + alerts)

What we're validating end-to-end:

| Layer | Check |
|---|---|
| Application | BankApp responds, login works, AI chatbot works |
| Data | MySQL healthy, data persists, Ollama model loaded |
| Infrastructure | Nodes healthy, Gateway routing, HPA scaling |
| Observability | Prometheus scraping, Grafana dashboards live |

---

## Expected Output
- Complete AI-BankApp stack deployed on EKS
- MySQL with persistent EBS storage, Ollama with model loaded
- Gateway API routing traffic, HPA scaling pods
- Monitoring stack (Prometheus + Grafana) observing the cluster
- Full end-to-end validation checklist passed
- Complete teardown of all AWS resources
- A markdown file: `day-83-eks-project.md`

---

## Challenge Tasks

### Task 1: Deploy the Complete AI-BankApp Stack
Make sure your EKS cluster is running:
```bash
kubectl get nodes
```

If you destroyed the cluster, re-provision it:
```bash
cd AI-BankApp-DevOps/terraform
terraform apply
aws eks update-kubeconfig --name bankapp-eks --region us-west-2
```

Deploy the entire application stack in order:
```bash
cd AI-BankApp-DevOps

# 1. Namespace and storage
kubectl apply -f k8s/namespace.yml
kubectl apply -f k8s/pv.yml
kubectl apply -f k8s/pvc.yml

# 2. Configuration
kubectl apply -f k8s/configmap.yml
kubectl apply -f k8s/secrets.yml

# 3. Database and AI service
kubectl apply -f k8s/mysql-deployment.yml
kubectl apply -f k8s/service.yml
kubectl apply -f k8s/ollama-deployment.yml

# 4. Wait for dependencies
echo "Waiting for MySQL..."
kubectl wait --for=condition=ready pod -l app=mysql -n bankapp --timeout=120s

echo "Waiting for Ollama (this takes 2-5 minutes for model pull)..."
kubectl wait --for=condition=ready pod -l app=ollama -n bankapp --timeout=600s

# 5. Application
kubectl apply -f k8s/bankapp-deployment.yml
kubectl apply -f k8s/hpa.yml

# 6. Wait for BankApp
echo "Waiting for BankApp..."
kubectl wait --for=condition=ready pod -l app=bankapp -n bankapp --timeout=300s
```

Verify everything is running:
```bash
kubectl get all -n bankapp
kubectl get pvc -n bankapp
```

You should see:
- MySQL: 1 pod running with 5Gi PVC bound
- Ollama: 1 pod running with 10Gi PVC bound
- BankApp: 2-4 pods running (managed by HPA)
- Services: 3 ClusterIP services

---

Verify: All pods Running, both PVCs Bound.

<img width="987" height="456" alt="image" src="https://github.com/user-attachments/assets/305d241a-e391-4ee8-a2f2-44bea36d8ced" />

---

### Task 2: Set Up Gateway API and Access the App
Install Envoy Gateway (if not done on Day 82):
```bash
helm install envoy-gateway oci://docker.io/envoyproxy/gateway-helm \
  --version v1.4.0 \
  -n envoy-gateway-system --create-namespace \
  --wait 2>/dev/null || echo "Already installed"
```

Apply the Gateway configuration:
```bash
kubectl apply -f k8s/gateway.yml
```

Wait for the NLB:
```bash
kubectl get gateway -n bankapp -w
```

Get the external address:
```bash
export APP_URL=$(kubectl get gateway bankapp-gateway -n bankapp -o jsonpath='{.status.addresses[0].value}')
echo "AI-BankApp URL: http://$APP_URL"
```

Test the application:
```bash
# Health check (Spring Boot Actuator)
curl -s http://$APP_URL/actuator/health | python3 -m json.tool

# Load the home page
curl -s -o /dev/null -w "%{http_code}" http://$APP_URL
```

Open `http://$APP_URL` in your browser:
1. Click "Register" and create an account
2. Log in with your credentials
3. Perform banking operations (deposit, withdraw, transfer)
4. Try the AI chatbot -- ask a financial question
5. Toggle dark/light mode

**The full stack is running on EKS:** Spring Boot serves the UI, MySQL stores accounts and transactions, Ollama's TinyLlama model powers the AI chatbot -- all on managed Kubernetes with persistent storage and autoscaling.

---

<img width="1162" height="245" alt="image" src="https://github.com/user-attachments/assets/58e97f2e-860f-4e63-8aa3-724be804b10b" />

<img width="1016" height="268" alt="image" src="https://github.com/user-attachments/assets/299ff3f7-9ec9-4ba0-8b98-219ba86f5fdf" />

Full application working:

<img width="1438" height="856" alt="image" src="https://github.com/user-attachments/assets/7b6fbbcf-a75b-471c-b3e7-d70e7fb23e2b" />


---

### Task 3: Deploy the Monitoring Stack
Deploy Prometheus and Grafana to monitor the AI-BankApp on EKS.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace \
  --set grafana.adminPassword=admin123 \
  --set prometheus.prometheusSpec.retention=3d \
  --set prometheus.prometheusSpec.resources.requests.memory=256Mi \
  --set prometheus.prometheusSpec.resources.requests.cpu=100m \
  --wait --timeout 600s
```

Verify:
```bash
kubectl get pods -n monitoring
```

**Access Grafana:**
```bash
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```

Open `http://localhost:3000`. Login: `admin` / `admin123`.

**The AI-BankApp exposes Prometheus metrics natively.** The Spring Boot Actuator endpoint at `/actuator/prometheus` provides JVM metrics, HTTP request metrics, and more.

Create a ServiceMonitor to scrape the BankApp:
```yaml
# bankapp-servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: bankapp-monitor
  namespace: monitoring
  labels:
    release: monitoring
spec:
  namespaceSelector:
    matchNames:
      - bankapp
  selector:
    matchLabels:
      app: bankapp
  endpoints:
    - port: "8080"
      path: /actuator/prometheus
      interval: 15s
```

```bash
kubectl apply -f bankapp-servicemonitor.yaml
```

**Access Prometheus:**
```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus -n monitoring 9090:9090
```

Query AI-BankApp metrics:
```promql
# JVM memory usage
jvm_memory_used_bytes{namespace="bankapp"}

# HTTP request rate
rate(http_server_requests_seconds_count{namespace="bankapp"}[5m])

# HTTP request latency (95th percentile)
histogram_quantile(0.95, rate(http_server_requests_seconds_bucket{namespace="bankapp"}[5m]))
```

Explore the pre-built Grafana dashboards:
- **Kubernetes / Compute Resources / Namespace (Pods)** -- select the `bankapp` namespace
- **Kubernetes / Compute Resources / Pod** -- drill into individual pods
- **Node Exporter / Nodes** -- EKS worker node health

---

Concept First:

The AI-BankApp uses Spring Boot Actuator with Micrometer — it natively exposes Prometheus metrics at /actuator/prometheus. This means you can scrape:

JVM memory usage
HTTP request rates and latencies
Database connection pool stats
Custom application metrics


Setup prometheus:

<img width="1405" height="729" alt="image" src="https://github.com/user-attachments/assets/cef76a7b-91e0-4a3d-9b80-61adf7706631" />

This installs:

Prometheus (metrics storage)
Grafana (dashboards)
Alertmanager (alerting)
Node Exporter (node metrics)
kube-state-metrics (Kubernetes object metrics)

<img width="915" height="178" alt="image" src="https://github.com/user-attachments/assets/208ae81c-60f1-4081-aa0e-99c748793436" />

vim bankapp-servicemonitor.yaml:

<img width="678" height="336" alt="image" src="https://github.com/user-attachments/assets/a9b64dbc-ebd9-4a50-921c-ee2677a2f083" />

<img width="1303" height="235" alt="image" src="https://github.com/user-attachments/assets/d5ec245c-b075-42e0-a70d-bf3d70b768b1" />

Access Grafana from browser:

<img width="1440" height="800" alt="image" src="https://github.com/user-attachments/assets/c594ee00-7084-4f59-926c-b1d2dddf0bb7" />

Explore more:

<img width="1439" height="861" alt="image" src="https://github.com/user-attachments/assets/f5150dd6-0ead-4e72-a7a2-42aabed45603" />

<img width="1440" height="859" alt="image" src="https://github.com/user-attachments/assets/068c34d1-6d64-4102-b7f6-5d40851c83a8" />

<img width="1440" height="861" alt="image" src="https://github.com/user-attachments/assets/fedf8c6a-5682-4555-abe9-a768101c0138" />

<img width="1439" height="855" alt="image" src="https://github.com/user-attachments/assets/eef51045-f0d8-4095-b30c-a3ff31921e47" />

Access prometheus :

<img width="953" height="147" alt="image" src="https://github.com/user-attachments/assets/4ee4cdb0-674a-4718-b7d0-1c180d2fb2b8" />

Browser:

<img width="1440" height="805" alt="image" src="https://github.com/user-attachments/assets/2e39028e-ecff-4c2c-87d2-8f502a7cdd3d" />

Promql queries:

<img width="1427" height="736" alt="image" src="https://github.com/user-attachments/assets/de89fe7d-c264-4520-a43b-b4dfd02600c8" />

<img width="1440" height="815" alt="image" src="https://github.com/user-attachments/assets/ca44ccff-899e-46e9-925f-b16d148b9dd1" />

<img width="1440" height="655" alt="image" src="https://github.com/user-attachments/assets/c487cead-3346-4b73-a811-90aa05aa7923" />

<img width="1440" height="814" alt="image" src="https://github.com/user-attachments/assets/6630dc50-0cca-41a6-ae8f-792fe6b12b3e" />

---

### Task 4: End-to-End Validation Checklist
Run through the complete validation:

**Application layer:**
```bash
# All pods running and ready
kubectl get pods -n bankapp
echo "---"

# App responds on health endpoint
curl -s http://$APP_URL/actuator/health
echo "---"

# HPA is active and monitoring CPU
kubectl get hpa -n bankapp
echo "---"

# Prometheus metrics endpoint works
curl -s http://$APP_URL/actuator/prometheus | head -10
```

**Data layer:**
```bash
# MySQL is healthy with persistent storage
kubectl exec -n bankapp deploy/mysql -- mysqladmin ping -h localhost -uroot -pTest@123
echo "---"

# PVCs are bound to EBS volumes
kubectl get pvc -n bankapp
echo "---"

# Ollama has the model loaded
kubectl exec -n bankapp deploy/ollama -- ollama list
```

**Infrastructure layer:**
```bash
# Nodes are healthy
kubectl get nodes
kubectl top nodes
echo "---"

# Gateway is serving traffic
kubectl get gateway -n bankapp
echo "---"

# Monitoring is running
kubectl get pods -n monitoring | head -5
```

**Security layer:**
```bash
# BankApp runs as non-root (devsecops user)
kubectl exec -n bankapp deploy/bankapp -- whoami

# Secrets are not exposed in environment
kubectl get secret bankapp-secret -n bankapp -o yaml | grep -c "MYSQL_ROOT_PASSWORD"
```

---

Application Layer:

<img width="867" height="567" alt="image" src="https://github.com/user-attachments/assets/b2d54520-4e2f-4a8e-8cfb-d088702df18a" />

Data Layer:

<img width="1127" height="301" alt="image" src="https://github.com/user-attachments/assets/f76e5335-e1aa-464b-b2b1-2db3e0371e05" />

Infrastructure Layer:

<img width="1044" height="486" alt="image" src="https://github.com/user-attachments/assets/c94fd470-5ac5-45db-9fe1-42ed84994d44" />

Security Layer:

<img width="873" height="204" alt="image" src="https://github.com/user-attachments/assets/418b8d2e-ed62-430c-9dc0-852d483a1457" />

---

### Task 5: Reflect on the Full EKS Journey
Map each concept to the day you learned it:

| Day | What You Built | AI-BankApp Connection |
|-----|---------------|----------------------|
| 81 | EKS cluster via Terraform, kubectl connection, manual deploy | Used the project's `terraform/` configs to provision infra |
| 82 | Gateway API, Envoy, TLS, EBS storage, session persistence | Used `k8s/gateway.yml`, `k8s/cert-manager.yml`, `k8s/pv.yml` |
| 83 | Full production deployment, monitoring, validation | Complete stack: app + DB + AI + networking + observability |

**What the AI-BankApp's EKS setup includes that you have now seen:**
- Terraform-provisioned VPC with 3-AZ networking
- Managed node group with auto-scaling
- 6 EKS add-ons (CoreDNS, VPC CNI, kube-proxy, Pod Identity, EBS CSI, Metrics Server)
- ArgoCD pre-installed (used on Days 84-86)
- Gateway API with Envoy for traffic management
- cert-manager for automated HTTPS
- Cookie-based session persistence for stateful app
- EBS persistent storage for MySQL and Ollama
- HPA with scale-up/down policies
- Spring Boot Actuator metrics for Prometheus
- Init containers for dependency ordering
- PostStart lifecycle hooks for Ollama model pull

**What you would add for a real production deployment:**
- DNS with Route 53 and ExternalDNS
- Network Policies for pod-to-pod isolation
- Pod Disruption Budgets for safe node draining
- External Secrets Operator for AWS Secrets Manager integration
- Database backups (automated MySQL dumps to S3)
- Log aggregation with Loki (you built this on Day 75)
- Multi-environment clusters (dev + prod)

---

## 3-Day EKS Block Summary

| Day | What I Built | Key Learning |
|---|---|---|
| Day 81 | EKS cluster via Terraform, kubectl connect, manual deploy | Terraform provisions 65+ AWS resources for EKS |
| Day 82 | Gateway API, Envoy, TLS with cert-manager, EBS deep dive | Gateway API replaces Ingress, EBS is AZ-locked |
| Day 83 | Full production stack, monitoring, validation, teardown | Complete prod deployment with observability |

What the AI-BankApp includes that we've now seen:

| Component | What it does |
|---|---|
| Terraform VPC | 9 subnets across 3 AZs |
| Managed node group | Auto-scaling EC2 workers |
| 6 EKS add-ons | CoreDNS, VPC CNI, EBS CSI, metrics-server etc. |
| ArgoCD | Pre-installed, used on Day 84 |
| Gateway API + Envoy | Production traffic management |
| cert-manager | Automated HTTPS |
| Cookie session affinity | Spring Security persistence |
| EBS storage | MySQL + Ollama persistent volumes |
| HPA | Auto-scaling 2-4 BankApp pods |
| Spring Boot Actuator | Native Prometheus metrics |
| Init containers | MySQL + Ollama dependency ordering |
| PostStart hooks | Ollama model pull on startup |

---

### Task 6: Complete Teardown
**This is critical -- do not leave resources running.**

Delete workloads first:
```bash
# Delete monitoring
helm uninstall monitoring -n monitoring

# Delete Gateway resources (releases the NLB)
kubectl delete -f k8s/gateway.yml 2>/dev/null

# Delete the BankApp stack
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

# Delete Envoy Gateway
helm uninstall envoy-gateway -n envoy-gateway-system 2>/dev/null

# Delete cert-manager
helm uninstall cert-manager -n cert-manager 2>/dev/null

# Delete namespaces
kubectl delete namespace monitoring envoy-gateway-system cert-manager 2>/dev/null
```

Wait for all LoadBalancers and EBS volumes to be released:
```bash
# Check for lingering load balancers
kubectl get svc -A | grep LoadBalancer

# Check for lingering PVCs
kubectl get pvc -A
```

**Destroy the infrastructure with Terraform:**
```bash
cd terraform
terraform destroy
```

This takes 10-15 minutes. It deletes:
- EKS cluster and control plane
- All node groups and EC2 instances
- ArgoCD Helm release
- VPC, subnets, NAT gateway, internet gateway
- IAM roles and policies

**Verify in the AWS Console:**
- EKS: no clusters
- EC2: no instances, no load balancers, no EBS volumes
- VPC: the `bankapp-eks` VPC is gone
- CloudFormation: no lingering stacks

**Check your AWS bill** in the Billing Dashboard. All charges should stop within the hour.

**Cost for this 3-day lab (approximate):** $15-25 depending on how long you kept the cluster running.

---

Critical — Do This In Order or Resources Will Block
Wrong order causes terraform destroy to fail because:

Kubernetes LoadBalancer Services create AWS NLBs that live INSIDE the VPC
Kubernetes PVCs create EBS volumes INSIDE the cluster
If Terraform tries to delete the VPC while these exist → it fails

<img width="1432" height="749" alt="image" src="https://github.com/user-attachments/assets/f0164b49-e1a7-4248-b5f1-d53501cec3a7" />

then terraform destroy:

<img width="1264" height="782" alt="image" src="https://github.com/user-attachments/assets/f042d42f-48ca-4677-99f6-1d7abcffe754" />

---

## Full Architecture

        Internet
        ↓
        AWS NLB (Envoy Gateway auto-provisioned)
        ↓
        Gateway: bankapp-gateway (HTTP:80, HTTPS:443)
        ↓
        HTTPRoute → bankapp-service:8080
        ↓
        EKS Cluster (3x t3.medium, 3 AZs)
        ├── bankapp namespace
        │    ├── BankApp pods (2-4, HPA managed)
        │    │    ├── Init: wait-for-mysql
        │    │    └── Init: wait-for-ollama
        │    ├── MySQL pod → 5Gi EBS (gp3)
        │    └── Ollama pod → 10Gi EBS (gp3)
        └── monitoring namespace
        ├── Prometheus → scrapes all pods + BankApp actuator
        └── Grafana → dashboards

## Deployment Order (Critical!)
1. Namespace + Storage (PVCs)
2. ConfigMap + Secrets
3. MySQL + Services + Ollama
4. Wait for MySQL ready (--timeout=120s)
5. Wait for Ollama ready (--timeout=600s, model pull)
6. BankApp + HPA
7. Wait for BankApp ready (--timeout=300s)
8. Gateway API
9. Monitoring stack

## Validation Results

| Check | Result |
|---|---|
| All pods Running | ✅ |
| PVCs Bound to EBS | ✅ |
| Health endpoint: db UP | ✅ |
| Ollama model loaded | ✅ |
| HPA active | ✅ |
| Gateway NLB provisioned | ✅ |
| App accessible in browser | ✅ |
| Banking operations work | ✅ |
| AI chatbot responds | ✅ |
| Grafana dashboards live | ✅ |
| Prometheus scraping BankApp | ✅ |

## Key PromQL Queries for AI-BankApp

| What | Query |
|---|---|
| JVM memory | `jvm_memory_used_bytes{namespace="bankapp"}` |
| HTTP rate | `rate(http_server_requests_seconds_count{namespace="bankapp"}[5m])` |

## Teardown Order (Critical!)
1. helm uninstall monitoring
2. kubectl delete gateway (removes NLB)
3. Wait for LB deletion
4. kubectl delete all k8s resources
5. kubectl delete PVCs (removes EBS)
6. helm uninstall envoy-gateway
7. terraform destroy

Wrong order = terraform destroy fails on VPC deletion.

## 3-Day EKS Block Cost
~$15-25 total depending on uptime.
EKS: $0.10/hr + 3x t3.medium: $0.126/hr + NAT: $0.045/hr = ~$0.30/hr

## Key Takeaways
- kubectl wait is better than kubectl get pods -w in scripts
- Delete K8s LoadBalancer services BEFORE terraform destroy
- Ollama needs bigger instance than t3.medium for production
- Spring Boot Actuator + Prometheus = zero-config app monitoring
- ServiceMonitor CRD bridges Kubernetes services and Prometheus scraping

###  Summary Table

| Concept | What it means |
|---|---|
| `kubectl wait --for=condition=ready` | Scripted waiting — no manual polling |
| `--timeout=600s` | Ollama needs up to 10 mins for model pull |
| kube-prometheus-stack | All-in-one monitoring: Prometheus + Grafana + Alertmanager |
| ServiceMonitor | Tells Prometheus which services to scrape and how |
| `release: monitoring` label | Must match Helm release name for ServiceMonitor to work |
| `/actuator/prometheus` | Spring Boot Actuator's Prometheus metrics endpoint |
| JVM metrics | Memory, GC, thread counts from the Spring Boot app |
| HTTP metrics | Request rate, latency, error rate per endpoint |
| Teardown order | Monitoring → Gateway → App → PVCs → Terraform |
| Delete PVCs before destroy | Releases EBS volumes — otherwise orphaned |
| Delete Gateway before destroy | Releases NLB — otherwise VPC deletion fails |
| `terraform destroy` | Removes all 65+ AWS resources in 10-15 minutes |
| Cost: ~$0.30/hr | Always destroy when not actively working |

