# Day 69 -- Ansible Playbooks and Modules

## Task
Ad-hoc commands are useful for quick checks, but real automation lives in playbooks. A playbook is a YAML file that describes the desired state of your servers -- which packages to install, which services to run, which files to place where. You write it once, run it a hundred times, and get the same result every time.

Today you write your first playbooks and learn the modules that you will use on every project.

---

What Are We Learning Today?:
Yesterday you ran ad-hoc commands — one-liners for quick tasks. Today you write playbooks — the real power of Ansible.

The difference:

| | Ad-hoc command | Playbook |
|---|---|---|
| Format | Single command in terminal | YAML file |
| Use case | Quick one-off tasks | Repeatable automation |
| Example | `ansible all -m ping` | `ansible-playbook install-nginx.yml` |
| Reusable | ❌ No | ✅ Yes |

The most important concept today — Idempotency:

Run a playbook once → Nginx installs → changed
Run the same playbook again → Nginx already installed → ok (no change made)
This is idempotency. Ansible checks if the desired state already exists before doing anything. Safe to run 100 times, same result every time.

---

## Challenge Tasks

### Task 1: Your First Playbook
Create `install-nginx.yml`:

```yaml
---
- name: Install and start Nginx on web servers
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: true

    - name: Create a custom index page
      copy:
        content: "<h1>Deployed by Ansible - TerraWeek Server</h1>"
        dest: /usr/share/nginx/html/index.html
```

(Use `apt` instead of `yum` if your instances run Ubuntu)

Run it:
```bash
ansible-playbook install-nginx.yml
```

Read the output carefully -- every task shows `changed`, `ok`, or `failed`.

Now run it **again**. Notice that tasks show `ok` instead of `changed`. This is **idempotency** -- Ansible only makes changes when needed.

**Verify:** Curl the web server's public IP. Do you see your custom page?

---

 Concept First — Playbook Structure:

    ---                          # Every YAML file starts with ---
    - name: Play name            # A PLAY targets a group of hosts
      hosts: web                 # Which group from inventory
      become: true               # Run as root (sudo)
    
      tasks:                     # List of things to do
        - name: Task description # One TASK = one unit of work
          module_name:           # Which Ansible MODULE to use
            key: value           # Module arguments

Think of it like:

Play = a chapter in a book (targets specific servers)
Task = a sentence in that chapter (one action)
Module = the verb (install, copy, start, create)

step 1:
Create the nginx playbook:

    vim install-nginx.yml

<img width="895" height="460" alt="image" src="https://github.com/user-attachments/assets/2a479696-a5a8-44eb-bfb1-3a737dd05b38" />

<img width="1398" height="110" alt="image" src="https://github.com/user-attachments/assets/93273279-06e1-4eaa-88f1-e3dc05627195" />

No errors = safe to run ✅

ran the playbook and got error :

<img width="1891" height="335" alt="image" src="https://github.com/user-attachments/assets/940dcf45-d9c0-42b8-bc44-a73e3be57ac1" />

debugging and resolved the issue:

using: 

Amazon Linux 2

And in AL2 nginx is not present in default repo.
It comes from:

  amazon-linux-extras

Added extras enable step BEFORE install.

    ---
    - name: Install and start Nginx on web servers
      hosts: web
      become: true
    
      tasks:
    
        - name: Enable nginx extras repo
          command: amazon-linux-extras install nginx1 -y

new yml:

<img width="1012" height="607" alt="image" src="https://github.com/user-attachments/assets/78abb312-a5f2-4f72-aad6-137801537a9f" />

<img width="1876" height="535" alt="image" src="https://github.com/user-attachments/assets/d0fe8806-0385-487e-9298-22004d66e00e" />

running for the second time:

<img width="1874" height="525" alt="image" src="https://github.com/user-attachments/assets/4f9fd9a5-1ff2-4b11-8e95-d9aa9ff68bcc" />

here the task :  [Enable nginx extras repo] is showing as changed because the task is not idempotent. (it does not change somwthing in the system).

this is idempotency! Ansible made zero changes because everything was already in the desired state.

check the website on the browser:

     http://16.112.191.158

<img width="1028" height="267" alt="image" src="https://github.com/user-attachments/assets/1c951b3d-d88f-4f7b-b69b-e306a8d05ef0" />

