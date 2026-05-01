# Observability: Traces, Metrics, Logs & Tools

### OpenTelemetry · Graylog · Kibana · Grafana · CloudWatch · and Everything You Must Know

#### For Everyone: 5-Year-Old 🧸 → Mid-Level 💻 → Expert 🧠 → Critics 🔥

---

## Table of Contents

1. [The Big Picture — What is Observability?](#1-the-big-picture)
2. [For a 5-Year-Old 🧸](#2-for-a-5-year-old-)
3. [For a Mid-Level Developer 💻](#3-for-a-mid-level-developer-)
   - 3.1 Logs — What, Why, How
   - 3.2 Metrics — What, Why, How
   - 3.3 Traces — What, Why, How
   - 3.4 The Three Pillars Together
   - 3.5 OpenTelemetry
   - 3.6 Graylog
   - 3.7 Kibana (ELK Stack)
   - 3.8 Grafana
   - 3.9 CloudWatch
   - 3.10 Tool Comparison — Which Does What
4. [For an Expert Developer 🧠](#4-for-an-expert-developer-)
   - 4.1 Structured Logging in Depth
   - 4.2 Distributed Tracing Deep Dive
   - 4.3 RED Method & USE Method
   - 4.4 OpenTelemetry Architecture
   - 4.5 Alerting & SLOs / SLAs / SLIs
   - 4.6 Log Aggregation Pipelines
   - 4.7 Cardinality Problem in Metrics
   - 4.8 Sampling Strategies
   - 4.9 Correlation — Tying It All Together
   - 4.10 PHP + Python Implementation
5. [For the Critics 🔥](#5-for-the-critics-)
6. [Full Tool Comparison Table](#6-full-tool-comparison-table)
7. [Decision Guide — What to Use When](#7-decision-guide)

---

## 1. The Big Picture

### What is Observability?

**Observability** means: _Can you understand what is happening INSIDE your system by looking at its OUTPUT?_

It is made of **three pillars**:

```
┌─────────────────────────────────────────────────────────┐
│                    OBSERVABILITY                        │
│                                                         │
│   📋 LOGS        📊 METRICS        🔍 TRACES           │
│   "What          "How much /        "What path did      │
│   happened?"     how fast?"         this request take?" │
│                                                         │
│   Events         Numbers            Journey             │
│   Text           Aggregates         Spans               │
│   Searchable     Graphable          Visualizable        │
└─────────────────────────────────────────────────────────┘
```

Without observability, your production system is a **black box**.
With it, it becomes a **glass box** — you see everything.

---

## 2. For a 5-Year-Old 🧸

### The Pizza Delivery Story 🍕

Imagine you run a pizza shop. You have 100 delivery people running around the city.
Something goes wrong — a customer calls and says "WHERE IS MY PIZZA?"

How do you find out what happened?

---

### 📋 Logs = A Diary

Every delivery person writes in a **diary** every time something happens:

```
10:01 AM — Pizza picked up from kitchen ✅
10:15 AM — Arrived at customer address ✅
10:16 AM — Customer not home ❌
10:17 AM — Left pizza at door 📦
```

**Logs tell you WHAT HAPPENED, step by step.**
When something goes wrong — you read the diary and find the exact problem.

---

### 📊 Metrics = A Speedometer Dashboard

Your shop has a big dashboard on the wall showing:

```
🍕 Pizzas delivered today:   347
⏱️  Average delivery time:   28 minutes
❌  Failed deliveries:        3
🛵  Active drivers:           42
```

You don't read each diary for this. The dashboard **updates every minute automatically**.

**Metrics tell you HOW MUCH and HOW FAST — at a glance.**
If the average delivery time jumps from 28 to 95 minutes — something is wrong. You see it immediately.

---

### 🔍 Traces = A GPS Tracker

Each pizza has a **GPS tracker** inside the box.
You can see the **entire journey** of that exact pizza:

```
Order #4521 journey:
  Kitchen      → [2 min] → packed
  Packed       → [5 min] → picked up by driver
  Driver       → [22 min] → arrived at address  ← SLOW! traffic jam here
  Address      → [1 min] → delivered
  Total: 30 min
```

**Traces tell you the exact JOURNEY of ONE specific request.**
Where did it slow down? Which step was broken?

---

### The Tools = The Office Staff 🏢

| Tool              | What They Do in the Pizza Shop                                                       |
| ----------------- | ------------------------------------------------------------------------------------ |
| **Graylog**       | The person who collects ALL diaries and lets you search them                         |
| **Kibana**        | Same — but with prettier search and charts                                           |
| **Grafana**       | The big screen showing all dashboards — speedometers                                 |
| **CloudWatch**    | Amazon's own all-in-one pizza shop monitoring (if your shop runs on AWS)             |
| **OpenTelemetry** | The standard notepad format ALL delivery people use — so any office tool can read it |

---

## 3. For a Mid-Level Developer 💻

### 3.1 Logs — What, Why, How

A **log** is a timestamped record of something that happened in your application.

#### Types of Log Levels

```
TRACE   → Most detailed — every tiny step (rarely used in prod)
DEBUG   → Developer-level detail — variable values, flow
INFO    → Normal operations — "User logged in", "Order created"
WARN    → Something unexpected but not breaking — "Retry attempt 2/3"
ERROR   → Something broke — "Payment failed", "DB connection lost"
FATAL   → System cannot continue — "Out of memory", "Disk full"
```

#### Bad Log (Unstructured) ❌

```
[2026-04-25 10:15:33] User 42 bought item 99 for 29.99
```

This is a plain string. You **cannot search it reliably**. You cannot filter by user ID, item ID, or price without regex hacks.

#### Good Log (Structured JSON) ✅

```json
{
  "timestamp": "2026-04-25T10:15:33Z",
  "level": "INFO",
  "message": "Order created",
  "user_id": 42,
  "order_id": "ORD-8821",
  "item_id": 99,
  "amount": 29.99,
  "currency": "EUR",
  "service": "order-service",
  "trace_id": "abc123def456"
}
```

Now you can search: `user_id:42 AND level:ERROR` — instantly find all errors for this user.

#### PHP — Structured Logging with Monolog

```php
<?php

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Formatter\JsonFormatter;

$log = new Logger('order-service');
$handler = new StreamHandler('php://stdout', Logger::DEBUG);
$handler->setFormatter(new JsonFormatter());
$log->pushHandler($handler);

// Context array = structured fields
$log->info('Order created', [
    'user_id'  => 42,
    'order_id' => 'ORD-8821',
    'amount'   => 29.99,
    'currency' => 'EUR',
]);

$log->error('Payment failed', [
    'user_id'      => 42,
    'order_id'     => 'ORD-8821',
    'error_code'   => 'CARD_DECLINED',
    'gateway'      => 'stripe',
]);
```

#### Python — Structured Logging

```python
import logging
import json
from datetime import datetime, timezone


class JsonFormatter(logging.Formatter):
    def format(self, record: logging.LogRecord) -> str:
        log_data = {
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "level":     record.levelname,
            "message":   record.getMessage(),
            "service":   "order-service",
        }
        # Merge extra fields
        if hasattr(record, "extra"):
            log_data.update(record.extra)
        return json.dumps(log_data)


logger = logging.getLogger("order-service")
handler = logging.StreamHandler()
handler.setFormatter(JsonFormatter())
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)


def log_with_context(level: str, message: str, **kwargs) -> None:
    record = logger.makeRecord(
        logger.name, getattr(logging, level),
        "", 0, message, (), None
    )
    record.extra = kwargs
    logger.handle(record)


log_with_context("INFO", "Order created",
    user_id=42, order_id="ORD-8821", amount=29.99)

log_with_context("ERROR", "Payment failed",
    user_id=42, order_id="ORD-8821", error_code="CARD_DECLINED")
```

---

### 3.2 Metrics — What, Why, How

**Metrics** are **numerical measurements** collected over time. They are aggregated — not per-event like logs.

#### Four Types of Metrics

```
COUNTER    → Only goes UP — total requests, total errors, total sales
             Example: requests_total = 10,482

GAUGE      → Goes UP and DOWN — current value right now
             Example: active_connections = 47, memory_used_mb = 812

HISTOGRAM  → Distribution of values — response time buckets
             Example: 80% requests < 100ms, 95% < 200ms, 99% < 500ms

SUMMARY    → Similar to histogram — pre-calculated quantiles
             Example: p50=95ms, p95=280ms, p99=450ms
```

#### What is Prometheus?

Prometheus is an open-source monitoring and alerting system

It is mainly used to:

- collect metrics (numbers over time)
- store them
- query them
- create alerts

#### PHP — Metrics with Prometheus

```php
<?php

use Prometheus\CollectorRegistry;
use Prometheus\Storage\InMemory;
use Prometheus\RenderTextFormat;

$registry = new CollectorRegistry(new InMemory());

// Counter — total HTTP requests
$requestCounter = $registry->registerCounter(
    'app',
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'route', 'status']
);

// Histogram — request duration
$durationHistogram = $registry->registerHistogram(
    'app',
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'route'],
    [0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5]
);

// Gauge — active DB connections
$dbConnectionsGauge = $registry->registerGauge(
    'app',
    'db_connections_active',
    'Active database connections'
);

// Record a request
$start = microtime(true);
// ... handle request ...
$duration = microtime(true) - $start;

$requestCounter->inc(['GET', '/api/orders', '200']);
$durationHistogram->observe($duration, ['GET', '/api/orders']);
$dbConnectionsGauge->set(42);
```

#### Python — Metrics with Prometheus

```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time
import random

# Counter
http_requests_total = Counter(
    "http_requests_total",
    "Total HTTP requests",
    ["method", "route", "status"]
)

# Histogram
http_duration_seconds = Histogram(
    "http_request_duration_seconds",
    "HTTP request duration in seconds",
    ["method", "route"],
    buckets=[0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5]
)

# Gauge
db_connections_active = Gauge(
    "db_connections_active",
    "Active database connections"
)

# Usage
def handle_request(method: str, route: str) -> None:
    start = time.time()
    try:
        # Simulate work
        time.sleep(random.uniform(0.01, 0.3))
        http_requests_total.labels(method, route, "200").inc()
    except Exception:
        http_requests_total.labels(method, route, "500").inc()
        raise
    finally:
        duration = time.time() - start
        http_duration_seconds.labels(method, route).observe(duration)

# Context manager approach
with http_duration_seconds.labels("GET", "/api/users").time():
    time.sleep(0.1)  # auto-records duration

# Expose metrics endpoint for Prometheus to scrape
start_http_server(8000)  # available at http://localhost:8000/metrics
```

---

### 3.3 Traces — What, Why, How

A **trace** is the **complete journey** of one request through your system — across all services it touches.

#### Key Concepts

```
TRACE    → The entire end-to-end journey of one request
           Has a unique trace_id

SPAN     → One step in that journey (one service, one operation)
           Has a span_id + parent_span_id

CONTEXT  → The trace_id + span_id passed between services
           So each service knows which trace it belongs to
```

#### Visual Example — One Request, Multiple Services

```
Trace ID: abc123
│
├─ Span 1: API Gateway          [0ms → 245ms] ← root span
│   │
│   ├─ Span 2: Auth Service     [5ms → 22ms]
│   │
│   ├─ Span 3: Order Service    [23ms → 198ms]
│   │   │
│   │   ├─ Span 4: MySQL Query  [25ms → 67ms]  ← slow!
│   │   │
│   │   └─ Span 5: Redis Cache  [68ms → 71ms]
│   │
│   └─ Span 6: Email Service    [199ms → 244ms]
│
Total: 245ms — bottleneck found at Span 4 (MySQL)
```

Without tracing — you'd see "request is slow" in metrics but not WHERE.
With tracing — you see exactly which span is the problem.

#### PHP — Tracing with OpenTelemetry

```php
<?php

use OpenTelemetry\API\Globals;
use OpenTelemetry\API\Trace\SpanKind;
use OpenTelemetry\API\Trace\StatusCode;

$tracer = Globals::tracerProvider()->getTracer('order-service');

// Create a root span
$span = $tracer->spanBuilder('create-order')
    ->setSpanKind(SpanKind::KIND_SERVER)
    ->startSpan();

$scope = $span->activate();

try {
    // Add attributes to the span
    $span->setAttribute('user.id', 42);
    $span->setAttribute('order.id', 'ORD-8821');
    $span->setAttribute('order.amount', 29.99);

    // Create a child span for DB operation
    $dbSpan = $tracer->spanBuilder('mysql.query')
        ->setSpanKind(SpanKind::KIND_CLIENT)
        ->startSpan();

    try {
        $dbSpan->setAttribute('db.system', 'mysql');
        $dbSpan->setAttribute('db.statement', 'INSERT INTO orders ...');
        // ... run query ...
        $dbSpan->setStatus(StatusCode::STATUS_OK);
    } finally {
        $dbSpan->end();
    }

    $span->setStatus(StatusCode::STATUS_OK);
    $span->addEvent('order.created', ['order_id' => 'ORD-8821']);

} catch (\Throwable $e) {
    $span->recordException($e);
    $span->setStatus(StatusCode::STATUS_ERROR, $e->getMessage());
    throw $e;
} finally {
    $scope->detach();
    $span->end();
}
```

#### Python — Tracing with OpenTelemetry

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.trace import StatusCode

# Setup
provider = TracerProvider()
provider.add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="http://otel-collector:4317"))
)
trace.set_tracer_provider(provider)

tracer = trace.get_tracer("order-service")


def create_order(user_id: int, amount: float) -> str:
    with tracer.start_as_current_span("create-order") as span:
        span.set_attribute("user.id", user_id)
        span.set_attribute("order.amount", amount)
        span.set_attribute("service.name", "order-service")

        order_id = save_to_db(user_id, amount)
        send_confirmation_email(order_id)

        span.add_event("order.created", {"order_id": order_id})
        span.set_status(StatusCode.OK)
        return order_id


def save_to_db(user_id: int, amount: float) -> str:
    with tracer.start_as_current_span("mysql.insert") as span:
        span.set_attribute("db.system", "mysql")
        span.set_attribute("db.statement", "INSERT INTO orders ...")
        # ... run query ...
        return "ORD-8821"


def send_confirmation_email(order_id: str) -> None:
    with tracer.start_as_current_span("email.send") as span:
        span.set_attribute("email.type", "order_confirmation")
        span.set_attribute("order.id", order_id)
        # ... send email ...
```

---

### 3.4 The Three Pillars Together

```
User reports: "Checkout was slow at 10:45 AM"

Step 1 → METRICS alert fired at 10:43 AM
         p99 latency jumped from 200ms → 4200ms
         → "Something is wrong" ← metrics told us WHEN

Step 2 → TRACES show which request path was slow
         Span: mysql.query → 3800ms  ← 90% of total time here
         → "MySQL is the problem" ← traces told us WHERE

Step 3 → LOGS from mysql span at 10:43-10:47 AM
         ERROR: "Lock wait timeout exceeded"
         WARNING: "Too many connections: 512/512"
         → "DB connection pool exhausted" ← logs told us WHY
```

**Metrics = WHEN. Traces = WHERE. Logs = WHY.**

---

### 3.5 OpenTelemetry

**OpenTelemetry (OTel)** is an **open standard** for collecting telemetry data (logs, metrics, traces) from your application.

Think of it as: _one SDK, one format, any backend._

#### Why It Exists

Before OTel, every tool had its own SDK:

- Datadog SDK → sends only to Datadog
- Jaeger SDK → sends only to Jaeger
- New Relic SDK → sends only to New Relic

You were **vendor-locked**. Switch tools? Rewrite all your instrumentation.

OTel breaks this:

```
Your App
   ↓
OpenTelemetry SDK (instrument once)
   ↓
OTel Collector (central hub)
   ↓
Send to ANY backend:
   ├── Jaeger (traces)
   ├── Prometheus (metrics)
   ├── Elasticsearch (logs)
   ├── Grafana Tempo (traces)
   ├── CloudWatch (everything)
   └── Datadog (everything)
```

#### OTel Key Components

```
INSTRUMENTATION LIBRARIES
   → Auto-instrument frameworks (Symfony, Django, FastAPI, MySQL, Redis)
   → Manual spans and attributes you add yourself

SDK
   → Collects telemetry from your app
   → PHP SDK, Python SDK, Go SDK, Java SDK...

OTLP PROTOCOL
   → OpenTelemetry Protocol — the wire format (gRPC or HTTP/JSON)
   → Standard format all tools speak

COLLECTOR
   → A standalone service (not in your app)
   → Receives, processes, exports telemetry
   → Can filter, sample, batch, and route to multiple backends
```

#### OTel Collector Config Example

```yaml
# otel-collector-config.yaml
receivers:
  otlp: # receive from your apps
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch: # batch for efficiency
    timeout: 10s
  filter: # drop health check traces
    traces:
      exclude:
        match_type: strict
        span_names: ["health_check"]

exporters:
  jaeger: # traces → Jaeger
    endpoint: jaeger:14250
  prometheus: # metrics → Prometheus
    endpoint: 0.0.0.0:8889
  elasticsearch: # logs → Elasticsearch
    endpoints: ["http://elasticsearch:9200"]

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch, filter]
      exporters: [jaeger]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [elasticsearch]
```

---

### 3.6 Graylog

**Graylog** is a **log management platform** — it collects, stores, searches, and alerts on logs.

#### What it Does

```
Your Services → send logs → Graylog → stores in OpenSearch/Elasticsearch
                                    → lets you search, filter, dashboard
                                    → sends alerts when patterns match
```

#### Key Features

- **GELF format** (Graylog Extended Log Format) — structured JSON logs
- **Streams** — route different logs to different teams/storage
- **Pipelines** — transform/enrich logs as they arrive
- **Alerts** — "If ERROR count > 50 in 5 minutes → send Slack alert"
- **Dashboards** — visualize log patterns

#### PHP — Send to Graylog via GELF

```php
<?php

use Monolog\Logger;
use Monolog\Handler\GelfHandler;
use Gelf\Publisher;
use Gelf\Transport\UdpTransport;

$publisher  = new Publisher(new UdpTransport('graylog.internal', 12201));
$gelfHandler = new GelfHandler($publisher, Logger::DEBUG);

$log = new Logger('order-service');
$log->pushHandler($gelfHandler);

$log->error('Payment failed', [
    'user_id'    => 42,
    'order_id'   => 'ORD-8821',
    'error_code' => 'CARD_DECLINED',
    'gateway'    => 'stripe',
    'trace_id'   => 'abc123def456',
]);
```

#### Python — Send to Graylog

```python
import logging
import graypy  # pip install graypy

logger = logging.getLogger("order-service")
logger.addHandler(
    graypy.GELFUDPHandler("graylog.internal", 12201)
)
logger.setLevel(logging.DEBUG)

logger.error("Payment failed", extra={
    "user_id":    42,
    "order_id":   "ORD-8821",
    "error_code": "CARD_DECLINED",
    "gateway":    "stripe",
})
```

#### When to Choose Graylog

- You want **self-hosted** log management (not cloud-vendor locked)
- You need **fine-grained stream routing** (different teams see different logs)
- Your team is comfortable with operations (you manage the server)
- Budget-conscious — Graylog Open is free

---

### 3.7 Kibana (ELK Stack)

**Kibana** is the **visualization UI** of the ELK Stack.

```
ELK Stack:
E → Elasticsearch  (stores and indexes logs — the database)
L → Logstash       (ingests, parses, transforms logs — the pipeline)
K → Kibana         (UI for search, dashboards, visualizations)

Modern version = ELK + Beats:
Beats (Filebeat, Metricbeat) → lightweight shippers running on each server
```

#### ELK Flow

```
Your App
  ↓
Filebeat (reads log files from disk)
  ↓
Logstash (parse, enrich, filter — e.g. extract fields from plain text logs)
  ↓
Elasticsearch (index and store)
  ↓
Kibana (search UI + dashboards)
```

#### What Kibana Lets You Do

- **Discover** — full-text search across all logs with filters
- **Lens / Visualize** — charts, histograms, pie charts from log data
- **Dashboards** — combine visualizations
- **Alerts** — notify on log patterns
- **APM** — Application Performance Monitoring (traces)
- **SIEM** — Security information and event management

#### Kibana vs Graylog

| Feature          | Kibana (ELK)          | Graylog                    |
| ---------------- | --------------------- | -------------------------- |
| Storage backend  | Elasticsearch         | OpenSearch / Elasticsearch |
| UI focus         | Visualization-first   | Search-first               |
| Setup complexity | Higher                | Lower                      |
| Stream routing   | Less native           | Native strength            |
| Cost             | Free (basic)          | Free (open)                |
| Best for         | Analysis & dashboards | Operational log management |

---

### 3.8 Grafana

**Grafana** is the **metrics visualization and dashboarding tool** — the industry standard for displaying time-series data.

```
Grafana does NOT store data — it READS from data sources:
  ├── Prometheus    (metrics)
  ├── Elasticsearch (logs)
  ├── InfluxDB      (time-series)
  ├── CloudWatch    (AWS metrics)
  ├── Loki          (logs — Grafana's own log tool)
  ├── Tempo         (traces — Grafana's own trace tool)
  ├── MySQL         (direct queries)
  └── 300+ other sources
```

#### Grafana Ecosystem

```
LGTM Stack (Grafana's own full observability):
  L → Loki    (log aggregation — like Elasticsearch but cheaper)
  G → Grafana (visualization)
  T → Tempo   (distributed tracing — like Jaeger)
  M → Mimir   (metrics — like Prometheus but scalable)
```

#### What Grafana Lets You Do

- **Dashboards** — beautiful, real-time panels: graphs, gauges, heatmaps, tables
- **Alerts** — fire on metric thresholds → Slack, PagerDuty, email
- **Explore** — ad-hoc queries on any data source
- **Correlate** — link a metrics spike to logs and traces in one click
- **Annotations** — mark deployments on graphs ("deployed v2.4 here")
- **Provisioning** — dashboards as code (JSON/YAML — version controlled)

#### Prometheus + Grafana (Most Common Setup)

```yaml
# docker-compose.yml — minimal monitoring stack
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports: ["9090:9090"]

  grafana:
    image: grafana/grafana
    ports: ["3000:3000"]
    environment:
      GF_SECURITY_ADMIN_PASSWORD: secret

  your-app:
    build: .
    ports: ["8080:8080"] # exposes /metrics endpoint
```

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "your-app"
    static_configs:
      - targets: ["your-app:8080"]
```

---

### 3.9 CloudWatch

**AWS CloudWatch** is Amazon's **all-in-one observability service** — logs, metrics, traces, dashboards, alerts — built for AWS infrastructure.

```
CloudWatch Features:
  ├── Logs Insights   → search and query logs
  ├── Metrics         → AWS service metrics (EC2, RDS, Lambda, ECS...)
  ├── Alarms          → alert on metric thresholds
  ├── Dashboards      → visualize metrics
  ├── X-Ray           → distributed tracing (AWS-native)
  ├── Container Insights → ECS/EKS monitoring
  └── Application Insights → automated anomaly detection
```

#### When CloudWatch Makes Sense

- Your infrastructure is **entirely on AWS**
- You use **Lambda, ECS, RDS, API Gateway** — they all auto-emit to CloudWatch
- You want **zero infrastructure to manage** for monitoring
- You are okay paying **AWS pricing** (can get expensive at scale)

#### PHP — Send to CloudWatch

```php
<?php

use Aws\CloudWatchLogs\CloudWatchLogsClient;
use Maxbanton\Cwh\Handler\CloudWatch;
use Monolog\Logger;

$client = new CloudWatchLogsClient([
    'region'  => 'eu-central-1',
    'version' => 'latest',
]);

$handler = new CloudWatch($client, '/app/order-service', 'production', 10000);
$log     = new Logger('order-service');
$log->pushHandler($handler);

$log->info('Order created', ['order_id' => 'ORD-8821', 'amount' => 29.99]);
```

#### Python — Send to CloudWatch

```python
import boto3
import logging
import watchtower  # pip install watchtower

boto3_session = boto3.Session(region_name="eu-central-1")

logger = logging.getLogger("order-service")
logger.addHandler(
    watchtower.CloudWatchLogHandler(
        log_group="/app/order-service",
        stream_name="production",
        boto3_session=boto3_session,
    )
)
logger.setLevel(logging.INFO)
logger.info("Order created", extra={"order_id": "ORD-8821", "amount": 29.99})
```

---

### 3.10 Tool Comparison — Which Does What

| Tool              | Type               | Stores Data?   | Primary Use                                |
| ----------------- | ------------------ | -------------- | ------------------------------------------ |
| **OpenTelemetry** | Standard / SDK     | ❌ No          | Instrument code, collect & route telemetry |
| **Prometheus**    | Metrics DB         | ✅ Yes         | Store & query time-series metrics          |
| **Grafana**       | Visualization      | ❌ No          | Dashboard & alert on any data source       |
| **Grafana Loki**  | Log storage        | ✅ Yes         | Store logs (cheaper than Elasticsearch)    |
| **Grafana Tempo** | Trace storage      | ✅ Yes         | Store distributed traces                   |
| **Jaeger**        | Trace UI + storage | ✅ Yes         | Distributed tracing (self-hosted)          |
| **Elasticsearch** | Search DB          | ✅ Yes         | Store and full-text search logs            |
| **Kibana**        | Visualization      | ❌ No          | UI for Elasticsearch data                  |
| **Logstash**      | Log pipeline       | ❌ No          | Ingest, parse, transform logs              |
| **Graylog**       | Log management     | ✅ Yes         | Collect, search, alert on logs             |
| **CloudWatch**    | All-in-one         | ✅ Yes         | AWS-native logs + metrics + traces         |
| **Datadog**       | SaaS all-in-one    | ✅ Yes (cloud) | Full observability as a service            |
| **New Relic**     | SaaS all-in-one    | ✅ Yes (cloud) | Full observability as a service            |

---

## 4. For an Expert Developer 🧠

### 4.1 Structured Logging in Depth

#### Log Correlation — The Most Important Field

Every log entry **must** carry the `trace_id`. This connects your logs to your traces.

```json
{
  "timestamp": "2026-04-25T10:15:33.421Z",
  "level": "ERROR",
  "message": "Payment declined",
  "service": "payment-service",
  "version": "2.4.1",
  "environment": "production",
  "trace_id": "4bf92f3577b34da6",
  "span_id": "00f067aa0ba902b7",
  "user_id": 42,
  "order_id": "ORD-8821",
  "error_code": "CARD_DECLINED",
  "gateway": "stripe",
  "duration_ms": 234,
  "host": "order-pod-7d9f4b-xkp2m",
  "region": "eu-central-1"
}
```

Fields every log should have:

- `trace_id` — link to distributed trace
- `span_id` — link to exact span
- `service` — which microservice
- `version` — which deployment
- `environment` — prod/staging/dev
- `host` / `pod` — which instance

#### Log Sampling

In high-traffic systems you cannot log everything at full verbosity.

```python
import random
import logging


class SampledLogger:
    """Log 100% of ERRORs, 10% of WARNs, 1% of INFOs"""

    SAMPLE_RATES = {
        logging.ERROR:   1.0,    # 100% — always log errors
        logging.WARNING: 0.1,    # 10%
        logging.INFO:    0.01,   # 1%
        logging.DEBUG:   0.001,  # 0.1%
    }

    def __init__(self, logger: logging.Logger) -> None:
        self._logger = logger

    def log(self, level: int, message: str, **kwargs) -> None:
        rate = self.SAMPLE_RATES.get(level, 1.0)
        if random.random() < rate:
            self._logger.log(level, message, extra=kwargs)
```

---

### 4.2 Distributed Tracing Deep Dive

#### Trace Context Propagation

When Service A calls Service B — the trace context must be passed in HTTP headers.

```
Service A creates a span:
  trace_id = 4bf92f3577b34da6
  span_id  = 00f067aa0ba902b7

Service A calls Service B:
  HTTP Header: traceparent: 00-4bf92f3577b34da6-00f067aa0ba902b7-01
                             ↑  ↑ trace_id        ↑ parent_span_id ↑ sampled

Service B receives the header:
  Creates a child span:
  trace_id       = 4bf92f3577b34da6  ← same trace!
  span_id        = a3ce929d0e0e4736  ← new span
  parent_span_id = 00f067aa0ba902b7  ← parent is Service A's span
```

This is the **W3C TraceContext** standard — supported by all OTel-compatible tools.

#### Span Events vs Span Attributes

```python
with tracer.start_as_current_span("process-payment") as span:

    # ATTRIBUTES — facts about the span (indexed, searchable)
    span.set_attribute("payment.method",    "credit_card")
    span.set_attribute("payment.amount",    99.99)
    span.set_attribute("payment.currency",  "EUR")
    span.set_attribute("payment.gateway",   "stripe")

    try:
        result = charge_card()

        # EVENTS — things that happened during the span (timestamped)
        span.add_event("card.authorized", {
            "authorization_code": result.auth_code,
            "network":            "visa"
        })

    except CardDeclinedException as e:
        # Record exception — adds stack trace to span
        span.record_exception(e)
        span.set_status(StatusCode.ERROR, str(e))

        # Event with error context
        span.add_event("card.declined", {
            "decline_code": e.code,
            "retry_count":  3
        })
        raise
```

---

### 4.3 RED Method & USE Method

Two frameworks for deciding WHAT to measure.

#### RED Method — for Services (APIs, microservices)

```
R → Rate        How many requests per second?
E → Errors      How many requests are failing? (and what type?)
D → Duration    How long do requests take? (p50, p95, p99)
```

These three metrics tell you everything about a service's health.

```python
from prometheus_client import Counter, Histogram

# R — Rate (counter, rate calculated in Grafana)
requests_total = Counter(
    "http_requests_total", "Total requests",
    ["method", "endpoint", "status_code"]
)

# E — Errors (filter requests_total where status_code >= 500)
# No separate metric needed — status_code label covers it

# D — Duration (histogram)
request_duration = Histogram(
    "http_request_duration_seconds", "Request duration",
    ["method", "endpoint"],
    buckets=[.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5]
)
```

#### USE Method — for Infrastructure (servers, databases, queues)

```
U → Utilization   How busy is the resource? (CPU %, memory %, disk %)
S → Saturation    How much work is queued / waiting?
E → Errors        Is the resource producing errors?
```

| Resource | Utilization  | Saturation           | Errors                   |
| -------- | ------------ | -------------------- | ------------------------ |
| CPU      | % used       | Run queue length     | Machine check exceptions |
| Memory   | % used       | Swap usage           | OOM kills                |
| MySQL    | % query time | Wait queue depth     | Deadlocks, timeouts      |
| Redis    | % memory     | Rejected connections | Evictions                |
| Disk     | % I/O time   | I/O wait queue       | Write errors             |

---

### 4.4 OpenTelemetry Architecture Deep Dive

```
┌─────────────────────────────────────────────────────────┐
│                    YOUR APPLICATION                     │
│                                                         │
│  Auto-instrumentation        Manual instrumentation     │
│  (HTTP, DB, Redis,           (your business spans,      │
│  Symfony, Django auto)        attributes, events)       │
│           ↓                          ↓                  │
│         OTel SDK                                        │
│  (TracerProvider + MeterProvider + LoggerProvider)      │
│           ↓                                             │
│       OTLP Exporter (gRPC/HTTP)                         │
└─────────────────┬───────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────┐
│               OTEL COLLECTOR (separate service)         │
│                                                         │
│  RECEIVERS    → otlp, jaeger, prometheus, zipkin        │
│  PROCESSORS   → batch, filter, sample, enrich, redact   │
│  EXPORTERS    → jaeger, prometheus, elasticsearch, loki  │
└─────────────────────────────────────────────────────────┘
                  ↓
    ┌─────────────┬─────────────┬──────────────┐
    ↓             ↓             ↓              ↓
  Jaeger      Prometheus   Elasticsearch    Loki
 (traces)     (metrics)      (logs)        (logs)
    ↓             ↓             ↓              ↓
              Grafana        Kibana         Grafana
           (dashboards)   (dashboards)   (dashboards)
```

#### Resource Attributes — What Every Service Must Declare

```python
from opentelemetry.sdk.resources import Resource, SERVICE_NAME, SERVICE_VERSION

resource = Resource.create({
    SERVICE_NAME:               "order-service",
    SERVICE_VERSION:            "2.4.1",
    "service.namespace":        "checkout",
    "deployment.environment":   "production",
    "host.name":                "order-pod-7d9f4b-xkp2m",
    "cloud.provider":           "aws",
    "cloud.region":             "eu-central-1",
    "k8s.pod.name":             "order-pod-7d9f4b-xkp2m",
    "k8s.namespace.name":       "production",
})
```

---

### 4.5 Alerting & SLOs / SLAs / SLIs

#### SLI → SLO → SLA — The Hierarchy

```
SLI (Service Level Indicator)
  → The actual measurement
  → "Our p99 latency is 245ms"
  → "Our error rate is 0.04%"
  → "Our availability is 99.97%"

SLO (Service Level Objective)
  → The TARGET you set for yourself internally
  → "p99 latency MUST be < 500ms"
  → "Error rate MUST be < 0.1%"
  → "Availability MUST be > 99.9% (43.8 minutes downtime/month)"

SLA (Service Level Agreement)
  → The CONTRACTUAL promise to customers
  → Usually less strict than SLO (buffer for error budget)
  → "We guarantee 99.5% uptime"
  → If breached → refunds, penalties
```

#### Error Budget

```
SLO: 99.9% availability = 43.8 minutes downtime allowed per month
Current: 99.95% availability = 21.9 minutes downtime used

Error budget remaining: 43.8 - 21.9 = 21.9 minutes

If error budget is > 50% remaining → deploy freely
If error budget is < 10% remaining → freeze risky deployments, focus on reliability
If error budget is 0 → stop all features, emergency reliability work only
```

#### Alerting Rules in Prometheus / Grafana

```yaml
# prometheus-alerts.yml
groups:
  - name: order-service
    rules:
      # Error rate > 1% for 5 minutes
      - alert: HighErrorRate
        expr: |
          rate(http_requests_total{status_code=~"5.."}[5m])
          /
          rate(http_requests_total[5m]) > 0.01
        for: 5m
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate on {{ $labels.service }}"
          description: "Error rate is {{ $value | humanizePercentage }}"

      # p99 latency > 1s for 10 minutes
      - alert: SlowP99Latency
        expr: |
          histogram_quantile(0.99,
            rate(http_request_duration_seconds_bucket[5m])
          ) > 1.0
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "p99 latency > 1s on {{ $labels.service }}"
```

---

### 4.6 Log Aggregation Pipelines

#### Full Production Pipeline

```
Application Containers
  ↓ stdout/stderr
Kubernetes (collects container logs)
  ↓
Fluentd / Filebeat (log shipper — runs as DaemonSet)
  ↓ enriches with pod name, namespace, labels
Logstash (parsing, field extraction, transformation)
  ↓
Elasticsearch (storage + indexing)
  ↓
Kibana (search, dashboards, alerts)
     ↑
  Grafana (alternative visualization via Elasticsearch datasource)
```

#### Logstash Pipeline Config

```ruby
# logstash.conf
input {
  beats { port => 5044 }     # from Filebeat
}

filter {
  json { source => "message" }       # parse JSON logs

  # Add geo data from IP
  geoip { source => "client_ip" }

  # Drop health check noise
  if [http_route] == "/health" {
    drop {}
  }

  # Enrich with deployment info
  mutate {
    add_field => {
      "datacenter" => "eu-central-1"
    }
  }
}

output {
  elasticsearch {
    hosts      => ["elasticsearch:9200"]
    index      => "logs-%{service}-%{+YYYY.MM.dd}"
  }
}
```

---

### 4.7 Cardinality Problem in Metrics

**Cardinality** = the number of unique combinations of label values in a metric.

This is the **most dangerous performance issue** in Prometheus/metrics systems.

```python
# ❌ BAD — user_id as a label = millions of unique series
http_requests = Counter(
    "http_requests_total", "Requests",
    ["method", "route", "user_id"]   # ← NEVER do this!
)
# 1000 users × 50 routes × 4 methods = 200,000 time series
# 1M users × 50 routes × 4 methods = 200,000,000 series → Prometheus dies

# ✅ GOOD — low cardinality labels only
http_requests = Counter(
    "http_requests_total", "Requests",
    ["method", "route", "status_code"]  # maybe 50 × 4 × 10 = 2,000 series
)

# If you need per-user data → use LOGS, not metrics
# Logs are made for high-cardinality data
# Metrics are made for low-cardinality aggregations
```

**Rule of thumb:** A label value should have fewer than 100 unique values. If it doesn't — it belongs in logs, not metrics.

---

### 4.8 Sampling Strategies

Full tracing of every request in high-traffic systems is expensive.

#### Head Sampling — Decision Made at Start

```python
from opentelemetry.sdk.trace.sampling import TraceIdRatioBased, ParentBased

# Sample 10% of all traces randomly
sampler = ParentBased(
    root=TraceIdRatioBased(0.1)  # 10% of root spans sampled
)
# Problem: you might miss the 1 error in 10 requests
```

#### Tail Sampling — Decision Made at End

```yaml
# OTel Collector tail sampling processor
processors:
  tail_sampling:
    decision_wait: 10s # wait 10s to see full trace before deciding
    policies:
      # Always sample errors — regardless of rate
      - name: errors-policy
        type: status_code
        status_code: { status_codes: [ERROR] }

      # Always sample slow traces > 1s
      - name: slow-traces-policy
        type: latency
        latency: { threshold_ms: 1000 }

      # Sample only 5% of normal/fast traces
      - name: low-rate-policy
        type: probabilistic
        probabilistic: { sampling_percentage: 5 }
```

Tail sampling is better — you never lose important signals (errors, slow requests). You only drop "boring" successful fast requests.

---

### 4.9 Correlation — Tying It All Together

The holy grail of observability: clicking from a metric alert → finding the trace → jumping to the logs for that trace.

```python
# Python — inject trace context into every log automatically
from opentelemetry import trace
import logging


class TraceContextFilter(logging.Filter):
    """Automatically add trace_id and span_id to every log record"""

    def filter(self, record: logging.LogRecord) -> bool:
        span = trace.get_current_span()
        ctx  = span.get_span_context()

        if ctx and ctx.is_valid:
            record.trace_id = format(ctx.trace_id, "032x")
            record.span_id  = format(ctx.span_id,  "016x")
            record.sampled  = ctx.trace_flags.sampled
        else:
            record.trace_id = "0" * 32
            record.span_id  = "0" * 16
            record.sampled  = False

        return True


logger = logging.getLogger("order-service")
logger.addFilter(TraceContextFilter())

# Now every log automatically includes trace_id + span_id
# Grafana / Kibana / Graylog can link log → trace automatically
```

---

### 4.10 PHP + Python Implementation — Production-Ready Setup

#### docker-compose — Full Observability Stack

```yaml
version: "3.9"
services:
  # Your PHP App
  php-app:
    build: ./php
    environment:
      OTEL_SERVICE_NAME: "order-service"
      OTEL_SERVICE_VERSION: "2.4.1"
      OTEL_EXPORTER_OTLP_ENDPOINT: "http://otel-collector:4317"

  # Your Python App
  python-app:
    build: ./python
    environment:
      OTEL_SERVICE_NAME: "payment-service"
      OTEL_EXPORTER_OTLP_ENDPOINT: "http://otel-collector:4318"

  # OTel Collector — central hub
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    volumes:
      - ./otel-config.yml:/etc/otelcol/config.yaml

  # Prometheus — metrics storage
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  # Grafana — dashboards
  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]

  # Grafana Tempo — traces
  tempo:
    image: grafana/tempo:latest

  # Grafana Loki — logs
  loki:
    image: grafana/loki:latest

  # Jaeger — trace UI (alternative)
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports: ["16686:16686"]
```

---

## 5. For the Critics 🔥

### Criticism 1 — Observability Is a Solved Problem That Nobody Implements

**The Argument:**

> "Every conference talk says 'instrument everything, structured logs, trace every request.' Every real production system I've worked in has inconsistent log formats, no trace IDs, dashboards nobody looks at, and alerts that fire so often they're ignored. Observability is aspirational, not practical."

**Valid — and this is a culture problem more than a tech problem.**
The tools exist and work. The failure mode is: teams add observability as an afterthought, alerts are never tuned (alert fatigue kills the signal), and nobody owns the dashboards. The fix is SLO-driven alerting — alert on what users feel, not every metric. And observability must be defined in code (OTel), not bolted on post-hoc.

---

### Criticism 2 — OpenTelemetry Is the New Vendor Lock-In

**The Argument:**

> "OTel promises vendor freedom but the Collector config is complex, the SDKs are heavy, auto-instrumentation breaks silently, and every vendor's 'OTel support' is subtly different. You traded one lock-in for complexity-in."

**Partially valid.**
OTel's collector config YAML is genuinely complex. Auto-instrumentation for PHP is still maturing. Vendor OTel support has gaps. But: the alternative — using Datadog SDK directly — is a hard lock-in. OTel gives you the freedom to switch backends without rewriting instrumentation code. The complexity cost is real but one-time. The vendor lock-in cost compounds forever.

---

### Criticism 3 — Prometheus Does Not Scale

**The Argument:**

> "Prometheus is a single binary with local storage. At scale (100+ microservices, millions of series), it becomes a single point of failure with a storage problem. The answer is Thanos or Cortex — which are Kubernetes-native distributed systems that most teams don't know how to operate."

**Correct — with context.**
Prometheus is designed for single-cluster, single-datacenter use. At Flixbus, Zalando, or similar scale you need Thanos (long-term storage + HA) or Grafana Mimir. This is a real operational burden. But: 95% of companies never hit this scale problem. Start with Prometheus. Add Thanos when metrics storage becomes a pain. Don't over-engineer before you need it.

---

### Criticism 4 — Logs Are Not Observability

**The Argument:**

> "Teams that rely primarily on logs for debugging are doing archaeology, not observability. Logs are unstructured archaeology through text. Real observability comes from traces and metrics — logs are the fallback when traces don't exist yet."

**Overstated but directionally correct.**
Logs alone are insufficient in distributed systems — you cannot correlate a log entry in Service A with what happened in Service B. But logs WITH trace_id injection become correlated records. The modern answer is not "logs OR traces" — it is logs + traces + metrics linked by trace_id. Logs provide the WHY that traces and metrics cannot.

---

### Criticism 5 — CloudWatch Pricing Is Predatory

**The Argument:**

> "CloudWatch seems free until you actually use it. Custom metrics: $0.30 per metric per month. Log ingestion: $0.50/GB. Log storage: $0.03/GB. API calls to retrieve logs: $0.01 per 1000. A reasonably instrumented system of 20 microservices can cost $2000+/month in CloudWatch alone — for a tool that is objectively worse than the open-source stack."

**Factually accurate.**
CloudWatch's pricing is opaque and scales poorly. Teams that start with CloudWatch because "it's built-in" often migrate to the LGTM stack or ELK at scale. The counterargument is operational cost — self-hosting Prometheus + Grafana + Loki + Tempo requires engineering time to maintain. CloudWatch's price includes that operational overhead. Both are real costs. CloudWatch makes sense for small AWS teams. The open-source stack wins at medium-to-large scale.

---

### Criticism 6 — Alert Fatigue Kills Observability ROI

**The Argument:**

> "Every monitoring system eventually degrades to: too many alerts → team stops reading them → incident happens → everyone is surprised. Observability tooling has not solved the fundamental problem of signal vs noise."

**This is the hardest problem in production engineering.**
The answer is SLO-based alerting — only alert on user-visible symptoms, not every internal metric. Alert on "error rate > 1% for 5 minutes" not "CPU > 70%". Alert on "p99 > 2s" not "a single slow query". Most teams alert on causes, not symptoms — and get buried. The tooling supports SLO alerting (Prometheus recording rules, Grafana SLO panels) — but the discipline to use it is rare.

---

## 6. Full Tool Comparison Table

|                 | OpenTelemetry   | Prometheus | Grafana     | ELK / Kibana | Graylog      | CloudWatch  | Datadog         | Jaeger    |
| --------------- | --------------- | ---------- | ----------- | ------------ | ------------ | ----------- | --------------- | --------- |
| **Type**        | SDK + Standard  | Metrics DB | Viz UI      | Log platform | Log platform | All-in-one  | SaaS all-in-one | Trace UI  |
| **Stores data** | ❌              | ✅         | ❌          | ✅           | ✅           | ✅          | ✅              | ✅        |
| **Logs**        | ✅ (collect)    | ❌         | Via Loki    | ✅           | ✅           | ✅          | ✅              | ❌        |
| **Metrics**     | ✅ (collect)    | ✅         | Via sources | Limited      | Limited      | ✅          | ✅              | ❌        |
| **Traces**      | ✅ (collect)    | ❌         | Via Tempo   | Via APM      | ❌           | Via X-Ray   | ✅              | ✅        |
| **Self-hosted** | ✅              | ✅         | ✅          | ✅           | ✅           | ❌ (AWS)    | ❌ (SaaS)       | ✅        |
| **Cost**        | Free            | Free       | Free/Pro    | Free/Paid    | Free/Paid    | Pay-per-use | Expensive       | Free      |
| **Complexity**  | Medium          | Low        | Low         | High         | Medium       | Low         | Low             | Low       |
| **Best for**    | Instrumentation | Metrics    | Dashboards  | Log search   | Log ops      | AWS teams   | Large budgets   | Trace viz |

---

## 7. Decision Guide

### What Stack Should I Use?

```
Where does your infrastructure live?
│
├── Entirely on AWS?
│       └── Start with CloudWatch + X-Ray
│           When cost hurts → migrate to LGTM stack
│
├── Kubernetes (self-managed)?
│       └── LGTM Stack:
│           Prometheus + Grafana + Loki + Tempo
│           All work natively in K8s, Helm charts available
│
├── Mixed / multi-cloud?
│       └── OpenTelemetry Collector → route to self-hosted backends
│           Full flexibility, no vendor lock-in
│
└── Small team, just need it to work?
        └── Managed: Grafana Cloud (free tier) or Datadog
            Higher cost, lower ops burden
```

### Which Pillar to Implement First?

```
Start with LOGS (highest immediate value)
   ↓
Add METRICS + Grafana dashboards (visibility into system health)
   ↓
Add TRACES (needed when you have 3+ services / hard-to-debug latency)
   ↓
Add ALERTING based on SLOs (not every metric)
   ↓
Tune sampling, cardinality, correlation
```

### What Do I Absolutely Need in Production?

| Must Have               | Nice to Have            | Advanced                 |
| ----------------------- | ----------------------- | ------------------------ |
| Structured JSON logs    | Log pipeline (Logstash) | Tail-based sampling      |
| Centralized log storage | Distributed tracing     | SLO error budgets        |
| trace_id in every log   | Prometheus metrics      | Anomaly detection        |
| Grafana dashboards      | Alerting rules          | Cardinality management   |
| Error rate alert        | p99 latency tracking    | Multi-cluster Prometheus |
| On-call runbooks        | Deployment annotations  | Continuous profiling     |

---

## Final One-Liner Summary

| Audience         | Summary                                                                                                                                                                                                                                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🧸 5-year-old    | Logs = diary. Metrics = speedometer dashboard. Traces = GPS tracker per pizza. Tools are the office staff who collect and show all of this.                                                                                                                                                             |
| 💻 Mid-level dev | Three pillars: Logs (what happened), Metrics (how much/fast), Traces (which path). OpenTelemetry instruments everything. Prometheus stores metrics. Grafana shows them. ELK/Graylog handle logs. CloudWatch does all of it if you're on AWS.                                                            |
| 🧠 Expert dev    | Correlate logs + traces via trace_id injection. Use RED method for services, USE method for infrastructure. Tail sampling > head sampling for important signal. Watch cardinality in Prometheus. SLO-driven alerting kills alert fatigue. OTel Collector as routing hub gives full backend flexibility. |
| 🔥 Critics       | Observability is a culture problem as much as a tooling problem. OTel is complex but beats vendor lock-in. Prometheus doesn't scale past single cluster. CloudWatch pricing is opaque and expensive. Alert fatigue is unsolved. Logs alone are archaeology — traces give you the map.                   |

---

_Happy Debugging! 🔍🚀_
