# Day 62 -- Providers, Resources and Dependencies

## Task
Yesterday you created standalone resources. But real infrastructure is connected -- a server lives inside a subnet, a subnet lives inside a VPC, a security group controls what traffic gets in. Today you build a complete networking stack on AWS and learn how Terraform figures out what to create first.

Understanding dependencies is what separates a Terraform beginner from someone who can build production infrastructure.

---

What Are We Learning Today?
Yesterday you created standalone resources — an S3 bucket and an EC2 instance that didn't talk to each other.
Today you build real connected infrastructure — the kind that actually exists in production:

    Internet
        ↓
    Internet Gateway
        ↓
    Route Table → Subnet (10.0.1.0/24)
        ↓                ↓
       VPC (10.0.0.0/16)
                         ↓
                  Security Group
                         ↓
                   EC2 Instance
                   
The key lesson today: Terraform figures out what to create first by reading the dependencies between your resources. You don't have to tell it the order — it figures it out from the references in your code.

---

## Challenge Tasks

### Task 1: Explore the AWS Provider
1. Create a new project directory: `terraform-aws-infra`
2. Write a `providers.tf` file:
   - Define the `terraform` block with `required_providers` pinning the AWS provider to version `~> 5.0`
   - Define the `provider "aws"` block with your region
3. Run `terraform init` and check the output -- what version was installed?
4. Read the provider lock file `.terraform.lock.hcl` -- what does it do?

**Document:** What does `~> 5.0` mean? How is it different from `>= 5.0` and `= 5.0.0`?

---

Concept First — Version Constraints:

When you write ~> 5.0 for the AWS provider, you're telling Terraform:

| Constraint | Meaning | Allows |
|---|---|---|
| `~> 5.0` | Pessimistic — allow patch/minor updates | 5.0, 5.1, 5.9 but NOT 6.0 |
| `>= 5.0` | Anything 5.0 or higher | 5.0, 5.1, 6.0, 7.0... |
| `= 5.0.0` | Exactly this version only | Only 5.0.0 |

~> 5.0 is the recommended approach — you get bug fixes and new features but no breaking changes.

<img width="1238" height="1027" alt="image" src="https://github.com/user-attachments/assets/f5306332-b542-4e61-8b2c-95ce9d0bee8b" />

This file locks the exact version that was installed. It's like package-lock.json in Node.js — ensures everyone on your team uses the same provider version.

Always commit .terraform.lock.hcl to Git — it's safe and important.
Never commit .terraform/ — it's huge and machine-specific.
 
Verify: Note the exact version installed.

---

### Task 2: Build a VPC from Scratch
Create a `main.tf` and define these resources one by one:

1. `aws_vpc` -- CIDR block `10.0.0.0/16`, tag it `"TerraWeek-VPC"`
2. `aws_subnet` -- CIDR block `10.0.1.0/24`, reference the VPC ID from step 1, enable public IP on launch, tag it `"TerraWeek-Public-Subnet"`
3. `aws_internet_gateway` -- attach it to the VPC
4. `aws_route_table` -- create it in the VPC, add a route for `0.0.0.0/0` pointing to the internet gateway
5. `aws_route_table_association` -- associate the route table with the subnet

Run `terraform plan` -- you should see 5 resources to create.

**Verify:** Apply and check the AWS VPC console. Can you see all five resources connected?

---

Concept First — What Are We Building?
Think of it like setting up a private office building:

| AWS Resource | Real world analogy |
|---|---|
| VPC | The entire office building + land |
| Subnet | One floor of the building |
| Internet Gateway | The main entrance/exit door |
| Route Table | The directory that tells you which exit to use |
| Route Table Association | Pinning the directory to a specific floor |

<img width="853" height="1019" alt="image" src="https://github.com/user-attachments/assets/ffa361d9-d6c7-4a45-984e-167f4fce69b0" />

<img width="1501" height="450" alt="image" src="https://github.com/user-attachments/assets/77089992-dc6c-4315-8ae0-faf3381d2223" />

<img width="1663" height="378" alt="image" src="https://github.com/user-attachments/assets/2e6d2b18-bea5-4af6-9668-c7aea9d3fcfc" />

<img width="1479" height="420" alt="image" src="https://github.com/user-attachments/assets/0c095c05-4aa5-47ee-880f-9413b6890668" />

<img width="1344" height="318" alt="image" src="https://github.com/user-attachments/assets/2240f1a7-5303-4aa9-9274-d752c5321806" />

---

### Task 3: Understand Implicit Dependencies
Look at your `main.tf` carefully:

1. The subnet references `aws_vpc.main.id` -- this is an implicit dependency
2. The internet gateway references the VPC ID -- another implicit dependency
3. The route table association references both the route table and the subnet

Answer these questions:
- How does Terraform know to create the VPC before the subnet?
- What would happen if you tried to create the subnet before the VPC existed?
- Find all implicit dependencies in your config and list them