---

### Task 2: Understand the Playbook Structure
Open your playbook and annotate each part in your notes:

```yaml
---                                    # YAML document start
- name: Play name                      # PLAY -- targets a group of hosts
  hosts: web                           # Which inventory group to run on
  become: true                         # Run tasks as root (sudo)

  tasks:                               # List of TASKS in this play
    - name: Task name                  # TASK -- one unit of work
      module_name:                     # MODULE -- what Ansible does
        key: value                     # Module arguments
```

Answer:
1. What is the difference between a play and a task?
2. Can you have multiple plays in one playbook?
3. What does `become: true` do at the play level vs the task level?
4. What happens if a task fails -- do remaining tasks still run?

---

Q1: Difference between a play and a task?

A play is a section targeting specific hosts: hosts: web — like "do these things on web servers"
A task is a single action inside a play: "install nginx", "start service" — each task calls one module

Q2: Can you have multiple plays in one playbook?
Yes! You'll do this in Task 6. One playbook can have a play for web servers, another play for app servers, all in the same file.

Q3: become: true at play level vs task level?

Play level (become: true under the play name) → ALL tasks in the play run as root
Task level (become: true inside one task) → ONLY that specific task runs as root

    # Play level — all tasks are root
    - name: My play
      hosts: web
      become: true        ← all tasks use sudo
      tasks:
        - name: task 1    ← runs as root
        - name: task 2    ← runs as root
    
    # Task level — only one task is root
    - name: My play
      hosts: web
      tasks:
        - name: task 1              ← runs as normal user
        - name: task 2
          become: true              ← only this runs as root

Q4: What if a task fails?
By default, Ansible stops running tasks on that host and marks it as failed. Other hosts continue. You can change this with ignore_errors: true on a task.

---

### Task 3: Learn the Essential Modules
Practice each of these modules by writing a playbook called `essential-modules.yml` with multiple tasks:

1. **`yum`/`apt`** -- Install and remove packages:
```yaml
- name: Install multiple packages
  yum:
    name:
      - git
      - curl
      - wget
      - tree
    state: present
```

2. **`service`** -- Manage services:
```yaml
- name: Ensure Nginx is running
  service:
    name: nginx
    state: started
    enabled: true
```

3. **`copy`** -- Copy files from control node to managed nodes:
```yaml
- name: Copy config file
  copy:
    src: files/app.conf
    dest: /etc/app.conf
    owner: root
    group: root
    mode: '0644'
```

4. **`file`** -- Create directories and manage permissions:
```yaml
- name: Create application directory
  file:
    path: /opt/myapp
    state: directory
    owner: ec2-user
    mode: '0755'
```

5. **`command`** -- Run a command (no shell features):
```yaml
- name: Check disk space
  command: df -h
  register: disk_output

- name: Print disk space
  debug:
    var: disk_output.stdout_lines
```

6. **`shell`** -- Run a command with shell features (pipes, redirects):
```yaml
- name: Count running processes
  shell: ps aux | wc -l
  register: process_count

- name: Show process count
  debug:
    msg: "Total processes: {{ process_count.stdout }}"
```

7. **`lineinfile`** -- Add or modify a single line in a file:
```yaml
- name: Set timezone in environment
  lineinfile:
    path: /etc/environment
    line: 'TZ=Asia/Kolkata'
    create: true
```

Create a `files/` directory with a sample `app.conf` file for the copy task. Run the playbook against all servers.

**Document:** What is the difference between `command` and `shell`? When should you use each?

---

created app.conf under files folder:

<img width="286" height="94" alt="image" src="https://github.com/user-attachments/assets/9f22c2c6-f513-4865-a439-2779ec5ac21d" />

all essential ansible modules:

vim essential-modules.yml:

<img width="553" height="942" alt="image" src="https://github.com/user-attachments/assets/b4442788-d9d4-4014-aa94-1d05baca74c0" />

ansible-playbook essential-modules.yml:

<img width="1896" height="954" alt="image" src="https://github.com/user-attachments/assets/d4839274-1861-4c6c-a37e-b0dacf191209" />

<img width="1901" height="337" alt="image" src="https://github.com/user-attachments/assets/b47d7321-d1dd-49ef-a77a-08f90d0b913f" />

