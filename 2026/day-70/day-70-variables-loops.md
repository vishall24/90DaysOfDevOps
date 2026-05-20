# Day 70 -- Variables, Facts, Conditionals and Loops

## Task
Your playbooks work, but they are static -- same packages, same config, same behavior on every server. Real infrastructure is not like that. Web servers need Nginx, app servers need Node.js, production gets more memory than dev. Today you make your playbooks smart.

Variables, facts, conditionals, and loops turn a rigid script into flexible automation that adapts to each host, each group, and each environment.

---

The Big Picture: Why Day 70 Matters:
In Day 69 you wrote playbooks that were static — same behavior every time, every server.
Day 70 makes them dynamic. 

Think of it this way:

Day 69 playbook: "Install Nginx on the web server" — hardcoded, rigid
Day 70 playbook: "Install the right packages for each server type, skip tasks that don't apply, do it for 10 users at once" — intelligent, flexible

Here's a visual overview of how all four concepts connect:

<img width="1440" height="840" alt="image" src="https://github.com/user-attachments/assets/757d1a7d-21d4-4a10-914e-8626855dafb6" />

---

## Challenge Tasks

### Task 1: Variables in Playbooks
Create `variables-demo.yml`:

```yaml
---
- name: Variable demo
  hosts: all
  become: true

  vars:
    app_name: terraweek-app
    app_port: 8080
    app_dir: "/opt/{{ app_name }}"
    packages:
      - git
      - curl
      - wget

  tasks:
    - name: Print app details
      debug:
        msg: "Deploying {{ app_name }} on port {{ app_port }} to {{ app_dir }}"

    - name: Create application directory
      file:
        path: "{{ app_dir }}"
        state: directory
        mode: '0755'

    - name: Install required packages
      yum:
        name: "{{ packages }}"
        state: present
```

Run it and verify the variables resolve correctly.

Now, override a variable from the command line:
```bash
ansible-playbook variables-demo.yml -e "app_name=my-custom-app app_port=9090"
```

**Verify:** Does the CLI variable override the playbook variable?

---

Variables in Ansible work exactly like variables in any programming language — they store a value you reference later with {{ }}.
Why use variables?:

Without them, if your app name changes from terraweek-app to myapp, you'd have to find and replace it in 20 places. With a variable, you change it once at the top.
The {{ }} Jinja2 syntax is how Ansible substitutes variables:

    app_dir: "/opt/{{ app_name }}"
    # becomes: /opt/terraweek-app at runtime

The most important concept today — Variable Precedence:Just like how a manager's direct instruction overrides a company-wide policy, variables closer to the host override variables set globally. 

The order from weakest to strongest:
       
        group_vars/all  →  group_vars/<group>  →  host_vars/<host>  →  playbook vars  →  -e flag


vim variables-demo:

<img width="570" height="428" alt="image" src="https://github.com/user-attachments/assets/feece938-8a00-4a2a-8a14-d552e3031d04" />

after running the ansible playbook:

Watch the debug output — you'll see the variables resolved:

<img width="882" height="483" alt="image" src="https://github.com/user-attachments/assets/a29ac7fe-7362-493b-a003-21a08411a492" />

Override a variable from the command line:

<img width="943" height="495" alt="image" src="https://github.com/user-attachments/assets/95d9e190-fd7d-4934-937c-0a25a2da59a3" />

-e overrides everything — even app_dir updated automatically because it references {{ app_name }}.
Verified: CLI variable completely overrode the playbook variable.

---

### Task 2: group_vars and host_vars
Variables should not live inside playbooks. Move them to dedicated files.

Create this structure:
```
ansible-practice/
  inventory.ini
  ansible.cfg
  group_vars/
    all.yml
    web.yml
    db.yml
  host_vars/
    web-server.yml
  playbooks/
    site.yml
```

**`group_vars/all.yml`** -- applies to every host:
```yaml
---
ntp_server: pool.ntp.org
app_env: development
common_packages:
  - vim
  - htop
  - tree
```

**`group_vars/web.yml`** -- applies only to the web group:
```yaml
---
http_port: 80
max_connections: 1000
web_packages:
  - nginx
```

**`group_vars/db.yml`** -- applies only to the db group:
```yaml
---
db_port: 3306
db_packages:
  - mysql-server
```

