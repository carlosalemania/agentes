# Naming Convention Specialist - Resources

> **Referencias al Dev-Kit padre y recursos externos**

---

## 📚 Referencias al Dev-Kit Padre

### Documentación Core
Este agente se fundamenta en los siguientes documentos del dev-kit:

#### [CODE-QUALITY-METRICS.md](../../../docs/CODE-QUALITY-METRICS.md)
- **Maintainability Index**: Nombres claros mejoran mantenibilidad
- **Cognitive Complexity**: Buenos nombres reducen complejidad mental
- **Code Duplication**: DRY en naming (un concepto = una palabra)

**Aplicación en Naming:**
```javascript
// ❌ BAJA MANTENIBILIDAD - Nombres crípticos
function proc(u, o) {
  const x = u.n + ' ' + u.l;
  const y = o.t * o.q;
  return { x, y };
}

// ✅ ALTA MANTENIBILIDAD - Nombres claros
function formatOrderSummary(user, order) {
  const customerFullName = user.firstName + ' ' + user.lastName;
  const orderTotal = order.price * order.quantity;
  return { customerFullName, orderTotal };
}
```

#### [CLEAN-CODE-PRINCIPLES.md](../../../docs/CLEAN-CODE-PRINCIPLES.md)
- **Meaningful Names**: Reveal intention
- **Avoid Disinformation**: No misleading names
- **Make Distinctions**: No noise words
- **Pronounceable Names**: Easy to discuss
- **Searchable Names**: Length corresponds to scope

#### [LESSONS-LEARNED-GODOT.md](../../../docs/LESSONS-LEARNED-GODOT.md)
- **Naming Patterns**: Observer pattern naming, Factory pattern naming
- **Domain Language**: Game entities use domain terms
- **Consistency**: Naming conventions across codebase

---

## 🛠️ Herramientas

### Linters y Analyzers

