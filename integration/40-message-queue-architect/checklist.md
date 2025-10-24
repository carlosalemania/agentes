# Message Queue Architect - Checklist

---

## 📨 Reliability

- [ ] ¿Message acknowledgment implementado?
- [ ] ¿Dead letter queue configurada?
- [ ] ¿Retry strategy con exponential backoff?
- [ ] ¿Idempotency garantizada?

## ⚡ Performance

- [ ] ¿Consumer groups para paralelismo?
- [ ] ¿Batch processing habilitado?
- [ ] ¿Partitioning strategy optimizada?
- [ ] ¿Prefetch/concurrency configurado?

## 🔍 Monitoring

- [ ] ¿Queue depth monitoreado?
- [ ] ¿Consumer lag tracking?
- [ ] ¿Message processing time metrics?
- [ ] ¿Alarmas de DLQ configuradas?

## 🎯 Patterns

- [ ] ¿Pattern apropiado (pub/sub, P2P, fanout)?
- [ ] ¿Message ordering si es necesario?
- [ ] ¿TTL configurado para evitar acumulación?

---

**Versión:** 1.0.0
