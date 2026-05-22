# Day 73 -- Introduction to Observability and Prometheus

## Task
You have built infrastructure with Terraform, configured servers with Ansible, and containerized applications with Docker. But once everything is running -- how do you know it is healthy? How do you find out why something broke at 3 AM?

That is where observability comes in. Today you learn the three pillars of observability -- metrics, logs, and traces -- and set up Prometheus, the most widely used metrics collection tool in the DevOps ecosystem.

---

 What Are We Building?:

     Your Terminal
         ↓
    docker compose up -d
         ↓
    ┌─────────────────────────────────────────┐
    │  Local Machine / EC2                    │
    │                                         │
    │  Prometheus (port 9090)                 │
    │  ← scrapes metrics every 15s           │
    │                                         │
    │  notes-app (port 8000)                 │
    │  ← target being monitored              │
    │                                         │
    │  You query metrics with PromQL          │
    │  in the Prometheus Web UI               │
    └─────────────────────────────────────────┘

The three pillars of observability ::

| Pillar | What it answers | Tools |
|---|---|---|
| Metrics | What is broken — numbers over time | Prometheus, Datadog |
| Logs | Why it broke — timestamped events | Loki, ELK Stack |
| Traces | Where it broke — request journey | Jaeger, OpenTelemetry |

---

## Challenge Tasks

### Task 1: Understand Observability
Research and write short notes on:

1. What is observability? How is it different from traditional monitoring?
   - **Monitoring** tells you _when_ something is wrong (alerts, thresholds)
   - **Observability** tells you _why_ something is wrong (explore, query, correlate)

2. The three pillars of observability:
   - **Metrics** -- numerical measurements over time (CPU usage, request count, error rate). Tools: Prometheus, Datadog, CloudWatch
   - **Logs** -- timestamped text records of events (application output, error messages). Tools: Loki, ELK Stack, Fluentd
   - **Traces** -- the journey of a single request across multiple services. Tools: OpenTelemetry, Jaeger, Zipkin

3. Why do DevOps engineers need all three?
   - Metrics tell you _what_ is broken (high error rate on `/api/users`)
   - Logs tell you _why_ it broke (stack trace showing a database timeout)
   - Traces tell you _where_ it broke (the payment service call took 12 seconds)

4. Draw or describe this architecture -- this is what you will build over the next 5 days:
   ```
   [Your App] --> metrics --> [Prometheus] --> [Grafana Dashboards]
   [Your App] --> logs    --> [Promtail]   --> [Loki] --> [Grafana]
   [Your App] --> traces  --> [OTEL Collector] --> [Grafana/Debug]
   [Host]     --> metrics --> [Node Exporter] --> [Prometheus]
   [Docker]   --> metrics --> [cAdvisor] --> [Prometheus]
   ```

---

Concept:

Traditional monitoring = you define thresholds and get paged when something crosses them. You knew what to measure in advance.
Observability = your system exposes enough data that you can ask any question about its internal state, even ones you didn't think of beforehand.

The architecture you'll build over Days 73–77:

    [Your App]  --> metrics --> [Prometheus]      --> [Grafana Dashboards]
    [Your App]  --> logs    --> [Promtail]        --> [Loki] --> [Grafana]
    [Your App]  --> traces  --> [OTEL Collector]  --> [Grafana/Debug]
    [Host]      --> metrics --> [Node Exporter]   --> [Prometheus]
    [Docker]    --> metrics --> [cAdvisor]        --> [Prometheus]

Today's focus: the Prometheus block in that diagram.

The three pillars of observability ::

| Pillar | What it answers | Tools |
|---|---|---|
| Metrics | What is broken — numbers over time | Prometheus, Datadog |
| Logs | Why it broke — timestamped events | Loki, ELK Stack |
| Traces | Where it broke — request journey | Jaeger, OpenTelemetry |


Four Prometheus metric types — know these cold:

| Type | Behavior | Real Example |
|---|---|---|
| Counter | Only goes up, never down | `http_requests_total` |
| Gauge | Goes up and down freely | `memory_usage_bytes` |
| Histogram | Counts values in buckets | `request_duration_seconds` |
| Summary | Calculates percentiles client-side | `rpc_duration_seconds` |

---

### Task 2: Set Up Prometheus with Docker
Create a project directory for this entire observability block -- you will keep adding to it over the next 5 days.

```bash
mkdir observability-stack && cd observability-stack
```

Create a `prometheus.yml` configuration file:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

This tells Prometheus to scrape its own metrics every 15 seconds.

