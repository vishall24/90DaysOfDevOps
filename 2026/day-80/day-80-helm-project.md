# Day 80 -- Helm Project: Multi-Environment Deployment and CI/CD

## Task
Two days of Helm -- chart basics and a custom chart for the AI-BankApp. Today you bring it all together. You will create environment-specific values for dev, staging, and production, add Helm hooks, package the chart, and integrate Helm into the AI-BankApp's CI/CD pipeline.

Reference: https://github.com/TrainWithShubham/AI-BankApp-DevOps (branch: `feat/gitops`)

---

What Are We Doing Today?

Day 78: Learned Helm basics, deployed MySQL from community chart
Day 79: Built custom Helm chart for AI-BankApp (12 YAML → 1 chart)
Day 80: Make it production-ready
         ├── 3 environment configs (dev/staging/prod)
         ├── Helm hooks (pre-install jobs)
         ├── Package into .tgz for distribution
         └── Understand CI/CD integration
         
The core idea today:
Same chart  →  dev:     1 replica, 2Gi storage, tinyllama, HPA off
            →  staging: 2-3 replicas, 5Gi storage, HPA on
            →  prod:    2-4 replicas, 20Gi storage, gateway on

Zero code duplication. Just different values files.

---

## Challenge Tasks

### Task 1: Create Environment-Specific Values
One chart, three environments. The AI-BankApp runs differently in dev vs production.

Create `bankapp/values-dev.yaml`:
```yaml
bankapp:
  replicaCount: 1
  image:
    repository: trainwithshubham/ai-bankapp-eks
    tag: "latest"
    pullPolicy: Always
  resources:
    requests:
      memory: "256Mi"
      cpu: "100m"
    limits:
      memory: "512Mi"
      cpu: "250m"
  autoscaling:
    enabled: false

mysql:
  enabled: true
  resources:
    requests:
      memory: "128Mi"
      cpu: "100m"
    limits:
      memory: "256Mi"
      cpu: "250m"
  persistence:
    size: 2Gi
    storageClass: standard

ollama:
  enabled: true
  model: tinyllama
  resources:
    requests:
      memory: "1Gi"
      cpu: "500m"
    limits:
      memory: "1.5Gi"
      cpu: "1000m"
  persistence:
    size: 5Gi
    storageClass: standard

storageClass:
  create: false
```

Create `bankapp/values-staging.yaml`:
```yaml
bankapp:
  replicaCount: 2
  image:
    repository: trainwithshubham/ai-bankapp-eks
    tag: "v1.2.0"
    pullPolicy: IfNotPresent
  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 3
    targetCPUUtilization: 75

mysql:
  enabled: true
  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  persistence:
    size: 5Gi
    storageClass: gp3

ollama:
  enabled: true
  model: tinyllama
  persistence:
    size: 10Gi
    storageClass: gp3

secrets:
  mysqlRootPassword: StagingPass@456
  mysqlUser: root
  mysqlPassword: StagingPass@456

storageClass:
  create: true
```

Create `bankapp/values-prod.yaml`:
```yaml
bankapp:
  replicaCount: 4
  image:
    repository: trainwithshubham/ai-bankapp-eks
    tag: "v1.2.0"
    pullPolicy: IfNotPresent
  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
    targetCPUUtilization: 70

mysql:
  enabled: true
  resources:
    requests:
      memory: "512Mi"
      cpu: "500m"
    limits:
      memory: "1Gi"
      cpu: "1000m"
  persistence:
    size: 20Gi
    storageClass: gp3

ollama:
  enabled: true
  model: tinyllama
  resources:
    requests:
      memory: "2Gi"
      cpu: "900m"
    limits:
      memory: "2.5Gi"
      cpu: "1500m"
  persistence:
    size: 10Gi
    storageClass: gp3

secrets:
  mysqlRootPassword: ProdSecure@789
  mysqlUser: root
  mysqlPassword: ProdSecure@789

storageClass:
  create: true

gateway:
  enabled: true
```

**Compare the environments:**

