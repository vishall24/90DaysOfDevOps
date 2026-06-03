# Day 79 -- Creating a Custom Helm Chart for AI-BankApp

## Task
Yesterday you deployed MySQL with a community Helm chart. Today you build a custom Helm chart for the AI-BankApp itself -- converting the 12 raw YAML files from the `k8s/` directory into a templated, configurable, reusable Helm chart.

The AI-BankApp (https://github.com/TrainWithShubham/AI-BankApp-DevOps, branch `feat/gitops`) has three services: the Spring Boot banking app, a MySQL database, and an Ollama AI chatbot. By the end of today, all of this will be deployable with a single `helm install` command.

---

Yesterday you used a community chart (Bitnami MySQL).
Today you build your own chart from scratch.The transformation:BEFORE (12 raw YAML files):              AFTER (1 Helm chart):

      k8s/bankapp-deployment.yml               helm install my-bankapp bankapp/
      k8s/mysql-deployment.yml                   --set bankapp.image.tag=v2
      k8s/ollama-deployment.yml                  --set ollama.enabled=false
      k8s/configmap.yml                          -n bankapp --create-namespace
      k8s/secrets.yml
      k8s/pvc.yml                              DONE. Entire stack deployed.
      k8s/pv.yml
      k8s/service.yml
      k8s/hpa.yml
      k8s/namespace.yml
      k8s/gateway.yml
      k8s/cert-manager.yml
      
The key Helm template concepts you'll use today:

| Syntax | What it does | Example |
|---|---|---|
| `{{ .Values.key }}` | Pull from values.yaml | `{{ .Values.bankapp.image.tag }}` |
| `{{ include "func" . }}` | Call a helper function | `{{ include "bankapp.fullname" . }}` |
| `{{- if .Values.key }}` | Conditional block | Only render if ollama.enabled = true |
| `{{ toYaml . \| nindent 12 }}` | Render nested YAML | Resource limits block |
| `{{ .b64enc }}` | Base64 encode | Auto-encode secrets |
| `{{ .Release.Namespace }}` | Current namespace | Where helm install was run |



---

## Challenge Tasks

### Task 1: Scaffold the Chart and Study the Raw Manifests
Make sure you have the AI-BankApp repo cloned:
```bash
cd AI-BankApp-DevOps
```

Study the raw manifests you are converting:
```bash
ls k8s/
```

Map each file to what it does:

| File | Purpose |
|------|---------|
| `namespace.yml` | Creates `bankapp` namespace |
| `configmap.yml` | MySQL host, port, database, Ollama URL |
| `secrets.yml` | MySQL credentials (base64 encoded) |
| `pv.yml` | StorageClass (gp3 via EBS CSI) |
| `pvc.yml` | PVCs for MySQL (5Gi) and Ollama (10Gi) |
| `bankapp-deployment.yml` | BankApp with init containers, probes, envFrom |
| `mysql-deployment.yml` | MySQL with EBS volume mount, probes |
| `ollama-deployment.yml` | Ollama with postStart model pull, probes |
| `service.yml` | ClusterIP services for all 3 components |
| `hpa.yml` | HPA for BankApp (2-4 replicas, 70% CPU) |
| `gateway.yml` | Envoy Gateway + HTTPRoute + TLS |
| `cert-manager.yml` | Let's Encrypt ClusterIssuer |

Now scaffold a Helm chart:
```bash
mkdir helm-chart && cd helm-chart
helm create bankapp
```

Delete the generated template files -- you will write your own from the raw manifests:
```bash
rm -rf bankapp/templates/*.yaml bankapp/templates/tests/
```

Keep `_helpers.tpl` and `NOTES.txt` -- you will customize them.

---

    ubuntu@ip-172-31-20-173:~$ cd AI-BankApp-DevOps/
    ubuntu@ip-172-31-20-173:~/AI-BankApp-DevOps$ ls k8s/
    bankapp-deployment.yml  configmap.yml  hpa.yml               namespace.yml          pv.yml   secrets.yml
    cert-manager.yml        gateway.yml    mysql-deployment.yml  ollama-deployment.yml  pvc.yml  service.yml
    ubuntu@ip-172-31-20-173:~/AI-BankApp-DevOps$ # Read the secrets
    cat k8s/secrets.yml
    apiVersion: v1
    kind: Secret
    metadata:
      name: bankapp-secret
      namespace: bankapp
    type: Opaque
    data:
      MYSQL_ROOT_PASSWORD: VGVzdEAxMjM=   # Test@123
      MYSQL_USER: cm9vdA==                 # root
      MYSQL_PASSWORD: VGVzdEAxMjM=         # Test@123
    ubuntu@ip-172-31-20-173:~/AI-BankApp-DevOps$ # Read the bankapp deployment
    cat k8s/bankapp-deployment.yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: bankapp
      namespace: bankapp
    spec:
      replicas: 4
      selector:
        matchLabels:
          app: bankapp
      template:
        metadata:
          labels:
            app: bankapp
        spec:
          initContainers:
            - name: wait-for-mysql
              image: busybox:1.36
              command: ["/bin/sh", "-c", "until nc -z mysql-service 3306; do sleep 2; done"]
              resources:
                requests:
                  memory: "32Mi"
                  cpu: "50m"
                limits:
                  memory: "64Mi"
                  cpu: "100m"
            - name: wait-for-ollama
              image: busybox:1.36
              command: ["/bin/sh", "-c", "until nc -z ollama-service 11434; do sleep 2; done"]
              resources:
                requests:
                  memory: "32Mi"
                  cpu: "50m"
                limits:
                  memory: "64Mi"
                  cpu: "100m"
          containers:
            - name: bankapp
              image: trainwithshubham/ai-bankapp-eks:1c7cb0e
              imagePullPolicy: Always
              ports:
                - containerPort: 8080
              envFrom:
                - configMapRef:
                    name: bankapp-config
                - secretRef:
                    name: bankapp-secret
              resources:
                requests:
                  memory: "256Mi"
                  cpu: "250m"
                limits:
                  memory: "512Mi"
                  cpu: "500m"
              readinessProbe:
                httpGet:
                  path: /actuator/health
                  port: 8080
                initialDelaySeconds: 30
                failureThreshold: 15
              livenessProbe:
                httpGet:
                  path: /actuator/health
                  port: 8080
                initialDelaySeconds: 60
                periodSeconds: 10
                failureThreshold: 5
    ubuntu@ip-172-31-20-173:~/AI-BankApp-DevOps$ # Read the HPA
    cat k8s/hpa.yml
    # ─── Horizontal Pod Autoscaler — BankApp ─────────────────────────
    # Requires metrics-server to be installed in the cluster.
    # Sized for t3.medium (2 vCPU / 4 GiB): max 4 replicas keeps
    # total workload requests under node capacity.
    # ─────────────────────────────────────────────────────────────────
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata:
      name: bankapp-hpa
      namespace: bankapp
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: bankapp
      minReplicas: 2
      maxReplicas: 4
      metrics:
        - type: Resource
          resource:
            name: cpu
            target:
              type: Utilization
              averageUtilization: 70
      behavior:
        scaleUp:
          stabilizationWindowSeconds: 30
          policies:
            - type: Pods
              value: 2
              periodSeconds: 60
        scaleDown:
          stabilizationWindowSeconds: 300
          policies:
            - type: Pods
              value: 1
              periodSeconds: 60


Notice hardcoded replica counts and CPU targets.
Now you understand what you're converting. Let's build the chart.

<img width="1361" height="404" alt="image" src="https://github.com/user-attachments/assets/00b907fa-8a0b-44bf-995c-8b797f2bb04f" />

    bankapp/
      Chart.yaml
      values.yaml
      charts/
      templates/
        deployment.yaml
        service.yaml
        ingress.yaml
        hpa.yaml
        serviceaccount.yaml
        _helpers.tpl
        NOTES.txt
        tests/

<img width="1028" height="76" alt="image" src="https://github.com/user-attachments/assets/29d524ff-598b-420e-85fd-fc55fd72a521" />

 Scaffold done — clean slate, ready to write templates.

---

### Task 2: Define Chart.yaml and values.yaml
Edit `bankapp/Chart.yaml`:
```yaml
apiVersion: v2
name: bankapp
description: AI-BankApp -- Spring Boot banking application with MySQL and Ollama AI chatbot
type: application
version: 0.1.0
appVersion: "1.0.0"
maintainers:
  - name: TrainWithShubham
    url: https://github.com/TrainWithShubham
keywords:
  - bankapp
  - spring-boot
  - mysql
  - ollama
  - ai
```

Now create `bankapp/values.yaml` -- extract every hardcoded value from the raw manifests into configurable values:
```yaml
# BankApp configuration
bankapp:
  replicaCount: 4
  image:
    repository: trainwithshubham/ai-bankapp-eks
    tag: "latest"
    pullPolicy: Always
  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  service:
    type: ClusterIP
    port: 8080
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
    targetCPUUtilization: 70

# MySQL configuration
mysql:
  enabled: true
  image:
    repository: mysql
    tag: "8.0"
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

# Ollama AI configuration
ollama:
  enabled: true
  image:
    repository: ollama/ollama
    tag: "latest"
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

# Shared configuration
config:
  mysqlDatabase: bankappdb
  ollamaUrl: ""  # Auto-generated from service name if empty

# Secrets
secrets:
  mysqlRootPassword: Test@123
  mysqlUser: root
  mysqlPassword: Test@123

# Storage
storageClass:
  create: true
  name: gp3
  provisioner: ebs.csi.aws.com

# Gateway (optional -- for EKS with Envoy Gateway)
gateway:
  enabled: false
  hostname: ""
  tls:
    enabled: false
```

**Compare:** The raw `k8s/secrets.yml` has base64-encoded credentials hardcoded. The Helm chart uses `values.yaml` and templates the Secret, so each environment can override credentials without editing YAML.

---

vim bankap/Chart.yaml:

<img width="1069" height="336" alt="image" src="https://github.com/user-attachments/assets/31502d3f-ff9b-4456-9efa-911fb6555f54" />

vim bankap/values.yaml:

    # ── BankApp (Spring Boot) ──────────────────────────────────────────────
    bankapp:
      replicaCount: 4
      image:
        repository: trainwithshubham/ai-bankapp-eks
        tag: "latest"
        pullPolicy: Always
      resources:
        requests:
          memory: "256Mi"
          cpu: "250m"
        limits:
          memory: "512Mi"
          cpu: "500m"
      service:
        type: ClusterIP
        port: 8080
      autoscaling:
        enabled: true
        minReplicas: 2
        maxReplicas: 4
        targetCPUUtilization: 70
    
    # ── MySQL ──────────────────────────────────────────────────────────────
    mysql:
      enabled: true
      image:
        repository: mysql
        tag: "8.0"
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
    
    # ── Ollama AI Chatbot ─────────────────────────────────────────────────
    ollama:
      enabled: true
      image:
        repository: ollama/ollama
        tag: "latest"
      model: tinyllama            # Change this to switch AI models
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
    
    # ── Shared Config ─────────────────────────────────────────────────────
    config:
      mysqlDatabase: bankappdb
      ollamaUrl: ""               # Auto-generated from service name if empty
    
    # ── Secrets ───────────────────────────────────────────────────────────
    secrets:
      mysqlRootPassword: Test@123
      mysqlUser: root
      mysqlPassword: Test@123
    
    # ── StorageClass (AWS EBS gp3) ────────────────────────────────────────
    storageClass:
      create: true
      name: gp3
      provisioner: ebs.csi.aws.com
    
    # ── Gateway (optional — EKS with Envoy) ──────────────────────────────
    gateway:
      enabled: false
      hostname: ""
      tls:
        enabled: false

Why this is better than raw YAML:
The raw k8s/secrets.yml has base64 passwords hardcoded. This values.yaml has plain text passwords that Helm auto-encodes. Want different passwords per environment? Just
change values.yaml.


<img width="1017" height="235" alt="image" src="https://github.com/user-attachments/assets/e49bf566-975e-4be4-9e5b-b540bff72bd9" />

---

### Task 3: Write the Core Templates
Convert the raw manifests into Helm templates. Each template uses `{{ .Values }}` instead of hardcoded values.

**`bankapp/templates/configmap.yaml`** (from `k8s/configmap.yml`):
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "bankapp.fullname" . }}-config
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "bankapp.labels" . | nindent 4 }}
data:
  MYSQL_HOST: {{ include "bankapp.fullname" . }}-mysql
  MYSQL_PORT: "3306"
  MYSQL_DATABASE: {{ .Values.config.mysqlDatabase | quote }}
  OLLAMA_URL: {{ default (printf "http://%s-ollama:11434" (include "bankapp.fullname" .)) .Values.config.ollamaUrl | quote }}
  SERVER_FORWARD_HEADERS_STRATEGY: "native"