Create a `docker-compose.yml` to run Prometheus:
```yaml
services:
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
    restart: unless-stopped

volumes:
  prometheus_data:
```

Start Prometheus:
```bash
docker compose up -d
```

Open `http://localhost:9090` in your browser. You should see the Prometheus web UI.

**Verify:** Go to Status > Targets. You should see one target (`prometheus`) with state `UP`.

---

Created dir observability-stack && cd observability-stack , 

vim prometheus.yaml:

<img width="571" height="173" alt="image" src="https://github.com/user-attachments/assets/8c085335-ecdf-41b8-a5e6-018580f67bf8" />

docker-compose.yaml: (for prometheus)

<img width="443" height="256" alt="image" src="https://github.com/user-attachments/assets/734747eb-0d15-4797-a72d-9ec67cc4393f" />

docker compose up:

<img width="1902" height="448" alt="image" src="https://github.com/user-attachments/assets/26cb1990-fa12-4c1a-b9be-bf6bb66763ba" />

Prometheus working:

<img width="1229" height="499" alt="image" src="https://github.com/user-attachments/assets/a721229e-705a-4c03-82e6-80e4e31711de" />

<img width="1894" height="406" alt="image" src="https://github.com/user-attachments/assets/c09d390c-6fda-43a3-9f7c-cae78abfe01c" />

Confirm the scrape target is UP:

Go to Status → Targets in the UI.
You should see one target (prometheus) with state UP.

---

### Task 3: Understand Prometheus Concepts
Explore the Prometheus UI and understand these concepts:

1. **Scrape targets** -- endpoints that Prometheus pulls metrics from at regular intervals (pull-based model)
2. **Metrics types:**
   - `Counter` -- only goes up (total requests served, total errors)
   - `Gauge` -- goes up and down (current CPU usage, memory in use, active connections)
   - `Histogram` -- distribution of values in buckets (request duration: how many took <100ms, <500ms, <1s)
   - `Summary` -- similar to histogram but calculates percentiles on the client side
3. **Labels** -- key-value pairs that add dimensions to metrics (e.g., `http_requests_total{method="GET", status="200"}`)
4. **Time series** -- a unique combination of metric name + labels

Go to the Prometheus UI graph page (`http://localhost:9090/graph`) and run these queries:

```
# How many metrics is Prometheus collecting about itself?
count({__name__=~".+"})

# How much memory is Prometheus using?
process_resident_memory_bytes

# Total HTTP requests to the Prometheus server
prometheus_http_requests_total

# Break it down by handler
prometheus_http_requests_total{handler="/api/v1/query"}
```

**Document:** What is the difference between a counter and a gauge? Give one real-world example of each.

---


<img width="1902" height="877" alt="image" src="https://github.com/user-attachments/assets/1af1662f-66bc-4014-9e6b-1804b90de447" />

<img width="1884" height="916" alt="image" src="https://github.com/user-attachments/assets/07ec40f3-8540-4ae6-861f-9a00e5329fe8" />

<img width="1898" height="964" alt="image" src="https://github.com/user-attachments/assets/0ee0996f-b0b0-42b7-a803-e788b5bf5c45" />

<img width="1900" height="908" alt="image" src="https://github.com/user-attachments/assets/34d60413-e2e1-44fc-842b-6d6541b2a705" />

Each result is a time series — a unique combination of metric name + labels. The labels (inside {}) are what make Prometheus powerful. You can slice and filter any metric by any label dimension.

---

### Task 4: Learn PromQL Basics
PromQL (Prometheus Query Language) is how you ask questions about your metrics. Run these queries in the Prometheus UI:

1. **Instant vector** -- current value of a metric:
```promql
up
```
This returns 1 (up) or 0 (down) for each scrape target.

2. **Range vector** -- values over a time window:
```promql
prometheus_http_requests_total[5m]
```
Returns all values from the last 5 minutes.

3. **Rate** -- per-second rate of a counter over a time window:
```promql
rate(prometheus_http_requests_total[5m])
```
This is the most common function you will use. Counters always go up -- `rate()` converts them to a useful per-second speed.

4. **Aggregation** -- sum across all label combinations:
```promql
sum(rate(prometheus_http_requests_total[5m]))
```

5. **Filter by label:**
```promql
prometheus_http_requests_total{code="200"}
prometheus_http_requests_total{code!="200"}
```

6. **Arithmetic:**
```promql
process_resident_memory_bytes / 1024 / 1024
```
This converts bytes to megabytes.

7. **Top-K:**
```promql
topk(5, prometheus_http_requests_total)
```

