# Redis/Cache Expert - Examples

---

## Example: Session Management

```javascript
// Store session
await redis.setex(
  `session:${sessionId}`,
  86400, // 24 hours
  JSON.stringify({ userId, data })
);

// Get session
const session = JSON.parse(
  await redis.get(`session:${sessionId}`)
);
```

---

**Versión:** 1.0.0
