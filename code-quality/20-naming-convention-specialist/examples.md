# Naming Convention Specialist - Examples

> **Casos de uso reales y refactoring de nombres**

---

## 📋 Tabla de Contenidos

- [Ejemplo 1: Refactoring Variables Crípticas](#ejemplo-1-refactoring-variables-crípticas)
- [Ejemplo 2: Funciones con Nombres Vagos](#ejemplo-2-funciones-con-nombres-vagos)
- [Ejemplo 3: Clases y Domain-Driven Design](#ejemplo-3-clases-y-domain-driven-design)
- [Ejemplo 4: Boolean Naming](#ejemplo-4-boolean-naming)
- [Ejemplo 5: Convenciones Multi-Lenguaje](#ejemplo-5-convenciones-multi-lenguaje)

---

## Ejemplo 1: Refactoring Variables Crípticas

### Problema
```javascript
// ❌ Variables crípticas sin contexto
function calc(a, b, c, d) {
  const x = a * b;
  const y = x * c / 100;
  const z = y - d;
  const result = z > 0 ? z : 0;

  return result;
}

const t = calc(50, 2, 15, 5);
const arr = [1, 2, 3, 4, 5];
const dt = new Date();
const flg = true;
const tmp = null;
```

**Problemas identificados:**
- ❌ Single letter variables (`a`, `b`, `c`, `d`, `x`, `y`, `z`, `t`)
- ❌ Abreviaciones no claras (`arr`, `dt`, `flg`, `tmp`)
- ❌ Nombres genéricos sin contexto
- ❌ No revela intención del código

### Solución
```javascript
// ✅ Variables descriptivas y autodocumentadas
function calculateNetPriceAfterTaxAndDiscount(
  unitPrice,
  quantity,
  taxRate,
  discountAmount
) {
  const subtotal = unitPrice * quantity;
  const taxAmount = subtotal * taxRate / 100;
  const totalWithTax = subtotal + taxAmount;
  const finalPrice = totalWithTax - discountAmount;
  const netPrice = finalPrice > 0 ? finalPrice : 0;

  return netPrice;
}

const totalOrderPrice = calculateNetPriceAfterTaxAndDiscount(50, 2, 15, 5);
const userIds = [1, 2, 3, 4, 5];
const orderCreatedAt = new Date();
const isPaymentProcessed = true;
const cachedUserData = null;
```

### Mejoras
- ✅ Nombres pronunciables y buscables
- ✅ Revelan intención clara
- ✅ Autodocumentados (no necesitan comentarios)
- ✅ Fáciles de entender para nuevos desarrolladores
- ✅ Contexto explícito en cada nombre

---

## Ejemplo 2: Funciones con Nombres Vagos

### Problema
```javascript
// ❌ Funciones con nombres genéricos
class UserManager {
  process(data) {
    // ¿Qué proceso? ¿Qué datos?
    // ...
  }

  handle(req) {
    // ¿Handle qué?
    // ...
  }

  doStuff(x, y) {
    // ¿Qué hace exactamente?
    // ...
  }

  manage() {
    // ¿Manage qué?
    // ...
  }

  update(obj) {
    // ¿Update qué propiedad?
    // ...
  }
}

// Uso poco claro
const um = new UserManager();
um.process({ id: 1 });
```

**Problemas:**
- ❌ Verbos genéricos (`process`, `handle`, `manage`, `doStuff`, `update`)
- ❌ No revela qué hace la función
- ❌ Parámetros crípticos (`data`, `req`, `x`, `y`, `obj`)
- ❌ Clase "Manager" (anti-pattern)

### Solución
```javascript
// ✅ Funciones con verbos específicos
class UserAuthenticator {
  authenticateUserWithCredentials(credentials) {
    // Autenticación clara
    const { email, password } = credentials;
    // ...
  }

  validateUserPermissions(user, resource) {
    // Validación específica
    // ...
  }

  generateAuthenticationToken(userId) {
    // Generación explícita
    // ...
  }

  revokeUserSession(sessionId) {
    // Revocación clara
    // ...
  }

  updateUserLastLoginTimestamp(userId) {
    // Update específico
    // ...
  }
}

// Uso autodocumentado
const authenticator = new UserAuthenticator();
authenticator.authenticateUserWithCredentials({
  email: 'user@example.com',
  password: 'secret'
});
```

### Patrones de Verbos

```javascript
// ✅ CRUD operations - verbos específicos
create...()    // createUser, createOrder
get...()       // getUserById, getActiveOrders
find...()      // findUserByEmail, findOrdersByDate
fetch...()     // fetchUserProfile, fetchLatestPosts
retrieve...()  // retrievePaymentHistory
update...()    // updateUserProfile, updateOrderStatus
delete...()    // deleteUser, deleteExpiredSessions
remove...()    // removeItemFromCart

// ✅ Boolean checks - is/has/can/should
is...()        // isAuthenticated, isValidEmail
has...()       // hasPermission, hasActiveSubscription
can...()       // canEditPost, canAccessResource
should...()    // shouldRetryRequest, shouldCacheResult

// ✅ Business operations - domain verbs
place...()     // placeOrder
cancel...()    // cancelSubscription
ship...()      // shipOrder
process...()   // processPayment (OK when specific)
calculate...() // calculateTotalPrice
generate...()  // generateInvoice
validate...()  // validateCreditCard
```

---

## Ejemplo 3: Clases y Domain-Driven Design

### Problema
```javascript
// ❌ Nombres técnicos sin lenguaje del dominio
class DataManager {
  saveData(data) {}
  loadData(id) {}
  deleteData(id) {}
}

class UserHandler {
  processUser(user) {}
  updateUserInfo(user, info) {}
}

class OrderProcessor {
  handleOrder(order) {}
  changeStatus(order, status) {}
}

// ❌ Inconsistencia en el dominio
class Customer {}  // En un contexto
class User {}      // En otro contexto - ¿son lo mismo?
class Client {}    // En otro más - confuso
```

**Problemas:**
- ❌ "Manager", "Handler", "Processor" - anti-patterns
- ❌ No usa ubiquitous language del dominio
- ❌ Responsabilidades vagas
- ❌ Inconsistencia en conceptos del negocio

### Solución
```javascript
// ✅ Domain-Driven Design naming

// === E-commerce Domain ===

// Aggregates
class Order {
  placeOrder(customerId, items) {}
  addItem(item) {}
  removeItem(itemId) {}
  cancelOrder(reason) {}
  shipOrder(shippingInfo) {}
  completeOrder() {}
}

class ShoppingCart {
  addProduct(product, quantity) {}
  removeProduct(productId) {}
  updateQuantity(productId, newQuantity) {}
  calculateTotal() {}
  checkout() {}
  abandonCart() {}
}

// Value Objects
class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }

  add(other) {}
  subtract(other) {}
  isGreaterThan(other) {}
}

class Address {
  constructor(street, city, province, postalCode) {
    this.street = street;
    this.city = city;
    this.province = province;
    this.postalCode = postalCode;
  }

  isValidForShipping() {}
}

// Services (cuando no pertenece a ningún aggregate)
class PaymentGateway {
  processPayment(paymentDetails) {}
  refundPayment(transactionId) {}
  validatePaymentMethod(method) {}
}

class InventoryService {
  reserveStock(productId, quantity) {}
  releaseStock(productId, quantity) {}
  checkAvailability(productId) {}
}

// Repositories (persistencia)
class OrderRepository {
  save(order) {}
  findById(orderId) {}
  findByCustomerId(customerId) {}
  findPendingOrders() {}
}

// Factories
class OrderFactory {
  createFromCart(cart, shippingAddress) {}
  createSubscriptionOrder(subscription) {}
}

// === Bounded Contexts ===

// Sales Context
class Customer {
  customerId: string;
  purchaseHistory: Order[];
  totalSpent: Money;
  loyaltyPoints: number;

  placeOrder(order) {}
  earnLoyaltyPoints(points) {}
}

// Support Context
class Customer {
  customerId: string;
  supportTickets: Ticket[];
  satisfactionScore: number;

  openTicket(issue) {}
  rateSupport(score) {}
}

// Shipping Context - mismo concepto, nombre diferente
class Recipient {
  recipientId: string;
  shippingAddress: Address;
  deliveryPreferences: DeliveryPreferences;

  updateShippingAddress(newAddress) {}
  setDeliveryPreferences(preferences) {}
}
```

### Ubiquitous Language

```javascript
// ✅ Mismo lenguaje en código y negocio

// El negocio dice: "El cliente coloca una orden"
customer.placeOrder(order);

// El negocio dice: "La orden se envía con tracking"
order.shipWithTracking(trackingNumber);

// El negocio dice: "El pago se procesa y se captura"
payment.process();
payment.capture();

// El negocio dice: "El inventario reserva stock"
inventory.reserveStock(productId, quantity);

// ❌ NO decir: "Manager procesa data"
manager.processData(data);  // Lenguaje técnico, no del dominio
```

---

## Ejemplo 4: Boolean Naming

### Problema
```javascript
// ❌ Booleans sin prefijo claro
const active = true;
const valid = user.email.includes('@');
const permission = user.role === 'admin';
const loading = true;
const disabled = false;
const visible = element.style.display !== 'none';

// ❌ Negative booleans (confusos)
const notReady = false;
const notFound = true;
const notValid = !isValid(data);

if (!notReady) {  // Doble negativo - confuso
  start();
}
```

**Problemas:**
- ❌ No queda claro que es boolean
- ❌ Negative booleans dificultan lectura
- ❌ Sin prefijo (is/has/can/should)

### Solución
```javascript
// ✅ Booleans con prefijos claros

// is - estado
const isActive = true;
const isLoading = true;
const isVisible = element.style.display !== 'none';
const isDisabled = false;
const isAuthenticated = checkAuth();
const isValid = validateEmail(email);

// has - posesión
const hasPermission = user.role === 'admin';
const hasChildren = node.children.length > 0;
const hasErrors = errors.length > 0;
const hasChanges = isDirty();
const hasActiveSubscription = checkSubscription();

// can - capacidad/permiso
const canEdit = user.isOwner || user.isAdmin;
const canDelete = hasPermission('delete');
const canAccess = checkAccess(resource);
const canSubmit = isValid && !isLoading;

// should - condición/recomendación
const shouldRetry = attempts < MAX_RETRIES;
const shouldCache = !isVolatile(data);
const shouldValidate = isDirty || isFirstSubmit;
const shouldShowWarning = remainingTime < 60;

// was/will - tiempo
const wasSuccessful = response.status === 200;
const wasCanceled = order.status === 'canceled';
const willExpire = expiryDate < futureDate;

// ✅ Evitar negative booleans
const isReady = true;        // ✅ En lugar de notReady
const isFound = true;         // ✅ En lugar de notFound
const isValid = true;         // ✅ En lugar de notValid

if (isReady) {  // ✅ Fácil de leer
  start();
}

// ✅ Uso en condicionales autodocumentados
if (isAuthenticated && hasPermission && canEdit) {
  allowEditing();
}

if (shouldRetry && !hasExceededLimit) {
  retryRequest();
}
```

---

## Ejemplo 5: Convenciones Multi-Lenguaje

### JavaScript/TypeScript
```typescript
// Variables y funciones: camelCase
const userId = 123;
const totalPrice = 99.99;
function getUserData() {}
function calculateTotal() {}

// Clases: PascalCase
class UserController {}
class PaymentService {}

// Interfaces: PascalCase (con o sin I)
interface User {}
interface IPaymentGateway {}

// Types: PascalCase
type UserRole = 'admin' | 'user';
type PaymentStatus = 'pending' | 'completed';

// Enums: PascalCase con valores SCREAMING
enum OrderStatus {
  PENDING = 'PENDING',
  SHIPPED = 'SHIPPED',
  DELIVERED = 'DELIVERED'
}

// Constants: SCREAMING_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = 'https://api.example.com';
const DEFAULT_TIMEOUT_MS = 5000;

// Private fields: # o _
class BankAccount {
  #balance = 0;  // Private field (ES2022)
  _internalId = '';  // Convención antigua
}
```

### Python
```python
# Variables y funciones: snake_case
user_id = 123
total_price = 99.99

def get_user_data():
    pass

def calculate_total_price():
    pass

# Clases: PascalCase
class UserController:
    pass

class PaymentService:
    pass

# Constants: SCREAMING_SNAKE_CASE
MAX_RETRY_COUNT = 3
API_BASE_URL = "https://api.example.com"
DEFAULT_TIMEOUT_MS = 5000

# Private: single underscore
class BankAccount:
    def __init__(self):
        self._balance = 0  # Protected
        self.__account_id = ""  # Private (name mangling)

    def _internal_method(self):  # Protected method
        pass

    def __private_method(self):  # Private method
        pass

# Modules: snake_case
# user_service.py
# payment_processor.py
```

### Java
```java
// Variables y métodos: camelCase
int userId = 123;
double totalPrice = 99.99;

public void getUserData() {}
public void calculateTotal() {}

// Clases: PascalCase
public class UserController {}
public class PaymentService {}

// Interfaces: PascalCase (adjectives)
public interface Readable {}
public interface Serializable {}
public interface PaymentGateway {}

// Constants: SCREAMING_SNAKE_CASE
public static final int MAX_RETRY_COUNT = 3;
public static final String API_BASE_URL = "https://api.example.com";
public static final long DEFAULT_TIMEOUT_MS = 5000L;

// Packages: lowercase
package com.company.product.service;
package com.example.payment.gateway;

// Enums: PascalCase con valores SCREAMING
public enum OrderStatus {
    PENDING,
    SHIPPED,
    DELIVERED
}
```

### C++
```cpp
// Variables: snake_case (Google Style)
int user_id = 123;
double total_price = 99.99;

// Functions: snake_case o CamelCase (consistencia)
void get_user_data() {}
void calculateTotal() {}

// Classes: PascalCase
class UserController {};
class PaymentService {};

// Constants: kConstantName o SCREAMING_SNAKE_CASE
const int kMaxRetryCount = 3;
const char* const kApiBaseUrl = "https://api.example.com";

// Macros: SCREAMING_SNAKE_CASE
#define MAX_BUFFER_SIZE 1024
#define API_VERSION "1.0"

// Namespaces: lowercase
namespace myproject {
namespace utils {
    void helper_function() {}
}
}

// Member variables: trailing underscore (Google)
class BankAccount {
private:
    double balance_;
    std::string account_id_;
};
```

### CSS/SCSS
```css
/* Classes: kebab-case */
.user-card {}
.nav-item {}
.button-primary {}

/* IDs: kebab-case */
#main-content {}
#sidebar-nav {}

/* BEM notation */
.block {}
.block__element {}
.block--modifier {}
.block__element--modifier {}

.card {}
.card__header {}
.card__body {}
.card--featured {}

/* SCSS variables: kebab-case */
$primary-color: #007bff;
$spacing-lg: 2rem;
$font-family-sans: system-ui, sans-serif;

/* CSS custom properties: kebab-case */
:root {
  --color-primary: #007bff;
  --spacing-lg: 2rem;
  --font-sans: system-ui, sans-serif;
}
```

### SQL/Database
```sql
-- Tables: snake_case, plural
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id),
    product_id INTEGER REFERENCES products(product_id),
    quantity INTEGER NOT NULL
);

-- Columns: snake_case
-- user_id, created_at, total_price

-- Indexes: idx_ prefix
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Constraints: naming convention
ALTER TABLE orders
ADD CONSTRAINT fk_orders_user_id
FOREIGN KEY (user_id) REFERENCES users(user_id);

ALTER TABLE users
ADD CONSTRAINT uk_users_email UNIQUE (email);
```

---

## 🎯 Patrones de Refactoring

### Rename Method
```javascript
// ❌ BEFORE
function proc(u) {
  return u.n + ' ' + u.l;
}

// ✅ AFTER
function formatUserFullName(user) {
  return user.firstName + ' ' + user.lastName;
}
```

### Extract Variable
```javascript
// ❌ BEFORE
if (order.total > 100 && order.items.length > 5 && order.customer.isPremium) {
  applyDiscount(order);
}

// ✅ AFTER
const isLargeOrder = order.total > 100;
const hasMultipleItems = order.items.length > 5;
const isPremiumCustomer = order.customer.isPremium;

if (isLargeOrder && hasMultipleItems && isPremiumCustomer) {
  applyDiscount(order);
}
```

---

**Versión:** 1.0.0 | **Última actualización:** 2025-10-24