**Try this exercise:** Write a PromQL query that shows the per-second rate of non-200 HTTP requests to Prometheus over the last 5 minutes. (Hint: use `rate()` with a label filter on `code!="200"`)

---

Concept First:
PromQL is the query language for Prometheus. Three things to understand before you start:

- Instant vector — current value right now.

- Range vector — values over a time window (e.g. last 5 minutes).

- rate() — converts a counter (always increasing) into a useful per-second speed

1) Instant vector — what's the state of every target right now?

       up
   
Returns 1 (healthy) or 0 (unreachable) for each target.

2) Range vector — give me all values from the last 5 minutes:
   
       prometheus_http_requests_total[5m]
   
3) Rate — convert a counter to per-second speed:

       rate(prometheus_http_requests_total[5m])
   
This is the most important function in PromQL. Raw counters only go up — rate() is what makes them meaningful.

4) Sum across all label combinations:
   
       sum(rate(prometheus_http_requests_total[5m]))
   
⚠️ Always rate() first, then sum(). Never the other way around.
5) Filter by label value:

      prometheus_http_requests_total{code="200"}
      
prometheus_http_requests_total{code!="200"}

6) Arithmetic — convert bytes to megabytes
7) 
promqlprocess_resident_memory_bytes / 1024 / 1024

7) Top-K — show the 5 highest values

       topk(5, prometheus_http_requests_total)

8) Exercise — write this query yourself

Show the per-second rate of non-200 HTTP requests to Prometheus over the last 5 minutes.

Hint: combine rate() + {code!="200"} label filter.

     rate(prometheus_http_requests_total{code!="200"}[5m])

---

### Task 5: Add a Sample Application as a Scrape Target
Prometheus needs something to monitor. Add a simple metrics-generating service.

Update your `docker-compose.yml` to include a sample app that exposes Prometheus metrics:
```yaml
services:
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
    restart: unless-stopped

  notes-app:
    image: trainwithshubham/notes-app:latest
    container_name: notes-app
    ports:
      - "8000:8000"
    restart: unless-stopped

volumes:
  prometheus_data:
```

Update `prometheus.yml` to scrape the app:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "notes-app"
    static_configs:
      - targets: ["notes-app:8000"]
```

Restart the stack:
```bash
docker compose up -d
```

Go back to Status > Targets. You should now see two targets. Generate some traffic to the app:
```bash
curl http://localhost:8000
curl http://localhost:8000
curl http://localhost:8000
```

**Note:** Not all applications expose Prometheus metrics natively. In later days you will learn how Node Exporter, cAdvisor, and OTEL Collector act as metric exporters for systems that do not have built-in Prometheus support.

---

Concept First
Prometheus is useless without targets to scrape. Right now it only monitors itself. Let's add a real app.

docker-compose.yml:

<img width="672" height="431" alt="image" src="https://github.com/user-attachments/assets/d32582f4-df9a-4f9d-9d20-f8033b8bf293" />

tried scraping metrics but the image : trainwithshubham does not expose it :

<img width="1919" height="896" alt="image" src="https://github.com/user-attachments/assets/8f69c438-64de-4989-8431-d45feacf0e7f" />

so used different image node-exporter:

<img width="1914" height="824" alt="image" src="https://github.com/user-attachments/assets/66d31995-9db4-4d89-8ece-8dfba321d171" />

dashboard:

<img width="1901" height="932" alt="image" src="https://github.com/user-attachments/assets/d0c9c4d7-df47-4a95-a494-7a61c902b16e" />

<img width="1885" height="833" alt="image" src="https://github.com/user-attachments/assets/8423c771-59eb-4a07-ad01-0ac2b8ac2fb3" />

---

### Task 6: Explore Data Retention and Storage
Understand how Prometheus stores data:

1. Check how much disk space Prometheus is using:
```bash
docker exec prometheus du -sh /prometheus
```

2. Prometheus stores data in a local time-series database (TSDB). Default retention is 15 days. You can change it:
```yaml
command:
  - '--config.file=/etc/prometheus/prometheus.yml'
  - '--storage.tsdb.retention.time=30d'
  - '--storage.tsdb.retention.size=1GB'
