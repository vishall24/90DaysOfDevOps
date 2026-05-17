# Day 67 -- TerraWeek Capstone: Multi-Environment Infrastructure with Workspaces and Modules

## Task
Seven days of Terraform -- HCL, providers, resources, dependencies, variables, outputs, data sources, state management, remote backends, custom modules, registry modules, and a full EKS cluster. Today you put it all together in one production-grade project.

Build a multi-environment AWS infrastructure using custom modules and Terraform workspaces. One codebase, three environments -- dev, staging, and prod. This is how infrastructure teams operate at scale.

---

What Are We Building Today?

Right now, most teams have 3 environments:

Dev — where developers test features, small instances, SSH open

Staging — where QA tests before production, medium instances

Prod — live user traffic, larger instances, SSH closed for security

The old way: 3 separate folders, 3 copies of the same Terraform code, drift everywhere.

Today's way — Workspaces + Modules:

    One codebase
         ↓
    terraform workspace select dev     → deploys to dev VPC (10.0.0.0/16), t2.micro
    terraform workspace select staging → deploys to staging VPC (10.1.0.0/16), t2.small
    terraform workspace select prod    → deploys to prod VPC (10.2.0.0/16), t3.small
    
    Same code. Different state. Different config. Completely isolated.

---

## Challenge Tasks

### Task 1: Learn Terraform Workspaces
Before building the project, understand workspaces:

```bash
mkdir terraweek-capstone && cd terraweek-capstone
terraform init

# See current workspace
terraform workspace show                    # default

# Create new workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# List all workspaces
terraform workspace list

# Switch between them
terraform workspace select dev
terraform workspace select staging
terraform workspace select prod
```

Answer:
1. What does `terraform.workspace` return inside a config?
2. Where does each workspace store its state file?
3. How is this different from using separate directories per environment?

---

 Concept First:

| | Without Workspaces | With Workspaces |
|---|---|---|
| How to separate envs | 3 separate directories | 1 directory, 3 workspaces |
| State files | 3 separate state files | 1 per workspace, auto-managed |
| Code duplication | High | Zero |
| Risk of mistakes | Copy-paste errors | One codebase |

Each workspace gets its own state file stored at:

    terraform.tfstate.d/
      dev/terraform.tfstate
      staging/terraform.tfstate
      prod/terraform.tfstate


created dir : terraweek-capstone and did terraform init:

Create all three workspaces

    terraform workspace new dev
    terraform workspace new staging
    terraform workspace new prod

Each creation automatically switches you to that workspace.

<img width="593" height="301" alt="image" src="https://github.com/user-attachments/assets/9864ddb1-d095-4d3d-b94c-f22e01fbd205" />

<img width="587" height="178" alt="image" src="https://github.com/user-attachments/assets/b04a298d-0022-4871-a492-b885c69e7057" />

Answers for notes:

terraform.workspace returns the current workspace name — "dev", "staging", or "prod"
Each workspace state: terraform.tfstate.d/<workspace>/terraform.tfstate
vs separate directories: workspaces use one codebase with shared modules, directories duplicate everything

---

### Task 2: Set Up the Project Structure
Create this layout:

```
terraweek-capstone/
  main.tf                   # Root module -- calls child modules
  variables.tf              # Root variables
  outputs.tf                # Root outputs
  providers.tf              # AWS provider and backend
  locals.tf                 # Local values using workspace
  dev.tfvars                # Dev environment values
  staging.tfvars            # Staging environment values
  prod.tfvars               # Prod environment values
  .gitignore                # Ignore state, .terraform, tfvars with secrets
  modules/
    vpc/
      main.tf
      variables.tf
      outputs.tf
    security-group/
      main.tf
      variables.tf
      outputs.tf
    ec2-instance/
      main.tf
      variables.tf
      outputs.tf
```

Create the `.gitignore`:
```
.terraform/
*.tfstate
*.tfstate.backup
*.tfvars
.terraform.lock.hcl
```

**Document:** Why is this file structure considered best practice?

---

Create all directories and files:

# Module directories
mkdir -p modules/vpc
mkdir -p modules/security-group
mkdir -p modules/ec2-instance

# Create all root files
touch main.tf variables.tf outputs.tf providers.tf locals.tf
touch dev.tfvars staging.tfvars prod.tfvars

