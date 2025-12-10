# Chapter 12: Monitoring

## Table of Contents
- [12.1 Golden Signals](#121-golden-signals)
- [12.2 Types of Metrics](#122-types-of-metrics)
- [12.3 Recommended Practices](#123-recommended-practices)
- [12.4 Collecting Metrics](#124-collecting-metrics)
- [12.5 Instrumenting](#125-instrumenting)
- [12.6 Raising Sensible & Actionable Alerts](#126-raising-sensible--actionable-alerts)
- [12.7 Using Logs & Traces](#127-using-logs--traces)
- [12.8 Useful Info in Log Entries](#128-useful-info-in-log-entries)
- [12.9 Tools for Logging](#129-tools-for-logging)
- [12.10 Logging the Right Information](#1210-logging-the-right-information)
- [12.11 Tracing Interaction Between Services](#1211-tracing-interaction-between-services)
- [12.12 Visualizing Traces](#1212-visualizing-traces)

---

## 12.1. Golden Signals

### What are Golden Signals?

The **Four Golden Signals** (from Google's SRE Book) are the key metrics to monitor for any system:

```
┌──────────────────────────────────────────────┐
│         THE FOUR GOLDEN SIGNALS              │
├──────────────────────────────────────────────┤
│  1. LATENCY    - How long requests take      │
│  2. TRAFFIC    - How much demand on system   │
│  3. ERRORS     - Rate of failed requests     │
│  4. SATURATION - How full the system is      │
└──────────────────────────────────────────────┘
```

### 1. Latency

**Definition**: Time taken to service a request

**Why Important**:
- Direct impact on user experience
- Indicates system health
- Shows performance degradation early

**What to Measure**:
```
- Response time (p50, p95, p99)
- API endpoint latency
- Database query time
- Network latency
- Cache hit/miss latency
```

**Example Metrics**:
```bash
# Request duration histogram
http_request_duration_seconds{method="GET", endpoint="/api/users"}

# Percentiles
http_request_duration_seconds{quantile="0.5"}   # p50 (median)
http_request_duration_seconds{quantile="0.95"}  # p95
http_request_duration_seconds{quantile="0.99"}  # p99
```

**Thresholds**:
- p50 < 100ms (Good)
- p95 < 500ms (Acceptable)
- p99 < 1s (Warning threshold)

### 2. Traffic

**Definition**: Measure of demand on your system

**Why Important**:
- Capacity planning
- Detect unusual patterns
- Understand growth trends

**What to Measure**:
```
- Requests per second (RPS)
- Transactions per second (TPS)
- Concurrent users
- Data throughput (MB/s)
- Message queue depth
```

**Example Metrics**:
```bash
# HTTP requests total
http_requests_total{method="GET", status="200"}

# Rate of requests (per second)
rate(http_requests_total[5m])

# Network I/O
container_network_receive_bytes_total
container_network_transmit_bytes_total
```

### 3. Errors

**Definition**: Rate of requests that fail

**Why Important**:
- Immediate indicator of problems
- Shows reliability issues
- Customer-facing impact

**What to Measure**:
```
- HTTP 5xx errors
- HTTP 4xx errors
- Application exceptions
- Failed database transactions
- Circuit breaker opens
```

**Example Metrics**:
```bash
# Error rate
http_requests_total{status=~"5.*"}

# Error percentage
(
  sum(rate(http_requests_total{status=~"5.*"}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) * 100
```

**Thresholds**:
- Error rate < 0.1% (Excellent)
- Error rate < 1% (Good)
- Error rate > 5% (Critical)

### 4. Saturation

**Definition**: How "full" your service is

**Why Important**:
- Predicts future problems
- Prevents outages
- Guides scaling decisions

**What to Measure**:
```
- CPU utilization
- Memory usage
- Disk I/O
- Network bandwidth
- Thread pool usage
- Database connection pool
```

**Example Metrics**:
```bash
# CPU usage
container_cpu_usage_seconds_total

# Memory usage
container_memory_usage_bytes / container_memory_limit_bytes * 100

# Disk usage
node_filesystem_avail_bytes / node_filesystem_size_bytes * 100
```

**Thresholds**:
- CPU < 70% (Safe)
- CPU 70-85% (Warning)
- CPU > 85% (Critical)
- Memory < 80% (Safe)
- Memory > 90% (Critical)

### Practical Example: Monitoring a Web Service

```yaml
# Prometheus alert rules for Golden Signals
groups:
  - name: golden_signals
    rules:
      # Latency Alert
      - alert: HighLatency
        expr: histogram_quantile(0.95, http_request_duration_seconds) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is {{ $value }}s"

      # Traffic Alert
      - alert: TrafficSpike
        expr: rate(http_requests_total[5m]) > 1000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Traffic spike detected"

      # Error Rate Alert
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.*"}[5m]))
            /
            sum(rate(http_requests_total[5m]))
          ) * 100 > 5
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate: {{ $value }}%"

      # Saturation Alert
      - alert: HighCPUUsage
        expr: container_cpu_usage_seconds_total > 0.85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "CPU usage is {{ $value }}%"
```

---

## 12.2. Types of Metrics

### Classification of Metrics

```
METRICS TYPES
├── Counters (always increasing)
├── Gauges (can go up or down)
├── Histograms (distribution of values)
└── Summaries (calculated quantiles)
```

### 1. Counters

**Definition**: Cumulative metric that only increases

**Use Cases**:
- Total number of requests
- Total errors
- Total bytes sent
- Total completed tasks

**Example**:
```python
from prometheus_client import Counter

# Define counter
http_requests_total = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

# Increment counter
http_requests_total.labels(method='GET', endpoint='/api', status='200').inc()
```

**Query Examples**:
```promql
# Total requests
http_requests_total

# Rate of increase (per second)
rate(http_requests_total[5m])

# Increase over time
increase(http_requests_total[1h])
```

### 2. Gauges

**Definition**: Metric that can increase or decrease

**Use Cases**:
- Current CPU usage
- Memory consumption
- Number of active connections
- Queue depth
- Temperature

**Example**:
```python
from prometheus_client import Gauge

# Define gauge
active_connections = Gauge(
    'active_connections',
    'Number of active connections'
)

# Set value
active_connections.set(42)

# Increment/Decrement
active_connections.inc()  # Add 1
active_connections.dec()  # Subtract 1
active_connections.inc(5)  # Add 5
```

**Query Examples**:
```promql
# Current value
active_connections

# Average over time
avg_over_time(active_connections[5m])

# Max value
max_over_time(active_connections[1h])
```

### 3. Histograms

**Definition**: Samples observations and counts them in configurable buckets

**Use Cases**:
- Request durations
- Response sizes
- Query execution times

**Example**:
```python
from prometheus_client import Histogram

# Define histogram
request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    buckets=[0.1, 0.5, 1.0, 2.0, 5.0]
)

# Observe value
request_duration.observe(0.7)
```

**Generated Metrics**:
```
http_request_duration_seconds_bucket{le="0.1"}  # Count <= 0.1s
http_request_duration_seconds_bucket{le="0.5"}  # Count <= 0.5s
http_request_duration_seconds_sum                # Sum of all values
http_request_duration_seconds_count              # Total count
```

**Query Examples**:
```promql
# Calculate p95
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Average duration
rate(http_request_duration_seconds_sum[5m])
/
rate(http_request_duration_seconds_count[5m])
```

### 4. Summaries

**Definition**: Similar to histograms, but calculates quantiles on the client side

**Use Cases**:
- Pre-calculated percentiles
- When bucket boundaries are unknown

**Example**:
```python
from prometheus_client import Summary

# Define summary
request_latency = Summary(
    'request_latency_seconds',
    'Request latency',
    ['endpoint']
)

# Observe value
request_latency.labels(endpoint='/api').observe(0.5)
```

### Business Metrics

Beyond technical metrics, track business KPIs:

```python
# Orders placed
orders_total = Counter('orders_total', 'Total orders')

# Revenue
revenue_dollars = Counter('revenue_dollars_total', 'Total revenue')

# Active users
active_users = Gauge('active_users', 'Current active users')

# Conversion rate
conversion_rate = Gauge('conversion_rate', 'Signup conversion rate')
```

### RED Method (Alternative to Golden Signals)

**For each service, measure:**
- **R**ate: Number of requests per second
- **E**rrors: Number of failed requests
- **D**uration: Time taken for requests

```promql
# Rate
sum(rate(http_requests_total[5m])) by (service)

# Errors
sum(rate(http_requests_total{status=~"5.*"}[5m])) by (service)

# Duration
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))
```

### USE Method (For Resources)

**For each resource, measure:**
- **U**tilization: How busy the resource is
- **S**aturation: Queued work
- **E**rrors: Error count

```promql
# Utilization
container_cpu_usage_seconds_total

# Saturation
container_memory_working_set_bytes / container_spec_memory_limit_bytes

# Errors
node_network_receive_errs_total
```

---

## 12.3. Recommended Practices

### 1. Monitor What Matters

**Do:**
- Focus on user-impacting metrics
- Monitor Golden Signals first
- Track business KPIs
- Measure SLIs (Service Level Indicators)

**Don't:**
- Monitor everything indiscriminately
- Create vanity metrics
- Over-complicate dashboards

### 2. Set Meaningful Thresholds

```yaml
# Good: Based on SLOs
- alert: APILatencyHigh
  expr: histogram_quantile(0.95, http_request_duration_seconds) > 0.5
  for: 5m

# Bad: Arbitrary threshold
- alert: SomeRandomMetric
  expr: some_metric > 100
```

### 3. Use Labels Wisely

**Good Labels**:
```python
http_requests_total{
    method="GET",
    endpoint="/api/users",
    status="200",
    environment="production"
}
```

**Avoid High Cardinality**:
```python
# Bad: Unique user IDs create too many time series
requests_total{user_id="12345"}

# Bad: Timestamps
requests_total{timestamp="2024-01-01T10:00:00Z"}
```

### 4. Implement SLOs (Service Level Objectives)

**Example SLOs**:
```yaml
Service: User API
SLI: Request success rate
SLO: 99.9% of requests succeed
Error Budget: 0.1% = 43 minutes downtime/month

Service: Search API
SLI: Response time
SLO: 95% of requests complete in < 200ms
```

**Calculating Error Budget**:
```promql
# Success rate over 30 days
1 - (
  sum(rate(http_requests_total{status=~"5.*"}[30d]))
  /
  sum(rate(http_requests_total[30d]))
)

# Error budget consumption
100 - (success_rate * 100)
```

### 5. Dashboard Best Practices

**Structure**:
```
Dashboard: Service Overview
├── Golden Signals Row
│   ├── Latency (p50, p95, p99)
│   ├── Traffic (RPS)
│   ├── Error Rate
│   └── Saturation (CPU, Memory)
├── Resource Utilization Row
│   ├── CPU Usage
│   ├── Memory Usage
│   ├── Disk I/O
│   └── Network I/O
└── Business Metrics Row
    ├── Active Users
    ├── Transactions
    └── Revenue
```

**Dashboard Guidelines**:
- One screen, no scrolling for overview
- Use consistent color schemes
- Red = bad, green = good, yellow = warning
- Show trends with time-series graphs
- Include links to runbooks

### 6. Retention Policies

```yaml
# Example Prometheus retention config
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Storage
storage:
  tsdb:
    retention.time: 15d  # Raw data: 15 days
    retention.size: 50GB

# Downsampling (if using VictoriaMetrics/Thanos)
# 5m resolution: 90 days
# 1h resolution: 1 year
```

### 7. Naming Conventions

**Prometheus Metric Names**:
```
Format: <namespace>_<name>_<unit>

Examples:
http_request_duration_seconds
http_requests_total
process_cpu_seconds_total
node_memory_bytes
```

**Label Names**:
```
Use lowercase with underscores:
  method, endpoint, status_code

Avoid:
  Method, end-point, StatusCode
```

### 8. Documentation

**Document Every Alert**:
```yaml
- alert: HighErrorRate
  expr: error_rate > 5
  for: 5m
  annotations:
    summary: "High error rate detected"
    description: "Error rate is {{ $value }}%"
    runbook_url: "https://wiki.company.com/runbooks/high-error-rate"
    dashboard: "https://grafana.company.com/d/errors"
```

---

## 12.4. Collecting Metrics

### Prometheus Architecture

```
┌─────────────────────────────────────────────────┐
│                  Prometheus                      │
│  ┌──────────┐    ┌──────────┐   ┌──────────┐  │
│  │ Scraper  │───▶│  Storage │──▶│   API    │  │
│  └──────────┘    └──────────┘   └──────────┘  │
└─────────────────────────────────────────────────┘
         │                              │
         │ Pull metrics                 │ Queries
         ▼                              ▼
┌──────────────────┐           ┌──────────────┐
│   Applications   │           │   Grafana    │
│  with /metrics   │           │  AlertMgr    │
└──────────────────┘           └──────────────┘
```

### Setting Up Prometheus

**1. Install Prometheus**:

`docker-compose.yml`:
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=15d'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

volumes:
  prometheus-data:
  grafana-data:
```

**2. Configure Prometheus**:

`prometheus.yml`:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # Scrape Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Scrape Node Exporter (system metrics)
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  # Scrape your application
  - job_name: 'myapp'
    static_configs:
      - targets: ['myapp:8080']
    metrics_path: '/metrics'
```

**3. Run Prometheus**:
```bash
# Start with Docker Compose
docker-compose up -d

# Verify Prometheus is running
curl http://localhost:9090/-/healthy

# Access UI
open http://localhost:9090

# Check targets
curl http://localhost:9090/api/v1/targets
```

### Exporters for Different Systems

#### Node Exporter (System Metrics)

```bash
# Run Node Exporter
docker run -d \
  --name node-exporter \
  -p 9100:9100 \
  --net="host" \
  prom/node-exporter

# Test metrics endpoint
curl http://localhost:9100/metrics
```

**Available Metrics**:
```
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_filesystem_avail_bytes
node_network_receive_bytes_total
node_disk_io_time_seconds_total
```

#### cAdvisor (Container Metrics)

```bash
# Run cAdvisor
docker run -d \
  --name cadvisor \
  -p 8080:8080 \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  google/cadvisor:latest

# Test
curl http://localhost:8080/metrics
```

#### MySQL Exporter

```bash
docker run -d \
  --name mysql-exporter \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="user:password@(mysql:3306)/" \
  prom/mysqld-exporter
```

#### Redis Exporter

```bash
docker run -d \
  --name redis-exporter \
  -p 9121:9121 \
  oliver006/redis_exporter \
  --redis.addr=redis://redis:6379
```

### Push vs Pull Metrics

**Pull Model (Prometheus)**:
```yaml
# Prometheus scrapes targets
scrape_configs:
  - job_name: 'myapp'
    scrape_interval: 30s
    static_configs:
      - targets: ['app1:8080', 'app2:8080']
```

**Push Model (Pushgateway)**:
```bash
# For batch jobs that can't be scraped

# Run Pushgateway
docker run -d -p 9091:9091 prom/pushgateway

# Push metrics from batch job
echo "batch_job_duration_seconds 45.2" | \
  curl --data-binary @- \
  http://pushgateway:9091/metrics/job/batch_job/instance/job1
```

---

## 12.5. Instrumenting

### Instrumenting Applications

#### Python Flask Application

```python
from flask import Flask
from prometheus_client import Counter, Histogram, Gauge, generate_latest
import time

app = Flask(__name__)

# Define metrics
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_DURATION = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

ACTIVE_REQUESTS = Gauge(
    'http_requests_active',
    'Active HTTP requests'
)

# Middleware to track requests
@app.before_request
def before_request():
    request.start_time = time.time()
    ACTIVE_REQUESTS.inc()

@app.after_request
def after_request(response):
    request_duration = time.time() - request.start_time
    
    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.endpoint,
        status=response.status_code
    ).inc()
    
    REQUEST_DURATION.labels(
        method=request.method,
        endpoint=request.endpoint
    ).observe(request_duration)
    
    ACTIVE_REQUESTS.dec()
    
    return response

# Application endpoints
@app.route('/')
def home():
    return 'Hello World!'

@app.route('/api/users')
def users():
    time.sleep(0.1)  # Simulate work
    return {'users': []}

# Metrics endpoint
@app.route('/metrics')
def metrics():
    return generate_latest()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

#### Node.js Express Application

```javascript
const express = require('express');
const client = require('prom-client');

const app = express();

// Create a Registry
const register = new client.Registry();

// Add default metrics
client.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status']
});

// Register metrics
register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestsTotal);

// Middleware to track requests
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    
    httpRequestDuration.labels(
      req.method,
      req.route?.path || req.path,
      res.statusCode
    ).observe(duration);
    
    httpRequestsTotal.labels(
      req.method,
      req.route?.path || req.path,
      res.statusCode
    ).inc();
  });
  
  next();
});

// Application routes
app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.get('/api/users', (req, res) => {
  setTimeout(() => {
    res.json({ users: [] });
  }, 100);
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

#### Go Application

```go
package main

import (
    "net/http"
    "time"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )
    
    httpRequestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "endpoint"},
    )
)

func init() {
    prometheus.MustRegister(httpRequestsTotal)
    prometheus.MustRegister(httpRequestDuration)
}

func instrumentedHandler(handler http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // Call original handler
        handler(w, r)
        
        duration := time.Since(start).Seconds()
        
        // Record metrics
        httpRequestsTotal.WithLabelValues(
            r.Method,
            r.URL.Path,
            "200",
        ).Inc()
        
        httpRequestDuration.WithLabelValues(
            r.Method,
            r.URL.Path,
        ).Observe(duration)
    }
}

func main() {
    http.HandleFunc("/", instrumentedHandler(func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello World!"))
    }))
    
    http.Handle("/metrics", promhttp.Handler())
    
    http.ListenAndServe(":8080", nil)
}
```

### Custom Business Metrics

```python
from prometheus_client import Counter, Gauge

# Business metrics
orders_total = Counter('orders_total', 'Total orders placed', ['product_type'])
revenue_dollars = Counter('revenue_dollars_total', 'Total revenue')
active_users = Gauge('active_users_count', 'Currently active users')
cart_abandonment = Counter('cart_abandonment_total', 'Cart abandonments')

# Track business events
def place_order(product_type, amount):
    orders_total.labels(product_type=product_type).inc()
    revenue_dollars.inc(amount)

def user_login():
    active_users.inc()

def user_logout():
    active_users.dec()

def abandon_cart():
    cart_abandonment.inc()
```

### Database Query Instrumentation

```python
import time
from prometheus_client import Histogram

db_query_duration = Histogram(
    'db_query_duration_seconds',
    'Database query duration',
    ['query_type', 'table']
)

@db_query_duration.labels(query_type='SELECT', table='users').time()
def get_users():
    # Database query
    return db.execute("SELECT * FROM users")

# Or manual timing
def get_products():
    with db_query_duration.labels(query_type='SELECT', table='products').time():
        return db.execute("SELECT * FROM products")
```

---

## 12.6. Raising Sensible & Actionable Alerts

### Alert Design Principles

**Good Alert Characteristics**:
1. **Actionable**: Requires human intervention
2. **Urgent**: Needs immediate attention
3. **User-impacting**: Affects end users
4. **Novel**: New information, not noise

**Bad Alerts** (Anti-patterns):
- Alerts that don't require action
- Duplicate alerts
- Too sensitive (frequent false positives)
- Too broad ("something is wrong")

### Alert Structure

```yaml
- alert: AlertName
  expr: <PromQL expression>
  for: <duration>
  labels:
    severity: critical|warning|info
    team: backend|frontend|infra
  annotations:
    summary: <short description>
    description: <detailed description with values>
    runbook_url: <link to runbook>
    dashboard: <link to dashboard>
```

### Example Alert Rules

**1. Latency Alerts**:
```yaml
- alert: HighLatency
  expr: |
    histogram_quantile(0.95,
      rate(http_request_duration_seconds_bucket[5m])
    ) > 1
  for: 5m
  labels:
    severity: warning
    team: backend
  annotations:
    summary: "API latency is high"
    description: "95th percentile latency is {{ $value }}s (threshold: 1s)"
    runbook_url: "https://wiki.company.com/runbooks/high-latency"
    dashboard: "https://grafana.company.com/d/latency"

- alert: CriticalLatency
  expr: |
    histogram_quantile(0.95,
      rate(http_request_duration_seconds_bucket[5m])
    ) > 5
  for: 2m
  labels:
    severity: critical
    team: backend
  annotations:
    summary: "API latency is critically high"
    description: "95th percentile latency is {{ $value }}s"
```

**2. Error Rate Alerts**:
```yaml
- alert: HighErrorRate
  expr: |
    (
      sum(rate(http_requests_total{status=~"5.*"}[5m]))
      /
      sum(rate(http_requests_total[5m]))
    ) * 100 > 5
  for: 5m
  labels:
    severity: critical
    team: backend
  annotations:
    summary: "High error rate detected"
    description: "Error rate is {{ $value | humanizePercentage }} (threshold: 5%)"
    runbook_url: "https://wiki.company.com/runbooks/high-errors"

- alert: ErrorRateSpike
  expr: |
    (
      rate(http_requests_total{status=~"5.*"}[5m])
      /
      rate(http_requests_total{status=~"5.*"}[1h] offset 1h)
    ) > 2
  for: 2m
  labels:
    severity: warning
  annotations:
    summary: "Error rate spike detected"
    description: "Error rate increased by {{ $value }}x"
```

**3. Saturation Alerts**:
```yaml
- alert: HighCPUUsage
  expr: |
    (
      rate(container_cpu_usage_seconds_total[5m])
    ) * 100 > 80
  for: 10m
  labels:
    severity: warning
    team: infra
  annotations:
    summary: "High CPU usage"
    description: "CPU usage is {{ $value }}% on {{ $labels.instance }}"

- alert: HighMemoryUsage
  expr: |
    (
      container_memory_working_set_bytes
      /
      container_spec_memory_limit_bytes
    ) * 100 > 90
  for: 5m
  labels:
    severity: critical
    team: infra
  annotations:
    summary: "High memory usage"
    description: "Memory usage is {{ $value }}% on {{ $labels.instance }}"

- alert: DiskSpaceLow
  expr: |
    (
      node_filesystem_avail_bytes
      /
      node_filesystem_size_bytes
    ) * 100 < 10
  for: 5m
  labels:
    severity: warning
    team: infra
  annotations:
    summary: "Low disk space"
    description: "Only {{ $value }}% disk space remaining"
```

**4. Service Availability Alerts**:
```yaml
- alert: ServiceDown
  expr: up{job="myapp"} == 0
  for: 1m
  labels:
    severity: critical
    team: backend
  annotations:
    summary: "Service is down"
    description: "{{ $labels.instance }} has been down for 1 minute"
    runbook_url: "https://wiki.company.com/runbooks/service-down"

- alert: TooManyRestarts
  expr: |
    rate(kube_pod_container_status_restarts_total[15m]) > 0
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Container restarting frequently"
    description: "{{ $labels.pod }} has restarted {{ $value }} times"
```

**5. SLO-Based Alerts**:
```yaml
# Error Budget Alert
- alert: ErrorBudgetBurning
  expr: |
    (
      1 - (
        sum(rate(http_requests_total{status=~"2.*|3.*"}[30d]))
        /
        sum(rate(http_requests_total[30d]))
      )
    ) > 0.001  # 99.9% SLO = 0.1% error budget
  for: 1h
  labels:
    severity: warning
  annotations:
    summary: "Error budget is burning"
    description: "Only {{ $value | humanizePercentage }} error budget remaining"
```

### Alert Severity Levels

```yaml
Severity Levels:
├── Critical (Page immediately)
│   - Service down
│   - Data loss
│   - Security breach
│   - SLO violation
│
├── Warning (Page during business hours)
│   - Performance degradation
│   - High resource usage
│   - Error rate elevated
│
└── Info (No page, log only)
    - Deployment started
    - Scaling event
    - Configuration change
```

### Alertmanager Configuration

`alertmanager.yml`:
```yaml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/XXX'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  
  routes:
    # Critical alerts - page immediately
    - match:
        severity: critical
      receiver: 'pagerduty'
      continue: true
    
    # Warning alerts - Slack only
    - match:
        severity: warning
      receiver: 'slack'
    
    # Team-specific routing
    - match:
        team: backend
      receiver: 'backend-team'

receivers:
  - name: 'default'
    slack_configs:
      - channel: '#alerts'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}\n{{ end }}'

  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'

  - name: 'slack'
    slack_configs:
      - channel: '#warnings'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}\n{{ end }}'

  - name: 'backend-team'
    slack_configs:
      - channel: '#backend-alerts'

inhibit_rules:
  # Don't alert on warning if critical alert is firing
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

### Testing Alerts

```bash
# 1. Check alert rules are valid
promtool check rules /etc/prometheus/alerts.yml

# 2. Test alert expression
curl 'http://prometheus:9090/api/v1/query?query=YOUR_ALERT_EXPRESSION'

# 3. Fire test alert
curl -X POST http://alertmanager:9093/api/v1/alerts \
  -H "Content-Type: application/json" \
  -d '[{
    "labels": {
      "alertname": "TestAlert",
      "severity": "warning"
    },
    "annotations": {
      "summary": "This is a test alert"
    }
  }]'

# 4. Check alert state
curl http://prometheus:9090/api/v1/alerts

# 5. Check Alertmanager status
curl http://alertmanager:9093/api/v1/status
```

### Runbook Example

```markdown
# Runbook: High Error Rate

## Severity
Critical

## Description
The application error rate has exceeded 5% for the last 5 minutes.

## Impact
Users are experiencing failures when using the service.

## Diagnosis Steps

1. Check the error dashboard:
   https://grafana.company.com/d/errors

2. View recent error logs:
   ```bash
   kubectl logs -l app=myapp --tail=100 | grep ERROR
   ```

3. Check recent deployments:
   ```bash
   kubectl get deployments -o wide
   ```

4. Review metrics:
   ```promql
   rate(http_requests_total{status=~"5.*"}[5m])
   ```

## Resolution Steps

1. If related to recent deployment:
   ```bash
   kubectl rollout undo deployment/myapp
   ```

2. If database connection issues:
   ```bash
   kubectl get pods -l app=database
   kubectl logs -l app=database
   ```

3. Check external dependencies:
   - API gateway status
   - Database connectivity
   - Cache availability

4. Scale up if resource constrained:
   ```bash
   kubectl scale deployment myapp --replicas=6
   ```

## Escalation
If unresolved after 15 minutes, page @backend-oncall

## Related Alerts
- HighLatency
- DatabaseConnectionIssues
- HighCPUUsage
```

---

## 12.7. Using Logs & Traces

### Observability Pillars

```
OBSERVABILITY
├── Metrics (What is broken?)
├── Logs (Why is it broken?)
└── Traces (Where is it broken?)
```

### Logging Basics

**Log Levels**:
```
DEBUG   - Detailed diagnostic info
INFO    - General informational messages
WARNING - Warning messages
ERROR   - Error messages
CRITICAL/FATAL - Critical errors
```

**Structured Logging** (JSON format):
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "user-service",
  "environment": "production",
  "message": "Failed to process user registration",
  "user_id": "12345",
  "error": "Database connection timeout",
  "trace_id": "abc-123-def",
  "span_id": "span-456"
}
```

### Distributed Tracing

**Why Tracing?**
- Understand request flow across services
- Identify performance bottlenecks
- Debug distributed systems
- Measure end-to-end latency

**Trace Structure**:
```
Trace (Complete request journey)
└── Span 1: API Gateway (100ms)
    ├── Span 2: User Service (50ms)
    │   └── Span 3: Database Query (30ms)
    └── Span 4: Auth Service (40ms)
        └── Span 5: Cache Lookup (10ms)
```

### Correlation IDs

**Add correlation ID to track requests**:

```python
import uuid
from flask import Flask, request, g

app = Flask(__name__)

@app.before_request
def before_request():
    # Get or generate correlation ID
    g.correlation_id = request.headers.get('X-Correlation-ID', str(uuid.uuid4()))

@app.after_request
def after_request(response):
    # Add to response headers
    response.headers['X-Correlation-ID'] = g.correlation_id
    return response

def log_info(message):
    print(json.dumps({
        'level': 'INFO',
        'message': message,
        'correlation_id': g.correlation_id,
        'timestamp': datetime.utcnow().isoformat()
    }))
```

### Log Aggregation with ELK Stack

**Architecture**:
```
Application Logs
       ↓
   Filebeat (Shipper)
       ↓
   Logstash (Processing)
       ↓
  Elasticsearch (Storage)
       ↓
    Kibana (Visualization)
```

**Docker Compose for ELK**:
```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    ports:
      - "5000:5000"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data:
```

`logstash.conf`:
```
input {
  tcp {
    port => 5000
    codec => json_lines
  }
}

filter {
  # Parse timestamp
  date {
    match => [ "timestamp", "ISO8601" ]
  }
  
  # Add geo-location for IPs
  geoip {
    source => "client_ip"
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
  }
}
```

### OpenTelemetry Tracing

**Python Example**:
```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from flask import Flask

# Setup tracing
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

# Setup Jaeger exporter
jaeger_exporter = JaegerExporter(
    agent_host_name="jaeger",
    agent_port=6831,
)
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)

app = Flask(__name__)

@app.route('/api/users')
def get_users():
    # Create span
    with tracer.start_as_current_span("get_users") as span:
        span.set_attribute("user.count", 10)
        
        # Simulate database call
        with tracer.start_as_current_span("database_query"):
            users = fetch_from_database()
        
        # Simulate external API call
        with tracer.start_as_current_span("external_api_call"):
            enriched_data = call_external_api()
        
        return {"users": users}
```

**Jaeger Setup**:
```bash
# Run Jaeger All-in-One
docker run -d \
  --name jaeger \
  -p 6831:6831/udp \
  -p 16686:16686 \
  jaegertracing/all-in-one:latest

# Access Jaeger UI
open http://localhost:16686
```

---

## 12.8. Useful Info in Log Entries

### Essential Log Fields

```json
{
  // When
  "timestamp": "2024-01-15T10:30:00.123Z",
  "date_time": "2024-01-15 10:30:00",
  
  // What
  "level": "ERROR",
  "message": "Failed to process payment",
  "event_type": "payment_failure",
  
  // Where
  "service": "payment-service",
  "host": "pod-abc-123",
  "environment": "production",
  "region": "us-east-1",
  "version": "1.2.3",
  
  // Who
  "user_id": "user-12345",
  "session_id": "sess-67890",
  "ip_address": "192.168.1.100",
  
  // Why
  "error_code": "INSUFFICIENT_FUNDS",
  "error_message": "Account balance too low",
  "stack_trace": "...",
  
  // Context
  "correlation_id": "req-abc-def-123",
  "trace_id": "trace-xyz-789",
  "span_id": "span-456",
  "parent_span_id": "span-123",
  
  // Additional Context
  "transaction_id": "txn-999",
  "amount": 99.99,
  "currency": "USD",
  "payment_method": "credit_card",
  
  // Performance
  "duration_ms": 1250,
  "response_code": 500
}
```

### Log Examples by Type

**1. Request Logs**:
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "type": "http_request",
  "method": "POST",
  "path": "/api/orders",
  "status_code": 201,
  "duration_ms": 145,
  "user_id": "user-123",
  "correlation_id": "req-abc-123",
  "user_agent": "Mozilla/5.0...",
  "ip": "192.168.1.100"
}
```

**2. Error Logs**:
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "type": "application_error",
  "message": "Database connection failed",
  "error_code": "DB_CONNECTION_ERROR",
  "error_type": "DatabaseError",
  "stack_trace": "Traceback...",
  "correlation_id": "req-abc-123",
  "retry_count": 3,
  "database": "users_db"
}
```

**3. Business Event Logs**:
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "type": "business_event",
  "event": "order_placed",
  "order_id": "ord-789",
  "user_id": "user-123",
  "total_amount": 99.99,
  "items_count": 3,
  "payment_method": "credit_card"
}
```

**4. Performance Logs**:
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "type": "performance",
  "operation": "database_query",
  "query_type": "SELECT",
  "table": "users",
  "duration_ms": 250,
  "rows_returned": 100,
  "cache_hit": false
}
```

**5. Security Logs**:
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "WARNING",
  "type": "security_event",
  "event": "failed_login_attempt",
  "user_id": "user-123",
  "ip": "192.168.1.100",
  "reason": "invalid_password",
  "attempt_count": 3
}
```

---

## 12.9. Tools for Logging

### 1. ELK Stack (Elasticsearch, Logstash, Kibana)

**Use Case**: General-purpose log aggregation and analysis

**Setup**:
```bash
# Using Docker Compose (shown in section 12.7)
docker-compose up -d

# Ship logs to Logstash
echo '{"level":"INFO","message":"Test log"}' | \
  nc localhost 5000
```

**Kibana Queries**:
```
# Search for errors
level: "ERROR"

# Time range and service
level: "ERROR" AND service: "user-service" AND @timestamp >= now-1h

# Complex query
(level: "ERROR" OR level: "WARNING") AND 
service: "payment-service" AND 
correlation_id: "req-abc-123"
```

### 2. Grafana Loki

**Use Case**: Kubernetes-native logging, lower cost than ELK

**Architecture**:
```
Application → Promtail → Loki → Grafana
```

**Docker Compose**:
```yaml
version: '3.8'

services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log
      - ./promtail-config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
```

`promtail-config.yml`:
```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

**LogQL Queries** (Loki's query language):
```logql
# Show all logs from a service
{service="user-service"}

# Filter by level
{service="user-service"} |= "ERROR"

# Regex filter
{service="user-service"} |~ "error|failed"

# Rate of errors
rate({service="user-service"} |= "ERROR" [5m])

# Count logs by level
sum by (level) (count_over_time({service="user-service"}[5m]))
```

### 3. Fluentd

**Use Case**: Unified logging layer, pluggable architecture

**Docker Setup**:
```bash
docker run -d \
  --name fluentd \
  -p 24224:24224 \
  -v $(pwd)/fluent.conf:/fluentd/etc/fluent.conf \
  fluent/fluentd:latest
```

`fluent.conf`:
```conf
<source>
  @type forward
  port 24224
</source>

<filter **>
  @type record_transformer
  <record>
    hostname ${hostname}
    environment production
  </record>
</filter>

<match **>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  logstash_prefix fluentd
  flush_interval 10s
</match>
```

**Application Integration**:
```python
from fluent import sender
from fluent import event

# Setup Fluentd logger
logger = sender.FluentSender('app', host='localhost', port=24224)

# Send log
logger.emit('follow', {
    'level': 'INFO',
    'message': 'User logged in',
    'user_id': '12345'
})
```

### 4. Cloud-Native Solutions

**AWS CloudWatch**:
```bash
# Install CloudWatch agent
docker run -d \
  --name cloudwatch-agent \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  amazon/cloudwatch-agent:latest

# Query logs
aws logs tail /aws/application/myapp --follow
```

**Google Cloud Logging**:
```python
from google.cloud import logging

client = logging.Client()
logger = client.logger('my-application')

logger.log_struct({
    'level': 'INFO',
    'message': 'Application started',
    'version': '1.0.0'
})
```

**Azure Monitor**:
```bash
# Query logs
az monitor log-analytics query \
  --workspace <workspace-id> \
  --analytics-query "traces | where severityLevel == 3"
```

### 5. Application Performance Monitoring (APM)

**Datadog**:
```python
from ddtrace import tracer

@tracer.wrap()
def process_order(order_id):
    # Your code here
    pass
```

**New Relic**:
```python
import newrelic.agent

newrelic.agent.initialize('newrelic.ini')

@newrelic.agent.background_task()
def background_job():
    # Your code here
    pass
```

**Dynatrace, AppDynamics, Splunk**: Similar APM solutions

---

## 12.10. Logging the Right Information

### What to Log

**✅ DO Log**:
1. **Request/Response Details**:
   - HTTP method, path, status code
   - Request/response duration
   - User ID, session ID, correlation ID

2. **Business Events**:
   - User registration/login
   - Order placement
   - Payment processing
   - Important state changes

3. **Errors and Exceptions**:
   - Stack traces
   - Error context
   - Retry attempts

4. **Performance Metrics**:
   - Database query duration
   - External API call times
   - Cache hit/miss

5. **Security Events**:
   - Authentication attempts
   - Authorization failures
   - Suspicious activities

**❌ DON'T Log**:
1. **Sensitive Data**:
   - Passwords
   - Credit card numbers
   - Social security numbers
   - API keys/secrets
   - Personal health information (PHI)

2. **High-Volume Low-Value Data**:
   - Health check requests (unless they fail)
   - Excessive debug logs in production
   - Internal polling requests

### Sanitizing Sensitive Data

```python
import re
import json

def sanitize_log_data(data):
    """Remove sensitive information from logs"""
    sensitive_fields = ['password', 'credit_card', 'ssn', 'api_key']
    
    if isinstance(data, dict):
        return {
            k: '***REDACTED***' if k.lower() in sensitive_fields else sanitize_log_data(v)
            for k, v in data.items()
        }
    elif isinstance(data, list):
        return [sanitize_log_data(item) for item in data]
    elif isinstance(data, str):
        # Redact credit card numbers
        data = re.sub(r'\b\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}\b', '****-****-****-****', data)
        # Redact email addresses (partially)
        data = re.sub(r'([a-zA-Z0-9._-]+)@', r'\1***@', data)
    return data

# Usage
log_data = {
    'user': 'john@example.com',
    'action': 'login',
    'password': 'secret123',
    'ip': '192.168.1.1'
}

sanitized = sanitize_log_data(log_data)
logger.info(json.dumps(sanitized))
```

### Log Levels Best Practices

```python
import logging

# DEBUG: Detailed diagnostic information
logging.debug(f"Processing item {item_id} with params {params}")

# INFO: General informational messages
logging.info(f"User {user_id} logged in successfully")

# WARNING: Something unexpected but not an error
logging.warning(f"API rate limit approaching: {current_rate}/{limit}")

# ERROR: Error occurred but application continues
logging.error(f"Failed to process payment for order {order_id}", exc_info=True)

# CRITICAL: Serious error, application may not continue
logging.critical(f"Database connection pool exhausted")
```

### Structured Logging Example

```python
import json
import logging
from datetime import datetime

class StructuredLogger:
    def __init__(self, service_name, environment):
        self.service_name = service_name
        self.environment = environment
        self.logger = logging.getLogger(service_name)
    
    def _format_log(self, level, message, **kwargs):
        log_entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': level,
            'service': self.service_name,
            'environment': self.environment,
            'message': message,
            **kwargs
        }
        return json.dumps(log_entry)
    
    def info(self, message, **kwargs):
        self.logger.info(self._format_log('INFO', message, **kwargs))
    
    def error(self, message, **kwargs):
        self.logger.error(self._format_log('ERROR', message, **kwargs))
    
    def warning(self, message, **kwargs):
        self.logger.warning(self._format_log('WARNING', message, **kwargs))

# Usage
logger = StructuredLogger('user-service', 'production')

logger.info(
    'User registered',
    user_id='12345',
    email='user@example.com',
    registration_type='oauth',
    correlation_id='req-abc-123'
)

logger.error(
    'Payment processing failed',
    order_id='ord-789',
    amount=99.99,
    error_code='INSUFFICIENT_FUNDS',
    correlation_id='req-abc-123'
)
```

### Sampling for High-Volume Logs

```python
import random

class SampledLogger:
    def __init__(self, logger, sample_rate=0.1):
        """
        sample_rate: Fraction of logs to keep (0.1 = 10%)
        """
        self.logger = logger
        self.sample_rate = sample_rate
    
    def info(self, message, always_log=False, **kwargs):
        if always_log or random.random() < self.sample_rate:
            self.logger.info(message, **kwargs)
    
    def error(self, message, **kwargs):
        # Always log errors
        self.logger.error(message, **kwargs)

# Usage
logger = SampledLogger(logging.getLogger(), sample_rate=0.1)

# Only 10% of these will be logged
for i in range(1000):
    logger.info(f"Processing item {i}")

# This will always be logged
logger.info("Important business event", always_log=True)

# Errors always logged
logger.error("Critical error occurred")
```

---

## 12.11. Tracing Interaction Between Services

### Distributed Tracing Concepts

**Trace**: Complete journey of a request through all services
**Span**: Single operation within a trace
**Context Propagation**: Passing trace information between services

```
Client Request
     │
     ▼
┌────────────────────────────────────────┐
│ Trace ID: trace-abc-123                │
│                                        │
│  Span 1: API Gateway (Total: 500ms)   │
│    │                                   │
│    ├─► Span 2: Auth Service (50ms)    │
│    │                                   │
│    ├─► Span 3: User Service (200ms)   │
│    │     │                             │
│    │     └─► Span 4: Database (150ms) │
│    │                                   │
│    └─► Span 5: Notification (100ms)   │
│                                        │
└────────────────────────────────────────┘
```

### OpenTelemetry Implementation

**1. Setup (Python)**:
```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from flask import Flask

# Initialize tracing
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

# Setup Jaeger exporter
jaeger_exporter = JaegerExporter(
    agent_host_name="jaeger",
    agent_port=6831,
)

trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)

# Create Flask app
app = Flask(__name__)

# Auto-instrument Flask
FlaskInstrumentor().instrument_app(app)

# Auto-instrument requests library
RequestsInstrumentor().instrument()
```

**2. Manual Span Creation**:
```python
import requests
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

@app.route('/api/users/<user_id>')
def get_user_profile(user_id):
    # Get current span
    current_span = trace.get_current_span()
    current_span.set_attribute("user.id", user_id)
    
    # Create child span for database query
    with tracer.start_as_current_span("database.get_user") as db_span:
        db_span.set_attribute("db.system", "postgresql")
        db_span.set_attribute("db.operation", "SELECT")
        user = db.query(f"SELECT * FROM users WHERE id = {user_id}")
        db_span.set_attribute("db.rows_returned", 1)
    
    # Create child span for external API
    with tracer.start_as_current_span("external_api.get_preferences") as api_span:
        api_span.set_attribute("http.method", "GET")
        api_span.set_attribute("http.url", "https://api.example.com/preferences")
        
        response = requests.get(f"https://api.example.com/preferences/{user_id}")
        
        api_span.set_attribute("http.status_code", response.status_code)
    
    # Add event to span
    current_span.add_event(
        "user_profile_retrieved",
        attributes={"user.name": user['name']}
    )
    
    return {"user": user, "preferences": response.json()}
```

**3. Error Tracking in Spans**:
```python
from opentelemetry.trace import Status, StatusCode

@app.route('/api/orders')
def create_order():
    span = trace.get_current_span()
    
    try:
        order = process_order()
        span.set_attribute("order.id", order['id'])
        span.set_status(Status(StatusCode.OK))
        return order
    except Exception as e:
        # Record exception in span
        span.record_exception(e)
        span.set_status(
            Status(StatusCode.ERROR, str(e))
        )
        raise
```

**4. Context Propagation Between Services**:

Service A (Python):
```python
import requests
from opentelemetry.propagate import inject

def call_service_b():
    headers = {}
    # Inject trace context into headers
    inject(headers)
    
    response = requests.get(
        'http://service-b/api/data',
        headers=headers
    )
    return response.json()
```

Service B (Node.js):
```javascript
const opentelemetry = require('@opentelemetry/api');
const { context, trace, propagation } = opentelemetry;

app.get('/api/data', (req, res) => {
  // Extract trace context from headers
  const extractedContext = propagation.extract(
    context.active(),
    req.headers
  );
  
  // Create span in extracted context
  const span = trace.getTracer('service-b').startSpan(
    'process_data',
    undefined,
    extractedContext
  );
  
  // Process request
  const data = processData();
  
  span.end();
  res.json(data);
});
```

### Jaeger Setup

```yaml
version: '3.8'

services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"  # UI
      - "14268:14268"
      - "14250:14250"
      - "9411:9411"
    environment:
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411
```

```bash
# Start Jaeger
docker-compose up -d jaeger

# Access Jaeger UI
open http://localhost:16686
```

### Trace Sampling

```python
from opentelemetry.sdk.trace.sampling import (
    TraceIdRatioBased,
    ParentBased,
)

# Sample 10% of traces
sampler = ParentBased(
    root=TraceIdRatioBased(0.1)
)

trace.set_tracer_provider(
    TracerProvider(sampler=sampler)
)
```

---

## 12.12. Visualizing Traces

### Jaeger UI Features

**1. Search Traces**:
```
Service: user-service
Operation: GET /api/users
Tags: http.status_code=500
Min Duration: 1s
Limit: 20 results
```

**2. Trace Timeline View**:
```
Trace: abc-123-def (Total: 523ms)
├─ API Gateway [0ms - 523ms]
│  ├─ Auth Check [5ms - 55ms]
│  ├─ User Service [60ms - 380ms]
│  │  ├─ DB Query [70ms - 220ms]  ◄─ SLOW
│  │  └─ Cache Update [225ms - 235ms]
│  └─ Logging [385ms - 395ms]
```

**3. Span Details**:
```
Span: database.get_user
Duration: 150ms
Tags:
  - db.system: postgresql
  - db.operation: SELECT
  - db.statement: SELECT * FROM users WHERE id = ?
  - db.rows_returned: 1

Events:
  - query_started (0ms)
  - connection_acquired (5ms)
  - query_completed (150ms)

Logs:
  - Slow query detected
  - Query plan: Index Scan on users_pkey
```

### Grafana with Tempo

**Setup**:
```yaml
version: '3.8'

services:
  tempo:
    image: grafana/tempo:latest
    command: [ "-config.file=/etc/tempo.yaml" ]
    volumes:
      - ./tempo.yaml:/etc/tempo.yaml
      - tempo-data:/var/tempo
    ports:
      - "3200:3200"   # tempo
      - "4317:4317"   # otlp grpc
      - "4318:4318"   # otlp http

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    volumes:
      - ./grafana-datasources.yaml:/etc/grafana/provisioning/datasources/datasources.yaml

volumes:
  tempo-data:
```

`tempo.yaml`:
```yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        http:
        grpc:

storage:
  trace:
    backend: local
    local:
      path: /var/tempo/traces
```

`grafana-datasources.yaml`:
```yaml
apiVersion: 1

datasources:
  - name: Tempo
    type: tempo
    access: proxy
    url: http://tempo:3200
```

### Trace Metrics

**Derive RED metrics from traces**:
```promql
# Request Rate
sum(rate(traces_spanmetrics_calls_total[5m])) by (service)

# Error Rate
sum(rate(traces_spanmetrics_calls_total{status_code="STATUS_CODE_ERROR"}[5m]))
/
sum(rate(traces_spanmetrics_calls_total[5m]))

# Duration (p95)
histogram_quantile(0.95,
  sum(rate(traces_spanmetrics_latency_bucket[5m])) by (le, service)
)
```

### Service Dependency Graph

Jaeger and Zipkin auto-generate service dependency graphs:

```
┌──────────────┐
│   Frontend   │
└───────┬──────┘
        │
        ├─────────► Auth Service
        │
        ├─────────► User Service ──────► Database
        │
        └─────────► Order Service ──┬──► Database
                                     │
                                     └──► Payment Gateway
```

### Alerting on Traces

```yaml
# Alert on high p99 latency for specific operation
- alert: HighTraceLatency
  expr: |
    histogram_quantile(0.99,
      sum(rate(traces_spanmetrics_latency_bucket{
        service="user-service",
        operation="GET /api/users"
      }[5m])) by (le)
    ) > 1
  for: 5m
  annotations:
    summary: "High trace latency detected"

# Alert on high error rate in traces
- alert: HighTraceErrorRate
  expr: |
    sum(rate(traces_spanmetrics_calls_total{
      status_code="STATUS_CODE_ERROR"
    }[5m]))
    /
    sum(rate(traces_spanmetrics_calls_total[5m]))
    > 0.05
  for: 5m
```

### Best Practices for Tracing

1. **Meaningful Span Names**:
   ```python
   # Good
   with tracer.start_as_current_span("database.get_user")
   with tracer.start_as_current_span("external_api.payment_gateway")
   
   # Bad
   with tracer.start_as_current_span("function1")
   with tracer.start_as_current_span("query")
   ```

2. **Add Relevant Attributes**:
   ```python
   span.set_attribute("user.id", user_id)
   span.set_attribute("order.total", order_total)
   span.set_attribute("http.method", "POST")
   span.set_attribute("db.rows_affected", rows_affected)
   ```

3. **Use Span Events for Important Milestones**:
   ```python
   span.add_event("cache_miss")
   span.add_event("retry_attempt", attributes={"attempt": 2})
   span.add_event("circuit_breaker_opened")
   ```

4. **Record Exceptions**:
   ```python
   try:
       result = risky_operation()
   except Exception as e:
       span.record_exception(e)
       span.set_status(Status(StatusCode.ERROR))
       raise
   ```

5. **Sample Appropriately**:
   - Production: 1-10% sampling
   - Staging: 50% sampling
   - Development: 100% sampling

---

## Summary

### Key Takeaways

1. **Monitor the Four Golden Signals**: Latency, Traffic, Errors, Saturation
2. **Use appropriate metric types**: Counters, Gauges, Histograms, Summaries
3. **Set actionable alerts** with clear runbooks
4. **Implement structured logging** with correlation IDs
5. **Use distributed tracing** to understand service interactions
6. **Visualize everything** with dashboards and trace viewers

### Complete Monitoring Stack

```
┌──────────────────────────────────────────────┐
│           Application Layer                   │
│  (Instrumented with OpenTelemetry)           │
└──────────┬───────────────┬───────────────────┘
           │               │
           │               │
    ┌──────▼──────┐   ┌───▼────────┐
    │  Metrics    │   │   Traces   │
    │ (Prometheus)│   │  (Jaeger)  │
    └──────┬──────┘   └───┬────────┘
           │               │
           └───────┬───────┘
                   │
              ┌────▼────┐       ┌──────────┐
              │ Grafana │◄──────┤   Loki   │
              │(Visualize)     (Logs)     │
              └─────────┘       └──────────┘
                   │
              ┌────▼────────┐
              │AlertManager │
              └─────────────┘
```

### Practice Exercises

1. Set up Prometheus + Grafana monitoring stack
2. Instrument a sample application
3. Create dashboards for Golden Signals
4. Configure meaningful alerts
5. Set up distributed tracing with Jaeger
6. Implement structured logging
7. Create service dependency graphs
8. Build runbooks for common issues

---

## Next Chapter

Continue to **Chapter 13: Kubernetes** to learn about container orchestration and deploying monitored applications to production.
