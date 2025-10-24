# Memory Leak Detector - Checklist

> **Lista de verificación para detectar y prevenir memory leaks**

---

## 🔍 Memory Leak Audit

### Event Listeners
- [ ] ¿Todos los event listeners tienen cleanup?
- [ ] ¿Se remueven listeners en componentWillUnmount/useEffect cleanup?
- [ ] ¿Se usan AbortController para múltiples listeners?
- [ ] ¿Listeners globales (window, document) se limpian?
- [ ] ¿No hay listeners duplicados?

### Timers e Intervals
- [ ] ¿Todos los setInterval tienen clearInterval?
- [ ] ¿Todos los setTimeout tienen clearTimeout?
- [ ] ¿Se guardan IDs para cleanup posterior?
- [ ] ¿Timers se limpian en unmount/destroy?
- [ ] ¿requestAnimationFrame tiene cancelAnimationFrame?

### Async Operations
- [ ] ¿Fetch requests se cancelan con AbortController?
- [ ] ¿setState tiene check de component mounted?
- [ ] ¿Promises tienen flag de cancellation?
- [ ] ¿Async iterators se cierran apropiadamente?

### DOM References
- [ ] ¿Cache de DOM nodes usa WeakMap?
- [ ] ¿Referencias a DOM se limpian al remover elementos?
- [ ] ¿No hay detached DOM nodes en cache?
- [ ] ¿Event handlers no retienen DOM innecesariamente?

### WebSockets y Connections
- [ ] ¿WebSocket connections se cierran en cleanup?
- [ ] ¿EventSource connections se cierran?
- [ ] ¿SSE connections tienen cleanup?
- [ ] ¿Database connections se cierran?

---

## 📝 React/Vue Specific

### React Hooks
- [ ] ¿useEffect retorna cleanup function?
- [ ] ¿useEffect cleanup remueve listeners?
- [ ] ¿useEffect cleanup cancela async?
- [ ] ¿Custom hooks implementan cleanup?
- [ ] ¿No hay setState después de unmount?

### React Class Components
- [ ] ¿componentWillUnmount limpia listeners?
- [ ] ¿componentWillUnmount cancela async?
- [ ] ¿componentWillUnmount limpia timers?
- [ ] ¿Refs se limpian apropiadamente?

### Vue Lifecycle
- [ ] ¿beforeUnmount limpia listeners?
- [ ] ¿beforeUnmount cancela async?
- [ ] ¿beforeUnmount limpia timers?
- [ ] ¿Watch cleanup functions implementadas?

---

## 🔤 Closures

### Closure Patterns
- [ ] ¿Closures solo capturan lo necesario?
- [ ] ¿No capturan datos grandes innecesariamente?
- [ ] ¿Variables temporales están fuera de closure?
- [ ] ¿Closures en loops usan IIFE o let?

### Large Data
- [ ] ¿Large arrays se limpian cuando no se usan?
- [ ] ¿Buffers se liberan después de uso?
- [ ] ¿Images/videos se limpian del DOM?
- [ ] ¿Cache tiene límite de tamaño?

---

## 🏗️ Data Structures

### Collections
- [ ] ¿Map/Set se limpian cuando no se usan?
- [ ] ¿WeakMap/WeakSet para referencias temporales?
- [ ] ¿Arrays grandes se null cuando terminan?
- [ ] ¿Cache tiene TTL o eviction policy?

### Cache
- [ ] ¿Cache tiene tamaño máximo?
- [ ] ¿Cache usa LRU o similar?
- [ ] ¿Cache entries se invalidan?
- [ ] ¿WeakMap para object caching?

### Global State
- [ ] ¿Redux/Vuex state se limpia?
- [ ] ¿Global variables se minimizan?
- [ ] ¿Window properties se limpian?
- [ ] ¿Module state se resetea cuando es necesario?

---

## 🚫 Common Anti-Patterns

### Event Listeners
- [ ] ❌ No: addEventListener sin removeEventListener
- [ ] ❌ No: Inline functions en addEventListener
- [ ] ❌ No: Múltiples listeners del mismo tipo sin cleanup
- [ ] ❌ No: Global listeners sin scope limitado

### Timers
- [ ] ❌ No: setInterval sin clearInterval
- [ ] ❌ No: setTimeout recursivo sin control
- [ ] ❌ No: Multiple intervals sin tracking
- [ ] ❌ No: Timers en loops sin cleanup

### References
- [ ] ❌ No: DOM nodes en variables globales
- [ ] ❌ No: Large objects en closures
- [ ] ❌ No: Circular references sin cleanup
- [ ] ❌ No: Cache infinito sin eviction

---

## 🔄 Testing Checklist

