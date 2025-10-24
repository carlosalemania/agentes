# Memory Leak Detector - Resources

> **Referencias al Dev-Kit padre y recursos externos**

---

## 📚 Referencias al Dev-Kit Padre

### Documentación Core
Este agente se fundamenta en los siguientes documentos del dev-kit:

#### [PERFORMANCE-OPTIMIZATION.md](../../../docs/PERFORMANCE-OPTIMIZATION.md)
- **Memory Management**: Strategies para optimizar memoria
- **Resource Cleanup**: Patrones de liberación de recursos
- **Profiling**: Técnicas de memory profiling

**Aplicación en Memory Leak Detection:**
```javascript
// Performance-aware cleanup
class Component {
  constructor() {
    this.cache = new WeakMap(); // ✅ From PERFORMANCE-OPTIMIZATION.md
    this.listeners = new Map();
  }

  destroy() {
    // Cleanup siguiendo patterns del doc
    this.listeners.forEach((handler, element) => {
      element.removeEventListener('click', handler);
    });
    this.listeners.clear();
  }
}
```

#### [CODE-QUALITY-METRICS.md](../../../docs/CODE-QUALITY-METRICS.md)
- **Resource Management Metrics**: Tracking de resource lifecycle
- **Memory Efficiency**: Métricas de uso de memoria

#### [FAULT-TOLERANCE-GUIDE.md](../../../docs/FAULT-TOLERANCE-GUIDE.md)
- **Resource Cleanup**: Graceful cleanup en error scenarios
- **Defensive Programming**: Prevent leaks en edge cases

---

## 🛠️ Herramientas

### Chrome DevTools

#### Memory Profiler
- **URL**: Built-in Chrome DevTools
- **Features**:
  - Heap snapshots
  - Allocation timeline
  - Allocation sampling
  - Detached DOM node detection
- **Shortcuts**:
  - `Cmd/Ctrl + Shift + I` → Memory tab
  - Force GC: Click trash icon
  - Take snapshot: "Take snapshot" button

**Usage:**
```javascript
// 1. Open DevTools → Memory
// 2. Select "Heap snapshot"
// 3. Click "Take snapshot"
// 4. Perform actions
// 5. Take another snapshot
// 6. Compare in "Comparison" view
```

#### Performance Tab
- **Memory Checkbox**: Track memory during recording
- **Analyze**: Memory growth patterns
- **Timeline**: Visual memory spikes

### Node.js Memory Tools

#### Built-in Inspector
```bash
# Start Node.js with inspector
node --inspect app.js

# Or with --inspect-brk to pause at start
node --inspect-brk app.js

# Connect Chrome DevTools
# Navigate to: chrome://inspect
```

#### Heap Snapshots
```javascript
const v8 = require('v8');
const fs = require('fs');

// Write heap snapshot
const snapshot = v8.writeHeapSnapshot();
console.log('Snapshot written to:', snapshot);

// Or custom filename
const filename = `heap-${Date.now()}.heapsnapshot`;
const snapshotStream = v8.writeHeapSnapshot(filename);
```

#### Process Memory Usage
```javascript
const used = process.memoryUsage();

console.log({
  rss: `${Math.round(used.rss / 1024 / 1024)}MB`,       // Resident Set Size
  heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)}MB`,
  heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)}MB`,
  external: `${Math.round(used.external / 1024 / 1024)}MB`,
  arrayBuffers: `${Math.round(used.arrayBuffers / 1024 / 1024)}MB`
});
```

### Third-Party Tools