**`host_vars/web-server.yml`** -- applies only to this specific host:
```yaml
---
max_connections: 2000
custom_message: "This is the primary web server"
```

Write a playbook `site.yml` that uses these variables:
```yaml
---
- name: Apply common config
  hosts: all
  become: true
  tasks:
    - name: Install common packages
      yum:
        name: "{{ common_packages }}"
        state: present
    - name: Show environment
      debug:
        msg: "Environment: {{ app_env }}"

- name: Configure web servers
  hosts: web
  become: true
  tasks:
    - name: Show web config
      debug:
        msg: "HTTP port: {{ http_port }}, Max connections: {{ max_connections }}"
    - name: Show host-specific message
      debug:
        msg: "{{ custom_message }}"
```

Run it and observe which variables apply to which hosts.

**Document:** What is the variable precedence? (hint: host_vars > group_vars > playbook vars, and `-e` overrides everything)

---

Concept First:
Variables should NOT live inside playbooks. 
They belong in dedicated files that Ansible auto-loads based on which host or group is being targeted.

    group_vars/all.yml        → loaded for EVERY host
    group_vars/web.yml        → loaded only for [web] group hosts
    group_vars/db.yml         → loaded only for [db] group hosts
    host_vars/web-server.yml  → loaded only for web-server specifically
    
Ansible finds these files automatically — you don't import them anywhere.

created dirs:

<img width="1320" height="55" alt="image" src="https://github.com/user-attachments/assets/1722ee48-e8da-4455-bf29-2121fb637cc3" />

vim group_vars/all.yml:

<img width="238" height="110" alt="image" src="https://github.com/user-attachments/assets/705643d6-5b8e-40df-b206-c84881bb1c40" />

vim group_vars/web.yml:

<img width="177" height="105" alt="image" src="https://github.com/user-attachments/assets/8652b6cf-d28c-4e70-a3dd-201e4c4a38c7" />

vim group_vars/db.yml:

<img width="144" height="65" alt="image" src="https://github.com/user-attachments/assets/1dc5e0fb-dfba-44e0-8885-ad04ba8dd340" />

vim host_vars/web-server.yml:

<img width="359" height="65" alt="image" src="https://github.com/user-attachments/assets/ee60dc33-045b-4f46-a3f2-0c394c7b9203" />

playbooks/site.yml:

<img width="607" height="338" alt="image" src="https://github.com/user-attachments/assets/856a3e7d-a92b-4c53-a9bd-1e68df1a25d3" />

ran the playbook:

<img width="872" height="620" alt="image" src="https://github.com/user-attachments/assets/03e23df7-39a1-477a-9e35-d9d2ea6e6f00" />

### max_connections is 1000 in group_vars/web.yml but 2000 in host_vars/web-server.yml. The host_vars wins because it has higher precedence.
✅ Document: Variable precedence in action — host_vars beat group_vars on max_connections.

---

### Task 3: Ansible Facts -- Gathering System Information
Ansible automatically collects "facts" about each managed node -- OS, IP, memory, CPU, disks, and hundreds more.

1. **See all facts for a host:**
```bash
ansible web-server -m setup
```

2. **Filter specific facts:**
```bash
ansible web-server -m setup -a "filter=ansible_os_family"
ansible web-server -m setup -a "filter=ansible_distribution*"
ansible web-server -m setup -a "filter=ansible_memtotal_mb"
ansible web-server -m setup -a "filter=ansible_default_ipv4"
```

3. **Use facts in a playbook** -- create `facts-demo.yml`:
```yaml
---
- name: Facts demo
  hosts: all
  tasks:
    - name: Show OS info
      debug:
        msg: >
          Hostname: {{ ansible_hostname }},
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }},
          RAM: {{ ansible_memtotal_mb }}MB,
          IP: {{ ansible_default_ipv4.address }}

    - name: Show all network interfaces
      debug:
        var: ansible_interfaces
```

Run it and observe the facts printed for each host.

**Document:** Name five facts you would use in real playbooks and why.

---

Concept First
Facts are variables Ansible automatically collects about each managed node before running any tasks. This happens during the TASK [Gathering Facts] step you saw in every Day 69 playbook run.
Ansible runs the setup module silently at the start of every play and collects hundreds of values — OS name, IP address, RAM, CPU, disk info, and more. You get all of this for free without writing anything.

