# Day 71 -- Roles, Galaxy, Templates and Vault

## Task
Your playbooks are getting bigger. Tasks, variables, handlers, files -- all living in one YAML file that grows longer every day. In real projects, you manage dozens of servers with different roles -- web servers, databases, monitoring agents, load balancers. You need a way to organize, reuse, and share automation.

Today you learn Ansible Roles (the standard way to structure automation), Jinja2 Templates (dynamic config files), Ansible Galaxy (the community marketplace), and Ansible Vault (secrets management).

---

What Are We Learning Today?
Your playbooks are getting bigger every day. Tasks, variables, handlers, files — all living in one YAML file. Today you fix that.

The difference:

| Before (Day 70) | After (Day 71) |
|---|---|
| Everything in one playbook | Organized into roles |
| Hardcoded static files | Dynamic Jinja2 templates |
| Write everything yourself | Galaxy roles in one command |
| Plain text passwords | Vault encrypted files |

Four things you learn today:

| Concept | What it does |
|---|---|
| Jinja2 Templates | Generate config files dynamically using variables and facts |
| Ansible Roles | Standard way to organize and reuse automation |
| Ansible Galaxy | Community marketplace — install pre-built roles |
| Ansible Vault | Encrypt secrets — passwords, API keys, tokens |

---

## Challenge Tasks

### Task 1: Jinja2 Templates
Templates let you generate config files dynamically using variables and facts.

1. Create `templates/nginx-vhost.conf.j2`:
```jinja2
# Managed by Ansible -- do not edit manually
server {
    listen {{ http_port | default(80) }};
    server_name {{ ansible_hostname }};

    root /var/www/{{ app_name }};
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/{{ app_name }}_access.log;
    error_log /var/log/nginx/{{ app_name }}_error.log;
}
```

2. Create a playbook `template-demo.yml`:
```yaml
---
- name: Deploy Nginx with template
  hosts: web
  become: true
  vars:
    app_name: terraweek-app
    http_port: 80

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Create web root
      file:
        path: "/var/www/{{ app_name }}"
        state: directory
        mode: '0755'

    - name: Deploy vhost config from template
      template:
        src: templates/nginx-vhost.conf.j2
        dest: "/etc/nginx/conf.d/{{ app_name }}.conf"
        owner: root
        mode: '0644'
      notify: Restart Nginx

    - name: Deploy index page
      copy:
        content: "<h1>{{ app_name }}</h1><p>Host: {{ ansible_hostname }} | IP: {{ ansible_default_ipv4.address }}</p>"
        dest: "/var/www/{{ app_name }}/index.html"

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

Run it with `--diff` to see the rendered template:
```bash
ansible-playbook template-demo.yml --diff
```

**Verify:** SSH into the web server and read the generated config. Are the variables replaced with actual values?

---

 Concept First:
 
A template is a config file with placeholders. Ansible fills in the placeholders using variables and facts at runtime. You write the template once, and it generates a different config for each server automatically.

    Template file (.j2)          →    Rendered config on server
    server_name {{ ansible_hostname }}  →  server_name web-server
    root /var/www/{{ app_name }}        →  root /var/www/terraweek-app

In templates:

| Syntax | What it does |
|---|---|
| `{{ variable }}` | Renders a variable value |
| `{% if condition %}` | Conditional block |
| `{% for item in list %}` | Loop block |
| `\| default(value)` | Fallback if variable is undefined |

vim templates/nginx-vhost.conf.j2:

<img width="422" height="240" alt="image" src="https://github.com/user-attachments/assets/2805a381-e631-46cb-a40e-25a26ea777c3" />

vim template-demo.yml:

<img width="854" height="538" alt="image" src="https://github.com/user-attachments/assets/d5259c9f-0ac8-4456-9fb8-080a63c8d2b1" />

The --diff flag shows you EXACTLY what the rendered template looks like before it lands on the server:

<img width="818" height="837" alt="image" src="https://github.com/user-attachments/assets/f8e3359d-b25f-4425-ac5a-60fa09d90fb5" />

 SSH into the server and verify the rendered config:
 
<img width="793" height="571" alt="image" src="https://github.com/user-attachments/assets/2f34b686-a2c9-4bb0-9fff-5b74372c8b4a" />

All variables replaced with actual values. ansible_hostname filled in automatically from facts — you never set it yourself.

--diff ≠ dry run (it does execute the playbook).

---

### Task 2: Understand the Role Structure
An Ansible role has a fixed directory structure. Each directory has a specific purpose:

```
roles/
  webserver/
    tasks/
      main.yml         # The main task list
    handlers/
      main.yml         # Handlers (restart services, etc.)
    templates/
      nginx.conf.j2    # Jinja2 templates
    files/
      index.html       # Static files to copy
    vars/
      main.yml         # Role variables (high priority)
    defaults/
      main.yml         # Default variables (low priority, easily overridden)
    meta/
      main.yml         # Role metadata and dependencies
