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

a<img width="777" height="297" alt="image" src="https://github.com/user-attachments/assets/9ab78bf3-eb98-4c64-96db-f79a68b2386b" />

helm package bankapp/
ls *.tgz

Output:

         bankapp-0.1.0.tgz
         bankapp-0.2.0.tgz

---

         mkdir chart-repo
         cp bankapp-*.tgz chart-repo/
         
         helm repo index chart-repo/ \
                    --url https://vishall24.github.io/helm-charts
                    
cat chart-repo/index.yaml:

<img width="610" height="677" alt="image" src="https://github.com/user-attachments/assets/2475174b-8fa8-4eeb-9194-477f530b0a04" />

Output shows both chart versions listed. This index.yaml + .tgz files is all you need for a Helm repository hosted on GitHub Pages.

---

### Task 4: Understand Helm in the AI-BankApp GitOps Pipeline
The AI-BankApp uses a GitOps pipeline. Study how Helm could integrate:

**Current pipeline (from `.github/workflows/gitops-ci.yml`):**
```
Developer pushes code
  -> GitHub Actions builds Docker image
  -> Tags with git commit SHA
  -> Updates image tag in k8s/bankapp-deployment.yml via sed
  -> Commits the change back to the repo
  -> ArgoCD detects the change and syncs to EKS
```

**With Helm, the pipeline becomes:**
```
Developer pushes code
  -> GitHub Actions builds Docker image
  -> Tags with git commit SHA
  -> Updates image.tag in helm-chart/values.yaml (or values-prod.yaml)
  -> Commits the change back to the repo
  -> ArgoCD detects the change and runs helm upgrade on EKS
```

Here is how the CI step would look with Helm (reference pattern):
```yaml
# In the GitHub Actions workflow
- name: Update Helm values with new image tag
  run: |
    TAG=${{ steps.tag.outputs.sha_short }}
    yq -i '.bankapp.image.tag = "'$TAG'"' helm-chart/bankapp/values-prod.yaml

- name: Commit updated Helm values
  run: |
    git config user.name "github-actions[bot]"
    git config user.email "github-actions[bot]@users.noreply.github.com"
    git add helm-chart/bankapp/values-prod.yaml
    git diff --staged --quiet || git commit -m "ci: update bankapp image to $TAG [skip ci]"
    git push
```

**ArgoCD with Helm** (the ArgoCD Application would change from):
```yaml
# Current: raw manifests
source:
  path: k8s
```

To:
```yaml
# With Helm
source:
  path: helm-chart/bankapp
  helm:
    valueFiles:
      - values-prod.yaml
```

ArgoCD natively supports Helm charts -- it renders templates and applies the result, tracking drift against the rendered output.

**Document:** What are the advantages of ArgoCD syncing a Helm chart vs raw manifests?

---

Concept First:

Current AI-BankApp pipeline:

         Code push
           → Build Docker image (tag with git SHA)
           → Update k8s/bankapp-deployment.yml with sed
           → Commit back to repo
           → ArgoCD sees changed manifest → applies to EKS

With Helm:

         Code push
           → Build Docker image (tag with git SHA)
           → Update helm-chart/bankapp/values-prod.yaml (bankapp.image.tag)
           → Commit back to repo
           → ArgoCD sees changed values → runs helm upgrade on EKS

With diagram:

         Developer
              │
              │ git push
              ▼
         GitHub Actions
              │
              ├── Login to DockerHub
              │
              ├── Build Docker Image
              │
              ├── Push Image
              │
              ├── Update deployment.yaml
              │        latest
              │          │
              │          ▼
              │      1a2b3c4
              │
              ├── Commit
              │
              └── Push to GitHub
                        │
                        ▼
                   Git Repository
                        │
                        ▼
                    Argo CD
                        │
                        ▼
                 Kubernetes Cluster
                        │
                        ▼
                  New Pods Created
         

### Helm vs raw manifests with ArgoCD:

