# Day 61 -- Introduction to Terraform and Your First AWS Infrastructure

## Task
You have been deploying containers, writing CI/CD pipelines, and orchestrating workloads on Kubernetes. But who creates the servers, networks, and clusters underneath? Today you start your Infrastructure as Code journey with Terraform -- the tool that lets you define, provision, and manage cloud infrastructure by writing code.

By the end of today, you will have created real AWS resources using nothing but a `.tf` file and a terminal.

---

## Challenge Tasks

### Task 1: Understand Infrastructure as Code
Before touching the terminal, research and write short notes on:

1. What is Infrastructure as Code (IaC)? Why does it matter in DevOps?
2. What problems does IaC solve compared to manually creating resources in the AWS console?
3. How is Terraform different from AWS CloudFormation, Ansible, and Pulumi?
4. What does it mean that Terraform is "declarative" and "cloud-agnostic"?

Write this in your own words -- not copy-pasted definitions.

---

Until now you've been working inside a cluster — deploying apps, managing pods, configuring services. But someone has to create the servers, networks, and clusters that all of that runs on.
That's what Terraform does.
Without Terraform (manual way):

Log into AWS console
Click around to create a server
Click around to create a network
Do it again for staging, prod, dev...
Forget what you clicked, can't reproduce it

With Terraform (IaC way):

Write a .tf file describing what you want
Run one command → AWS creates it
Run another command → AWS destroys it
Same file works for dev, staging, prod
Commit the file to Git → full history of your infrastructure

---

1. What is IaC and why does it matter?
   
Infrastructure as Code means defining your servers, networks, databases, and cloud resources in text files instead of clicking through a 
web console. It matters in DevOps because infrastructure becomes reproducible, version-controlled, and automatable — the same way 
application code is.

2. What problems does IaC solve vs manual console clicks?
   
Problem with manualHow IaC fixes itCan't reproduce exactlySame file always creates same infraNo history of changesGit tracks every 
changeHuman error from clickingCode is reviewed and testedSlow to scaleOne command creates 100 serversDifferent envs drift apartSame 
code for dev/staging/prod

3. How is Terraform different from others?

markdown| Tool | What it does | Key difference |
|---|---|---|
| Terraform | Creates/manages cloud infra | Cloud-agnostic, declarative |
| CloudFormation | Creates AWS infra | AWS only, JSON/YAML |
| Ansible | Configures servers after creation | Procedural, not infra creation |
| Pulumi | Creates cloud infra | Uses real programming languages (Python, JS) |

4. What does declarative and cloud-agnostic mean?

Declarative — You say WHAT you want, not HOW to create it. "I want an S3 bucket" not "run these 10 API calls to create a bucket."
Cloud-agnostic — Same Terraform tool works with AWS, Azure, GCP, Kubernetes, and 1000+ other providers. Just change the provider block.

Concept First
Terraform is a CLI tool you install on your machine. It talks to AWS using your AWS credentials. You need both Terraform AND AWS CLI set up before anything works.



---

### Task 2: Install Terraform and Configure AWS
1. Install Terraform:
```bash
# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Linux (amd64)
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Windows
choco install terraform
```

2. Verify:
```bash
terraform -version
```

3. Install and configure the AWS CLI:
```bash
aws configure
# Enter your Access Key ID, Secret Access Key, default region (e.g., ap-south-1), output format (json)
```

4. Verify AWS access:
```bash
aws sts get-caller-identity
```

You should see your AWS account ID and ARN.

---

<img width="771" height="183" alt="image" src="https://github.com/user-attachments/assets/644b9dc7-d0b9-4112-ac4d-65ae29f9feda" />

---

### Task 3: Your First Terraform Config -- Create an S3 Bucket
Create a project directory and write your first Terraform config:

```bash
mkdir terraform-basics && cd terraform-basics
```

Create a file called `main.tf` with:
1. A `terraform` block with `required_providers` specifying the `aws` provider
2. A `provider "aws"` block with your region
3. A `resource "aws_s3_bucket"` that creates a bucket with a globally unique name

Run the Terraform lifecycle:
```bash
terraform init      # Download the AWS provider
terraform plan      # Preview what will be created
terraform apply     # Create the bucket (type 'yes' to confirm)
```

Go to the AWS S3 console and verify your bucket exists.

**Document:** What did `terraform init` download? What does the `.terraform/` directory contain?

---

Terraform has 3 key blocks:
 
| Block | Purpose | Example |
|---|---|---|
| `terraform {}` | Declares which providers are needed | `required_providers { aws = ... }` |
| `provider "aws" {}` | Configures the provider (region, credentials) | `region = "ap-south-1"` |
| `resource "type" "name" {}` | Declares an actual cloud resource | `resource "aws_s3_bucket" "my_bucket"` |

