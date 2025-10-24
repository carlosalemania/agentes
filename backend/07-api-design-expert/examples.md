# API Design Expert - Examples

> **Ejemplos de diseño de APIs RESTful y GraphQL**

---

## Ejemplo 1: REST API - E-commerce

### Endpoints Design
```
# Products
GET    /api/v1/products              # List products
GET    /api/v1/products/{id}          # Get product
POST   /api/v1/products               # Create product (admin)
PATCH  /api/v1/products/{id}          # Update product (admin)
DELETE /api/v1/products/{id}          # Delete product (admin)

# Categories
GET    /api/v1/categories             # List categories
GET    /api/v1/products?category=electronics  # Filter by category

# Orders
GET    /api/v1/orders                 # List my orders
GET    /api/v1/orders/{id}            # Get order
POST   /api/v1/orders                 # Create order
PATCH  /api/v1/orders/{id}/cancel     # Cancel order
GET    /api/v1/users/{id}/orders      # User's orders (admin)

# Cart
GET    /api/v1/cart                   # Get cart
POST   /api/v1/cart/items             # Add item
PATCH  /api/v1/cart/items/{id}        # Update quantity
DELETE /api/v1/cart/items/{id}        # Remove item
POST   /api/v1/cart/checkout          # Checkout
```

### Response Format
```json
// GET /api/v1/products?limit=2
{
  "data": [
    {
      "id": 1,
      "name": "Laptop",
      "price": 999.99,
      "category": "electronics",
      "_links": {
        "self": "/api/v1/products/1",
        "category": "/api/v1/categories/electronics"
      }
    }
  ],
  "pagination": {
    "limit": 2,
    "offset": 0,
    "total": 150,
    "nextCursor": "eyJpZCI6M30"
  },
  "_links": {
    "self": "/api/v1/products?limit=2",
    "next": "/api/v1/products?cursor=eyJpZCI6M30&limit=2"
  }
}
```

---

## Ejemplo 2: GraphQL Schema - Social Network

```graphql
type User {
  id: ID!
  username: String!
  email: String!
  posts(first: Int, after: String): PostConnection!
  followers(first: Int): UserConnection!
  following(first: Int): UserConnection!
}

type Post {
  id: ID!
  content: String!
  author: User!
  likes: Int!
  comments(first: Int): CommentConnection!
  createdAt: DateTime!
}

type Query {
  me: User
  user(username: String!): User
  posts(first: Int, after: String): PostConnection!
}

type Mutation {
  createPost(input: CreatePostInput!): CreatePostPayload!
  likePost(postId: ID!): LikePostPayload!
}
```

---

## Ejemplo 3: Versioning Strategy

```javascript
// v1 - Initial version
GET /api/v1/users
Response: { id, name, email }

// v2 - Add new fields
GET /api/v2/users
Response: { id, name, email, avatar, bio }

// Deprecation notice in v1
HTTP/1.1 200 OK
Deprecation: true
Sunset: 2025-12-31
Link: </api/v2/users>; rel="successor-version"
```

---

**Versión:** 1.0.0
