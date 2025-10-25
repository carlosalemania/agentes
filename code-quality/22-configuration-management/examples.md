# Configuration Management - Examples

---

## Example 1: Complete .env Setup

```bash
# .env.example (commit this to git)
# Server
NODE_ENV=development
PORT=3000
HOST=0.0.0.0

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/myapp_dev
DATABASE_POOL_SIZE=10

# Redis
REDIS_URL=redis://localhost:6379

# Auth (generate with: openssl rand -base64 32)
JWT_SECRET=your-secret-key-here-min-32-chars
JWT_EXPIRES_IN=1d

# External APIs
API_BASE_URL=https://api.example.com
API_KEY=your-api-key
API_TIMEOUT_MS=10000

# Feature Flags
ENABLE_CACHING=true
ENABLE_RATE_LIMITING=true
ENABLE_ANALYTICS=false

# Logging
LOG_LEVEL=debug
SENTRY_DSN=

# .gitignore
.env
.env.local
.env.*.local
```

---

## Example 2: Complete TypeScript Configuration

```typescript
// config.ts
import { z } from 'zod';
import dotenv from 'dotenv';

dotenv.config();

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'staging', 'production']),
  PORT: z.coerce.number().int().positive(),
  HOST: z.string(),

  DATABASE_URL: z.string().url(),
  DATABASE_POOL_SIZE: z.coerce.number().int().min(1).max(100),

  REDIS_URL: z.string().url(),

  JWT_SECRET: z.string().min(32),
  JWT_EXPIRES_IN: z.string(),

  API_BASE_URL: z.string().url(),
  API_KEY: z.string().min(1),

  ENABLE_CACHING: z.coerce.boolean(),
  ENABLE_RATE_LIMITING: z.coerce.boolean(),

  LOG_LEVEL: z.enum(['error', 'warn', 'info', 'debug', 'trace']),
  SENTRY_DSN: z.string().url().optional(),
});

export type Env = z.infer<typeof envSchema>;

class Config {
  private env: Env;

  constructor() {
    const result = envSchema.safeParse(process.env);

    if (!result.success) {
      console.error('❌ Invalid environment variables:');
      console.error(result.error.format());
      process.exit(1);
    }

    this.env = result.data;
  }

  get isDevelopment() {
    return this.env.NODE_ENV === 'development';
  }

  get isProduction() {
    return this.env.NODE_ENV === 'production';
  }

  get server() {
    return {
      port: this.env.PORT,
      host: this.env.HOST,
    };
  }

  get database() {
    return {
      url: this.env.DATABASE_URL,
      poolSize: this.env.DATABASE_POOL_SIZE,
    };
  }

  get redis() {
    return {
      url: this.env.REDIS_URL,
    };
  }

  get auth() {
    return {
      jwtSecret: this.env.JWT_SECRET,
      jwtExpiresIn: this.env.JWT_EXPIRES_IN,
    };
  }

  get api() {
    return {
      baseUrl: this.env.API_BASE_URL,
      key: this.env.API_KEY,
    };
  }

  get features() {
    return {
      caching: this.env.ENABLE_CACHING,
      rateLimiting: this.env.ENABLE_RATE_LIMITING,
    };
  }

  get logging() {
    return {
      level: this.env.LOG_LEVEL,
      sentryDsn: this.env.SENTRY_DSN,
    };
  }
}

export const config = new Config();

// Usage
import { config } from './config';

console.log(`Server running on ${config.server.host}:${config.server.port}`);
console.log(`Environment: ${config.isProduction ? 'PRODUCTION' : 'DEVELOPMENT'}`);

if (config.features.caching) {
  // Enable caching
}
```

---

## Example 3: Docker Compose with Secrets

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      PORT: 3000
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD}@db:5432/myapp
      REDIS_URL: redis://redis:6379
    secrets:
      - jwt_secret
      - api_key
    env_file:
      - .env.production
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

secrets:
  jwt_secret:
    file: ./secrets/jwt_secret.txt
  api_key:
    file: ./secrets/api_key.txt
  db_password:
    file: ./secrets/db_password.txt

volumes:
  postgres_data:
  redis_data:
```

---

## Example 4: Kubernetes ConfigMap and Secrets

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  NODE_ENV: "production"
  PORT: "3000"
  LOG_LEVEL: "info"
  ENABLE_CACHING: "true"
  ENABLE_RATE_LIMITING: "true"
  API_BASE_URL: "https://api.example.com"

---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  DATABASE_URL: postgresql://user:password@db:5432/myapp
  REDIS_URL: redis://redis:6379
  JWT_SECRET: your-jwt-secret-here
  API_KEY: your-api-key-here

---
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:latest
        ports:
        - containerPort: 3000
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
```

---

## Example 5: Feature Flag with Percentage Rollout

```javascript
// featureFlags.js
const crypto = require('crypto');

class FeatureFlags {
  constructor(config) {
    this.flags = config;
  }

  isEnabled(flagName, context = {}) {
    const flag = this.flags[flagName];

    if (!flag) return false;

    // Boolean flag
    if (typeof flag === 'boolean') {
      return flag;
    }

    // Percentage rollout
    if (flag.percentage !== undefined) {
      if (!context.userId) return false;

      const hash = this.hashUserId(context.userId, flagName);
      return hash < flag.percentage;
    }

    // User whitelist
    if (flag.whitelist && context.userId) {
      return flag.whitelist.includes(context.userId);
    }

    // Custom predicate
    if (typeof flag === 'function') {
      return flag(context);
    }

    return false;
  }

  hashUserId(userId, flagName) {
    const hash = crypto
      .createHash('md5')
      .update(`${userId}:${flagName}`)
      .digest('hex');
    return parseInt(hash.substring(0, 8), 16) % 100;
  }
}

const featureFlags = new FeatureFlags({
  // Simple boolean
  DARK_MODE: true,

  // Percentage rollout (25% of users)
  NEW_DASHBOARD: {
    percentage: 25,
  },

  // User whitelist
  BETA_FEATURES: {
    whitelist: ['user123', 'user456'],
  },

  // Custom logic
  ADMIN_PANEL: (context) => context.user?.role === 'admin',

  // Combined
  EXPERIMENTAL: {
    percentage: 10,
    whitelist: ['beta_tester_1'],
  },
});

// Usage
if (featureFlags.isEnabled('NEW_DASHBOARD', { userId: user.id })) {
  // Show new dashboard
}
```

---

**Versión:** 1.0.0