Terraform lifecycle commands:

| Command | What it does |
|---|---|
| `terraform init` | Downloads provider plugins (like npm install) |
| `terraform plan` | Shows what WILL be created/changed/destroyed (dry run) |
| `terraform apply` | Actually creates the resources in AWS |
| `terraform destroy` | Deletes everything Terraform manages |


created main.tf file:

<img width="1321" height="1008" alt="image" src="https://github.com/user-attachments/assets/81ad96ff-99d8-4858-b45e-6b174c081356" />

<img width="625" height="865" alt="image" src="https://github.com/user-attachments/assets/b73986ab-7d7f-4b10-bfbb-db6d2381a9da" />

<img width="618" height="891" alt="image" src="https://github.com/user-attachments/assets/0e4826ba-276d-4b45-8317-109a2f790b10" />

checked:

<img width="1273" height="627" alt="image" src="https://github.com/user-attachments/assets/fb9bac7a-0a17-46cf-8962-fdd3d3e64d97" />

inside .terraform folder:

<img width="1120" height="384" alt="image" src="https://github.com/user-attachments/assets/99c38c93-755e-4746-9ed3-a9515c78abaa" />

This shows the downloaded AWS provider plugin — the binary that Terraform uses to make AWS API calls.

---

### Task 4: Add an EC2 Instance
In the same `main.tf`, add:
1. A `resource "aws_instance"` using AMI `ami-0f5ee92e2d63afc18` (Amazon Linux 2 in ap-south-1 -- use the correct AMI for your region)
2. Set instance type to `t2.micro`
3. Add a tag: `Name = "TerraWeek-Day1"`

Run:
```bash
terraform plan      # You should see 1 resource to add (bucket already exists)
terraform apply
```

Go to the AWS EC2 console and verify your instance is running with the correct name tag.

**Document:** How does Terraform know the S3 bucket already exists and only the EC2 instance needs to be created?

---

Concept First:

