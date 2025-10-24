# Database Migration Specialist - Checklist

---

## 🔒 Safety

- [ ] ¿Backup completo antes de migración?
- [ ] ¿Tested en staging primero?
- [ ] ¿Rollback plan definido?
- [ ] ¿Migration time estimado?

## 📋 Migration Design

- [ ] ¿Backward compatible changes?
- [ ] ¿Expand-contract pattern para zero-downtime?
- [ ] ¿Nullable columns primero, NOT NULL después?
- [ ] ¿Indexes creados concurrently?

## 🔄 Execution

- [ ] ¿Migrations en transacciones cuando posible?
- [ ] ¿Batch processing para tablas grandes?
- [ ] ¿Monitoring durante migration?
- [ ] ¿Validation después de migration?

## 📊 Data Migrations

- [ ] ¿Data transformations tested?
- [ ] ¿Idempotent migrations?
- [ ] ¿Progress tracking para large datasets?

---

**Versión:** 1.0.0