# Create module files
touch modules/vpc/main.tf modules/vpc/variables.tf modules/vpc/outputs.tf
touch modules/security-group/main.tf modules/security-group/variables.tf modules/security-group/outputs.tf
touch modules/ec2-instance/main.tf modules/ec2-instance/variables.tf modules/ec2-instance/outputs.tf


<img width="864" height="276" alt="image" src="https://github.com/user-attachments/assets/43607136-36e5-493d-af40-fdbd7d7ac22c" />

.gitignore:

<img width="340" height="209" alt="image" src="https://github.com/user-attachments/assets/e3eb6e80-deef-43d6-a9b4-339121516dc6" />

Why this structure is best practice:

Each file has one clear responsibility — no 500-line main.tf
Modules are reusable across projects
.gitignore protects secrets and state from being committed
tfvars per environment = no manual editing when switching envs


---

### Task 3: Build the Custom Modules
Create three focused modules:

**Module 1: `modules/vpc/`**
- Input: `cidr`, `public_subnet_cidr`, `environment`, `project_name`
- Resources: VPC, public subnet, internet gateway, route table, route table association
- Output: `vpc_id`, `subnet_id`
- All resources tagged with environment and project name

**Module 2: `modules/security-group/`**
- Input: `vpc_id`, `ingress_ports`, `environment`, `project_name`
- Resources: Security group with dynamic ingress rules, allow all egress
- Output: `sg_id`

**Module 3: `modules/ec2-instance/`**
- Input: `ami_id`, `instance_type`, `subnet_id`, `security_group_ids`, `environment`, `project_name`
- Resources: EC2 instance with tags
- Output: `instance_id`, `public_ip`

Write and validate each module:
```bash
terraform validate
```

---

<img width="539" height="385" alt="image" src="https://github.com/user-attachments/assets/bb58cb1b-5b36-4ec4-8987-00bf0492a47f" />

<img width="349" height="604" alt="image" src="https://github.com/user-attachments/assets/4ae8eff0-20c5-427e-9446-4eeff3c8a6f9" />

<img width="498" height="198" alt="image" src="https://github.com/user-attachments/assets/94b346aa-73fa-4a67-b32d-d443211edc05" />

<img width="583" height="410" alt="image" src="https://github.com/user-attachments/assets/02af5d96-44eb-4390-8eb3-147d4c9f4493" />

<img width="651" height="590" alt="image" src="https://github.com/user-attachments/assets/c14b61da-8ed4-4039-b2d6-6b7f0aedd8bd" />

<img width="543" height="105" alt="image" src="https://github.com/user-attachments/assets/8f8fb320-b8ad-411b-aaab-36d6d5c02333" />

<img width="593" height="570" alt="image" src="https://github.com/user-attachments/assets/9b2f3103-303b-4e3f-a258-2aa5e1496cd1" />

<img width="623" height="275" alt="image" src="https://github.com/user-attachments/assets/4d555617-0a0a-43f8-a604-b8efa0d44668" />

<img width="564" height="208" alt="image" src="https://github.com/user-attachments/assets/f5fde225-0e2f-48dc-81ae-c5f946c7bfed" />


---

### Task 4: Wire It All Together with Workspace-Aware Config
In the root module, use `terraform.workspace` to drive environment-specific behavior.

**`locals.tf`:**
```hcl
locals {
  environment = terraform.workspace
  name_prefix = "${var.project_name}-${local.environment}"

  common_tags = {
    Project     = var.project_name
    Environment = local.environment
    ManagedBy   = "Terraform"
    Workspace   = terraform.workspace
  }
}
```

**`variables.tf`:**
```hcl
variable "project_name" {
  type    = string
  default = "terraweek"
}

variable "vpc_cidr" {
  type = string
}

variable "subnet_cidr" {
  type = string
}

variable "instance_type" {
  type = string
}

variable "ingress_ports" {
  type    = list(number)
  default = [22, 80]
}
```

**`main.tf`** -- call all three modules, passing workspace-aware names and variables.

**Environment-specific tfvars:**

`dev.tfvars`:
```hcl
vpc_cidr      = "10.0.0.0/16"
subnet_cidr   = "10.0.1.0/24"
instance_type = "t2.micro"
ingress_ports = [22, 80]
```