| Setting | Dev | Staging | Prod |
|---------|-----|---------|------|
| BankApp replicas | 1 (fixed) | 2-3 (HPA) | 2-4 (HPA) |
| Image tag | latest | v1.2.0 | v1.2.0 |
| MySQL storage | 2Gi | 5Gi | 20Gi |
| MySQL resources | 128Mi/100m | 256Mi/250m | 512Mi/500m |
| Ollama memory | 1Gi | 2Gi | 2.5Gi |
| Gateway | disabled | disabled | enabled |

**Deploy to different environments:**
```bash
# Dev (on Kind)
helm install bankapp-dev bankapp/ -f bankapp/values-dev.yaml -n dev --create-namespace

# Staging (render to check)
helm template bankapp-staging bankapp/ -f bankapp/values-staging.yaml | grep "replicas:"

# Prod (render to check)
helm template bankapp-prod bankapp/ -f bankapp/values-prod.yaml | grep "replicas:"
```

Same chart, wildly different deployments.

---

Concept First:

| | Dev | Staging | Prod |
|---|---|---|---|
| Replicas | 1 fixed | 2-3 HPA | 2-4 HPA |
| Image tag | latest | v1.2.0 | v1.2.0 |
| MySQL storage | 2Gi | 5Gi | 20Gi |
| StorageClass | standard (Kind!) | gp3 (AWS) | gp3 (AWS) |
| Gateway | off | off | on |
| HPA | disabled | enabled | enabled |


vim values-dev.yaml:

<img width="660" height="818" alt="image" src="https://github.com/user-attachments/assets/1474f4f3-3c32-4aaf-af5a-406b8fcc1ce1" />

vim values-staging.yaml:

<img width="603" height="845" alt="image" src="https://github.com/user-attachments/assets/40ae80c2-15ca-46fb-a51e-9e23163b9d16" />

vim values-prod.yaml:

<img width="469" height="881" alt="image" src="https://github.com/user-attachments/assets/c49c609d-3bd4-41d3-83bd-82f856e8c1ed" />

Deployed using helm only for dev:

<img width="1124" height="456" alt="image" src="https://github.com/user-attachments/assets/ec24498b-1d3c-4ee7-b95e-eafc27764a65" />

checking resources:

<img width="1341" height="470" alt="image" src="https://github.com/user-attachments/assets/94bc8b90-d37f-40c0-8a85-0d92b5135633" />

rendering template:

<img width="656" height="134" alt="image" src="https://github.com/user-attachments/assets/8f60646a-7056-4727-a917-e7ea64521827" />

<img width="692" height="109" alt="image" src="https://github.com/user-attachments/assets/b61f76f2-30cd-420b-9990-018f72a1ba75" />

<img width="859" height="191" alt="image" src="https://github.com/user-attachments/assets/d682b4bf-c4f0-4461-b5ae-9898223ef634" />

Dev shows replicas: 1 in the Deployment. Staging shows NO replicas in Deployment (HPA manages it) but shows minReplicas: 2 in HPA. ✅

---

### Task 2: Add Helm Hooks
The AI-BankApp uses init containers to wait for MySQL. Helm hooks offer another approach -- running pre-install jobs.

Create `bankapp/templates/pre-install-job.yaml`:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "bankapp.fullname" . }}-db-ready
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "bankapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      containers:
        - name: db-check
          image: busybox:1.36
          command:
            - /bin/sh
            - -c
            - |
              echo "Waiting for MySQL to be ready..."
              until nc -z {{ include "bankapp.fullname" . }}-mysql 3306; do
                echo "MySQL not ready, retrying in 3s..."
                sleep 3
              done
              echo "MySQL is ready!"
          resources:
            requests: { memory: "32Mi", cpu: "50m" }
            limits: { memory: "64Mi", cpu: "100m" }
      restartPolicy: Never
  backoffLimit: 10
```

**How hooks work in the AI-BankApp context:**
- `helm.sh/hook: pre-install,pre-upgrade` -- runs before install and before upgrade
- This ensures MySQL is up before the BankApp Deployment is created
- `before-hook-creation` -- deletes the old job before creating a new one on re-runs
- Combined with init containers in the Deployment, this provides defense-in-depth

**Other useful hook types:**
- `post-install` -- run database migrations after deploy
- `pre-delete` -- backup database before teardown
- `test` -- runs when you execute `helm test`

**Add a Helm test:**

Create `bankapp/templates/tests/test-connection.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: {{ include "bankapp.fullname" . }}-test
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "bankapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: test
      image: busybox:1.36
      command: ['sh', '-c', 'wget -qO- http://{{ include "bankapp.fullname" . }}-service:8080/actuator/health']
  restartPolicy: Never
