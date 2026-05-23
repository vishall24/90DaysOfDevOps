# Day 74 -- Node Exporter, cAdvisor, and Grafana Dashboards

## Task
Prometheus is running and you can query metrics. But right now it is only monitoring itself. In production, you need to monitor two critical things: the **host machine** (CPU, memory, disk, network) and the **Docker containers** running on it.

Today you add Node Exporter for host metrics, cAdvisor for container metrics, and set up Grafana to visualize everything in dashboards instead of raw PromQL.

---

What Are We Building Today?

Yesterday (Day 73) you set up Prometheus monitoring itself. Today you add the full observability stack:

    ┌─────────────────────────────────────────────────┐
    │  Your Server                                    │
    │                                                 │
    │  ┌─────────────┐    scrapes    ┌─────────────┐  │
    │  │ Node        │ ←──────────── │             │  │
    │  │ Exporter    │  host metrics │             │  │
    │  │ :9100       │               │ Prometheus  │  │
    │  └─────────────┘               │ :9090       │  │
    │                                │             │  │
    │  ┌─────────────┐    scrapes    │             │  │
    │  │ cAdvisor    │ ←──────────── │             │  │
    │  │ :8080       │ container     └─────────────┘  │
    │  └─────────────┘  metrics            │          │
    │                                      │ data     │
    │  ┌─────────────┐                     ↓          │
    │  │ Grafana     │ ────────────── queries &        │
    │  │ :3000       │               dashboards        │
    │  └─────────────┘                                │
    └─────────────────────────────────────────────────┘

Three new tools today:

| Tool | What it monitors | Metrics prefix |
|---|---|---|
| Node Exporter | Host machine — CPU, RAM, disk, network | `node_` |
| cAdvisor | Docker containers — per-container resources | `container_` |
| Grafana | Visualization — turns PromQL into beautiful dashboards | N/A |

Why all three?

- Prometheus collects and stores metrics — but raw numbers are hard to read
- Node Exporter tells you "the server is using 80% RAM"
- cAdvisor tells you "the nginx container is using 200MB of that RAM"
- Grafana shows all of this in beautiful charts and graphs

---

## Challenge Tasks

### Task 1: Add Node Exporter for Host Metrics
Node Exporter exposes Linux system metrics (CPU, memory, disk, filesystem, network) in Prometheus format.

Update your `docker-compose.yml` from Day 73 -- add the Node Exporter service:
```yaml
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    restart: unless-stopped
```

**Why these volume mounts?**
- `/proc` -- kernel and process information (CPU stats, memory info)
- `/sys` -- hardware and driver details
- `/` -- filesystem usage (disk space)

All mounted read-only (`ro`) -- Node Exporter only reads, never modifies.

Add it as a scrape target in `prometheus.yml`:
```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]
```

Restart the stack:
```bash
docker compose up -d
```

Verify Node Exporter is healthy:
```bash
curl http://localhost:9100/metrics | head -20
```

Check Prometheus Targets page -- `node-exporter` should show as `UP`.

Run these queries in Prometheus to see host metrics:
```promql
# CPU: percentage of time spent idle (per core)
node_cpu_seconds_total{mode="idle"}

# Memory: total vs available
node_memory_MemTotal_bytes
node_memory_MemAvailable_bytes

# Memory usage percentage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Disk: filesystem usage percentage
(1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100

# Network: bytes received per second
rate(node_network_receive_bytes_total[5m])
```

---

Concept First:
Node Exporter is a tiny process that reads Linux system files (/proc, /sys) and exposes them in Prometheus format. It's like a translator — Linux kernel stats → Prometheus metrics.

/proc/cpuinfo  →  node_cpu_seconds_total
/proc/meminfo  →  node_memory_MemTotal_bytes
/sys/block     →  node_disk_read_bytes_total

The volume mounts are read-only (ro) — Node Exporter only reads, never writes

vim docker-compose.yaml:

<img width="1006" height="801" alt="image" src="https://github.com/user-attachments/assets/a22b4c1b-dca3-41aa-8d1d-bc3b2c55c382" />

 Update prometheus.yml to scrape Node Exporter:

