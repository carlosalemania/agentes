# Logging Best Practices - Checklist

---

## 📋 Structured Logging

- [ ] ¿Logs en formato JSON para parsing?
- [ ] ¿Campos consistentes en todos los logs?
- [ ] ¿Timestamp en ISO 8601?
- [ ] ¿Service/application name incluido?

## 🔍 Context & Correlation

- [ ] ¿Correlation ID en cada request?
- [ ] ¿User ID incluido cuando disponible?
- [ ] ¿Correlation ID propagado a servicios externos?
- [ ] ¿Stack traces completos en errores?

## 🎚️ Log Levels

- [ ] ¿ERROR solo para errores reales?
- [ ] ¿WARN para situaciones recuperables?
- [ ] ¿INFO para eventos importantes?
- [ ] ¿DEBUG/TRACE solo en desarrollo?
- [ ] ¿Log level configurable por environment?

## 🔒 Security

- [ ] ¿Passwords/tokens nunca loggeados?
- [ ] ¿PII data redacted automáticamente?
- [ ] ¿Sensitive headers filtrados?
- [ ] ¿Compliance requirements met (GDPR, HIPAA)?

## ⚡ Performance

- [ ] ¿Logging asíncrono cuando posible?
- [ ] ¿Sampling para high-volume logs?
- [ ] ¿Log rotation configurado?
- [ ] ¿Impacto de performance medido?

## 📊 Aggregation & Search

- [ ] ¿Logs enviados a sistema centralizado?
- [ ] ¿Retention policy definida?
- [ ] ¿Logs searchable por correlation ID?
- [ ] ¿Dashboards y alertas configurados?

## 🎯 Best Practices

- [ ] ¿Mensajes de log descriptivos?
- [ ] ¿Contexto suficiente para debugging?
- [ ] ¿No logging excesivo (noise)?
- [ ] ¿Timezone consistency (UTC)?

---

**Versión:** 1.0.0
