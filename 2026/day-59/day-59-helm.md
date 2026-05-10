# Day 59 – Helm — Kubernetes Package Manager

## Task
Over the past eight days you have written Deployments, Services, ConfigMaps, Secrets, PVCs, and more — all as individual YAML files. For a real application you might have dozens of these. Helm is the package manager for Kubernetes, like apt for Ubuntu. Today you install charts, customize them, and create your own.

---

What Are We Learning Today?
Think about this — over the last 8 days you wrote:

Deployment YAML
Service YAML
ConfigMap YAML
Secret YAML
PVC YAML
HPA YAML

That's 6+ files just for one app. In real production, it could be 20-30 files. And every time you want to deploy the same app in a different environment (dev, staging, prod) you copy-paste and manually change values.
Helm fixes this.
Helm is like apt for Ubuntu or npm for Node.js — but for Kubernetes. Instead of managing 20 YAML files, you install one Chart with one command and customize it with a simple values file.

3 Core Concepts:


| Term | What it means | Real world analogy |
|---|---|---|
| Chart | Package of Kubernetes manifest templates | Like a .deb package in Ubuntu |
| Release | One installed instance of a chart in your cluster | Like an installed app |
| Repository | Collection of charts hosted online | Like apt repository |


---

## Challenge Tasks

### Task 1: Install Helm
1. Install Helm (brew, curl script, or chocolatey depending on your OS)
2. Verify with `helm version` and `helm env`

Three core concepts:
- **Chart** — a package of Kubernetes manifest templates
- **Release** — a specific installation of a chart in your cluster
- **Repository** — a collection of charts (like a package repo)

**Verify:** What version of Helm is installed?

---

Concept First:

Helm is a CLI tool you install on your machine (not inside Kubernetes). It talks to your cluster via kubectl context — 
so whatever cluster kubectl points to, that's where Helm deploys.


<img width="1428" height="559" alt="image" src="https://github.com/user-attachments/assets/d18c2626-3fef-4045-9439-6c4cc45fb006" />

<img width="1421" height="268" alt="image" src="https://github.com/user-attachments/assets/1a5581ce-1132-40c6-8fba-1a32d51978cd" />

<img width="960" height="511" alt="image" src="https://github.com/user-attachments/assets/3ab9e40a-b283-4800-aa79-bfb2526e3ab7" />

version is: v4.1.4

---

### Task 2: Add a Repository and Search
1. Add the Bitnami repository: `helm repo add bitnami https://charts.bitnami.com/bitnami`
2. Update: `helm repo update`
3. Search: `helm search repo nginx` and `helm search repo bitnami`

**Verify:** How many charts does Bitnami have?

---

Concept First:

By default Helm has no repositories — like a fresh Ubuntu install with no apt sources. You add repos to tell Helm where to find charts.
Bitnami is the most popular Helm chart repository — they maintain production-ready charts for nginx, MySQL, PostgreSQL, Redis, WordPress, and hundreds more.


installed and checked repo related activities with helm:

<img width="1427" height="715" alt="image" src="https://github.com/user-attachments/assets/626c5bf9-9c5b-47cc-952c-81f5889f43db" />

Added repos list:

<img width="722" height="66" alt="image" src="https://github.com/user-attachments/assets/441a6a3d-38d8-4c3c-96e1-0ea46d66c331" />

---

### Task 3: Install a Chart
1. Deploy nginx: `helm install my-nginx bitnami/nginx`
2. Check what was created: `kubectl get all`
3. Inspect the release: `helm list`, `helm status my-nginx`, `helm get manifest my-nginx`

One command replaced writing a Deployment, Service, and ConfigMap by hand.

**Verify:** How many Pods are running? What Service type was created?

---


<img width="1424" height="659" alt="image" src="https://github.com/user-attachments/assets/9b084766-3fa0-49bd-89e4-a4bced78cc95" />


<img width="934" height="661" alt="image" src="https://github.com/user-attachments/assets/b30e82ea-3c80-42ea-9070-0a70a8abfc6b" />


<img width="603" height="490" alt="image" src="https://github.com/user-attachments/assets/cef601a4-e67d-4c57-a66e-b4b2f1d59e6f" />


1-pod, 1-deployment, 1-replica,... service type was load balancer.


---

### Task 4: Customize with Values
1. View defaults: `helm show values bitnami/nginx`
2. Install a custom release with `--set replicaCount=3 --set service.type=NodePort`
3. Create a `custom-values.yaml` file with replicaCount, service type, and resource limits
4. Install another release using `-f custom-values.yaml`
5. Check overrides: `helm get values <release-name>`

