# SQL Query Optimizer - Examples

---

## Example: Optimized Pagination

```sql
-- ❌ SLOW - OFFSET for large datasets
SELECT * FROM posts
ORDER BY created_at DESC
LIMIT 20 OFFSET 10000;

-- ✅ FAST - Cursor-based pagination
SELECT * FROM posts
WHERE created_at < '2024-01-20T12:00:00'
ORDER BY created_at DESC
LIMIT 20;
```

---

**Versión:** 1.0.0
