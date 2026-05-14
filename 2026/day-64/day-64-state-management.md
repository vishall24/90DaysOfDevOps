# Day 64 -- Terraform State Management and Remote Backends

## Task
The state file is the single most important thing in Terraform. It is the source of truth -- the map between your `.tf` files and what actually exists in the cloud. Lose it and Terraform forgets everything. Corrupt it and your next apply could destroy production.

Today you learn to manage state like a professional -- remote backends, locking, importing existing resources, and handling drift.

---

What Are We Learning Today?
The state file (terraform.tfstate) is Terraform's memory. It knows:

What resources exist in AWS
What their IDs, IPs, and attributes are
What YOU defined vs what actually EXISTS

The problem with local state:
Your laptop crashes → state file gone → Terraform forgets everything
Teammate runs apply → two state files → conflict and corruption
No history → can't recover previous state

Today's solution — Remote State:

| Problem | Solution |
|---|---|
| State on laptop = lost if laptop dies | Store state in S3 bucket |
| Two people apply at same time = corruption | DynamoDB locking — only one person at a time |
| No recovery if state corrupted | S3 versioning — roll back to previous state |
| Resource exists but not in Terraform | terraform import |
| Someone changed AWS manually | Detect and fix drift |

---

## Challenge Tasks

### Task 1: Inspect Your Current State
Use your Day 63 config (or create a small config with a VPC and EC2 instance). Apply it and then explore the state:

```bash
terraform show                                    # Full state in human-readable format
terraform state list                              # All resources tracked by Terraform
terraform state show aws_instance.<name>          # Every attribute of the instance
terraform state show aws_vpc.<name>               # Every attribute of the VPC
```

Answer:
1. How many resources does Terraform track?
2. What attributes does the state store for an EC2 instance? (hint: way more than what you defined)
3. Open `terraform.tfstate` in an editor -- find the `serial` number. What does it represent?

---

Concept First
The state file stores WAY more information than what you defined. You defined instance_type = "t2.micro" but the state stores 50+ attributes — public IP, private IP, VPC ID, subnet ID, ARN, launch time, everything AWS returned.
The serial number in the state file increments every time the state changes. It's like a version counter.


<img width="696" height="1026" alt="image" src="https://github.com/user-attachments/assets/66ae70a1-26cd-4057-8928-0dad0524c0b0" />

terraform show: (shows all in human readable)

<img width="896" height="865" alt="image" src="https://github.com/user-attachments/assets/8fc2b62c-5683-4d21-8bb3-b60ac87d50df" />

<img width="640" height="99" alt="image" src="https://github.com/user-attachments/assets/78d177ef-c19d-4e55-b4ed-b2bb71375f55" />

show state for particular resource:

<img width="889" height="541" alt="image" src="https://github.com/user-attachments/assets/575e1d8f-842f-4b27-be06-32f253e569ab" />

<img width="786" height="429" alt="image" src="https://github.com/user-attachments/assets/739c943b-f028-4fcf-878a-4739a9b5c749" />

The serial number increments every time the state changes. If two people have state files with different serials, Terraform knows there's a conflict.

Answers:

Terraform tracks 2 resources (VPC + EC2)
EC2 state stores 50+ attributes — way more than you defined
serial = version counter, increments with every state change

Concept First:
 
LOCAL STATE (dangerous):          REMOTE STATE (production-ready):
terraform.tfstate                 S3 Bucket
  ↓                                 ↓
On your laptop                    In the cloud
No versioning                     Full version history
No locking                        DynamoDB locking
Lost if deleted                   Never lost

---

### Task 2: Set Up S3 Remote Backend
Storing state locally is dangerous -- one deleted file and you lose everything. Time to move it to S3.

1. First, create the backend infrastructure (do this manually or in a separate Terraform config):
```bash
# Create S3 bucket for state storage
aws s3api create-bucket \
  --bucket terraweek-state-<yourname> \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1

# Enable versioning (so you can recover previous state)
aws s3api put-bucket-versioning \
  --bucket terraweek-state-<yourname> \
  --versioning-configuration Status=Enabled

# Create DynamoDB table for state locking
aws dynamodb create-table \
  --table-name terraweek-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-south-1
```

2. Add the backend block to your Terraform config:
```hcl
terraform {
  backend "s3" {
    bucket         = "terraweek-state-<yourname>"
    key            = "dev/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraweek-state-lock"
    encrypt        = true
  }
}
```

3. Run:
```bash
terraform init
```
Terraform will ask: "Do you want to copy existing state to the new backend?" -- say yes.

4. Verify:
   - Check the S3 bucket -- you should see `dev/terraform.tfstate`
   - Your local `terraform.tfstate` should now be empty or gone
   - Run `terraform plan` -- it should show no changes (state migrated correctly)

