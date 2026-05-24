# Day 75 -- Log Management with Loki and Promtail

## Task
Metrics tell you _what_ is broken. Logs tell you _why_. Yesterday you built the metrics pipeline with Prometheus, Node Exporter, cAdvisor, and Grafana. Today you add the second pillar of observability -- logs.

You will set up Grafana Loki (a log aggregation system built by the Grafana team) and Promtail (the agent that ships logs to Loki). By the end of today, your Grafana instance will show both metrics and logs side by side.

---

What Are We Learning Today?

Yesterday you built the metrics pipeline. Today you add logs.

Metrics = WHAT is broken
Logs    = WHY it is broken

Real world example:

Prometheus alerts: "CPU spike at 2:47 AM" ← metrics
Loki shows: "Out of memory error in notes-app container at 2:47 AM" ← logs

Together: you know WHAT happened AND WHY

The full observability pipeline after today:

    Docker Containers
          ↓ writes JSON logs
    /var/lib/docker/containers/
          ↓ Promtail reads & ships
        Loki :3100
          ↓ stores logs by labels
        Grafana :3000
          ↓ queries with LogQL
          You

Why Loki instead of ELK (Elasticsearch)?

| | Loki | ELK Stack |
|---|---|---|
| What it indexes | Labels only | Full log text |
| Resource cost | Very low | Very high |
| Query speed | Fast for label searches | Fast for any text |
| Complexity | Simple | Complex |
| Like | Prometheus but for logs | Google for logs |

Loki's design philosophy: don't index everything, just index what you filter by (container name, job, filename). This makes it 10x cheaper than ELK.

---

## Challenge Tasks

### Task 1: Understand the Logging Pipeline
Before writing any config, understand how the pieces fit together:

```
[Docker Containers]
       |
       | (write JSON logs to /var/lib/docker/containers/)
       v
  [Promtail]
       |
       | (reads log files, adds labels, pushes to Loki)
       v
    [Loki]
       |
       | (stores logs, indexes by labels)
       v
   [Grafana]
       |
       | (queries Loki with LogQL, displays logs)
       v
   [You]
```

Key differences from the ELK stack:
- Loki does **not** index the full text of logs -- it only indexes labels (like container name, job, filename)
- This makes Loki much cheaper to run and simpler to operate
- Think of it as "Prometheus, but for logs" -- same label-based approach

**Document:** Why does Loki only index labels instead of full text? What is the trade-off?

---

Why does Loki only index labels, not full text?:

Indexing full log text is expensive — it requires building a massive inverted index (like a search engine) that consumes huge amounts of RAM and disk. Loki avoids this by only indexing metadata labels (container name, job, filename). When you search for a keyword, Loki first uses labels to find the right log streams, then scans only those lines for the keyword.
Trade-off:

✅ Much cheaper and simpler to run
✅ Same storage/query model as Prometheus (familiar to DevOps teams)
❌ Slower for full-text searches across all logs
❌ Not a replacement for Elasticsearch when you need complex text search

Concept First:

Loki is the log storage backend — it receives logs from Promtail, stores them compressed on disk, and answers LogQL queries from Grafana.

---

### Task 2: Add Loki to the Stack
Create the Loki configuration file.

```bash
mkdir -p loki
```

Create `loki/loki-config.yml`:
```yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory
  replication_factor: 1
  path_prefix: /loki

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

storage_config:
  filesystem:
    directory: /loki/chunks
```

**What this config does:**
- `auth_enabled: false` -- single-tenant mode, no authentication needed
- `store: tsdb` -- uses Loki's time-series database for indexing
- `object_store: filesystem` -- stores log chunks on local disk
- `replication_factor: 1` -- single instance, no replication (fine for learning)

Add Loki to your `docker-compose.yml`:
```yaml
  loki:
    image: grafana/loki:latest
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - ./loki/loki-config.yml:/etc/loki/loki-config.yml
      - loki_data:/loki
    command: -config.file=/etc/loki/loki-config.yml
    restart: unless-stopped
```

Add `loki_data` to your volumes section:
```yaml
volumes:
  prometheus_data:
  grafana_data:
  loki_data:
```

Start Loki:
```bash
docker compose up -d loki
```

Verify Loki is running:
```bash
curl http://localhost:3100/ready
```

You should see `ready`.

---

vim docker-compose.yaml:

