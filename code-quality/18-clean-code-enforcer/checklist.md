# Clean Code Enforcer - Checklist

> **Checklist para code review y refactoring**

---

## ✅ SOLID Principles

### Single Responsibility
- [ ] ¿Cada clase tiene una sola razón para cambiar?
- [ ] ¿Responsabilidades bien definidas?
- [ ] ¿Sin God classes?

### Open/Closed
- [ ] ¿Abierto para extensión?
- [ ] ¿Cerrado para modificación?
- [ ] ¿Usa polimorfismo en lugar de conditionals?

### Liskov Substitution
- [ ] ¿Subclases son sustituibles?
- [ ] ¿No lanzan excepciones inesperadas?
- [ ] ¿Cumplen contratos de la clase base?

### Interface Segregation
- [ ] ¿Interfaces pequeñas y cohesivas?
- [ ] ¿No fuerzan implementación innecesaria?

### Dependency Inversion
- [ ] ¿Depende de abstracciones?
- [ ] ¿No depende de implementaciones concretas?
- [ ] ¿Usa inyección de dependencias?

---

## 📝 Code Quality

### DRY
- [ ] ¿Sin duplicación de código?
- [ ] ¿Lógica extraída en funciones reutilizables?

### KISS
- [ ] ¿Solución simple?
- [ ] ¿Sin over-engineering?

### YAGNI
- [ ] ¿Sin código especulativo?
- [ ] ¿Solo lo necesario implementado?

---

## 🚫 Code Smells

- [ ] ❌ No hay métodos largos (>20 líneas)
- [ ] ❌ No hay clases grandes (>200 líneas)
- [ ] ❌ No hay feature envy
- [ ] ❌ No hay data clumps
- [ ] ❌ No hay primitive obsession
- [ ] ❌ No hay shotgun surgery

---

**Versión:** 1.0.0
