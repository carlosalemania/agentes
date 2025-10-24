# TypeScript Expert - Examples

---

## Example: API Client with Types

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

class ApiClient {
  async get<T>(url: string): Promise<T> {
    const response = await fetch(url);
    return response.json();
  }

  async post<T, D>(url: string, data: D): Promise<T> {
    const response = await fetch(url, {
      method: 'POST',
      body: JSON.stringify(data),
    });
    return response.json();
  }
}

// Usage
const client = new ApiClient();
const user = await client.get<User>('/api/users/1');
```

---

**Versión:** 1.0.0