---

Concept First:

When you write aws_vpc.main.id inside a resource, Terraform reads that reference and thinks: "Ah, this resource needs the VPC to exist first — I'll create the VPC before this resource."
That's an implicit dependency — you never told Terraform the order, it figured it out from your code.

Step 1: Find ALL implicit dependencies in your config
Look at your main.tf and trace every reference:

| Resource | References | So must wait for... |
|---|---|---|
| `aws_vpc.main` | Nothing | Creates first |
| `aws_subnet.public` | `aws_vpc.main.id` | VPC |
| `aws_internet_gateway.main` | `aws_vpc.main.id` | VPC |
| `aws_route_table.public` | `aws_vpc.main.id`, `aws_internet_gateway.main.id` | VPC AND IGW |
| `aws_route_table_association.public` | `aws_subnet.public.id`, `aws_route_table.public.id` | Subnet AND Route Table |

### Q: How does Terraform know to create VPC before subnet?
The subnet block contains vpc_id = aws_vpc.main.id. Terraform sees this reference and knows the subnet depends on the VPC. It builds a dependency graph internally and creates resources in the correct order.

### Q: What would happen if subnet was created before VPC?
It would fail — AWS would return an error saying "VPC does not exist." Terraform prevents this by always resolving dependencies first.

---

### Task 4: Add a Security Group and EC2 Instance
Add to your config:

1. `aws_security_group` in the VPC:
   - Ingress rule: allow SSH (port 22) from `0.0.0.0/0`
   - Ingress rule: allow HTTP (port 80) from `0.0.0.0/0`
   - Egress rule: allow all outbound traffic
   - Tag: `"TerraWeek-SG"`

2. `aws_instance` in the subnet:
   - Use Amazon Linux 2 AMI for your region
   - Instance type: `t2.micro`
   - Associate the security group
   - Set `associate_public_ip_address = true`
   - Tag: `"TerraWeek-Server"`

Apply and verify -- your EC2 instance should have a public IP and be reachable.

---

Concept First — Security Groups
A security group is a virtual firewall for your EC2 instance. It controls:

Ingress = incoming traffic (who can connect TO your instance)
Egress = outgoing traffic (what your instance can connect TO)

<img width="629" height="834" alt="image" src="https://github.com/user-attachments/assets/2b4a3a32-adfb-475f-b8bd-f47632d8e3f0" />

<img width="1197" height="577" alt="image" src="https://github.com/user-attachments/assets/f7be50a9-3dbb-4689-83ab-81238bb52b55" />

its actually 2 resources but 1 was added before only that why showing 1.

<img width="1512" height="390" alt="image" src="https://github.com/user-attachments/assets/93ec7a3b-625a-4431-a628-2fed2567b83b" />


---

### Task 5: Explicit Dependencies with depends_on
Sometimes Terraform cannot detect a dependency automatically.

1. Add a second `aws_s3_bucket` resource for application logs
2. Add `depends_on = [aws_instance.main]` to the S3 bucket -- even though there is no direct reference, you want the bucket created only after the instance
3. Run `terraform plan` and observe the order

Now visualize the entire dependency tree:
```bash
terraform graph | dot -Tpng > graph.png
```
If you don't have `dot` (Graphviz) installed, use:
```bash
terraform graph
```
and paste the output into an online Graphviz viewer.

**Document:** When would you use `depends_on` in real projects? Give two examples.

---

Concept First:

Sometimes two resources have NO direct reference to each other in code, but you still want one created before the other. That's when you use depends_on.
Real examples:

An S3 bucket for logs should be created before the app that writes to it
An IAM role should exist before an EC2 instance that uses it

<img width="1218" height="884" alt="image" src="https://github.com/user-attachments/assets/e9b48485-5dc3-4362-81e5-37e2e3979dc6" />

Visual representation of graph from terraform graph:

<img width="928" height="402" alt="image" src="https://github.com/user-attachments/assets/8a1c00b9-7b19-4665-ac8a-7d6697a0ad59" />

When to use depends_on:

S3 bucket before app — Your app writes logs to S3, but there's no Terraform reference between them. Use depends_on to ensure the bucket exists first.

IAM role before EC2 — EC2 needs an IAM role attached, but if there's no iam_instance_profile reference, use depends_on to ensure the role is ready.


---

### Task 6: Lifecycle Rules and Destroy
1. Add a `lifecycle` block to your EC2 instance:
```hcl
lifecycle {
  create_before_destroy = true
}
```
2. Change the AMI ID to a different one and run `terraform plan` -- observe that Terraform plans to create the new instance before destroying the old one

3. Destroy everything:
```bash
terraform destroy
```
4. Watch the destroy order -- Terraform destroys in reverse dependency order. Verify in the AWS console that everything is cleaned up.