#### Clinic.js Heap Profiler
- **URL**: [https://clinicjs.org/heapprofiler/](https://clinicjs.org/heapprofiler/)
- **Install**: `npm install -g clinic`
- **Usage**:
```bash
# Profile heap
clinic heapprofiler -- node app.js

# Genera HTML report con análisis
```

#### memlab (Meta)
- **URL**: [https://facebook.github.io/memlab/](https://facebook.github.io/memlab/)
- **Features**:
  - Automated leak detection
  - Heap analysis
  - Leak patterns identification
- **Install**: `npm install -g memlab`
- **Usage**:
```bash
# Run leak detection
memlab run --scenario my-scenario.js

# Find leaks
memlab find-leaks
```

#### Lighthouse
- **URL**: Built-in Chrome / [https://developers.google.com/web/tools/lighthouse](https://developers.google.com/web/tools/lighthouse)
- **Memory Audits**: Performance audits incluyen memory
- **Usage**: DevTools → Lighthouse → Run audit

---

## 📖 Learning Resources

### Official Documentation

#### MDN - Memory Management
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)
- **Topics**:
  - Garbage collection
  - Mark-and-sweep algorithm
  - Reference-counting
  - Memory leaks

#### Chrome DevTools - Memory Profiling
- **URL**: [https://developer.chrome.com/docs/devtools/memory-problems/](https://developer.chrome.com/docs/devtools/memory-problems/)
- **Topics**:
  - Heap snapshots
  - Allocation timeline
  - Finding detached nodes
  - Understanding retainers

#### Node.js - Memory
- **URL**: [https://nodejs.org/en/docs/guides/simple-profiling/](https://nodejs.org/en/docs/guides/simple-profiling/)
- **Topics**:
  - V8 heap
  - Memory profiling
  - Heap snapshots
  - Memory leaks debugging

### Articles

#### Finding and Fixing Memory Leaks
- **URL**: [https://web.dev/fix-memory-problems/](https://web.dev/fix-memory-problems/)
- **Author**: Addy Osmani
- **Topics**: Practical debugging techniques

#### JavaScript Memory Leaks
- **URL**: [https://auth0.com/blog/four-types-of-leaks-in-your-javascript-code-and-how-to-get-rid-of-them/](https://auth0.com/blog/four-types-of-leaks-in-your-javascript-code-and-how-to-get-rid-of-them/)
- **Topics**: 4 common leak patterns

#### Memory Management in React
- **URL**: [https://felixgerschau.com/react-memory-leaks/](https://felixgerschau.com/react-memory-leaks/)
- **Topics**: React-specific leaks

### Videos

#### Chrome DevTools - Memory Profiling
- **Platform**: YouTube - Chrome Developers
- **Topics**: Complete DevTools walkthrough

#### Finding Memory Leaks (Jake Archibald)
- **Platform**: YouTube
- **Topics**: Practical debugging session

---

## 📚 Libros

### High Performance JavaScript
- **Autor**: Nicholas C. Zakas
- **Capítulo**: Memory Management
- **Topics**: Garbage collection, memory optimization

### JavaScript: The Good Parts
- **Autor**: Douglas Crockford
- **Topics**: Memory-efficient patterns

---

## 🔗 Related Dev-Kit Skills

Este agente complementa y se integra con:

### Performance Skills
- **Web Performance Optimizer** (#6) - General performance
- **JavaScript Performance Tuner** (#22) - JS optimization
- **React Performance Optimizer** (#24) - React-specific
- **Bundle Size Analyzer** (#25) - Asset optimization

### Code Quality Skills
- **Clean Code Enforcer** (#18) - Clean patterns
- **Code Review Bot** (#21) - Automated review

### Testing Skills
- **Performance Test Architect** (#29) - Performance testing
- **E2E Test Designer** (#28) - Integration testing

---

## 🌐 Browser Support

### Modern APIs

#### WeakMap/WeakSet
```javascript
// Support: All modern browsers
const cache = new WeakMap();

// Polyfill: No polyfill needed (feature detection)
if (typeof WeakMap === 'undefined') {
  // Fallback to regular Map (with manual cleanup)
}
```

#### AbortController
```javascript
// Support: Chrome 66+, Firefox 57+, Safari 12.1+
const controller = new AbortController();

// Polyfill:
// npm install abortcontroller-polyfill
import 'abortcontroller-polyfill/dist/abortcontroller-polyfill-only';
```

---

## 🎯 Memory Leak Patterns

### Common Patterns

#### Pattern 1: Event Listeners
```javascript
// Leak Pattern
element.addEventListener('click', handler);
// Missing: removeEventListener

// Prevention
const controller = new AbortController();
element.addEventListener('click', handler, { signal: controller.signal });
controller.abort(); // Cleanup
```

#### Pattern 2: Timers
```javascript
// Leak Pattern
setInterval(() => {}, 1000);
// Missing: clearInterval

// Prevention
const id = setInterval(() => {}, 1000);
clearInterval(id); // Cleanup
```

#### Pattern 3: Closures
```javascript
// Leak Pattern
function create() {
  const large = new Array(1000000);
  return () => console.log('hi'); // Captures large
}

// Prevention
function create() {
  const large = new Array(1000000);
  const result = processLarge(large);
  return () => console.log(result); // Only captures result
}
```

#### Pattern 4: DOM References
```javascript
// Leak Pattern
const cache = {};
cache[id] = document.getElementById(id);
// DOM node removed but cache persists

// Prevention
const cache = new WeakMap();
cache.set(element, data);
// Auto GC when element removed
```

---

## 📊 Benchmarking Tools

### JavaScript Benchmarking
- **Benchmark.js**: [https://benchmarkjs.com/](https://benchmarkjs.com/)
- **jsperf**: [https://jsperf.com/](https://jsperf.com/)

### Memory Benchmarks
```javascript
// Before
const before = process.memoryUsage().heapUsed;

// Operation
for (let i = 0; i < 10000; i++) {
  // Your code
}

// After
const after = process.memoryUsage().heapUsed;
console.log(`Memory used: ${(after - before) / 1024 / 1024}MB`);
```

---

## 🛡️ Best Practices

### Lifecycle Management
```javascript
class Component {
  constructor() {
    this.cleanup = [];
  }

  addListener(element, event, handler) {
    element.addEventListener(event, handler);
    this.cleanup.push(() => {
      element.removeEventListener(event, handler);
    });
  }

  destroy() {
    this.cleanup.forEach(fn => fn());
    this.cleanup = [];
  }
}
```

### Resource Pooling
```javascript
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.available = [];
    this.inUse = new Set();

    for (let i = 0; i < initialSize; i++) {
      this.available.push(this.createFn());
    }
  }

  acquire() {
    let obj = this.available.pop();
    if (!obj) {
      obj = this.createFn();
    }
    this.inUse.add(obj);
    return obj;
  }

  release(obj) {
    if (this.inUse.has(obj)) {
      this.inUse.delete(obj);
      this.resetFn(obj);
      this.available.push(obj);
    }
  }
}
```

---

**Versión:** 1.0.0 | **Última actualización:** 2025-10-24
