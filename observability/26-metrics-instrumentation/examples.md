# Metrics & Instrumentation - Examples

---

## Example 1: Complete Microservice with Prometheus

```javascript
// server.js
const express = require('express');
const client = require('prom-client');

const app = express();
const register = new client.Registry();

// Default metrics
client.collectDefaultMetrics({
  register,
  prefix: 'nodejs_',
  gcDurationBuckets: [0.001, 0.01, 0.1, 1, 2, 5],
});

// HTTP metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.001, 0.01, 0.1, 0.5, 1, 2, 5],
  registers: [register],
});

const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register],
});

const activeRequests = new client.Gauge({
  name: 'http_active_requests',
  help: 'Number of requests currently being processed',
  registers: [register],
});

// Database metrics
const dbQueryDuration = new client.Histogram({
  name: 'db_query_duration_seconds',
  help: 'Database query duration',
  labelNames: ['operation', 'table'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 2],
  registers: [register],
});

const dbConnectionPoolSize = new client.Gauge({
  name: 'db_connection_pool_size',
  help: 'Database connection pool size',
  labelNames: ['state'],
  registers: [register],
});

// Business metrics
const ordersCreated = new client.Counter({
  name: 'orders_created_total',
  help: 'Total orders created',
  labelNames: ['status', 'payment_method'],
  registers: [register],
});

const revenue = new client.Counter({
  name: 'revenue_total_cents',
  help: 'Total revenue in cents',
  labelNames: ['currency'],
  registers: [register],
});

// Middleware
app.use((req, res, next) => {
  const start = process.hrtime.bigint();
  activeRequests.inc();

  res.on('finish', () => {
    const duration = Number(process.hrtime.bigint() - start) / 1e9;
    const route = req.route?.path || req.path;

    httpRequestDuration
      .labels(req.method, route, res.statusCode)
      .observe(duration);

    httpRequestsTotal
      .labels(req.method, route, res.statusCode)
      .inc();

    activeRequests.dec();
  });

  next();
});

app.use(express.json());

// Routes
app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.post('/orders', async (req, res) => {
  const start = Date.now();

  try {
    const order = await db.orders.create(req.body);

    dbQueryDuration
      .labels('INSERT', 'orders')
      .observe((Date.now() - start) / 1000);

    ordersCreated
      .labels('created', order.paymentMethod)
      .inc();

    revenue
      .labels(order.currency)
      .inc(order.amountCents);

    res.status(201).json(order);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Update pool metrics every 10 seconds
setInterval(() => {
  const pool = db.connectionManager.pool;

  dbConnectionPoolSize.labels('idle').set(pool._availableObjects.length);
  dbConnectionPoolSize.labels('active').set(pool._inUseObjects.length);
}, 10000);

app.listen(3000);
```

---

## Example 2: Grafana Dashboard JSON

```json
{
  "dashboard": {
    "title": "Application Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{route}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Request Duration (p95)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "{{method}} {{route}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status_code=~\"5..\"}[5m])",
            "legendFormat": "{{route}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Active Requests",
        "targets": [
          {
            "expr": "http_active_requests",
            "legendFormat": "Active"
          }
        ],
        "type": "graph"
      }
    ]
  }
}
```

---

## Example 3: Python FastAPI with Prometheus

```python
from fastapi import FastAPI, Request
from prometheus_client import Counter, Histogram, Gauge, generate_latest, REGISTRY
from prometheus_client import CollectorRegistry
import time

app = FastAPI()

# Metrics
http_requests_total = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status_code']
)

http_request_duration_seconds = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint'],
    buckets=[0.01, 0.05, 0.1, 0.5, 1.0, 2.0, 5.0, 10.0]
)

http_requests_in_progress = Gauge(
    'http_requests_in_progress',
    'HTTP requests in progress'
)

# Middleware
@app.middleware("http")
async def metrics_middleware(request: Request, call_next):
    http_requests_in_progress.inc()
    start_time = time.time()

    response = await call_next(request)

    duration = time.time() - start_time
    endpoint = request.url.path

    http_requests_total.labels(
        method=request.method,
        endpoint=endpoint,
        status_code=response.status_code
    ).inc()

    http_request_duration_seconds.labels(
        method=request.method,
        endpoint=endpoint
    ).observe(duration)

    http_requests_in_progress.dec()

    return response

# Metrics endpoint
@app.get("/metrics")
async def metrics():
    from starlette.responses import Response
    return Response(generate_latest(), media_type="text/plain")

# Business logic with metrics
orders_total = Counter(
    'orders_total',
    'Total number of orders',
    ['status']
)

@app.post("/orders")
async def create_order(order_data: dict):
    # ... create order ...

    orders_total.labels(status='created').inc()

    return {"id": 123, "status": "created"}
```