: See ALL facts for your web server

    ansible web-server -m setup

<img width="677" height="1008" alt="image" src="https://github.com/user-attachments/assets/80c8ab0a-3729-4fff-913b-537d7ee0f008" />

This dumps hundreds of facts. Scroll through and get familiar with the format.

 Filter specific facts you'll actually use:

    ansible web-server -m setup -a "filter=ansible_os_family"
    ansible web-server -m setup -a "filter=ansible_distribution*"
    ansible web-server -m setup -a "filter=ansible_memtotal_mb"
    ansible web-server -m setup -a "filter=ansible_default_ipv4"

<img width="797" height="656" alt="image" src="https://github.com/user-attachments/assets/2710f832-fc47-4f5f-ab0b-afe62fd15974" />

Each filter shows just that fact. Notice ansible_default_ipv4 returns a nested dictionary — you access the IP with ansible_default_ipv4.address.

vim facts-demo.yml:

<img width="590" height="252" alt="image" src="https://github.com/user-attachments/assets/628f48fc-ddd1-4028-b1ad-2134ede080bc" />

ran:

<img width="915" height="626" alt="image" src="https://github.com/user-attachments/assets/573728ff-a48f-4362-9e76-293344636df7" />

Every value came automatically from the host — you didn't configure any of this.

Document: Five facts you'd use in real playbooks:

    Fact                     |   Why you'd use it
    
    ansible_distribution     | Run different tasks for Amazon Linux vs Ubuntu
    ansible_memtotal_mb    | Alert when a server has less than 1GB RAM
    ansible_default_ipv4.address | Write the server's own IP into a config file
    ansible_hostname      | Name log files or reports per server
    ansible_os_family   | Broader check — RedHat covers Amazon, CentOS, RHEL all at once

---

### Task 4: Conditionals with when
Tasks should not always run on every host. Use `when` to control execution.

Create `conditional-demo.yml`:

```yaml
---
- name: Conditional tasks demo
  hosts: all
  become: true

  tasks:
    - name: Install Nginx (only on web servers)
      yum:
        name: nginx
        state: present
      when: "'web' in group_names"

    - name: Install MySQL (only on db servers)
      yum:
        name: mysql-server
        state: present
      when: "'db' in group_names"

    - name: Show warning on low memory hosts
      debug:
        msg: "WARNING: This host has less than 1GB RAM"
      when: ansible_memtotal_mb < 1024

    - name: Run only on Amazon Linux
      debug:
        msg: "This is an Amazon Linux machine"
      when: ansible_distribution == "Amazon"

    - name: Run only on Ubuntu
      debug:
        msg: "This is an Ubuntu machine"
      when: ansible_distribution == "Ubuntu"

    - name: Run only in production
      debug:
        msg: "Production settings applied"
      when: app_env == "production"

    - name: Multiple conditions (AND)
      debug:
        msg: "Web server with enough memory"
      when:
        - "'web' in group_names"
        - ansible_memtotal_mb >= 512

    - name: OR condition
      debug:
        msg: "Either web or app server"
      when: "'web' in group_names or 'app' in group_names"
```

Run it and observe which tasks are skipped on which hosts.

**Verify:** Are tasks correctly skipping on hosts that don't match the condition?

---

Concept First
when tells Ansible to only run a task if the condition is true. If false, the task shows as skipping in the output — it's not an error, it's intentional.
Critical syntax rule — NO {{ }} inside when:

    # WRONG:
    when: "{{ app_env }}" == "production"
    
    # CORRECT:
    when: app_env == "production"

group_names is a built-in variable — a list of all groups the current host belongs to. So 'web' in group_names is true on web-server and false on db-server.

created conditional-demo.yml:

<img width="465" height="725" alt="image" src="https://github.com/user-attachments/assets/f9f1310b-ed7f-4fda-b30e-5e727743e09f" />

Applied and ran the playbook:

<img width="810" height="921" alt="image" src="https://github.com/user-attachments/assets/59edfa35-6f8d-4b4b-8d49-5f98d837420e" />

