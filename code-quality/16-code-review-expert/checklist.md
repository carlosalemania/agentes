# Code Review Checklist

## Pre-Review

- [ ] PR has clear description
- [ ] Commit messages are meaningful
- [ ] Tests are included
- [ ] CI/CD passes

## Logic Review

- [ ] Code does what it claims
- [ ] Edge cases are handled
- [ ] Error handling is present
- [ ] Input validation exists
- [ ] No off-by-one errors
- [ ] No race conditions

## Security Review

- [ ] No SQL injection vulnerabilities
- [ ] No XSS vulnerabilities
- [ ] No hardcoded secrets
- [ ] Input is sanitized
- [ ] Authentication is proper
- [ ] Authorization is checked
- [ ] CSRF protection present
- [ ] No sensitive data in logs

## Performance Review

- [ ] No N+1 queries
- [ ] Efficient algorithms used
- [ ] No unnecessary loops
- [ ] Database indexes considered
- [ ] Caching opportunities identified
- [ ] No memory leaks
- [ ] Async operations used properly

## Code Quality

- [ ] SOLID principles followed
- [ ] DRY - no duplication
- [ ] Clear naming conventions
- [ ] Appropriate abstraction level
- [ ] Single responsibility
- [ ] Proper error messages
- [ ] Comments where needed
- [ ] No commented-out code

## Testing

- [ ] Unit tests present
- [ ] Integration tests if needed
- [ ] Edge cases tested
- [ ] Error cases tested
- [ ] Mocks used appropriately
- [ ] Test coverage acceptable

## Documentation

- [ ] README updated if needed
- [ ] API docs updated
- [ ] Breaking changes documented
- [ ] Migration guide if needed

## Communication

- [ ] Feedback is constructive
- [ ] Examples provided
- [ ] Priority indicated
- [ ] Alternative suggested
