# Refactoring Checklist

## Before Refactoring

- [ ] Test coverage exists
- [ ] Tests are passing
- [ ] Code is in version control
- [ ] Create a branch
- [ ] Document current behavior

## Code Smells to Address

### Bloaters
- [ ] Long method (>20 lines)
- [ ] Large class (>200 lines)
- [ ] Long parameter list (>3 params)
- [ ] Data clumps

### Object-Orientation Abusers
- [ ] Switch statements
- [ ] Temporary fields
- [ ] Refused bequest
- [ ] Alternative classes with different interfaces

### Change Preventers
- [ ] Divergent change
- [ ] Shotgun surgery
- [ ] Parallel inheritance hierarchies

### Dispensables
- [ ] Comments (should be self-documenting)
- [ ] Duplicate code
- [ ] Dead code
- [ ] Speculative generality

### Couplers
- [ ] Feature envy
- [ ] Inappropriate intimacy
- [ ] Message chains
- [ ] Middle man

## During Refactoring

- [ ] Make one change at a time
- [ ] Run tests after each change
- [ ] Commit frequently
- [ ] Keep changes small
- [ ] Don't add features
- [ ] Don't mix refactoring with feature work

## After Refactoring

- [ ] All tests still pass
- [ ] No new bugs introduced
- [ ] Code is more readable
- [ ] Code is simpler
- [ ] Performance is maintained or improved
- [ ] Documentation updated
- [ ] Team review completed
