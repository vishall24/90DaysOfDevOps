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