```

Every directory contains a `main.yml` that Ansible loads automatically. You only create the directories you need.

Generate a skeleton with:
```bash
ansible-galaxy init roles/webserver
```

Explore the generated directory. Read the README.md that Galaxy creates.

**Document:** What is the difference between `vars/main.yml` and `defaults/main.yml`?

---

Concept First:

A role is just a folder with a fixed directory structure that Ansible understands automatically. Instead of one giant playbook, you split everything into dedicated files and Ansible loads them in the right order.

    roles/
      webserver/
        tasks/
          main.yml       ← The main task list — loaded automatically
        handlers/
          main.yml       ← Handlers — loaded automatically
        templates/
          nginx.conf.j2  ← Jinja2 templates for this role
        files/
          index.html     ← Static files to copy
        vars/
          main.yml       ← Role variables — HIGH priority
        defaults/
          main.yml       ← Default variables — LOW priority, easily overridden
        meta/
          main.yml       ← Role metadata and dependencies


did galaxy init:

<img width="644" height="149" alt="image" src="https://github.com/user-attachments/assets/11785ade-a76e-4c0b-9f75-c005690ccc57" />

### vars/main.yml vs defaults/main.yml:

| Feature | `vars/main.yml` | `defaults/main.yml` |
|---|---|---|
| Priority | HIGH — hard to override | LOW — easily overridden |
| Use for | Values that must not change | Sensible fallback values |
| Example | Internal paths, required config | Port numbers, package names |
| Can caller override? | Only with `-e` flag | Yes, from anywhere |

---

### Task 3: Build a Custom Webserver Role
Build a complete `webserver` role from scratch:

**`roles/webserver/defaults/main.yml`:**
```yaml
---
http_port: 80
app_name: myapp
max_connections: 512
```

**`roles/webserver/tasks/main.yml`:**
```yaml
---
- name: Install Nginx
  yum:
    name: nginx
    state: present

- name: Deploy Nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    mode: '0644'
  notify: Restart Nginx

- name: Deploy vhost config
  template:
    src: vhost.conf.j2
    dest: "/etc/nginx/conf.d/{{ app_name }}.conf"
    owner: root
    mode: '0644'
  notify: Restart Nginx

- name: Create web root
  file:
    path: "/var/www/{{ app_name }}"
    state: directory
    mode: '0755'

- name: Deploy index page
  template:
    src: index.html.j2
    dest: "/var/www/{{ app_name }}/index.html"
    mode: '0644'

- name: Start and enable Nginx
  service:
    name: nginx
    state: started
    enabled: true
