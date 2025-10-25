# Logging Best Practices - Examples

---

## Example 1: Complete Express App with Logging

```javascript
const express = require('express');
const winston = require('winston');
const { v4: uuidv4 } = require('uuid');

// Configure logger
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'api-server' },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

const app = express();

// Correlation ID middleware
app.use((req, res, next) => {
  req.correlationId = req.headers['x-correlation-id'] || uuidv4();
  res.setHeader('x-correlation-id', req.correlationId);

  req.logger = logger.child({
    correlationId: req.correlationId,
    method: req.method,
    path: req.path
  });

  const startTime = Date.now();

  res.on('finish', () => {
    req.logger.info('Request completed', {
      statusCode: res.statusCode,
      durationMs: Date.now() - startTime
    });
  });

  next();
});

// Routes
app.get('/users/:id', async (req, res) => {
  req.logger.info('Fetching user', { userId: req.params.id });

  try {
    const user = await getUserById(req.params.id);
    res.json(user);
  } catch (error) {
    req.logger.error('Failed to fetch user', {
      userId: req.params.id,
      error: error.message,
      stack: error.stack
    });
    res.status(500).json({ error: 'Internal server error' });
  }
});

app.listen(3000);
```

---

## Example 2: Python FastAPI with Structured Logging

```python
import logging
import json
import time
from datetime import datetime
from fastapi import FastAPI, Request
from contextvars import ContextVar
import uuid

# Context variable for correlation ID
correlation_id_var: ContextVar[str] = ContextVar('correlation_id')

# Structured JSON formatter
class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_obj = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'correlation_id': correlation_id_var.get(None),
        }

        if hasattr(record, 'extra'):
            log_obj.update(record.extra)

        if record.exc_info:
            log_obj['exception'] = self.formatException(record.exc_info)

        return json.dumps(log_obj)

# Configure logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)

app = FastAPI()

@app.middleware("http")
async def logging_middleware(request: Request, call_next):
    # Generate correlation ID
    correlation_id = request.headers.get('x-correlation-id', str(uuid.uuid4()))
    correlation_id_var.set(correlation_id)

    start_time = time.time()

    logger.info("Request started", extra={
        'method': request.method,
        'path': request.url.path,
        'client_ip': request.client.host
    })

    response = await call_next(request)
    duration = time.time() - start_time

    logger.info("Request completed", extra={
        'method': request.method,
        'path': request.url.path,
        'status_code': response.status_code,
        'duration_sec': f"{duration:.3f}"
    })

    response.headers['x-correlation-id'] = correlation_id
    return response

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    logger.info(f"Fetching user", extra={'user_id': user_id})
    # ... implementation ...
    return {"id": user_id, "name": "John Doe"}
```

---

## Example 3: Database Query Logging

```javascript
// Sequelize query logging
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize({
  // ... config ...
  logging: (sql, timing) => {
    logger.debug('Database query', {
      sql: sql,
      durationMs: timing,
      correlationId: getCurrentCorrelationId()
    });
  },
  benchmark: true
});

// Slow query detection
const originalQuery = sequelize.query.bind(sequelize);
sequelize.query = function(...args) {
  const startTime = Date.now();

  return originalQuery(...args).then((result) => {
    const duration = Date.now() - startTime;

    if (duration > 1000) {
      logger.warn('Slow query detected', {
        query: args[0],
        durationMs: duration,
        threshold: 1000
      });
    }

    return result;
  });
};
```

---

## Example 4: Error Context Logging

```javascript
class AppError extends Error {
  constructor(message, context = {}) {
    super(message);
    this.name = this.constructor.name;
    this.context = context;
    Error.captureStackTrace(this, this.constructor);
  }

  log(logger) {
    logger.error(this.message, {
      errorName: this.name,
      context: this.context,
      stack: this.stack
    });
  }
}

// Usage
try {
  const user = await fetchUser(userId);
} catch (error) {
  const appError = new AppError('Failed to fetch user', {
    userId,
    service: 'user-service',
    timestamp: new Date().toISOString()
  });

  appError.log(logger);
  throw appError;
}
```

---

**Versión:** 1.0.0