<img width="950" height="659" alt="image" src="https://github.com/user-attachments/assets/77b1a738-d066-438a-975f-d1e1fa0c7732" />

vim loki/loki-config.yaml:

<img width="929" height="642" alt="image" src="https://github.com/user-attachments/assets/991e7df9-495c-458c-8faf-db5108e2a106" />

docker ps && curl hhtp://localhost:3100/ready:

<img width="1896" height="853" alt="image" src="https://github.com/user-attachments/assets/9a9429c2-fa1a-40aa-926f-4cf0f4d7d704" />

<img width="1091" height="116" alt="image" src="https://github.com/user-attachments/assets/4caa8ed8-aabb-4baa-8ce2-e103885cc0b7" />

Empty labels is fine — no logs ingested yet. That comes with Promtail.
✅ Verify: curl http://localhost:3100/ready returns ready

---

### Task 3: Add Promtail to Collect Container Logs
Promtail is the log collection agent. It reads Docker container log files from the host and pushes them to Loki.

```bash
mkdir -p promtail
```

Create `promtail/promtail-config.yml`:
```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: docker
    static_configs:
      - targets:
          - localhost
        labels:
          job: docker
          __path__: /var/lib/docker/containers/*/*-json.log
    pipeline_stages:
      - docker: {}
```

**What this config does:**
- `positions` -- tracks which log lines have already been shipped (like a bookmark)
- `clients` -- where to send logs (Loki endpoint)
- `__path__` -- the glob pattern to find Docker JSON log files on the host
- `pipeline_stages: docker: {}` -- parses the Docker JSON log format and extracts timestamp, stream (stdout/stderr), and the log message

Add Promtail to your `docker-compose.yml`:
```yaml
  promtail:
    image: grafana/promtail:latest
    container_name: promtail
    volumes:
      - ./promtail/promtail-config.yml:/etc/promtail/promtail-config.yml
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock
    command: -config.file=/etc/promtail/promtail-config.yml
    restart: unless-stopped
```

**Why these volume mounts?**
- `/var/lib/docker/containers` -- where Docker stores container log files (read-only)
- `/var/run/docker.sock` -- lets Promtail discover container metadata (names, labels)

Restart the stack:
```bash
docker compose up -d
```

Generate some logs by hitting the notes app:
```bash
for i in $(seq 1 20); do curl -s http://localhost:8000 > /dev/null; done
```

---

 Concept First:
 
Promtail is the log collection agent — it runs as a container, reads Docker log files from the host, adds metadata labels, and pushes them to Loki.

    /var/lib/docker/containers/
      <container-id>/
        <container-id>-json.log    ← Docker writes logs here as JSON
                    ↑
               Promtail reads these
               
Each line in the Docker JSON log file looks like:

    {"log":"GET /api/notes HTTP/1.1\n","stream":"stdout","time":"2026-05-23T10:00:00Z"}

The pipeline_stages: docker: {} step parses this format automatically.

vim promtail/promtail-config.yml:

<img width="1160" height="524" alt="image" src="https://github.com/user-attachments/assets/641a7d4c-6cf6-409f-88fc-2b1ab8fde7e6" />

vim docker-compose.yaml:

<img width="1095" height="641" alt="image" src="https://github.com/user-attachments/assets/1b08eb19-ab0b-42e1-8bdc-89d5e0c02006" />

added promtail details into docker compose.

docker ps and curl http://localhost:9090/targets:

<img width="1897" height="1104" alt="image" src="https://github.com/user-attachments/assets/e15bdeec-db7b-4a76-bf6b-d094d7e3c74b" />

generated logs and check if its generating:

<img width="1902" height="1064" alt="image" src="https://github.com/user-attachments/assets/ac196c21-6ca8-4e3a-9de0-408ff5f2135b" />

Verified : Promtail running, targeting Docker log files, no errors in logs.

---

### Task 4: Add Loki as a Grafana Datasource
You can add it manually through the UI or auto-provision it with YAML.

**Option A -- Provision via YAML (recommended):**

Update `grafana/provisioning/datasources/datasources.yml`:
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false

  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    editable: false
```

Restart Grafana to pick up the new datasource:
```bash
docker compose restart grafana
```

**Option B -- Manual UI setup:**
1. Go to Connections > Data Sources > Add data source
2. Select Loki
3. URL: `http://loki:3100`
4. Save & Test

Either way, you should now have two datasources in Grafana: Prometheus and Loki.

---

