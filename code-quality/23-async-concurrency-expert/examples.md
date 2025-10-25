# Async/Concurrency Expert - Examples

---

## Example 1: Batch Processing with Concurrency Limit

```javascript
// Process 10,000 items with max 50 concurrent operations
async function processBatch(items, batchSize = 100, concurrency = 50) {
  const pLimit = require('p-limit');
  const limit = pLimit(concurrency);

  const results = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);

    const batchPromises = batch.map(item =>
      limit(async () => {
        try {
          const result = await processItem(item);
          console.log(`Processed item ${item.id}`);
          return { success: true, result };
        } catch (error) {
          console.error(`Failed to process item ${item.id}:`, error);
          return { success: false, error: error.message };
        }
      })
    );

    const batchResults = await Promise.all(batchPromises);
    results.push(...batchResults);

    // Log progress
    console.log(`Progress: ${results.length}/${items.length}`);
  }

  const successful = results.filter(r => r.success).length;
  const failed = results.filter(r => !r.success).length;

  console.log(`Completed: ${successful} successful, ${failed} failed`);

  return results;
}

// Usage
const items = Array.from({ length: 10000 }, (_, i) => ({ id: i }));
const results = await processBatch(items, 100, 50);
```

---

## Example 2: Parallel API Calls with Fallback

```javascript
// Fetch from primary API, fallback to secondary if fails
async function fetchWithFallback(endpoint) {
  const primaryUrl = `https://api-primary.example.com${endpoint}`;
  const secondaryUrl = `https://api-secondary.example.com${endpoint}`;

  try {
    // Try primary first
    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), 3000);

    const response = await fetch(primaryUrl, { signal: controller.signal });
    clearTimeout(timeout);

    if (!response.ok) throw new Error(`HTTP ${response.status}`);

    return await response.json();
  } catch (error) {
    console.warn('Primary API failed, trying secondary:', error.message);

    // Fallback to secondary
    try {
      const response = await fetch(secondaryUrl);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return await response.json();
    } catch (secondaryError) {
      console.error('Both APIs failed');
      throw new Error('All API endpoints failed');
    }
  }
}

// Fetch multiple endpoints in parallel
async function fetchMultiple(endpoints) {
  const results = await Promise.allSettled(
    endpoints.map(endpoint => fetchWithFallback(endpoint))
  );

  return results.map((result, index) => ({
    endpoint: endpoints[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason.message : null,
  }));
}
```

---

## Example 3: Rate-Limited API Client

```javascript
// API client with rate limiting
class RateLimitedClient {
  constructor(requestsPerSecond = 10) {
    this.requestsPerSecond = requestsPerSecond;
    this.interval = 1000 / requestsPerSecond;
    this.queue = [];
    this.processing = false;
  }

  async request(url, options = {}) {
    return new Promise((resolve, reject) => {
      this.queue.push({ url, options, resolve, reject });
      this.processQueue();
    });
  }

  async processQueue() {
    if (this.processing || this.queue.length === 0) {
      return;
    }

    this.processing = true;

    while (this.queue.length > 0) {
      const { url, options, resolve, reject } = this.queue.shift();
      const startTime = Date.now();

      try {
        const response = await fetch(url, options);
        const data = await response.json();
        resolve(data);
      } catch (error) {
        reject(error);
      }

      // Wait for rate limit interval
      const elapsed = Date.now() - startTime;
      const waitTime = Math.max(0, this.interval - elapsed);

      if (waitTime > 0 && this.queue.length > 0) {
        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }

    this.processing = false;
  }

  async get(url) {
    return this.request(url, { method: 'GET' });
  }

  async post(url, data) {
    return this.request(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
  }
}

// Usage
const client = new RateLimitedClient(10); // 10 requests per second

const promises = [];
for (let i = 0; i < 100; i++) {
  promises.push(client.get(`https://api.example.com/item/${i}`));
}

const results = await Promise.all(promises);
console.log(`Fetched ${results.length} items`);
```

---

## Example 4: Async Pool with Progress Tracking

```javascript
// Async pool with progress tracking
class AsyncPool {
  constructor(concurrency = 5) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
    this.completed = 0;
    this.total = 0;
    this.onProgress = null;
  }

  async add(fn) {
    this.total++;

    return new Promise((resolve, reject) => {
      this.queue.push({ fn, resolve, reject });
      this.process();
    });
  }

  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }

    this.running++;
    const { fn, resolve, reject } = this.queue.shift();

    try {
      const result = await fn();
      this.completed++;

      if (this.onProgress) {
        this.onProgress({
          completed: this.completed,
          total: this.total,
          percentage: (this.completed / this.total) * 100,
        });
      }

      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process();
    }
  }

  async all(items, fn) {
    const promises = items.map(item => this.add(() => fn(item)));
    return Promise.all(promises);
  }
}

// Usage
const pool = new AsyncPool(10);

pool.onProgress = ({ completed, total, percentage }) => {
  console.log(`Progress: ${completed}/${total} (${percentage.toFixed(1)}%)`);
};

const items = Array.from({ length: 1000 }, (_, i) => i);

const results = await pool.all(items, async (item) => {
  await new Promise(resolve => setTimeout(resolve, 100)); // Simulate work
  return item * 2;
});

console.log('All items processed!');
```

---

## Example 5: Python Async Batch Processor

```python
import asyncio
import aiohttp
from typing import List, TypeVar, Callable
import time

T = TypeVar('T')
R = TypeVar('R')

class AsyncBatchProcessor:
    def __init__(self, concurrency: int = 10, batch_size: int = 100):
        self.concurrency = concurrency
        self.batch_size = batch_size
        self.semaphore = asyncio.Semaphore(concurrency)

    async def process_item(
        self,
        item: T,
        fn: Callable[[T], R]
    ) -> R:
        async with self.semaphore:
            return await fn(item)

    async def process_batch(
        self,
        items: List[T],
        fn: Callable[[T], R]
    ) -> List[R]:
        tasks = [self.process_item(item, fn) for item in items]
        results = await asyncio.gather(*tasks, return_exceptions=True)

        # Separate successful results from errors
        successful = []
        errors = []

        for i, result in enumerate(results):
            if isinstance(result, Exception):
                errors.append((items[i], result))
            else:
                successful.append(result)

        return successful, errors

    async def process_all(
        self,
        items: List[T],
        fn: Callable[[T], R]
    ) -> List[R]:
        all_results = []
        all_errors = []

        for i in range(0, len(items), self.batch_size):
            batch = items[i:i + self.batch_size]

            print(f"Processing batch {i//self.batch_size + 1}...")
            start_time = time.time()

            results, errors = await self.process_batch(batch, fn)

            all_results.extend(results)
            all_errors.extend(errors)

            elapsed = time.time() - start_time
            print(f"Batch completed in {elapsed:.2f}s")

        print(f"Total: {len(all_results)} successful, {len(all_errors)} errors")

        return all_results

# Usage
async def fetch_user(user_id: int):
    async with aiohttp.ClientSession() as session:
        async with session.get(f"https://api.example.com/users/{user_id}") as response:
            return await response.json()

async def main():
    processor = AsyncBatchProcessor(concurrency=50, batch_size=100)
    user_ids = list(range(1, 1001))

    results = await processor.process_all(user_ids, fetch_user)
    print(f"Fetched {len(results)} users")

if __name__ == "__main__":
    asyncio.run(main())
```

---

**Versión:** 1.0.0
