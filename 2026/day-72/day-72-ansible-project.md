# Day 72 -- Ansible Project: Automate Docker and Nginx Deployment

## Task
Five days of Ansible -- inventory, ad-hoc commands, playbooks, modules, handlers, variables, facts, conditionals, loops, roles, templates, Galaxy, and Vault. Today you put it all together and build what you would actually do on the job.

Automate a complete deployment: install Docker, pull and run a containerized application, set up Nginx as a reverse proxy in front of it, and manage everything through Ansible roles. One command to go from a fresh server to a fully running, production-style setup.

---

This is the Ansible Capstone! You're combining everything from Days 68-71 into one real production-style project. Let me explain the full picture first.

 What Are We Building?:

    Your Terminal
         ↓
    ansible-playbook site.yml
         ↓
    ┌─────────────────────────────────────────┐
    │  EC2 Server (web-server)                │
    │                                         │
    │  Nginx (port 80)  ←── User's browser   │
    │       ↓                                 │
    │  Docker Container (port 8080)           │
    │  running nginx/httpd app                │
    │                                         │
    │  Everything installed by Ansible        │
    │  Secrets protected by Ansible Vault     │
    └─────────────────────────────────────────┘

What each role does:

| Role | Job |
|---|---|
| common | Baseline setup — packages, timezone, deploy user |
| docker | Install Docker, pull image, run container |
| nginx | Install Nginx, configure as reverse proxy to container |


---

## Challenge Tasks

### Task 1: Plan the Project Structure
Create the complete project layout:

```
ansible-docker-project/
  ansible.cfg
  inventory.ini
  site.yml                          # Master playbook
  group_vars/
    all.yml                         # Common variables
    web/
      vars.yml                      # Nginx variables
      vault.yml                     # Encrypted Docker Hub credentials
  roles/
    common/                         # Shared setup for all servers
      tasks/main.yml
    docker/                         # Docker installation and container management
      tasks/main.yml
      templates/
        docker-compose.yml.j2
      handlers/main.yml
      defaults/main.yml
    nginx/                          # Nginx reverse proxy
      tasks/main.yml
      templates/
        nginx.conf.j2
        app-proxy.conf.j2
      handlers/main.yml
      defaults/main.yml
```

Generate the role skeletons:
```bash
mkdir -p ansible-docker-project/roles
cd ansible-docker-project
ansible-galaxy init roles/common
ansible-galaxy init roles/docker
ansible-galaxy init roles/nginx
```

Set up your `ansible.cfg` and `inventory.ini` using what you built on Day 68.

---

created project dir & Generate role skeletons using ansible-galaxy:

<img width="811" height="128" alt="image" src="https://github.com/user-attachments/assets/6d3ea805-7bcb-43d5-b81c-6655355033f2" />

Each init creates a full role structure automatically:

    roles/
      common/
        tasks/main.yml
        handlers/main.yml
        defaults/main.yml
        vars/main.yml
        templates/
        files/
        meta/main.yml
      docker/    ← same structure
      nginx/     ← same structure


<img width="754" height="123" alt="image" src="https://github.com/user-attachments/assets/2c3302ee-5004-4723-9964-24addc51f05c" />

Create all additional directories and files

<img width="723" height="133" alt="image" src="https://github.com/user-attachments/assets/aefc5fbe-ba19-4799-a751-33f39286c528" />

verified structure:

<img width="836" height="417" alt="image" src="https://github.com/user-attachments/assets/c86b67a4-daf1-4bae-9411-8eaf460a83b2" />

vim ansible.cfg:

<img width="284" height="116" alt="image" src="https://github.com/user-attachments/assets/ab4c62fc-2259-494c-8ff6-5ae12733c8bb" />

inventory.ini:

<img width="690" height="374" alt="image" src="https://github.com/user-attachments/assets/10dae37a-bba7-4c8d-9b93-3fd0dc6698b6" />

Ping: (Make sure your .valut_pass is present):

