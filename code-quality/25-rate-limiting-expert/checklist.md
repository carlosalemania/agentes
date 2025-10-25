# Rate Limiting Expert - Checklist

---

## 🚦 Rate Limiting Implementation

- [ ] ¿Rate limiting implementado en todos los endpoints públicos?
- [ ] ¿Diferentes límites para diferentes endpoints?
- [ ] ¿Per-IP y per-user rate limiting?
- [ ] ¿Distributed rate limiting con Redis?
- [ ] ¿Algorithm apropiado (token bucket, sliding window)?

## 🔒 Authentication Protection

- [ ] ¿Login endpoints rate limited (5 attempts / 15 min)?
- [ ] ¿Password reset rate limited?
- [ ] ¿Registration rate limited?
- [ ] ¿Successful attempts no contados?
- [ ] ¿Account lockout después de múltiples intentos?

## 📊 Headers & Response

- [ ] ¿Rate limit headers incluidos (X-RateLimit-*)?
- [ ] ¿Retry-After header en 429 responses?
- [ ] ¿Mensajes de error claros?
- [ ] ¿HTTP 429 status code usado?
- [ ] ¿Documentación de límites para usuarios?

## 🎯 Tier-Based Limits

- [ ] ¿Diferentes límites por tier (free, premium, enterprise)?
- [ ] ¿User tier determinado correctamente?
- [ ] ¿Upgrade path claro en error messages?
- [ ] ¿Límites documentados por tier?

## 🛡️ DDoS Protection

- [ ] ¿Burst detection implementado?
- [ ] ¿IP banning automático para abusers?
- [ ] ¿Whitelist para IPs confiables?
- [ ] ¿Logging de violations?
- [ ] ¿Alertas para ataques DDoS?

## 🔄 Circuit Breakers

- [ ] ¿Circuit breakers para servicios externos?
- [ ] ¿Fallback mechanisms?
- [ ] ¿Timeout configurado?
- [ ] ¿Half-open state testing?
- [ ] ¿Metrics de circuit breaker estado?

## ⚙️ Configuration

- [ ] ¿Límites configurables por environment?
- [ ] ¿Easy to adjust sin deploy?
- [ ] ¿Feature flags para rate limiting?
- [ ] ¿Monitoring de rate limit hits?

## 📈 Monitoring

- [ ] ¿Rate limit violations logged?
- [ ] ¿Dashboards para rate limit metrics?
- [ ] ¿Alertas para unusual patterns?
- [ ] ¿Análisis de abusers?

---

**Versión:** 1.0.0
