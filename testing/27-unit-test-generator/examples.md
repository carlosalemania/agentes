# Unit Test Generator - Examples

---

## Example 1: Async API Tests

```javascript
// API client to test
export class ApiClient {
  async fetchUser(userId) {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  }

  async createUser(userData) {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  }
}

// ✅ Tests with fetch mocking
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { ApiClient } from './ApiClient';

describe('ApiClient', () => {
  let client;

  beforeEach(() => {
    client = new ApiClient();
    global.fetch = vi.fn();
  });

  describe('fetchUser', () => {
    it('should fetch user successfully', async () => {
      // Arrange
      const mockUser = { id: 1, name: 'John' };
      global.fetch.mockResolvedValue({
        ok: true,
        json: async () => mockUser
      });

      // Act
      const result = await client.fetchUser(1);

      // Assert
      expect(result).toEqual(mockUser);
      expect(global.fetch).toHaveBeenCalledWith('/api/users/1');
    });

    it('should throw error on HTTP error', async () => {
      // Arrange
      global.fetch.mockResolvedValue({
        ok: false,
        status: 404
      });

      // Act & Assert
      await expect(client.fetchUser(999)).rejects.toThrow('HTTP 404');
    });
  });

  describe('createUser', () => {
    it('should create user with correct payload', async () => {
      // Arrange
      const userData = { name: 'Jane', email: 'jane@example.com' };
      const createdUser = { id: 2, ...userData };

      global.fetch.mockResolvedValue({
        ok: true,
        json: async () => createdUser
      });

      // Act
      const result = await client.createUser(userData);

      // Assert
      expect(result).toEqual(createdUser);
      expect(global.fetch).toHaveBeenCalledWith(
        '/api/users',
        expect.objectContaining({
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(userData)
        })
      );
    });
  });
});
```

## Example 2: Class with State

```typescript
// Shopping cart class
export class ShoppingCart {
  private items: Map<string, number> = new Map();

  addItem(productId: string, quantity: number = 1): void {
    if (quantity <= 0) {
      throw new Error('Quantity must be positive');
    }

    const currentQty = this.items.get(productId) || 0;
    this.items.set(productId, currentQty + quantity);
  }

  removeItem(productId: string): void {
    this.items.delete(productId);
  }

  getTotal(): number {
    return Array.from(this.items.values()).reduce((sum, qty) => sum + qty, 0);
  }

  clear(): void {
    this.items.clear();
  }
}

// ✅ Tests for stateful class
import { describe, it, expect, beforeEach } from 'vitest';
import { ShoppingCart } from './ShoppingCart';

describe('ShoppingCart', () => {
  let cart: ShoppingCart;

  beforeEach(() => {
    cart = new ShoppingCart();
  });

  describe('addItem', () => {
    it('should add new item to cart', () => {
      cart.addItem('product1', 2);
      expect(cart.getTotal()).toBe(2);
    });

    it('should increment quantity for existing item', () => {
      cart.addItem('product1', 2);
      cart.addItem('product1', 3);
      expect(cart.getTotal()).toBe(5);
    });

    it('should throw error for zero quantity', () => {
      expect(() => cart.addItem('product1', 0)).toThrow('Quantity must be positive');
    });

    it('should throw error for negative quantity', () => {
      expect(() => cart.addItem('product1', -1)).toThrow('Quantity must be positive');
    });
  });

  describe('removeItem', () => {
    it('should remove item from cart', () => {
      cart.addItem('product1', 2);
      cart.removeItem('product1');
      expect(cart.getTotal()).toBe(0);
    });

    it('should not throw when removing non-existent item', () => {
      expect(() => cart.removeItem('nonexistent')).not.toThrow();
    });
  });

  describe('getTotal', () => {
    it('should return 0 for empty cart', () => {
      expect(cart.getTotal()).toBe(0);
    });

    it('should return correct total for multiple items', () => {
      cart.addItem('product1', 2);
      cart.addItem('product2', 3);
      cart.addItem('product3', 1);
      expect(cart.getTotal()).toBe(6);
    });
  });

  describe('clear', () => {
    it('should remove all items from cart', () => {
      cart.addItem('product1', 2);
      cart.addItem('product2', 3);
      cart.clear();
      expect(cart.getTotal()).toBe(0);
    });
  });
});
```

---

**Versión:** 1.0.0
