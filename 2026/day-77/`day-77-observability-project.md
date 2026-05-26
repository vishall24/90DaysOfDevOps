#  Day 77 -- Observability Project: Full Stack with Docker Compose

## Task
Four days of building -- Prometheus, Node Exporter, cAdvisor, Grafana, Loki, Promtail, OpenTelemetry Collector, and alerting. Today you put it all together using a production-ready reference architecture.

You will clone the observability-for-devops reference repo, spin up the complete 8-service stack in one command, validate every data flow end to end, build a unified dashboard, and document the entire setup as if you were handing it off to a teammate.

---

What Are We Doing Today?
Over the last 4 days you built pieces one by one:

Day 73: Prometheus
Day 74: Node Exporter, cAdvisor, Grafana
Day 75: Loki, Promtail
Day 76: OTEL Collector, Alerting

Today you:

Clone a reference repo that has everything pre-wired
Spin up all 8 services with ONE command
Validate every pipeline (metrics, logs, traces)
Build a unified "Production Overview" dashboard
Compare your configs with the reference to see what you learned

The full stack:

    ┌──────────────────────────────────────────────────────┐
    │  METRICS PIPELINE                                    │
    │  Node Exporter :9100 ──┐                            │
    │  cAdvisor :8080 ───────┼──► Prometheus :9090        │
    │  OTEL Collector :8889 ─┘         │                  │
    │                                  ▼                  │
    │  LOGS PIPELINE              Grafana :3000           │
    │  Docker containers ──► Promtail ──► Loki :3100 ─────┤
    │                                  │                  │
    │  TRACES PIPELINE                 ▼                  │
    │  curl/App ──► OTEL Collector ──► Debug/Console      │
    │                                                      │
    │  Notes App :8000 (generates real traffic)            │
    └──────────────────────────────────────────────────────┘

---

## Challenge Tasks

### Task 1: Clone and Launch the Reference Stack
Clone the reference repository that contains the complete observability setup:

```bash
git clone https://github.com/LondheShubham153/observability-for-devops.git
cd observability-for-devops
```

Examine the project structure:
```bash
tree -I 'node_modules|build|staticfiles|__pycache__'
```

```
observability-for-devops/
  docker-compose.yml                    # 8 services orchestrated together
  prometheus.yml                        # Prometheus scrape configuration
  alert-rules.yml                       # (you will add this)
  grafana/
    provisioning/
      datasources/datasources.yml       # Auto-provisioned: Prometheus + Loki
      dashboards/dashboards.yml         # Dashboard provisioning config
  loki/
    loki-config.yml                     # Loki storage and schema config
  promtail/
    promtail-config.yml                 # Docker log collection config
  otel-collector/
    otel-collector-config.yml           # OTLP receivers, processors, exporters
  notes-app/                            # Sample Django + React application
```

Launch the entire stack:
```bash
docker compose up -d
```

Wait for all containers to start:
```bash
docker compose ps
```

All 8 services should show as running:

| Service | Port | Check |
|---------|------|-------|
| Prometheus | 9090 | `http://localhost:9090` |
| Node Exporter | 9100 | `curl http://localhost:9100/metrics \| head -5` |
| cAdvisor | 8080 | `http://localhost:8080` |
| Grafana | 3000 | `http://localhost:3000` (admin/admin) |
| Loki | 3100 | `curl http://localhost:3100/ready` |
| Promtail | 9080 | Internal only |
| OTEL Collector | 4317/4318 | `docker logs otel-collector` |
| Notes App | 8000 | `http://localhost:8000` |

---

Concept First:
Instead of manually wiring everything, the reference repo has a production-ready setup. You'll learn from it by comparing it to what you built.

     tree -I 'node_modules|build|staticfiles|__pycache__':
 
<img width="731" height="1074" alt="image" src="https://github.com/user-attachments/assets/519d2b54-28c9-40f2-a743-fae214825d8b" />

    cat docker-compose.yaml:
    
<img width="643" height="1086" alt="image" src="https://github.com/user-attachments/assets/2476afa9-a099-41ed-b78c-8b6869f13458" />

docker ps:

<img width="1694" height="146" alt="image" src="https://github.com/user-attachments/assets/fd0992d7-aec2-48cf-aeba-11245126f7cd" />