```

**`roles/webserver/handlers/main.yml`:**
```yaml
---
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```

**`roles/webserver/templates/index.html.j2`:**
```html
<h1>{{ app_name }}</h1>
<p>Server: {{ ansible_hostname }}</p>
<p>IP: {{ ansible_default_ipv4.address }}</p>
<p>Environment: {{ app_env | default('development') }}</p>
<p>Managed by Ansible</p>
```

Create the `vhost.conf.j2` and `nginx.conf.j2` templates yourself based on what you learned in Task 1.

Now call the role from a playbook `site.yml`:
```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  roles:
    - role: webserver
      vars:
        app_name: terraweek
        http_port: 80
```

Run it:
```bash
ansible-playbook site.yml
```

**Verify:** Curl the web server. Does the custom page load?

---

Concept First:
 
You call a role from a playbook with the roles: key instead of tasks:. Ansible then runs everything inside that role automatically — tasks, handlers, templates — all from their dedicated files.

vim roles/webserver/defaults/main.yml:

<img width="292" height="131" alt="image" src="https://github.com/user-attachments/assets/5d5a17aa-d27d-4402-a793-3390dc6dadc3" />

vim roles/webserver/tasks/main.yml:

<img width="349" height="612" alt="image" src="https://github.com/user-attachments/assets/de8a88ca-895d-450b-9b27-67bf73394cef" />

vim roles/webserver/handlers/main.yml:

<img width="260" height="140" alt="image" src="https://github.com/user-attachments/assets/d76627e5-7087-4a27-935c-c64b81d97c1e" />

vim roles/webserver/templates/index.html.j2:

<img width="460" height="93" alt="image" src="https://github.com/user-attachments/assets/ecc7aa05-620d-46b1-9ed0-5168a1bbda1d" />

vim roles/webserver/templates/vhost.conf.j2:

<img width="404" height="231" alt="image" src="https://github.com/user-attachments/assets/2d049aff-5a4f-4e41-9bdf-fff30002c150" />

vim roles/webserver/templates/nginx.conf.j2:

<img width="552" height="294" alt="image" src="https://github.com/user-attachments/assets/a0b0d838-d642-4637-9aa6-d41644d271f7" />

vim site.yml:

<img width="239" height="170" alt="image" src="https://github.com/user-attachments/assets/fb125fa9-7cdc-4e63-93a7-039642de6cdc" />

ran the playbook:

<img width="901" height="439" alt="image" src="https://github.com/user-attachments/assets/9dad349c-ba3e-421e-88e5-17bd185d27c4" />

Notice the tasks are prefixed with webserver : — that shows which role each task came from.

checked in the browser:

<img width="487" height="270" alt="image" src="https://github.com/user-attachments/assets/41773761-a583-4767-a7bd-71f27606bb4d" />

---

### Task 4: Ansible Galaxy -- Use Community Roles
Ansible Galaxy is a marketplace of pre-built roles.

1. **Search for roles:**
```bash
ansible-galaxy search nginx --platforms EL
ansible-galaxy search mysql
```

2. **Install a role from Galaxy:**
```bash
ansible-galaxy install geerlingguy.docker
```

3. **Check where it was installed:**
```bash
ansible-galaxy list
```

4. **Use the installed role** -- create `docker-setup.yml`:
```yaml
---
- name: Install Docker using Galaxy role
  hosts: app
  become: true
  roles:
    - geerlingguy.docker
```

Run it -- Docker gets installed with a single role call.

5. **Use a requirements file** for managing multiple roles. Create `requirements.yml`:
```yaml
---
roles:
  - name: geerlingguy.docker
    version: "7.4.1"
  - name: geerlingguy.ntp
