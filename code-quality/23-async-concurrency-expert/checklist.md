# Async/Concurrency Expert - Checklist

---

## ⚡ Async Patterns

- [ ] ¿Promise.all usado para operaciones paralelas independientes?
- [ ] ¿await en secuencia solo cuando hay dependencias?
- [ ] ¿Promise.allSettled para mixed success/failure?
- [ ] ¿Promise.race para timeouts?
- [ ] ¿Async/await con try-catch proper?

## 🔄 Concurrency Control

- [ ] ¿Concurrency limitada para evitar overwhelm?
- [ ] ¿Rate limiting implementado para APIs externas?
- [ ] ¿Semaphore/mutex para recursos compartidos?
- [ ] ¿Task queue para ordenar operaciones?
- [ ] ¿Batch processing con límite concurrency?

## 🎯 Performance

- [ ] ¿Operaciones CPU-intensive en worker threads?
- [ ] ¿Event loop no bloqueado?
- [ ] ¿Paralelismo máximo sin overwhelm?
- [ ] ¿Streaming data cuando posible?
- [ ] ¿Lazy loading implementado?

## ⏱️ Timeouts & Cancellation

- [ ] ¿Timeouts en todas las operaciones async externas?
- [ ] ¿AbortController para cancellation?
- [ ] ¿Cleanup en cancellation?
- [ ] ¿Graceful timeout handling?

## 🐛 Error Handling

- [ ] ¿Unhandled promise rejections manejados?
- [ ] ¿Error propagation correcta?
- [ ] ¿Retry logic con exponential backoff?
- [ ] ¿Fallback mechanisms?
- [ ] ¿Errors logged con contexto?

## 🚦 Rate Limiting

- [ ] ¿Debouncing en user input?
- [ ] ¿Throttling en scroll/resize?
- [ ] ¿API rate limits respetados?
- [ ] ¿Queue para burst traffic?

## 📊 Monitoring

- [ ] ¿Progress tracking para long operations?
- [ ] ¿Metrics de concurrency levels?
- [ ] ¿Queue length monitored?
- [ ] ¿Event loop lag detectado?

---

**Versión:** 1.0.0
