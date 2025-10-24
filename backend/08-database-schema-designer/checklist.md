# Database Schema Designer - Checklist

---

## 🗄️ Schema Design
- [ ] ¿Está normalizado a 3NF como mínimo?
- [ ] ¿Primary keys en todas las tablas?
- [ ] ¿Foreign keys con ON DELETE/UPDATE clauses?
- [ ] ¿Constraints (UNIQUE, CHECK, NOT NULL)?
- [ ] ¿Tipos de datos apropiados?

## 📊 Indexing
- [ ] ¿Índices para foreign keys?
- [ ] ¿Índices para columnas en WHERE frecuentes?
- [ ] ¿Composite indexes para queries multi-columna?
- [ ] ¿Evitando over-indexing?
- [ ] ¿Partial indexes para casos específicos?

## ⚡ Performance
- [ ] ¿Denormalización justificada documentada?
- [ ] ¿Partitioning considerado para tablas grandes?
- [ ] ¿EXPLAIN ANALYZE ejecutado en queries críticas?
- [ ] ¿Columnas calculadas con GENERATED STORED?

## 🔒 Data Integrity
- [ ] ¿Constraints para reglas de negocio?
- [ ] ¿Default values apropiados?
- [ ] ¿Triggers para maintain denormalized data?
- [ ] ¿Cascading deletes configurados correctamente?

---

**Versión:** 1.0.0
