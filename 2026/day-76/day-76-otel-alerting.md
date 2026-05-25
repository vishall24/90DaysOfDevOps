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