vim grafana/provisioning/datasources/datasources.yml:

<img width="762" height="395" alt="image" src="https://github.com/user-attachments/assets/e3dee9ca-1df0-45c9-921d-b1fa4d1cc8b9" />

docker compose restart graphana:

<img width="1839" height="312" alt="image" src="https://github.com/user-attachments/assets/5f3d0339-e902-428e-b1c7-851213d4cb4a" />

checked loki was already there:

<img width="1703" height="550" alt="image" src="https://github.com/user-attachments/assets/34b59a80-901d-4529-ad11-236e6040d812" />

---

### Task 5: Query Logs with LogQL
LogQL is Loki's query language -- similar to PromQL but for logs.

Go to Grafana > Explore (compass icon). Select Loki as the datasource.

1. **Stream selector** -- filter logs by labels:
```logql
{job="docker"}
```
This shows all Docker container logs.

2. **Filter by container name:**
```logql
{container_name="prometheus"}
```

3. **Keyword search** -- filter log lines by content:
```logql
{job="docker"} |= "error"
```
`|=` means "line contains". This finds all log lines with the word "error".

4. **Negative filter:**
```logql
{job="docker"} != "health"
```
Excludes lines containing "health" (useful to filter out health check noise).

5. **Regex filter:**
```logql
{job="docker"} |~ "status=[45]\\d{2}"
```
Finds lines with HTTP 4xx or 5xx status codes.

6. **Log metric queries** -- count log lines over time:
```logql
count_over_time({job="docker"}[5m])
```

7. **Rate of logs per second:**
```logql
rate({job="docker"}[5m])
```

8. **Top containers by log volume:**
```logql
topk(5, sum by (container_name) (rate({job="docker"}[5m])))
```

**Exercise:** Write a LogQL query that finds all error logs from the notes-app container in the last 1 hour. Then write another query that counts how many error lines per minute.

---

Concept First: — LogQL Syntax
LogQL has two parts:

Stream selector {label="value"} — which logs to look at (required)
Filter expression |= "text" — what to search for (optional)


    {container_name="prometheus"}     |=    "error"
           ↑                                  ↑
      Stream selector              Filter expression
      (which container)            (which log lines)

graphana UI > Explore > select datasource as loki > run this query:

    {job="docker"}
    
<img width="1919" height="1140" alt="image" src="https://github.com/user-attachments/assets/6427fda1-d587-43fc-8f0c-5724d16fcfea" />

Shows every log line from every container. You'll see logs from Prometheus, Grafana, cAdvisor, etc.

    {job="docker"} |= "grafana":
    {job="docker"} |= "prometheus"
    
<img width="1919" height="1060" alt="image" src="https://github.com/user-attachments/assets/5be15b51-b7d3-469a-8cae-c7542914f95f" />

<img width="1738" height="1065" alt="image" src="https://github.com/user-attachments/assets/f17a6fe5-fb8d-4bdb-82ff-963a94b14825" />

<img width="1697" height="1111" alt="image" src="https://github.com/user-attachments/assets/531448d6-2158-48d2-8f81-c1b0ecf4f4c3" />

<img width="1740" height="1069" alt="image" src="https://github.com/user-attachments/assets/82f8e934-6f10-46b4-98f1-0261a2e98775" />

Returns a number: how many log lines in each 5-minute window.:

<img width="1715" height="896" alt="image" src="https://github.com/user-attachments/assets/a969b076-f16a-4e6f-bba1-95a8363d2d54" />

Shows how fast logs are being generated.:

<img width="1625" height="955" alt="image" src="https://github.com/user-attachments/assets/c1ff58ed-5432-4fee-9f22-a7f6d69f2e8b" />

Which container is generating the most logs? Useful to spot a runaway container:

<img width="1705" height="1054" alt="image" src="https://github.com/user-attachments/assets/dffb254e-3117-431e-8c5d-1fdbee3b43b7" />

Find error logs from notes-app in last 1 hour::

<img width="1641" height="1067" alt="image" src="https://github.com/user-attachments/assets/cff0432c-895a-4c0f-90b4-05cf52353045" />

Count errors per minute::

<img width="1601" height="1068" alt="image" src="https://github.com/user-attachments/assets/437665ec-7d1a-4f49-843d-e126b32de612" />

|= means "line contains" — finds all lines with the word "error".

