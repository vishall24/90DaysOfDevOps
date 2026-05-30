# Day 78 -- Introduction to Helm and Chart Basics

## Task
You have deployed applications with raw Kubernetes manifests -- writing Deployments, Services, ConfigMaps, and Secrets by hand. The AI-BankApp project (https://github.com/TrainWithShubham/AI-BankApp-DevOps, branch `feat/gitops`) has 12 YAML files in its `k8s/` directory. Managing those across dev, staging, and production with slightly different configurations is painful.

Helm is the package manager for Kubernetes. It lets you template, package, version, and deploy Kubernetes applications as reusable units called charts. Today you install Helm, understand chart structure, and deploy your first applications using community charts -- including MySQL, which the AI-BankApp depends on.

---

| Day 59 Helm | Day 78 Helm |
|---|---|
| General Helm intro | Connected to a REAL project (AI-BankApp) |
| Bitnami nginx | Bitnami MySQL (what AI-BankApp actually needs) |
| General charts | Charts with specific configs matching an app |a
| No upgrade/rollback deep dive | Full revision history with rollback |
| No chart structure exploration | Pull and examine chart internals |

Today is about understanding Helm deeply enough to convert the AI-BankApp's 12 raw YAML files into a Helm chart tomorrow (Day 79).

---

## Challenge Tasks

### Task 1: Understand Helm Concepts
Research and write notes on:

1. **What is Helm?**
   - A package manager for Kubernetes (like apt for Ubuntu or yum for RHEL)
   - Packages Kubernetes manifests into reusable, versioned units called **charts**
   - Supports templating -- one chart, many environments

2. **Core concepts:**
   - **Chart** -- a collection of files that describe a set of Kubernetes resources (Deployment + Service + ConfigMap + Secret = one chart)
   - **Release** -- a running instance of a chart in a cluster. You can install the same chart multiple times with different release names
   - **Repository** -- a place where charts are stored and shared (like DockerHub for images)
   - **Values** -- configuration that customizes a chart for each deployment (replicas, image tag, resource limits)

3. **Why Helm over raw manifests?**
   - Look at the AI-BankApp's `k8s/` directory -- 12 separate YAML files. To change the image tag, you edit `bankapp-deployment.yml`. To switch environments, you manually update ConfigMaps and Secrets. Helm solves this:
   - Templating: one chart serves dev, staging, and prod with different values
   - Versioning: charts have version numbers, you can rollback to previous versions
   - Dependencies: a chart can depend on other charts (your app chart depends on a MySQL chart)
   - Community: thousands of pre-built charts for common software (MySQL, Redis, Prometheus, ArgoCD)

---

What is Helm?:
Helm is a package manager for Kubernetes — like apt for Ubuntu. Instead of managing 12 separate YAML files for one application, you package everything into a chart and deploy it with one command.

Why does AI-BankApp need Helm?:

Look at the AI-BankApp's k8s/ directory:
    
    bankapp-deployment.yml
    configmap.yml
    gateway.yml
    mysql-deployment.yml
    namespace.yml
    ollama-deployment.yml
    pv.yml
    pvc.yml
    secrets.yml
    service.yml
    hpa.yml
    cert-manager.yml

12 files! Every environment (dev, staging, prod) needs different values. Without Helm you manually edit each file. With Helm you just change a values file.
Four core concepts for your notes:

| Concept | What it means | Real example |
|---|---|---|
| Chart | Package of K8s manifest templates | bitnami/mysql |
| Release | One installed instance of a chart | bankapp-mysql |
| Repository | Online collection of charts | charts.bitnami.com |
| Values | Config that customizes the chart | rootPassword, database name |

---

### Task 2: Install Helm and Explore the AI-BankApp
You need a running Kubernetes cluster. Use any of these:
- **Kind** (recommended for this block): Use the AI-BankApp's Kind config
- **Minikube**: `minikube start`
- **Docker Desktop Kubernetes**: enable in settings

**Set up a Kind cluster using the AI-BankApp's config:**
```bash
git clone -b feat/gitops https://github.com/TrainWithShubham/AI-BankApp-DevOps.git
cd AI-BankApp-DevOps

kind create cluster --config setup-k8s/kind-config.yml
```

This creates a cluster with 1 control plane and 2 worker nodes.

**Install Helm:**
```bash
# macOS
brew install helm

# Linux (script)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version
```

Confirm Helm can talk to your cluster:
```bash
kubectl cluster-info
helm list
```

**Explore the raw manifests you will eventually replace with Helm:**
```bash
ls k8s/
```

```
bankapp-deployment.yml   configmap.yml   gateway.yml   mysql-deployment.yml
namespace.yml   ollama-deployment.yml   pv.yml   pvc.yml   secrets.yml
service.yml   hpa.yml   cert-manager.yml
```

12 files -- Deployments, Services, ConfigMaps, Secrets, PVCs, HPA, and more. All hardcoded values. On Day 79, you will convert these into a Helm chart.

---

<img width="1273" height="490" alt="image" src="https://github.com/user-attachments/assets/677bdac3-5b79-40e0-b796-c456c301e408" />

<img width="1103" height="299" alt="image" src="https://github.com/user-attachments/assets/06681d2a-0fbe-4760-b066-19654fe909f8" />

This creates a cluster with 1 control plane + 2 worker nodes — exactly what the AI-BankApp needs.

<img width="1162" height="228" alt="image" src="https://github.com/user-attachments/assets/893c832b-e547-4399-9f69-8954f6bbf39d" />

installed helm: 

<img width="1520" height="292" alt="image" src="https://github.com/user-attachments/assets/f8c0221e-c46c-413a-849a-eb33f6960c17" />
    
    # Look at the MySQL deployment
    cat k8s/mysql-deployment.yml
    
    # Look at secrets
    cat k8s/secrets.yml
    
    # Look at the PVC
    cat k8s/pvc.yml

Notice: all values are hardcoded. Image tags, passwords, resource limits — baked in. This is what Helm replaces.

---

### Task 3: Deploy MySQL Using a Helm Chart
The AI-BankApp needs MySQL. Instead of applying raw YAML like `k8s/mysql-deployment.yml`, deploy it with Helm.

Add the Bitnami chart repository:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Search for MySQL:
```bash
helm search repo bitnami/mysql
```

**Deploy MySQL with the same config the AI-BankApp expects:**
```bash
helm install bankapp-mysql bitnami/mysql \
  --set auth.rootPassword=Test@123 \
  --set auth.database=bankappdb \
  --set primary.resources.requests.memory=256Mi \
  --set primary.resources.requests.cpu=250m \
  --set primary.resources.limits.memory=512Mi \
  --set primary.resources.limits.cpu=500m \
  --set primary.persistence.size=5Gi
```

Compare this single command to the raw manifest approach which needs `mysql-deployment.yml` + `secrets.yml` + `pvc.yml` + `pv.yml` + `service.yml`. Helm handles all of it.

Check what was created:
```bash
helm list
kubectl get all -l app.kubernetes.io/instance=bankapp-mysql
kubectl get pvc -l app.kubernetes.io/instance=bankapp-mysql
kubectl get secret -l app.kubernetes.io/instance=bankapp-mysql
```

Verify MySQL is running:
```bash
kubectl exec -it bankapp-mysql-0 -- mysql -uroot -pTest@123 -e "SHOW DATABASES;"
```

You should see `bankappdb` in the output.

---

Concept First:

The AI-BankApp needs MySQL. Without Helm you'd apply:

mysql-deployment.yml
secrets.yml
pvc.yml
pv.yml
service.yml

With Helm: one command.

<img width="1163" height="200" alt="image" src="https://github.com/user-attachments/assets/b3faeff8-35d7-4089-a2c3-2f70bd23cf95" />


<img width="1179" height="763" alt="image" src="https://github.com/user-attachments/assets/7665f893-ba03-4858-b32d-43dcfee68cf4" />


<img width="1488" height="76" alt="image" src="https://github.com/user-attachments/assets/993e5980-f817-4a55-bd1e-6b4d3006003f" />


<img width="1469" height="277" alt="image" src="https://github.com/user-attachments/assets/76321c94-a504-49e7-8f12-54c859f22f7c" />

<img width="1659" height="398" alt="image" src="https://github.com/user-attachments/assets/0ce10f44-c547-463e-942c-bfd7e008b95f" />

Notice Helm created the StatefulSet, Service, PVC, AND Secret — all from one command.

deployed my-sql using helm:

helm uninstall bankapp-mysql

    helm install bankapp-mysql bitnami/mysql \
    --set auth.rootPassword=Test@123 \
    --set auth.database=bankappdb \
    --set image.registry=docker.io \
    --set image.repository=bitnamilegacy/mysql \
    --set image.tag=8.0 \
    --set global.security.allowInsecureImages=true \
    --set primary.resources.requests.memory=256Mi \
    --set primary.resources.requests.cpu=250m \
    --set primary.resources.limits.memory=512Mi \
    --set primary.resources.limits.cpu=500m \
    --set primary.persistence.size=5Gi

the original problem was:

    Bitnami MySQL image was removed/restricted

so Kubernetes could not pull the container image and pod stayed in ImagePullBackOff.

Then you switched to the free bitnamilegacy/mysql image, which is compatible with the Bitnami Helm chart, so now:

image pulls successfully
init containers work
MySQL pod starts normally.

thats why used different command.

<img width="1517" height="101" alt="image" src="https://github.com/user-attachments/assets/5fb9ec0a-7991-4ec4-8a7d-1641fe5e8938" />

<img width="1604" height="328" alt="image" src="https://github.com/user-attachments/assets/aae8f0fe-3f4d-4d22-9910-9a20e3620863" />

Noticed Helm created the StatefulSet, Service, PVC, AND Secret — all from one command.

<img width="1190" height="246" alt="image" src="https://github.com/user-attachments/assets/183171d4-825c-49b3-ba64-f56946a1445d" />

MySQL running with bankappdb visible.

---

### Task 4: Customize a Deployment with Values Files
`--set` works for quick overrides, but real projects use values files.

Create `mysql-values.yaml`:
```yaml
auth:
  rootPassword: Test@123
  database: bankappdb
primary:
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 250m
      memory: 256Mi
  persistence:
    size: 5Gi
    storageClass: ""
metrics:
  enabled: true
  serviceMonitor:
    enabled: false
```

Deploy with the values file:
```bash
helm install bankapp-mysql-v2 bitnami/mysql -f mysql-values.yaml
```

**To see all configurable values for a chart:**
```bash
helm show values bitnami/mysql | head -80
```

This is your reference for every knob you can turn. Notice how the chart supports metrics, replication, custom init scripts, and dozens more options -- all through values.

**Clean up the second release:**
```bash
helm uninstall bankapp-mysql-v2
```

---

Concept First
--set is fine for 1-2 values. But for production you want a file — version-controlled, reviewable, repeatable.

<img width="1887" height="1083" alt="image" src="https://github.com/user-attachments/assets/cfb765ec-106e-4c91-9a84-1cce406ebde9" />

This shows EVERYTHING you can configure — dozens of options including replication, metrics, backup, custom scripts, init containers.

vim mysql-values.yaml:

<img width="562" height="587" alt="image" src="https://github.com/user-attachments/assets/1bbf6949-8f75-4a9b-a459-253cd3c7d72c" />

<img width="1588" height="263" alt="image" src="https://github.com/user-attachments/assets/645ebac1-6306-4317-a90e-a75da0967570" />

<img width="678" height="500" alt="image" src="https://github.com/user-attachments/assets/30aaffdf-cd69-4c1f-b069-01193e139b63" />

<img width="1449" height="117" alt="image" src="https://github.com/user-attachments/assets/34ce061b-9f2b-4ab7-bffe-250218a3fbd5" />

---

### Task 5: Manage Releases -- Upgrade, Rollback, Uninstall
Helm tracks every change as a **revision**. This lets you upgrade and rollback safely.

**Upgrade MySQL to enable metrics:**
```bash
helm upgrade bankapp-mysql bitnami/mysql \
  --set auth.rootPassword=Test@123 \
  --set auth.database=bankappdb \
  --set metrics.enabled=true
```

Check the revision history:
```bash
helm history bankapp-mysql
```

You should see revision 1 (original) and revision 2 (metrics enabled).

**Rollback to the previous version:**
```bash
helm rollback bankapp-mysql 1
```

Check history again:
```bash
helm history bankapp-mysql
```

Revision 3 appears -- a rollback to revision 1.

**Compare this to raw manifests:** With `kubectl apply`, there is no built-in rollback. You would have to `git revert` or manually re-apply old YAML. Helm gives you `helm rollback` out of the box.

---

Concept First: — Revision History

Install → Revision 1
Upgrade → Revision 2
Rollback → Revision 3 (copy of Revision 1)

Helm never overwrites history. Every change creates a new revision. This is why helm rollback is so powerful — you can always go back.

Compare to raw kubectl:

| | kubectl apply | helm upgrade |
|---|---|---|
| Rollback | Manual git revert + re-apply | helm rollback release 1 |
| History | No built-in history | Full revision history |
| Diff preview | No | helm diff (plugin) |

executed:
    
    helm upgrade bankapp-mysql bitnami/mysql \
      --reuse-values \
      --set primary.resources.limits.memory=1Gi
      
<img width="1896" height="415" alt="image" src="https://github.com/user-attachments/assets/550e9bc1-8372-4b40-973d-e6ca7da7efb3" />

here we are using a different command than mentioned one since:

Because the chart version you're using (14.0.3) generates a StatefulSet change that Kubernetes considers immutable.

The error is not:

    metrics.enabled=true is wrong

The error is:

    the chart-generated StatefulSet changed an immutable field

