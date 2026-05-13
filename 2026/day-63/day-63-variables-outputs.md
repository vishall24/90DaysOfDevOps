# Day 63 -- Variables, Outputs, Data Sources and Expressions

## Task
Your Day 62 config works, but it is full of hardcoded values -- region, CIDR blocks, AMI IDs, instance types, tags. Change the region and everything breaks. Today you make your Terraform configs dynamic, reusable, and environment-aware.

This is the difference between a config that works once and a config you can use across projects.

---

What Are We Learning Today?
 
Look at your Day 62 config. It has hardcoded values everywhere:

hclregion        = "ap-south-1"      # hardcoded
cidr_block    = "10.0.0.0/16"     # hardcoded
instance_type = "t2.micro"        # hardcoded
ami           = "ami-0f5ee92e2d63afc18"  # hardcodedProblems with hardcoded values:

Want to deploy to a different region? Edit 10 files.
Want prod to use bigger instances? Edit manually.
Want a teammate to use the same config? They have to change everything.

Today you fix all of this by making your config fully dynamic:


| Concept | What it does | Analogy |
|---|---|---|
| `variable` | Input to your config — like function parameters | Like filling a form |
| `local` | Computed values inside config — not exposed outside | Like a variable in code |
| `output` | Values printed after apply — like return values | Like a function's return |
| `data` source | Fetches existing info from AWS — doesn't create anything | Like a read-only query |

---

## Challenge Tasks

### Task 1: Extract Variables
Take your Day 62 infrastructure config and refactor it:

1. Create a `variables.tf` file with input variables for:
   - `region` (string, default: your preferred region)
   - `vpc_cidr` (string, default: `"10.0.0.0/16"`)
   - `subnet_cidr` (string, default: `"10.0.1.0/24"`)
   - `instance_type` (string, default: `"t2.micro"`)
   - `project_name` (string, no default -- force the user to provide it)
   - `environment` (string, default: `"dev"`)
   - `allowed_ports` (list of numbers, default: `[22, 80, 443]`)
   - `extra_tags` (map of strings, default: `{}`)

2. Replace every hardcoded value in `main.tf` with `var.<name>` references
3. Run `terraform plan` -- it should prompt you for `project_name` since it has no default

**Document:** What are the five variable types in Terraform? (`string`, `number`, `bool`, `list`, `map`)

---

Concept First — Variable Types

| Type | Example | Use case |
|---|---|---|
| `string` | `"t2.micro"` | Region, AMI ID, instance type |
| `number` | `3` | Port numbers, replica count |
| `bool` | `true` | Enable/disable features |
| `list` | `["22", "80", "443"]` | List of ports, AZs |
| `map` | `{env = "dev", team = "ops"}` | Tags, key-value settings |

Created variables.tf:

<img width="624" height="827" alt="image" src="https://github.com/user-attachments/assets/476209ad-573a-4614-bdb7-e25792f1c2a2" />

using those variables using var. :

<img width="243" height="95" alt="image" src="https://github.com/user-attachments/assets/cffc31c6-6a7a-4940-94c9-b62d1b597399" />

<img width="679" height="564" alt="image" src="https://github.com/user-attachments/assets/88b382c0-69bf-4107-b15a-a2a42fb33d20" />

Since project_name has no default, Terraform will prompt:
var.project_name
  Name of the project — used in all resource tags

  Enter a value: terraweek
  
<img width="1125" height="578" alt="image" src="https://github.com/user-attachments/assets/3071317b-3380-4758-b0e4-abe6593bb67b" />

Plan shows resources with your project_name in the names.

---

### Task 2: Variable Files and Precedence
1. Create `terraform.tfvars`:
```hcl
project_name = "terraweek"
environment  = "dev"
instance_type = "t2.micro"
```

2. Create `prod.tfvars`:
```hcl
project_name = "terraweek"
environment  = "prod"
instance_type = "t3.small"
vpc_cidr     = "10.1.0.0/16"
subnet_cidr  = "10.1.1.0/24"
```

3. Apply with the default file:
```bash
terraform plan                              # Uses terraform.tfvars automatically
```

4. Apply with the prod file:
```bash
terraform plan -var-file="prod.tfvars"      # Uses prod.tfvars
```

5. Override with CLI:
```bash
terraform plan -var="instance_type=t2.nano"  # CLI overrides everything
```

6. Set an environment variable:
```bash
export TF_VAR_environment="staging"
terraform plan                              # env var overrides default but not tfvars
```

**Document:** Write the variable precedence order from lowest to highest priority.

---

Concept First — Variable Precedence (lowest to highest):

