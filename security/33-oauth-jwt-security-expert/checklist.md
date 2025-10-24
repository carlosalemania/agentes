# OAuth/JWT Security Expert - Checklist

---

## 🔐 OAuth 2.0

- [ ] ¿PKCE implementado para SPAs/mobile?
- [ ] ¿State parameter para CSRF protection?
- [ ] ¿Redirect URI validation?
- [ ] ¿HTTPS enforcement?

## 🎫 JWT

- [ ] ¿RS256 (asymmetric) en lugar de HS256?
- [ ] ¿Access tokens de corta duración (15-60 min)?
- [ ] ¿Claims apropiados (sub, iat, exp)?
- [ ] ¿Token validation en cada request?

## 🔄 Refresh Tokens

- [ ] ¿Refresh tokens con rotación?
- [ ] ¿Refresh tokens en database?
- [ ] ¿Revocación implementada?
- [ ] ¿Duración apropiada (7-30 días)?

## 💾 Token Storage

- [ ] ¿Tokens NO en localStorage?
- [ ] ¿httpOnly cookies o sessionStorage?
- [ ] ¿Secure flag en cookies?
- [ ] ¿SameSite attribute configurado?

---

**Versión:** 1.0.0
