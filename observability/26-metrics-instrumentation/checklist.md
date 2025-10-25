# Metrics & Instrumentation - Checklist

---

## 📊 Metric Types

- [ ] ¿Counter usado para valores incrementales?
- [ ] ¿Gauge usado para valores que suben/bajan?
- [ ] ¿Histogram para distribuciones de latencia?
- [ ] ¿Summary para percentiles pre-calculated?
- [ ] ¿Tipo correcto para cada métrica?

## 🏷️ Naming & Labels

- [ ] ¿Metric names en snake_case?
- [ ] ¿Nombres descriptivos y claros?
- [ ] ¿Sufijo apropiado (_total, _seconds, _bytes)?
- [ ] ¿Labels con baja cardinalidad?
- [ ] ¿No user IDs en labels?
- [ ] ¿Units en help text, no en nombre?

## 🎯 HTTP Metrics

- [ ] ¿Request count tracked?
- [ ] ¿Request duration histogram?
- [ ] ¿Error rate counter?
- [ ] ¿Active requests gauge?
- [ ] ¿Labels: method, route, status_code?

## 🗄️ Database Metrics

- [ ] ¿Query duration tracked?
- [ ] ¿Query count por operation type?
- [ ] ¿Connection pool size monitoreado?
- [ ] ¿Slow queries detectados?
- [ ] ¿N+1 queries alertados?

## 💼 Business Metrics

- [ ] ¿Key business events tracked?
- [ ] ¿Revenue metrics?
- [ ] ¿User signup/churn?
- [ ] ¿Conversion rates?
- [ ] ¿Feature usage?

## 📈 Visualization

- [ ] ¿/metrics endpoint expuesto?
- [ ] ¿Prometheus scraping configurado?
- [ ] ¿Grafana dashboards creados?
- [ ] ¿Alerting rules definidas?
- [ ] ¿SLO/SLI tracking?

## ⚡ Performance

- [ ] ¿Sampling para high-volume metrics?
- [ ] ¿Aggregation apropiada?
- [ ] ¿No blocking en metric collection?
- [ ] ¿Metric storage size monitoreado?

## 🔧 Best Practices

- [ ] ¿Default metrics habilitadas?
- [ ] ¿Critical paths instrumentados first?
- [ ] ¿Histogram buckets apropiados?
- [ ] ¿Documentation de métricas?
- [ ] ¿Consistent metric naming?

---

**Versión:** 1.0.0