#### ESLint (JavaScript/TypeScript)
- **Plugin**: `@typescript-eslint/naming-convention`
```json
{
  "rules": {
    "@typescript-eslint/naming-convention": [
      "error",
      {
        "selector": "variable",
        "format": ["camelCase", "UPPER_CASE"]
      },
      {
        "selector": "function",
        "format": ["camelCase"]
      },
      {
        "selector": "typeLike",
        "format": ["PascalCase"]
      }
    ]
  }
}
```
- **URL**: [https://eslint.org/](https://eslint.org/)
- **Naming Convention Rule**: [https://typescript-eslint.io/rules/naming-convention/](https://typescript-eslint.io/rules/naming-convention/)

#### Pylint (Python)
- **Rules**: `invalid-name`, `naming-style`
```ini
[BASIC]
good-names=i,j,k,x,y,z,id
variable-naming-style=snake_case
function-naming-style=snake_case
class-naming-style=PascalCase
const-naming-style=UPPER_CASE
```
- **URL**: [https://pylint.org/](https://pylint.org/)
- **Naming Convention**: [https://pylint.pycqa.org/en/latest/user_guide/messages/convention/invalid-name.html](https://pylint.pycqa.org/en/latest/user_guide/messages/convention/invalid-name.html)

#### SonarQube
- **Analysis**: Naming conventions, code smells
- **Rules**: `S100` (method names), `S101` (class names), `S116` (field names)
- **URL**: [https://www.sonarqube.org/](https://www.sonarqube.org/)

#### IntelliJ IDEA / WebStorm
- **Inspections**: Naming convention inspections
- **Refactoring**: Safe rename (Shift + F6)
- **URL**: [https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)

#### VS Code Extensions
- **Better Comments**: Highlight naming issues
- **SonarLint**: Real-time naming analysis
- **ESLint**: Inline naming violations
- **Pylint**: Python naming checks

---

## 📖 Style Guides

### JavaScript/TypeScript

#### Airbnb JavaScript Style Guide
- **Naming**: Comprehensive naming conventions
- **URL**: [https://github.com/airbnb/javascript](https://github.com/airbnb/javascript)
- **Naming Section**: camelCase, PascalCase, SCREAMING_SNAKE_CASE

#### Google JavaScript Style Guide
- **Naming**: Clear rules for all identifiers
- **URL**: [https://google.github.io/styleguide/jsguide.html](https://google.github.io/styleguide/jsguide.html)

#### TypeScript Style Guide (Microsoft)
- **Naming**: TypeScript-specific conventions
- **URL**: [https://github.com/microsoft/TypeScript/wiki/Coding-guidelines](https://github.com/microsoft/TypeScript/wiki/Coding-guidelines)

### Python

#### PEP 8 - Style Guide for Python Code
- **Naming Conventions**: Official Python naming
- **URL**: [https://peps.python.org/pep-0008/](https://peps.python.org/pep-0008/)
- **Key Sections**:
  - Function names: lowercase with underscores
  - Class names: CapWords convention
  - Constants: ALL_CAPS with underscores

#### Google Python Style Guide
- **Naming**: Google's Python conventions
- **URL**: [https://google.github.io/styleguide/pyguide.html](https://google.github.io/styleguide/pyguide.html)

### Java

#### Oracle Java Code Conventions
- **Naming**: Official Java conventions
- **URL**: [https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html](https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html)

#### Google Java Style Guide
- **Naming**: Google's Java conventions
- **URL**: [https://google.github.io/styleguide/javaguide.html](https://google.github.io/styleguide/javaguide.html)

### C++

#### Google C++ Style Guide
- **Naming**: Comprehensive C++ naming
- **URL**: [https://google.github.io/styleguide/cppguide.html](https://google.github.io/styleguide/cppguide.html)

#### LLVM Coding Standards
- **Naming**: LLVM project conventions
- **URL**: [https://llvm.org/docs/CodingStandards.html](https://llvm.org/docs/CodingStandards.html)

#### C++ Core Guidelines
- **Naming**: Modern C++ conventions
- **URL**: [https://isocpp.github.io/CppCoreGuidelines/](https://isocpp.github.io/CppCoreGuidelines/)

### CSS/SCSS

#### BEM (Block Element Modifier)
- **Naming**: `.block__element--modifier`
- **URL**: [https://getbem.com/naming/](https://getbem.com/naming/)

#### CSS Guidelines by Harry Roberts
- **Naming**: Pragmatic CSS conventions
- **URL**: [https://cssguidelin.es/](https://cssguidelin.es/)

---

## 📚 Libros

### Clean Code
- **Autor**: Robert C. Martin
- **Capítulo**: Chapter 2 - Meaningful Names
- **Temas**:
  - Use Intention-Revealing Names
  - Avoid Disinformation
  - Make Meaningful Distinctions
  - Use Pronounceable Names
  - Use Searchable Names
  - Avoid Encodings
  - Avoid Mental Mapping
  - Class Names and Method Names
  - Pick One Word per Concept

### Domain-Driven Design
- **Autor**: Eric Evans
- **Tema**: Ubiquitous Language
- **Aplicación**: Naming basado en el dominio del negocio

### Refactoring: Improving the Design of Existing Code
- **Autor**: Martin Fowler
- **Refactorings**:
  - Rename Variable
  - Rename Function
  - Rename Field
  - Extract Variable
  - Inline Variable

### The Art of Readable Code
- **Autores**: Dustin Boswell, Trevor Foucher
- **Temas**: Naming, readability, self-documenting code

---

## 🎓 Learning Resources

### Articles

#### Naming Things in Code
- **URL**: [https://dmitripavlutin.com/coding-like-shakespeare-practical-function-naming-conventions/](https://dmitripavlutin.com/coding-like-shakespeare-practical-function-naming-conventions/)
- **Tema**: Function naming conventions

#### The Art of Naming Variables
- **URL**: [https://medium.com/@abstarreveld/the-art-of-naming-variables-52f44de00aad](https://medium.com/@abstarreveld/the-art-of-naming-variables-52f44de00aad)
- **Tema**: Variable naming best practices

#### Why Naming Things is Hard
- **URL**: [https://neilkakkar.com/why-naming-things-is-hard.html](https://neilkakkar.com/why-naming-things-is-hard.html)
- **Tema**: Challenges in naming

### Videos

#### Naming Things in Code (Kevlin Henney)
- **Platform**: YouTube
- **Tema**: Comprehensive talk on naming

#### Clean Code - Uncle Bob (Meaningful Names)
- **Platform**: YouTube / Pluralsight
- **Tema**: Chapter 2 explanations

---

## 🔗 Related Dev-Kit Skills

Este agente complementa y se integra con:

### Code Quality Skills
- **Clean Code Enforcer** (#18) - Clean code principles
- **Code Review Bot** (#21) - Automated review
- **SOLID Principles Guide** (#19) - Design principles

### Language-Specific Skills
- **JavaScript Vanilla Specialist** (#3) - JS conventions
- **TypeScript Type System Expert** (#4) - TS conventions
- **Python Best Practices Expert** (#50) - Python conventions
- **Modern C++ Expert** (#39) - C++ conventions
- **Java Enterprise Architect** (#44) - Java conventions

### Domain Skills
- **API Design Expert** (#7) - API naming conventions
- **Database Schema Designer** (#12) - DB naming conventions

---

## 🌐 Community Resources

### GitHub Style Guides
- **URL**: [https://github.com/topics/style-guide](https://github.com/topics/style-guide)
- **Tema**: Community style guides

### Stack Overflow
- **Tag**: [naming-conventions]
- **URL**: [https://stackoverflow.com/questions/tagged/naming-conventions](https://stackoverflow.com/questions/tagged/naming-conventions)

---

## 🛠️ IDE Extensions

### VS Code
- **ESLint**: Naming convention enforcement
- **SonarLint**: Real-time naming analysis
- **Better Comments**: Highlight naming TODOs
- **Code Spell Checker**: Catch typos in names

### JetBrains IDEs
- **SonarLint**: Naming inspections
- **CheckStyle-IDEA**: Java naming checks
- **Inspection Tools**: Built-in naming inspections

---

## 📊 Naming Convention Patterns

### Common Prefixes/Suffixes

```javascript
// Getters/Accessors
get...()
fetch...()
retrieve...()
find...()

// Setters/Mutators
set...()
update...()
modify...()
change...()

// Boolean checks
is...()
has...()
can...()
should...()
was...()
will...()

// Converters
to...()
from...()
as...()

// Validators
validate...()
check...()
verify...()

// Handlers
handle...()  // Use sparingly, be specific
on...()      // Event handlers
```

---

## 🎯 Domain-Driven Design Resources

### Ubiquitous Language
- **URL**: [https://martinfowler.com/bliki/UbiquitousLanguage.html](https://martinfowler.com/bliki/UbiquitousLanguage.html)
- **Tema**: Domain language in code

### Bounded Context
- **URL**: [https://martinfowler.com/bliki/BoundedContext.html](https://martinfowler.com/bliki/BoundedContext.html)
- **Tema**: Context-specific naming

---

**Versión:** 1.0.0 | **Última actualización:** 2025-10-24