| Priority | Method | Example |
|---|---|---|
| 1 (lowest) | Default in variables.tf | `default = "dev"` |
| 2 | `terraform.tfvars` file | Auto-loaded |
| 3 | `*.auto.tfvars` files | Auto-loaded |
| 4 | `-var-file` flag | `terraform plan -var-file="prod.tfvars"` |
| 5 | `-var` flag | `terraform plan -var="instance_type=t2.nano"` |
| 6 (highest) | `TF_VAR_*` env vars | `export TF_VAR_environment="staging"` |

the TF_VAR_* cannot override the tfvars file.

corrected:

1. -var / -var-file CLI
2. terraform.tfvars / *.auto.tfvars
3. TF_VAR_ environment variables
4. default values

Higher number = wins over lower number.

created tfvars:

<img width="686" height="260" alt="image" src="https://github.com/user-attachments/assets/d0aac634-7ea9-4e69-bd62-05ca00b441a5" />

<img width="515" height="255" alt="image" src="https://github.com/user-attachments/assets/5e4c3726-3697-4e8f-abbf-00afefdc1510" />

VAriables got picked automatically from the tfvars file, No prompts! terraform.tfvars was automatically loaded

<img width="602" height="496" alt="image" src="https://github.com/user-attachments/assets/fa9bf5d4-db59-4b91-a248-9cdf7eedc47c" />

After specifying terraform -var-file="prod.tfvars":

<img width="638" height="726" alt="image" src="https://github.com/user-attachments/assets/82ab105e-d4da-4544-826f-61b13b9e5c0e" />

terraform plan -var="instance_type=t2.nano":

<img width="642" height="456" alt="image" src="https://github.com/user-attachments/assets/9721ed4e-c13d-44b9-af24-1dd1b11ea5bd" />

CLI -var overrides everything including tfvars.

Override with environment variable
export TF_VAR_environment="t2.micro"

terraform plan

<img width="664" height="98" alt="image" src="https://github.com/user-attachments/assets/4ede1e2d-be7e-45bb-ad4b-e90944bcef99" />

The instance_type still shows t3.micro since the TF_VAR cannot override the tfvars. To reset:

unset TF_VAR_instance_type

---

### Task 3: Add Outputs
Create an `outputs.tf` file with outputs for:

1. `vpc_id` -- the VPC ID
2. `subnet_id` -- the public subnet ID
3. `instance_id` -- the EC2 instance ID
4. `instance_public_ip` -- the public IP of the EC2 instance
5. `instance_public_dns` -- the public DNS name
6. `security_group_id` -- the security group ID

Apply your config and verify the outputs are printed at the end:
```bash
terraform apply

# After apply, you can also run:
terraform output                          # Show all outputs
terraform output instance_public_ip       # Show a specific output
terraform output -json                    # JSON format for scripting
```

**Verify:** Does `terraform output instance_public_ip` return the correct IP?

---

Concept First:

Outputs are like return values from your Terraform config. After terraform apply runs, all outputs are printed to screen. This is how you get important info like public IPs, resource IDs, DNS names.
They're also used when one Terraform config needs to pass values to another.

created output.tf file:

<img width="667" height="891" alt="image" src="https://github.com/user-attachments/assets/64300c87-a161-45f3-a167-f47bc26a7e7a" />

<img width="727" height="716" alt="image" src="https://github.com/user-attachments/assets/5fb9a252-1bc1-4083-94c3-ec841a33c0b0" />

<img width="429" height="172" alt="image" src="https://github.com/user-attachments/assets/744a1ea4-6eec-431c-807a-a16c246a2360" />

<img width="406" height="69" alt="image" src="https://github.com/user-attachments/assets/fbde9b2b-88bb-4037-a42d-b9e115474433" />

<img width="406" height="754" alt="image" src="https://github.com/user-attachments/assets/388ca0f8-e4ab-49f9-bb17-e29c8ae14704" />

terraform output instance_public_ip returns the correct IP — verify it matches AWS console EC2 public IP.

---

### Task 4: Use Data Sources
Stop hardcoding the AMI ID. Use a data source to fetch it dynamically.

1. Add a `data "aws_ami"` block that:
   - Filters for Amazon Linux 2 images
   - Filters for `hvm` virtualization and `gp2` root device
   - Uses `owners = ["amazon"]`
   - Sets `most_recent = true`

2. Replace the hardcoded AMI in your `aws_instance` with `data.aws_ami.amazon_linux.id`

3. Add a `data "aws_availability_zones"` block to fetch available AZs in your region

4. Use the first AZ in your subnet: `data.aws_availability_zones.available.names[0]`

Apply and verify -- your config now works in any region without changing the AMI.

**Document:** What is the difference between a `resource` and a `data` source?

---

Concept First:

| | resource | data source |
|---|---|---|
| What it does | CREATES something in AWS | READS existing info from AWS |
| Syntax | `resource "aws_instance" "main"` | `data "aws_ami" "amazon_linux"` |
| Changes AWS | Yes | No — read only |
| Example | Create an EC2 | Find the latest Amazon Linux AMI ID |