```

**`bankapp/templates/secrets.yaml`** (from `k8s/secrets.yml`):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "bankapp.fullname" . }}-secret
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "bankapp.labels" . | nindent 4 }}
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: {{ .Values.secrets.mysqlRootPassword | b64enc | quote }}
  MYSQL_USER: {{ .Values.secrets.mysqlUser | b64enc | quote }}
  MYSQL_PASSWORD: {{ .Values.secrets.mysqlPassword | b64enc | quote }}
```

Notice: `b64enc` automatically base64 encodes the values. No more manual encoding.

**`bankapp/templates/storage.yaml`** (from `k8s/pv.yml` + `k8s/pvc.yml`):
```yaml
{{- if .Values.storageClass.create }}
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: {{ .Values.storageClass.name }}
provisioner: {{ .Values.storageClass.provisioner }}
parameters:
  type: gp3
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
{{- end }}
---
{{- if .Values.mysql.enabled }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "bankapp.fullname" . }}-mysql-pvc
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "bankapp.labels" . | nindent 4 }}
spec:
  storageClassName: {{ .Values.mysql.persistence.storageClass }}
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: {{ .Values.mysql.persistence.size }}
{{- end }}
---
{{- if .Values.ollama.enabled }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "bankapp.fullname" . }}-ollama-pvc
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "bankapp.labels" . | nindent 4 }}
spec:
  storageClassName: {{ .Values.ollama.persistence.storageClass }}
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: {{ .Values.ollama.persistence.size }}
{{- end }}
```