run the playbook again:

<img width="1887" height="335" alt="image" src="https://github.com/user-attachments/assets/97410312-0d4b-47f8-a06d-0eedb3534948" />

---

| Module | Supports pipes/redirects | Supports variables | Use when |
|---|---|---|---|
| `command` | ❌ No | ❌ No | Simple commands: `df -h`, `uptime` |
| `shell` | ✅ Yes | ✅ Yes | Complex: `ps aux \| grep nginx` |

Rule of thumb: Use command by default (more secure). Only switch to shell when you need pipes (|), redirects (>), or shell variables ($HOME).

---

### Task 4: Handlers -- Restart Services Only When Needed
Handlers are tasks that run only when triggered by a `notify`. This avoids unnecessary service restarts.

Create `nginx-config.yml`:
```yaml
---
- name: Configure Nginx with a custom config
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Deploy Nginx config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
        owner: root
        mode: '0644'
      notify: Restart Nginx

    - name: Deploy custom index page
      copy:
        content: "<h1>Managed by Ansible</h1><p>Server: {{ inventory_hostname }}</p>"
        dest: /usr/share/nginx/html/index.html

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

Create `files/nginx.conf` with a basic Nginx config.

Run the playbook:
- First run: handler triggers because the config file is new
- Second run: handler does NOT trigger because nothing changed

**Verify:** Run it twice and compare the output. Does the handler run both times?

---

Concept First:

The problem without handlers:
Every time you run the playbook, you restart Nginx — even if nothing changed. Unnecessary restarts cause downtime!
Handlers solve this:

Nginx config changed → notify handler → Nginx restarts ✅
Nginx config unchanged → no notification → Nginx does NOT restart ✅

Handlers only run when:

A task sends a notify signal
That task made an actual change (changed, not ok)

    Task runs → config changed? → YES → notify "Restart Nginx" → handler runs at end → NO  → no notify → handler SKIPPED

vim files/nginx.conf:

<img width="622" height="418" alt="image" src="https://github.com/user-attachments/assets/467b6c7a-34bd-4108-99d9-c907aa97e174" />

vim nginx-config.yml:

<img width="629" height="498" alt="image" src="https://github.com/user-attachments/assets/db2cd19e-8448-479f-91f8-8c9306f2130d" />

ansible-playbook nginx-config.yml:

<img width="1891" height="317" alt="image" src="https://github.com/user-attachments/assets/aac6b01c-d676-4407-b09c-113f66e3b192" />

    TASK [Deploy Nginx config] ***
    changed: [web-server]         ← Config changed!
    
    ...
    
    RUNNING HANDLERS ***
    TASK [Restart Nginx] ***
    changed: [web-server]         ← Handler ran because config changed

ran the playbook again:

<img width="1899" height="278" alt="image" src="https://github.com/user-attachments/assets/ab178887-fbca-4719-b37e-9d66ee77e88a" />

    TASK [Deploy Nginx config] ***
    ok: [web-server]              ← No change
    
    ...
    
                                  ← No "RUNNING HANDLERS" section!
                                  ← Handler was never triggered

Handler ran on first run but NOT on second run — no unnecessary restarts!

changed files/nginx.conf:

<img width="539" height="398" alt="image" src="https://github.com/user-attachments/assets/05b2b663-0c83-4707-b4e8-951fe90d7cf0" />

ran the playbook again and handler got notified and restarted the nginx.

<img width="1892" height="329" alt="image" src="https://github.com/user-attachments/assets/3e8b9d94-9d96-410a-95b4-2166f0a4e6fd" />

Handler runs again because the config file changed ✅

---

### Task 5: Dry Run, Diff, and Verbosity
Before running playbooks on production, always preview changes first.

1. **Dry run (check mode)** -- shows what would change without changing anything:
```bash
ansible-playbook install-nginx.yml --check
```

2. **Diff mode** -- shows the actual file differences:
```bash
ansible-playbook nginx-config.yml --check --diff
```

3. **Verbosity** -- increase output detail for debugging:
```bash
ansible-playbook install-nginx.yml -v       # verbose
ansible-playbook install-nginx.yml -vv      # more verbose
ansible-playbook install-nginx.yml -vvv     # connection debugging
```

4. **Limit to specific hosts:**
```bash
ansible-playbook install-nginx.yml --limit web-server
```

5. **List what would be affected without running:**
```bash
ansible-playbook install-nginx.yml --list-hosts
ansible-playbook install-nginx.yml --list-tasks
```

**Document:** Why is `--check --diff` the most important flag combination for production use?

---

Concept First
Never run an untested playbook on production. Always preview first. These flags are your safety net:

| Flag | What it does | When to use |
|---|---|---|
| `--check` | Dry run — shows what WOULD change | Before any production run |
| `--diff` | Shows exact file differences | When files are being modified |
| `--check --diff` | Both together — best combo | Always before prod |
| `-v` | Verbose output | Basic troubleshooting |
| `-vv` | More verbose | Deeper troubleshooting |
| `-vvv` | Connection debug | SSH/connectivity issues |
| `--limit` | Run on specific host only | Test on one server first |

Dry run — see what would change without changing anything:

ansible-playbook install-nginx.yml --check:

<img width="1896" height="294" alt="image" src="https://github.com/user-attachments/assets/f6c3a8c7-db43-4184-82d8-1d206a2bbc04" />

changed the files/nginx.conf file and checked with diff:

<img width="938" height="510" alt="image" src="https://github.com/user-attachments/assets/28ebd597-3fa6-4abb-a420-fc0fd6504a98" />

ansible-playbook install-nginx.yml -v:

<img width="1885" height="890" alt="image" src="https://github.com/user-attachments/assets/dc8dd82d-bcd4-4f4c-a33d-82d50169e5a2" />

ansible-playbook install-nginx.yml -vv:

<img width="1906" height="954" alt="image" src="https://github.com/user-attachments/assets/dd3da021-aad4-451b-81b9-c7d87730c35e" />

ansible-playbook install-nginx.yml -vvv:

<img width="1912" height="1045" alt="image" src="https://github.com/user-attachments/assets/6c573751-d8a4-4de9-8679-1bd5b703d8f5" />

Limit to one host (test on one before all)
 
   ansible-playbook install-nginx.yml --limit web-server:

<img width="1297" height="327" alt="image" src="https://github.com/user-attachments/assets/ba1bae8d-b058-45db-9211-5ade42f3355f" />

Only runs on web-server even if playbook targets all web servers.

List affected hosts and tasks without running:

    # show which hosts would be targeted:
     ansible-playbook install-nginx.yml --list-hosts:
    
    # Show which tasks would run
     ansible-playbook install-nginx.yml --list-tasks:

<img width="753" height="258" alt="image" src="https://github.com/user-attachments/assets/fbe411bb-0d41-4b5b-9771-106e8faeff58" />

## Why --check --diff is critical for production:
It shows you exactly what Ansible WOULD do without actually doing it. You can review every file change before it happens — prevents surprises and accidental config overwrites in production.

---

### Task 6: Multiple Plays in One Playbook
Write `multi-play.yml` with separate plays for each server group:

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present
    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: true

- name: Configure app servers
  hosts: app
  become: true
  tasks:
    - name: Install Node.js dependencies
      yum:
        name:
          - gcc
          - make
        state: present
    - name: Create app directory
      file:
        path: /opt/app
        state: directory
        mode: '0755'

- name: Configure database servers
  hosts: db
  become: true
  tasks:
    - name: Install MySQL client
      yum:
        name: mysql
        state: present
    - name: Create data directory
      file:
        path: /var/lib/appdata
        state: directory
        mode: '0700'
```

