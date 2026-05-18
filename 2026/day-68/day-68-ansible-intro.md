# Day 68 -- Introduction to Ansible and Inventory Setup

## Task
Terraform provisions infrastructure. But who installs packages, configures services, manages users, and keeps servers in the desired state after they exist? That is the job of a configuration management tool, and Ansible is the industry standard.

Today you install Ansible, set up an inventory of servers, and run your first ad-hoc commands -- all without installing a single agent on the target machines. Ansible is agentless. SSH is all it needs.

---

What Are We Learning Today?
Remember this gap:
Terraform creates the server
        ↓
Server is running... but EMPTY
        ↓
Who installs Nginx? Who creates users?
Who configures the firewall? Who keeps
everything consistent across 100 servers?
        ↓
ANSIBLE

The difference:

| Tool | Job | Analogy |
|---|---|---|
| Terraform | Creates servers | Building a house |
| Ansible | Configures servers | Furnishing and maintaining the house |

Why Ansible over Chef/Puppet/Salt?

| Tool | Agent needed? | Language | Learning curve |
|---|---|---|---|
| Ansible | ❌ No (agentless!) | YAML | Easy |
| Chef | ✅ Yes | Ruby | Hard |
| Puppet | ✅ Yes | Puppet DSL | Hard |
| Salt | ✅ Yes | YAML/Python | Medium |

Ansible wins because it's agentless — it uses SSH to connect. No software to install on managed servers. Just SSH access and Python (already on most Linux servers).

Ansible Architecture:

Your Laptop (Control Node)
  └── Ansible installed here
        ↓ SSH
  ┌─────────────────────────┐
  │  Inventory (who to talk to)
  │  Modules (what to do)
  │  Playbooks (how to do it)
  └─────────────────────────┘
        ↓ SSH            ↓ SSH            ↓ SSH
  EC2 Instance 1    EC2 Instance 2    EC2 Instance 3
  (web server)      (app server)      (db server)
  No Ansible        No Ansible        No Ansible
  needed here!      needed here!      needed here!

---

## Challenge Tasks

### Task 1: Understand Ansible
Research and write short notes on:

1. What is configuration management? Why do we need it?
2. How is Ansible different from Chef, Puppet, and Salt?
3. What does "agentless" mean? How does Ansible connect to managed nodes?
4. Draw or describe the Ansible architecture:
   - **Control Node** -- the machine where Ansible runs (your laptop or a jump server)
   - **Managed Nodes** -- the servers Ansible configures (your EC2 instances)
   - **Inventory** -- the list of managed nodes
   - **Modules** -- units of work Ansible executes (install a package, copy a file, start a service)
   - **Playbooks** -- YAML files that define what to do on which hosts

---

What is configuration management?

Configuration management means keeping all your servers in a consistent, known state automatically. Without it, you SSH into each server manually, run commands, make mistakes, and over time servers drift apart — "it works on server 1 but not server 2." Configuration management tools like Ansible fix this by defining the desired state in code.

What does agentless mean?

Ansible doesn't need any software installed on the servers it manages. It connects via SSH (same as you do manually) and uses Python (pre-installed on most Linux systems). Compare this to Chef/Puppet which require an agent process running permanently on every server — more overhead, more things to maintain.

## What is Ansible?

Ansible is an agentless configuration management tool. It uses SSH to
connect to managed nodes and run tasks defined in YAML files called playbooks.
No software needed on managed servers — just SSH and Python.

## Ansible Architecture:

    Control Node (your laptop)
    └── Ansible installed
    ↓ SSH only
    ┌──────────────────────────────┐
    │  Inventory  → who to manage  │
    │  Modules    → what to do     │
    │  Playbooks  → how to do it   │
    └──────────────────────────────┘
    ↓           ↓           ↓
    web-server   app-server   db-server
    (no Ansible) (no Ansible) (no Ansible)

---

### Task 2: Set Up Your Lab Environment
You need 2-3 EC2 instances to practice on. Choose one approach:

**Option A: Use Terraform (recommended -- you just learned this)**
Use your TerraWeek skills to provision 3 EC2 instances with:
- Amazon Linux 2 or Ubuntu 22.04
- `t2.micro` instance type
- A security group allowing SSH (port 22)
- A key pair for SSH access

**Option B: Launch manually from AWS Console**
Create 3 instances with the same specs above.

Label them mentally:
- **Instance 1:** web server
- **Instance 2:** app server
- **Instance 3:** db server