Watch the output — tasks that don't match show skipping.

      TASK [Install Nginx (only on web servers)] ***
      changed: [web-server]
      skipping: [db-server]        ← db-server is not in web group
      
      TASK [Install MySQL (only on db servers)] ***
      skipping: [web-server]       ← web-server is not in db group
      changed: [db-server]
      
      TASK [Run only on Ubuntu] ***
      skipping: [web-server]       ← web-server is Amazon Linux, not Ubuntu
      skipping: [db-server]
      
      TASK [Run only in production] ***
      skipping: [web-server]       ← app_env is "development" from group_vars/all.yml
      skipping: [db-server]

Conditional types:

| Type | Syntax | Example |
|---|---|---|
| Simple equality | `when: var == "value"` | `when: app_env == "production"` |
| Group check | `when: "'group' in group_names"` | `when: "'web' in group_names"` |
| Number compare | `when: fact < number` | `when: ansible_memtotal_mb < 1024` |
| AND (both true) | List format under `when:` | `when:`<br>`  - app_env == "prod"`<br>`  - ansible_memtotal_mb > 1024` |
| OR (either true) | Single line with `or` | `when: "'web' in group_names or 'app' in group_names"` |

---

### Task 5: Loops
Create `loops-demo.yml`:

```yaml
---
- name: Loops demo
  hosts: all
  become: true

  vars:
    users:
      - name: deploy
        groups: wheel
      - name: monitor
        groups: wheel
      - name: appuser
        groups: users

    directories:
      - /opt/app/logs
      - /opt/app/config
      - /opt/app/data
      - /opt/app/tmp

  tasks:
    - name: Create multiple users
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        state: present
      loop: "{{ users }}"

    - name: Create multiple directories
      file:
        path: "{{ item }}"
        state: directory
        mode: '0755'
      loop: "{{ directories }}"

    - name: Install multiple packages
      yum:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - curl
        - unzip
        - jq

    - name: Print each user created
      debug:
        msg: "Created user {{ item.name }} in group {{ item.groups }}"
      loop: "{{ users }}"
```

Run it and observe the loop output -- each iteration is shown separately.

**Document:** What is the difference between `loop` and the older `with_items`? (hint: `loop` is the modern recommended syntax)

---


Concept First:

loop lets you run one task for multiple items without copy-pasting the same task 10 times. Ansible iterates through the list and runs the task once per item. The current item is always available as {{ item }}.
For simple lists, {{ item }} is the value itself. For lists of dictionaries, you access fields with {{ item.fieldname }}.

created loop-demo.yml:

<img width="501" height="650" alt="image" src="https://github.com/user-attachments/assets/4c37b3ef-2137-4d7d-a1a7-6e9afaced364" />

ansible-playbook loop-demo.yml:

<img width="815" height="997" alt="image" src="https://github.com/user-attachments/assets/99e5c978-dadd-4e5f-a268-6a92344751b3" />

loop vs with_items:

| Feature | `loop` | `with_items` |
|---|---|---|
| Status | Modern, recommended | Old, deprecated |
| Available since | Ansible 2.5 | Original Ansible |
| Works for | Lists, dicts, nested loops | Basic lists only |
| Use it? | Yes, always | No, avoid in new playbooks |


---

### Task 6: Register, Debug, and Combine Everything
Build a real-world playbook `server-report.yml` that combines variables, facts, conditionals, and register:

```yaml
---
- name: Server Health Report
  hosts: all

  tasks:
    - name: Check disk space
      command: df -h /
      register: disk_result

    - name: Check memory
      command: free -m
      register: memory_result

    - name: Check running services
      shell: systemctl list-units --type=service --state=running | head -20
      register: services_result

    - name: Generate report
      debug:
        msg:
          - "========== {{ inventory_hostname }} =========="
          - "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
          - "IP: {{ ansible_default_ipv4.address }}"
          - "RAM: {{ ansible_memtotal_mb }}MB"
          - "Disk: {{ disk_result.stdout_lines[1] }}"
          - "Running services (first 20): {{ services_result.stdout_lines | length }}"

    - name: Flag if disk is critically low
      debug:
        msg: "ALERT: Check disk space on {{ inventory_hostname }}"
      when: "'9[0-9]%' in disk_result.stdout or '100%' in disk_result.stdout"

    - name: Save report to file
      copy:
        content: |
          Server: {{ inventory_hostname }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          IP: {{ ansible_default_ipv4.address }}
          RAM: {{ ansible_memtotal_mb }}MB
          Disk: {{ disk_result.stdout }}
          Checked at: {{ ansible_date_time.iso8601 }}
        dest: "/tmp/server-report-{{ inventory_hostname }}.txt"
      become: true
```