---

We need to create TWO AWS resources BEFORE adding the backend config:

S3 bucket — stores the state file
DynamoDB table — handles locking (prevents concurrent applies)

Created s3 bucket and versioning and dynamoDB as well:

<img width="1576" height="884" alt="image" src="https://github.com/user-attachments/assets/98900322-1066-4029-8b66-50a818a29f25" />

<img width="782" height="724" alt="image" src="https://github.com/user-attachments/assets/027534a1-4858-49e1-91e3-e9b2f907785a" />

<img width="697" height="146" alt="image" src="https://github.com/user-attachments/assets/ea8e184e-8bbb-493f-b5ce-6ed121b356ee" />

added this block insode terraform{}:

      backend "s3" {
        bucket         = "terraweek-state-warrior-v2"    # Your bucket name
        key            = "dev/terraform.tfstate"      # Path inside the bucket
        region         = "ap-south-2"
        dynamodb_table = "terraweek-state-lock"       # Your DynamoDB table
        encrypt        = true                          # Encrypt state at rest
      }
    }

Terraform init:

<img width="713" height="726" alt="image" src="https://github.com/user-attachments/assets/7576229c-ae66-48a0-9691-73266bfb2139" />

Type yes. Your local state is now copied to S3!
Output:
Successfully configured the backend "s3"!
Terraform will automatically use this backend unless the backend
configuration changes.

confirmed the file:

<img width="550" height="97" alt="image" src="https://github.com/user-attachments/assets/7ebb2d60-7030-49bc-aabf-320562e6164b" />

Your state is now in the cloud! 

the tfstate file shows nothing:

<img width="352" height="56" alt="image" src="https://github.com/user-attachments/assets/76504897-e4c0-4171-8b28-e755c9057e09" />

<img width="1027" height="290" alt="image" src="https://github.com/user-attachments/assets/f84aba3b-467f-468a-af75-8e74387eff71" />

---

### Task 3: Test State Locking
State locking prevents two people from running `terraform apply` at the same time and corrupting the state.

1. Open **two terminals** in the same project directory
2. In Terminal 1, run:
```bash
terraform apply
```
3. While Terminal 1 is waiting for confirmation, in Terminal 2 run:
```bash
terraform plan
```
4. Terminal 2 should show a **lock error** with a Lock ID

**Document:** What is the error message? Why is locking critical for team environments?

5. After the test, if you get stuck with a stale lock:
```bash
terraform force-unlock <LOCK_ID>
```

---

Concept First:

Without locking, this can happen:

Person A runs terraform apply at 2pm — starts modifying EC2
Person B runs terraform apply at 2pm — also starts modifying EC2
Both finish — state file is now corrupted with conflicting data

DynamoDB locking prevents this. When Person A starts, Terraform writes a lock record to DynamoDB. Person B tries to apply and sees the lock — they have to wait.

---

<img width="1915" height="1099" alt="image" src="https://github.com/user-attachments/assets/90bd91aa-dbc7-43dd-8b74-595595b3bc51" />

This is the lock error! Terminal 2 can't proceed because Terminal 1 holds the lock.

Cancel Terminal 1 (Ctrl+C) and release the lock
The lock is automatically released when Terminal 1 exits.

If you ever get a stale lock (rare but happens):
Get the Lock ID from the error message:

    terraform force-unlock <LOCK_ID>
  
Warning: Only use this if you're 100% sure no other apply is running — otherwise you risk corruption!
### Why is locking critical?
Without locking, two simultaneous applies could corrupt the state file, leaving your infrastructure in an unknown state that's very difficult to recover from.

---

### Task 4: Import an Existing Resource
Not everything starts with Terraform. Sometimes resources already exist in AWS and you need to bring them under Terraform management.

1. Manually create an S3 bucket in the AWS console -- name it `terraweek-import-test-<yourname>`
2. Write a `resource "aws_s3_bucket"` block in your config for this bucket (just the bucket name, nothing else)
3. Import it:
```bash
terraform import aws_s3_bucket.imported terraweek-import-test-<yourname>
```
4. Run `terraform plan`:
   - If you see "No changes" -- the import was perfect
   - If you see changes -- your config does not match reality. Update your config to match, then plan again until you get "No changes"

5. Run `terraform state list` -- the imported bucket should now appear alongside your other resources

**Document:** What is the difference between `terraform import` and creating a resource from scratch?

---

Concept First:

| | terraform import | creating from scratch |
|---|---|---|
| Resource in AWS | Already exists | Doesn't exist yet |
| What Terraform does | Reads existing resource into state | Creates new resource in AWS |
| Risk | None — just updates state | Creates a new resource |
| Use case | Adopt resources created manually | New infrastructure |

