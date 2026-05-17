# Day 66 -- Provision an EKS Cluster with Terraform Modules

## Task
You built Kubernetes clusters manually in the Kubernetes week. Today you provision one the DevOps way -- fully automated, repeatable, and destroyable with a single command. You will use Terraform registry modules to create an AWS EKS cluster with a managed node group, connect kubectl, and deploy a workload.

This is what infrastructure teams do every day in production.

---

What Are We Building Today?
Your Terminal
     ↓
  Terraform
     ↓
┌─────────────────────────────────────┐
│  AWS VPC (10.0.0.0/16)              │
│  ├── Public Subnet 1 (10.0.1.0/24)  │
│  ├── Public Subnet 2 (10.0.2.0/24)  │
│  ├── Private Subnet 1 (10.0.3.0/24) │← EKS nodes live here
│  ├── Private Subnet 2 (10.0.4.0/24) │← EKS nodes live here
│  ├── NAT Gateway                    │← Private nodes reach internet
│  └── Internet Gateway               │
│                                     │
│  EKS Cluster (terraweek-eks)        │
│  └── Managed Node Group             │
│       ├── Node 1 (t3.medium)        │
│       └── Node 2 (t3.medium)        │
└─────────────────────────────────────┘
     ↓
  kubectl
     ↓
  Nginx Deployment (3 pods)
  LoadBalancer Service → Browser

Why is this impressive?

| Manual way (Day 50) | Terraform way (Today) |
|---|---|
| Click through AWS console for hours | One `terraform apply` command |
| Can't reproduce exactly | Same config = same cluster every time |
| Hard to clean up | One `terraform destroy` = everything gone |
| No version history | Git tracks every infrastructure change |
| Error-prone | Automated = consistent |

EKS is NOT free tier. Here's what this will cost:

| Resource | Cost |
|---|---|
| EKS Cluster | ~$0.10/hour |
| 2x t3.medium nodes | ~$0.084/hour each |
| NAT Gateway | ~$0.045/hour |
| **Total** | **~$0.31/hour** |

You'll spend roughly $0.60-1.00 if you complete this in 2-3 hours. Always destroy when done!

---

## Challenge Tasks

### Task 1: Project Setup
Create a new project directory with proper file structure:

```
terraform-eks/
  providers.tf        # Provider and backend config
  vpc.tf              # VPC module call
  eks.tf              # EKS module call
  variables.tf        # All input variables
  outputs.tf          # Cluster outputs
  terraform.tfvars    # Variable values
```

In `providers.tf`:
1. Pin the AWS provider to `~> 5.0`
2. Pin the Kubernetes provider (you will need it later)
3. Set your region

In `variables.tf`, define:
- `region` (string)
- `cluster_name` (string, default: `"terraweek-eks"`)
- `cluster_version` (string, default: `"1.31"`)
- `node_instance_type` (string, default: `"t3.medium"`)
- `node_desired_count` (number, default: `2`)
- `vpc_cidr` (string, default: `"10.0.0.0/16"`)

---

Wrote all the tf files with file structure:

<img width="576" height="392" alt="image" src="https://github.com/user-attachments/assets/4114b5b6-a212-4a36-9738-a9731df39771" />

terraform init:

<img width="591" height="359" alt="image" src="https://github.com/user-attachments/assets/cd17ff61-bb9d-4bcd-94a4-7b37b79c6001" />

---

### Task 2: Create the VPC with Registry Module
EKS requires a VPC with both public and private subnets across multiple availability zones.

In `vpc.tf`, use the `terraform-aws-modules/vpc/aws` module:
1. CIDR: `var.vpc_cidr`
2. At least 2 availability zones
3. 2 public subnets and 2 private subnets
4. Enable NAT gateway (single NAT to save cost): `enable_nat_gateway = true`, `single_nat_gateway = true`
5. Enable DNS hostnames: `enable_dns_hostnames = true`
6. Add the required EKS tags on subnets:
```hcl
public_subnet_tags = {
  "kubernetes.io/role/elb" = 1
}

private_subnet_tags = {
  "kubernetes.io/role/internal-elb" = 1
}
```

Run `terraform init` and `terraform plan` to verify the VPC config before moving on.

**Document:** Why does EKS need both public and private subnets? What do the subnet tags do?

---

Concept First — Why EKS Needs This Specific VPC Setup:

| Subnet Type | What runs here | Why |
|---|---|---|
| Public subnets | NAT Gateway, Load Balancers | Needs direct internet access |
| Private subnets | EKS worker nodes | Security — nodes not directly exposed |
| NAT Gateway | In public subnet | Lets private nodes reach internet for updates |

The subnet tags are critical for EKS:

kubernetes.io/role/elb = 1 → Tells AWS where to put public LoadBalancers
kubernetes.io/role/internal-elb = 1 → Tells AWS where to put internal LoadBalancers

Without these tags, when you create a Kubernetes Service of type LoadBalancer, AWS won't know which subnet to use!

