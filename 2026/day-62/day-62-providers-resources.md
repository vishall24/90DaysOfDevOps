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



