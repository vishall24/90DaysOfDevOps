# Day 76 -- OpenTelemetry and Alerting

## Task
You have metrics (Prometheus) and logs (Loki). Today you add the third pillar -- traces -- using OpenTelemetry, the industry-standard framework for collecting telemetry data. Then you set up alerting so your system notifies you when something goes wrong, instead of you staring at dashboards all day.

By the end of today, your observability stack covers all three pillars and actively alerts on problems.

---

## Expected Output
- OpenTelemetry Collector running and exporting metrics to Prometheus
- OTLP traces sent to the collector and visible in debug output
- Prometheus alerting rules configured for critical conditions
- Grafana alert rules with notification contacts
- A markdown file: `day-76-otel-alerting.md`

---

What Are We Learning Today?:
You now have 2 of the 3 pillars of observability:

✅ Metrics — Prometheus, Node Exporter, cAdvisor
✅ Logs — Loki, Promtail
❌ Traces — missing! Adding today with OpenTelemetry

Plus today you add Alerting — so instead of staring at dashboards all day, the system tells YOU when something breaks.

What is OpenTelemetry?

| | What it is | Example |
|---|---|---|
| Prometheus | Metrics backend — stores and queries metrics | CPU usage over time |
| Loki | Log backend — stores and queries logs | Container error logs |
| OpenTelemetry | Collection framework — NOT a backend | Collects and ships telemetry to backends |

OpenTelemetry (OTEL) is like a universal translator for telemetry data. Your app sends one format (OTLP), OTEL collector converts it to whatever backend format you need.

What are Distributed Traces?

User clicks "Buy" on website
        ↓ Span 1 (10ms)
  API Gateway receives request
        ↓ Span 2 (5ms)
  Auth Service validates token
        ↓ Span 3 (50ms)
  Database query runs
        ↓ Span 4 (2ms)
  Response sent back

Total trace = 67ms across 4 services

Traces show you WHERE time is being spent across multiple services. 
Metrics tell you WHAT is slow. Traces tell you WHERE.

---

## Challenge Tasks

### Task 1: Understand OpenTelemetry
Research and write notes on:

1. **What is OpenTelemetry (OTEL)?**
   - A vendor-neutral, open-source framework for generating, collecting, and exporting telemetry data (metrics, logs, traces)
   - It is not a backend -- it collects and ships data to backends like Prometheus, Jaeger, Loki, Datadog

2. **What is the OTEL Collector?**
   - A standalone service that receives, processes, and exports telemetry
   - Three components in the pipeline:
     - **Receivers** -- accept data (OTLP, Prometheus, Jaeger formats)
     - **Processors** -- transform data (batching, filtering, sampling)
     - **Exporters** -- send data to backends (Prometheus, debug console, Jaeger)

3. **What is OTLP?**
   - OpenTelemetry Protocol -- the standard wire format for sending telemetry
   - Supports gRPC (port 4317) and HTTP (port 4318)

4. **What are distributed traces?**
   - A trace tracks a single request as it travels through multiple services
   - Each step in the trace is called a **span**
   - Spans have: trace ID, span ID, parent span ID, start time, duration, attributes
   - Example: User request -> API Gateway (span 1) -> Auth Service (span 2) -> Database (span 3)

---

OTEL Collector Pipeline:

App/curl
   ↓ sends OTLP data
[Receivers]        ← Accept data (OTLP, Prometheus, Jaeger formats)
   ↓
[Processors]       ← Transform data (batch, filter, sample)
   ↓
[Exporters]        ← Send to backends (Prometheus, Jaeger, debug console)

Three pillars summary:

| Pillar | Answers | Tool |
|---|---|---|
| Metrics | WHAT is broken and WHEN | Prometheus + Grafana |
| Logs | WHY it broke | Loki + Promtail + Grafana |
| Traces | WHERE it's slow | OTEL Collector + Jaeger/Tempo |

### 1. What is OpenTelemetry (OTEL)?

OpenTelemetry (OTEL) is a vendor-neutral, open-source observability framework used to generate, collect, process, and export telemetry data.

Telemetry data includes:
- Metrics
- Logs
- Traces

OTEL itself is NOT a monitoring backend or storage system.

Instead, it acts as a telemetry pipeline that sends data to observability backends such as:
- Prometheus
- Jaeger
- Loki
- Grafana
- Datadog
- New Relic

### Main goal of OTEL
Standardize observability across different programming languages, cloud platforms, and monitoring vendors.

---

