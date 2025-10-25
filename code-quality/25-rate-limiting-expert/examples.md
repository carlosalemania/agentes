# Rate Limiting Expert - Examples

---

## Example 1: Complete Express App with Rate Limiting

```javascript
// app.js
const express = require('express');
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const Redis = require('ioredis');

const app = express();
const redis = new Redis(process.env.REDIS_URL);

// Global rate limiter
const globalLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:global:',
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
  message: { error: 'Too many requests, please try again later' },
});

// Apply to all requests
app.use(globalLimiter);

// Strict limiter for auth endpoints
const authLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:auth:',
  }),
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  message: { error: 'Too many authentication attempts' },
});

// More lenient for public endpoints
const publicLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 30,
});

// Routes
app.post('/auth/login', authLimiter, async (req, res) => {
  // Login logic
  const { email, password } = req.body;

  try {
    const user = await authenticateUser(email, password);
    res.json({ token: generateToken(user) });
  } catch (error) {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

app.post('/auth/register', authLimiter, async (req, res) => {
  // Registration logic
  const user = await createUser(req.body);
  res.status(201).json({ user });
});

app.get('/api/public/data', publicLimiter, async (req, res) => {
  const data = await getPublicData();
  res.json(data);
});

// Premium users - higher limits
const premiumLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 1000,
  skip: (req) => !req.user?.isPremium,
});

app.get('/api/premium/data', premiumLimiter, async (req, res) => {
  const data = await getPremiumData();
  res.json(data);
});

app.listen(3000);
```

---

## Example 2: Multi-Tier Rate Limiting

```javascript
// multi-tier-limiter.js
class MultiTierRateLimiter {
  constructor(redis) {
    this.redis = redis;

    this.tiers = {
      free: { limit: 10, window: 60 },      // 10 req/min
      basic: { limit: 100, window: 60 },    // 100 req/min
      premium: { limit: 1000, window: 60 }, // 1000 req/min
      enterprise: { limit: 10000, window: 60 }, // 10000 req/min
    };
  }

  async check(userId, tier = 'free') {
    const config = this.tiers[tier];
    if (!config) {
      throw new Error(`Unknown tier: ${tier}`);
    }

    const key = `rate-limit:${tier}:${userId}`;
    const now = Date.now();
    const windowStart = now - config.window * 1000;

    // Remove old entries
    await this.redis.zremrangebyscore(key, 0, windowStart);

    // Count requests
    const count = await this.redis.zcard(key);

    if (count >= config.limit) {
      // Get time until oldest request expires
      const oldest = await this.redis.zrange(key, 0, 0, 'WITHSCORES');
      const retryAfter = Math.ceil((oldest[1] - windowStart) / 1000);

      return {
        allowed: false,
        limit: config.limit,
        remaining: 0,
        retryAfter,
        tier,
      };
    }

    // Add current request
    await this.redis.zadd(key, now, `${now}-${Math.random()}`);
    await this.redis.expire(key, config.window);

    return {
      allowed: true,
      limit: config.limit,
      remaining: config.limit - count - 1,
      retryAfter: null,
      tier,
    };
  }

  middleware() {
    return async (req, res, next) => {
      const userId = req.user?.id || req.ip;
      const tier = req.user?.tier || 'free';

      const result = await this.check(userId, tier);

      // Set headers
      res.setHeader('X-RateLimit-Limit', result.limit);
      res.setHeader('X-RateLimit-Remaining', result.remaining);
      res.setHeader('X-RateLimit-Tier', result.tier);

      if (!result.allowed) {
        res.setHeader('Retry-After', result.retryAfter);
        return res.status(429).json({
          error: 'Rate limit exceeded',
          tier: result.tier,
          limit: result.limit,
          retryAfter: result.retryAfter,
          upgradeMessage: tier === 'free' ? 'Upgrade to get higher limits' : null,
        });
      }

      next();
    };
  }
}

// Usage
const limiter = new MultiTierRateLimiter(redis);
app.use(limiter.middleware());
```

---

## Example 3: Circuit Breaker with Fallback

```javascript
// circuit-breaker.js
class CircuitBreaker {
  constructor(fn, options = {}) {
    this.fn = fn;
    this.failureThreshold = options.failureThreshold || 5;
    this.successThreshold = options.successThreshold || 2;
    this.timeout = options.timeout || 60000;
    this.fallback = options.fallback || (() => null);

    this.state = 'CLOSED';
    this.failures = 0;
    this.successes = 0;
    this.nextAttempt = Date.now();
    this.stats = {
      total: 0,
      successes: 0,
      failures: 0,
      fallbacks: 0,
    };
  }

  async execute(...args) {
    this.stats.total++;

    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        console.log('Circuit breaker OPEN, using fallback');
        this.stats.fallbacks++;
        return this.fallback(...args);
      }
      this.state = 'HALF_OPEN';
      console.log('Circuit breaker HALF_OPEN, trying request');
    }

    try {
      const result = await this.fn(...args);
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure(error);

      if (this.state === 'OPEN') {
        console.log('Using fallback after failure');
        this.stats.fallbacks++;
        return this.fallback(...args);
      }

      throw error;
    }
  }

  onSuccess() {
    this.failures = 0;
    this.stats.successes++;

    if (this.state === 'HALF_OPEN') {
      this.successes++;

      if (this.successes >= this.successThreshold) {
        this.state = 'CLOSED';
        this.successes = 0;
        console.log('Circuit breaker CLOSED');
      }
    }
  }

  onFailure(error) {
    this.failures++;
    this.successes = 0;
    this.stats.failures++;

    console.error('Circuit breaker failure:', error.message);

    if (this.failures >= this.failureThreshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
      console.log(`Circuit breaker OPEN (retry after ${this.timeout}ms)`);
    }
  }

  getStats() {
    const successRate = this.stats.total > 0
      ? ((this.stats.successes / this.stats.total) * 100).toFixed(2)
      : 0;

    return {
      state: this.state,
      stats: {
        ...this.stats,
        successRate: `${successRate}%`,
      },
    };
  }

  reset() {
    this.state = 'CLOSED';
    this.failures = 0;
    this.successes = 0;
    this.stats = {
      total: 0,
      successes: 0,
      failures: 0,
      fallbacks: 0,
    };
  }
}

// Usage
const cachedData = new Map();

const apiCall = new CircuitBreaker(
  async (endpoint) => {
    const response = await fetch(`https://api.example.com${endpoint}`);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    const data = await response.json();

    // Cache successful response
    cachedData.set(endpoint, data);

    return data;
  },
  {
    failureThreshold: 5,
    successThreshold: 2,
    timeout: 60000,
    fallback: (endpoint) => {
      console.log('Using cached data');
      return cachedData.get(endpoint) || { error: 'Service unavailable' };
    },
  }
);

