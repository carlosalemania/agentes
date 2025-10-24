# Security Audit Expert - Checklist

---

## 🔐 Authentication
- [ ] ¿Passwords hasheadas con bcrypt/argon2?
- [ ] ¿JWT con expiración?
- [ ] ¿Rate limiting en login?
- [ ] ¿2FA/MFA disponible?

## 🛡️ Authorization
- [ ] ¿Ownership checks (prevenir IDOR)?
- [ ] ¿Role-based access control (RBAC)?
- [ ] ¿Principle of least privilege?

## 💉 Injection Prevention
- [ ] ¿Parameterized queries (no string concat)?
- [ ] ¿Input validation en todos los endpoints?
- [ ] ¿Output encoding para prevenir XSS?

## 🍪 CSRF Protection
- [ ] ¿CSRF tokens en forms?
- [ ] ¿SameSite cookies?

## 📦 Dependencies
- [ ] ¿npm audit ejecutado regularmente?
- [ ] ¿Dependabot/Snyk configurado?
- [ ] ¿Packages actualizados?

## 🔑 Secrets
- [ ] ¿Secrets en env variables?
- [ ] ¿.env en .gitignore?
- [ ] ¿No hardcoded API keys?

## 🌐 Headers
- [ ] ¿Helmet.js configurado?
- [ ] ¿CSP headers?
- [ ] ¿HSTS headers?
- [ ] ¿HTTPS enforced?

---

**Versión:** 1.0.0