`staging.tfvars`:
```hcl
vpc_cidr      = "10.1.0.0/16"
subnet_cidr   = "10.1.1.0/24"
instance_type = "t2.small"
ingress_ports = [22, 80, 443]
```

`prod.tfvars`:
```hcl
vpc_cidr      = "10.2.0.0/16"
subnet_cidr   = "10.2.1.0/24"
instance_type = "t3.small"
ingress_ports = [80, 443]
```

Notice: dev allows SSH, prod does not. Different CIDRs prevent overlap. Instance types scale up per environment.

---

<img width="435" height="242" alt="image" src="https://github.com/user-attachments/assets/85f34966-ce82-4f7a-88d3-ef5c8e724e7e" />

<img width="508" height="320" alt="image" src="https://github.com/user-attachments/assets/8eaf1809-e0bf-448a-852b-4c290d078810" />

<img width="555" height="496" alt="image" src="https://github.com/user-attachments/assets/07ff73f9-cd24-4116-838f-9f5848ad7e24" />

<img width="458" height="677" alt="image" src="https://github.com/user-attachments/assets/2eb6b3e2-cf92-4fb5-9121-101f6fcd89ce" />

<img width="501" height="560" alt="image" src="https://github.com/user-attachments/assets/bf2e3ea6-83af-4caa-acba-9e40fa38b459" />

<img width="434" height="138" alt="image" src="https://github.com/user-attachments/assets/980a6edb-5f4a-4e6f-a846-502769f7401d" />

<img width="587" height="159" alt="image" src="https://github.com/user-attachments/assets/847327b0-8872-4c1e-ae6d-b8e7fff7b824" />

<img width="525" height="142" alt="image" src="https://github.com/user-attachments/assets/b12c670c-5c6d-423b-acaa-dfa5489b148b" />

Notice the key differences:

| | Dev | Staging | Prod |
|---|---|---|---|
| VPC CIDR | 10.0.0.0/16 | 10.1.0.0/16 | 10.2.0.0/16 |
| SSH (port 22) | ✅ Open | ✅ Open | ❌ Closed |
| HTTPS (443) | ❌ | ✅ Open | ✅ Open |
| Instance type | t3.micro | t3.micro | t3.micro |

<img width="658" height="380" alt="image" src="https://github.com/user-attachments/assets/ff213d79-593b-4108-8f75-19fd9b6850be" />

---

### Task 5: Deploy All Three Environments
Deploy each environment using its workspace and tfvars file:

**Dev:**
```bash
terraform workspace select dev
terraform plan -var-file="dev.tfvars"
terraform apply -var-file="dev.tfvars"
```

**Staging:**
```bash
terraform workspace select staging
terraform plan -var-file="staging.tfvars"
terraform apply -var-file="staging.tfvars"
```

**Prod:**
```bash
terraform workspace select prod
terraform plan -var-file="prod.tfvars"
terraform apply -var-file="prod.tfvars"
```

After all three are deployed, verify:
```bash
# Check each workspace's resources
terraform workspace select dev && terraform output
terraform workspace select staging && terraform output
terraform workspace select prod && terraform output
```

Go to the AWS console and verify:
- Three separate VPCs with different CIDR ranges
- Three EC2 instances with different instance types
- Different Name tags per environment: `terraweek-dev-server`, `terraweek-staging-server`, `terraweek-prod-server`

**Verify:** Are all three environments completely isolated from each other?

---

Cost Warning:
Each environment creates a VPC + EC2 instance. 3 environments = 3 EC2 instances running simultaneously. Destroy promptly!

<img width="994" height="568" alt="image" src="https://github.com/user-attachments/assets/094ed1dc-a122-4e99-9e0e-70934b19f87d" />

<img width="734" height="575" alt="image" src="https://github.com/user-attachments/assets/2bb3e610-f577-4cd3-9acf-7d83344a8a89" />

<img width="773" height="566" alt="image" src="https://github.com/user-attachments/assets/1ef211c4-8ebe-4c5e-8077-bc3e697c858f" />

<img width="1080" height="497" alt="image" src="https://github.com/user-attachments/assets/8bd060d4-6bb3-47d3-b4d9-efec6fc53ee0" />

