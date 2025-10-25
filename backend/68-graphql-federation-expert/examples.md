# GraphQL Federation Examples

## Example: Federated Schema

```graphql
# Users service
type User @key(fields: "id") {
  id: ID!
  name: String!
}

# Orders service
extend type User @key(fields: "id") {
  id: ID! @external
  orders: [Order!]!
}

type Order {
  id: ID!
  userId: ID!
  total: Float!
}
```