---

<img width="1020" height="233" alt="image" src="https://github.com/user-attachments/assets/02109aeb-ccad-430f-8f68-f6e5e76a4e33" />

Explanation of template syntax used:

{{ include "bankapp.fullname" . }} → calls the _helpers.tpl function that generates the full release name
{{ .Release.Namespace }} → the namespace where helm install runs
{{ .Values.config.mysqlDatabase | quote }} → pulls value and wraps in quotes
{{ default "fallback" .Values.config.ollamaUrl }} → uses fallback if value is empty

vim bankapp/templates/secrets.yaml:

<img width="635" height="211" alt="image" src="https://github.com/user-attachments/assets/6a7b5ac8-e12f-48d6-84eb-6be591261d11" />

Key feature: b64enc automatically base64 encodes the values from values.yaml. The raw k8s/secrets.yml required manual encoding — very error-prone. Helm does it for you.

vim bankapp/templates/storage.yaml:

<img width="538" height="764" alt="image" src="https://github.com/user-attachments/assets/4a4a0420-4b81-44e1-a66b-f68b7dbf83dd" />

Power of conditionals: {{- if .Values.mysql.enabled }} — if someone sets mysql.enabled=false, the entire PVC block disappears from the rendered YAML.

vim bankapp/templates/bankapp-deployment.yaml:

<img width="964" height="1027" alt="image" src="https://github.com/user-attachments/assets/b967307c-8318-44fa-a4eb-3d91b9dbd956" />

Three smart template decisions here:

{{- if not .Values.bankapp.autoscaling.enabled }} — when HPA manages replicas, the replicas field is omitted (HPA and static replicas conflict)
Ollama init container is conditional — disable Ollama and the wait loop disappears
{{- with .Values.bankapp.resources }} + toYaml . | nindent 12 — cleanly renders the entire resources block from values

vim bankapp/templates/mysql-deployment.yaml:

<img width="698" height="942" alt="image" src="https://github.com/user-attachments/assets/b661f640-82dd-4b63-83ab-910d5adb035d" />

vim bankapp/templates/ollama-deployment.yaml:

<img width="746" height="930" alt="image" src="https://github.com/user-attachments/assets/4e4d195a-d2af-4f6e-8207-95454d78b6c5" />

Key feature: {{ .Values.ollama.model }} in both the postStart hook and readiness probe means switching from tinyllama to llama3 requires zero template edits — just change the value.
    
    vim bankapp/templates/services.yaml:
    
    <img width="442" height="803" alt="image" src="https://github.com/user-attachments/assets/3720481b-eeac-4a23-a190-4ee7af2c7a97" />
    
    vim bankapp/templates/hpa.yaml:
    
    <img width="681" height="598" alt="image" src="https://github.com/user-attachments/assets/4c0ecdb5-6c61-4dbe-bfc3-b7d485fa2601" />
    
    vim bankapp/templates/NOTES.txt:
    
    <img width="782" height="307" alt="image" src="https://github.com/user-attachments/assets/36d774d5-3c9c-486f-b527-abf0218e906b" />
    
    If you see errors — fix the YAML indentation in the relevant template file.
    
    ubuntu@ip-172-31-20-173:~/helm-chart$ helm template my-bankapp bankapp/
    ---
    # Source: bankapp/templates/secrets.yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: my-bankapp-secret
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    type: Opaque
    data:
      MYSQL_ROOT_PASSWORD: "VGVzdEAxMjM="
      MYSQL_USER: "cm9vdA=="
      MYSQL_PASSWORD: "VGVzdEAxMjM="
    ---
    # Source: bankapp/templates/configmap.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: my-bankapp-config
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    data:
      MYSQL_HOST: my-bankapp-mysql
      MYSQL_PORT: "3306"
      MYSQL_DATABASE: "bankappdb"
      OLLAMA_URL: "http://my-bankapp-ollama:11434"
      SERVER_FORWARD_HEADERS_STRATEGY: "native"
    ---
    # Source: bankapp/templates/storage.yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: gp3
    provisioner: ebs.csi.aws.com
    parameters:
      type: gp3
      fsType: ext4
    reclaimPolicy: Delete
    volumeBindingMode: WaitForFirstConsumer
    allowVolumeExpansion: true
    ---
    # Source: bankapp/templates/storage.yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-bankapp-mysql-pvc
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      storageClassName: gp3
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
    ---
    # Source: bankapp/templates/storage.yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-bankapp-ollama-pvc
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      storageClassName: gp3
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
    ---
    # Source: bankapp/templates/services.yaml
    # MySQL Service — always created when mysql is enabled
    apiVersion: v1
    kind: Service
    metadata:
      name: my-bankapp-mysql
      namespace: default
    spec:
      selector:
        app: my-bankapp-mysql
      ports:
        - port: 3306
          targetPort: 3306
    ---
    # Source: bankapp/templates/services.yaml
    # Ollama Service — only when ollama is enabled
    apiVersion: v1
    kind: Service
    metadata:
      name: my-bankapp-ollama
      namespace: default
    spec:
      selector:
        app: my-bankapp-ollama
      ports:
        - port: 11434
          targetPort: 11434
    ---
    # Source: bankapp/templates/services.yaml
    # BankApp Service — always created
    apiVersion: v1
    kind: Service
    metadata:
      name: my-bankapp-service
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      type: ClusterIP
      sessionAffinity: ClientIP
      sessionAffinityConfig:
        clientIP:
          timeoutSeconds: 3600
      selector:
        app: my-bankapp
      ports:
        - port: 8080
          targetPort: 8080
    ---
    # Source: bankapp/templates/bankapp-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-bankapp
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      selector:
        matchLabels:
          app: my-bankapp
      template:
        metadata:
          labels:
            app: my-bankapp
        spec:
          initContainers:
            - name: wait-for-mysql
              image: busybox:1.36
              command:
               - /bin/sh
               - -c
               - 'until nc -z my-bankapp-mysql 3306; do sleep 2; done'
              resources:
                requests: { memory: "32Mi", cpu: "50m" }
                limits: { memory: "64Mi", cpu: "100m" }
            - name: wait-for-ollama
              image: busybox:1.36
              command:
               - /bin/sh
               - -c
               - 'until nc -z my-bankapp-ollama 11434; do sleep 2; done'
              resources:
                requests: { memory: "32Mi", cpu: "50m" }
                limits: { memory: "64Mi", cpu: "100m" }
          containers:
            - name: bankapp
              image: "trainwithshubham/ai-bankapp-eks:latest"
              imagePullPolicy: Always
              ports:
                - containerPort: 8080
              envFrom:
                - configMapRef:
                    name: my-bankapp-config
                - secretRef:
                    name: my-bankapp-secret
              resources:
                limits:
                  cpu: 500m
                  memory: 512Mi
                requests:
                  cpu: 250m
                  memory: 256Mi
              readinessProbe:
                httpGet:
                  path: /actuator/health
                  port: 8080
                initialDelaySeconds: 30
                failureThreshold: 15
              livenessProbe:
                httpGet:
                  path: /actuator/health
                  port: 8080
                initialDelaySeconds: 60
                periodSeconds: 10
                failureThreshold: 5
    ---
    # Source: bankapp/templates/mysql-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-bankapp-mysql
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      selector:
        matchLabels:
          app: my-bankapp-mysql
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: my-bankapp-mysql
        spec:
          containers:
            - name: mysql
              image: "mysql:8.0"
              ports:
                - containerPort: 3306
              env:
                - name: MYSQL_ROOT_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: my-bankapp-secret
                      key: MYSQL_ROOT_PASSWORD
                - name: MYSQL_DATABASE
                  valueFrom:
                    configMapKeyRef:
                      name: my-bankapp-config
                      key: MYSQL_DATABASE
              resources:
                limits:
                  cpu: 500m
                  memory: 512Mi
                requests:
                  cpu: 250m
                  memory: 256Mi
              volumeMounts:
                - name: mysql-storage
                  mountPath: /var/lib/mysql
              readinessProbe:
                exec:
                  command: ["mysqladmin", "ping", "-h", "localhost"]
                initialDelaySeconds: 15
                failureThreshold: 10
              livenessProbe:
                exec:
                  command: ["mysqladmin", "ping", "-h", "localhost"]
                initialDelaySeconds: 30
                periodSeconds: 10
                failureThreshold: 5
          volumes:
            - name: mysql-storage
              persistentVolumeClaim:
                claimName: my-bankapp-mysql-pvc
    ---
    # Source: bankapp/templates/ollama-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-bankapp-ollama
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      selector:
        matchLabels:
          app: my-bankapp-ollama
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: my-bankapp-ollama
        spec:
          containers:
            - name: ollama
              image: "ollama/ollama:latest"
              ports:
                - containerPort: 11434
              resources:
                limits:
                  cpu: 1500m
                  memory: 2.5Gi
                requests:
                  cpu: 900m
                  memory: 2Gi
              volumeMounts:
                - name: ollama-storage
                  mountPath: /root/.ollama
              lifecycle:
                postStart:
                  exec:
                    command:
                      - /bin/sh
                      - -c
                      - |
                        until ollama list > /dev/null 2>&1; do sleep 2; done
                        ollama pull tinyllama
              readinessProbe:
                exec:
                  command: ["/bin/sh", "-c", "ollama list | grep -q tinyllama"]
                initialDelaySeconds: 30
                failureThreshold: 30
              livenessProbe:
                httpGet:
                  path: /
                  port: 11434
                initialDelaySeconds: 60
                periodSeconds: 10
                failureThreshold: 5
          volumes:
            - name: ollama-storage
              persistentVolumeClaim:
                claimName: my-bankapp-ollama-pvc
    ---
    # Source: bankapp/templates/hpa.yaml
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata:
      name: my-bankapp-hpa
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: my-bankapp
      minReplicas: 2
      maxReplicas: 4
      metrics:
        - type: Resource
          resource:
            name: cpu
            target:
              type: Utilization
              averageUtilization: 70
      behavior:
        scaleUp:
          stabilizationWindowSeconds: 30
          policies:
            - type: Pods
              value: 2
              periodSeconds: 60
        scaleDown:
          stabilizationWindowSeconds: 300
          policies:
            - type: Pods
              value: 1
              periodSeconds: 60