<img width="757" height="568" alt="image" src="https://github.com/user-attachments/assets/cc029df7-e831-4df4-aa0f-f28985f797db" />

<img width="975" height="528" alt="image" src="https://github.com/user-attachments/assets/f0072b8b-5216-4679-a922-0e680ec1493a" />

Here in the prod its showing as 1 added since I have already executed terraform apply for prod and it failed since t2.micro is not eligible 
for free tier and changed it to t3.small:

<img width="662" height="565" alt="image" src="https://github.com/user-attachments/assets/dc623e6a-a883-4b26-8548-cd71c81a1a55" />

<img width="614" height="384" alt="image" src="https://github.com/user-attachments/assets/4cbd7fb8-d985-4b1a-99aa-0f30f5a9778a" />

Verified:

<img width="1152" height="362" alt="image" src="https://github.com/user-attachments/assets/b62a58f4-3002-4f52-8085-6c73fe73ea08" />

<img width="1046" height="400" alt="image" src="https://github.com/user-attachments/assets/b508ac96-7376-4740-bddb-02a047283bd0" />

<img width="1123" height="463" alt="image" src="https://github.com/user-attachments/assets/d39ef6fd-bda7-47b9-92af-3e472f83b8b6" />

<img width="583" height="49" alt="image" src="https://github.com/user-attachments/assets/def2da91-c482-4dd6-8fb3-c2c7fe4786bb" />

Each folder contains a separate terraform.tfstate file — completely isolated!

All 3 environments isolated, different CIDRs, different instance types, different security rules.

---


### Task 6: Document Best Practices
Write down everything you have learned this week as a Terraform best practices guide:

1. **File structure** -- separate files for providers, variables, outputs, main, locals
2. **State management** -- always use remote backend, enable locking, enable versioning
3. **Variables** -- never hardcode, use tfvars per environment, validate with `validation` blocks
4. **Modules** -- one concern per module, always define inputs/outputs, pin registry module versions
5. **Workspaces** -- use for environment isolation, reference `terraform.workspace` in configs
6. **Security** -- .gitignore for state and tfvars, encrypt state at rest, restrict backend access
7. **Commands** -- always run `plan` before `apply`, use `fmt` and `validate` before committing
8. **Tagging** -- tag every resource with project, environment, and managed-by
9. **Naming** -- consistent prefix pattern: `<project>-<environment>-<resource>`
10. **Cleanup** -- always `terraform destroy` non-production environments when not in use

---

Here are the 10 best practices written out:

## Terraform Best Practices Guide

1. **File structure** — separate providers.tf, variables.tf, outputs.tf, main.tf, locals.tf.
   Never put everything in one file.

2. **State management** — always remote backend (S3 + DynamoDB), enable versioning,
   enable encryption. Never store state locally in teams.

3. **Variables** — never hardcode values. Use tfvars per environment.
   Add validation blocks for critical variables.

4. **Modules** — one concern per module. Always define inputs and outputs.
   Pin registry module versions. Add README.md.

5. **Workspaces** — use for environment isolation. Reference terraform.workspace
   in locals for dynamic naming and tagging.

6. **Security** — .gitignore state files, tfvars with secrets, .terraform directory.
   Encrypt state at rest. Restrict S3 bucket access via IAM.

7. **Commands** — always terraform plan before apply. Run fmt and validate
   before committing. Review plan output carefully.

8. **Tagging** — tag every single resource: Project, Environment, ManagedBy.
   Use locals for consistent common_tags across all resources.

9. **Naming** — consistent pattern: project-environment-resource.
   Example: terraweek-prod-vpc, terraweek-dev-server.

10. **Cleanup** — always terraform destroy non-production environments
    when not in use. EKS and NAT Gateways especially — they cost money.


---

### Task 7: Destroy All Environments
Clean up all three environments in reverse order:

```bash
terraform workspace select prod
terraform destroy -var-file="prod.tfvars"

terraform workspace select staging
terraform destroy -var-file="staging.tfvars"

terraform workspace select dev
terraform destroy -var-file="dev.tfvars"
```

Verify in the AWS console -- all VPCs, instances, security groups, and gateways should be gone.

