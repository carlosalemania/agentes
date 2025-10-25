# Multi-Tenancy Examples

## Example: Tenant Context Management

```javascript
class TenantContext {
  constructor() {
    this.storage = new AsyncLocalStorage();
  }

  run(tenantId, callback) {
    this.storage.run({ tenantId }, callback);
  }

  getTenantId() {
    const store = this.storage.getStore();
    return store?.tenantId;
  }
}

const tenantContext = new TenantContext();

app.use((req, res, next) => {
  const tenantId = req.headers['x-tenant-id'];
  tenantContext.run(tenantId, () => next());
});
```