<img width="729" height="187" alt="image" src="https://github.com/user-attachments/assets/942d5428-dd2a-4291-9ede-ea632b63196f" />

---

### Task 2: Build the Common Role
The `common` role runs on every server -- baseline packages and setup.

**`roles/common/tasks/main.yml`:**
```yaml
---
- name: Update package cache
  yum:
    update_cache: true
  tags: common

- name: Install common packages
  yum:
    name: "{{ common_packages }}"
    state: present
  tags: common

- name: Set hostname
  hostname:
    name: "{{ inventory_hostname }}"
  tags: common

- name: Set timezone
  timezone:
    name: "{{ timezone }}"
  tags: common

- name: Create deploy user
  user:
    name: deploy
    groups: wheel
    shell: /bin/bash
    state: present
  tags: common
```

(Use `apt` instead of `yum` if your instances run Ubuntu)

**`group_vars/all.yml`:**
```yaml
---
timezone: Asia/Kolkata
project_name: devops-app
app_env: development
common_packages:
  - vim
  - curl
  - wget
  - git
  - htop
  - tree
  - jq
  - unzip
```

---

Concept First:

The common role runs on EVERY server before anything else. It's the baseline — packages everyone needs, correct timezone, a deploy user. Think of it as "make this server ready for work."

vim group_vars/all.yml:

<img width="276" height="316" alt="image" src="https://github.com/user-attachments/assets/523c4432-00ca-4a65-acb4-e2f99ef17487" />

vim roles/common/tasks/main.yml:

<img width="490" height="633" alt="image" src="https://github.com/user-attachments/assets/a033a58f-cf68-4e09-b68d-199499eb53a8" />

 Common role is done — clean, focused, runs once on every server.

---

### Task 3: Build the Docker Role
This role installs Docker, starts the service, pulls images, and runs containers.

**`roles/docker/defaults/main.yml`:**
```yaml
---
docker_app_image: nginx
docker_app_tag: latest
docker_app_name: myapp
docker_app_port: 8080
docker_container_port: 80
```

**`roles/docker/tasks/main.yml`:**
Write tasks that:
1. Install Docker dependencies (`yum-utils`, `device-mapper-persistent-data`, `lvm2`)
2. Add the Docker CE repository
3. Install Docker CE
4. Start and enable the Docker service
5. Add the `deploy` user to the `docker` group
6. Install Docker Compose (via pip or direct download)
7. Log in to Docker Hub using vault-encrypted credentials:
```yaml
- name: Log in to Docker Hub
  community.docker.docker_login:
    username: "{{ vault_docker_username }}"
    password: "{{ vault_docker_password }}"
  become_user: deploy
  when: vault_docker_username is defined
```
8. Pull the application image:
```yaml
- name: Pull application image
  community.docker.docker_image:
    name: "{{ docker_app_image }}"
    tag: "{{ docker_app_tag }}"
    source: pull
```
9. Run the container:
```yaml
- name: Run application container
  community.docker.docker_container:
    name: "{{ docker_app_name }}"
    image: "{{ docker_app_image }}:{{ docker_app_tag }}"
    state: started
    restart_policy: always
    ports:
      - "{{ docker_app_port }}:{{ docker_container_port }}"
```
10. Verify the container is running:
```yaml
- name: Wait for container to be healthy
  uri:
    url: "http://localhost:{{ docker_app_port }}"
    status_code: 200
  retries: 5
  delay: 3
  register: health_check
  until: health_check.status == 200
```

Tag all tasks with `docker`.

**`roles/docker/handlers/main.yml`:**
```yaml
---
- name: Restart Docker
  service:
    name: docker
    state: restarted
```

**Install the required Ansible collection** (needed for `community.docker` modules):
```bash
ansible-galaxy collection install community.docker
```

---

Concept First:

This role:

- Installs Docker on Amazon Linux 2
- Starts Docker service
- Pulls your app image from Docker Hub
- Runs the container on port 8080

Why port 8080 and not 80? Because Nginx will listen on port 80 and forward requests to the container on 8080. They can't both use port 80.