Delete the workspaces:
```bash
terraform workspace select default
terraform workspace delete dev
terraform workspace delete staging
terraform workspace delete prod
```

**Verify:** Is your AWS account completely clean?

---

<img width="711" height="518" alt="image" src="https://github.com/user-attachments/assets/4b038e7f-fd69-4a25-918e-1b9a4a78f2f8" />

<img width="733" height="570" alt="image" src="https://github.com/user-attachments/assets/c420d44c-4993-4ab6-bb6a-79447499664d" />

<img width="709" height="566" alt="image" src="https://github.com/user-attachments/assets/e570ecf0-a59b-4f9d-a866-79029c47818a" />

<img width="1069" height="159" alt="image" src="https://github.com/user-attachments/assets/bf3f4a6a-16c8-4d4f-801a-d4c25acc3e4b" />

## Project Structure
terraweek-capstone/
main.tf          ← Calls all 3 modules, uses workspace-aware locals
variables.tf     ← Input variables (no defaults for env-specific ones)
outputs.tf       ← Instance IP, VPC ID, environment name
providers.tf     ← AWS provider pinned to ~> 5.0
locals.tf        ← environment = terraform.workspace, common_tags
dev.tfvars       ← 10.0.0.0/16, t2.micro, SSH open
staging.tfvars   ← 10.1.0.0/16, t2.micro, HTTPS added
prod.tfvars      ← 10.2.0.0/16, t2.micro, NO SSH
modules/
vpc/            ← VPC, subnet, IGW, route table
security-group/ ← Dynamic ingress rules from port list
ec2-instance/   ← EC2 with environment-aware tags

## How Workspaces Work

| | Without Workspaces | With Workspaces |
|---|---|---|
| Code duplication | 3 copies | 0 — one codebase |
| State files | Manual management | Auto-managed per workspace |
| Environment switch | Change directory | terraform workspace select |

## Environment Differences

| | Dev | Staging | Prod |
|---|---|---|---|
| VPC CIDR | 10.0.0.0/16 | 10.1.0.0/16 | 10.2.0.0/16 |
| SSH (22) | ✅ | ✅ | ❌ |
| HTTP (80) | ✅ | ✅ | ✅ |
| HTTPS (443) | ❌ | ✅ | ✅ |

## Workspace State Location
terraform.tfstate.d/
dev/terraform.tfstate
staging/terraform.tfstate
prod/terraform.tfstate

## TerraWeek Summary

| Day | Concepts Learned |
|---|---|
| 61 | IaC, HCL, init/plan/apply/destroy, state basics |
| 62 | Providers, resources, dependencies, lifecycle rules |
| 63 | Variables, outputs, data sources, locals, functions |
| 64 | Remote backend, locking, import, drift detection |
| 65 | Custom modules, registry modules, version pinning |
| 66 | EKS with modules, real-world cluster provisioning |
| 67 | Workspaces, multi-env, capstone project |

## 10 Terraform Best Practices
1. Separate files for providers, variables, outputs, main, locals
2. Always use remote backend with locking and versioning
3. Never hardcode — tfvars per environment
4. One concern per module, always define inputs/outputs
5. Use workspaces for environment isolation
6. .gitignore state, tfvars, .terraform directory
7. Always plan before apply, fmt and validate before commit
8. Tag every resource: Project, Environment, ManagedBy
9. Consistent naming: project-environment-resource
10. Always destroy non-production environments when not in use

## Summary Table

| Concept | What it means |
|---|---|
| Workspace | Isolated state environment within one codebase |
| `terraform.workspace` | Built-in variable returning current workspace name |
| `terraform workspace new` | Creates a new workspace and switches to it |
| `terraform workspace select` | Switches to an existing workspace |
| `terraform workspace list` | Shows all workspaces (* = current) |
| `terraform workspace delete` | Deletes a workspace (must switch away first) |
| State location | `terraform.tfstate.d/<workspace>/terraform.tfstate` |
| `-var-file` flag | Load environment-specific variables |
| Different CIDRs per env | Prevents VPC overlap if ever peered |
| No SSH in prod | Security hardening — access via bastion or SSM |
| `locals.environment` | `terraform.workspace` used as environment name |
| common_tags | Consistent tags on every resource via merge() |
| Destroy order | Prod → Staging → Dev — always reverse of creation |