```

Install all at once:
```bash
ansible-galaxy install -r requirements.yml
```

**Document:** Why use a `requirements.yml` instead of installing roles manually?

---

Concept First
Ansible Galaxy is a community marketplace of pre-built roles. Instead of writing a role to install Docker from scratch (50+ tasks), you install a Galaxy role in one command and call it in 3 lines of YAML.

Search for avaiable roles:

ansible-galaxy search nginx --platforms EL:

<img width="1063" height="891" alt="image" src="https://github.com/user-attachments/assets/16b527fb-d89d-4f89-9ea9-b9de8303776f" />

<img width="879" height="596" alt="image" src="https://github.com/user-attachments/assets/947c83a7-af3a-4287-889e-a0af729ae479" />

Install the Docker role from Galaxy:

    ansible-galaxy install geerlingguy.docker

<img width="781" height="68" alt="image" src="https://github.com/user-attachments/assets/7059d4c5-0cdc-465b-b2d1-6bb20cc8029d" />

    - downloading role 'docker', owned by geerlingguy
    - downloading role from https://...
    - extracting geerlingguy.docker to /home/ec2-user/.ansible/roles/geerlingguy.docker
    - geerlingguy.docker (7.4.1) was installed successfully

vim docker-setup.yml:

<img width="334" height="119" alt="image" src="https://github.com/user-attachments/assets/ec44b318-5769-4dd4-8b5e-e45e953d9723" />

ran the playbook:

<img width="838" height="222" alt="image" src="https://github.com/user-attachments/assets/371f716d-9992-4445-a13e-d5a07dfce932" />

Docker installs completely with one role call — no writing 50 tasks yourself.

<img width="710" height="184" alt="image" src="https://github.com/user-attachments/assets/b056cb3c-fea4-4a19-b680-f33d5c077db7" />

 Why use requirements.yml:
 
| Feature | Manual Install | `requirements.yml` |
|---|---|---|
| Command | Run install per role | One command installs all |
| Version control | No version pinning | Pin exact versions |
| Team sharing | Each person installs manually | Everyone runs same command |
| CI/CD pipelines | Hard to automate | Easy — `install -r requirements.yml` |
| Reproducibility | Different versions per person | Same versions everywhere |

---

### Task 5: Ansible Vault -- Encrypt Secrets
Never put passwords, API keys, or tokens in plain text. Ansible Vault encrypts sensitive data.

1. **Create an encrypted file:**
```bash
ansible-vault create group_vars/db/vault.yml
```
It will ask for a vault password, then open an editor. Add:
```yaml
vault_db_password: SuperSecretP@ssw0rd
vault_db_root_password: R00tP@ssw0rd123
vault_api_key: sk-abc123xyz789
```
Save and exit. Open the file with `cat` -- it is fully encrypted.

2. **Edit an encrypted file:**
```bash
ansible-vault edit group_vars/db/vault.yml
```

3. **View without editing:**
```bash
ansible-vault view group_vars/db/vault.yml
```

4. **Encrypt an existing file:**
```bash
ansible-vault encrypt group_vars/db/secrets.yml
```

5. **Use vault variables in a playbook** -- create `db-setup.yml`:
```yaml
---
- name: Configure database
  hosts: db
  become: true

  tasks:
    - name: Show DB password (never do this in production)
      debug:
        msg: "DB password is set: {{ vault_db_password | length > 0 }}"
```

Run with the vault password:
```bash
ansible-playbook db-setup.yml --ask-vault-pass
```

6. **Use a password file** (better for CI/CD):
```bash
echo "YourVaultPassword" > .vault_pass
chmod 600 .vault_pass
echo ".vault_pass" >> .gitignore