### 2. What is the OpenTelemetry Collector?

The OpenTelemetry Collector is a standalone service that receives, processes, and exports telemetry data.

It acts as the central telemetry pipeline component.

### OTEL Collector Pipeline

```text
Application → Receiver → Processor → Exporter → Backend
````

### Core Components

| Component  | Purpose                          |
| ---------- | -------------------------------- |
| Receivers  | Accept telemetry data            |
| Processors | Transform/filter/batch data      |
| Exporters  | Send data to monitoring backends |

---

### Receivers

Receivers ingest telemetry data from applications or other systems.

Examples:

* OTLP
* Prometheus
* Jaeger
* Zipkin

---

### Processors

Processors modify or optimize telemetry data before exporting.

Examples:

* Batch processor
* Memory limiter
* Sampling
* Filtering

---

### Exporters

Exporters send telemetry data to external systems.

Examples:

* Prometheus
* Jaeger
* Loki
* Debug console
* Datadog

---

### 3. What is OTLP?

OTLP stands for OpenTelemetry Protocol.

It is the standard protocol used by OpenTelemetry to transmit telemetry data.

OTLP supports:

* Metrics
* Logs
* Traces

### OTLP Transport Protocols

| Protocol | Port |
| -------- | ---- |
| gRPC     | 4317 |
| HTTP     | 4318 |

---

### 4. What are Distributed Traces?

Distributed tracing tracks a single request as it travels through multiple services in a distributed system.

A complete request journey is called a:

* Trace

Each operation inside the trace is called a:

* Span

---

### Span Information

Each span contains:

* Trace ID
* Span ID
* Parent Span ID
* Start time
* End time
* Duration
* Attributes/metadata

---

### Example Distributed Trace

```text
User Request
   │
   ├── API Gateway Span
   │
   ├── Auth Service Span
   │
   └── Database Span
```

Example flow:

```text
User → API Gateway → Auth Service → Database
```

All spans together form one complete trace.

---

### Why Distributed Tracing is Important

Distributed tracing helps identify:

* Slow services
* Bottlenecks
* Failed requests
* Latency issues
* Service dependencies

It is extremely useful in:

* Microservices architectures
* Kubernetes environments
* Cloud-native applications

---

---

### Task 2: Add the OpenTelemetry Collector
Create the collector configuration:

```bash
mkdir -p otel-collector
```

Create `otel-collector/otel-collector-config.yml`:
```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:

exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"
  debug:
    verbosity: detailed

service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus]
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]
```

**What this config does:**
- **Receivers:** Accepts OTLP data via gRPC (4317) and HTTP (4318)
- **Processors:** Batches data before exporting (reduces overhead)
- **Exporters:**
  - Metrics go to a Prometheus-compatible endpoint on port 8889 (Prometheus scrapes this)
  - Traces and logs go to debug output (console) -- in production you would send these to Jaeger or Tempo

Add the collector to your `docker-compose.yml`:
```yaml
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    container_name: otel-collector
    ports:
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
      - "8889:8889"   # Prometheus exporter
    volumes:
      - ./otel-collector/otel-collector-config.yml:/etc/otelcol-contrib/config.yaml
    restart: unless-stopped
```

Add the OTEL Collector as a Prometheus scrape target in `prometheus.yml`:
```yaml
  - job_name: "otel-collector"
    static_configs:
      - targets: ["otel-collector:8889"]