Run it and verify the report file is created on each server.

**Verify:** SSH into a server and read `/tmp/server-report-*.txt`. Does it contain accurate information?

---

Concept First:

register saves the entire output of a task into a variable you can use in later tasks. This is how you make decisions based on what actually happened on the server at runtime.

    - name: Check disk
      command: df -h /
      register: disk_result        # saves everything into this variable
    
    - name: Use the output
      debug:
        msg: "{{ disk_result.stdout }}"   # the command output as a string

The register object always contains:

| Key | What it holds |
|---|---|
| `disk_result.stdout` | Command output as one big string |
| `disk_result.stdout_lines` | Command output as a list, one element per line |
| `disk_result.rc` | Return code — `0` means success |
| `disk_result.stderr` | Any error output |

vim server-report.yml: 

<img width="611" height="574" alt="image" src="https://github.com/user-attachments/assets/28dc00d1-2160-44d3-b7e3-c65e9891293a" />

ran the playbook:

<img width="807" height="916" alt="image" src="https://github.com/user-attachments/assets/f720c344-e7ec-4498-a80d-dfc33b081e95" />

see report generated.

 SSH into the server and verify the report file was created:

<img width="1359" height="174" alt="image" src="https://github.com/user-attachments/assets/f5ed2336-9dfc-482f-aa02-1d6fb65b41c3" />

 Report file exists on each server with accurate live data pulled from that server automatically.

---

## Directory Structure

ansible-practice/
  inventory.ini
  ansible.cfg
  group_vars/
    all.yml        ← applies to every host
    web.yml        ← applies to [web] group only
    db.yml         ← applies to [db] group only
  host_vars/
    web-server.yml ← applies to web-server only
  playbooks/
    site.yml

## Variable Precedence (low → high)

| Priority | Source | Example |
|---|---|---|
| Lowest | group_vars/all.yml | app_env: development |
| ↓ | group_vars/web.yml | max_connections: 1000 |
| ↓ | host_vars/web-server.yml | max_connections: 2000 ← wins over group |
| ↓ | playbook vars: section | app_name: terraweek-app |
| Highest | -e on command line | -e "app_env=production" ← always wins |

## Five Useful Ansible Facts

| Fact | What it contains | Where I would use it |
|---|---|---|
| `ansible_distribution` | "Amazon", "Ubuntu", "CentOS" | Conditionally use yum vs apt |
| `ansible_memtotal_mb` | Total RAM in MB e.g. 985 | Alert when server has less than 1GB RAM |
| `ansible_default_ipv4.address` | Primary IP e.g. 10.0.1.5 | Write server IP into config files |
| `ansible_hostname` | Short hostname e.g. web-server | Name log files or reports per server |
| `ansible_os_family` | "RedHat", "Debian" | Broader OS check — covers Amazon, CentOS, RHEL |

## loop vs with_items

| | `loop` | `with_items` |
|---|---|---|
| Status | Modern, recommended | Old, deprecated |
| Available since | Ansible 2.5 | Original Ansible |
| Use it? | Yes, always | No, avoid in new playbooks |

## Server Report Output

<img width="1359" height="174" alt="image" src="https://github.com/user-attachments/assets/f5ed2336-9dfc-482f-aa02-1d6fb65b41c3" />

### Summary:

| Concept | What it means |
|---|---|
| `vars:` in playbook | Define variables inside the playbook itself |
| `group_vars/` | Auto-loaded variable files per group |
| `host_vars/` | Auto-loaded variable files per host — beats `group_vars` |
| `-e` flag | CLI override — beats everything including `host_vars` |
| Facts | Auto-collected from hosts at runtime — OS, IP, RAM etc |
| `when:` | Skip task if condition is false — shows `skipping` in output |
| `loop:` | Run one task for multiple items — modern syntax |
| `register:` | Save task output into a variable for use in later tasks |
| `stdout_lines` | The registered output split into a list line by line |
| `group_names` | Built-in list of groups the current host belongs to |