**Verify:** Does the values file release have the correct replicas and service type?

---

Concept First
Every chart has a values.yaml file with defaults. You can override any value two ways:

--set key=value → quick single override on command line
-f custom-values.yaml → file with multiple overrides (better for real use)

For nested values use dots: service.type=NodePort means:

    service:
      type: NodePort


Saw all available values for bitnami/nginx

    helm show values bitnami/nginx

created with --set overrides:

<img width="1433" height="666" alt="image" src="https://github.com/user-attachments/assets/6238ca57-1236-4acb-926d-b592816c35a2" />

created custom-values.yaml

<img width="729" height="286" alt="image" src="https://github.com/user-attachments/assets/23813c3a-7aa0-4001-9d93-f3013b07a37d" />

deployed using yaml file:

<img width="1205" height="245" alt="image" src="https://github.com/user-attachments/assets/f0694d9e-7118-4bc5-9113-be20657cfd16" />

Check what values were applied:

<img width="813" height="242" alt="image" src="https://github.com/user-attachments/assets/40963ba7-3560-463d-b74a-b2494e8dc59f" />

This shows only your overrides (not defaults).

This shows ALL values including defaults.:

<img width="374" height="399" alt="image" src="https://github.com/user-attachments/assets/ffd02912-2e99-41ce-a60a-533b9da586b6" />

<img width="707" height="186" alt="image" src="https://github.com/user-attachments/assets/9c95fa03-5626-478c-98a1-bbfc5f8ed6f9" />

helm get values my-nginx-file shows replicaCount: 3 and service.type: NodePort.

---

### Task 5: Upgrade and Rollback
1. Upgrade: `helm upgrade my-nginx bitnami/nginx --set replicaCount=5`
2. Check history: `helm history my-nginx`
3. Rollback: `helm rollback my-nginx 1`
4. Check history again — rollback creates a new revision (3), not overwriting revision 2

Same concept as Deployment rollouts from Day 52, but at the full stack level.

**Verify:** How many revisions after the rollback?

---

Concept First
This is like kubectl rollout from Day 52 — but at the full Helm release level. Every change creates a new revision in history.
Revision | Action
1        |  Initial install
2        |  Upgrade (replicaCount=5)
3        |  Rollback to revision 1

Rollback creates revision 3 — it does NOT delete revision 2. Full audit trail is preserved.

<img width="1330" height="647" alt="image" src="https://github.com/user-attachments/assets/c4c46d9b-093a-46ce-a3e9-297b631713b4" />

<img width="1440" height="345" alt="image" src="https://github.com/user-attachments/assets/bc563b04-24b5-4cbf-b7e9-e88f9c570614" />

<img width="1151" height="326" alt="image" src="https://github.com/user-attachments/assets/26475fc2-f926-4b9b-8cf3-19e0d365a7ff" />

After rollback there are 3 revisions in history. Pods went back to original replica count.

---

### Task 6: Create Your Own Chart
1. Scaffold: `helm create my-app`
2. Explore the directory: `Chart.yaml`, `values.yaml`, `templates/deployment.yaml`
3. Look at the Go template syntax in templates: `{{ .Values.replicaCount }}`, `{{ .Chart.Name }}`
4. Edit `values.yaml` — set replicaCount to 3 and image to nginx:1.25
5. Validate: `helm lint my-app`
6. Preview: `helm template my-release ./my-app`
7. Install: `helm install my-release ./my-app`
8. Upgrade: `helm upgrade my-release ./my-app --set replicaCount=5`

**Verify:** After installing, 3 replicas? After upgrading, 5?

---

Concept First:

This is the most important task — understanding what's inside a chart.
When you run helm create my-app it generates this structure:


Concept First
This is the most important task — understanding what's inside a chart.
When you run helm create my-app it generates this structure:

        my-app/
        ├── Chart.yaml          ← Chart metadata (name, version, description)
        ├── values.yaml         ← Default values (users override these)
        └── templates/          ← Kubernetes YAML templates with Go templating
            ├── deployment.yaml
            ├── service.yaml
            ├── _helpers.tpl    ← Reusable template snippets
            └── ...


Go templating lets you use variables in your YAML:

    replicas: {{ .Values.replicaCount }}    ← pulls from values.yaml
    name: {{ .Chart.Name }}                 ← pulls from Chart.yaml
    name: {{ .Release.Name }}              ← the name you give at install

So the same template file generates different YAML depending on what values you pass in.