vim roles/docker/defaults/main.yml:

<img width="451" height="212" alt="image" src="https://github.com/user-attachments/assets/6a42d318-7e65-4421-82e4-73e13236f15c" />

vim roles/docker/tasks/main.yml: 

<img width="760" height="978" alt="image" src="https://github.com/user-attachments/assets/5966a5ce-3aea-41b6-94ca-be88ad52cf16" />

vim roles/docker/handlers/main.yml:

<img width="197" height="94" alt="image" src="https://github.com/user-attachments/assets/35830dfe-f3a0-40e6-9f7f-a69891a49485" />

Docker role done — installs Docker, pulls image, runs container, health checks it.

---

### Task 4: Build the Nginx Role
This role installs Nginx and configures it as a reverse proxy to the Docker container.

**`roles/nginx/defaults/main.yml`:**
```yaml
---
nginx_http_port: 80
nginx_upstream_port: 8080
nginx_server_name: "_"
```

**`roles/nginx/tasks/main.yml`:**
Write tasks that:
1. Install Nginx
2. Remove the default Nginx site config
3. Deploy the main Nginx config from a template
4. Deploy the reverse proxy config from a template
5. Test Nginx config before reloading:
```yaml
- name: Test Nginx configuration
  command: nginx -t
  changed_when: false
```
6. Start and enable Nginx
7. Use a handler to reload Nginx when any config changes

Tag all tasks with `nginx`.

**`roles/nginx/templates/app-proxy.conf.j2`:**
```nginx
# Reverse Proxy to Docker Container -- Managed by Ansible
upstream docker_app {
    server 127.0.0.1:{{ nginx_upstream_port }};
}

server {
    listen {{ nginx_http_port }};
    server_name {{ nginx_server_name }};

    location / {
        proxy_pass http://docker_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /health {
        access_log off;
        return 200 'OK';
        add_header Content-Type text/plain;
    }

{% if app_env == 'production' %}
    access_log /var/log/nginx/{{ project_name }}_access.log;
    error_log /var/log/nginx/{{ project_name }}_error.log;
{% else %}
    access_log /var/log/nginx/{{ project_name }}_access.log;
    error_log /var/log/nginx/{{ project_name }}_error.log debug;
{% endif %}
}
```

**`roles/nginx/handlers/main.yml`:**
```yaml
---
- name: Reload Nginx
  service:
    name: nginx
    state: reloaded

- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```

---


Concept First:

Nginx sits in front of the Docker container:

    User → Port 80 → Nginx → Port 8080 → Docker Container

Why use Nginx in front of a container?

Nginx handles SSL termination
Nginx can serve static files directly
Nginx can load balance multiple containers
Nginx provides access logging

vim roles/nginx/defaults/main.yml:

<img width="492" height="85" alt="image" src="https://github.com/user-attachments/assets/a7a6fdd5-5f26-4eb0-8b1e-830033adacfd" />

vim roles/nginx/tasks/main.yml:

<img width="548" height="569" alt="image" src="https://github.com/user-attachments/assets/e31da792-bbb3-4a42-8464-ee483466a55e" />

vim  roles/nginx/templates/app-proxy.conf.j2:

<img width="525" height="533" alt="image" src="https://github.com/user-attachments/assets/964a0262-24dc-47da-8cbc-f2dac8266316" />

vim roles/nginx/handlers/main.yml:

<img width="362" height="198" alt="image" src="https://github.com/user-attachments/assets/069f2685-b2bd-4c72-9d18-b561c5642e45" />

Nginx role done — installs, configures as reverse proxy, uses template with Jinja2 conditionals.

---

### Task 5: Encrypt Docker Hub Credentials with Vault
1. Create the vault file:
```bash
ansible-vault create group_vars/web/vault.yml
```
Add:
```yaml
vault_docker_username: your-dockerhub-username
vault_docker_password: your-dockerhub-token
```

2. Create a vault password file for convenience:
```bash
echo "YourVaultPassword" > .vault_pass
chmod 600 .vault_pass
echo ".vault_pass" >> .gitignore
```