```

Restart everything:
```bash
docker compose up -d
```

Verify the collector is running:
```bash
docker logs otel-collector 2>&1 | tail -5
```

Check Prometheus Targets -- you should now see `otel-collector` as UP.

---

Concept First:

The OTEL Collector is a standalone service that:

Receives telemetry via OTLP (your apps send to port 4317 or 4318)
Processes it (batching reduces overhead)
Exports metrics to Prometheus, traces to debug/Jaeger

vim otel-collector/otel-collector-config.yml:

<img width="983" height="745" alt="image" src="https://github.com/user-attachments/assets/d6d3ce52-6f22-4f47-9df1-239f5159aad4" />

vim docker-compose.yaml:

<img width="1060" height="655" alt="image" src="https://github.com/user-attachments/assets/d2e94ce1-d76e-4128-be2e-a63efbd29698" />

vim prometheus.yaml:

<img width="859" height="475" alt="image" src="https://github.com/user-attachments/assets/f7beed58-3b1b-4e0f-a7c5-ab367d44a426" />

docker ps and logs from otel-collector:

<img width="1913" height="730" alt="image" src="https://github.com/user-attachments/assets/fc7692f7-16e4-4728-9361-911febacf3b4" />

verified in the prometheus:

<img width="1919" height="788" alt="image" src="https://github.com/user-attachments/assets/e6b98df5-442d-40a6-9c33-ac0202a3d5dd" />

targets showing UP — prometheus, node-exporter, cadvisor, otel-collector

---

### Task 3: Send Test Traces to the Collector
Send a sample OTLP trace using curl:

```bash
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{
    "resourceSpans": [{
      "resource": {
        "attributes": [{
          "key": "service.name",
          "value": { "stringValue": "my-test-service" }
        }]
      },
      "scopeSpans": [{
        "spans": [{
          "traceId": "5b8efff798038103d269b633813fc60c",
          "spanId": "eee19b7ec3c1b174",
          "name": "test-span",
          "kind": 1,
          "startTimeUnixNano": "1544712660000000000",
          "endTimeUnixNano": "1544712661000000000",
          "attributes": [{
            "key": "http.method",
            "value": { "stringValue": "GET" }
          },
          {
            "key": "http.status_code",
            "value": { "intValue": "200" }
          }]
        }]
      }]
    }]
  }'
```

Check the collector debug output to see the trace:
```bash
docker logs otel-collector 2>&1 | grep -A 10 "test-span"
```

You should see the span details printed to the console. In a production setup, you would send these to a trace backend like Jaeger or Grafana Tempo for storage and visualization.

**Send OTLP metrics too:**
```bash
curl -X POST http://localhost:4318/v1/metrics \
  -H "Content-Type: application/json" \
  -d '{
    "resourceMetrics": [{
      "resource": {
        "attributes": [{
          "key": "service.name",
          "value": { "stringValue": "my-test-service" }
        }]
      },
      "scopeMetrics": [{
        "metrics": [{
          "name": "test_requests_total",
          "sum": {
            "dataPoints": [{
              "asInt": "42",
              "startTimeUnixNano": "1544712660000000000",
              "timeUnixNano": "1544712661000000000"
            }],
            "aggregationTemporality": 2,
            "isMonotonic": true
          }
        }]
      }]
    }]
  }'