Run it:
```bash
ansible-playbook multi-play.yml
```

Watch the output -- each play targets a different group, and tasks run only on the relevant hosts.

**Verify:** Is Nginx only installed on web servers? Is MySQL only on db servers?

---

Concept First
One playbook can have multiple plays. Each play targets a different group. They run in order — web play first, then app play, then db play.

    multi-play.yml
      Play 1: hosts: web  → runs on web-server only
      Play 2: hosts: app  → runs on app-server only
      Play 3: hosts: db   → runs on db-server only

vim multi-play.yml:

<img width="522" height="973" alt="image" src="https://github.com/user-attachments/assets/103cee4e-9a7d-4100-ab68-f531e708980d" />

list all hosts:

<img width="743" height="259" alt="image" src="https://github.com/user-attachments/assets/5fbcca06-d6c5-40e7-932b-27dc91943fcf" />

applied or ran the playbook:

<img width="880" height="693" alt="image" src="https://github.com/user-attachments/assets/8df63f64-f188-47aa-9efb-7304d284d364" />

different secitons for all hosts.

verify isolation:
    
    # Nginx only on web server
    ansible web -m command -a "nginx -v"
    
    # MySQL only on db server
    ansible db -m command -a "mysql --version"
    
    # Nginx NOT on db server (should fail)
    ansible db -m command -a "nginx -v"