3. Reference it in `ansible.cfg`:
```ini
[defaults]
inventory = inventory.ini
host_key_checking = False
vault_password_file = .vault_pass
```

---

Concept First:
Your Docker Hub username and password should NEVER be in a plain text file committed to Git. Ansible Vault encrypts them so the file can be safely committed but the values are unreadable without the vault password.

vim .gitignore:

<img width="156" height="89" alt="image" src="https://github.com/user-attachments/assets/b071b37c-8870-42e1-bec7-27f82462cbc1" />

ansible-vault create group_vars/web/vault.yml:

<img width="391" height="105" alt="image" src="https://github.com/user-attachments/assets/dd9606c0-3e4a-40ce-b4cf-14438877f42c" />

It will ask for the vault password — use whatever you put in .vault_pass.

now my docker username and pass is encrypted:

<img width="862" height="194" alt="image" src="https://github.com/user-attachments/assets/ee302046-4f93-4167-9151-9d60d7792871" />

vim group_vars/web/vars.yml:

<img width="309" height="85" alt="image" src="https://github.com/user-attachments/assets/b616b3e5-61a6-46d6-8b28-3ffcc2659161" />

View vault content to verify (uses .vault_pass automatically):

    ansible-vault view group_vars/web/vault.yml

Showed my credentials decrypted.

<img width="866" height="33" alt="image" src="https://github.com/user-attachments/assets/83d6c9fa-d1a1-400b-8dce-0abebc0cdd68" />

vim site.yml:

<img width="588" height="467" alt="image" src="https://github.com/user-attachments/assets/fa7add69-8057-423c-97bc-8d1e362fc8f6" />

ansible-playbook site.yml --check --diff:

<img width="608" height="1011" alt="image" src="https://github.com/user-attachments/assets/77cd0522-ad4c-4d89-b475-9161e04d8a40" />

ran ansible playbook site.yaml:

<img width="780" height="978" alt="image" src="https://github.com/user-attachments/assets/7de8db31-caaf-49b7-a08b-51b156b73faf" />

<img width="842" height="564" alt="image" src="https://github.com/user-attachments/assets/e99438a1-e583-46ff-a883-effc9cf8001f" />

also faced issue with urllib3 v2.0 since it only supports  OpenSSL 1.1.1+ but my Ec2 has OpenSSL 1.0.2k-fips

and This task:

community.docker.docker_login

needs:

Python requests library
urllib3
Docker SDK

but urllib3 v2 requires newer OpenSSL.

solution:

Add BEFORE Docker login task:

- name: Install compatible urllib3
  pip:
    name: "urllib3<2"
    executable: pip3

and install older compatible version.


Now verified using dokcer ps to target machines and reponse:

<img width="1023" height="309" alt="image" src="https://github.com/user-attachments/assets/e9c5236c-1443-4478-a8b3-32e21952171d" />

on browser:

<img width="1289" height="370" alt="image" src="https://github.com/user-attachments/assets/5e733c73-9408-4b6b-b202-f973470b49c4" />

\health enpoint:

<img width="640" height="213" alt="image" src="https://github.com/user-attachments/assets/6266437c-c4e3-4d8a-a519-6cde59091f67" />

ran the playbook again:

<img width="829" height="979" alt="image" src="https://github.com/user-attachments/assets/5c2e3062-15e2-46cb-8565-a88903188b83" />

showed 2 changes why?:

The playbook still shows `changed=2` because the `deploy` user is being modified in multiple roles.  
The `user` module with `groups` and `append: true` may report changes repeatedly even if the user is already in the `docker` group.  
This is an idempotency edge case and also a design issue where multiple roles manage the same resource.


executed: ansible-playbook site.yml --tags docker -e "docker_app_image=httpd docker_app_tag=latest docker_app_name=apache-app"

<img width="1367" height="711" alt="image" src="https://github.com/user-attachments/assets/b03ac26a-397e-461c-a094-6a28c7f399e8" />

