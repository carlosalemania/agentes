# Performance Profiling - Examples

---

## Example 1: Complete Node.js Profiling Setup

```javascript
// profiler.js
const v8Profiler = require('v8-profiler-next');
const fs = require('fs');

class Profiler {
  constructor() {
    this.profiling = false;
    this.profileName = null;
  }

  startCPUProfile(name = 'profile') {
    if (this.profiling) {
      console.warn('Already profiling');
      return;
    }

    this.profileName = `${name}-${Date.now()}`;
    v8Profiler.startProfiling(this.profileName, true);
    this.profiling = true;

    console.log(`Started CPU profiling: ${this.profileName}`);
  }

  stopCPUProfile() {
    if (!this.profiling) {
      console.warn('Not currently profiling');
      return;
    }

    const profile = v8Profiler.stopProfiling(this.profileName);

    profile.export((error, result) => {
      const filename = `${this.profileName}.cpuprofile`;
      fs.writeFileSync(filename, result);
      profile.delete();

      console.log(`CPU profile saved: ${filename}`);
      console.log(`Open in Chrome DevTools > Performance > Load Profile`);
    });

    this.profiling = false;
  }

  takeHeapSnapshot(name = 'snapshot') {
    const filename = `${name}-${Date.now()}.heapsnapshot`;
    v8.writeHeapSnapshot(filename);

    console.log(`Heap snapshot saved: ${filename}`);
    console.log(`Open in Chrome DevTools > Memory > Load Snapshot`);

    return filename;
  }

  compareHeapSnapshots(before, after) {
    // Load and compare snapshots programmatically
    const beforeData = JSON.parse(fs.readFileSync(before));
    const afterData = JSON.parse(fs.readFileSync(after));

    // Simple analysis
    console.log('Heap Comparison:');
    console.log(`Before: ${beforeData.snapshot.node_count} nodes`);
    console.log(`After: ${afterData.snapshot.node_count} nodes`);
    console.log(`Difference: ${afterData.snapshot.node_count - beforeData.snapshot.node_count} nodes`);
  }
}

module.exports = new Profiler();

// Usage in app
const profiler = require('./profiler');

// Start profiling
profiler.startCPUProfile('api-load-test');

// ... run code to profile ...
await loadTest();

// Stop profiling
profiler.stopCPUProfile();
```

---

## Example 2: Express Middleware with Detailed Metrics

```javascript
// performance-middleware.js
class PerformanceTracker {
  constructor() {
    this.routes = new Map();
  }

  middleware() {
    return (req, res, next) => {
      const start = process.hrtime.bigint();
      const route = `${req.method} ${req.route?.path || req.path}`;

      res.on('finish', () => {
        const end = process.hrtime.bigint();
        const durationNs = end - start;
        const durationMs = Number(durationNs) / 1_000_000;

        this.recordMetric(route, {
          duration: durationMs,
          statusCode: res.statusCode,
          contentLength: res.get('Content-Length'),
        });
      });

      next();
    };
  }

  recordMetric(route, data) {
    if (!this.routes.has(route)) {
      this.routes.set(route, {
        count: 0,
        errors: 0,
        durations: [],
        totalDuration: 0,
      });
    }

    const metrics = this.routes.get(route);
    metrics.count++;
    metrics.totalDuration += data.duration;
    metrics.durations.push(data.duration);

    if (data.statusCode >= 400) {
      metrics.errors++;
    }

    // Keep only last 100 durations
    if (metrics.durations.length > 100) {
      metrics.durations.shift();
    }

    // Alert on slow requests
    if (data.duration > 1000) {
      console.warn(`⚠️ Slow request: ${route} took ${data.duration.toFixed(2)}ms`);
    }
  }

  getMetrics() {
    const result = {};

    for (const [route, metrics] of this.routes) {
      const durations = [...metrics.durations].sort((a, b) => a - b);

      result[route] = {
        count: metrics.count,
        errors: metrics.errors,
        errorRate: ((metrics.errors / metrics.count) * 100).toFixed(2) + '%',
        avgDuration: (metrics.totalDuration / metrics.count).toFixed(2) + ' ms',
        p50: durations[Math.floor(durations.length * 0.5)]?.toFixed(2) + ' ms',
        p95: durations[Math.floor(durations.length * 0.95)]?.toFixed(2) + ' ms',
        p99: durations[Math.floor(durations.length * 0.99)]?.toFixed(2) + ' ms',
      };
    }

    return result;
  }

  reset() {
    this.routes.clear();
  }
}

const tracker = new PerformanceTracker();

module.exports = tracker;

// app.js
const express = require('express');
const tracker = require('./performance-middleware');

const app = express();

app.use(tracker.middleware());

// ... your routes ...

// Metrics endpoint
app.get('/metrics', (req, res) => {
  res.json(tracker.getMetrics());
});

// Reset endpoint
app.post('/metrics/reset', (req, res) => {
  tracker.reset();
  res.json({ message: 'Metrics reset' });
});
```

---

## Example 3: Python Performance Profiler

