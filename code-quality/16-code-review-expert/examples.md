# Code Review Examples

## Example 1: Security Issue

### ❌ Code Being Reviewed
```javascript
app.get('/user/:id', (req, res) => {
  const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
  db.query(query, (err, results) => {
    res.json(results[0]);
  });
});
```

### 💬 Review Comment
```markdown
🚨 CRITICAL: SQL Injection Vulnerability

**Issue:** The code constructs SQL query using string interpolation with user input, making it vulnerable to SQL injection attacks.

**Why it matters:** An attacker could inject malicious SQL (e.g., `1 OR 1=1`) to access unauthorized data.

**Suggestion:** Use parameterized queries to prevent SQL injection:

\`\`\`javascript
app.get('/user/:id', async (req, res) => {
  const query = 'SELECT * FROM users WHERE id = ?';
  const results = await db.query(query, [req.params.id]);
  res.json(results[0]);
});
\`\`\`

**Reference:** https://owasp.org/www-community/attacks/SQL_Injection
```

---

## Example 2: Logic Error

### ❌ Code Being Reviewed
```javascript
function calculateDiscount(price, discountPercent) {
  return price - (price * discountPercent);
}

// Usage: calculateDiscount(100, 20) // Expected: 80, Actual: 1980
```

### 💬 Review Comment
```markdown
⚠️ Important: Incorrect Discount Calculation

**Issue:** The function treats `discountPercent` as a whole number but doesn't divide by 100.

**Current behavior:**
- `calculateDiscount(100, 20)` returns `1980` (100 - 100*20)
- Expected: `80` (20% off $100)

**Suggestion:** Divide discount by 100 or document that it expects decimal:

\`\`\`javascript
// Option 1: Handle percentage
function calculateDiscount(price, discountPercent) {
  return price - (price * (discountPercent / 100));
}

// Option 2: Use decimal and rename
function calculateDiscount(price, discountRate) {
  // discountRate: 0.2 for 20% discount
  return price - (price * discountRate);
}
\`\`\`

**Test cases needed:**
- `calculateDiscount(100, 20)` should return `80`
- `calculateDiscount(50, 0)` should return `50`
- `calculateDiscount(100, 100)` should return `0`
```

---

## Example 3: Performance Issue

### ❌ Code Being Reviewed
```javascript
async function getUsersWithOrders(userIds) {
  const users = [];

  for (const userId of userIds) {
    const user = await db.users.findById(userId);
    const orders = await db.orders.find({ userId });
    users.push({ ...user, orders });
  }

  return users;
}
```

### 💬 Review Comment
```markdown
💡 Suggestion: N+1 Query Problem

**Issue:** This code makes `2 * userIds.length` database queries, which will be slow for large datasets.

**Why it matters:**
- For 100 users, this makes 200 database queries
- Each query has network latency
- Can cause timeout issues

**Suggestion:** Use batch queries for better performance:

\`\`\`javascript
async function getUsersWithOrders(userIds) {
  // Fetch all users in one query
  const users = await db.users.find({ id: { $in: userIds } });

  // Fetch all orders in one query
  const orders = await db.orders.find({ userId: { $in: userIds } });

  // Group orders by userId
  const ordersByUser = orders.reduce((acc, order) => {
    if (!acc[order.userId]) acc[order.userId] = [];
    acc[order.userId].push(order);
    return acc;
  }, {});

  // Combine data
  return users.map(user => ({
    ...user,
    orders: ordersByUser[user.id] || []
  }));
}
\`\`\`

**Performance impact:**
- Before: `O(n)` queries
- After: `O(1)` queries (just 2 queries regardless of user count)

**Reference:** https://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem
```

---

## Example 4: Missing Error Handling

### ❌ Code Being Reviewed
```javascript
async function processPayment(orderId, amount) {
  const order = await db.orders.findById(orderId);
  const result = await paymentGateway.charge(order.userId, amount);
  await db.orders.update(orderId, { status: 'paid' });
  return result;
}
```

### 💬 Review Comment
```markdown
⚠️ Important: Missing Error Handling and Validation

**Issues identified:**
1. No null check for order
2. No error handling for payment failure
3. Order marked as paid even if payment fails
4. No validation for amount

**Suggestions:**

\`\`\`javascript
async function processPayment(orderId, amount) {
  // 1. Validate input
  if (amount <= 0) {
    throw new Error('Invalid amount');
  }

  // 2. Check order exists
  const order = await db.orders.findById(orderId);
  if (!order) {
    throw new Error(`Order ${orderId} not found`);
  }

  // 3. Check order not already paid
  if (order.status === 'paid') {
    throw new Error('Order already paid');
  }

  try {
    // 4. Attempt payment
    const result = await paymentGateway.charge(order.userId, amount);

    // 5. Update order only if payment succeeds
    await db.orders.update(orderId, {
      status: 'paid',
      paidAt: new Date(),
      transactionId: result.transactionId
    });

    return result;
  } catch (error) {
    // 6. Log payment failure
    logger.error('Payment failed', { orderId, amount, error });

    // 7. Mark order as failed
    await db.orders.update(orderId, {
      status: 'payment_failed',
      failureReason: error.message
    });

    throw new Error('Payment processing failed');
  }
}
\`\`\`

**Test cases to add:**
- Valid payment
- Order not found
- Order already paid
- Payment gateway failure
- Invalid amount
```

---

## Example 5: Good Code (Positive Feedback)

### ✅ Code Being Reviewed
```javascript
async function createUser(userData) {
  // Validate input
  const schema = z.object({
    email: z.string().email(),
    name: z.string().min(1),
    age: z.number().int().positive().optional(),
  });

  const validData = schema.parse(userData);

  // Check if user exists
  const existing = await db.users.findOne({ email: validData.email });
  if (existing) {
    throw new Error('User already exists');
  }

  // Hash password
  const hashedPassword = await bcrypt.hash(validData.password, 10);

  // Create user
  const user = await db.users.create({
    ...validData,
    password: hashedPassword,
    createdAt: new Date(),
  });

  // Don't return password
  const { password, ...userWithoutPassword } = user;
  return userWithoutPassword;
}
```

### 💬 Review Comment
```markdown
✅ Excellent implementation!

**What I like:**
1. ✅ Input validation with Zod
2. ✅ Duplicate email check
3. ✅ Password hashing with bcrypt
4. ✅ Password excluded from response
5. ✅ Clear error messages

**Minor suggestion:**
Consider adding a try-catch for database errors and logging:

\`\`\`javascript
try {
  const user = await db.users.create({...});
  // ...
} catch (error) {
  logger.error('User creation failed', { email: validData.email, error });
  throw new Error('Failed to create user');
}
\`\`\`

Great work! 🎉
```
