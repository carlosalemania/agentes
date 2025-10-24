# Spring Boot Architect - Checklist

---

## 🏗️ Architecture
- [ ] ¿Controller → Service → Repository?
- [ ] ¿DTOs para request/response?
- [ ] ¿@Transactional en service layer?
- [ ] ¿Global exception handling?

## 🔐 Security
- [ ] ¿Spring Security configurado?
- [ ] ¿Passwords hasheadas con BCrypt?
- [ ] ¿JWT o OAuth2?
- [ ] ¿CORS configurado?

## 💾 Data Access
- [ ] ¿JPA entities con relaciones correctas?
- [ ] ¿Queries optimizadas (no N+1)?
- [ ] ¿Pagination implementada?
- [ ] ¿@Query para queries complejas?

## ✅ Validation
- [ ] ¿@Valid en controllers?
- [ ] ¿ConstraintViolationException manejada?

---

**Versión:** 1.0.0