**Document:** What are the three lifecycle arguments (`create_before_destroy`, `prevent_destroy`, `ignore_changes`) and when would you use each?

---

Concept First — Lifecycle Arguments:

| Lifecycle argument | What it does | When to use |
|---|---|---|
| `create_before_destroy` | Creates new resource BEFORE destroying old one | AMI changes, zero-downtime deploys |
| `prevent_destroy` | Blocks `terraform destroy` for this resource | Production databases, critical infra |
| `ignore_changes` | Ignores changes to specific attributes | Tags managed outside Terraform, auto-scaling |


<img width="796" height="709" alt="image" src="https://github.com/user-attachments/assets/fd84ef65-7d67-4848-8e68-216935b7950f" />

Without lifecycle:

-/+ aws_instance.main (destroy then create)

With create_before_destroy:

+/- aws_instance.main (create then destroy)

The + comes first — new instance is created before old one is terminated. This prevents downtime.

terraform destory:

Terraform destroys in reverse dependency order:
aws_instance.main — destroyed first
aws_s3_bucket.logs — destroyed
aws_security_group.main — destroyed
aws_route_table_association.public — destroyed
aws_route_table.public — destroyed
aws_internet_gateway.main — destroyed
aws_subnet.public — destroyed
aws_vpc.main — destroyed last
The VPC is always last because everything else depends on it!

<img width="767" height="650" alt="image" src="https://github.com/user-attachments/assets/95d30159-669e-43a1-b835-84b256d88d2a" />

deleted from aws cloud as well.

---

## Provider Version Constraints

| Constraint | Meaning | Allows |
|---|---|---|
| `~> 5.0` | Pessimistic — allow minor/patch updates | 5.0, 5.1, 5.9 but NOT 6.0 |
| `>= 5.0` | Anything 5.0 or higher | 5.0, 6.0, 7.0... |
| `= 5.0.0` | Exactly this version | Only 5.0.0 |

## What the Lock File Does
`.terraform.lock.hcl` records the exact provider version installed.
Like package-lock.json in Node.js — ensures all team members use
the same provider version. Always commit this file to Git.

## Resources Built Today

| Resource | Purpose |
|---|---|
| `aws_vpc` | Main network container — 10.0.0.0/16 |
| `aws_subnet` | Public subnet — 10.0.1.0/24 |
| `aws_internet_gateway` | Door between VPC and internet |
| `aws_route_table` | Traffic rules — send 0.0.0.0/0 to IGW |
| `aws_route_table_association` | Links route table to subnet |
| `aws_security_group` | Firewall — allow SSH (22) and HTTP (80) |
| `aws_instance` | EC2 server inside the subnet |
| `aws_s3_bucket` | Log storage with explicit depends_on |

## Implicit vs Explicit Dependencies

| Type | How defined | Example |
|---|---|---|
| Implicit | Reference in code `aws_vpc.main.id` | Subnet referencing VPC ID |
| Explicit | `depends_on = [resource]` | S3 bucket depending on EC2 |

## Dependency Order (Creation)
VPC → Subnet + IGW → Route Table → Route Table Association → SG → EC2 → S3

## Destroy Order
Reverse of creation — S3 → EC2 → SG → Route Table Association →
Route Table → IGW + Subnet → VPC

## Lifecycle Arguments

| Argument | What it does | Use case |
|---|---|---|
| `create_before_destroy` | New resource created before old one destroyed | Zero-downtime AMI changes |
| `prevent_destroy` | Blocks terraform destroy for this resource | Production RDS databases |
| `ignore_changes` | Ignores drift on specific attributes | Tags managed by another team |

## When to Use depends_on
1. S3 bucket should exist before app that writes to it (no Terraform reference between them)
2. IAM role must exist before EC2 instance that assumes it

---

| Concept | What it means |
|---|---|
| Provider | Plugin that lets Terraform talk to a cloud (AWS, GCP, Azure) |
| `~> 5.0` | Allow 5.x updates but not 6.x — recommended version constraint |
| `.terraform.lock.hcl` | Locks exact provider version — commit this to Git |
| Implicit dependency | Detected from resource references like `aws_vpc.main.id` |
| Explicit dependency | Manually declared with `depends_on = [resource]` |
| VPC | Private network in AWS — container for all other network resources |
| Subnet | A segment of the VPC — where EC2 instances actually live |
| Internet Gateway | Connects your VPC to the public internet |
| Route Table | Rules for where network traffic should go |
| Security Group | Virtual firewall — controls inbound and outbound traffic |
| `create_before_destroy` | Creates new resource before destroying old one — zero downtime |
| `prevent_destroy` | Blocks accidental terraform destroy on critical resources |
| `ignore_changes` | Tells Terraform to ignore drift on specific attributes |
| `terraform graph` | Outputs dependency graph in DOT format — visualize at dreampuf.github.io |
