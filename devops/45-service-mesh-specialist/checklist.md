# Service Mesh Checklist

## Installation

- [ ] Service mesh installed (Istio/Linkerd/Consul)
- [ ] Control plane healthy
- [ ] Sidecar injection configured
- [ ] Ingress gateway deployed
- [ ] Egress gateway if needed

## Traffic Management

- [ ] VirtualServices defined
- [ ] DestinationRules configured
- [ ] Load balancing strategy set
- [ ] Routing rules tested
- [ ] Traffic splitting working

## Resilience

- [ ] Retries configured
- [ ] Timeouts set
- [ ] Circuit breakers enabled
- [ ] Fault injection tested
- [ ] Rate limiting applied

## Security

- [ ] mTLS enabled
- [ ] Certificate rotation automated
- [ ] Authorization policies defined
- [ ] Service-to-service auth working
- [ ] Egress traffic controlled

## Observability

- [ ] Distributed tracing enabled
- [ ] Metrics exported to Prometheus
- [ ] Access logs configured
- [ ] Service graph visible
- [ ] Dashboards created

## Performance

- [ ] Resource limits set
- [ ] Sidecar overhead acceptable
- [ ] Latency monitored
- [ ] Connection pooling optimized

## Testing

- [ ] Canary deployments tested
- [ ] A/B testing working
- [ ] Fault injection validated
- [ ] Load testing completed
- [ ] Failover tested

## Operations

- [ ] Upgrade strategy defined
- [ ] Rollback procedure tested
- [ ] Monitoring alerts configured
- [ ] Documentation updated
