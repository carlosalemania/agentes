# Search/Elasticsearch Examples

## Example: E-commerce Product Search

```javascript
class ProductSearch {
  async search({ query, filters, sort, page = 1, size = 20 }) {
    const must = [];
    const filter = [];

    if (query) {
      must.push({
        multi_match: {
          query,
          fields: ['name^3', 'description', 'brand^2'],
          fuzziness: 'AUTO',
        },
      });
    }

    if (filters.category) {
      filter.push({ term: { 'category.keyword': filters.category } });
    }

    if (filters.brand) {
      filter.push({ terms: { 'brand.keyword': filters.brand } });
    }

    if (filters.price) {
      filter.push({
        range: {
          price: {
            gte: filters.price.min,
            lte: filters.price.max,
          },
        },
      });
    }

    const result = await client.search({
      index: 'products',
      body: {
        query: {
          bool: {
            must: must.length ? must : { match_all: {} },
            filter,
          },
        },
        aggs: {
          categories: { terms: { field: 'category.keyword' } },
          brands: { terms: { field: 'brand.keyword' } },
          price_stats: { stats: { field: 'price' } },
        },
        sort: this.buildSort(sort),
        from: (page - 1) * size,
        size,
      },
    });

    return {
      products: result.hits.hits.map((h) => h._source),
      total: result.hits.total.value,
      facets: result.aggregations,
    };
  }

  buildSort(sort) {
    switch (sort) {
      case 'price_asc':
        return [{ price: 'asc' }];
      case 'price_desc':
        return [{ price: 'desc' }];
      case 'newest':
        return [{ created_at: 'desc' }];
      default:
        return [{ _score: 'desc' }];
    }
  }
}
```