created VPC.tf:

<img width="1004" height="1416" alt="image" src="https://github.com/user-attachments/assets/e12c75e0-a6ee-4495-9a05-cf5e5e802a06" />

terraform init:

<img width="1158" height="656" alt="image" src="https://github.com/user-attachments/assets/3043bcdf-cf25-4556-955b-4bf59c8b89f8" />

terraform plan:

<img width="590" height="443" alt="image" src="https://github.com/user-attachments/assets/ff6bf4e9-5c5d-4222-97f6-e0eeaca02424" />

You should see the VPC module planning to create ~19+ resources. Review them — VPC, subnets, route tables, NAT Gateway, Elastic IP, Internet Gateway.
Don't apply yet — wait until EKS config is also ready.

---

### Task 3: Create the EKS Cluster with Registry Module
In `eks.tf`, use the `terraform-aws-modules/eks/aws` module:

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = var.cluster_name
  cluster_version = var.cluster_version

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  cluster_endpoint_public_access = true

  eks_managed_node_groups = {
    terraweek_nodes = {
      ami_type       = "AL2_x86_64"
      instance_types = [var.node_instance_type]

      min_size     = 1
      max_size     = 3
      desired_size = var.node_desired_count
    }
  }

  tags = {
    Environment = "dev"
    Project     = "TerraWeek"
    ManagedBy   = "Terraform"
  }
}
```

Run:
```bash
terraform init      # Download EKS module and its dependencies
terraform plan      # Review -- this will create 30+ resources
```

Review the plan carefully before applying. You should see: EKS cluster, IAM roles, node group, security groups, and more.

---

Concept First — What the EKS Module Creates Automatically

Without Terraform, setting up EKS requires:

Creating the cluster IAM role manually
Creating node group IAM role manually
Attaching 5+ IAM policies manually
Configuring security groups manually
Setting up the node group manually

The EKS Terraform module does ALL of this automatically in ~15 lines of config.

done setting up eks.tf and output.tf

terraform init:

<img width="682" height="538" alt="image" src="https://github.com/user-attachments/assets/b9da88cb-133c-4739-a8db-cd6ac245de1e" />

terraform plan:

<img width="1007" height="593" alt="image" src="https://github.com/user-attachments/assets/1aeed84d-18da-4ca8-a2a0-ce17910b4c58" />

Read through this carefully before applying. Once you apply, it takes 10-15 minutes.

---

### Task 4: Apply and Connect kubectl
1. Apply the config:
```bash
terraform apply
```
This will take 10-15 minutes. EKS cluster creation is slow -- be patient.

2. Add outputs in `outputs.tf`:
```hcl
output "cluster_name" {
  value = module.eks.cluster_name
}

output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}

output "cluster_region" {
  value = var.region
}
```

3. Update your kubeconfig:
```bash
aws eks update-kubeconfig --name terraweek-eks --region <your-region>
```

4. Verify:
```bash
kubectl get nodes
kubectl get pods -A
kubectl cluster-info
```

**Verify:** Do you see 2 nodes in `Ready` state? Can you see the kube-system pods running?

---

Important: This takes 10-15 minutes. Don't close your terminal!

applied:

<img width="1076" height="472" alt="image" src="https://github.com/user-attachments/assets/e7aae6d0-3098-4feb-89a9-e5d5214e30cf" />

configured kubectl using output command:

<img width="1067" height="213" alt="image" src="https://github.com/user-attachments/assets/8f442b26-9584-43dc-b85e-3c69ed476e1d" />

<img width="1065" height="89" alt="image" src="https://github.com/user-attachments/assets/998eaa80-7f52-4d8e-8858-029c2b462ab8" />

---

### Task 5: Deploy a Workload on the Cluster
Your Terraform-provisioned cluster is live. Deploy something on it.

1. Create a file `k8s/nginx-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-terraweek
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

2. Apply:
```bash
kubectl apply -f k8s/nginx-deployment.yaml
```

3. Wait for the LoadBalancer to get an external IP:
```bash
kubectl get svc nginx-service -w
```

4. Access the Nginx page via the LoadBalancer URL

5. Verify the full picture:
```bash
kubectl get nodes
kubectl get deployments
kubectl get pods
kubectl get svc
```

**Verify:** Can you access the Nginx welcome page through the LoadBalancer URL?

---

Concept First
This is where Terraform week meets Kubernetes week. You provisioned the cluster with Terraform — now you deploy workloads with kubectl exactly like you learned in Days 52-60.

<img width="1062" height="250" alt="image" src="https://github.com/user-attachments/assets/8b0a5720-6c99-4504-a20c-f3b8fd825a18" />

<img width="1219" height="469" alt="image" src="https://github.com/user-attachments/assets/ffd88b18-f6ce-42f5-969f-806bd08d09ba" />

<img width="1031" height="254" alt="image" src="https://github.com/user-attachments/assets/dd059ca6-e5ec-492b-9fd1-9f14c07d2a2a" />