```

Now query it in Prometheus:
```promql
test_requests_total
```

The metric traveled: your curl command -> OTEL Collector (OTLP receiver) -> Prometheus exporter -> Prometheus scraped it. This is how OTEL bridges different telemetry formats.

---

Concept First:

In production, apps automatically send traces using OTEL SDKs (Python, Go, Java). For learning, you simulate this using curl with OTLP JSON format.
The trace you'll send has:

service.name — which service sent it
traceId — unique ID for this entire request journey
spanId — unique ID for this specific step
name — name of the operation
startTime/endTime — duration measurement


send test trace via http:

<img width="1894" height="935" alt="image" src="https://github.com/user-attachments/assets/03cb1deb-ff4a-4c55-95e9-6711ba1f6e00" />

The trace traveled: your curl → OTEL Collector → printed to console

 Send a test METRIC via OTLP:

 <img width="1111" height="506" alt="image" src="https://github.com/user-attachments/assets/1cd3035f-dbb9-47b6-86ab-d8bb3edc9965" />

prometheus UI:

<img width="1919" height="303" alt="image" src="https://github.com/user-attachments/assets/ecd43473-5b31-46ec-b864-da57f474152a" />

 see the value 42 — the metric traveled

     curl → OTEL Collector (OTLP receiver) → Prometheus exporter → Prometheus scraped → visible!

 This proves the OTEL bridge works end-to-end!

 ---

### Task 4: Set Up Prometheus Alerting Rules
Alerts notify you when something is wrong. Prometheus evaluates alerting rules and fires alerts when conditions are met.

Create an alerting rules file `alert-rules.yml`:
```yaml
groups:
  - name: system-alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
          description: "CPU usage has been above 80% for more than 2 minutes. Current value: {{ $value }}%"

      - alert: HighMemoryUsage
        expr: (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 > 85
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage detected"
          description: "Memory usage is above 85%. Current value: {{ $value }}%"

      - alert: ContainerDown
        expr: absent(container_last_seen{name="notes-app"})
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container is down"
          description: "The notes-app container has not been seen for over 1 minute"

      - alert: TargetDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Scrape target is down"
          description: "{{ $labels.job }} target {{ $labels.instance }} is unreachable"

      - alert: HighDiskUsage
        expr: (1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk space running low"
          description: "Root filesystem usage is above 90%. Current value: {{ $value }}%"
```

**What each alert does:**
- `expr` -- the PromQL condition that triggers the alert
- `for` -- how long the condition must be true before firing (avoids flapping)
- `labels` -- metadata for routing (severity: warning vs critical)
- `annotations` -- human-readable description

Update `prometheus.yml` to load the rules:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - /etc/prometheus/alert-rules.yml

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

  - job_name: "otel-collector"
    static_configs:
      - targets: ["otel-collector:8889"]
```

Mount the rules file in `docker-compose.yml` under the Prometheus service:
```yaml
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert-rules.yml:/etc/prometheus/alert-rules.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    restart: unless-stopped
```

Restart Prometheus:
```bash
docker compose up -d prometheus
```

Check the rules in the Prometheus UI: go to Status > Rules. You should see all five alert rules listed.

Go to Alerts -- they should be in `inactive` state (green). If any condition is true, the alert moves to `pending`, then `firing` after the `for` duration.

**Test it:** Stop the notes-app container and watch the `TargetDown` alert fire:
```bash
docker compose stop notes-app
```

Wait 1-2 minutes, then check Alerts in the Prometheus UI. Start it back up when done:
```bash
docker compose start notes-app
```

---

Concept First:

Prometheus alerting rules define conditions that trigger alerts. An alert goes through 3 states:

| State | Meaning |
|---|---|
| Inactive | Condition is NOT true — everything is fine |
| Pending | Condition IS true but hasn't been true long enough (waiting for `for` duration) |
| Firing | Condition has been true longer than `for` duration — alert is active |

The for duration prevents flapping — brief CPU spikes shouldn't page you at 3 AM.

vim alert-rules.yaml:

<img width="1444" height="1094" alt="image" src="https://github.com/user-attachments/assets/62e4926f-ce28-4d66-bde2-a5ad1e43f569" />

mounted the rules file:

<img width="1160" height="960" alt="image" src="https://github.com/user-attachments/assets/fa6f2942-23d6-4302-ab72-8564a24c35ae" />

debugging: make sure to edit the prometheus.yaml and add the rules file there as well or else it will not work !!:

<img width="912" height="599" alt="image" src="https://github.com/user-attachments/assets/321c19fc-dadd-44f9-aa61-edb765df585e" />

prometheus/rules:

<img width="1908" height="610" alt="image" src="https://github.com/user-attachments/assets/e2a79578-888c-44f0-8c5c-710d5a699cfa" />

prometheus/alerts:

<img width="1915" height="654" alt="image" src="https://github.com/user-attachments/assets/06ed14db-5689-4a02-9755-7988f8251aaf" />

                # Stop a container
                docker compose stop cadvisor
                
                # Wait 1-2 minutes, then check
                # http://localhost:9090/alerts
                # cadvisor TargetDown should move to Pending then Firing

<img width="1909" height="944" alt="image" src="https://github.com/user-attachments/assets/33036d04-1f48-4561-8279-253799b743cd" />

<img width="1919" height="911" alt="image" src="https://github.com/user-attachments/assets/c829d696-8029-4a1d-9f4b-9310d08e29c3" />

Wait and watch the alert go: Inactive → Pending → Firing 

                docker compose start cadvisor

<img width="1919" height="907" alt="image" src="https://github.com/user-attachments/assets/bf293041-9ed5-4af2-8c9c-2345e6d8deb9" />

Verify: All 5 rules visible in Prometheus, TargetDown fires when a service stops.

---

### Task 5: Set Up Grafana Alerts
Grafana can also evaluate alerts and send notifications to Slack, email, PagerDuty, and more.

1. **Create a contact point:**
   - Go to Alerting > Contact points > Add contact point
   - Name: "DevOps Team"
   - Integration: Choose email (or Slack webhook if you have one)
   - For email: just enter your email address
   - Save

2. **Create an alert rule in Grafana:**
   - Go to Alerting > Alert rules > New alert rule
   - Name: "High Container Memory"
   - Query: `container_memory_usage_bytes{name="notes-app"} / 1024 / 1024`
   - Condition: IS ABOVE 100 (fire if container uses more than 100MB)
   - Evaluation: every 1m, for 2m
   - Add label: severity = warning
   - Link to the "DevOps Team" contact point
   - Save

3. **Create a notification policy:**
   - Go to Alerting > Notification policies
   - Set the default contact point to "DevOps Team"
   - Add a nested policy: match label `severity=critical` -> route to a different contact point (or the same one with different settings)

4. **View alert state:**
   - Go to Alerting > Alert rules
   - You should see your rule in Normal, Pending, or Firing state

**Document:** What is the difference between Prometheus alerts and Grafana alerts? When would you use each?

---

 Concept First:
 
| | Prometheus Alerts | Grafana Alerts |
|---|---|---|
| Evaluates | PromQL only | PromQL + LogQL + other datasources |
| Notifications | Needs Alertmanager | Built-in (email, Slack, PagerDuty) |
| Setup | Config file | Web UI |
| Best for | Infrastructure alerts | Business/app alerts |


Grafana alerts are easier to set up with notifications. For complex routing, use Prometheus + Alertmanager.

sent a sample notification but before that to work you must add the SMTP config in your docker file like this :

<img width="996" height="567" alt="image" src="https://github.com/user-attachments/assets/56ebd488-df51-417a-aace-4d3bd4a974a5" />

see that SMTP configs you must add then only it will work or else it wont.

<img width="1912" height="908" alt="image" src="https://github.com/user-attachments/assets/bde7fe33-25c0-40ab-97cc-fb60b46c41c1" />

checked:

<img width="799" height="768" alt="image" src="https://github.com/user-attachments/assets/16e5dd10-a06a-4281-b767-090885697906" />

<img width="1765" height="755" alt="image" src="https://github.com/user-attachments/assets/378101b9-80cf-4a6e-8b46-affa5393af85" />

Fill in:

Name: High Container Memory

Section 1 — Query:

Data source: Prometheus
Query A:

        promqlcontainer_memory_usage_bytes{name="notes-app"} / 1024 / 1024
        
Section 2 — Alert condition:

        Condition: IS ABOVE 100 (fire if memory > 100MB)

Section 3 — Evaluation:

        Evaluate every: 1m
        
        For: 2m (must be above for 2 minutes before firing)

Section 4 — Labels and notifications:

        Add label: severity = warning
        Contact point: DevOps Team

Click Save rule and exit.

<img width="1690" height="933" alt="image" src="https://github.com/user-attachments/assets/8f2f90a6-146a-464a-a124-d1bae309b0c2" />

#### View alert state:

Go to: Grafana → Alerting → Alert rules

You'll see your "High Container Memory" rule. State should be Normal (green).

If the container uses more than 100MB, it'll go to Pending, then Firing and send you an email.

✅ Verify: Alert rule visible in Grafana with Normal state.

<img width="1649" height="642" alt="image" src="https://github.com/user-attachments/assets/83dd22e6-676f-4eaf-acbe-8b6fb0f7267c" />

---

### Task 6: Review the Full Stack Architecture
Your observability stack now covers all three pillars. Map out what you have built:

```
                    METRICS PIPELINE
[Node Exporter] -----> [Prometheus] -----> [Grafana Dashboards]
[cAdvisor] ----------> [Prometheus] -----> [Grafana Dashboards]
[OTEL Collector:8889]> [Prometheus] -----> [Grafana Dashboards]
                                    -----> [Alert Rules -> Notifications]

                    LOGS PIPELINE
[Docker Containers] -> [Promtail] -> [Loki] -> [Grafana Explore/Dashboards]

                    TRACES PIPELINE
[curl/App OTLP] -----> [OTEL Collector] -> [Debug Output / Future: Jaeger/Tempo]
```

**Services running:**

| Service | Port | Purpose |
|---------|------|---------|
| Prometheus | 9090 | Metrics storage and querying |
| Node Exporter | 9100 | Host system metrics |
| cAdvisor | 8080 | Container metrics |
| Grafana | 3000 | Visualization and alerting |
| Loki | 3100 | Log storage |
| Promtail | 9080 | Log collection agent |
| OTEL Collector | 4317/4318/8889 | Telemetry collection |
| Notes App | 8000 | Sample application |

Verify all services are running:
```bash
docker compose ps
```

All 8 containers should be healthy and running.

---

<img width="1897" height="396" alt="image" src="https://github.com/user-attachments/assets/b689b832-fd5e-436a-b1f4-36e5d3bc6307" />

executed this command to fire alert:

         docker exec -it notes-app python3 -c "a=['A'*1024*1024 for _ in range(150)]; input('Holding memory...')"
         
<img width="1624" height="849" alt="image" src="https://github.com/user-attachments/assets/13571188-cef9-4f92-97f9-3e0e66519624" />

checked notification:

<img width="731" height="792" alt="image" src="https://github.com/user-attachments/assets/6e000d41-68af-46b2-9530-a1d75b2e8a64" />

<img width="711" height="824" alt="image" src="https://github.com/user-attachments/assets/34d17ee2-3935-4ff8-825c-d1dd5dadf0a0" />

reverted my command to recover the alert firing:

<img width="1499" height="913" alt="image" src="https://github.com/user-attachments/assets/5ac0a94c-1df5-46e7-a983-2ef184461f76" />

---

# Day 76 – OpenTelemetry and Alerting

## The Three Pillars — Complete

| Pillar | Answers | Tools |
|---|---|---|
| Metrics | WHAT broke and WHEN | Prometheus, Node Exporter, cAdvisor |
| Logs | WHY it broke | Loki, Promtail |
| Traces | WHERE it's slow | OTEL Collector |

## OpenTelemetry Architecture
                
                ↓ port 4317 (gRPC) or 4318 (HTTP)
                OTEL Collector
                ├── Receivers: accept OTLP data
                ├── Processors: batch (reduce overhead)
                └── Exporters:
                ├── Prometheus endpoint :8889 ← metrics
                └── Debug console       ← traces and logs
                Prometheus scrapes :8889
                Grafana queries Prometheus


## Full Stack Architecture

METRICS:  Node Exporter → Prometheus → Grafana
cAdvisor      → Prometheus → Grafana
OTEL :8889    → Prometheus → Grafana
→ Alert Rules
LOGS:     Docker → Promtail → Loki → Grafana
TRACES:   App/curl → OTEL Collector → Debug/Jaeger

## Services Running

| Service | Port | Purpose |
|---|---|---|
| Prometheus | 9090 | Metrics storage and querying |
| Node Exporter | 9100 | Host system metrics |
| cAdvisor | 8080 | Container metrics |
| Grafana | 3000 | Visualization and alerting |
| Loki | 3100 | Log storage |
| Promtail | 9080 | Log collection agent |
| OTEL Collector | 4317/4318/8889 | Telemetry collection |

## Prometheus Alert States

| State | Meaning |
|---|---|
| Inactive | Condition not true — all good |
| Pending | Condition true but waiting for `for` duration |
| Firing | Condition true longer than `for` — alert active |

## Alert Rules Created

| Alert | Condition | Severity |
|---|---|---|
| HighCPUUsage | CPU > 80% for 2m | warning |
| HighMemoryUsage | RAM > 85% for 2m | warning |
| ContainerDown | notes-app absent for 1m | critical |
| TargetDown | any scrape target down for 1m | critical |
| HighDiskUsage | disk > 90% for 5m | critical |

## Prometheus Alerts vs Grafana Alerts

| | Prometheus | Grafana |
|---|---|---|
| Notifications | Needs Alertmanager | Built-in |
| Data sources | PromQL only | Multi-source |
| Best for | Infrastructure | App/business metrics |

## What is OTLP?
OpenTelemetry Protocol — standard wire format for telemetry.
gRPC on port 4317, HTTP on port 4318.
Apps send one format, OTEL converts to any backend format.

## What are Traces?
A trace = one request's journey across multiple services.
Each step = a span (has traceId, spanId, name, duration, attributes).
Example: API Gateway → Auth Service → Database — 3 spans, 1 trace.

###  Summary Table

| Concept | What it means |
|---|---|
| OpenTelemetry (OTEL) | Vendor-neutral framework for collecting metrics, logs, traces |
| OTEL Collector | Receives, processes, exports telemetry — not a backend |
| OTLP | OpenTelemetry Protocol — standard format, gRPC:4317 HTTP:4318 |
| Trace | Complete journey of one request across multiple services |
| Span | One step in a trace — has ID, name, duration, attributes |
| Receivers | OTEL pipeline stage: accept incoming data |
| Processors | OTEL pipeline stage: transform data (batch, filter) |
| Exporters | OTEL pipeline stage: send to backends |
| `debug` exporter | Prints telemetry to console — use for learning |
| `prometheus` exporter | Exposes metrics for Prometheus to scrape |
| Alert: Inactive | Condition not true — everything fine |
| Alert: Pending | Condition true, waiting for `for` duration |
| Alert: Firing | Active alert — condition true longer than `for` |
| `for: 2m` | Prevents flapping — brief spikes don't page you |
| `absent()` | PromQL function — fires when a metric disappears |
| Contact point | Where Grafana sends notifications (email, Slack) |
| Notification policy | Rules for routing alerts to contact points |
