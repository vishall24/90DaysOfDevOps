# Day 81 -- Introduction to Amazon EKS with Terraform

## Task
You have been running Kubernetes locally with Kind. That works for learning, but the AI-BankApp needs a production environment -- managed control plane, auto-scaling nodes, persistent EBS storage, and IAM integration.

Amazon EKS (Elastic Kubernetes Service) is AWS's managed Kubernetes offering. The AI-BankApp project (https://github.com/TrainWithShubham/AI-BankApp-DevOps, branch `feat/gitops`) already has a complete Terraform configuration in its `terraform/` directory that provisions a production-grade EKS cluster. Today you understand EKS architecture, study the Terraform configs, provision the cluster, and connect to it.

---

What Are We Learning Today?:

You've been running Kubernetes on Kind (local containers). Today you move to real AWS EKS — the same setup production teams use.

| | Kind (what you've been using) | EKS (today) |
|---|---|---|
| Control plane | Runs in Docker container | Managed by AWS |
| Worker nodes | Docker containers | Real EC2 instances |
| Storage | hostPath / local | AWS EBS volumes |
| Cost | Free | ~$7/day |
| Use case | Learning | Production |


Cost Warning:

EKS costs money:

EKS control plane: $0.10/hr
3x t3.medium nodes: ~$0.13/hr total
NAT Gateway: ~$0.045/hr
Total: ~$7/day

Only keep it running while actively working. Destroy when done!

---

## Challenge Tasks

### Task 1: Understand EKS Architecture
Research and write notes on:

1. **What does "managed Kubernetes" mean?**
   - AWS manages the **control plane** (API server, etcd, scheduler, controller manager)
   - You manage the **data plane** (worker nodes where your pods run)
   - AWS handles control plane upgrades, patching, and high availability across multiple AZs

2. **EKS components:**
   - **EKS Control Plane** -- managed by AWS, runs in AWS-owned VPC, accessible via API endpoint
   - **Node Groups** -- EC2 instances that run your pods
     - **Managed Node Groups** -- AWS handles provisioning, scaling, and updates
     - **Self-Managed Nodes** -- you manage the EC2 instances yourself
     - **Fargate Profiles** -- serverless, no nodes to manage at all
   - **VPC and Networking** -- EKS runs inside your VPC with subnets across AZs
   - **IAM Integration** -- EKS uses IAM roles for cluster access and pod-level permissions (IRSA)

3. **EKS add-ons the AI-BankApp uses** (from `terraform/eks.tf`):
   - `coredns` -- DNS resolution inside the cluster
   - `kube-proxy` -- network routing for services
   - `vpc-cni` -- AWS VPC CNI plugin, assigns VPC IPs to pods
   - `eks-pod-identity-agent` -- enables pod-level IAM roles
   - `aws-ebs-csi-driver` -- allows pods to use EBS volumes (needed for MySQL and Ollama storage)
   - `metrics-server` -- enables `kubectl top` and HPA

---

What is Managed Kubernetes? :

AWS manages the brain (control plane) — the API server, etcd database, scheduler. You manage the muscles (worker nodes) — the EC2 instances 
where your pods actually run. If the control plane crashes, AWS fixes it. You never SSH into it.


EKS Components:

    ┌─────────────────────────────────────────────────────┐
    │  Your AWS Account                                   │
    │                                                     │
    │  VPC (10.0.0.0/16)                                  │
    │  ├── Public Subnets (3 AZs) ← Load Balancers        │
    │  ├── Private Subnets (3 AZs) ← Worker Nodes         │
    │  └── Intra Subnets (3 AZs) ← Control Plane ENIs     │
    │                                                     │
    │  EKS Control Plane (AWS managed)                    │
    │  ├── API Server                                     │
    │  ├── etcd                                           │
    │  ├── Scheduler                                      │
    │  └── Controller Manager                             │
    │                                                     │
    │  Managed Node Group                                 │
    │  ├── Node 1 (t3.medium, AZ-a)                       │
    │  ├── Node 2 (t3.medium, AZ-b)                       │
    │  └── Node 3 (t3.medium, AZ-c)                       │
    │                                                     │
    │  EKS Add-ons                                        │
    │  ├── coredns (DNS)                                  │
    │  ├── kube-proxy (networking)                        │
    │  ├── vpc-cni (pod IPs from VPC)                     │
    │  ├── eks-pod-identity-agent (IAM for pods)          │
    │  ├── aws-ebs-csi-driver (EBS storage)               │
    │  └── metrics-server (HPA + kubectl top)             │
    └─────────────────────────────────────────────────────┘


---

### Task 2: Study the AI-BankApp Terraform Configuration
Clone the repo and examine the `terraform/` directory:

```bash
git clone -b feat/gitops https://github.com/TrainWithShubham/AI-BankApp-DevOps.git
cd AI-BankApp-DevOps/terraform
ls
```

```
argocd.tf           # ArgoCD Helm release
eks.tf              # EKS cluster + node group + IRSA
outputs.tf          # Cluster info and helper commands
provider.tf         # AWS + Helm providers, locals
terraform.tfvars    # Default variable values
variables.tf        # Input variables
vpc.tf              # VPC with public/private/intra subnets
```

**Study each file and understand what it provisions:**

**`variables.tf` and `terraform.tfvars`:**
```hcl
# The defaults:
aws_region         = "us-west-2"
cluster_name       = "bankapp-eks"
cluster_version    = "1.35"
node_instance_type = "t3.medium"
node_desired_count = 3
node_max_count     = 5
```

**`vpc.tf`** -- Networking foundation:
- Uses the `terraform-aws-modules/vpc/aws` module
- 3 Availability Zones with:
  - **Public subnets** (10.0.1-3.0/24) -- for load balancers, tagged with `kubernetes.io/role/elb`
  - **Private subnets** (10.0.4-6.0/24) -- for worker nodes, tagged with `kubernetes.io/role/internal-elb`
  - **Intra subnets** (10.0.7-9.0/24) -- for EKS control plane ENIs
- NAT Gateway enabled for outbound internet from private subnets

**`eks.tf`** -- The cluster itself:
- Uses the `terraform-aws-modules/eks/aws` module (version ~> 21.0)
- AL2023 AMI for nodes (Amazon Linux 2023)
- 3x `t3.medium` instances (min 3, max 5)
- All 6 EKS add-ons installed as cluster add-ons
- IRSA configured for the EBS CSI driver
- Public + private API endpoint access

**`argocd.tf`** -- ArgoCD via Helm:
- Installs ArgoCD using the `argo-cd` Helm chart
- Exposed as a LoadBalancer service
- Depends on the EKS module (created after the cluster is ready)

**`outputs.tf`** -- Helper commands:
- Outputs the `aws eks update-kubeconfig` command
- Outputs the ArgoCD initial password retrieval command

**Document:** Draw the architecture: VPC -> Subnets -> EKS Control Plane -> Node Group -> Pods

---

Reviewed all the tf files present.

Architecture diagram:

    ┌─────────────────────────────────────────────────────┐
    │  Your AWS Account                                   │
    │                                                     │
    │  VPC (10.0.0.0/16)                                  │
    │  ├── Public Subnets (3 AZs) ← Load Balancers        │
    │  ├── Private Subnets (3 AZs) ← Worker Nodes         │
    │  └── Intra Subnets (3 AZs) ← Control Plane ENIs     │
    │                                                     │
    │  EKS Control Plane (AWS managed)                    │
    │  ├── API Server                                     │
    │  ├── etcd                                           │
    │  ├── Scheduler                                      │
    │  └── Controller Manager                             │
    │                                                     │
    │  Managed Node Group                                 │
    │  ├── Node 1 (t3.medium, AZ-a)                       │
    │  ├── Node 2 (t3.medium, AZ-b)                       │
    │  └── Node 3 (t3.medium, AZ-c)                       │
    │                                                     │
    │  EKS Add-ons                                        │
    │  ├── coredns (DNS)                                  │
    │  ├── kube-proxy (networking)                        │
    │  ├── vpc-cni (pod IPs from VPC)                     │
    │  ├── eks-pod-identity-agent (IAM for pods)          │
    │  ├── aws-ebs-csi-driver (EBS storage)               │
    │  └── metrics-server (HPA + kubectl top)             │
    └─────────────────────────────────────────────────────┘

---

### Task 3: Provision the EKS Cluster
Make sure you have the required tools:
```bash
terraform --version    # >= 1.0
aws --version          # AWS CLI v2
kubectl version --client
helm version
```

Configure AWS credentials:
```bash
aws configure
# Enter: Access Key ID, Secret Access Key, Region (us-west-2), Output (json)

# Verify
aws sts get-caller-identity
```

Initialize and apply:
```bash
cd terraform

terraform init
terraform plan
```

Review the plan carefully. It will create:
- 1 VPC with 9 subnets, NAT gateway, internet gateway
- 1 EKS cluster with control plane
- 1 managed node group (3x t3.medium)
- 6 EKS add-ons
- IAM roles and policies for the cluster, nodes, and EBS CSI driver
- ArgoCD Helm release

```bash
terraform apply
```

This takes 15-20 minutes. While waiting, review the Terraform output for CloudFormation-like progress.

After completion, note the outputs:
```bash
terraform output
```

---

<img width="1115" height="400" alt="image" src="https://github.com/user-attachments/assets/a2f07200-93a5-45cb-bebf-07c0377f0b61" />

aws configure :

<img width="671" height="114" alt="image" src="https://github.com/user-attachments/assets/3f5aa587-be6e-4302-a7cb-281061d8a1f3" />

terraform init:

<img width="712" height="524" alt="image" src="https://github.com/user-attachments/assets/8169e9f2-74de-43ee-9bfc-8560b1f0ef2f" />

terraform plan:

<img width="1080" height="736" alt="image" src="https://github.com/user-attachments/assets/768c8cf0-c0a0-4e30-b8b9-092af3f54a0c" />

== 

VPC resources (~20)
EKS cluster and IAM roles (~15)
Node group and launch template (~10)
Add-on resources (~10)
ArgoCD Helm release

terraform output:

<img width="938" height="292" alt="image" src="https://github.com/user-attachments/assets/ee9bfbb8-0131-4060-af37-5eb1956c3b32" />

Note the configure_kubectl output — we will need this.

---

### Task 4: Connect to Your Cluster
Update kubeconfig using the Terraform output:
```bash
aws eks update-kubeconfig --name bankapp-eks --region us-west-2
```

Verify the connection:
```bash
# Check context
kubectl config current-context

# Cluster info
kubectl cluster-info

# List nodes
kubectl get nodes -o wide
```

You should see 3 nodes with status `Ready`, instance type `t3.medium`, spread across 3 AZs.

Explore the cluster:
```bash
# System pods
kubectl get pods -n kube-system

# All the add-ons are running
kubectl get daemonsets -n kube-system

# EBS CSI driver
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-ebs-csi-driver

# Metrics server (enables kubectl top and HPA)
kubectl top nodes
```

Check ArgoCD is running:
```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
```

Get the ArgoCD admin password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Get the ArgoCD LoadBalancer URL:
```bash
kubectl get svc -n argocd argocd-server -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Open the URL in your browser and log in with `admin` and the password from above. You will use ArgoCD on Days 84-86.

---

configure kubectl and get the context of your cluster:

<img width="1437" height="259" alt="image" src="https://github.com/user-attachments/assets/68e69aae-cbbd-4bf1-86a9-bc61457966ae" />

verify csi & argocd application:

<img width="1275" height="729" alt="image" src="https://github.com/user-attachments/assets/f20ee45e-d063-4da4-8525-64974c2ba0e3" />

accessing ARGO CD application through LB URL:

<img width="1124" height="65" alt="image" src="https://github.com/user-attachments/assets/ef258e49-2ba0-494f-907b-79a6a5fccb6d" />

<img width="1423" height="768" alt="image" src="https://github.com/user-attachments/assets/c25d805c-a75a-488d-b50b-9f1c170910fc" />

<img width="1440" height="675" alt="image" src="https://github.com/user-attachments/assets/3cd32260-aca1-404d-b05c-0888578dd9d0" />

---

### Task 5: Deploy the AI-BankApp Manually (Before ArgoCD)
Before setting up GitOps, deploy the app manually to validate the cluster works.

Apply the raw manifests from the `k8s/` directory:
```bash
cd ../  # Back to the repo root

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

Watch the pods come up:
```bash
kubectl get pods -n bankapp -w
```

The startup order is:
1. MySQL starts and becomes healthy (15-30 seconds)
2. Ollama starts and pulls the TinyLlama model (2-5 minutes)
3. BankApp init containers wait for both, then the app starts (30-60 seconds after dependencies)

Check PVCs are bound to EBS volumes:
```bash
kubectl get pvc -n bankapp
kubectl get pv
```

You should see 5Gi and 10Gi EBS volumes in the correct AZs.

Once all pods are running, access the app:
```bash
kubectl port-forward svc/bankapp-service -n bankapp 8080:8080
```

Open `http://localhost:8080` -- you should see the AI-BankApp login page. Register an account, log in, and try the AI chatbot.

**Verify the HPA:**
```bash
kubectl get hpa -n bankapp
```

---

Why deploy manually first?:

Before setting up GitOps (ArgoCD) on Day 84, you validate the cluster works with the raw manifests. If something fails here, you fix it now, not on GitOps day.

applying all manifest manually:

<img width="955" height="268" alt="image" src="https://github.com/user-attachments/assets/d0885770-f6aa-4ae3-8391-69fafdcc3aeb" />

<img width="549" height="82" alt="image" src="https://github.com/user-attachments/assets/e859c011-46db-4012-872f-94a3e5c39ca2" />

all pods are running: ( had to debug to increase the instance type since the ollama pod requires much more RAM)

<img width="545" height="107" alt="image" src="https://github.com/user-attachments/assets/56dee83a-4624-40d3-b19b-ee24b8066b6d" />

PVs:

<img width="1226" height="132" alt="image" src="https://github.com/user-attachments/assets/04543f96-7fcf-4920-829d-0c7dbeaf2c33" />

<img width="1079" height="244" alt="image" src="https://github.com/user-attachments/assets/598d8e48-e761-4142-a012-669ce55f407c" />

accessing application :

<img width="876" height="63" alt="image" src="https://github.com/user-attachments/assets/843abe33-923c-4311-a2bf-f898454daa9f" />

<img width="1204" height="707" alt="image" src="https://github.com/user-attachments/assets/6ce33ce6-533f-45df-b4a1-74bfb6b895c5" />

<img width="977" height="73" alt="image" src="https://github.com/user-attachments/assets/b926ef84-3970-42c9-805d-aeb784994f41" />

---

### Task 6: Understand EKS Costs and Clean Up Strategy
EKS is not free. The AI-BankApp cluster costs:

| Component | Cost (approximate) |
|-----------|-------------------|
| EKS Control Plane | $0.10/hr (~$73/month) |
| t3.medium nodes (3x) | ~$0.042/hr each (~$91/month total) |
| NAT Gateway | ~$0.045/hr + data transfer (~$33/month) |
| EBS volumes (15Gi total) | ~$1.50/month |
| LoadBalancer (ArgoCD) | ~$0.025/hr (~$18/month) |
| **Total for this lab** | **~$220/month (~$7/day)** |

**Important:** Do NOT leave the cluster running when you are not using it.

Delete the BankApp workload (keep the cluster for Days 82-83):
```bash
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

To destroy everything (do this at the end of Day 83 or if taking a break):
```bash
cd terraform
terraform destroy
```

**Document:** What are the cost components of the AI-BankApp EKS setup? Why is the NAT Gateway surprisingly expensive?

---

| Component | Cost/hr | Cost/month |
|---|---|---|
| EKS Control Plane | $0.10 | ~$73 |
| 3x t3.medium nodes | $0.126 | ~$91 |
| NAT Gateway | $0.045 | ~$33 |
| EBS volumes (15Gi) | negligible | ~$1.50 |
| ArgoCD LoadBalancer | $0.025 | ~$18 |
| **Total** | **~$0.30/hr** | **~$220/month** |

### Why is NAT Gateway expensive?

Every private subnet (where your nodes live) needs to reach the internet to pull Docker images, call AWS APIs, etc. The NAT Gateway is the 
exit point for all that traffic — it charges per hour AND per GB of data processed. Even idle clusters generate NAT traffic.

<img width="906" height="511" alt="image" src="https://github.com/user-attachments/assets/62ed7acf-4822-4fc3-98e0-c3c5d1e7583d" />


---


## Introduction to Amazon EKS with Terraform

## What is EKS?
Amazon EKS (Elastic Kubernetes Service) is managed Kubernetes.
AWS manages the control plane (API server, etcd, scheduler).
You manage the data plane (EC2 worker nodes where pods run).

## EKS vs Kind

| | Kind | EKS |
|---|---|---|
| Control plane | Docker container | AWS managed |
| Worker nodes | Docker containers | Real EC2 instances |
| Storage | hostPath | AWS EBS (gp3) |
| Cost | Free | ~$7/day |
| Use case | Learning | Production |

## Architecture
VPC (10.0.0.0/16)
├── Public Subnets (3 AZs) ← Load Balancers
├── Private Subnets (3 AZs) ← Worker Nodes
└── Intra Subnets (3 AZs) ← Control Plane ENIs

EKS Control Plane (AWS managed)
└── Managed Node Group: 3x t3.medium

EKS Add-ons:
- coredns: DNS inside cluster
- kube-proxy: service networking
- vpc-cni: pod IPs from VPC CIDR
- eks-pod-identity-agent: IAM for pods
- aws-ebs-csi-driver: EBS storage for MySQL + Ollama
- metrics-server: HPA + kubectl top

## Terraform Files

| File | What it does |
|---|---|
| variables.tf | Input variable definitions |
| terraform.tfvars | Default values (region, instance type, counts) |
| vpc.tf | VPC with 9 subnets across 3 AZs |
| eks.tf | EKS cluster, node group, add-ons, IRSA |
| argocd.tf | ArgoCD via Helm (used on Day 84) |
| outputs.tf | Helper commands after provisioning |

## EKS Cost Breakdown

| Component | Cost/hr | Cost/month |
|---|---|---|
| EKS Control Plane | $0.10 | ~$73 |
| 3x t3.medium nodes | $0.126 | ~$91 |
| NAT Gateway | $0.045 | ~$33 |
| EBS volumes | ~$0.002 | ~$1.50 |
| ArgoCD LoadBalancer | $0.025 | ~$18 |
| **Total** | **~$0.30/hr** | **~$220/month** |

NAT Gateway is expensive because it charges per hour AND per GB processed.
Every container image pull, AWS API call from private nodes goes through it.

## Key Commands
```bash
# Provision cluster
terraform init && terraform apply

# Connect kubectl
aws eks update-kubeconfig --name bankapp-eks --region us-west-2

# Verify
kubectl get nodes -o wide
kubectl get pods -n kube-system

# Destroy (always when done!)
terraform destroy
```

## Summary Table:

| Concept | What it means |
|---|---|
| EKS | AWS managed Kubernetes — AWS runs control plane, you run nodes |
| Managed Node Group | AWS manages EC2 workers — auto-healing, auto-scaling |
| VPC CNI | Gives each pod its own real VPC IP address |
| EBS CSI Driver | Allows pods to use AWS EBS volumes for persistent storage |
| IRSA | IAM Roles for Service Accounts — pod-level AWS permissions |
| Intra subnets | Special subnets only for EKS control plane network interfaces |
| Private subnets | Where worker nodes live — no direct internet, use NAT Gateway |
| Public subnets | Where Load Balancers go — directly internet accessible |
| NAT Gateway | Allows private subnet nodes to reach internet — charges per GB |
| metrics-server | Enables kubectl top nodes/pods and HPA — add-on on EKS |
| `terraform apply` | Creates all 65+ resources in ~15-20 minutes |
| `terraform destroy` | ALWAYS run when done — cluster costs ~$7/day |
| ArgoCD | Pre-installed by Terraform — you'll use it on Day 84 |

