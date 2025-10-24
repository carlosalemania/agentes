# Naming Convention Specialist - Checklist

> **Lista de verificación para code review y naming audit**

---

## 🔍 Naming Audit General

### Legibilidad
- [ ] ¿Los nombres son pronunciables?
- [ ] ¿Los nombres son buscables (no single letter)?
- [ ] ¿Se evitan abreviaciones crípticas?
- [ ] ¿El código se lee como prosa?
- [ ] ¿Los nombres autodocumentan el código?

### Revelación de Intención
- [ ] ¿Cada nombre revela su propósito?
- [ ] ¿Las funciones dicen QUÉ hacen (no CÓMO)?
- [ ] ¿Las variables describen el contenido?
- [ ] ¿Se evitan comentarios explicando nombres?

### Consistencia
- [ ] ¿Se sigue la convención del lenguaje?
- [ ] ¿Un concepto = una palabra en todo el proyecto?
- [ ] ¿Se usa consistentemente el mismo patrón?
- [ ] ¿No hay sinónimos para el mismo concepto?

### Contexto
- [ ] ¿Los nombres tienen el contexto adecuado?
- [ ] ¿Se evita contexto redundante?
- [ ] ¿Se usa el lenguaje del dominio?
- [ ] ¿Los nombres tienen el scope correcto?

---

## 📝 Convenciones por Lenguaje

### JavaScript/TypeScript
- [ ] ✅ Variables/funciones: `camelCase`
- [ ] ✅ Clases: `PascalCase`
- [ ] ✅ Constants: `SCREAMING_SNAKE_CASE`
- [ ] ✅ Private: `#field` o `_field`
- [ ] ✅ Interfaces: `PascalCase`
- [ ] ✅ Types: `PascalCase`
- [ ] ✅ Enums: `PascalCase` con valores `SCREAMING`

### Python
- [ ] ✅ Variables/funciones: `snake_case`
- [ ] ✅ Clases: `PascalCase`
- [ ] ✅ Constants: `SCREAMING_SNAKE_CASE`
- [ ] ✅ Private: `_protected` o `__private`
- [ ] ✅ Modules: `snake_case`

### Java
- [ ] ✅ Variables/métodos: `camelCase`
- [ ] ✅ Clases: `PascalCase`
- [ ] ✅ Interfaces: `PascalCase` (adjectives)
- [ ] ✅ Constants: `SCREAMING_SNAKE_CASE`
- [ ] ✅ Packages: `lowercase`

### C++
- [ ] ✅ Variables: `snake_case` o `camelCase` (consistente)
- [ ] ✅ Functions: `snake_case` o `CamelCase`
- [ ] ✅ Classes: `PascalCase`
- [ ] ✅ Constants: `kConstantName` o `SCREAMING_SNAKE_CASE`
- [ ] ✅ Macros: `SCREAMING_SNAKE_CASE`
- [ ] ✅ Namespaces: `lowercase`

### CSS/SCSS
- [ ] ✅ Classes: `kebab-case`
- [ ] ✅ IDs: `kebab-case`
- [ ] ✅ BEM: `block__element--modifier`
- [ ] ✅ Variables: `kebab-case` o `$variable-name`

### SQL
- [ ] ✅ Tables: `snake_case`, plural
- [ ] ✅ Columns: `snake_case`
- [ ] ✅ Indexes: `idx_` prefix
- [ ] ✅ Constraints: `fk_`, `pk_`, `uk_` prefix

---

## 🔤 Variables

### Nombres
- [ ] ¿Son sustantivos o frases nominales?
- [ ] ¿Describen el contenido, no el tipo?
- [ ] ¿Evitan prefijos de tipo (Hungarian)?
- [ ] ¿No tienen números (`user1`, `temp2`)?
- [ ] ¿Son específicos, no genéricos (`data`, `temp`)?

### Booleans
- [ ] ¿Tienen prefijo `is/has/can/should/was/will`?
- [ ] ¿Son afirmativos (no `notReady`, `notValid`)?
- [ ] ¿El nombre hace obvio que es boolean?

### Collections
- [ ] ¿Plural para arrays/lists (`users`, `orders`)?
- [ ] ¿Nombres descriptivos (`activeUsers`, no `arr`)?

### Constants
- [ ] ¿SCREAMING_SNAKE_CASE?
- [ ] ¿Nombres semánticos, no magic numbers?
- [ ] ¿Agrupados lógicamente?

---

## 🔧 Funciones/Métodos

### Naming
- [ ] ¿Empiezan con verbo?
- [ ] ¿Revelan qué hacen, no cómo?
- [ ] ¿Nombres específicos (no `process`, `handle`, `manage`)?
- [ ] ¿Consistentes con naming de clase?

### Patrones
- [ ] ✅ CRUD: `create`, `get`, `update`, `delete`
- [ ] ✅ Boolean: `is`, `has`, `can`, `should`
- [ ] ✅ Conversión: `toString`, `toJSON`, `fromX`
- [ ] ✅ Factory: `create`, `build`, `make`
- [ ] ✅ Business: verbos del dominio

### Parámetros
- [ ] ¿Nombres descriptivos (no `x`, `y`, `data`)?
- [ ] ¿Consistentes con el cuerpo de la función?
- [ ] ¿Orden lógico (más importantes primero)?

---

## 🏗️ Clases

### Nombres
- [ ] ¿Son sustantivos o frases nominales?
- [ ] ¿PascalCase en la mayoría de lenguajes?
- [ ] ¿Singulares (no plural)?
- [ ] ¿Específicos (no `Manager`, `Handler`, `Processor`)?
- [ ] ¿Responsabilidad clara en el nombre?