The problem with hardcoded AMI IDs: ami-0f5ee92e2d63afc18 only works in ap-south-2. In us-east-1 it's completely different. A data source fetches the correct AMI for whatever region you're in — automatically.

Also update the subnet to use the first available AZ

Created data.tf and replaced their usage.

<img width="703" height="625" alt="image" src="https://github.com/user-attachments/assets/bea71bab-c21a-45ad-a4cd-a7341cc0b087" />

<img width="772" height="261" alt="image" src="https://github.com/user-attachments/assets/ba014252-1b40-428b-ab9a-85f32f1b7851" />

<img width="935" height="425" alt="image" src="https://github.com/user-attachments/assets/0f640f26-6a18-4c05-a5a3-9e7b3f528535" />

Applied:

<img width="692" height="412" alt="image" src="https://github.com/user-attachments/assets/cdce0b2a-d4ed-4e8d-a844-2103ee6352ed" />

 The config now works in ANY region without changing the AMI.

 ---

### Task 5: Use Locals for Dynamic Values
1. Add a `locals` block:
```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

2. Replace all Name tags with `local.name_prefix`:
   - VPC: `"${local.name_prefix}-vpc"`
   - Subnet: `"${local.name_prefix}-subnet"`
   - Instance: `"${local.name_prefix}-server"`

3. Merge common tags with resource-specific tags:
```hcl
tags = merge(local.common_tags, {
  Name = "${local.name_prefix}-server"
})
```

Apply and check the tags in the AWS console -- every resource should have consistent tagging.

---

Concept First:

Locals are like variables you compute inside the config — not inputs from outside, not outputs to the user. They help you avoid repeating the same expression in multiple places.

Concept First
Locals are like variables you compute inside the config — not inputs from outside, not outputs to the user. They help you avoid repeating the same expression in multiple places.

# Without locals — repeated 8 times in your config:
Name = "terraweek-dev-vpc"
Name = "terraweek-dev-subnet"
Name = "terraweek-dev-server"

# With locals — defined once, used everywhere:
local.name_prefix = "terraweek-dev"

created locals.tf:

<img width="863" height="313" alt="image" src="https://github.com/user-attachments/assets/f5f46c39-3a1e-4687-8db0-ee91327eb933" />

Changed the tags to use local name prefix :

<img width="765" height="760" alt="image" src="https://github.com/user-attachments/assets/e647a850-c266-4f8a-9aa8-073619916709" />

After applying: saw terraweek-vpc etc...

<img width="750" height="631" alt="image" src="https://github.com/user-attachments/assets/081d040e-f55f-4075-bb2e-6d664d4cdf78" />

Every resource has consistent tags! Change project_name or environment and ALL tags update automatically.

Verified: Every resource in AWS console has Project, Environment, ManagedBy, and Name tags.

---

### Task 6: Built-in Functions and Conditional Expressions
Practice these in `terraform console`:
```bash
terraform console
```

1. **String functions:**
   - `upper("terraweek")` -> `"TERRAWEEK"`
   - `join("-", ["terra", "week", "2026"])` -> `"terra-week-2026"`
   - `format("arn:aws:s3:::%s", "my-bucket")`

2. **Collection functions:**
   - `length(["a", "b", "c"])` -> `3`
   - `lookup({dev = "t2.micro", prod = "t3.small"}, "dev")` -> `"t2.micro"`
   - `toset(["a", "b", "a"])` -> removes duplicates

3. **Networking function:**
   - `cidrsubnet("10.0.0.0/16", 8, 1)` -> `"10.0.1.0/24"`

4. **Conditional expression** -- add this to your config:
```hcl
instance_type = var.environment == "prod" ? "t3.small" : "t2.micro"
```

Apply with `environment = "prod"` and verify the instance type changes.

**Document:** Pick five functions you find most useful and explain what each does.

---

Concept First:

terraform console is an interactive calculator for Terraform expressions. You can test functions without deploying anything.

String functions:

<img width="533" height="409" alt="image" src="https://github.com/user-attachments/assets/8b2efba3-fa94-4867-a886-26eb07552d52" />

Collection functions:

<img width="698" height="445" alt="image" src="https://github.com/user-attachments/assets/05eba59d-fd36-4111-9337-397c8b5f429e" />

Networking function:

<img width="352" height="137" alt="image" src="https://github.com/user-attachments/assets/69cc3344-7b03-4a59-9eb0-9e0f7c743cbc" />

This is powerful — you can auto-generate subnet CIDRs from a VPC CIDR!

cidrsubnet("10.0.0.0/16", 8, 0)

8

Add Additional 8 bits for subnettinge.

So:

/16 + 8 = /24

Output:

10.0.0.0/24

Last number (0, 1, 2)

Which subnet number you want.:

Example:

Subnet 0

cidrsubnet("10.0.0.0/16", 8, 0).
= 10.0.0.0/24

Subnet 1
cidrsubnet("10.0.0.0/16", 8, 1).

HERE : first 16 bits are fixed if /16

<img width="897" height="434" alt="image" src="https://github.com/user-attachments/assets/82b32146-fec8-4e49-8d19-45b0d5ef4ccb" />

This means: "if environment is prod, use t3.small, otherwise use t2.micro"

with default:

<img width="729" height="472" alt="image" src="https://github.com/user-attachments/assets/b307952c-3981-46fd-8282-4122fef13f61" />

with prod:

Shows t3.small since the env is prod:

<img width="728" height="306" alt="image" src="https://github.com/user-attachments/assets/ecbe4303-f60e-4829-8462-54a961a14fc0" />

 5 most useful functions to document:

 | Function | What it does | Example |
|---|---|---|
| `merge()` | Combines two maps — perfect for tags | `merge(common_tags, {Name = "server"})` |
| `lookup()` | Gets value from map with a fallback default | `lookup(var.instance_map, "prod", "t2.micro")` |
| `cidrsubnet()` | Calculates subnet CIDR from parent CIDR | `cidrsubnet("10.0.0.0/16", 8, 1)` → `"10.0.1.0/24"` |
| `join()` | Joins list into string with separator | `join("-", ["app", "dev"])` → `"app-dev"` |
| `toset()` | Removes duplicates from list | `toset(["a","b","a"])` → `["a","b"]` |


---

## Four Key Concepts

| Concept | What it does | Analogy |
|---|---|---|
| `variable` | Input to your config | Function parameters |
| `local` | Computed value inside config | Internal variable |
| `output` | Value printed after apply | Function return value |
| `data` source | Reads existing AWS info | Read-only query |

## Variable Types

| Type | Example | Use case |
|---|---|---|
| `string` | `"t2.micro"` | Region, AMI, instance type |
| `number` | `3` | Port numbers, replica count |
| `bool` | `true` | Enable/disable features |
| `list` | `[22, 80, 443]` | Ports, AZ names |
| `map` | `{env = "dev"}` | Tags, key-value config |

## Variable Precedence (lowest to highest)

| Priority | Method |
|---|---|
| 1 (lowest) | Default value in `variables.tf` |
| 2 | `terraform.tfvars` (auto-loaded) |
| 3 | `*.auto.tfvars` (auto-loaded) |
| 4 | `-var-file="prod.tfvars"` flag |
| 5 | `-var="key=value"` flag |
| 6 (highest) | `TF_VAR_*` environment variables |

## resource vs data source

| | resource | data source |
|---|---|---|
| Creates AWS resources | ✅ Yes | ❌ No |
| Read-only | ❌ No | ✅ Yes |
| Syntax | `resource "aws_instance" "main"` | `data "aws_ami" "linux"` |
| Use case | Create EC2 | Find latest AMI ID |

## 5 Most Useful Functions

| Function | What it does | Example |
|---|---|---|
| `merge()` | Combines two maps | `merge(common_tags, {Name = "server"})` |
| `lookup()` | Gets value from map with fallback | `lookup(map, "prod", "t2.micro")` |
| `cidrsubnet()` | Calculates subnet from parent CIDR | `cidrsubnet("10.0.0.0/16", 8, 1)` |
| `join()` | Joins list into string | `join("-", ["app", "dev"])` |
| `toset()` | Removes duplicates from list | `toset(["a","b","a"])` |

## Conditional Expression
Syntax: `condition ? value_if_true : value_if_false`
Example: `var.environment == "prod" ? "t3.small" : "t2.micro"`


Summary:

| Concept | What it means |
|---|---|
| `variable` block | Input to your config — like a function parameter |
| `var.name` | Reference to a variable value |
| `terraform.tfvars` | Auto-loaded variable file — put your defaults here |
| `prod.tfvars` | Environment-specific file — loaded with `-var-file` flag |
| `TF_VAR_name` | Environment variable that sets a Terraform variable |
| `output` block | Value printed after apply — use for IPs, IDs, DNS names |
| `terraform output` | Shows all outputs after apply |
| `data` source | Reads existing AWS info — never creates anything |
| `data.aws_ami` | Fetches latest AMI — no more hardcoded AMI IDs |
| `locals` block | Computed values used inside config — DRY principle |
| `local.name` | Reference to a local value |
| `merge()` | Combines two maps — perfect for tags |
| `cidrsubnet()` | Calculates subnet CIDR from parent |
| Conditional `? :` | `condition ? true_value : false_value` |
| `terraform console` | Interactive REPL for testing expressions |