Real world scenario: Your team manually created an S3 bucket in the AWS console 6 months ago. Now you want to manage it with Terraform. You can't just write the resource block — Terraform would try to create a SECOND bucket. You need terraform import.
Step 1: Manually create an S3 bucket in AWS console
Go to AWS Console → S3 → Create bucket

Name: terraweek-import-test-warrior
Region: ap-south-2
Leave all other settings default
Click Create bucket

created bucket in AWS console:

<img width="1437" height="266" alt="image" src="https://github.com/user-attachments/assets/51ca5865-ce6a-40fa-bbe5-014777c1af5d" />

imported:

<img width="743" height="921" alt="image" src="https://github.com/user-attachments/assets/d0243a0a-6228-4036-8ca0-95ccbd02e023" />

after importing did terraform plan:

<img width="1053" height="290" alt="image" src="https://github.com/user-attachments/assets/cb670ca8-5c24-4242-b3d3-77ae07e87f2b" />

<img width="292" height="100" alt="image" src="https://github.com/user-attachments/assets/d2fd65c4-553e-4591-9c35-f3723add894d" />

The imported bucket appears in state alongside resources Terraform created.

---

### Task 5: State Surgery -- mv and rm
Sometimes you need to rename a resource or remove it from state without destroying it in AWS.

1. **Rename a resource in state:**
```bash
terraform state list                              # Note the current resource names
terraform state mv aws_s3_bucket.imported aws_s3_bucket.logs_bucket
```
Update your `.tf` file to match the new name. Run `terraform plan` -- it should show no changes.

2. **Remove a resource from state (without destroying it):**
```bash
terraform state rm aws_s3_bucket.logs_bucket
```
Run `terraform plan` -- Terraform no longer knows about the bucket, but it still exists in AWS.

3. **Re-import it** to bring it back:
```bash
terraform import aws_s3_bucket.logs_bucket terraweek-import-test-<yourname>
```

**Document:** When would you use `state mv` in a real project? When would you use `state rm`?

---

Concept First:

| Command | What it does | When to use |
|---|---|---|
| `terraform state mv` | Renames resource in state | When you rename a resource in .tf files |
| `terraform state rm` | Removes resource from state | When you want Terraform to forget a resource without deleting it |

Real world uses:

state mv — You renamed aws_s3_bucket.imported to aws_s3_bucket.logs_bucket in your code
state rm — You want to hand off a resource to another team's Terraform config