### Manual Testing
- [ ] ¿Heap snapshot antes/después muestra growth?
- [ ] ¿Allocation timeline muestra leaks?
- [ ] ¿Detached DOM nodes detectados?
- [ ] ¿Memory profiler muestra picos?

### Automated Testing
- [ ] ¿Tests de memory leak en CI/CD?
- [ ] ¿Memory budget enforcement?
- [ ] ¿Performance regression detection?
- [ ] ¿Automated heap snapshot comparison?

### Chrome DevTools
- [ ] ¿Memory tab - Heap snapshot?
- [ ] ¿Memory tab - Allocation timeline?
- [ ] ¿Performance tab - Memory checkbox?
- [ ] ¿Filter "Detached" en snapshots?

---

## 📊 Memory Profiling

### Before Testing
- [ ] Clear browser cache y reload
- [ ] Close other tabs
- [ ] Disable extensions
- [ ] Use Incognito mode

### During Testing
- [ ] Take initial heap snapshot
- [ ] Perform action múltiples veces
- [ ] Force garbage collection (DevTools)
- [ ] Take final heap snapshot
- [ ] Compare snapshots

### Analysis
- [ ] ¿Size delta es aceptable?
- [ ] ¿# New objects es razonable?
- [ ] ¿Hay detached DOM nodes?
- [ ] ¿Retaining tree muestra leaks?

---

## ✅ Pre-Deploy Checklist

### Code Review
- [ ] Todos los listeners tienen cleanup
- [ ] Todos los timers tienen cleanup
- [ ] Async operations se cancelan
- [ ] DOM references se limpian
- [ ] No hay closures con large data

### Performance Testing
- [ ] Memory profiling completado
- [ ] No memory leaks detectados
- [ ] Memory growth es acceptable
- [ ] GC se ejecuta apropiadamente

### Monitoring
- [ ] Memory metrics en producción
- [ ] Alertas de high memory usage
- [ ] Error tracking para out-of-memory
- [ ] Performance monitoring habilitado

---

## 🎓 Framework-Specific Checks

### React
- [ ] ✅ useEffect cleanup functions
- [ ] ✅ componentWillUnmount cleanup
- [ ] ✅ No setState after unmount
- [ ] ✅ Custom hooks con cleanup
- [ ] ✅ Event handlers no retienen components

### Vue
- [ ] ✅ beforeUnmount cleanup
- [ ] ✅ Watch cleanup
- [ ] ✅ Event bus listeners removed
- [ ] ✅ Global event listeners cleaned
- [ ] ✅ Refs cleared

### Angular
- [ ] ✅ ngOnDestroy cleanup
- [ ] ✅ Subscriptions unsubscribed
- [ ] ✅ Event listeners removed
- [ ] ✅ Timers cleared
- [ ] ✅ takeUntil pattern usado

---

## 🛠️ Tools Checklist

### Chrome DevTools
- [ ] Memory profiler usado
- [ ] Heap snapshots comparados
- [ ] Allocation timeline analizado
- [ ] Detached nodes identificados

### Node.js
- [ ] --inspect usado
- [ ] Heap snapshots exportados
- [ ] v8.writeHeapSnapshot() utilizado
- [ ] Memory usage monitoreado

### Third-Party Tools
- [ ] Clinic.js heap profiler
- [ ] memlab (Meta)
- [ ] Lighthouse memory audits
- [ ] Performance budgets

---

## 📈 Memory Metrics

### Acceptable Ranges
- [ ] Initial load: < 50MB
- [ ] After interaction: < 100MB
- [ ] Growth rate: < 10MB/minute
- [ ] GC frequency: Every 30s-60s

### Warning Signs
- [ ] ❌ Memory growth sin plateau
- [ ] ❌ Detached DOM nodes > 50
- [ ] ❌ Event listeners duplicados
- [ ] ❌ Large closures > 10MB
- [ ] ❌ Cache size > 100MB

---

## 🔍 Debugging Workflow

### Step 1: Reproduce
- [ ] Identify action que causa leak
- [ ] Repetir acción múltiples veces
- [ ] Observar memory growth

### Step 2: Profile
- [ ] Take heap snapshots
- [ ] Use allocation timeline
- [ ] Filter detached nodes
- [ ] Analyze retaining paths

### Step 3: Identify
- [ ] Encontrar retaining objects
- [ ] Identificar código responsable
- [ ] Determinar tipo de leak

### Step 4: Fix
- [ ] Implementar cleanup
- [ ] Add null references
- [ ] Use WeakMap/WeakSet
- [ ] Test fix con profiler

### Step 5: Verify
- [ ] Re-test con profiler
- [ ] Confirm memory plateaus
- [ ] No detached nodes
- [ ] Growth rate acceptable

---

**Versión:** 1.0.0 | **Última actualización:** 2025-10-24