Output shows all 2 nodes, 1 deployment, 3 pods, 1 LoadBalancer service — all healthy ✅

---

### Task 6: Destroy Everything
This is the most important step. EKS clusters cost money. Clean up completely.

1. First, remove the Kubernetes resources (so the AWS LoadBalancer gets deleted):
```bash
kubectl delete -f k8s/nginx-deployment.yaml
```

2. Wait for the LoadBalancer to be fully removed (check EC2 > Load Balancers in AWS console)

3. Destroy all Terraform resources:
```bash
terraform destroy
```
This will take 10-15 minutes.

4. Verify in the AWS console:
   - EKS clusters: empty
   - EC2 instances: no node group instances
   - VPC: the terraweek VPC should be gone
   - NAT Gateways: deleted
   - Elastic IPs: released

**Verify:** Is your AWS account completely clean? No leftover resources?

---

CRITICAL — Always Clean Up EKS or You'll Get Charged!
The order matters. You MUST delete Kubernetes resources BEFORE running terraform destroy. Here's why:
When you created the LoadBalancer service, Kubernetes asked AWS to create an Elastic Load Balancer. This ELB lives inside your VPC. If you try to delete the VPC before the ELB is gone, terraform destroy will fail and get stuck!

deleted kubernetes resources first:

<img width="1061" height="199" alt="image" src="https://github.com/user-attachments/assets/42e86678-f20b-4cb3-9fc6-abef77462f8d" />


---

## What We Built
A complete AWS EKS cluster provisioned entirely through Terraform modules:
- VPC with public + private subnets across 2 AZs
- NAT Gateway for private subnet internet access
- EKS control plane (managed by AWS)
- Managed node group (2x t3.medium EC2 instances)
- Nginx deployment with LoadBalancer service

## File Structure
terraform-eks/
providers.tf     ← AWS + Kubernetes providers
vpc.tf           ← terraform-aws-modules/vpc/aws
eks.tf           ← terraform-aws-modules/eks/aws (~20.0)
variables.tf     ← All input variables
outputs.tf       ← Cluster name, endpoint, kubectl command
terraform.tfvars ← Variable values
k8s/
nginx-deployment.yaml

## Why Public AND Private Subnets?

| Subnet | What lives here | Why |
|---|---|---|
| Public | NAT Gateway, Load Balancers | Needs direct internet access |
| Private | EKS worker nodes | Security — nodes not exposed |

## Subnet Tags — Why They Matter

| Tag | Purpose |
|---|---|
| `kubernetes.io/role/elb = 1` | Public LoadBalancers use this subnet |
| `kubernetes.io/role/internal-elb = 1` | Internal LBs use this subnet |

Without these tags, Kubernetes LoadBalancer Services can't find the right subnet.

## Resources Created by Terraform
Total: ~56 resources including:
- VPC, 4 subnets, IGW, NAT GW, route tables (~20)
- EKS cluster, IAM roles and policies (~15)
- Managed node group, security groups (~15)
- CloudWatch log group, misc (~6)

## Manual vs Terraform EKS

| | Manual (Day 50) | Terraform |
|---|---|---|
| Setup time | Hours | 15 minutes |
| Reproducible | No | Yes |
| Version controlled | No | Yes |
| Destroy time | Manual, error-prone | One command |
| IAM setup | Manual | Automatic |

## Destroy Order — Critical!
1. `kubectl delete -f k8s/nginx-deployment.yaml` ← Delete K8s resources FIRST
2. Wait for LoadBalancer ELB to be removed from AWS
3. `terraform destroy` ← Then destroy all infra

Skipping step 1 causes terraform destroy to fail — ELB blocks VPC deletion.

## Cost
- EKS Cluster: ~$0.10/hour
- 2x t3.medium: ~$0.17/hour
- NAT Gateway: ~$0.045/hour
- Total: ~$0.31/hour — ALWAYS destroy when done!

## Summary Table:

| Concept | What it means |
|---|---|
| EKS | AWS managed Kubernetes — AWS runs the control plane, you manage nodes |
| Managed node group | AWS manages EC2 worker nodes — auto-healing, auto-scaling |
| Private subnets for nodes | Security best practice — nodes not directly internet-accessible |
| NAT Gateway | Lets private subnet nodes reach internet (for image pulls, updates) |
| `single_nat_gateway = true` | One NAT instead of one per AZ — saves ~$0.09/hour in learning |
| Subnet tags | Tell AWS Load Balancer controller which subnets to use for LBs |
| `enable_cluster_creator_admin_permissions` | Gives your IAM user kubectl admin access automatically |
| `cluster_endpoint_public_access = true` | Allows kubectl from your laptop — false in production |
| `aws eks update-kubeconfig` | Adds cluster credentials to your ~/.kube/config |
| Delete K8s resources before destroy | LoadBalancer Service creates AWS ELB which blocks VPC deletion |
| Total resources | ~56 resources from 2 module calls — shows power of modules |