| | Raw manifests | Helm with ArgoCD |
|---|---|---|
| Image tag update | sed string replace | yq YAML-aware update |
| Environment differences | Separate file copies | One chart, different values |
| Rollback | Manual git revert | helm rollback or ArgoCD sync |
| Drift detection | Exact manifest comparison | Rendered template comparison |
| Dependencies | Manual | helm dependencies |

---

### Task 5: Helm Best Practices for Production
Review these patterns used in production AI-BankApp deployments:

**1. Always use `helm upgrade --install`:**
```bash
helm upgrade --install bankapp bankapp/ \
  -f bankapp/values-prod.yaml \
  --set bankapp.image.tag=$GIT_SHA \
  -n bankapp --create-namespace \
  --wait --timeout 300s \
  --atomic
```

- `--install` -- creates if missing, upgrades if exists
- `--set bankapp.image.tag=$GIT_SHA` -- pins to exact git commit
- `--wait` -- waits for all pods to be ready
- `--atomic` -- rolls back automatically if the upgrade fails

**2. Use `helm diff` before upgrading:**
```bash
helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade bankapp bankapp/ -f bankapp/values-prod.yaml
```

Shows exactly what would change before you commit to the upgrade.

**3. Resource quotas per namespace:**
```yaml
# Add to templates/resourcequota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: {{ include "bankapp.fullname" . }}-quota
  namespace: {{ .Release.Namespace }}
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

**4. Never store real secrets in values.yaml.** In production, use:
- External Secrets Operator with AWS Secrets Manager
- Sealed Secrets
- Vault by HashiCorp

The `values.yaml` defaults are fine for local dev but should be overridden in CI/CD via `--set` with pipeline secrets.

---

what production team does:

| Method | How it works | Complexity |
|---|---|---|
| --set in CI/CD | `--set secrets.mysqlPassword=$SECRET` | Simple |
| External Secrets Operator | Reads from AWS Secrets Manager | Medium |
| Sealed Secrets | Encrypted secrets committed to Git | Medium |
| HashiCorp Vault | Full secrets management platform | Complex |

For now, --set in CI/CD is fine. The key rule: never commit real production passwords to Git.

---

### Task 6: Clean Up and Review
Check what you have deployed:
```bash
helm list -A
```

**Reflect and document the 3-day Helm journey:**

| Day | Concept | AI-BankApp Connection |
|-----|---------|----------------------|
| 78 | Helm install, repos, values, upgrade, rollback | Deployed MySQL for the BankApp via Bitnami chart |
| 79 | Custom chart from scratch, Go templates | Converted 12 raw `k8s/` manifests into a Helm chart |
| 80 | Multi-env values, hooks, packaging, CI/CD | Production-ready chart with dev/staging/prod configs |

**When would you use Helm vs raw manifests vs Kustomize?**

| Approach | Best For | AI-BankApp Example |
|----------|---------|-------------------|
| Raw manifests | Simple, single-env deployments | The current `k8s/` directory |
| Helm | Multi-env, complex apps with dependencies | The chart you built (3 services, HPA, hooks) |
| Kustomize | Overlays on existing manifests, no templating | Good if you want to patch `k8s/` without rewriting |

**Clean up:**
```bash
helm uninstall bankapp-dev -n dev
kubectl delete namespace dev
kind delete cluster --name tws-cluster
```

---

<img width="1110" height="64" alt="image" src="https://github.com/user-attachments/assets/3da0aab6-1b12-4590-adfc-9c208f2ec426" />

<img width="757" height="132" alt="image" src="https://github.com/user-attachments/assets/9d6d5ebc-779d-440d-a183-db2be9541f29" />

3-day Helm journey:

| Day | What you built |
|---|---|
| 78 | Helm basics, deployed MySQL from Bitnami community chart, upgrade/rollback/history |
| 79 | Built custom chart from scratch, 12 YAML → 8 templates, conditional components |
| 80 | Multi-env values, hooks, packaging, CI/CD integration |

---

## Environment Comparison

| Setting | Dev | Staging | Prod |
|---|---|---|---|
| BankApp replicas | 1 (fixed) | 2-3 (HPA) | 2-4 (HPA) |
| Image tag | latest | v1.2.0 | v1.2.0 |
| MySQL storage | 2Gi | 5Gi | 20Gi |
| StorageClass | standard (Kind) | gp3 (AWS) | gp3 (AWS) |
| Ollama memory limit | 1.5Gi | 2Gi | 2.5Gi |
| HPA | disabled | enabled (75%) | enabled (70%) |
| Gateway | off | off | on |

## Helm Hooks Explained

| Annotation | What it does |
|---|---|
| `helm.sh/hook: pre-install` | Runs before any resources created |
| `helm.sh/hook: pre-upgrade` | Runs before every upgrade |
| `helm.sh/hook-weight: "0"` | Order when multiple hooks exist |
| `helm.sh/hook-delete-policy: before-hook-creation` | Clean up old job before new run |
| `helm.sh/hook: test` | Only runs when helm test is called |

## Chart Versions

| Version | Changes |
|---|---|
| 0.1.0 | Initial chart — 8 templates, 3 deployments |
| 0.2.0 | Added pre-install hook and test pod |

## CI/CD Integration

GitOps flow with Helm:
1. Code pushed → GitHub Actions builds image with git SHA tag
2. yq updates bankapp.image.tag in values-prod.yaml
3. Commit pushed back to repo
4. ArgoCD detects change → runs helm upgrade on EKS

ArgoCD Application change:
- Before: source.path = k8s (raw manifests)
- After: source.path = helm-chart/bankapp + helm.valueFiles = [values-prod.yaml]

## Production Helm Command

```bash
helm upgrade --install bankapp bankapp/ \
  -f bankapp/values-prod.yaml \
  --set bankapp.image.tag=$GIT_SHA \
  -n bankapp --create-namespace \
  --wait --timeout 300s \
  --atomic