Verify you can SSH into each one from your control node:
```bash
ssh -i ~/your-key.pem ec2-user@<public-ip-1>
ssh -i ~/your-key.pem ec2-user@<public-ip-2>
ssh -i ~/your-key.pem ec2-user@<public-ip-3>
```

---

Recommendation — Use Terraform (You Just Learned It!)
Since just completed TerraWeek, this is the perfect opportunity to use it. I'll create 3 EC2 instances automatically.

terraform apply:

<img width="614" height="280" alt="image" src="https://github.com/user-attachments/assets/73f917db-a826-4300-904a-97755b995eb3" />

able to ssh using keys:

<img width="651" height="266" alt="image" src="https://github.com/user-attachments/assets/0afcf3fc-cab4-4b6d-b659-2a52a859bc75" />


---

### Task 3: Install Ansible
Install Ansible on your **control node** (your laptop or one dedicated EC2 instance):

```bash
# macOS
brew install ansible

# Ubuntu/Debian
sudo apt update
sudo apt install ansible -y

# Amazon Linux / RHEL
sudo yum install ansible -y
# or
pip3 install ansible

# Verify
ansible --version
```

Confirm the output shows the Ansible version, config file path, and Python version.

**Document:** On which machine did you install Ansible? Why is it only needed on the control node?

---

Key Point
Ansible is ONLY installed on your control node (your laptop or one dedicated machine). The managed nodes (EC2 instances) need NOTHING installed — that's the whole point of agentless.

installed ansible:

<img width="1066" height="325" alt="image" src="https://github.com/user-attachments/assets/e1a8a1b1-0305-4e1a-b315-c321fe03438e" />

Ansible only needs to be on the control node because it connects to managed nodes via SSH and runs modules remotely. The managed nodes just need SSH access and Python — both already present on Amazon Linux.

---

### Task 4: Create Your Inventory File
The inventory tells Ansible which servers to manage. Create a project directory and your first inventory:

```bash
mkdir ansible-practice && cd ansible-practice
```

Create a file called `inventory.ini`:
```ini
[web]
web-server ansible_host=<PUBLIC_IP_1>

[app]
app-server ansible_host=<PUBLIC_IP_2>

[db]
db-server ansible_host=<PUBLIC_IP_3>

[all:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/your-key.pem
```

Verify Ansible can reach all hosts:
```bash
ansible all -i inventory.ini -m ping
```

You should see green `SUCCESS` with `"ping": "pong"` for each host.

**Troubleshoot:** If ping fails:
- Check the SSH key path and permissions (`chmod 400 your-key.pem`)
- Check the security group allows SSH from your IP
- Check the `ansible_user` matches your AMI (ec2-user for Amazon Linux, ubuntu for Ubuntu)

---

Concept First:

The inventory file is like a phone book for Ansible. It tells Ansible:

Which servers exist
How to connect to them
How they're grouped

<img width="452" height="374" alt="image" src="https://github.com/user-attachments/assets/1dd6d985-d00b-4984-993c-aeb5a36417e7" />

<img width="632" height="187" alt="image" src="https://github.com/user-attachments/assets/3ef6b055-0ef5-4816-bfbe-6087d995b4d6" />

If ping fails, debug in this order:

  Check key permissions: chmod 400 ~/.ssh/ansible-lab
  Try SSH manually: ssh -i ~/.ssh/ansible-lab ec2-user@<IP>
  Check the security group allows port 22 from your IP
  Make sure ansible_user=ec2-user (Amazon Linux) or ubuntu (Ubuntu AMI)

So you don't have to type -i inventory.ini on every command:

<img width="511" height="191" alt="image" src="https://github.com/user-attachments/assets/546d45c7-c143-4d6b-af16-0b2885d601eb" />

Same green output, no -i inventory.ini needed.

---

### Task 5: Run Ad-Hoc Commands
Ad-hoc commands let you run quick one-off tasks without writing a playbook.

1. **Check uptime on all servers:**
```bash
ansible all -i inventory.ini -m command -a "uptime"
```

2. **Check free memory on web servers only:**
```bash
ansible web -i inventory.ini -m command -a "free -h"
```

3. **Check disk space on all servers:**
```bash
ansible all -i inventory.ini -m command -a "df -h"
```

4. **Install a package on the web group:**
```bash
ansible web -i inventory.ini -m yum -a "name=git state=present" --become
```
(Use `apt` instead of `yum` if running Ubuntu)

5. **Copy a file to all servers:**
```bash
echo "Hello from Ansible" > hello.txt
ansible all -i inventory.ini -m copy -a "src=hello.txt dest=/tmp/hello.txt"
```