Terraform is smart about state. It already knows the S3 bucket exists (it's tracked in terraform.tfstate). When you add the EC2 instance and run terraform plan, it will show only 1 new resource to add — not touch the bucket


<img width="565" height="987" alt="image" src="https://github.com/user-attachments/assets/c4ce159d-a303-46cb-88ec-7df44922134e" />

Only EC2 is being added! Terraform knows S3 already exists because it's in the state file.

<img width="752" height="751" alt="image" src="https://github.com/user-attachments/assets/a84e6bf0-ac68-42cf-816a-6f21313e83f4" />

<img width="1324" height="319" alt="image" src="https://github.com/user-attachments/assets/b47df59b-4235-4e60-85f3-304f2c627862" />

### How does Terraform know S3 already exists?
Terraform reads terraform.tfstate — the state file tracks everything it has previously created. When you run plan, it compares your .tf file against the state file and only creates what's missing.

---

### Task 5: Understand the State File
Terraform tracks everything it creates in a state file. Time to inspect it.

1. Open `terraform.tfstate` in your editor -- read the JSON structure
2. Run these commands and document what each returns:
```bash
terraform show                          # Human-readable view of current state
terraform state list                    # List all resources Terraform manages
terraform state show aws_s3_bucket.<name>   # Detailed view of a specific resource
terraform state show aws_instance.<name>
```

3. Answer these questions in your notes:
   - What information does the state file store about each resource?
   - Why should you never manually edit the state file?
   - Why should the state file not be committed to Git?

---

Concept First:

The state file (terraform.tfstate) is Terraform's memory. It stores the current known state of every resource it manages. Without it, Terraform would have no idea what already exists in AWS.
Critical rules:

Never manually edit it
Never commit it to Git (contains sensitive data like IPs, ARNs)
Add it to .gitignore

<img width="1341" height="937" alt="image" src="https://github.com/user-attachments/assets/23160640-ea12-4031-92a4-58ae58c764b9" />


It's a JSON file showing every attribute of every resource Terraform created — IDs, IPs, ARNs, tags, everything.

<img width="899" height="1016" alt="image" src="https://github.com/user-attachments/assets/baef8a1a-82c7-49ef-a249-4e845f8c4000" />

Shows all resources in a readable format.

<img width="569" height="104" alt="image" src="https://github.com/user-attachments/assets/afe340ee-d556-4a71-ae14-000a88c01ce9" />

Detailed view of particular resource:

<img width="772" height="929" alt="image" src="https://github.com/user-attachments/assets/8b30cd12-7e36-46e4-af1a-c3ea38694ac4" />

.gitignore:
<img width="537" height="268" alt="image" src="https://github.com/user-attachments/assets/005610b8-a981-417a-bf3e-3f263fdba6ea" />

Answers for your notes:

State file stores: resource IDs, ARNs, IPs, all attributes AWS assigned at creation

Never manually edit: Terraform will get confused and try to recreate or delete resources it thinks don't match

Never commit to Git: contains sensitive resource details, and if two people have different state files, conflicts destroy infrastructure

---

### Task 6: Modify, Plan, and Destroy
1. Change the EC2 instance tag from `"TerraWeek-Day1"` to `"TerraWeek-Modified"` in your `main.tf`
2. Run `terraform plan` and read the output carefully:
   - What do the `~`, `+`, and `-` symbols mean?
   - Is this an in-place update or a destroy-and-recreate?
3. Apply the change
4. Verify the tag changed in the AWS console
5. Finally, destroy everything:
```bash
terraform destroy
```
6. Verify in the AWS console -- both the S3 bucket and EC2 instance should be gone

---

| Symbol | Meaning |
|---|---|
| `+` (green) | Resource will be CREATED |
| `-` (red) | Resource will be DESTROYED |
| `~` (yellow) | Resource will be UPDATED in-place |
| `-/+` | Resource will be DESTROYED and RECREATED |

Tag changes are usually ~ (in-place update) — no need to destroy and recreate the instance.

Modified the tag for ec2 instance:

<img width="948" height="1038" alt="image" src="https://github.com/user-attachments/assets/225625c7-74f9-4f79-936b-12ae48013b69" />

Changes in AWS console:

<img width="1573" height="436" alt="image" src="https://github.com/user-attachments/assets/3ab3836e-9037-4cf6-9cf1-d041dd7d9e93" />

terraform fmt :

This auto-formats spacing and indentation in your .tf files.

Validate syntax:

terraform validate

Output:
Success! The configuration is valid.

Destroyed:

<img width="1119" height="902" alt="image" src="https://github.com/user-attachments/assets/6f27e6cf-59bd-4182-9bfe-dd19e655130d" />

---

## What is Infrastructure as Code?
Infrastructure as Code means writing your cloud resources — servers, networks,
buckets — as code in text files instead of clicking through a web console.
It makes infrastructure reproducible, version-controlled, and automatable,
the same way application code is.

## IaC vs Manual Console

| Problem with manual | How IaC fixes it |
|---|---|
| Can't reproduce exactly | Same file always creates same infra |
| No history of changes | Git tracks every change |
| Human error from clicking | Code is reviewed and tested |
| Slow to scale | One command creates 100 servers |

## Terraform vs Other Tools

| Tool | What it does | Key difference |
|---|---|---|
| Terraform | Creates/manages cloud infra | Cloud-agnostic, declarative |
| CloudFormation | Creates AWS infra | AWS only |
| Ansible | Configures servers | Not for infra creation |
| Pulumi | Creates cloud infra | Uses real programming languages |

## Terraform Commands

| Command | What it does |
|---|---|
| `terraform init` | Downloads provider plugins |
| `terraform plan` | Dry run — shows what will change |
| `terraform apply` | Creates/updates real resources in AWS |
| `terraform destroy` | Deletes all managed resources |
| `terraform show` | Human-readable view of current state |
| `terraform state list` | Lists all resources Terraform manages |
| `terraform fmt` | Auto-formats .tf files |
| `terraform validate` | Checks for syntax errors |

## Plan Symbols

| Symbol | Meaning |
|---|---|
| `+` green | Resource will be CREATED |
| `-` red | Resource will be DESTROYED |
| `~` yellow | Resource will be UPDATED in-place |
| `-/+` | Destroy and recreate |

## The State File
terraform.tfstate tracks everything Terraform has created — IDs, ARNs,
IPs, all attributes. It's Terraform's memory.

Rules:
- Never manually edit it — Terraform will get confused
- Never commit to Git — contains sensitive data
- Always add to .gitignore

## What terraform init downloads
The .terraform/ directory contains the provider plugin — in our case the
AWS provider binary that makes the actual AWS API calls.

---

| Concept | What it means |
|---|---|
| IaC | Define cloud infra in code files instead of clicking in console |
| Declarative | Say WHAT you want, not HOW to create it |
| Cloud-agnostic | Same Terraform works with AWS, Azure, GCP, and 1000+ providers |
| `terraform init` | Downloads provider plugins — like npm install |
| `terraform plan` | Dry run — shows changes without applying them |
| `terraform apply` | Creates real resources in AWS |
| `terraform destroy` | Deletes everything Terraform manages |
| State file | Terraform's memory — tracks all created resources |
| `.terraform/` | Downloaded provider plugins — don't commit to Git |
| `*.tfstate` | State file — never commit, add to .gitignore |
| Provider block | Configures which cloud and region to use |
| Resource block | Declares one cloud resource (S3 bucket, EC2 instance) |

