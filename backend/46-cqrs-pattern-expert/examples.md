# CQRS Examples

## Example 1: E-commerce Order System

```javascript
// Commands
class CreateOrderCommand {
  constructor(customerId, items) {
    this.orderId = generateId();
    this.customerId = customerId;
    this.items = items;
  }
}

class CreateOrderHandler {
  async handle(command) {
    // Validate
    const customer = await customerRepo.findById(command.customerId);
    if (!customer) throw new Error('Customer not found');

    // Check inventory
    for (const item of command.items) {
      const product = await productRepo.findById(item.productId);
      if (product.stock < item.quantity) {
        throw new Error(`Insufficient stock for ${product.name}`);
      }
    }

    // Create order (write model)
    const order = await orderWriteRepo.create({
      id: command.orderId,
      customerId: command.customerId,
      status: 'pending',
      createdAt: new Date(),
    });

    // Reduce inventory
    for (const item of command.items) {
      await productRepo.reduceStock(item.productId, item.quantity);
    }

    // Publish event
    await eventBus.publish({
      type: 'OrderCreated',
      data: {
        orderId: order.id,
        customerId: command.customerId,
        items: command.items,
        total: calculateTotal(command.items),
        createdAt: order.createdAt,
      },
    });

    return order.id;
  }
}

// Queries
class GetOrderDetailsQuery {
  constructor(orderId) {
    this.orderId = orderId;
  }
}

class GetOrderDetailsHandler {
  async handle(query) {
    // Read from denormalized read model
    return await orderReadRepo.findById(query.orderId);
  }
}

// Read model projection
class OrderProjection {
  async onOrderCreated(event) {
    const { orderId, customerId, items, total, createdAt } = event.data;

    // Denormalize with customer and product details
    const customer = await customerService.getById(customerId);
    const itemsWithDetails = await Promise.all(
      items.map(async (item) => {
        const product = await productService.getById(item.productId);
        return {
          productId: item.productId,
          name: product.name,
          price: item.price,
          quantity: item.quantity,
          subtotal: item.price * item.quantity,
        };
      })
    );

    // Insert to read model (e.g., MongoDB)
    await orderReadRepo.insert({
      _id: orderId,
      customerId,
      customerName: customer.name,
      customerEmail: customer.email,
      items: itemsWithDetails,
      total,
      status: 'pending',
      createdAt,
    });
  }

  async onOrderPaid(event) {
    await orderReadRepo.update(
      { _id: event.data.orderId },
      { $set: { status: 'paid', paidAt: event.data.paidAt } }
    );
  }
}
```

---

## Example 2: Banking Account with CQRS