ansible-playbook db-setup.yml --vault-password-file .vault_pass
```

Or set it in `ansible.cfg`:
```ini
[defaults]
vault_password_file = .vault_pass
```

**Document:** Why is `--vault-password-file` better than `--ask-vault-pass` for automated pipelines?

---

Concept First:

Never put passwords or API keys in plain text YAML. Ansible Vault encrypts the file so it looks like gibberish to anyone without the password — but Ansible decrypts it transparently at runtime.

    Plain text YAML          →   vault encrypt   →   Encrypted blob
    vault_db_password: secret                        $ANSIBLE_VAULT;1.1;AES256
                                                     34663966383061353562623965...

mkdir -p group_vars/db

-Create an encrypted vault file

ansible-vault create group_vars/db/vault.yml

It asks for a vault password — enter one and remember it. Then your editor opens. Add:

<img width="381" height="117" alt="image" src="https://github.com/user-attachments/assets/a73d27b6-f3d8-4bcb-8712-ec6ef64cb282" />

the file is encrypted:

<img width="612" height="158" alt="image" src="https://github.com/user-attachments/assets/16e5931c-004f-45cd-aef5-e5be78988670" />

put password and see the encrypted file:

<img width="712" height="87" alt="image" src="https://github.com/user-attachments/assets/013709ad-7620-4250-86e6-3dc117d560cd" />

to edit the encrypted file:

    ansible-vault edit group_vars/db/vault.yml
    
This decrypts, opens your editor, and re-encrypts on save.

Encrypt an existing plain text file:

    ansible-vault encrypt group_vars/db/secrets.yml

vim db-setup.yml:

<img width="582" height="165" alt="image" src="https://github.com/user-attachments/assets/5fb4a680-1521-4949-8042-5f26f3809969" />

Ansible asks for the vault password, decrypts the file in memory, and the variables are available like any other variable.:

<img width="834" height="221" alt="image" src="https://github.com/user-attachments/assets/ae7ccaf6-dbb2-4b77-90bd-ac6115d9b1da" />

automate or skip the ask for password:

<img width="833" height="261" alt="image" src="https://github.com/user-attachments/assets/e975a5e0-56dd-4d12-9a22-27c9fe4b6f1f" />

this time it did not asked password since its stored in .vault_pass file.

 Set the password file in ansible.cfg so you never need the flag
 
    vim ansible.cfg

Add under [defaults]:

    vault_password_file = .vault_pass

Didn't ask for password and nor we need to pass it:

<img width="895" height="240" alt="image" src="https://github.com/user-attachments/assets/fd3b49c4-cdcb-453e-9cd1-f9d37c7622ac" />

Vault decrypts automatically. No prompt, no flag.

--ask-vault-pass vs --vault-password-file:

| Feature | `--ask-vault-pass` | `--vault-password-file` |
|---|---|---|
| How it works | Prompts you to type the password | Reads password from a file |
| CI/CD pipelines | Breaks — no human to type | Works — file is always there |
| Automation | Cannot automate | Fully automatable |
| Security | Password in your head | File must be `chmod 600` |
| Best for | Manual one-off runs | Automated pipelines, cron jobs |

---

### Task 6: Combine Roles, Templates, and Vault
Write a complete `site.yml` that uses everything you learned today:

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  roles:
    - role: webserver
      vars:
        app_name: terraweek
        http_port: 80

- name: Configure app servers with Docker
  hosts: app
  become: true
  roles:
    - geerlingguy.docker

- name: Configure database servers
  hosts: db
  become: true
  tasks:
    - name: Create DB config with secrets
      template:
        src: templates/db-config.j2
        dest: /etc/db-config.env
        owner: root
        mode: '0600'
```

Create `templates/db-config.j2`:
```jinja2
# Database Configuration -- Managed by Ansible
DB_HOST={{ ansible_default_ipv4.address }}
DB_PORT={{ db_port | default(3306) }}
DB_PASSWORD={{ vault_db_password }}
DB_ROOT_PASSWORD={{ vault_db_root_password }}
```

Run:
```bash
ansible-playbook site.yml
```

**Verify:** SSH into the db server and check `/etc/db-config.env`. Are the secrets rendered correctly? Is the file permission `600`?

---

Concept First:

This is what production Ansible looks like — roles for web, Galaxy roles for Docker, vault for secrets, and templates that pull secrets into config files at runtime.

vim templates/db-config.j2:

<img width="417" height="103" alt="image" src="https://github.com/user-attachments/assets/7aea8267-52b5-47ad-8902-b2a73ed05af4" />

vim site.yml:

<img width="439" height="764" alt="image" src="https://github.com/user-attachments/assets/43abd253-b4be-454c-8116-5b7209a66e33" />