// Use circuit breaker
app.get('/api/data', async (req, res) => {
  try {
    const data = await apiCall.execute('/data');
    res.json(data);
  } catch (error) {
    res.status(503).json({ error: 'Service unavailable' });
  }
});

// Stats endpoint
app.get('/circuit-breaker/stats', (req, res) => {
  res.json(apiCall.getStats());
});
```

---

## Example 4: DDoS Protection Middleware

```javascript
// ddos-protection.js
class DDoSProtection {
  constructor(redis, options = {}) {
    this.redis = redis;

    // Burst detection
    this.burstThreshold = options.burstThreshold || 50; // requests
    this.burstWindow = options.burstWindow || 10; // seconds

    // Ban settings
    this.banDuration = options.banDuration || 3600; // 1 hour
    this.banThreshold = options.banThreshold || 3; // violations

    // Whitelist
    this.whitelist = new Set(options.whitelist || []);
  }

  async check(ip) {
    // Check whitelist
    if (this.whitelist.has(ip)) {
      return { allowed: true, whitelisted: true };
    }

    // Check if banned
    const banKey = `ddos:ban:${ip}`;
    const isBanned = await this.redis.get(banKey);

    if (isBanned) {
      const ttl = await this.redis.ttl(banKey);
      return {
        allowed: false,
        banned: true,
        retryAfter: ttl,
      };
    }

    // Check burst
    const burstKey = `ddos:burst:${ip}`;
    const now = Date.now();
    const windowStart = now - this.burstWindow * 1000;

    await this.redis.zremrangebyscore(burstKey, 0, windowStart);
    const count = await this.redis.zcard(burstKey);

    if (count >= this.burstThreshold) {
      // Burst detected - increment violations
      await this.incrementViolations(ip);

      return {
        allowed: false,
        burst: true,
        violations: await this.getViolations(ip),
      };
    }

    // Add to burst tracking
    await this.redis.zadd(burstKey, now, `${now}-${Math.random()}`);
    await this.redis.expire(burstKey, this.burstWindow);

    return { allowed: true };
  }

  async incrementViolations(ip) {
    const violationsKey = `ddos:violations:${ip}`;
    const violations = await this.redis.incr(violationsKey);
    await this.redis.expire(violationsKey, 3600); // Reset after 1 hour

    if (violations >= this.banThreshold) {
      await this.banIP(ip);
    }
  }

  async getViolations(ip) {
    const violationsKey = `ddos:violations:${ip}`;
    return parseInt(await this.redis.get(violationsKey) || 0);
  }

  async banIP(ip) {
    const banKey = `ddos:ban:${ip}`;
    await this.redis.setex(banKey, this.banDuration, '1');

    console.warn(`⚠️ IP banned for DDoS: ${ip} (${this.banDuration}s)`);

    // Log to external service (Sentry, CloudWatch, etc.)
    this.logBan(ip);
  }

  async unbanIP(ip) {
    const banKey = `ddos:ban:${ip}`;
    await this.redis.del(banKey);

    const violationsKey = `ddos:violations:${ip}`;
    await this.redis.del(violationsKey);

    console.log(`IP unbanned: ${ip}`);
  }

  logBan(ip) {
    // Log to monitoring service
    console.error('DDoS Ban:', {
      ip,
      timestamp: new Date().toISOString(),
      duration: this.banDuration,
    });
  }

  middleware() {
    return async (req, res, next) => {
      const ip = req.ip;
      const result = await this.check(ip);

      if (!result.allowed) {
        if (result.banned) {
          return res.status(403).json({
            error: 'Access forbidden',
            retryAfter: result.retryAfter,
          });
        }

        if (result.burst) {
          return res.status(429).json({
            error: 'Too many requests',
            violations: result.violations,
            threshold: this.banThreshold,
          });
        }
      }

      next();
    };
  }
}

// Usage
const ddosProtection = new DDoSProtection(redis, {
  burstThreshold: 50,
  burstWindow: 10,
  banDuration: 3600,
  banThreshold: 3,
  whitelist: ['127.0.0.1', '::1'],
});

app.use(ddosProtection.middleware());

// Admin endpoint to unban IPs
app.post('/admin/unban/:ip', async (req, res) => {
  await ddosProtection.unbanIP(req.params.ip);
  res.json({ message: 'IP unbanned' });
});
```

---

**Versión:** 1.0.0