```javascript
// Commands
class DepositCommand {
  constructor(accountId, amount) {
    this.accountId = accountId;
    this.amount = amount;
  }
}

class DepositHandler {
  async handle(command) {
    if (command.amount <= 0) {
      throw new Error('Amount must be positive');
    }

    // Load aggregate
    const account = await accountRepo.findById(command.accountId);
    if (!account) throw new Error('Account not found');

    // Update balance (write model)
    await accountRepo.updateBalance(account.id, account.balance + command.amount);

    // Publish event
    await eventBus.publish({
      type: 'MoneyDeposited',
      data: {
        accountId: account.id,
        amount: command.amount,
        newBalance: account.balance + command.amount,
        timestamp: new Date(),
      },
    });
  }
}

class WithdrawCommand {
  constructor(accountId, amount) {
    this.accountId = accountId;
    this.amount = amount;
  }
}

class WithdrawHandler {
  async handle(command) {
    const account = await accountRepo.findById(command.accountId);

    if (account.balance < command.amount) {
      throw new Error('Insufficient funds');
    }

    await accountRepo.updateBalance(account.id, account.balance - command.amount);

    await eventBus.publish({
      type: 'MoneyWithdrawn',
      data: {
        accountId: account.id,
        amount: command.amount,
        newBalance: account.balance - command.amount,
        timestamp: new Date(),
      },
    });
  }
}

// Queries
class GetAccountBalanceQuery {
  constructor(accountId) {
    this.accountId = accountId;
  }
}

class GetAccountBalanceHandler {
  async handle(query) {
    // Read from optimized read model
    const account = await accountReadRepo.findById(query.accountId);
    return {
      accountId: account._id,
      balance: account.balance,
      currency: account.currency,
      lastUpdated: account.lastUpdated,
    };
  }
}

class GetTransactionHistoryQuery {
  constructor(accountId, pagination) {
    this.accountId = accountId;
    this.pagination = pagination;
  }
}

class GetTransactionHistoryHandler {
  async handle(query) {
    // Read from denormalized transaction log
    return await transactionReadRepo.find(
      { accountId: query.accountId },
      {
        sort: { timestamp: -1 },
        skip: query.pagination.skip,
        limit: query.pagination.limit,
      }
    );
  }
}

// Projections
class AccountBalanceProjection {
  async onMoneyDeposited(event) {
    await accountReadRepo.update(
      { _id: event.data.accountId },
      {
        $set: {
          balance: event.data.newBalance,
          lastUpdated: event.data.timestamp,
        },
      }
    );
  }

  async onMoneyWithdrawn(event) {
    await accountReadRepo.update(
      { _id: event.data.accountId },
      {
        $set: {
          balance: event.data.newBalance,
          lastUpdated: event.data.timestamp,
        },
      }
    );
  }
}

class TransactionHistoryProjection {
  async onMoneyDeposited(event) {
    await transactionReadRepo.insert({
      accountId: event.data.accountId,
      type: 'deposit',
      amount: event.data.amount,
      balance: event.data.newBalance,
      timestamp: event.data.timestamp,
    });
  }

  async onMoneyWithdrawn(event) {
    await transactionReadRepo.insert({
      accountId: event.data.accountId,
      type: 'withdrawal',
      amount: event.data.amount,
      balance: event.data.newBalance,
      timestamp: event.data.timestamp,
    });
  }
}
```

---

## Example 3: Handling Eventual Consistency

```javascript
// API Controller
class OrderController {
  constructor(commandBus, queryBus) {
    this.commandBus = commandBus;
    this.queryBus = queryBus;
  }

  async createOrder(req, res) {
    try {
      const command = new CreateOrderCommand(req.user.id, req.body.items);

      const orderId = await this.commandBus.execute(command);

      // Return 202 Accepted (not 201 Created)
      // because read model might not be updated yet
      res.status(202).json({
        orderId,
        message: 'Order is being processed',
        status: 'pending',
        _links: {
          self: `/orders/${orderId}`,
          poll: `/orders/${orderId}/status`,
        },
      });
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }

  async getOrder(req, res) {
    const query = new GetOrderDetailsQuery(req.params.id);

    try {
      const order = await this.queryBus.execute(query);

      if (!order) {
        // Check if order exists in write model
        const writeModel = await orderWriteRepo.findById(req.params.id);

        if (writeModel) {
          // Order exists but projection not ready
          return res.status(202).json({
            message: 'Order data is being synchronized',
            status: 'processing',
          });
        }

        return res.status(404).json({ error: 'Order not found' });
      }

      res.json(order);
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }

  // Polling endpoint for eventual consistency
  async getOrderStatus(req, res) {
    const order = await queryBus.execute(new GetOrderDetailsQuery(req.params.id));

    if (order) {
      res.json({
        ready: true,
        order,
      });
    } else {
      res.json({
        ready: false,
        message: 'Order data is still being processed',
      });
    }
  }
}
```

---

## Example 4: Saga Pattern with CQRS

```javascript
// Saga for handling order fulfillment
class OrderFulfillmentSaga {
  async onOrderCreated(event) {
    const { orderId, items } = event.data;

    try {
      // Step 1: Reserve inventory
      await commandBus.execute(new ReserveInventoryCommand(orderId, items));

      // Step 2: Process payment
      await commandBus.execute(new ProcessPaymentCommand(orderId));

      // Step 3: Ship order
      await commandBus.execute(new ShipOrderCommand(orderId));
    } catch (error) {
      // Compensating actions
      await commandBus.execute(new CancelOrderCommand(orderId, error.message));
      await commandBus.execute(new ReleaseInventoryCommand(orderId));
    }
  }
}

// Register saga
eventBus.subscribe('OrderCreated', (event) => saga.onOrderCreated(event));
```