ran playbook:

<img width="963" height="817" alt="image" src="https://github.com/user-attachments/assets/9546f438-7d60-4c7a-8881-3aaa376f9b15" />

logged in into the db ec2 instance:

<img width="448" height="146" alt="image" src="https://github.com/user-attachments/assets/c99416c7-53b0-42f4-8bcf-06e0f7521da7" />

verified into the ec2 instance:

<img width="495" height="253" alt="image" src="https://github.com/user-attachments/assets/ba43e4dc-c8de-44cf-84ed-fb3a2e69e06a" />

Secrets rendered correctly, file is 600 — only root can read 

---

## Webserver Role Directory Structure

roles/
  webserver/
    tasks/
      main.yml       ← Main task list — runs automatically
    handlers/
      main.yml       ← Restart Nginx handler
    templates/
      nginx.conf.j2  ← Dynamic main nginx config
      vhost.conf.j2  ← Dynamic vhost config
      index.html.j2  ← Dynamic index page
    files/           ← Static files (empty for this role)
    vars/
      main.yml       ← High priority variables
    defaults/
      main.yml       ← Low priority defaults: http_port, app_name
    meta/
      main.yml       ← Role metadata

## vars/main.yml vs defaults/main.yml

| | `vars/main.yml` | `defaults/main.yml` |
|---|---|---|
| Priority | HIGH — hard to override | LOW — easily overridden |
| Use for | Values that must not change | Sensible fallback values |
| Can caller override? | Only with `-e` flag | Yes, from anywhere |

## Jinja2 Template Syntax

| Syntax | What it does |
|---|---|
| `{{ variable }}` | Renders a variable value |
| `{% if condition %}` | Conditional block |
| `{% for item in list %}` | Loop block |
| `\| default(value)` | Fallback if variable is undefined |

## Galaxy Requirements File

Using requirements.yml pins versions and lets the whole team install the
same roles with one command: ansible-galaxy install -r requirements.yml

## Vault Workflow

| Command | What it does |
|---|---|
| `ansible-vault create file.yml` | Create a new encrypted file |
| `ansible-vault edit file.yml` | Edit an encrypted file |
| `ansible-vault view file.yml` | View without editing |
| `ansible-vault encrypt file.yml` | Encrypt an existing plain file |
| `ansible-vault decrypt file.yml` | Decrypt to plain text |
| `--ask-vault-pass` | Prompt for password at runtime |
| `--vault-password-file .vault_pass` | Read password from file |

## When to Use What

| Situation | Use |
|---|---|
| Quick one-off task | Ad-hoc command |
| Repeatable automation for one server type | Playbook |
| Reusable automation across projects | Role |
| Installing community software | Galaxy role |
| Dynamic config files | Jinja2 template |
| Passwords, API keys, tokens | Ansible Vault |

## Summary :

| Concept | What it means |
|---|---|
| Jinja2 template | Config file with `{{ }}` placeholders filled at runtime |
| `.j2` extension | Convention for Jinja2 template files |
| `template:` module | Renders a `.j2` file and copies it to the server |
| `\| default(value)` | Fallback value if variable is undefined |
| Role | Folder with fixed structure — tasks, handlers, templates, defaults |
| `defaults/main.yml` | Low priority role variables — easily overridden by callers |
| `vars/main.yml` | High priority role variables — only `-e` can override |
| `roles:` in playbook | Call a role instead of listing tasks manually |
| `ansible-galaxy init` | Generate the role skeleton directory structure |
| Galaxy | Community marketplace of pre-built roles |
| `requirements.yml` | Pin role names and versions — install all with one command |
| `ansible-vault create` | Create a new encrypted YAML file |
| `ansible-vault edit` | Edit an encrypted file — decrypts, opens editor, re-encrypts |
| `--vault-password-file` | Read vault password from file — works in CI/CD pipelines |
| `.vault_pass` | Local password file — always add to `.gitignore` |