This prints all generated YAML. Read through it carefully:

Are all {{ }} blocks resolved?
Do resource names look right?
Are secrets base64 encoded?
    
Executed:
   
  # Disable Ollama — watch everything Ollama-related disappear
  helm template my-bankapp bankapp/ --set ollama.enabled=false
   
    ---
    # Source: bankapp/templates/secrets.yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: my-bankapp-secret
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    type: Opaque
    data:
      MYSQL_ROOT_PASSWORD: "VGVzdEAxMjM="
      MYSQL_USER: "cm9vdA=="
      MYSQL_PASSWORD: "VGVzdEAxMjM="
    ---
    # Source: bankapp/templates/configmap.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: my-bankapp-config
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    data:
      MYSQL_HOST: my-bankapp-mysql
      MYSQL_PORT: "3306"
      MYSQL_DATABASE: "bankappdb"
      OLLAMA_URL: "http://my-bankapp-ollama:11434"
      SERVER_FORWARD_HEADERS_STRATEGY: "native"
    ---
    # Source: bankapp/templates/storage.yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: gp3
    provisioner: ebs.csi.aws.com
    parameters:
      type: gp3
      fsType: ext4
    reclaimPolicy: Delete
    volumeBindingMode: WaitForFirstConsumer
    allowVolumeExpansion: true
    ---
    # Source: bankapp/templates/storage.yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-bankapp-mysql-pvc
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      storageClassName: gp3
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
    ---
    # Source: bankapp/templates/services.yaml
    # MySQL Service — always created when mysql is enabled
    apiVersion: v1
    kind: Service
    metadata:
      name: my-bankapp-mysql
      namespace: default
    spec:
      selector:
        app: my-bankapp-mysql
      ports:
        - port: 3306
          targetPort: 3306
    ---
    # Source: bankapp/templates/services.yaml
    # BankApp Service — always created
    apiVersion: v1
    kind: Service
    metadata:
      name: my-bankapp-service
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      type: ClusterIP
      sessionAffinity: ClientIP
      sessionAffinityConfig:
        clientIP:
          timeoutSeconds: 3600
      selector:
        app: my-bankapp
      ports:
        - port: 8080
          targetPort: 8080
    ---
    # Source: bankapp/templates/bankapp-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-bankapp
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      selector:
        matchLabels:
          app: my-bankapp
      template:
        metadata:
          labels:
            app: my-bankapp
        spec:
          initContainers:
            - name: wait-for-mysql
              image: busybox:1.36
              command:
               - /bin/sh
               - -c
               - 'until nc -z my-bankapp-mysql 3306; do sleep 2; done'
              resources:
                requests: { memory: "32Mi", cpu: "50m" }
                limits: { memory: "64Mi", cpu: "100m" }
          containers:
            - name: bankapp
              image: "trainwithshubham/ai-bankapp-eks:latest"
              imagePullPolicy: Always
              ports:
                - containerPort: 8080
              envFrom:
                - configMapRef:
                    name: my-bankapp-config
                - secretRef:
                    name: my-bankapp-secret
              resources:
                limits:
                  cpu: 500m
                  memory: 512Mi
                requests:
                  cpu: 250m
                  memory: 256Mi
              readinessProbe:
                httpGet:
                  path: /actuator/health
                  port: 8080
                initialDelaySeconds: 30
                failureThreshold: 15
              livenessProbe:
                httpGet:
                  path: /actuator/health
                  port: 8080
                initialDelaySeconds: 60
                periodSeconds: 10
                failureThreshold: 5
    ---
    # Source: bankapp/templates/mysql-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-bankapp-mysql
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      selector:
        matchLabels:
          app: my-bankapp-mysql
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: my-bankapp-mysql
        spec:
          containers:
            - name: mysql
              image: "mysql:8.0"
              ports:
                - containerPort: 3306
              env:
                - name: MYSQL_ROOT_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: my-bankapp-secret
                      key: MYSQL_ROOT_PASSWORD
                - name: MYSQL_DATABASE
                  valueFrom:
                    configMapKeyRef:
                      name: my-bankapp-config
                      key: MYSQL_DATABASE
              resources:
                limits:
                  cpu: 500m
                  memory: 512Mi
                requests:
                  cpu: 250m
                  memory: 256Mi
              volumeMounts:
                - name: mysql-storage
                  mountPath: /var/lib/mysql
              readinessProbe:
                exec:
                  command: ["mysqladmin", "ping", "-h", "localhost"]
                initialDelaySeconds: 15
                failureThreshold: 10
              livenessProbe:
                exec:
                  command: ["mysqladmin", "ping", "-h", "localhost"]
                initialDelaySeconds: 30
                periodSeconds: 10
                failureThreshold: 5
          volumes:
            - name: mysql-storage
              persistentVolumeClaim:
                claimName: my-bankapp-mysql-pvc
    ---
    # Source: bankapp/templates/hpa.yaml
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata:
      name: my-bankapp-hpa
      namespace: default
      labels:
        helm.sh/chart: bankapp-0.1.0
        app.kubernetes.io/name: bankapp
        app.kubernetes.io/instance: my-bankapp
        app.kubernetes.io/version: "1.0.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: my-bankapp
      minReplicas: 2
      maxReplicas: 4
      metrics:
        - type: Resource
          resource:
            name: cpu
            target:
              type: Utilization
              averageUtilization: 70
      behavior:
        scaleUp:
          stabilizationWindowSeconds: 30
          policies:
            - type: Pods
              value: 2
              periodSeconds: 60
        scaleDown:
          stabilizationWindowSeconds: 300
          policies:
            - type: Pods
              value: 1
              periodSeconds: 60
    ---

