# Memory Leak Detector - Examples

> **Casos de uso reales de memory leaks y soluciones**

---

## 📋 Tabla de Contenidos

- [Ejemplo 1: Event Listeners en SPA](#ejemplo-1-event-listeners-en-spa)
- [Ejemplo 2: React useEffect Memory Leaks](#ejemplo-2-react-useeffect-memory-leaks)
- [Ejemplo 3: Detached DOM Nodes Cache](#ejemplo-3-detached-dom-nodes-cache)
- [Ejemplo 4: Closures y Large Data](#ejemplo-4-closures-y-large-data)
- [Ejemplo 5: Timers sin Cleanup](#ejemplo-5-timers-sin-cleanup)

---

## Ejemplo 1: Event Listeners en SPA

### Problema
```javascript
// ❌ MEMORY LEAK - Modal component
class Modal {
  constructor() {
    this.modal = null;
  }

  show() {
    // Create modal element
    this.modal = document.createElement('div');
    this.modal.className = 'modal';
    this.modal.innerHTML = `
      <div class="modal-content">
        <button class="close">×</button>
        <div class="modal-body">Modal content</div>
      </div>
    `;
    document.body.appendChild(this.modal);

    // Add event listeners
    const closeBtn = this.modal.querySelector('.close');
    closeBtn.addEventListener('click', () => this.hide());

    // ESC key listener
    document.addEventListener('keydown', (e) => {
      if (e.key === 'Escape') this.hide();
    });

    // Click outside listener
    this.modal.addEventListener('click', (e) => {
      if (e.target === this.modal) this.hide();
    });
  }

  hide() {
    if (this.modal) {
      this.modal.remove();
      this.modal = null;
      // ❌ PROBLEMA: Event listeners nunca se remueven!
      // - document keydown listener persiste
      // - Modal click listener persiste si hay referencias
    }
  }
}

// Usage:
const modal1 = new Modal();
modal1.show(); // Agrega listeners
modal1.hide(); // Remueve DOM pero NO listeners

const modal2 = new Modal();
modal2.show(); // Agrega MÁS listeners
// Ahora hay 2x document.keydown listeners activos!
```

**Problemas:**
- ❌ Event listeners no se remueven al ocultar modal
- ❌ Cada instancia agrega listeners permanentes
- ❌ Memory leak + performance degradation
- ❌ Multiple handlers ejecutándose

### Solución
```javascript
// ✅ FIXED - Cleanup apropiado
class Modal {
  constructor() {
    this.modal = null;
    this.handleKeydown = null;
    this.handleClickOutside = null;
    this.handleClose = null;
  }

  show() {
    // Create modal
    this.modal = document.createElement('div');
    this.modal.className = 'modal';
    this.modal.innerHTML = `
      <div class="modal-content">
        <button class="close">×</button>
        <div class="modal-body">Modal content</div>
      </div>
    `;
    document.body.appendChild(this.modal);

    // Store handler references for cleanup
    this.handleKeydown = (e) => {
      if (e.key === 'Escape') this.hide();
    };

    this.handleClickOutside = (e) => {
      if (e.target === this.modal) this.hide();
    };

    this.handleClose = () => this.hide();

    // Add listeners
    const closeBtn = this.modal.querySelector('.close');
    closeBtn.addEventListener('click', this.handleClose);
    document.addEventListener('keydown', this.handleKeydown);
    this.modal.addEventListener('click', this.handleClickOutside);
  }

  hide() {
    if (this.modal) {
      // ✅ Remove all listeners before removing DOM
      const closeBtn = this.modal.querySelector('.close');
      if (closeBtn) {
        closeBtn.removeEventListener('click', this.handleClose);
      }

      document.removeEventListener('keydown', this.handleKeydown);
      this.modal.removeEventListener('click', this.handleClickOutside);

      // Remove DOM
      this.modal.remove();
      this.modal = null;

      // Clear handler references
      this.handleKeydown = null;
      this.handleClickOutside = null;
      this.handleClose = null;
    }
  }

  // Cleanup method for complete destruction
  destroy() {
    this.hide();
  }
}
```

### Alternativa con AbortController
```javascript
// ✅ Modern approach con AbortController
class Modal {
  constructor() {
    this.modal = null;
    this.abortController = null;
  }

  show() {
    // Create abort controller
    this.abortController = new AbortController();
    const { signal } = this.abortController;

    // Create modal
    this.modal = document.createElement('div');
    this.modal.className = 'modal';
    this.modal.innerHTML = `
      <div class="modal-content">
        <button class="close">×</button>
        <div class="modal-body">Modal content</div>
      </div>
    `;
    document.body.appendChild(this.modal);

    // Add listeners with signal
    const closeBtn = this.modal.querySelector('.close');
    closeBtn.addEventListener('click', () => this.hide(), { signal });
    document.addEventListener('keydown', (e) => {
      if (e.key === 'Escape') this.hide();
    }, { signal });
    this.modal.addEventListener('click', (e) => {
      if (e.target === this.modal) this.hide();
    }, { signal });
  }

  hide() {
    if (this.modal) {
      // ✅ Remove ALL listeners at once
      this.abortController.abort();

      this.modal.remove();
      this.modal = null;
      this.abortController = null;
    }
  }
}
```

---

## Ejemplo 2: React useEffect Memory Leaks

### Problema
```javascript
// ❌ MEMORY LEAK - Multiple patterns

// 1. setState después de unmount
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data); // ❌ Puede ejecutarse después de unmount
      });
  }, [userId]);

  return <div>{user?.name}</div>;
}

// 2. Event listener sin cleanup
function WindowResize() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    // ❌ Missing cleanup!
  }, []);

  return <div>Width: {width}</div>;
}

// 3. Timer sin cleanup
function AutoRefresh() {
  const [data, setData] = useState(null);

  useEffect(() => {
    setInterval(() => {
      fetchData().then(setData);
    }, 5000);
    // ❌ Missing cleanup!
  }, []);

  return <div>{data}</div>;
}

// 4. WebSocket sin cleanup
function LiveFeed() {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const ws = new WebSocket('wss://api.example.com');

    ws.onmessage = (event) => {
      setMessages(prev => [...prev, event.data]);
    };
    // ❌ Missing cleanup!
  }, []);

  return <ul>{messages.map(m => <li key={m.id}>{m}</li>)}</ul>;
}
```

### Solución
```javascript
// ✅ FIXED - All patterns with cleanup

// 1. Cancelar fetch después de unmount
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let cancelled = false;

    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        if (!cancelled) { // ✅ Check antes de setState
          setUser(data);
        }
      });

    return () => {
      cancelled = true; // ✅ Cleanup
    };
  }, [userId]);

  return <div>{user?.name}</div>;
}

// Alternativa con AbortController
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();

    fetch(`/api/users/${userId}`, {
      signal: abortController.signal
    })
      .then(res => res.json())
      .then(data => setUser(data))
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err);
        }
      });

    return () => {
      abortController.abort(); // ✅ Cleanup
    };
  }, [userId]);

  return <div>{user?.name}</div>;
}

// 2. Event listener con cleanup
function WindowResize() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('resize', handleResize); // ✅ Cleanup
    };
  }, []);

  return <div>Width: {width}</div>;
}

// 3. Timer con cleanup
function AutoRefresh() {
  const [data, setData] = useState(null);

  useEffect(() => {
    const intervalId = setInterval(() => {
      fetchData().then(setData);
    }, 5000);

    return () => {
      clearInterval(intervalId); // ✅ Cleanup
    };
  }, []);

  return <div>{data}</div>;
}

// 4. WebSocket con cleanup
function LiveFeed() {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const ws = new WebSocket('wss://api.example.com');

    ws.onmessage = (event) => {
      setMessages(prev => [...prev, event.data]);
    };

    return () => {
      ws.close(); // ✅ Cleanup
    };
  }, []);

  return <ul>{messages.map(m => <li key={m.id}>{m}</li>)}</ul>;
}
```

---

## Ejemplo 3: Detached DOM Nodes Cache

### Problema
```javascript
// ❌ MEMORY LEAK - Detached DOM nodes en cache

class DataTable {
  constructor() {
    this.rowCache = {}; // ❌ Map regular
  }

  renderRow(data) {
    // Check cache
    if (this.rowCache[data.id]) {
      return this.rowCache[data.id];
    }

    // Create row
    const row = document.createElement('tr');
    row.innerHTML = `
      <td>${data.name}</td>
      <td>${data.email}</td>
      <td><button class="delete">Delete</button></td>
    `;

    // Cache row
    this.rowCache[data.id] = row; // ❌ Strong reference

    return row;
  }

  deleteRow(id) {
    const row = this.rowCache[id];
    if (row) {
      row.remove(); // ❌ Removed from DOM but still in cache!
      // rowCache[id] still holds reference
    }
  }

  renderTable(dataArray) {
    const tbody = document.querySelector('#data-table tbody');
    tbody.innerHTML = ''; // ❌ Clears DOM but cache persists

    dataArray.forEach(data => {
      const row = this.renderRow(data);
      tbody.appendChild(row);
    });
  }
}

// After many re-renders and deletes:
// rowCache contains hundreds of detached DOM nodes
// Memory leak!
```

**Chrome DevTools Detection:**
```
1. Memory → Heap Snapshot
2. Filter: "Detached"
3. See: HTMLTableRowElement (detached)
4. Retainers: rowCache object
```

### Solución
```javascript
// ✅ FIXED - WeakMap para detached nodes

class DataTable {
  constructor() {
    this.rowCache = new WeakMap(); // ✅ Weak references
    this.rowElements = new Map(); // For lookups by ID
  }

  renderRow(data) {
    // Create row
    const row = document.createElement('tr');
    row.innerHTML = `
      <td>${data.name}</td>
      <td>${data.email}</td>
      <td><button class="delete">Delete</button></td>
    `;

    // Cache with WeakMap
    this.rowCache.set(row, data); // ✅ Row can be GC'd
    this.rowElements.set(data.id, row);

    return row;
  }

  deleteRow(id) {
    const row = this.rowElements.get(id);
    if (row) {
      row.remove();

      // ✅ Clean up strong reference
      this.rowElements.delete(id);

      // WeakMap entry will be GC'd automatically
    }
  }

  renderTable(dataArray) {
    const tbody = document.querySelector('#data-table tbody');

    // ✅ Clear strong references
    this.rowElements.clear();

    tbody.innerHTML = '';

    dataArray.forEach(data => {
      const row = this.renderRow(data);
      tbody.appendChild(row);
    });
  }

  // ✅ Cleanup method
  destroy() {
    this.rowElements.clear();
    // WeakMap entries will be GC'd
  }
}
```

### Alternativa sin Cache
```javascript
// ✅ Alternative - No caching (simpler)
class DataTable {
  renderRow(data) {
    const row = document.createElement('tr');
    row.innerHTML = `
      <td>${data.name}</td>
      <td>${data.email}</td>
      <td><button class="delete">Delete</button></td>
    `;
    return row;
  }

  renderTable(dataArray) {
    const tbody = document.querySelector('#data-table tbody');
    tbody.innerHTML = '';

    dataArray.forEach(data => {
      const row = this.renderRow(data);
      tbody.appendChild(row);
    });
  }

  // No cache = no leak
}
```

---

## Ejemplo 4: Closures y Large Data

### Problema
```javascript
// ❌ MEMORY LEAK - Closure captura large data innecesariamente

function processLargeDataset() {
  // Large dataset (10MB+)
  const largeData = Array.from({ length: 1000000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`,
    description: `Description for item ${i}`,
    metadata: { /* large object */ }
  }));

  // Process data
  const results = largeData.map(item => item.id);

  // Return handler
  return {
    getCount: function() {
      // ❌ PROBLEMA: Closure mantiene referencia a largeData
      // aunque solo necesita results.length
      return results.length;
    },
    hasResults: function() {
      // ❌ Mismo problema
      return results.length > 0;
    }
  };
}

// Usage
const handlers = [];
for (let i = 0; i < 100; i++) {
  handlers.push(processLargeDataset());
  // ❌ Cada handler mantiene 10MB+ en memoria!
  // 100 handlers = 1GB+ memory leak
}
```

### Solución
```javascript
// ✅ FIXED - Solo capturar lo necesario

function processLargeDataset() {
  // Large dataset
  const largeData = Array.from({ length: 1000000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`,
    description: `Description for item ${i}`,
    metadata: { /* large object */ }
  }));

  // Process data
  const results = largeData.map(item => item.id);

  // ✅ Extract solo lo necesario
  const count = results.length;

  // ✅ No capturar largeData ni results
  return {
    getCount: function() {
      return count; // ✅ Solo number primitive
    },
    hasResults: function() {
      return count > 0; // ✅ Solo number primitive
    }
  };

  // largeData y results pueden ser GC'd
}

// Usage
const handlers = [];
for (let i = 0; i < 100; i++) {
  handlers.push(processLargeDataset());
  // ✅ Cada handler solo guarda un number
  // 100 handlers = minimal memory
}
```

---

## Ejemplo 5: Timers sin Cleanup

### Problema
```javascript
// ❌ MEMORY LEAK - Polling service sin cleanup

class DataPollingService {
  constructor(apiUrl) {
    this.apiUrl = apiUrl;
    this.data = null;
    this.intervalId = null; // ⚠️ Defined pero nunca limpiado apropiadamente
  }

  startPolling() {
    // Poll every 5 seconds
    this.intervalId = setInterval(async () => {
      try {
        const response = await fetch(this.apiUrl);
        this.data = await response.json();
        this.onData(this.data);
      } catch (error) {
        console.error('Polling error:', error);
      }
    }, 5000);
  }

  onData(data) {
    // Handle data
    console.log('Data received:', data);
  }

  // ❌ NO hay método stopPolling!
}

// Usage in React/Vue component
function Dashboard() {
  const [service] = useState(() => new DataPollingService('/api/data'));

  useEffect(() => {
    service.startPolling();
    // ❌ NO cleanup function!
  }, []);

  return <div>{/* render */}</div>;
}

// Cuando el componente unmounts:
// - Interval sigue ejecutándose
// - Continúa haciendo fetch requests
// - Service instance no puede ser GC'd
// - Memory leak + wasted API calls
```

### Solución
```javascript
// ✅ FIXED - Cleanup apropiado

class DataPollingService {
  constructor(apiUrl) {
    this.apiUrl = apiUrl;
    this.data = null;
    this.intervalId = null;
    this.isPolling = false;
  }

  startPolling() {
    if (this.isPolling) return; // Prevent multiple intervals

    this.isPolling = true;
    this.intervalId = setInterval(async () => {
      try {
        const response = await fetch(this.apiUrl);
        this.data = await response.json();
        this.onData(this.data);
      } catch (error) {
        console.error('Polling error:', error);
      }
    }, 5000);
  }

  stopPolling() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
      this.isPolling = false;
    }
  }

  onData(data) {
    console.log('Data received:', data);
  }

  // ✅ Cleanup method
  destroy() {
    this.stopPolling();
    this.data = null;
  }
}

// ✅ Usage with cleanup
function Dashboard() {
  const [service] = useState(() => new DataPollingService('/api/data'));

  useEffect(() => {
    service.startPolling();

    return () => {
      service.stopPolling(); // ✅ Cleanup
    };
  }, [service]);

  return <div>{/* render */}</div>;
}
```

### Alternativa con Custom Hook
```javascript
// ✅ Custom hook con cleanup automático

function usePolling(url, interval = 5000) {
  const [data, setData] = useState(null);

  useEffect(() => {
    let isActive = true;

    const poll = async () => {
      try {
        const response = await fetch(url);
        const result = await response.json();

        if (isActive) {
          setData(result);
        }
      } catch (error) {
        console.error('Polling error:', error);
      }
    };

    // Initial poll
    poll();

    // Start interval
    const intervalId = setInterval(poll, interval);

    return () => {
      isActive = false; // ✅ Prevent setState after unmount
      clearInterval(intervalId); // ✅ Clear interval
    };
  }, [url, interval]);

  return data;
}

// Usage
function Dashboard() {
  const data = usePolling('/api/data');

  return <div>{JSON.stringify(data)}</div>;
}
```

---

## 🎯 Chrome DevTools Detection Guide

### Heap Snapshot Comparison
```javascript
// 1. Open Chrome DevTools → Memory
// 2. Take "Heap snapshot" (Snapshot 1)
// 3. Perform action that may leak (e.g., open/close modal 10 times)
// 4. Take another "Heap snapshot" (Snapshot 2)
// 5. Select Snapshot 2
// 6. Change view from "Summary" to "Comparison"
// 7. Compare against Snapshot 1
// 8. Look for:
//    - Objects with high "# New" count
//    - Large "Size Delta"
//    - Detached DOM nodes
```

### Allocation Timeline
```javascript
// 1. Open Chrome DevTools → Memory
// 2. Select "Allocation instrumentation on timeline"
// 3. Click "Start"
// 4. Perform actions
// 5. Click "Stop"
// 6. Analyze timeline:
//    - Blue bars = allocations
//    - Gray bars = deallocations
//    - Persistent blue bars = potential leaks
```

---

**Versión:** 1.0.0 | **Última actualización:** 2025-10-24
