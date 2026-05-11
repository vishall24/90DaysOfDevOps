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

