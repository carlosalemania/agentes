# Clean Code Enforcer - Examples

> **Ejemplos de refactoring y aplicación de Clean Code principles**

---

## Ejemplo 1: SOLID Refactoring

### Before - Multiple Violations
```javascript
// ❌ Viola SRP, OCP, DIP
class UserManager {
  constructor() {
    this.db = new MySQLDatabase(); // Hard dependency
  }

  createUser(data) {
    // Validation
    if (!data.email || !data.email.includes('@')) {
      throw new Error('Invalid email');
    }

    // Save to DB
    this.db.insert('users', data);

    // Send email
    const smtp = new SMTPClient();
    smtp.send(data.email, 'Welcome!');

    // Log
    console.log(`User created: ${data.email}`);
  }

  deleteUser(id) {
    const user = this.db.find('users', id);

    // Different payment logic per type
    if (user.type === 'premium') {
      // Refund logic
      const stripe = new StripeAPI();
      stripe.refund(user.subscriptionId);
    } else if (user.type === 'basic') {
      // Different refund logic
      const paypal = new PayPalAPI();
      paypal.refund(user.subscriptionId);
    }

    this.db.delete('users', id);
  }
}
```

### After - SOLID Applied
```javascript
// ✅ SRP: Each class has single responsibility
// ✅ OCP: Open for extension, closed for modification
// ✅ DIP: Depend on abstractions, not concretions

// Abstractions
interface Database {
  insert(table: string, data: any): void;
  find(table: string, id: string): any;
  delete(table: string, id: string): void;
}

interface EmailService {
  send(to: string, subject: string, body: string): void;
}

interface PaymentRefundStrategy {
  refund(subscriptionId: string): void;
}

// Implementations
class UserValidator {
  validate(data) {
    if (!data.email || !data.email.includes('@')) {
      throw new Error('Invalid email');
    }
  }
}

class UserRepository {
  constructor(private db: Database) {}

  create(user) {
    this.db.insert('users', user);
  }

  findById(id) {
    return this.db.find('users', id);
  }

  delete(id) {
    this.db.delete('users', id);
  }
}

class UserNotificationService {
  constructor(private emailService: EmailService) {}

  sendWelcomeEmail(user) {
    this.emailService.send(user.email, 'Welcome!', '...');
  }
}

class Logger {
  log(message) {
    console.log(message);
  }
}

// Refund strategies (OCP)
class StripeRefundStrategy implements PaymentRefundStrategy {
  refund(subscriptionId) {
    const stripe = new StripeAPI();
    stripe.refund(subscriptionId);
  }
}

class PayPalRefundStrategy implements PaymentRefundStrategy {
  refund(subscriptionId) {
    const paypal = new PayPalAPI();
    paypal.refund(subscriptionId);
  }
}

// Coordinating service (SRP)
class UserService {
  constructor(
    private validator: UserValidator,
    private repository: UserRepository,
    private notificationService: UserNotificationService,
    private logger: Logger,
    private refundStrategyFactory: RefundStrategyFactory
  ) {}

  createUser(data) {
    this.validator.validate(data);
    this.repository.create(data);
    this.notificationService.sendWelcomeEmail(data);
    this.logger.log(`User created: ${data.email}`);
  }

  deleteUser(id) {
    const user = this.repository.findById(id);

    const refundStrategy = this.refundStrategyFactory.create(user.type);
    refundStrategy.refund(user.subscriptionId);

    this.repository.delete(id);
  }
}
```

---

## Ejemplo 2: DRY Violation

### Before
```javascript
function calculatePriceForRegularUser(basePrice) {
  const tax = basePrice * 0.15;
  const shipping = basePrice > 100 ? 0 : 10;
  return basePrice + tax + shipping;
}

function calculatePriceForPremiumUser(basePrice) {
  const tax = basePrice * 0.15;
  const discount = basePrice * 0.1;
  const shipping = 0; // Free shipping
  return basePrice + tax - discount + shipping;
}

function calculatePriceForCorporateUser(basePrice) {
  const tax = basePrice * 0.15;
  const discount = basePrice * 0.2;
  const shipping = 0;
  return basePrice + tax - discount + shipping;
}
```

### After
```javascript
class PriceCalculator {
  calculate(basePrice, userType) {
    const tax = this.calculateTax(basePrice);
    const discount = this.calculateDiscount(basePrice, userType);
    const shipping = this.calculateShipping(basePrice, userType);

    return basePrice + tax - discount + shipping;
  }

  calculateTax(basePrice) {
    return basePrice * 0.15;
  }

  calculateDiscount(basePrice, userType) {
    const discounts = {
      'regular': 0,
      'premium': basePrice * 0.1,
      'corporate': basePrice * 0.2
    };
    return discounts[userType] || 0;
  }

  calculateShipping(basePrice, userType) {
    if (userType !== 'regular') return 0;
    return basePrice > 100 ? 0 : 10;
  }
}
```

---

**Versión:** 1.0.0