### Anti-Patterns
- [ ] ❌ No hay clases `Manager`
- [ ] ❌ No hay clases `Handler`
- [ ] ❌ No hay clases `Processor`
- [ ] ❌ No hay clases `Data`, `Info`, `Object` suffix
- [ ] ❌ No hay clases con responsabilidades vagas

### Interfaces/Protocols
- [ ] ¿PascalCase?
- [ ] ¿Adjetivos (`Readable`, `Serializable`) o sustantivos?
- [ ] ¿Prefijo `I` solo si es convención del proyecto?

---

## 🎯 Domain-Driven Design

### Ubiquitous Language
- [ ] ¿Se usa el lenguaje del negocio?
- [ ] ¿Los nombres coinciden con términos del dominio?
- [ ] ¿El equipo de negocio entiende los nombres?
- [ ] ¿Consistente en todo el bounded context?

### Entities y Value Objects
- [ ] ¿Nombres del dominio (no técnicos)?
- [ ] ¿Entities: identidad (User, Order)?
- [ ] ¿Value Objects: valor (Money, Address)?

### Services
- [ ] ¿Responsabilidad específica en el nombre?
- [ ] ¿Verbos del dominio?
- [ ] ¿No abusan de "Service" suffix?

### Aggregates
- [ ] ¿Nombre del aggregate root claro?
- [ ] ¿Refleja concepto del negocio?

---

## 🚫 Anti-Patterns

### Evitar
- [ ] ❌ No single letter (excepto `i`, `j`, `k` en loops)
- [ ] ❌ No números en nombres (`user1`, `temp2`)
- [ ] ❌ No noise words (`UserData` vs `UserInfo` vs `User`)
- [ ] ❌ No Hungarian notation (`strName`, `m_description`)
- [ ] ❌ No abreviaciones crípticas (`usrMgr`, `calcTtl`)
- [ ] ❌ No nombres cute/humor (`whack()`, `kill()`)
- [ ] ❌ No puns/wordplay
- [ ] ❌ No encodings innecesarios

### Manager/Handler/Processor
- [ ] ❌ `*Manager` → Responsabilidad específica
- [ ] ❌ `*Handler` → Responsabilidad específica
- [ ] ❌ `*Processor` → Responsabilidad específica
- [ ] ❌ `*Helper` → Responsabilidad específica
- [ ] ❌ `*Util` → Responsabilidad específica

---

## 🔄 Refactoring Checklist

### Antes de Renombrar
- [ ] ¿Identifiqué todos los usos?
- [ ] ¿Tengo tests que me protejan?
- [ ] ¿El nuevo nombre es mejor que el actual?
- [ ] ¿Es consistente con el resto del código?

### Durante Refactoring
- [ ] ¿Uso herramienta de rename (F2, refactor)?
- [ ] ¿Renombro en una sola operación?
- [ ] ¿Actualizo comentarios relacionados?
- [ ] ¿Actualizo documentación?

### Después de Renombrar
- [ ] ¿Los tests siguen pasando?
- [ ] ¿El código es más legible?
- [ ] ¿No rompí referencias externas?
- [ ] ¿Hice commit del rename por separado?

---

## 📊 Code Review Checklist

### Variables
- [ ] ¿Nombres pronunciables y buscables?
- [ ] ¿Revelan intención clara?
- [ ] ¿Booleans con prefijo is/has/can/should?
- [ ] ¿Constants en SCREAMING_SNAKE_CASE?
- [ ] ¿Sin single letter (excepto counters)?

### Funciones
- [ ] ¿Empiezan con verbo?
- [ ] ¿Nombres específicos (no process/handle)?
- [ ] ¿Parámetros descriptivos?
- [ ] ¿Consistentes con responsabilidad?

### Clases
- [ ] ¿Sustantivos en PascalCase?
- [ ] ¿Responsabilidad clara?
- [ ] ¿No abusan de Manager/Handler?
- [ ] ¿Nombres del dominio?

### Consistencia
- [ ] ¿Sigue convención del lenguaje?
- [ ] ¿Un concepto = una palabra?
- [ ] ¿Consistente en el proyecto?
- [ ] ¿Usa ubiquitous language?

---

## ✅ Pre-Commit Checklist

### Quick Check
- [ ] No hay variables single letter (excepto loops)
- [ ] No hay funciones `process()`, `handle()`, `doStuff()`
- [ ] No hay clases `*Manager`, `*Handler`, `*Processor`
- [ ] Booleans tienen prefijo is/has/can/should
- [ ] Constants en SCREAMING_SNAKE_CASE
- [ ] Sigo la convención del lenguaje
- [ ] Nombres autodocumentan el código
- [ ] Uso lenguaje del dominio

### Linting
- [ ] ESLint/Pylint naming rules pasan
- [ ] SonarQube analysis OK
- [ ] No warnings de naming conventions

---

## 🎓 Team Guidelines Checklist

### Documentación
- [ ] ¿Hay guía de naming del proyecto?
- [ ] ¿Ejemplos de buenos nombres?
- [ ] ¿Anti-patterns documentados?
- [ ] ¿Convenciones por lenguaje claras?

### Enforcement
- [ ] ¿Linters configurados?
- [ ] ¿Pre-commit hooks activos?
- [ ] ¿Code review incluye naming?
- [ ] ¿Team training realizado?

### Ubiquitous Language
- [ ] ¿Glosario de términos del dominio?
- [ ] ¿Sincronizado con equipo de negocio?
- [ ] ¿Actualizado regularmente?

---

**Versión:** 1.0.0 | **Última actualización:** 2025-10-24
