# CQRS Checklist

## Commands

- [ ] Command objects defined
- [ ] Command handlers implemented
- [ ] Validation logic present
- [ ] Business rules enforced
- [ ] Write model updated
- [ ] Events published

## Queries

- [ ] Query objects defined
- [ ] Query handlers implemented
- [ ] Read model optimized
- [ ] Indexes created
- [ ] Pagination supported

## Write Model

- [ ] Normalized schema
- [ ] ACID transactions
- [ ] Constraints enforced
- [ ] Audit trail
- [ ] Event sourcing if needed

## Read Model

- [ ] Denormalized for queries
- [ ] Appropriate storage (SQL/NoSQL)
- [ ] Optimized indexes
- [ ] Projection logic implemented
- [ ] Cache layer if needed

## Events

- [ ] Event schema defined
- [ ] Event bus configured
- [ ] Event handlers registered
- [ ] Event versioning strategy
- [ ] Event replay capability

## Consistency

- [ ] Eventual consistency accepted
- [ ] Projection lag monitored
- [ ] Conflict resolution strategy
- [ ] Stale read handling
- [ ] Compensation logic if needed

## Scalability

- [ ] Read/write scaled independently
- [ ] Appropriate storage for each
- [ ] Connection pooling
- [ ] Caching strategy

## Monitoring

- [ ] Command metrics
- [ ] Query metrics
- [ ] Projection lag
- [ ] Event processing time
- [ ] Error rates

## Testing

- [ ] Command handlers tested
- [ ] Query handlers tested
- [ ] Projections tested
- [ ] Event flow tested
- [ ] Eventually consistent scenarios tested
