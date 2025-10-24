# Message Queue Architect - Examples

---

## Example: Event-Driven Order Processing

```javascript
// Kafka - Order processing pipeline

// Order service publishes event
async function createOrder(order) {
  const orderId = uuid.v4();

  await db.orders.insert({ ...order, id: orderId });

  await producer.send({
    topic: 'orders',
    messages: [{
      key: orderId,
      value: JSON.stringify({
        type: 'ORDER_CREATED',
        orderId,
        customerId: order.customerId,
        items: order.items,
        total: order.total,
        timestamp: new Date().toISOString()
      })
    }]
  });

  return orderId;
}

// Inventory service consumes and updates stock
await consumer.subscribe({ topic: 'orders' });

await consumer.run({
  eachMessage: async ({ message }) => {
    const event = JSON.parse(message.value.toString());

    if (event.type === 'ORDER_CREATED') {
      // Reserve inventory
      await reserveInventory(event.items);

      // Publish inventory reserved event
      await producer.send({
        topic: 'inventory',
        messages: [{
          value: JSON.stringify({
            type: 'INVENTORY_RESERVED',
            orderId: event.orderId
          })
        }]
      });
    }
  }
});
```

---

## Example: RabbitMQ Task Distribution

```javascript
// Worker pool with RabbitMQ

// Producer
async function submitJob(jobData) {
  await channel.assertQueue('jobs', {
    durable: true,
    arguments: {
      'x-max-priority': 10
    }
  });

  channel.sendToQueue(
    'jobs',
    Buffer.from(JSON.stringify(jobData)),
    {
      persistent: true,
      priority: jobData.priority || 5
    }
  );
}

// Worker
async function startWorker(workerId) {
  await channel.assertQueue('jobs', { durable: true });

  channel.prefetch(1);  // One job at a time

  channel.consume('jobs', async (msg) => {
    const job = JSON.parse(msg.content.toString());

    console.log(`Worker ${workerId} processing job ${job.id}`);

    try {
      await processJob(job);
      channel.ack(msg);
    } catch (error) {
      console.error(`Worker ${workerId} failed:`, error);
      channel.nack(msg, false, false);  // Send to DLQ
    }
  });
}
```

---

**Versión:** 1.0.0
