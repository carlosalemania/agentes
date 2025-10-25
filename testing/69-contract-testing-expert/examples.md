# Contract Testing Examples

## Example: Pact Consumer Test

```javascript
const { Pact } = require('@pact-foundation/pact');

const provider = new Pact({
  consumer: 'Frontend',
  provider: 'API',
});

describe('API Contract', () => {
  beforeAll(() => provider.setup());
  afterAll(() => provider.finalize());

  it('retrieves products', async () => {
    await provider.addInteraction({
      uponReceiving: 'request for products',
      withRequest: {
        method: 'GET',
        path: '/products',
      },
      willRespondWith: {
        status: 200,
        body: [{
          id: 1,
          name: 'Product',
        }],
      },
    });
  });
});
```