<img width="1894" height="523" alt="image" src="https://github.com/user-attachments/assets/5a40e32b-e7f2-42c2-a309-3e1f4267681d" />

     docker exec -it prometheus wget -qO- http://node-exporter:9100/metrics | head:
 
<img width="1893" height="266" alt="image" src="https://github.com/user-attachments/assets/108e0af6-17a5-4395-b943-f3410f616495" />

    Here we are checking the metrics from inside prometheus since we have not exported the port of node-exporter so EC2 instance will be not be able to access it.

<img width="1913" height="115" alt="image" src="https://github.com/user-attachments/assets/c2a56ec8-84c7-43cf-a2b3-5c8eadc70a46" />

<img width="1019" height="78" alt="image" src="https://github.com/user-attachments/assets/a7e6282a-d8b8-4fb5-a211-8efd4429efcb" />

    # Check logs of a specific service
    docker compose logs prometheus
    docker compose logs loki
    docker compose logs notes-app
    
    # Restart a specific service
    docker compose restart prometheus

 All 8 containers running, all health checks pass.

 ---

### Task 2: Validate the Metrics Pipeline
Confirm Prometheus is scraping all targets:

1. Open `http://localhost:9090/targets`
2. Verify all 4 scrape jobs are UP:
   - `prometheus` (self-monitoring)
   - `node-exporter` (host metrics)
   - `docker` / `cadvisor` (container metrics)
   - `otel-collector` (OTLP metrics)

Run these validation queries:
```promql
# All targets are healthy
up

# Host CPU usage
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Container CPU per container
rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100

# Top 3 memory-hungry containers
topk(3, container_memory_usage_bytes{name!=""})
```

Compare the `prometheus.yml` from the reference repo with the one you built over days 73-76. Note the scrape jobs and intervals.

---

    http://<EC-2-Public-IP>:9090/targets:

<img width="1916" height="870" alt="image" src="https://github.com/user-attachments/assets/b283ace4-ec13-4c76-bbb2-7896e1b4603a" />

You should see ALL jobs showing UP:

If any shows DOWN, check the service is running and the job name in prometheus.yml.

<img width="1883" height="1039" alt="image" src="https://github.com/user-attachments/assets/38b6eb9e-4e07-43f4-ac84-39fbf48e82af" />

Should return 1 for every target. Any 0 = problem.

    100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100):