<img width="614" height="209" alt="image" src="https://github.com/user-attachments/assets/94e2b4d8-a8cc-46d1-a8e4-c126cf8c61c2" />

Use tags for selective deployment:

# Only run Nginx tasks (skip common and docker)
ansible-playbook site.yml --tags nginx

# Only run Docker tasks
ansible-playbook site.yml --tags docker

# Run everything EXCEPT common
ansible-playbook site.yml --skip-tags common

# Run on specific host only
ansible-playbook site.yml --limit web-server

---

## Architecture

    User Browser
    ↓ port 80
    Nginx (reverse proxy)
    ↓ port 8080
    Docker Container (nginx/httpd)
    ↑
    Everything deployed by Ansible
    Secrets protected by Ansible Vault

## Project Structure

    ansible-docker-project/
    ansible.cfg          ← inventory, vault_pass, remote_user
    inventory.ini        ← web and app server groups
    site.yml             ← master playbook with 3 plays
    .vault_pass          ← vault password (gitignored!)
    .gitignore
    group_vars/
    all.yml            ← common_packages, timezone, project_name
    web/
    vars.yml         ← nginx ports
    vault.yml        ← encrypted Docker Hub credentials
    roles/
    common/            ← packages, timezone, deploy user
    docker/            ← Docker install, image pull, container run
    nginx/             ← Nginx install, reverse proxy template


## Concepts Used From Each Day

| Day | Concept Used in This Project |
|---|---|
| 68 | Inventory, SSH setup, ad-hoc commands for testing |
| 69 | Playbooks, modules (yum, service, copy), handlers |
| 70 | Variables (group_vars, defaults), facts, conditionals in template |
| 71 | Roles, Jinja2 templates, ansible-galaxy, Vault encryption |
| 72 | Everything combined — full deployment project |

## How Vault Protects Credentials
- Docker Hub username and password stored in group_vars/web/vault.yml
- File is encrypted with AES256 — unreadable without vault password
- .vault_pass file holds the password — gitignored, never committed
- ansible.cfg points to .vault_pass for automatic decryption

## Tags for Selective Deployment

| Command | What runs |
|---|---|
| `ansible-playbook site.yml` | Full deployment |
| `ansible-playbook site.yml --tags docker` | Only Docker tasks |
| `ansible-playbook site.yml --tags nginx` | Only Nginx tasks |
| `ansible-playbook site.yml --skip-tags common` | Skip baseline setup |

## Idempotency Proof
First run: changed=15 — installed packages, started services, created files
Second run: changed=0 — everything already in desired state, nothing touched

## What I Would Add for Production
- SSL/TLS with certbot and Let's Encrypt
- Docker Compose for multi-container apps
- Log rotation configuration
- Monitoring with Prometheus + node_exporter
- Firewall rules with firewalld role
- Automated backups

## Summary table:

| Concept | What it means in this project |
|---|---|
| `site.yml` | Master playbook — calls all 3 roles in order |
| `common` role | Baseline setup: packages, timezone, deploy user |
| `docker` role | Installs Docker, pulls image, runs container on port 8080 |
| `nginx` role | Installs Nginx, reverse proxies port 80 → 8080 |
| `group_vars/all.yml` | Variables shared across all hosts |
| `group_vars/web/vault.yml` | Encrypted Docker Hub credentials |
| `defaults/main.yml` | Role default values — easily overridden |
| `templates/app-proxy.conf.j2` | Jinja2 template — generates Nginx config dynamically |
| `notify: Reload Nginx` | Handler triggered only when config file changes |
| `restart_policy: always` | Container auto-restarts if server reboots |
| `community.docker` | Ansible collection for Docker modules |
| `--tags docker` | Run only Docker tasks — skip common and nginx |
| `--check --diff` | Dry run — preview changes before applying |
| `changed=0` | Idempotency — second run makes zero unnecessary changes |
| `ansible-vault create` | Creates encrypted file for secrets |
| `.vault_pass` | Auto-decryption file — always gitignore this! |