Troubleshooting — if you see no logs:

    # Check Promtail targets
    curl http://localhost:9080/targets
    
    # Check if Loki has received any labels
    curl http://localhost:3100/loki/api/v1/labels
    
    # Check Promtail logs for errors
    docker logs promtail --tail 30
    
    # Make sure the log path exists
    ls /var/lib/docker/containers/

---

### Task 6: Correlate Metrics and Logs in Grafana
The real power of observability is correlation -- seeing metrics and logs together.

1. **Add a logs panel to your dashboard:**
   - Open the dashboard you built on Day 74
   - Add a new panel
   - Select Loki as the datasource
   - Query: `{job="docker"}`
   - Visualization: Logs
   - Title: "Container Logs"

2. **Use the Explore split view:**
   - Go to Explore
   - Click the split button (two panels side by side)
   - Left panel: Prometheus -- `rate(container_cpu_usage_seconds_total{name="notes-app"}[5m])`
   - Right panel: Loki -- `{container_name="notes-app"}`
   - Now you can see CPU spikes and the corresponding log output at the same time

3. **Time sync:** Click on a spike in the metrics graph and both panels will zoom to that time range. This is how you debug in production -- you see a metric anomaly and immediately check the logs from that exact moment.

**Document:** How does having metrics and logs in the same tool (Grafana) help during incident response compared to checking separate systems?

---

Concept First:

This is the most powerful feature — seeing metrics AND logs together at the same time in the same tool. In production this is how you debug:

Prometheus alert fires: "CPU spike at 2:47 AM"
Open Grafana Explore split view
Left panel: CPU metric spike visible at 2:47 AM
Right panel: logs from that exact container at that exact time
You immediately see: "OutOfMemoryError" at 2:47 AM

No switching between tools. No manual timestamp correlation. Just click on the spike.

added visualization : added data source as loki : and executed query ( {job="docker"} ) 

<img width="1910" height="1056" alt="image" src="https://github.com/user-attachments/assets/017f5f24-472b-4ad0-80f7-ce95040173b3" />

Grafana UI > Explore > split tab > one datasource ( Prometheus ) query (rate(container_cpu_usage_seconds_total{}[5m]) * 100) 
Grafana UI > Explore > split tab > one datasource ( Loki ) query ({job="docker"})

<img width="1634" height="996" alt="image" src="https://github.com/user-attachments/assets/653b9486-8950-406b-8888-740ec9eef2ff" />

In the Prometheus panel, click on a spike or drag to select a time range. Both panels will zoom to that same time window.
Now you can see:

WHAT happened (CPU spike in metrics panel)
WHY it happened (log lines from that exact moment in logs panel)

<img width="1619" height="1005" alt="image" src="https://github.com/user-attachments/assets/34f735ce-2e68-4f5d-a055-38e340d9e5c5" />

### Final docker-compose.yml

     version: '3.8'
    
    services:
      # ── Prometheus ───────────────────────────────────────────────────────
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
    
      # ── Node Exporter — Host machine metrics ─────────────────────────────
      node-exporter:
        image: prom/node-exporter:latest
        container_name: node-exporter
        ports:
          - "9100:9100"
        volumes:
          - /proc:/host/proc:ro       # CPU, process info from kernel
          - /sys:/host/sys:ro         # Hardware and driver details
          - /:/rootfs:ro              # Filesystem usage (disk space)
        command:
          - '--path.procfs=/host/proc'
          - '--path.sysfs=/host/sys'
          - '--path.rootfs=/rootfs'
          - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
        restart: unless-stopped
    
      # ── cAdvisor — Docker container metrics ───────────────────────────────
      cadvisor:
        image: gcr.io/cadvisor/cadvisor:latest
        container_name: cadvisor
        ports:
          - "8080:8080"
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock:ro   # Discover containers
          - /sys:/sys:ro                                    # cgroup stats
          - /var/lib/docker/:/var/lib/docker:ro            # Container filesystem
        restart: unless-stopped
    
    
      # ── Grafana — Visualization and dashboards ─────────────────────────
      grafana:
        image: grafana/grafana-enterprise:latest
        container_name: grafana
        ports:
          - "3000:3000"
        volumes:
          - grafana_data:/var/lib/grafana                    # Persistent data
          - ./grafana/provisioning:/etc/grafana/provisioning # Auto-config (Task 5)
        environment:
          - GF_SECURITY_ADMIN_USER=admin
          - GF_SECURITY_ADMIN_PASSWORD=admin123
        restart: unless-stopped
        depends_on:
          - prometheus
    
      # ── Loki — Log storage backend ─────────────────────────────────────
      loki:
        image: grafana/loki:latest
        container_name: loki
        ports:
          - "3100:3100"
        volumes:
          - ./loki/loki-config.yml:/etc/loki/loki-config.yml
          - loki_data:/loki
        command: -config.file=/etc/loki/loki-config.yml
        restart: unless-stopped
    
      # ── Promtail — Log collection agent ────────────────────────────────
      promtail:
        image: grafana/promtail:latest
        container_name: promtail
        volumes:
          - ./promtail/promtail-config.yml:/etc/promtail/promtail-config.yml
          - /var/lib/docker/containers:/var/lib/docker/containers:ro    # Read Docker logs
          - /var/run/docker.sock:/var/run/docker.sock                   # Discover containers
        command: -config.file=/etc/promtail/promtail-config.yml
        restart: unless-stopped
        depends_on:
          - loki
    
    volumes:
      prometheus_data:
      grafana_data:
      loki_data:


