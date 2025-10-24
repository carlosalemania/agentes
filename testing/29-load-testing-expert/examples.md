# Load Testing Expert - Examples

---

## Example: Complete Load Test Suite

```javascript
// k6 - Multi-scenario test
import http from 'k6/http';
import { sleep, check } from 'k6';

export const options = {
  scenarios: {
    // Read-heavy workload
    readers: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '1m', target: 100 },
        { duration: '5m', target: 100 },
        { duration: '1m', target: 0 },
      ],
      exec: 'readScenario'
    },
    // Write workload
    writers: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '1m', target: 20 },
        { duration: '5m', target: 20 },
        { duration: '1m', target: 0 },
      ],
      exec: 'writeScenario'
    }
  },
  thresholds: {
    'http_req_duration{scenario:readers}': ['p(95)<500'],
    'http_req_duration{scenario:writers}': ['p(95)<1000'],
  }
};

export function readScenario() {
  http.get('https://api.example.com/items');
  sleep(1);
}

export function writeScenario() {
  http.post('https://api.example.com/items', JSON.stringify({
    name: 'Item'
  }));
  sleep(2);
}
```

---

**Versión:** 1.0.0