```

--atomic is critical: auto-rollbacks if upgrade fails.

## Helm vs Raw Manifests vs Kustomize

| Approach | Best for | AI-BankApp example |
|---|---|---|
| Raw manifests | Simple single-env | Current k8s/ directory |
| Helm | Multi-env, complex apps | Chart with dev/staging/prod |
| Kustomize | Overlays on existing YAML | Patching k8s/ without rewriting |

## Production Secrets — Never in values.yaml
- --set secrets.password=$CI_SECRET (simplest)
- External Secrets Operator with AWS Secrets Manager
- Sealed Secrets (encrypted, safe to commit)
- HashiCorp Vault (full secrets platform)

---

## Summary Table:

| Concept | What it means |
|---|---|
| values-dev.yaml | Dev config: minimal resources, standard StorageClass for Kind |
| values-staging.yaml | Staging config: HPA on, gp3 storage, staging passwords |
| values-prod.yaml | Prod config: max resources, gateway on, prod passwords |
| `storageClass: standard` | Kind's default StorageClass — use instead of gp3 on Kind |
| `storageClass.create: false` | Don't create gp3 on Kind — it doesn't have AWS EBS |
| Helm hook | Job that runs at specific lifecycle points (pre-install, post-install) |
| `helm.sh/hook` annotation | What makes a template a hook instead of a regular resource |
| `--atomic` | Auto-rollback if upgrade fails — critical for CI/CD |
| `--wait` | Wait for all pods Ready before returning |
| `helm diff` plugin | Preview exactly what will change before upgrading |
| `helm package` | Creates distributable .tgz file |
| `helm repo index` | Creates index.yaml for hosting as a chart repository |
| `yq` | YAML-aware tool for updating values in CI (better than sed) |
| Multiple `-f` flags | Later files override earlier ones — base + env pattern |

