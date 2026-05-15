# Day 65 -- Terraform Modules: Build Reusable Infrastructure

## Task
You have been writing everything in one big `main.tf` file. That works for learning, but in real teams you manage dozens of environments with hundreds of resources. Copy-pasting configs across projects is a recipe for disaster.

Today you learn Terraform modules -- the way to package, reuse, and share infrastructure code. Think of modules as functions in programming. Write once, call many times.

---

What Are We Learning Today?
Look at your previous Terraform configs. Every time you want to deploy a new EC2 instance, you copy-paste the entire aws_instance block. Want 3 EC2 instances? Copy-paste 3 times. Want to change something? Edit 3 places. Miss one? Bug in production.
Modules fix this — they're like functions in programming:

    Without modules:                    With modules:
  
    aws_instance (web)                  module "web_server" {
    aws_instance (api)       →            source = "./modules/ec2-instance"
    aws_instance (db)                     instance_name = "web"
    # 3x copy-pasted blocks             }
                                        module "api_server" {
                                          source = "./modules/ec2-instance"
                                          instance_name = "api"
                                        }
                                        # Same module, different inputs



Two types of modules:

| Type | What it means | Example |
|---|---|---|
| Root module | Your main config — calls other modules | `main.tf` in your project |
| Child module | Reusable package of resources | `modules/ec2-instance/` |
| Registry module | Pre-built module from Terraform Registry | `terraform-aws-modules/vpc/aws` |

---

## Challenge Tasks

### Task 1: Understand Module Structure
A Terraform module is just a directory with `.tf` files. Create this structure:

```
terraform-modules/
  main.tf                    # Root module -- calls child modules
  variables.tf               # Root variables
  outputs.tf                 # Root outputs
  providers.tf               # Provider config
  modules/
    ec2-instance/
      main.tf                # EC2 resource definition
      variables.tf           # Module inputs
      outputs.tf             # Module outputs
    security-group/
      main.tf                # Security group resource definition
      variables.tf           # Module inputs
      outputs.tf             # Module outputs
```

Create all the directories and empty files. This is the standard layout every Terraform project follows.

**Document:** What is the difference between a "root module" and a "child module"?

---

Concept First:

A module is just a folder with .tf files. Nothing special. When you call a module, Terraform runs all the .tf files in that folder with the inputs you provide.
Every module has the same 3 files:

variables.tf — what inputs it accepts (like function parameters)
main.tf — what resources it creates
outputs.tf — what values it exposes back to the caller (like return values)

Created Modules folders:

<img width="644" height="555" alt="image" src="https://github.com/user-attachments/assets/d0cef4b4-239b-4dd0-98eb-fb89859e0163" />

File structure:

<img width="639" height="216" alt="image" src="https://github.com/user-attachments/assets/53c5de9f-a641-4c2a-947b-e66688ff70bf" />

Answer: Root module vs Child module:

Root module = your project's main config that calls other modules — terraform apply is run here
Child module = reusable package called from the root module — never run directly

---

### Task 2: Build a Custom EC2 Module
Create `modules/ec2-instance/`:

1. **`variables.tf`** -- define inputs:
   - `ami_id` (string)
   - `instance_type` (string, default: `"t2.micro"`)
   - `subnet_id` (string)
   - `security_group_ids` (list of strings)
   - `instance_name` (string)
   - `tags` (map of strings, default: `{}`)

2. **`main.tf`** -- define the resource:
   - `aws_instance` using all the variables
   - Merge the Name tag with additional tags

3. **`outputs.tf`** -- expose:
   - `instance_id`
   - `public_ip`
   - `private_ip`

Do NOT apply yet -- just write the module.

---

Concept First:

Think of this module as a template. You write it once and can call it 10 times with different inputs to create 10 different EC2 instances — all with the same standards applied.

main.tf:

<img width="710" height="327" alt="image" src="https://github.com/user-attachments/assets/79a6bf30-c88e-4a13-b11f-f75326508a8f" />

nothing is hardcoded , its coming from variables.tf

variables.tf:

<img width="729" height="448" alt="image" src="https://github.com/user-attachments/assets/ad98b9e2-e0f2-4d18-a0c6-6cfed4ce1fda" />

output.tf:

<img width="648" height="429" alt="image" src="https://github.com/user-attachments/assets/3333180c-fbe3-4a54-80c8-2b78d89827c8" />

 EC2 module is done — 3 files, clean, reusable.

 ---

### Task 3: Build a Custom Security Group Module
Create `modules/security-group/`:

1. **`variables.tf`** -- define inputs:
   - `vpc_id` (string)
   - `sg_name` (string)
   - `ingress_ports` (list of numbers, default: `[22, 80]`)
   - `tags` (map of strings, default: `{}`)

2. **`main.tf`** -- define the resource:
   - `aws_security_group` in the given VPC
   - Use `dynamic "ingress"` block to create rules from the `ingress_ports` list
   - Allow all egress

3. **`outputs.tf`** -- expose:
   - `sg_id`

This is your first time using a `dynamic` block -- it loops over a list to generate repeated nested blocks.

---

Concept First — Dynamic Blocks
Normally if you want 3 ingress rules, you write 3 ingress {} blocks. But what if the number of rules is variable — sometimes 2, sometimes 5?

A dynamic block loops over a list and generates repeated nested blocks automatically:

    # Without dynamic — hardcoded
    ingress { from_port = 22  ... }
    ingress { from_port = 80  ... }
    ingress { from_port = 443 ... }
    
    # With dynamic — generated from list
    dynamic "ingress" {
      for_each = var.ingress_ports    # Loop over [22, 80, 443]
      content {
        from_port = ingress.value     # Each iteration uses the port
        ...
      }
    }

variables.tf:

<img width="676" height="645" alt="image" src="https://github.com/user-attachments/assets/e2bd9210-95a1-488e-b97a-e098b5d6b2db" />

main.tf:

<img width="754" height="687" alt="image" src="https://github.com/user-attachments/assets/2f2c4393-2142-4e2a-8fcf-9079a9cef4ae" />

output.tf:

<img width="531" height="144" alt="image" src="https://github.com/user-attachments/assets/8882b253-a5e5-46b4-9c54-3716d5f28bdb" />


Security group module done. Notice how module.web_sg.sg_id will return this value to the root.

---

### Task 4: Call Your Modules from Root
In the root `main.tf`, wire everything together:

1. Create a VPC and subnet directly (or reuse your Day 62 config)
2. Call the security group module:
```hcl
module "web_sg" {
  source        = "./modules/security-group"
  vpc_id        = aws_vpc.main.id
  sg_name       = "terraweek-web-sg"
  ingress_ports = [22, 80, 443]
  tags          = local.common_tags
}
```

3. Call the EC2 module -- deploy **two instances** with different names using the same module:
```hcl
module "web_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t2.micro"
  subnet_id          = aws_subnet.public.id
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "terraweek-web"
  tags               = local.common_tags
}

module "api_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t2.micro"
  subnet_id          = aws_subnet.public.id
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "terraweek-api"
  tags               = local.common_tags
}
```

4. Add root outputs that reference module outputs:
```hcl
output "web_server_ip" {
  value = module.web_server.public_ip
}

output "api_server_ip" {
  value = module.api_server.public_ip
}
```

5. Apply:
```bash
terraform init    # Downloads/links the local modules
terraform plan    # Should show all resources from both module calls
terraform apply
```

**Verify:** Two EC2 instances running, same security group, different names. Check the AWS console.

---

Concept First — Module Syntax:

    module "name_you_choose" {     # Like calling a function
      source = "./modules/ec2-instance"  # Where the module lives
      
      # These match the variable names in the module's variables.tf
      ami_id        = "ami-xxx"
      instance_name = "web-server"
    }
    
    # Access module outputs like this:
    module.name_you_choose.public_ip

output.tf:

<img width="673" height="694" alt="image" src="https://github.com/user-attachments/assets/33468814-f9d6-4cc3-b593-e57d5c99a3e2" />

variables.tf:

<img width="504" height="464" alt="image" src="https://github.com/user-attachments/assets/c17741a5-f059-4d2f-a7ac-15a2eaf005fb" />

main.tf:

<img width="717" height="946" alt="image" src="https://github.com/user-attachments/assets/ce682bcf-3ca8-4777-b2fa-56df28d0128f" />

<img width="647" height="751" alt="image" src="https://github.com/user-attachments/assets/06d397e8-b3a6-49b8-a7b0-dd4331aab475" />

<img width="637" height="766" alt="image" src="https://github.com/user-attachments/assets/5f7ade40-8fd9-4e8c-9fda-53285e209240" />

terraform init:

<img width="748" height="522" alt="image" src="https://github.com/user-attachments/assets/df25c857-5075-425e-b473-9e022579b18e" />

terraform plan:

<img width="713" height="824" alt="image" src="https://github.com/user-attachments/assets/0b8ec236-2138-40f9-9f6b-eb3099d67400" />

terraform apply:

<img width="860" height="811" alt="image" src="https://github.com/user-attachments/assets/247ad41e-f96d-49e2-bdb1-503b2b897bb3" />

Verified:

<img width="1216" height="143" alt="image" src="https://github.com/user-attachments/assets/7a4c6518-7233-4892-8ae6-9e6703922ebd" />

same security group:

<img width="514" height="167" alt="image" src="https://github.com/user-attachments/assets/13f202f2-9231-45ab-ab87-90df563a0965" />

---

### Task 5: Use a Public Registry Module
Instead of building your own VPC from scratch, use the official module from the Terraform Registry.

1. Replace your hand-written VPC resources with:
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "terraweek-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-south-1a", "ap-south-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]

  enable_nat_gateway = false
  enable_dns_hostnames = true

  tags = local.common_tags
}
```

2. Update your EC2 and SG module calls to reference `module.vpc.vpc_id` and `module.vpc.public_subnets[0]`

3. Run:
```bash
terraform init     # Downloads the registry module
terraform plan
terraform apply
```

4. Compare: how many resources did the VPC module create vs your hand-written VPC from Day 62?

**Document:** Where does Terraform download registry modules to? Check `.terraform/modules/`.

---

Concept First
The Terraform Registry (registry.terraform.io) has thousands of pre-built modules. Instead of writing 50+ lines of VPC code yourself, you use the official terraform-aws-modules/vpc/aws module — tested, maintained, and used by thousands of teams.

| | Hand-written VPC (Day 62) | Registry VPC module |
|---|---|---|
| Lines of code | ~50 lines | ~15 lines |
| Resources created | 5 | 15+ (subnets, route tables, NACLs...) |
| Maintenance | You | Module maintainers |
| Features | Basic | Full production-ready |

terraform init:

Output:

    Initializing modules...
    Downloading registry.terraform.io/terraform-aws-modules/vpc/aws 5.x.x for vpc...
    - vpc in .terraform/modules/vpc
    - api_server in modules/ec2-instance
    - web_server in modules/ec2-instance
    - web_sg in modules/security-group

<img width="1624" height="408" alt="image" src="https://github.com/user-attachments/assets/7998e4a6-1e41-4135-b42c-874532d11dec" />

ls .terraform/modules/ :

Output:

    modules.json
    vpc/          ← Registry module downloaded here

<img width="1593" height="195" alt="image" src="https://github.com/user-attachments/assets/7c468585-47de-409c-ba86-9375bbd7eccd" />

ls .terraform/modules/vpc/ :

You'll see all the files of the VPC module — it's just more .tf files, like your custom modules!

<img width="1603" height="442" alt="image" src="https://github.com/user-attachments/assets/070d39a3-7ae8-4e2b-9042-f9e8715f94da" />

Destoryed:

<img width="746" height="467" alt="image" src="https://github.com/user-attachments/assets/93bdd4b2-490c-4a54-9ebc-27535879b585" />

terraform plan:

<img width="578" height="859" alt="image" src="https://github.com/user-attachments/assets/370b0e53-e992-4e8b-bc18-caa4dea572f5" />

Notice — the VPC module creates MANY more resources than your hand-written VPC:

Your hand-written VPC: 5 resources
Registry VPC module: 15+ resources (more complete, production-ready)

apply:

<img width="814" height="627" alt="image" src="https://github.com/user-attachments/assets/1671f0b8-0dc2-4dfb-b28d-b345ff4a5c00" />

Verified: Same two EC2 instances running, but now inside a VPC created by the registry module.

---

### Task 6: Module Versioning and Best Practices
1. Pin your registry module version explicitly:
   - `version = "5.1.0"` -- exact version
   - `version = "~> 5.0"` -- any 5.x version
   - `version = ">= 5.0, < 6.0"` -- range

2. Run `terraform init -upgrade` to check for newer versions

3. Check the state to see how modules appear:
```bash
terraform state list
```
Notice the `module.vpc.`, `module.web_server.`, `module.web_sg.` prefixes.

4. Destroy everything:
```bash
terraform destroy
```

**Document:** Write down five module best practices:
- Always pin versions for registry modules
- Keep modules focused -- one concern per module
- Use variables for everything, hardcode nothing
- Always define outputs so callers can reference resources
- Add a README.md to every custom module

---

 Concept First — Version Pinning:

| Version constraint | What it allows | Best for |
|---|---|---|
| `version = "5.1.0"` | Exactly 5.1.0 only | Maximum stability |
| `version = "~> 5.0"` | Any 5.x (5.0, 5.1, 5.9) | Recommended |
| `version = ">= 5.0, < 6.0"` | Explicit range | When you need fine control |

terraform init -upgrade:

<img width="1164" height="621" alt="image" src="https://github.com/user-attachments/assets/a6e513f0-a524-4e71-b2af-7950c48560ce" />


<img width="1662" height="400" alt="image" src="https://github.com/user-attachments/assets/985f76c2-eee0-4049-bc08-e5324da616c6" />

Every resource is prefixed with module.<name>. — this is how Terraform organises state for modules.

did terraform destroy.

---

## What is a Module?
A module is a folder with .tf files — write once, call many times.
Like functions in programming: define inputs (variables), logic (main.tf),
and outputs. Call the same module with different inputs to get different resources.

## Module Types

| Type | What it is | Example |
|---|---|---|
| Root module | Your main config — runs terraform apply | `./main.tf` |
| Child module | Custom reusable package | `./modules/ec2-instance/` |
| Registry module | Pre-built from Terraform Registry | `terraform-aws-modules/vpc/aws` |

## Directory Structure
    
    terraform-modules/
    main.tf              ← Root: calls modules
    variables.tf
    outputs.tf
    providers.tf
    modules/
    ec2-instance/
    main.tf          ← Creates aws_instance
    variables.tf     ← ami_id, instance_type, subnet_id...
    outputs.tf       ← instance_id, public_ip, private_ip
    security-group/
    main.tf          ← Creates aws_security_group with dynamic block
    variables.tf     ← vpc_id, sg_name, ingress_ports...
    outputs.tf       ← sg_id


## How to Call a Module
```hcl
module "web_server" {
  source        = "./modules/ec2-instance"   # local module
  ami_id        = data.aws_ami.amazon_linux.id
  instance_name = "web"
}

