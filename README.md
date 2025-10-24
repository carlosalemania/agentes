# 🤖 Agentes Especializados - Dev Kit

> **Colección de 75 agentes especializados para desarrollo de software**

---

## 📋 Descripción

Sistema jerárquico de agentes especializados donde cada agente es un experto en una única responsabilidad. Todos los agentes heredan conocimiento del [dev-kit enterprise documentation](https://github.com/carlosalemania/-dev-kit).

Cada agente incluye:
- **README.md**: Overview y responsabilidades
- **prompt.md**: System prompt con expertise completo
- **examples.md**: Ejemplos prácticos del mundo real
- **checklist.md**: Listas de verificación
- **resources.md**: Enlaces a documentación y recursos

---

## 🎯 Agentes Implementados (25/75)

### Frontend (5)
- ✅ **01 - CSS Master** - Arquitectura CSS, BEM, performance, dark mode
- ✅ **02 - HTML Semantic Expert** - HTML5 semántico, ARIA, SEO, accesibilidad
- ✅ **03 - JavaScript Performance Optimizer** - Code splitting, Web Vitals, bundle optimization
- ✅ **04 - React/Vue/Angular Specialist** - Frameworks modernos, hooks, state management
- ✅ **05 - Responsive Design Expert** - Mobile-first, fluid typography, breakpoints

### Backend (4)
- ✅ **07 - API Design Expert** - REST, GraphQL, versioning, pagination
- ✅ **08 - Database Schema Designer** - SQL, NoSQL, normalización, indexing
- ✅ **09 - SQL Query Optimizer** - EXPLAIN, indexes, N+1 queries, performance
- ✅ **11 - GraphQL Schema Designer** - Schema design, DataLoader, resolvers

### Code Quality (2)
- ✅ **18 - Clean Code Enforcer** - SOLID, DRY, KISS, refactoring patterns
- ✅ **20 - Naming Convention Specialist** - camelCase, PascalCase, conventions

### Performance (1)
- ✅ **23 - Memory Leak Detector** - Event listeners, closures, React cleanup

### Testing (2)
- ✅ **27 - Unit Test Generator** - Jest, Vitest, pytest, GoogleTest, mocking
- ✅ **28 - E2E Test Expert** - Playwright, Cypress, Page Object Model

### Security (1)
- ✅ **32 - Security Audit Expert** - OWASP Top 10, SQL Injection, XSS, CSRF

### DevOps (1)
- ✅ **37 - CI/CD Pipeline Designer** - GitHub Actions, GitLab CI, Jenkins

### C++ (1)
- ✅ **39 - Modern C++ Expert** - C++11/14/17/20/23, RAII, smart pointers

### Java (1)
- ✅ **49 - Spring Boot Architect** - Spring MVC, Spring Security, JPA

### Python (1)
- ✅ **50 - Python Best Practices** - PEP 8, type hints, async/await

### Data Science (1)
- ✅ **55 - Python Data Analysis** - Pandas, NumPy, visualization

### Machine Learning (2)
- ✅ **59 - ML Pipeline Architect** - scikit-learn, MLflow, experiment tracking
- ✅ **65 - Feature Engineering Expert** - Encoding, scaling, feature selection

### Deep Learning (2)
- ✅ **63 - TensorFlow/Keras Architect** - Neural networks, callbacks, optimization
- ✅ **70 - PyTorch Expert** - Custom models, training loops, GPU acceleration

### AI Specialized (1)
- ✅ **74 - NLP Specialist** - Transformers, BERT, GPT, Hugging Face

---

## 📁 Estructura

```
skills/
├── frontend/
├── backend/
├── code-quality/
├── performance/
├── testing/
├── security/
├── devops/
├── cpp/
├── java/
├── python/
├── data-science/
├── machine-learning/
├── deep-learning/
├── ai-specialized/
├── integration/
├── infrastructure/
└── utilities/
```

---

## 🚀 Uso

Cada agente puede ser usado como:
1. **System prompt** completo para Claude/GPT
2. **Referencia** de best practices
3. **Checklist** para code reviews
4. **Guía** de aprendizaje

### Ejemplo de Uso

```bash
# Ver el prompt del agente CSS Master
cat frontend/01-css-master/prompt.md

# Usar checklist en code review
cat frontend/01-css-master/checklist.md
```

---

## 🎓 Roadmap (50 agentes pendientes)

Próximos agentes en desarrollo:
- TypeScript Expert
- Webpack/Vite Optimizer
- Kubernetes Specialist
- Docker Container Expert
- AWS Cloud Architect
- Redis/Cache Expert
- Message Queue Architect
- Y 43 más...

---

## 📊 Estadísticas

- **Total agentes:** 25/75 (33.3%)
- **Total archivos:** 125
- **Categorías:** 11/17
- **Líneas de código/docs:** ~14,000

---

## 🤝 Contribución

Este proyecto forma parte del dev-kit enterprise. Cada agente sigue estándares estrictos de documentación y calidad.

---

## 📄 Licencia

MIT License - Ver repositorio principal para detalles

---

**Versión:** 1.0.0
**Última actualización:** 2025-01-24