vim prometheus.yaml:

<img width="859" height="322" alt="image" src="https://github.com/user-attachments/assets/1b6b8263-d57f-4387-a1aa-87c4a5b7382e" />

<img width="1919" height="198" alt="image" src="https://github.com/user-attachments/assets/9071b933-3655-47e5-b947-3bc8c680da67" />

docker ps & curl http://localhost:9100/metrics | head -30

<img width="1889" height="910" alt="image" src="https://github.com/user-attachments/assets/c2b75473-05f5-46a8-8f04-294471271c7c" />

verified in dashboard:

<img width="1919" height="570" alt="image" src="https://github.com/user-attachments/assets/68b1965b-88d1-4b53-8f03-c10b73e1c983" />

Executed:

    # Current memory usage percentage
    (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
    
<img width="1919" height="293" alt="image" src="https://github.com/user-attachments/assets/3e38ef60-ed44-4ab5-b41c-c753fb0e3a49" />

= 55.4

Executed:

      # Disk usage percentage on root filesystem
    (1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100

<img width="1919" height="358" alt="image" src="https://github.com/user-attachments/assets/477c3161-98ec-43e6-9a4a-23de14e64254" />


Executed:
    
    # Network bytes received per second (5 min average)
    rate(node_network_receive_bytes_total[5m])

<img width="1919" height="364" alt="image" src="https://github.com/user-attachments/assets/de4421db-3d52-4e1a-a0c1-1b91c3b57d4d" />

Execued:

    # CPU usage percentage (100% minus idle time)
    100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

<img width="1919" height="293" alt="image" src="https://github.com/user-attachments/assets/333edfab-bc7f-4860-9eb5-01090cd4c8e3" />

Verified:
Node Exporter shows as UP in Prometheus Targets and queries return data.

---

### Task 2: Add cAdvisor for Container Metrics
cAdvisor (Container Advisor) monitors resource usage and performance of running Docker containers.

Add it to your `docker-compose.yml`:
```yaml
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    restart: unless-stopped
```

**Why these volume mounts?**
- Docker socket (`docker.sock`) -- lets cAdvisor discover and query running containers
- `/sys` -- kernel-level container stats (cgroups)
- `/var/lib/docker/` -- container filesystem information

Add cAdvisor as a Prometheus scrape target:
```yaml
  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]
```

Restart and verify:
```bash
docker compose up -d
```

Open `http://localhost:8080` to see the cAdvisor web UI. Click on Docker Containers to see per-container stats.

Run these queries in Prometheus:
```promql
# CPU usage per container (in seconds)
rate(container_cpu_usage_seconds_total{name!=""}[5m])

# Memory usage per container
container_memory_usage_bytes{name!=""}

# Network received bytes per container
rate(container_network_receive_bytes_total{name!=""}[5m])

# Which container is using the most memory?
topk(3, container_memory_usage_bytes{name!=""})
```

The `{name!=""}` filter removes aggregated/system-level entries and shows only named containers.

**Document:** What is the difference between Node Exporter and cAdvisor? When would you use each?

---

Concept First:

Node Exporter tells you about the HOST. cAdvisor tells you about individual CONTAINERS

| Question | Use |
|---|---|
| "Is my server out of memory?" | Node Exporter |
| "Which container is using all the memory?" | cAdvisor |
| "What's the server's disk usage?" | Node Exporter |
| "How much CPU is my nginx container using?" | cAdvisor |

cAdvisor connects to the Docker socket (docker.sock) — this lets it discover all running containers and read their stats from Linux cgroups.

vim docker-compose.yaml:

<img width="969" height="1042" alt="image" src="https://github.com/user-attachments/assets/1894200b-8947-4ca8-88e7-7e0a43d79f6e" />

vim prometheus.yaml:

<img width="509" height="398" alt="image" src="https://github.com/user-attachments/assets/0d00615e-b354-4ec3-8fc0-7438bd23c049" />

<img width="1881" height="343" alt="image" src="https://github.com/user-attachments/assets/f5e1fa39-9df1-4d17-b89c-998f8513c856" />

http://<EC2-public-IP>:8080:

<img width="1919" height="1063" alt="image" src="https://github.com/user-attachments/assets/b3b24580-6abc-4259-a846-e0c9b4e2f822" />

saw a web UI showing all containers with their resource usage. Click "Docker Containers" to see per-container stats.

<img width="1904" height="1025" alt="image" src="https://github.com/user-attachments/assets/e31371b4-7304-431f-b974-3c8b9761f4d3" />

confirmed inthe targets: (prometheus may not show the latest data so do : docker compose restart prometheus):

<img width="1919" height="718" alt="image" src="https://github.com/user-attachments/assets/2e10f1ef-b189-42ca-b570-dda4b8ce412b" />

Executed:

    container_memory_working_set_bytes / 1024 / 1024
    
<img width="1909" height="1066" alt="image" src="https://github.com/user-attachments/assets/2393ac6d-8add-42dd-8f4f-d30d4a4cd59a" />

Executed:

    rate(container_cpu_usage_seconds_total[5m])
    
<img width="1919" height="1022" alt="image" src="https://github.com/user-attachments/assets/4a892732-3db9-40bc-8aa9-3ab2c8cd14e2" />

Executed:

    topk(5, container_memory_working_set_bytes)
    
<img width="1919" height="545" alt="image" src="https://github.com/user-attachments/assets/ef7c98b7-8c8e-4b0a-8188-183ca32445f6" />

In prometheus:
metric names differ across exporter versions

### Node Exporter vs cAdvisor:

| | Node Exporter | cAdvisor |
|---|---|---|
| Monitors | Host machine (Linux server) | Docker containers |
| Metrics prefix | `node_` | `container_` |
| Use when | Server CPU/RAM/disk is high | Need to find which container is responsible |
| Port | 9100 | 8080 |

---

### Task 3: Set Up Grafana
Grafana is the visualization layer. It connects to Prometheus (and later Loki) and lets you build dashboards, set alerts, and share views with your team.

Add Grafana to your `docker-compose.yml`:
```yaml
  grafana:
    image: grafana/grafana-enterprise:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    restart: unless-stopped
```

Add the volume at the bottom of your compose file:
```yaml
volumes:
  prometheus_data:
  grafana_data:
```

Restart:
```bash
docker compose up -d
```

Open `http://localhost:3000`. Log in with `admin` / `admin123`.

**Add Prometheus as a datasource:**
1. Go to Connections > Data Sources > Add data source
2. Select Prometheus
3. Set URL to `http://prometheus:9090` (use the container name, not localhost -- they are on the same Docker network)
4. Click Save & Test -- you should see "Successfully queried the Prometheus API"

---

Prometheus stores metrics and lets you query them. But raw numbers in a query box aren't useful for a team. Grafana connects to Prometheus and turns those queries into:

- Time series graphs (CPU over last hour)
- Gauges (current memory %)
- Alerts (email/Slack when disk > 90%)
- Dashboards shared across the whole team

added graphana in docker compose file:

<img width="1019" height="785" alt="image" src="https://github.com/user-attachments/assets/175776f5-0adf-46bd-80ef-a4751c8a640a" />

docker compos ps:

<img width="1900" height="774" alt="image" src="https://github.com/user-attachments/assets/8ccf7fb1-ac12-4e1f-b245-5a6e79344311" />

opened graphana and addedprometheus in data source :

<img width="1901" height="895" alt="image" src="https://github.com/user-attachments/assets/4357a34b-d5e6-4f71-a747-bd88eeb06a7e" />

Why? Grafana and Prometheus are in the same Docker network. They talk to each other using container names, not localhost.

✅ "Successfully queried the Prometheus API"

---

### Task 4: Build Your First Dashboard
Create a dashboard that shows the health of your system at a glance.

1. Go to Dashboards > New Dashboard > Add Visualization
2. Select Prometheus as the datasource

**Panel 1 -- CPU Usage (Gauge):**
```promql
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```
- Visualization: Gauge
- Title: "CPU Usage %"
- Set thresholds: green < 60, yellow < 80, red >= 80

**Panel 2 -- Memory Usage (Gauge):**
```promql
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
```
- Visualization: Gauge
- Title: "Memory Usage %"

**Panel 3 -- Container CPU Usage (Time Series):**
```promql
rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100
```
- Visualization: Time series
- Title: "Container CPU Usage"
- Legend: `{{name}}`

**Panel 4 -- Container Memory Usage (Bar Chart):**
```promql
container_memory_usage_bytes{name!=""} / 1024 / 1024
```
- Visualization: Bar chart
- Title: "Container Memory (MB)"
- Legend: `{{name}}`

**Panel 5 -- Disk Usage (Stat):**
```promql
(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100
```
- Visualization: Stat
- Title: "Disk Usage %"

Save the dashboard as "DevOps Observability Overview".

---

 Concept First:
 
A Grafana dashboard is a collection of panels. Each panel runs a PromQL query and visualizes the result. You're going to build 5 panels showing CPU, memory, disk, and container metrics.

added new dashboard and executed query:

    100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
    
<img width="1895" height="1060" alt="image" src="https://github.com/user-attachments/assets/9796ecec-1545-4edd-b036-a7f9f7e48922" />

ccreated visualization panel and set the thresholds as well:

<img width="1617" height="997" alt="image" src="https://github.com/user-attachments/assets/6981efb5-071b-401d-958b-9610c6e4d5bd" />

added all the visualization and dashboard renamed it to DevOps Observability:

<img width="1919" height="1013" alt="image" src="https://github.com/user-attachments/assets/abf89905-c1aa-4c4f-a2ae-0857d8c7f6b1" />

with promql queries:

      - 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
      - (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
      - rate(container_cpu_usage_seconds_total{id=~".*docker.*"}[5m]) * 100
      - container_memory_usage_bytes{id=~".*docker.*"} / 1024 / 1024
      - (1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100

 Now have a 5-panel dashboard showing host and container metrics!

 ---

### Task 5: Auto-Provision Datasources with YAML
In production, you do not click through the UI to add datasources. You provision them with configuration files so the setup is repeatable.

Create the provisioning directory structure:
```bash
mkdir -p grafana/provisioning/datasources
mkdir -p grafana/provisioning/dashboards
```

Create `grafana/provisioning/datasources/datasources.yml`:
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false
```

Update the Grafana service in `docker-compose.yml` to mount the provisioning directory:
```yaml
  grafana:
    image: grafana/grafana-enterprise:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    restart: unless-stopped
```

Restart Grafana:
```bash
docker compose up -d grafana
```

Check Connections > Data Sources -- Prometheus should already be there without any manual setup.

**Document:** Why is provisioning datasources via YAML better than configuring them manually through the UI?

---

 Concept First:
 
Clicking through the Grafana UI to add datasources works once. But if you rebuild your stack, you have to click again. If you have 5 team members, they all have to click.
Provisioning via YAML = datasources are automatically configured when Grafana starts. No clicking, fully reproducible, stored in Git

vim grafana/provisioning/datasources/datasources.yml:

<img width="778" height="205" alt="image" src="https://github.com/user-attachments/assets/f8c1f0e1-e235-4bc9-9766-c500fc62fcff" />

Go to Grafana: Connections → Data Sources
data source was already there:

<img width="1916" height="567" alt="image" src="https://github.com/user-attachments/assets/c17d56d5-95d5-4cae-b4b4-ee6f35e5bea6" />

Why provisioning via YAML is better::

| | Manual UI config | YAML provisioning |
|---|---|---|
| Reproducible | ❌ Click every time | ✅ Auto on startup |
| In Git | ❌ Stored in DB only | ✅ Versioned |
| Team sharing | ❌ Everyone clicks | ✅ Everyone gets it |
| Disaster recovery | ❌ Reconfigure manually | ✅ Just redeploy |


---

### Task 6: Import a Community Dashboard
The Grafana community maintains thousands of pre-built dashboards. Import one for Node Exporter:

1. Go to Dashboards > New > Import
2. Enter dashboard ID: **1860** (Node Exporter Full)
3. Select your Prometheus datasource
4. Click Import

Explore the imported dashboard. It has dozens of panels covering CPU, memory, disk, network, and more -- all built on the same Node Exporter metrics you queried manually.

**Try another one:** Import dashboard ID **193** (Docker monitoring via cAdvisor). Select Prometheus as the datasource and explore container-level stats.

**Your full `docker-compose.yml` should now have these services:**
- `prometheus`
- `node-exporter`
- `cadvisor`
- `grafana`
- `notes-app` (from Day 73)

Verify all are running:
```bash
docker compose ps
```

---

---

### Task 6: Import a Community Dashboard
The Grafana community maintains thousands of pre-built dashboards. Import one for Node Exporter:

1. Go to Dashboards > New > Import
2. Enter dashboard ID: **1860** (Node Exporter Full)
3. Select your Prometheus datasource
4. Click Import

Explore the imported dashboard. It has dozens of panels covering CPU, memory, disk, network, and more -- all built on the same Node Exporter metrics you queried manually.

**Try another one:** Import dashboard ID **193** (Docker monitoring via cAdvisor). Select Prometheus as the datasource and explore container-level stats.

**Your full `docker-compose.yml` should now have these services:**
- `prometheus`
- `node-exporter`
- `cadvisor`
- `grafana`
- `notes-app` (from Day 73)

Verify all are running:
```bash
docker compose ps
```

---

Concept First:
The Grafana community has built thousands of pre-made dashboards. Instead of building from scratch, you import them by ID. The Node Exporter Full dashboard (ID 1860) is the industry standard — almost every DevOps team uses it.

imported 1860 :

<img width="1919" height="1093" alt="image" src="https://github.com/user-attachments/assets/73d8eedf-4658-4ed3-84ef-bd173121b219" />

Explore what you get — dozens of panels covering:

CPU usage per core
Memory breakdown (used, cached, free)
Disk I/O (reads/writes per second)
Network traffic
System load averages
File descriptor usage

This is what production monitoring looks like!

<img width="1919" height="1083" alt="image" src="https://github.com/user-attachments/assets/a53655ca-198a-4d93-884e-b0ebb4df976b" />

docker ps:

<img width="1908" height="328" alt="image" src="https://github.com/user-attachments/assets/636d9bf8-bcf0-4b53-8d3b-7f311a53e9eb" />

Full observability stack running!


Final, docker-compose.yaml:

      version: '3.8'
      
      services:
      
        # ── Prometheus — Metrics database ──────────────────────────────────
        prometheus:
          image: prom/prometheus:latest
          container_name: prometheus
          ports:
            - "9090:9090"
          volumes:
            - ./prometheus.yml:/etc/prometheus/prometheus.yml
            - prometheus_data:/prometheus
          command:
            - '--config.file=/etc/prometheus/prometheus.yml'
            - '--storage.tsdb.path=/prometheus'
            - '--web.console.libraries=/etc/prometheus/console_libraries'
            - '--web.console.templates=/etc/prometheus/consoles'
          restart: unless-stopped
      
        # ── Node Exporter — Host machine metrics ───────────────────────────
        node-exporter:
          image: prom/node-exporter:latest
          container_name: node-exporter
          ports:
            - "9100:9100"
          volumes:
            - /proc:/host/proc:ro
            - /sys:/host/sys:ro
            - /:/rootfs:ro
          command:
            - '--path.procfs=/host/proc'
            - '--path.sysfs=/host/sys'
            - '--path.rootfs=/rootfs'
            - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
          restart: unless-stopped
      
        # ── cAdvisor — Docker container metrics ────────────────────────────
        cadvisor:
          image: gcr.io/cadvisor/cadvisor:latest
          container_name: cadvisor
          ports:
            - "8080:8080"
          volumes:
            - /var/run/docker.sock:/var/run/docker.sock:ro
            - /sys:/sys:ro
            - /var/lib/docker/:/var/lib/docker:ro
          restart: unless-stopped
      
        # ── Grafana — Dashboards and visualization ─────────────────────────
        grafana:
          image: grafana/grafana-enterprise:latest
          container_name: grafana
          ports:
            - "3000:3000"
          volumes:
            - grafana_data:/var/lib/grafana
            - ./grafana/provisioning:/etc/grafana/provisioning
          environment:
            - GF_SECURITY_ADMIN_USER=admin
            - GF_SECURITY_ADMIN_PASSWORD=admin123
          depends_on:
            - prometheus
          restart: unless-stopped
      
      volumes:
        prometheus_data:
        grafana_data:


Final, prometheus.yaml:

    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    scrape_configs:
      - job_name: "prometheus"
        static_configs:
          - targets: ["localhost:9090"]
    
      - job_name: "node-exporter"
        static_configs:
          - targets: ["node-exporter:9100"]
    
      - job_name: "cadvisor"
        static_configs:
          - targets: ["cadvisor:8080"]


---

## Stack Architecture
    
    Prometheus :9090
    ├── scrapes Node Exporter :9100  → host CPU, RAM, disk, network
    ├── scrapes cAdvisor :8080       → per-container CPU, RAM, network
    └── scrapes itself :9090         → Prometheus health
    Grafana :3000
    └── queries Prometheus           → dashboards and visualization


## Node Exporter vs cAdvisor

| | Node Exporter | cAdvisor |
|---|---|---|
| Monitors | Linux host machine | Docker containers |
| Metrics prefix | `node_` | `container_` |
| Port | 9100 | 8080 |
| Use case | Server running out of RAM | Which container is eating RAM |

## Key PromQL Queries

| What | Query |
|---|---|
| CPU usage % | `100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)` |
| Memory usage % | `(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100` |
| Disk usage % | `(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100` |
| Container memory | `container_memory_usage_bytes{name!=""} / 1024 / 1024` |
| Container CPU | `rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100` |
| Top 3 by memory | `topk(3, container_memory_usage_bytes{name!=""})` |

## Dashboard Panels Built
1. CPU Usage % — Gauge with green/yellow/red thresholds
2. Memory Usage % — Gauge
3. Container CPU Usage — Time series with per-container lines
4. Container Memory — Bar chart in MB
5. Disk Usage % — Stat panel

## Community Dashboards Imported

| ID | Name | What it shows |
|---|---|---|
| 1860 | Node Exporter Full | Complete host metrics — industry standard |
| 193 | Docker cAdvisor | Per-container resource usage |

## YAML Provisioning vs Manual UI

| | Manual | YAML |
|---|---|---|
| Reproducible | ❌ | ✅ |
| In Git | ❌ | ✅ |
| Auto-configured | ❌ | ✅ |
| Team-friendly | ❌ | ✅ |

## Why container name not localhost for Grafana datasource
Grafana and Prometheus are in the same Docker network.
They communicate using container names as hostnames.
`http://prometheus:9090` works — `http://localhost:9090` does NOT.

### Summary Table:

| Concept | What it means |
|---|---|
| Node Exporter | Exposes Linux host metrics (CPU, RAM, disk) in Prometheus format |
| cAdvisor | Exposes Docker container metrics in Prometheus format |
| Grafana | Visualization tool — turns PromQL queries into dashboards |
| `node_` prefix | All Node Exporter metrics start with this |
| `container_` prefix | All cAdvisor metrics start with this |
| `{name!=""}` | Filter to exclude cAdvisor system entries, show only named containers |
| `topk(3, metric)` | Returns top 3 highest values — great for finding resource hogs |
| `rate()[5m]` | Per-second rate over 5 minute window — use for counters |
| Gauge panel | Shows current value with thresholds (good for CPU/memory %) |
| Time series panel | Shows value over time (good for trends) |
| Stat panel | Shows one big number (good for disk usage) |
| Dashboard ID 1860 | Node Exporter Full — industry standard host dashboard |
| Dashboard ID 193 | Docker cAdvisor — container monitoring dashboard |
| Provisioning YAML | Auto-configure Grafana datasources on startup — no clicking |
| `http://prometheus:9090` | Use container name, not localhost, inside Docker network |