```

3. Check the TSDB status in the UI: Status > TSDB Status

**Document:** What happens when retention is exceeded? Why is a volume mount important for Prometheus data?

---

<img width="949" height="125" alt="image" src="https://github.com/user-attachments/assets/139ec8b5-3e4c-4955-ba47-f0545eaa30f8" />

Understand the retention settings:

Default retention is 15 days. For production, you'd configure it like this in docker-compose.yml:
   
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'
      - '--storage.tsdb.retention.size=1GB'
      
When retention is exceeded, Prometheus deletes the oldest data blocks first. This is why the named volume (prometheus_data) matters — without it, every docker compose down would wipe all your historical metrics.


TSDB ( Time series databse ):

<img width="1722" height="974" alt="image" src="https://github.com/user-attachments/assets/88d7371d-cb51-42a2-9235-261c44422677" />

---

## The Three Pillars of Observability

| Pillar | What it tells you | Tool used |
|---|---|---|
| Metrics | What is broken — numbers over time | Prometheus |
| Logs | Why it broke — timestamped event records | Loki / ELK |
| Traces | Where it broke — single request journey | Jaeger / OTEL |

Monitoring = you defined thresholds in advance and get paged when crossed.
Observability = your system exposes enough data to answer questions you didn't anticipate.

## Architecture (Days 73–77)

    [Your App]  --> metrics --> [Prometheus]      --> [Grafana Dashboards]
    [Your App]  --> logs    --> [Promtail]        --> [Loki] --> [Grafana]
    [Your App]  --> traces  --> [OTEL Collector]  --> [Grafana/Debug]
    [Host]      --> metrics --> [Node Exporter]   --> [Prometheus]
    [Docker]    --> metrics --> [cAdvisor]        --> [Prometheus]

## Files Created Today

### prometheus.yml
```yaml
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
```

### docker-compose.yml
```yaml
services:
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
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter
    ports:
    - "9100:9100"
    restart: unless-stopped
volumes:
  prometheus_data:
```

## PromQL Queries I Ran

| Query | What it returns |
|---|---|
| `up` | 1 (healthy) or 0 (down) for every scrape target |
| `process_resident_memory_bytes / 1024 / 1024` | Prometheus RAM usage in MB |
| `rate(prometheus_http_requests_total[5m])` | Per-second request rate over 5 min |
| `prometheus_http_requests_total{code="200"}` | Only successful requests |
| `rate(prometheus_http_requests_total{code!="200"}[5m])` | Per-second error rate |

## Counter vs Gauge

**Counter** — only ever goes up. Reset to zero on restart.
Example: `http_requests_total` — total requests served since the process started.
You use `rate()` to make it useful: how many requests per second right now?

**Gauge** — goes up and down freely.
Example: `memory_usage_bytes` — current RAM in use. Can increase or decrease any time.
You read it directly — no `rate()` needed.

## Key Concepts

| Concept | What it means |
|---|---|
| Pull model | Prometheus scrapes targets — targets don't push to Prometheus |
| `up` metric | Auto-created for every target — 1 = healthy, 0 = unreachable |
| Labels | Key-value pairs that add dimensions: `{method="GET", code="200"}` |
| Time series | Unique combination of metric name + label set |
| `rate()` | Converts a counter into per-second speed — use before `sum()` |
| Named volume | Keeps TSDB data alive across container restarts |
| Retention | Default 15 days — oldest blocks deleted first when exceeded |

## What's Coming Next

| Day | Topic |
|---|---|
| 74 | Node Exporter + cAdvisor — system and container metrics |
| 75 | Grafana — visualize Prometheus data in dashboards |
| 76 | Loki + Promtail — centralized log aggregation |
| 77 | Alertmanager — alerts when things go wrong |

### Summary Table

| Concept | What it means in this project |
|---|---|
| `prometheus.yml` | Config file — defines scrape targets and intervals |
| `scrape_interval: 15s` | Prometheus pulls metrics from every target every 15 seconds |
| `job_name` | Logical name for a group of scrape targets |
| `targets` | The `host:port` endpoints Prometheus scrapes |
| `docker-compose.yml` | Runs Prometheus + notes-app as containers |
| `prometheus_data` volume | Persists TSDB data across restarts |
| `up` metric | Auto-generated — `1` means target is reachable, `0` means down |
| `rate()` | Converts ever-increasing counter to per-second rate |
| `sum(rate(...))` | Aggregate rate across all label combinations |
| `{code!="200"}` | Label filter — only return non-success responses |
| `topk(5, ...)` | Return top 5 time series by value |
| `[5m]` range selector | Look back 5 minutes of data for range functions |
| `Status → Targets` | UI page showing health of every scrape target |
| `Status → TSDB Status` | UI page showing storage usage and series count |
| `--storage.tsdb.retention.time` | How long Prometheus keeps data before deleting |