Verified: Nginx only on web, MySQL only on db — each play targeted the right servers.

<img width="689" height="92" alt="image" src="https://github.com/user-attachments/assets/38696e5e-4c1a-4ad1-a44b-cbf7e20e0010" />

---

## Day 69 – Ansible Playbooks and Modules - Summary

## Ad-hoc vs Playbook

| | Ad-hoc | Playbook |
|---|---|---|
| Format | Terminal command | YAML file |
| Reusable | ❌ | ✅ |
| Complex logic | ❌ | ✅ |
| Best for | Quick checks | Real automation |

## Playbook Structure
```yaml
---
- name: Play name          # Targets a group of hosts
  hosts: web               # Which inventory group
  become: true             # All tasks run as root

  tasks:
    - name: Task name      # One unit of work
      module_name:         # Which module to use
        key: value         # Module arguments

  handlers:
    - name: Handler name   # Only runs when notified
      service:
        name: nginx
        state: restarted
```

## Idempotency
First run: task shows `changed` — Ansible made a change.
Second run: task shows `ok` — already in desired state, no action taken.
Safe to run 100 times. Same result every time.

## Essential Modules

| Module | What it does | Key arguments |
|---|---|---|
| `yum` / `apt` | Install/remove packages | `name`, `state: present/absent` |
| `service` | Manage services | `name`, `state: started/stopped`, `enabled` |
| `copy` | Copy files to managed node | `src`, `dest`, `owner`, `mode` |
| `file` | Create dirs, manage permissions | `path`, `state: directory/touch/absent` |
| `command` | Run commands (no pipes) | `cmd` or bare argument |
| `shell` | Run commands with pipes | `cmd` or bare argument |
| `lineinfile` | Add/modify one line in file | `path`, `line`, `create` |
| `debug` | Print messages/variables | `msg` or `var` |

## command vs shell

| Module | Pipes | Shell vars | Use when |
|---|---|---|---|
| `command` | ❌ | ❌ | Simple: `df -h`, `uptime` |
| `shell` | ✅ | ✅ | Complex: `ps aux \| wc -l` |

## Handlers
Handlers run only when a task sends a notify AND that task changed.
No change → no notify → handler skipped → no unnecessary restarts.

## Useful Flags

| Flag | What it does |
|---|---|
| `--check` | Dry run — shows changes without applying |
| `--diff` | Shows exact file differences |
| `--check --diff` | Best combo before production |
| `-v / -vv / -vvv` | Increasing verbosity for debugging |
| `--limit hostname` | Run on one specific host only |
| `--list-hosts` | Show which hosts would be targeted |
| `--list-tasks` | Show which tasks would run |
| `--syntax-check` | Validate YAML before running |

##  Summary Table

| Concept | What it means |
|---|---|
| Playbook | YAML file defining what to do on which servers |
| Play | One section targeting a specific host group |
| Task | One unit of work — calls one module |
| Module | The action: install, copy, start, create |
| Idempotency | Run 10 times = same result. No change if already correct state |
| `changed` | Task made an actual change to the server |
| `ok` | Task found server already in desired state — no action taken |
| `failed` | Task encountered an error |
| `become: true` | Run as root (sudo) |
| Handler | Task that only runs when notified by another task |
| `notify` | Signal sent by a task when it changes something |
| `register` | Save task output to a variable |
| `debug` | Print a variable or message |
| `--check` | Dry run — preview changes without applying |
| `--diff` | Show exact file differences before applying |
| `--limit` | Restrict playbook to specific host |
| `state: present` | Install/create if not exists |
| `state: absent` | Remove if exists |
| `state: started` | Start if not running |
| `state: restarted` | Always restart |