<img width="869" height="625" alt="image" src="https://github.com/user-attachments/assets/749c842f-dadf-4029-911b-ab47c66e8eb9" />

Notice the {{ .Values.xxx }} placeholders — these get replaced with real values at install time.

cat my-app/Charts.yaml:

<img width="1003" height="504" alt="image" src="https://github.com/user-attachments/assets/facfd84d-77af-463e-853c-61b5237be78f" />

This has the chart name, version, and description.

edited these values:

<img width="897" height="311" alt="image" src="https://github.com/user-attachments/assets/94c87fab-52da-435e-b1f8-a9694c3e392c" />

Validate the chart:

helm lint my-app
 
    Output:
    ==> Linting my-app
    [INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
No errors = chart is valid!

Preview what Kubernetes YAML will be generated (without installing):

helm template my-release ./my-app

This prints all the rendered YAML — great for debugging before you actually apply anything.

Install your custom chart:

<img width="1432" height="288" alt="image" src="https://github.com/user-attachments/assets/be986e92-5f96-45cb-bdcc-58d23a6b0b80" />

Verified with pods:

<img width="754" height="246" alt="image" src="https://github.com/user-attachments/assets/575d9f97-436e-45e5-843a-c428f17c5e76" />

set replica to 5:

<img width="1426" height="584" alt="image" src="https://github.com/user-attachments/assets/3b18f307-b3f1-4c9c-8843-b3d12e85a858" />

---

### Task 7: Clean Up
1. Uninstall all releases: `helm uninstall <name>` for each
2. Remove chart directory and values file
3. Use `--keep-history` if you want to retain release history for auditing

**Verify:** Does `helm list` show zero releases?

---


<img width="1348" height="629" alt="image" src="https://github.com/user-attachments/assets/ba708027-aca7-4231-a8da-cad5beef3bb8" />


---

## What is Helm?
Helm is the package manager for Kubernetes — like apt for Ubuntu or npm for Node.js.
Instead of managing 20+ individual YAML files per app, you install one Chart
with one command and customize it with a values file.

## Three Core Concepts

| Term | What it means |
|---|---|
| Chart | Package of Kubernetes manifest templates |
| Release | One installed instance of a chart in your cluster |
| Repository | Collection of charts hosted online (like Bitnami) |

## How to Install, Customize, Upgrade, Rollback

| Action | Command |
|---|---|
| Install | `helm install <release> <chart>` |
| Customize (CLI) | `helm install <release> <chart> --set key=value` |
| Customize (file) | `helm install <release> <chart> -f values.yaml` |
| Upgrade | `helm upgrade <release> <chart>` |
| Rollback | `helm rollback <release> <revision>` |
| History | `helm history <release>` |
| Uninstall | `helm uninstall <release>` |

## Helm Chart Structure

        my-app/
        ├── Chart.yaml        ← Chart metadata (name, version)
        ├── values.yaml       ← Default values (users override these)
        └── templates/        ← Kubernetes YAML with Go templating
        ├── deployment.yaml
        └── service.yaml


## Go Templating
Templates use {{ }} syntax to inject values at install time:
- {{ .Values.replicaCount }} → from values.yaml
- {{ .Chart.Name }} → from Chart.yaml
- {{ .Release.Name }} → the name given at helm install

## Upgrade and Rollback
- Every change creates a new revision in history
- Rollback creates a NEW revision — it never overwrites history
- Full audit trail is always preserved

## Useful Commands
- `helm show values <chart>` → see all customizable values
- `helm template <release> <chart>` → preview YAML without installing
- `helm lint <chart>` → validate chart before installing
- `helm get values <release>` → see what overrides were applied
- `helm get manifest <release>` → see all generated Kubernetes YAML


| Concept | What it means |
|---|---|
| Chart | Package of Kubernetes templates — like a .deb package |
| Release | One installed instance of a chart in your cluster |
| Repository | Online collection of charts (Bitnami, ArtifactHub) |
| `helm install` | Deploy a chart as a new release |
| `--set key=value` | Quick single value override on command line |
| `-f values.yaml` | File-based overrides — better for multiple values |
| `helm upgrade` | Update a release — creates new revision in history |
| `helm rollback` | Revert to a previous revision — creates a NEW revision |
| `helm history` | Full audit trail of all revisions for a release |
| `helm lint` | Validate chart structure before installing |
| `helm template` | Preview generated YAML without installing anything |
| Go templating | `{{ .Values.key }}` syntax — injects values into YAML templates |