<img width="1889" height="982" alt="image" src="https://github.com/user-attachments/assets/9ccb44d4-5737-44cc-9b62-0ee9b8dec6c6" />

    (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100:
    
<img width="1919" height="977" alt="image" src="https://github.com/user-attachments/assets/d05355b7-f022-493a-a159-f13d26e88593" />

    rate(container_cpu_usage_seconds_total{id=~".*docker.*"}[5m]) * 100:

<img width="1892" height="1025" alt="image" src="https://github.com/user-attachments/assets/93b75f91-df71-4967-b69f-0634f0180b92" />

    topk(3, container_memory_usage_bytes{id=~".*docker.*"}):
    
<img width="1866" height="1039" alt="image" src="https://github.com/user-attachments/assets/0815e728-18d2-46b2-90ea-d55fe379feff" />

    cat prometheus.yaml:

<img width="826" height="451" alt="image" src="https://github.com/user-attachments/assets/6cdc6083-9135-4e87-8955-fae02662b647" />


Compare with what you built in Days 73-76. Note:

Are the job names the same? Yes
Are there extra scrape configs? No
What's the scrape_interval? 15s

one change is these lines in prometheus is missing:


rule_files:
  - /etc/prometheus/alert-rules.yml

without this the rules wont appear in the dashboard.

Verify: All 4 jobs UP, queries returning data.

---

### Task 3: Validate the Logs Pipeline
Generate traffic so there are logs to see:

```bash
for i in $(seq 1 50); do
  curl -s http://localhost:8000 > /dev/null
  curl -s http://localhost:8000/api/ > /dev/null
done
```

Open Grafana (`http://localhost:3000`) and go to Explore:

1. Select Loki as the datasource
2. Run these LogQL queries:

```logql
# All container logs
{job="docker"}

# Only notes-app logs
{container_name="notes-app"}

# Errors across all containers
{job="docker"} |= "error"

# HTTP request logs from the app
{container_name="notes-app"} |= "GET"

# Rate of log lines per container
sum by (container_name) (rate({job="docker"}[5m]))
```

Check Promtail's targets to see which log files it is watching:
```bash
curl -s http://localhost:9080/targets | head -30
```

Compare `promtail/promtail-config.yml` from the reference repo with yours from Day 75.

---

<img width="1914" height="1037" alt="image" src="https://github.com/user-attachments/assets/64472c24-6604-4c11-b14b-c8a4968357f4" />

<img width="1919" height="1028" alt="image" src="https://github.com/user-attachments/assets/8a4d046b-16dd-45b0-94ce-269929fda341" />

here using {job="docker"} |= "notes-app" , container_name label does not exist in Loki labels, Because Promtail is currently NOT extracting Docker container metadata as Loki labels.

Docker logs → Promtail → Loki

<img width="1919" height="1086" alt="image" src="https://github.com/user-attachments/assets/df0352aa-fa8d-4235-9dfd-5fef4206826c" />

<img width="1889" height="1042" alt="image" src="https://github.com/user-attachments/assets/ccdcd469-9e5d-468e-8460-3a5cc8d0b6f2" />

<img width="1903" height="1043" alt="image" src="https://github.com/user-attachments/assets/bcd65b33-cad9-4a7a-9784-8f2eaf52ad7c" />

everything inthe promtail.yaml as compared to previous day is same except thi line   

grpc_listen_port: 0


---

### Task 4: Validate the Traces Pipeline
Send OTLP traces to the collector:

```bash
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{
    "resourceSpans": [{
      "resource": {
        "attributes": [{
          "key": "service.name",
          "value": { "stringValue": "notes-app" }
        }]
      },
      "scopeSpans": [{
        "spans": [{
          "traceId": "aaaabbbbccccdddd1111222233334444",
          "spanId": "1111222233334444",
          "name": "GET /api/notes",
          "kind": 2,
          "startTimeUnixNano": "1700000000000000000",
          "endTimeUnixNano": "1700000000150000000",
          "attributes": [{
            "key": "http.method",
            "value": { "stringValue": "GET" }
          },
          {
            "key": "http.route",
            "value": { "stringValue": "/api/notes" }
          },
          {
            "key": "http.status_code",
            "value": { "intValue": "200" }
          }],
          "status": { "code": 1 }
        },
        {
          "traceId": "aaaabbbbccccdddd1111222233334444",
          "spanId": "5555666677778888",
          "parentSpanId": "1111222233334444",
          "name": "SELECT notes FROM database",
          "kind": 3,
          "startTimeUnixNano": "1700000000020000000",
          "endTimeUnixNano": "1700000000120000000",
          "attributes": [{
            "key": "db.system",
            "value": { "stringValue": "sqlite" }
          },
          {
            "key": "db.statement",
            "value": { "stringValue": "SELECT * FROM notes" }
          }]
        }]
      }]
    }]
  }'
```

This simulates a two-span trace: an HTTP request that calls a database query.

Check the debug output:
```bash
docker logs otel-collector 2>&1 | grep -A 20 "GET /api/notes"
```

You should see both spans with their attributes, the parent-child relationship, and timing data.

Compare `otel-collector/otel-collector-config.yml` from the reference repo with yours from Day 76.

---

sent a two-span trace
This simulates a real HTTP request that queries a database — two linked spans in one trace:

<img width="1110" height="1076" alt="image" src="https://github.com/user-attachments/assets/05fcb181-bd26-4ef6-9f4a-dfd2d788f6f3" />

     docker logs otel-collector 2>&1 | grep -A 30 "GET /api/notes":

<img width="1911" height="676" alt="image" src="https://github.com/user-attachments/assets/3b1f6854-f30b-4b65-bcb2-586a24077749" />

    here I have changed the verbosity: basic to verbosity: detailed in the otel-collector.yml, because in basic you wont get "/api/notes" in the otel logs.

Notice the parent-child relationship — parentSpanId links the DB query to the HTTP request. This is how you trace a request through multiple services! ✅


cat otel-collector:

<img width="1084" height="667" alt="image" src="https://github.com/user-attachments/assets/7e47f4dc-3c7d-460d-bc8c-25429ad64c35" />

---

### Task 5: Build a Unified "Production Overview" Dashboard
Create a single Grafana dashboard that gives a complete picture of your system.

Go to Dashboards > New Dashboard. Add these panels:

**Row 1 -- System Health (Node Exporter + Prometheus):**

| Panel | Type | Query |
|-------|------|-------|
| CPU Usage | Gauge | `100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)` |
| Memory Usage | Gauge | `(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100` |
| Disk Usage | Gauge | `(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100` |
| Targets Up | Stat | `sum(up)` / `count(up)` |

**Row 2 -- Container Metrics (cAdvisor):**

| Panel | Type | Query |
|-------|------|-------|
| Container CPU | Time series | `rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100` (legend: `{{name}}`) |
| Container Memory | Bar chart | `container_memory_usage_bytes{name!=""} / 1024 / 1024` (legend: `{{name}}`) |
| Container Count | Stat | `count(container_last_seen{name!=""})` |

**Row 3 -- Application Logs (Loki):**

| Panel | Type | Query (Loki datasource) |
|-------|------|-------|
| App Logs | Logs | `{container_name="notes-app"}` |
| Error Rate | Time series | `sum(rate({job="docker"} \|= "error" [5m]))` |
| Log Volume | Time series | `sum by (container_name) (rate({job="docker"}[5m]))` |

**Row 4 -- Service Overview:**

| Panel | Type | Query |
|-------|------|-------|
| Prometheus Scrape Duration | Time series | `prometheus_target_interval_length_seconds{quantile="0.99"}` |
| OTEL Metrics Received | Stat | `otelcol_receiver_accepted_metric_points` (if available) |

Save the dashboard as "Production Overview -- Observability Stack".

Set the dashboard time range to "Last 30 minutes" and enable auto-refresh (every 10s).

---

 Concept First:
 
A good production dashboard shows everything at a glance:

Is the server healthy? (CPU, RAM, Disk)
Are containers behaving? (per-container CPU, memory)
Are there errors? (log error rate)
Is everything being monitored? (targets up count)


Step 1: Create a new dashboard
Go to: Grafana → Dashboards → New Dashboard

Step 2: Add Row 1 — System Health
Panel 1: CPU Usage Gauge

Visualization: Gauge
Datasource: Prometheus
Query: 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
Title: CPU Usage %
Thresholds: green < 60, yellow < 80, red ≥ 80
Unit: Percent (0-100)

<img width="1679" height="895" alt="image" src="https://github.com/user-attachments/assets/c304abdc-d567-4f4b-b6e5-b1a910f653c5" />

<img width="1828" height="628" alt="image" src="https://github.com/user-attachments/assets/835afc24-baba-4d75-a5ef-efe75f25362c" />

Panel 2: Memory Usage Gauge

Visualization: Gauge
Datasource: Prometheus
Query: (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
Title: Memory Usage %
Same thresholds

<img width="1661" height="577" alt="image" src="https://github.com/user-attachments/assets/50b2b259-21cc-4cbc-8848-682f34f7c876" />

Panel 3: Disk Usage Gauge

Visualization: Gauge
Datasource: Prometheus
Query: (1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100
Title: Disk Usage %

<img width="1656" height="554" alt="image" src="https://github.com/user-attachments/assets/80d7f157-336b-4ff2-8e4e-deef042b6de4" />

Panel 4: Targets Up Stat

Visualization: Stat
Datasource: Prometheus
Query A: sum(up)
Query B: count(up)
Title: Targets Healthy
Value: A/B (shows "4/4" or however many are up)

<img width="1651" height="798" alt="image" src="https://github.com/user-attachments/assets/f5e844a4-509a-45d4-906a-d7a380f793ff" />

Step 3: Add Row 2 — Container Metrics

Panel 5: Container CPU Time Series

Visualization: Time series
Datasource: Prometheus
Query: rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100
Title: Container CPU Usage %
Legend: {{name}}

<img width="921" height="369" alt="image" src="https://github.com/user-attachments/assets/ca9f265c-c506-4c29-b712-4c028bfa3d77" />

Panel 6: Container Memory Bar Chart

Visualization: Bar chart
Datasource: Prometheus
Query: container_memory_usage_bytes{name!=""} / 1024 / 1024
Title: Container Memory (MB)
Legend: {{name}}

<img width="1670" height="399" alt="image" src="https://github.com/user-attachments/assets/fb843706-d2cf-4909-9ea1-f083858c9e21" />

Panel 7: Container Count Stat

Visualization: Stat
Datasource: Prometheus
Query: count(container_last_seen{name!=""})
Title: Running Containers

<img width="1603" height="761" alt="image" src="https://github.com/user-attachments/assets/3a149d30-240f-418c-a9e7-2c5fa92c3634" />

Step 4: Add Row 3 — Application Logs (switch datasource to Loki!)

Panel 8: App Logs Panel

Visualization: Logs
Datasource: Loki ← important, not Prometheus!
Query: {container_name="notes-app"}
Title: Notes App Logs


Panel 9: Error Rate Time Series

Visualization: Time series
Datasource: Loki
Query: sum(rate({job="docker"} |= "error" [5m]))
Title: Error Rate (errors/sec)

Panel 10: Log Volume per Container

Visualization: Time series
Datasource: Loki
Query: sum by (container_name) (rate({job="docker"}[5m]))
Title: Log Volume by Container
Legend: {{container_name}}

Step 5: Save and configure

Click Save dashboard
Name: Production Overview — Observability Stack
Set time range: Last 30 minutes
Enable auto-refresh: 10s


<img width="1593" height="837" alt="image" src="https://github.com/user-attachments/assets/c5d414a0-3427-4c88-9bd8-18f2ffe1f072" />

<img width="1562" height="125" alt="image" src="https://github.com/user-attachments/assets/6421296d-61bb-46b5-a99f-ec42b42b89cc" />

<img width="1689" height="930" alt="image" src="https://github.com/user-attachments/assets/76e62d94-ae5a-4a68-bfb7-8c9e8cbf7421" />

---

### Task 6: Compare Your Stack with the Reference and Document
Now compare what you built over days 73-76 with the reference repository.

| Component | Your Version | Reference Repo | Differences |
|-----------|-------------|----------------|-------------|
| `prometheus.yml` | Day 73-74 | Root directory | Compare scrape jobs |
| `loki-config.yml` | Day 75 | `loki/` directory | Compare storage config |
| `promtail-config.yml` | Day 75 | `promtail/` directory | Compare scrape configs |
| `otel-collector-config.yml` | Day 76 | `otel-collector/` directory | Compare pipelines |
| `datasources.yml` | Day 74 | `grafana/provisioning/` | Compare provisioned sources |
| `docker-compose.yml` | Days 73-76 | Root directory | Compare all 8 services |

**Reflect and document:**

1. Map each observability concept to the day you learned it:

| Day | What You Built |
|-----|---------------|
| 73 | Prometheus, PromQL, metrics fundamentals |
| 74 | Node Exporter, cAdvisor, Grafana dashboards |
| 75 | Loki, Promtail, LogQL, log-metric correlation |
| 76 | OTEL Collector, traces, alerting rules |
| 77 | Full stack integration, unified dashboard |

2. What would you add for production?
   - Alertmanager for routing alerts to Slack/PagerDuty
   - Grafana Tempo for trace storage (replacing debug exporter)
   - HTTPS/TLS for all endpoints
   - Authentication on Grafana and Prometheus
   - Log retention policies and storage limits
   - High availability (multiple Prometheus/Loki replicas)

3. How does this stack compare to managed solutions like Datadog, New Relic, or AWS CloudWatch?

**Clean up when done:**
```bash
docker compose down -v
```

The `-v` flag removes named volumes (Prometheus data, Grafana data, Loki data). Only use this if you are done exploring.

---

1) mentioned difference above in tasks.


2) 
### Check what would be added for production:

Write these in your notes for the markdown file:

Alertmanager — routes alerts to Slack, PagerDuty, email based on severity labels
Grafana Tempo — replaces debug exporter for actual trace storage and visualization
HTTPS/TLS — all endpoints need SSL in production (use nginx as reverse proxy)
Authentication — Prometheus and Grafana need proper auth (not default admin/admin)
Log retention — Loki needs compaction rules so logs don't fill the disk
HA setup — multiple Prometheus replicas with Thanos for production scale

3) Done:

<img width="1391" height="444" alt="image" src="https://github.com/user-attachments/assets/0e26251d-c977-4e12-ade0-8413496561ea" />

---

## Architecture

    METRICS:
    Node Exporter :9100 ──┐
    cAdvisor :8080 ────────┼──► Prometheus :9090 ──► Grafana :3000
    OTEL Collector :8889 ──┘         │
    ▼
    LOGS:                       Alert Rules
    Docker → Promtail ──► Loki :3100 ──► Grafana :3000
    TRACES:
    curl/App ──► OTEL Collector :4318 ──► Debug Console
    Notes App :8000 → generates real traffic for all pipelines



## Services Running

| Service | Port | Purpose |
|---|---|---|
| Prometheus | 9090 | Metrics storage and querying |
| Node Exporter | 9100 | Host CPU, RAM, disk, network |
| cAdvisor | 8080 | Docker container metrics |
| Grafana | 3000 | Dashboards, alerts, explore |
| Loki | 3100 | Log storage |
| Promtail | 9080 | Log collection agent |
| OTEL Collector | 4317/4318/8889 | Traces and metrics pipeline |
| Notes App | 8000 | Sample Django app for traffic |

## 5-Day Observability Block Summary

| Day | What I Built |
|---|---|
| 73 | Prometheus, PromQL, metrics fundamentals |
| 74 | Node Exporter, cAdvisor, Grafana dashboards |
| 75 | Loki, Promtail, LogQL, log-metric correlation |
| 76 | OTEL Collector, traces, alerting rules |
| 77 | Full stack integration, unified dashboard |