Count the resources in the output. Compare to the full render. You should see:

No Ollama deployment
No Ollama service
No Ollama PVC
No wait-for-ollama init container in bankapp deployment

executed: (One boolean removes an entire component.)

    # Test with different image tag
    helm template my-bankapp bankapp/ \
      --set bankapp.image.tag=abc1234 \
      --set bankapp.replicaCount=2
  
<img width="663" height="833" alt="image" src="https://github.com/user-attachments/assets/ecb78878-cca5-4093-93a7-7ebb4176910f" />

Executed:
    
    helm install my-bankapp bankapp/ \
      --dry-run --debug \
      -n bankapp --create-namespace

--dry-run simulates against the cluster. --debug shows the rendered templates. This catches cluster-level issues before actual deployment.

<img width="1007" height="1071" alt="image" src="https://github.com/user-attachments/assets/6c0e5b77-d1a6-4e14-b111-861396b899d7" />

<img width="564" height="289" alt="image" src="https://github.com/user-attachments/assets/ae603c26-7c8c-4e7b-aea5-c16eae19efaa" />

Kind uses standard storage class, not gp3 (that's AWS EBS). Override storage setting

Executed:

    helm install my-bankapp bankapp/ \
      -n bankapp --create-namespace \
      --set storageClass.create=false \
      --set mysql.persistence.storageClass=standard \
      --set ollama.persistence.storageClass=standard

<img width="1876" height="377" alt="image" src="https://github.com/user-attachments/assets/d5c09f8b-28dd-4ab4-b746-27cb4c0dcdf3" />

checked and verified:

<img width="1178" height="629" alt="image" src="https://github.com/user-attachments/assets/22fc8d08-66f7-431e-8b3d-eabad1d64406" />

All running.

Order of startup:

MySQL pod starts first
BankApp waits for MySQL (init container)
Ollama starts pulling tinyllama model (takes a few minutes)
BankApp also waits for Ollama (second init container)
All pods Running ✅

port-forward:

<img width="837" height="49" alt="image" src="https://github.com/user-attachments/assets/11a2e266-d3dd-4a6d-a976-f657d4e0d037" />