Change:

    resource "aws_s3_bucket" "imported" {    # OLD

To:

    resource "aws_s3_bucket" "logs_bucket" {    # NEW — matches state

    terraform plan

Output:

No changes. Your infrastructure matches the configuration.

The rename worked — Terraform knows logs_bucket is the same bucket as imported was.

<img width="1100" height="637" alt="image" src="https://github.com/user-attachments/assets/79d64ee3-aa4a-45a5-b741-032628df4fe6" />

Plan: 1 to add, 0 to change, 0 to destroy.
  + aws_s3_bucket.logs_bucket    ← Terraform forgot it exists!
But the bucket is still in AWS — Terraform just forgot about it.

<img width="1046" height="594" alt="image" src="https://github.com/user-attachments/assets/ef94b43e-0cbb-4601-a77e-b77afd32f89f" />

Document:

Use state mv when you rename a resource in your .tf files — prevents Terraform from destroying and recreating it
Use state rm when you want to hand off a resource to another team or another Terraform workspace

---

### Task 6: Simulate and Fix State Drift
State drift happens when someone changes infrastructure outside of Terraform -- through the AWS console, CLI, or another tool.

1. Apply your full config so everything is in sync
2. Go to the **AWS console** and manually:
   - Change the Name tag of your EC2 instance to `"ManuallyChanged"`
   - Change the instance type if it's stopped (or add a new tag)
3. Run:
```bash
terraform plan
```
You should see a **diff** -- Terraform detects that reality no longer matches the desired state.

4. You have two choices:
   - **Option A:** Run `terraform apply` to force reality back to match your config (reconcile)
   - **Option B:** Update your `.tf` files to match the manual change (accept the drift)

5. Choose Option A -- apply and verify the tags are restored.

6. Run `terraform plan` again -- it should show "No changes." Drift resolved.

**Document:** How do teams prevent state drift in production? (hint: restrict console access, use CI/CD for all changes)

---

Concept First
Drift = when the real AWS infrastructure no longer matches what Terraform's state file says it should be.
Common causes:

Someone clicks in the AWS console and changes a setting
An automated script modifies a resource
AWS auto-modifies something (like security group rules)

Terraform detects drift on the next terraform plan by comparing:

State file (what Terraform last knew)
Real AWS resources (what actually exists right now)

Go to AWS Console and manually change something
EC2 Instance:

Go to EC2 → Instances → select terraweek-server
Actions → Instance settings → Manage tags
Change the Name tag from terraweek-server to ManuallyChanged
Save

Changed:

<img width="1466" height="372" alt="image" src="https://github.com/user-attachments/assets/548d8515-2ad6-431f-a6ea-ba652ae4bede" />

Terraform plan:

<img width="1207" height="587" alt="image" src="https://github.com/user-attachments/assets/732d2053-f0b2-4ae9-8352-774e538c1e18" />

Terraform detected the drift! It shows the manual change and wants to revert it back to what the .tf file says.

Step 4: Option A — Reconcile (revert manual change):

    terraform apply

Type yes. Terraform reverts the tag back to terraweek-server.

Step 5: Verify drift is resolved
    
    terraform plan

Output:
No changes. Your infrastructure matches the configuration.
Drift resolved ✅

Step 6: Use refresh-only to just update state without changing anything

#Just refresh state to match reality — don't make any changes

    terraform apply -refresh-only

This is useful when you WANT to accept the manual change instead of reverting it.

Document: How teams prevent drift in production:

Restrict AWS console access — only Terraform can make changes (IAM policies)
Use CI/CD pipelines for all infrastructure changes
Run terraform plan on a schedule to detect drift early
Use Terraform Cloud/Enterprise which has drift detection built in


Summary:

## Local State vs Remote State

LOCAL STATE                        REMOTE STATE
terraform.tfstate                  S3 Bucket (terraweek-state-vishal)
↓                                  ↓
On your laptop                     In AWS — never lost
No versioning                      Full version history (S3 versioning)
No locking                         DynamoDB locking
One person only                    Safe for teams

## Why Remote State?

| Problem | Solution |
|---|---|
| Laptop dies = state lost | S3 stores state in cloud |
| Two applies at once = corruption | DynamoDB locking |
| State corrupted = no recovery | S3 versioning = rollback |
| Team can't collaborate | Everyone points to same S3 bucket |

## Backend Configuration
```hcl
backend "s3" {
  bucket         = "terraweek-state-vishal"
  key            = "dev/terraform.tfstate"
  region         = "ap-south-1"
  dynamodb_table = "terraweek-state-lock"
  encrypt        = true
}
```

## State Commands

| Command | What it does |
|---|---|
| `terraform show` | Human-readable view of all state |
| `terraform state list` | List all tracked resources |
| `terraform state show <resource>` | Deep dive into one resource |
| `terraform state mv` | Rename resource in state |
| `terraform state rm` | Remove resource from state (without deleting from AWS) |
| `terraform import` | Bring existing AWS resource into state |
| `terraform force-unlock` | Release a stale lock (use with caution!) |
| `terraform apply -refresh-only` | Update state to match reality without making changes |

## terraform import vs Creating from Scratch

| | terraform import | Creating from scratch |
|---|---|---|
| Resource in AWS | Already exists | Doesn't exist yet |
| What Terraform does | Reads into state | Creates new resource |
| Risk | None | Creates duplicate |

## When to Use Each State Command

| Command | Real-world use case |
|---|---|
| `state mv` | Renamed resource in .tf file — prevents destroy/recreate |
| `state rm` | Hand off resource to another team's Terraform config |
| `import` | Adopt manually created AWS resources |
| `force-unlock` | Stale lock after a crashed apply |

## State Drift
Drift = real AWS infrastructure no longer matches Terraform state.
Causes: manual console changes, scripts, AWS auto-modifications.
Detection: terraform plan shows a diff.
Fix: terraform apply (revert) or terraform apply -refresh-only (accept).

## Preventing Drift in Production
- Restrict AWS console access via IAM — only Terraform makes changes
- Use CI/CD pipelines for all infrastructure
- Schedule regular terraform plan runs to catch drift early
- Use Terraform Cloud drift detection feature



| Concept | What it means |
|---|---|
| State file | Terraform's memory — maps .tf files to real AWS resources |
| `serial` number | Version counter in state — increments with every change |
| Remote backend | Store state in S3 instead of locally — safe for teams |
| S3 backend | Stores the actual state file — enables shared state |
| DynamoDB locking | Prevents two applies at once — locks the state file |
| S3 versioning | Recover previous state if current state gets corrupted |
| `terraform import` | Bring existing AWS resource under Terraform management |
| `state mv` | Rename a resource in state — use when you rename in .tf |
| `state rm` | Remove resource from state without deleting from AWS |
| `force-unlock` | Release a stale lock — use with extreme caution |
| State drift | Real AWS no longer matches Terraform state |
| `-refresh-only` | Update state to match reality without making any changes |
| `encrypt = true` | Encrypt state file at rest in S3 |