## Unified Dashboard Panels Built
Row 1 — System Health: CPU gauge, Memory gauge, Disk gauge, Targets Up stat
Row 2 — Containers: Container CPU time series, Container memory bar, Container count
Row 3 — Logs: App logs panel, Error rate time series, Log volume by container

## Config Comparison: Mine vs Reference Repo

| File | Key differences noted |
|---|---|
| prometheus.yml | Reference has cleaner job names |
| loki-config.yml | Same storage config |
| promtail-config.yml | Reference has extra labels |
| otel-collector-config.yml | Same pipeline structure |
| datasources.yml | Reference pre-provisions both Prometheus and Loki |

## What I Would Add for Production
- Alertmanager — route alerts to Slack/PagerDuty by severity
- Grafana Tempo — replace debug exporter with real trace storage
- HTTPS/TLS — nginx reverse proxy in front of all endpoints
- Proper auth — no default admin/admin passwords
- Log retention — Loki compaction rules to manage disk
- High availability — multiple Prometheus replicas + Thanos

## Key Takeaways from 5-Day Block
- Metrics show WHAT is broken
- Logs show WHY it broke
- Traces show WHERE time is spent
- All three together = full observability
- One docker compose file can run the entire stack
- YAML provisioning > manual UI config (reproducible)

### Summary:

| Concept | What it means in this project |
|---|---|
| Full stack | All 8 services running from one docker compose up -d |
| Reference repo | Production-ready template to learn from and compare against |
| Metrics pipeline | Node Exporter + cAdvisor + OTEL → Prometheus → Grafana |
| Logs pipeline | Docker containers → Promtail → Loki → Grafana Explore |
| Traces pipeline | curl/App → OTEL Collector → Debug output (→ Tempo in prod) |
| Parent-child spans | parentSpanId links DB query span to HTTP request span |
| Production dashboard | Unified view of system health, containers, and logs in one place |
| `sum(up)` | Count of healthy targets — quick system health check |
| `topk(3, ...)` | Top 3 resource consumers — find who's eating your RAM |
| Log error rate | `sum(rate({job="docker"} \|= "error" [5m]))` — errors per second |
| `docker compose down -v` | Stops and deletes all data — only when fully done |
| Dashboard-as-code | Export JSON and save to Git for reproducible dashboards |
