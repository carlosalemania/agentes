# gRPC Checklist

## Proto Design

- [ ] Use proto3 syntax
- [ ] Package name follows reverse domain naming
- [ ] Messages use CamelCase
- [ ] Fields use snake_case
- [ ] Field numbers are unique and reserved
- [ ] Deprecated fields are marked
- [ ] Backward compatibility maintained

## Service Definition

- [ ] RPC methods are well-named
- [ ] Request/Response types defined
- [ ] Streaming type chosen appropriately
- [ ] Comments document behavior
- [ ] Error cases documented

## Server Implementation

- [ ] All RPC methods implemented
- [ ] Error handling with gRPC status codes
- [ ] Input validation
- [ ] Logging and monitoring
- [ ] Graceful shutdown
- [ ] Health check endpoint
- [ ] TLS configured (production)

## Client Implementation

- [ ] Connection pooling
- [ ] Retry logic with exponential backoff
- [ ] Deadline/timeout set
- [ ] Error handling
- [ ] Circuit breaker (if needed)
- [ ] Load balancing configured

## Streaming

- [ ] Stream backpressure handled
- [ ] Resources cleaned up on end/error
- [ ] Cancellation handled
- [ ] Heartbeat for long connections

## Security

- [ ] TLS enabled
- [ ] Authentication implemented
- [ ] Authorization checks
- [ ] Rate limiting
- [ ] Input sanitization

## Performance

- [ ] Message size reasonable
- [ ] Compression enabled if beneficial
- [ ] Connection reuse
- [ ] Appropriate streaming type
- [ ] Benchmarked under load

## Observability

- [ ] Request tracing
- [ ] Metrics (latency, errors)
- [ ] Structured logging
- [ ] Error reporting