6. **Verify the file was copied:**
```bash
ansible all -i inventory.ini -m command -a "cat /tmp/hello.txt"
```

**Document:** What does `--become` do? When do you need it?

---

This is the hands-on part. Run each command and note the output.

Check uptime on all servers:

<img width="611" height="105" alt="image" src="https://github.com/user-attachments/assets/179717ff-da9f-4766-808f-45b730bbdfd6" />

<img width="1243" height="701" alt="image" src="https://github.com/user-attachments/assets/e2e77836-17fd-444f-9719-583545d42e9d" />

--become means "escalate to sudo/root." You need this for anything that requires admin privileges — installing packages, managing services, writing to system directories.

<img width="759" height="618" alt="image" src="https://github.com/user-attachments/assets/65b0073c-270d-4f56-b482-40c4fa3d735b" />

<img width="698" height="112" alt="image" src="https://github.com/user-attachments/assets/b62a573b-e11f-4bfc-b51f-d8ce7fd663bf" />

---

### Task 6: Explore Inventory Groups and Patterns
1. **Create a group of groups** -- add this to your `inventory.ini`:
```ini
[application:children]
web
app

[all_servers:children]
application
db
```

2. Run commands against different groups:
```bash
ansible application -i inventory.ini -m ping     # web + app servers
ansible db -i inventory.ini -m ping               # only db server
ansible all_servers -i inventory.ini -m ping      # everything
```

3. **Use patterns:**
```bash
ansible 'web:app' -i inventory.ini -m ping        # OR: web or app
ansible 'all:!db' -i inventory.ini -m ping        # NOT: all except db
```

4. **Create an `ansible.cfg`** to avoid typing `-i inventory.ini` every time:
```ini
[defaults]
inventory = inventory.ini
host_key_checking = False
remote_user = ec2-user
private_key_file = ~/your-key.pem
```

Now you can simply run:
```bash
ansible all -m ping
```

**Verify:** Does `ansible all -m ping` work without specifying the inventory file?

---

<img width="433" height="478" alt="image" src="https://github.com/user-attachments/assets/03df9960-7163-46ad-a34d-a72efc80d344" />

<img width="685" height="386" alt="image" src="https://github.com/user-attachments/assets/ddbf3642-f285-4888-aaba-fae2c60f24ee" />

<img width="545" height="130" alt="image" src="https://github.com/user-attachments/assets/08201eb5-3e80-4696-88bf-a1582d496c76" />

These patterns are powerful when you have 50 servers and only want to target a specific subset.

---

## Ansible Architecture

- **Control Node**: My laptop — only machine with Ansible installed
- **Managed Nodes**: 3 EC2 instances — no Ansible installed, just SSH
- **Inventory**: inventory.ini — lists all servers with IPs and SSH config
- **Modules**: Pre-built actions (yum, copy, command, ping)
- **Playbooks**: YAML files defining what to run where (tomorrow's topic)

## Lab Setup

Provisioned 3 EC2 instances using Terraform (t2.micro, Amazon Linux 2):
- web-server: <IP_1>
- app-server: <IP_2>  
- db-server: <IP_3>

## inventory.ini

[paste your file here, redact IPs if sharing publicly]

## Ad-Hoc Commands Run

1. `ansible all -m ping` — connectivity check, all returned pong
2. `ansible all -m command -a "uptime"` — uptime on all 3 servers
3. `ansible web -m command -a "free -h"` — memory on web server only
4. `ansible web -m yum -a "name=git state=present" --become` — installed git
5. `ansible all -m copy -a "src=hello.txt dest=/tmp/hello.txt"` — file copy

## command vs shell module

- `command`: runs a single command, no shell features (no pipes, no redirects)
  - Safer, preferred for simple commands
- `shell`: runs through /bin/sh, supports pipes and redirects
  - Use when you need: `cat file | grep something` or `echo "x" >> file`

## What --become does

Escalates privileges to root (like sudo). Required for:
- Installing/removing packages
- Managing system services
- Writing to protected directories (/etc, /var, etc.)

| Task | What you learned |
|---|---|
| Installed Ansible | Control node only — agentless means no install on managed nodes |
| Created inventory.ini | Groups (web, app, db), group of groups, SSH vars |
| ansible.cfg | Defaults so you don't repeat flags every command |
| Ping module | Tests SSH connectivity — first thing to run always |
| Ad-hoc commands | Quick one-off tasks without a playbook |
| --become | Privilege escalation for admin tasks |
| Patterns | Target specific host subsets with :, :!, wildcards |