```

After deploying, run:
```bash
helm test bankapp-dev -n dev
```

This hits the Spring Boot health endpoint and confirms the app is running.

---

Concept First:

What are Helm hooks?
Regular Kubernetes resources in your chart deploy at install time. Hooks run at specific points in the lifecycle:

| Hook | When it runs | Use case |
|---|---|---|
| pre-install | Before any resources created | Check dependencies ready |
| post-install | After all resources created | Run DB migrations |
| pre-upgrade | Before upgrade | Backup database |
| post-upgrade | After upgrade | Notify team |
| pre-delete | Before uninstall | Cleanup jobs |
| test | When helm test runs | Health checks |

How it's different from init containers:

Init containers run inside pods at start time
Hooks run as separate Kubernetes Jobs before/after the whole release

Defense in depth for AI-BankApp:

pre-install hook (Job) → checks MySQL is ready
   ↓
BankApp Deployment created with init containers
   ↓
Init containers also wait for MySQL
Two layers of MySQL readiness checks = very reliable startup.

vim templates/pre-install-job.yaml:

<img width="693" height="681" alt="image" src="https://github.com/user-attachments/assets/23b9fdf5-ce3f-45a3-ab77-be82787015d1" />

vim bankapp/templates/tests/test-connection.yaml:

<img width="1059" height="479" alt="image" src="https://github.com/user-attachments/assets/42f8f940-41a8-46d2-96a5-550d8278cb07" />

<img width="970" height="531" alt="image" src="https://github.com/user-attachments/assets/1ca7a521-b5a9-4e39-a1d1-75a3fdb1ca7a" />

<img width="1200" height="489" alt="image" src="https://github.com/user-attachments/assets/7903a792-a1ab-45ee-a2e6-38d551850e7e" />

<img width="756" height="580" alt="image" src="https://github.com/user-attachments/assets/51294bab-93fe-482a-aa61-f7472a2681b9" />

---

### Task 3: Package and Version the Chart
Package the chart into a distributable `.tgz` file:

```bash
# Lint first
helm lint bankapp/

# Package
helm package bankapp/
```

This creates `bankapp-0.1.0.tgz`.

**Bump the version after changes:**
Edit `bankapp/Chart.yaml`:
```yaml
version: 0.2.0        # Chart structure changed (added hooks)
appVersion: "1.1.0"    # App version updated
```

Re-package:
```bash
helm package bankapp/
```

Now you have `bankapp-0.1.0.tgz` and `bankapp-0.2.0.tgz`.

**Install from a package:**
```bash
helm install my-bankapp bankapp-0.2.0.tgz -f bankapp/values-dev.yaml -n bankapp --create-namespace
```

**Create a chart repository index** (for sharing via GitHub Pages):
```bash
mkdir chart-repo
cp bankapp-*.tgz chart-repo/
helm repo index chart-repo/ --url https://your-username.github.io/helm-charts
cat chart-repo/index.yaml
```

---

Concept first:

| Version | Changes | Use case |
|---|---|---|
| 0.1.0 | Initial chart | First release |
| 0.2.0 | Added hooks, test pod | Chart structure changed |
| 0.2.1 | Bug fixes | Patch release |

Packaging creates a .tgz file you can share with teams, push to GitHub Pages, or store in a chart museum.

<img width="967" height="162" alt="image" src="https://github.com/user-attachments/assets/54ef8241-32a5-49f9-9dce-5c29750ca3fb" />

<img width="906" height="566" alt="image" src="https://github.com/user-attachments/assets/933745a5-19cc-408c-8dc9-65cee1011e73" />

vim bankapp/Chart.yaml:

<img width="765" height="279" alt="image" src="https://github.com/user-attachments/assets/4c7f0d8c-6bb9-44e7-bb52-7787067b032b" />

helm package bankapp/
ls *.tgz

Output:

         bankapp-0.1.0.tgz
         bankapp-0.2.0.tgz