```python
# profiler.py
import cProfile
import pstats
import io
import time
from functools import wraps
from typing import Callable, Any

class PerformanceProfiler:
    def __init__(self):
        self.profiles = {}
        self.timings = {}

    def profile(self, func: Callable) -> Callable:
        """Decorator to profile a function"""
        @wraps(func)
        def wrapper(*args, **kwargs):
            profiler = cProfile.Profile()
            profiler.enable()

            start_time = time.perf_counter()
            result = func(*args, **kwargs)
            end_time = time.perf_counter()

            profiler.disable()

            # Store timing
            func_name = func.__name__
            duration = (end_time - start_time) * 1000  # ms

            if func_name not in self.timings:
                self.timings[func_name] = []

            self.timings[func_name].append(duration)

            # Store profile
            self.profiles[func_name] = profiler

            print(f"{func_name} took {duration:.2f}ms")

            return result

        return wrapper

    def print_stats(self, func_name: str, top_n: int = 20):
        """Print profiling stats for a function"""
        if func_name not in self.profiles:
            print(f"No profile data for {func_name}")
            return

        s = io.StringIO()
        ps = pstats.Stats(self.profiles[func_name], stream=s)
        ps.sort_stats(pstats.SortKey.CUMULATIVE)
        ps.print_stats(top_n)

        print(f"\n=== Profile for {func_name} ===")
        print(s.getvalue())

    def get_summary(self):
        """Get summary of all profiled functions"""
        summary = {}

        for func_name, timings in self.timings.items():
            timings_sorted = sorted(timings)
            summary[func_name] = {
                'count': len(timings),
                'avg': sum(timings) / len(timings),
                'p50': timings_sorted[len(timings_sorted) // 2],
                'p95': timings_sorted[int(len(timings_sorted) * 0.95)],
                'p99': timings_sorted[int(len(timings_sorted) * 0.99)],
            }

        return summary

# Usage
profiler = PerformanceProfiler()

@profiler.profile
def expensive_operation(n: int):
    result = []
    for i in range(n):
        result.append(i ** 2)
    return result

@profiler.profile
def another_operation():
    time.sleep(0.1)
    return [i for i in range(10000)]

# Run operations
expensive_operation(100000)
another_operation()

# Print stats
profiler.print_stats('expensive_operation')
print("\nSummary:")
print(profiler.get_summary())
```

---

## Example 4: Memory Leak Detector

```javascript
// memory-leak-detector.js
const v8 = require('v8');
const fs = require('fs');

class MemoryLeakDetector {
  constructor(options = {}) {
    this.thresholdMB = options.thresholdMB || 50;
    this.checkIntervalMs = options.checkIntervalMs || 30000;
    this.snapshotOnLeak = options.snapshotOnLeak !== false;

    this.baseline = null;
    this.measurements = [];
    this.interval = null;
  }

  start() {
    if (this.interval) {
      console.warn('Detector already running');
      return;
    }

    // Force GC before baseline (if --expose-gc)
    if (global.gc) {
      global.gc();
    }

    this.baseline = this.getMemoryStats();
    console.log('Memory leak detector started');
    console.log('Baseline:', this.formatMemory(this.baseline.heapUsed));

    this.interval = setInterval(() => {
      this.check();
    }, this.checkIntervalMs);
  }

  stop() {
    if (this.interval) {
      clearInterval(this.interval);
      this.interval = null;
      console.log('Memory leak detector stopped');
    }
  }

  getMemoryStats() {
    const usage = process.memoryUsage();

    return {
      heapUsed: usage.heapUsed,
      heapTotal: usage.heapTotal,
      external: usage.external,
      rss: usage.rss,
    };
  }

  check() {
    // Force GC if available
    if (global.gc) {
      global.gc();
    }

    const current = this.getMemoryStats();
    const increase = current.heapUsed - this.baseline.heapUsed;
    const increasePercent = (increase / this.baseline.heapUsed) * 100;

    this.measurements.push({
      timestamp: Date.now(),
      heapUsed: current.heapUsed,
      increase: increase,
    });

    // Keep last 20 measurements
    if (this.measurements.length > 20) {
      this.measurements.shift();
    }

    console.log(`Memory: ${this.formatMemory(current.heapUsed)} (+${this.formatMemory(increase)}, +${increasePercent.toFixed(1)}%)`);

    // Check for leak
    if (increase > this.thresholdMB * 1024 * 1024) {
      this.onLeakDetected(current, increase);
    }
  }

  onLeakDetected(current, increase) {
    console.error('⚠️ MEMORY LEAK DETECTED!');
    console.error(`Heap increased by ${this.formatMemory(increase)}`);
    console.error('Current memory:', this.formatMemory(current.heapUsed));

    // Take heap snapshot
    if (this.snapshotOnLeak) {
      const filename = `leak-${Date.now()}.heapsnapshot`;
      v8.writeHeapSnapshot(filename);
      console.error(`Heap snapshot saved: ${filename}`);
    }

    // Analyze trend
    this.analyzeTrend();
  }

  analyzeTrend() {
    if (this.measurements.length < 5) return;

    const recent = this.measurements.slice(-5);
    const isIncreasing = recent.every((m, i) => {
      if (i === 0) return true;
      return m.heapUsed > recent[i - 1].heapUsed;
    });

    if (isIncreasing) {
      console.error('⚠️ Memory is consistently increasing!');
    }
  }

  formatMemory(bytes) {
    return `${(bytes / 1024 / 1024).toFixed(2)} MB`;
  }
}

module.exports = MemoryLeakDetector;

// Usage
const MemoryLeakDetector = require('./memory-leak-detector');

const detector = new MemoryLeakDetector({
  thresholdMB: 100,
  checkIntervalMs: 30000,
  snapshotOnLeak: true,
});

detector.start();

// Run with: node --expose-gc app.js
```

---

**Versión:** 1.0.0
