# GraphQL Schema Designer - Examples

---

## Example: Complete GraphQL Server

```javascript
const { ApolloServer } = require('apollo-server');

const typeDefs = gql`
  type Query {
    users: [User!]!
  }
  type User {
    id: ID!
    name: String!
  }
`;

const resolvers = {
  Query: {
    users: () => [{ id: '1', name: 'John' }]
  }
};

const server = new ApolloServer({ typeDefs, resolvers });
```

---

**Versión:** 1.0.0
