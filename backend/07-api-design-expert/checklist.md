# API Design Expert - Checklist

> **Checklist para diseño y revisión de APIs**

---

## 🔍 REST API Design

### Resource Naming
- [ ] ¿Recursos son sustantivos plurales? (`/users` not `/user`)
- [ ] ¿kebab-case para multi-word? (`/order-items`)
- [ ] ¿Sin verbos en URLs? (`/users` not `/getUsers`)
- [ ] ¿Jerarquía clara? (`/users/{id}/orders`)

### HTTP Methods
- [ ] ¿GET para read-only operations?
- [ ] ¿POST para crear recursos?
- [ ] ¿PUT para replace completo?
- [ ] ¿PATCH para updates parciales?
- [ ] ¿DELETE para eliminar?
- [ ] ¿Métodos son idempotentes cuando corresponde?

### Status Codes
- [ ] ¿200 para GET success?
- [ ] ¿201 para POST success con resource created?
- [ ] ¿204 para DELETE success?
- [ ] ¿400 para bad request/validation?
- [ ] ¿401 para unauthorized?
- [ ] ¿403 para forbidden?
- [ ] ¿404 para not found?
- [ ] ¿422 para semantic validation errors?
- [ ] ¿429 para rate limit exceeded?
- [ ] ¿500 para server errors?

### Pagination
- [ ] ¿List endpoints tienen paginación?
- [ ] ¿Cursor-based para large datasets?
- [ ] ¿Incluye hasNext/hasPrev?
- [ ] ¿Total count cuando es apropiado?

### Filtering & Sorting
- [ ] ¿Query params para filtering?
- [ ] ¿Consistent sort syntax?
- [ ] ¿Search functionality clara?

### Error Handling
- [ ] ¿Error format consistente?
- [ ] ¿Mensajes descriptivos?
- [ ] ¿Field-level errors para validation?
- [ ] ¿Trace ID para debugging?

### Versioning
- [ ] ¿Versioning strategy definida?
- [ ] ¿Deprecation policy clara?
- [ ] ¿Sunset headers cuando sea apropiado?

### Authentication & Authorization
- [ ] ¿Auth mechanism definido? (JWT, OAuth2, API Key)
- [ ] ¿Authorization checks implementados?
- [ ] ¿Rate limiting activo?
- [ ] ¿CORS configurado apropiadamente?

### Documentation
- [ ] ¿OpenAPI/Swagger spec completo?
- [ ] ¿Ejemplos de requests/responses?
- [ ] ¿Error responses documentados?
- [ ] ¿Authentication docs claros?

---

## 📊 GraphQL Schema

### Schema Design
- [ ] ¿Types bien definidos?
- [ ] ¿Non-null (!) usado apropiadamente?
- [ ] ¿Enums para valores finitos?
- [ ] ¿Input types para mutations?
- [ ] ¿Payload types con error handling?

### Queries
- [ ] ¿Naming descriptivo?
- [ ] ¿Pagination (Relay-style) para lists?
- [ ] ¿Filtering arguments?

### Mutations
- [ ] ¿Input types separados?
- [ ] ¿Payload con user y errors?
- [ ] ¿Naming consistente? (create, update, delete)

### Performance
- [ ] ¿DataLoader para N+1 queries?
- [ ] ¿Query complexity limiting?
- [ ] ¿Depth limiting?
- [ ] ¿Caching strategy?

---

## ✅ Pre-Release Checklist

- [ ] API spec completo (OpenAPI/GraphQL schema)
- [ ] Authentication implementado
- [ ] Rate limiting activo
- [ ] Error handling consistente
- [ ] Tests (unit, integration, E2E)
- [ ] Security audit completado
- [ ] Documentation publicada
- [ ] Versioning strategy clara

---

**Versión:** 1.0.0
