# Cron/Scheduler Checklist

## Schedule Design

- [ ] Cron expression is correct
- [ ] Time zone specified
- [ ] Execution frequency appropriate
- [ ] Resource impact considered
- [ ] Dependencies identified

## Implementation

- [ ] Job is idempotent
- [ ] Timeout configured
- [ ] Error handling implemented
- [ ] Logging added
- [ ] Metrics tracked
- [ ] Graceful shutdown

## Distributed Systems

- [ ] Single instance enforcement (locks)
- [ ] Leader election if needed
- [ ] Job persistence
- [ ] Failure recovery
- [ ] No duplicate execution

## Reliability

- [ ] Retry logic
- [ ] Dead letter queue
- [ ] Circuit breaker if needed
- [ ] Health monitoring
- [ ] Alerting configured

## Performance

- [ ] Resource usage monitored
- [ ] Concurrency limited
- [ ] Database impact assessed
- [ ] Rate limiting if needed

## Testing

- [ ] Job logic tested
- [ ] Schedule tested
- [ ] Failure scenarios tested
- [ ] Time zone handling verified
- [ ] Load tested

## Observability

- [ ] Execution logged
- [ ] Duration tracked
- [ ] Success/failure metrics
- [ ] Stale job detection
- [ ] Dashboard created

## Security

- [ ] Credentials secured
- [ ] Access controlled
- [ ] Input validated
- [ ] Audit trail