---

## Example 4: StatsD Integration

```javascript
// statsd-metrics.js
const StatsD = require('node-statsd');

class MetricsClient {
  constructor(options = {}) {
    this.client = new StatsD({
      host: options.host || 'localhost',
      port: options.port || 8125,
      prefix: options.prefix || 'myapp.',
      cacheDns: true,
    });
  }

  timing(metric, value, tags = []) {
    this.client.timing(metric, value, tags);
  }

  increment(metric, value = 1, tags = []) {
    this.client.increment(metric, value, tags);
  }

  gauge(metric, value, tags = []) {
    this.client.gauge(metric, value, tags);
  }

  histogram(metric, value, tags = []) {
    this.client.histogram(metric, value, tags);
  }

  set(metric, value, tags = []) {
    this.client.set(metric, value, tags);
  }

  close() {
    this.client.close();
  }
}

const metrics = new MetricsClient({ prefix: 'api.' });

// Express middleware
function statsMiddleware(req, res, next) {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    const route = req.route?.path || 'unknown';

    const tags = [
      `method:${req.method}`,
      `route:${route}`,
      `status:${res.statusCode}`,
    ];

    metrics.timing('http.request.duration', duration, tags);
    metrics.increment('http.requests', 1, tags);

    if (res.statusCode >= 400) {
      metrics.increment('http.errors', 1, tags);
    }
  });

  next();
}

app.use(statsMiddleware);

// Business metrics
function trackOrder(order) {
  metrics.increment('orders.created', 1, [
    `payment_method:${order.paymentMethod}`,
    `status:${order.status}`,
  ]);

  metrics.gauge('orders.value', order.amount, [
    `currency:${order.currency}`,
  ]);
}

// Usage
app.post('/orders', async (req, res) => {
  const order = await createOrder(req.body);
  trackOrder(order);
  res.json(order);
});

module.exports = metrics;
```

---

## Example 5: Custom Metrics Dashboard

```javascript
// metrics-dashboard.js
class MetricsDashboard {
  constructor(register) {
    this.register = register;
  }

  async getMetrics() {
    const metrics = await this.register.getMetricsAsJSON();

    return metrics.map(metric => ({
      name: metric.name,
      type: metric.type,
      help: metric.help,
      values: metric.values,
    }));
  }

  async getMetricsSummary() {
    const raw = await this.register.metrics();
    const lines = raw.split('\n');

    const summary = {};

    for (const line of lines) {
      if (line.startsWith('#') || !line.trim()) continue;

      const [metricPart, value] = line.split(' ');
      const metricName = metricPart.split('{')[0];

      if (!summary[metricName]) {
        summary[metricName] = {
          values: [],
          count: 0,
        };
      }

      summary[metricName].values.push(parseFloat(value));
      summary[metricName].count++;
    }

    return summary;
  }

  async getHealthStatus() {
    const metrics = await this.getMetricsSummary();

    const health = {
      status: 'healthy',
      checks: [],
    };

    // Check error rate
    const httpRequests = metrics.http_requests_total?.count || 0;
    const httpErrors = metrics.http_errors_total?.count || 0;
    const errorRate = httpRequests > 0 ? (httpErrors / httpRequests) * 100 : 0;

    health.checks.push({
      name: 'error_rate',
      status: errorRate < 5 ? 'pass' : 'fail',
      value: errorRate.toFixed(2) + '%',
    });

    // Check active connections
    const activeConns = metrics.active_connections?.values[0] || 0;

    health.checks.push({
      name: 'active_connections',
      status: activeConns < 1000 ? 'pass' : 'warn',
      value: activeConns,
    });

    if (health.checks.some(c => c.status === 'fail')) {
      health.status = 'unhealthy';
    } else if (health.checks.some(c => c.status === 'warn')) {
      health.status = 'degraded';
    }

    return health;
  }
}

// Usage
const dashboard = new MetricsDashboard(register);

app.get('/metrics/summary', async (req, res) => {
  const summary = await dashboard.getMetricsSummary();
  res.json(summary);
});

app.get('/health/detailed', async (req, res) => {
  const health = await dashboard.getHealthStatus();
  res.json(health);
});
```

---

**Versión:** 1.0.0
