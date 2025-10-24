# E2E Test Expert - Examples

---

## Example: Cypress E2E Test

```javascript
describe('User Registration', () => {
  it('should register new user', () => {
    cy.visit('/register');

    cy.get('[data-testid="name-input"]').type('John Doe');
    cy.get('[data-testid="email-input"]').type('john@example.com');
    cy.get('[data-testid="password-input"]').type('SecurePass123');
    cy.get('[data-testid="submit-btn"]').click();

    cy.url().should('include', '/dashboard');
    cy.contains('Welcome, John').should('be.visible');
  });
});
```

---

**Versión:** 1.0.0