---

## Architecture

    Docker Containers
    ↓ writes JSON logs to /var/lib/docker/containers/
    Promtail :9080
    ↓ reads, labels, pushes logs
    Loki :3100
    ↓ stores compressed, indexes by labels
    Grafana :3000
    ↓ queries with LogQL, shows alongside metrics
    You


## Why Loki Instead of ELK

| | Loki | ELK Stack |
|---|---|---|
| Indexes | Labels only | Full log text |
| Resource cost | Very low | Very high |
| Query language | LogQL | Lucene/KQL |
| Best for | Cloud-native, container logs | Full-text search |

## Why Loki Only Indexes Labels
Full-text indexing is expensive — it builds a massive inverted index
like a search engine. Loki only indexes labels (container_name, job)
then scans matching log streams for keywords. Trade-off: cheaper and
simpler but slower for full-text search than Elasticsearch.

## Key Config Files

### loki-config.yml
- auth_enabled: false → single-tenant, no auth
- store: tsdb → time-series index
- object_store: filesystem → logs on local disk
- replication_factor: 1 → single instance

### promtail-config.yml
- positions.yaml → bookmark tracking what's been sent
- __path__ glob → finds all Docker JSON log files
- pipeline_stages: docker → parses Docker JSON format

## LogQL Queries

| Query | What it does |
|---|---|
| `{job="docker"}` | All container logs |
| `{container_name="prometheus"}` | One container's logs |
| `{job="docker"} \|= "error"` | Lines containing "error" |
| `{job="docker"} != "health"` | Exclude health check noise |
| `{job="docker"} \|~ "(?i)error"` | Case-insensitive error search |
| `rate({job="docker"}[5m])` | Log lines per second |
| `topk(5, sum by (container_name)(rate({job="docker"}[5m])))` | Busiest containers |

## Metrics + Logs Correlation
Grafana Explore split view: Prometheus on left, Loki on right.
Click on a CPU spike → both panels zoom to that time window.
See WHAT happened (metric) and WHY (log lines) without switching tools.

## How Metrics + Logs Together Help Incident Response
Single tool instead of switching between Grafana and Kibana/Splunk.
Time-synced views — click a metric spike, logs jump to that exact moment.
No manual timestamp correlation.
Faster MTTR (Mean Time To Resolution).

 Summary Table:

 | Concept | What it means |
|---|---|
| Loki | Log storage backend — like Prometheus but for logs |
| Promtail | Log collection agent — reads Docker logs and ships to Loki |
| LogQL | Loki's query language — like PromQL but for logs |
| Stream selector `{}` | Which logs to look at — required in every LogQL query |
| `\|=` | Line contains this string |
| `!=` | Line does NOT contain this string |
| `\|~` | Line matches this regex |
| Labels | Metadata attached to log streams — container_name, job |
| positions.yaml | Promtail's bookmark — tracks which log lines already sent |
| `rate()[5m]` | Log lines per second over 5 minute window |
| `count_over_time()[5m]` | Total log lines in each 5 minute window |
| `topk(5, ...)` | Top 5 highest values — find busiest containers |
| `http://loki:3100` | Use container name not localhost in Grafana |
| `curl localhost:3100/ready` | Health check — should return "ready" |
| Explore split view | Side-by-side metrics + logs for incident correlation |
| YAML provisioning | Auto-configure Loki datasource on Grafana startup |