# Access outputs:
module.web_server.public_ip
```

## Dynamic Block
Used to create repeated nested blocks from a list:
```hcl
dynamic "ingress" {
  for_each = var.ingress_ports
  content {
    from_port   = ingress.value
    to_port     = ingress.value
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

## Hand-written VPC vs Registry VPC Module

| | Hand-written (Day 62) | Registry module |
|---|---|---|
| Lines of code | ~50 | ~15 |
| Resources created | 5 | 15+ |
| Maintenance | You | Module maintainers |
| Production-ready | Basic | Full |

## Where Registry Modules Are Downloaded
`.terraform/modules/` — downloaded during `terraform init`

## Module Version Constraints

| Constraint | Allows | Use when |
|---|---|---|
| `"5.1.0"` | Exactly 5.1.0 | Maximum stability |
| `"~> 5.0"` | Any 5.x | Recommended |
| `">= 5.0, < 6.0"` | Explicit range | Fine control |

## 5 Module Best Practices
1. Always pin versions for registry modules — never use unversioned modules in production
2. Keep modules focused — one concern per module (EC2 module only does EC2)
3. Use variables for everything — hardcode nothing inside a module
4. Always define outputs — callers need to reference created resources
5. Add README.md to every custom module — document inputs, outputs, and usage examples


Summary:

| Concept | What it means |
|---|---|
| Module | A folder of .tf files — reusable infrastructure package |
| Root module | Your project's main config — where terraform apply runs |
| Child module | Called from root — defines reusable resources |
| Registry module | Pre-built module from registry.terraform.io |
| `source = "./modules/ec2-instance"` | Call a local module |
| `source = "terraform-aws-modules/vpc/aws"` | Call a registry module |
| `version = "~> 5.0"` | Pin to 5.x — recommended for registry modules |
| `module.<name>.<output>` | Access a module's output value |
| `dynamic` block | Generate repeated nested blocks from a list |
| `for_each` in dynamic | The list to loop over |
| `terraform init` | Must re-run after adding new module sources |
| `terraform init -upgrade` | Check for newer module versions |
| `.terraform/modules/` | Where registry modules are downloaded |
| `module.*` prefix in state | How Terraform organises module resources in state |
