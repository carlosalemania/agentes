# Caching Strategy Examples

## Example: API Response Caching

```javascript
const express = require('express');
const Redis = require('ioredis');
const redis = new Redis();

const app = express();

// Cache middleware
function cacheMiddleware(ttl = 300) {
  return async (req, res, next) => {
    const key = `cache:${req.method}:${req.originalUrl}`;

    try {
      const cached = await redis.get(key);

      if (cached) {
        return res.json(JSON.parse(cached));
      }

      // Store original send
      const originalSend = res.json;

      // Override json method
      res.json = function (data) {
        // Cache the response
        redis.setex(key, ttl, JSON.stringify(data));

        // Send response
        originalSend.call(this, data);
      };

      next();
    } catch (error) {
      console.error('Cache error:', error);
      next();
    }
  };
}

// Usage
app.get('/api/products', cacheMiddleware(600), async (req, res) => {
  const products = await db.products.find();
  res.json(products);
});

// Invalidation endpoint
app.post('/api/products/:id', async (req, res) => {
  const product = await db.products.update(req.params.id, req.body);

  // Invalidate related caches
  const keys = await redis.keys('cache:GET:/api/products*');
  if (keys.length > 0) {
    await redis.del(...keys);
  }

  res.json(product);
});
```

## Example: Session Store with Redis

```javascript
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const Redis = require('ioredis');

const redis = new Redis({
  host: 'localhost',
  port: 6379,
});

app.use(
  session({
    store: new RedisStore({ client: redis }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: process.env.NODE_ENV === 'production',
      httpOnly: true,
      maxAge: 1000 * 60 * 60 * 24, // 24 hours
    },
  })
);

// Session is now stored in Redis
app.get('/profile', (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  res.json({ userId: req.session.userId });
});
```

## Example: Query Result Caching

```javascript
class UserRepository {
  constructor(db, cache) {
    this.db = db;
    this.cache = cache;
  }

  async findById(id) {
    const key = `user:${id}`;

    // Try cache
    const cached = await this.cache.get(key);
    if (cached) {
      return cached;
    }

    // Query database
    const user = await this.db.users.findById(id);

    if (user) {
      // Cache for 1 hour
      await this.cache.set(key, user, 3600);
    }

    return user;
  }

  async update(id, data) {
    const user = await this.db.users.update(id, data);

    // Invalidate cache
    await this.cache.invalidate(`user:${id}`);

    return user;
  }

  async findByEmail(email) {
    const key = `user:email:${email}`;

    const cached = await this.cache.get(key);
    if (cached) {
      return cached;
    }

    const user = await this.db.users.findOne({ email });

    if (user) {
      // Cache both by ID and email
      await this.cache.set(`user:${user.id}`, user, 3600);
      await this.cache.set(key, user, 3600);
    }

    return user;
  }
}
```
