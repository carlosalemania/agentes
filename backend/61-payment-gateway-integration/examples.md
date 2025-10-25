# Payment Gateway Examples

## Example: Complete Stripe Checkout

```javascript
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const express = require('express');
const app = express();

app.post('/create-payment-intent', async (req, res) => {
  const { amount, currency } = req.body;

  const paymentIntent = await stripe.paymentIntents.create({
    amount: Math.round(amount * 100),
    currency,
    automatic_payment_methods: { enabled: true },
  });

  res.json({ clientSecret: paymentIntent.client_secret });
});

// Frontend
const stripe = Stripe('pk_test_...');
const elements = stripe.elements();
const paymentElement = elements.create('payment');
paymentElement.mount('#payment-element');

async function handleSubmit(e) {
  e.preventDefault();

  const {error} = await stripe.confirmPayment({
    elements,
    confirmParams: {
      return_url: 'https://example.com/success',
    },
  });

  if (error) {
    showError(error.message);
  }
}
```

## Example: Subscription with Trial

```javascript
app.post('/create-subscription', async (req, res) => {
  const { customerId, priceId } = req.body;

  const subscription = await stripe.subscriptions.create({
    customer: customerId,
    items: [{ price: priceId }],
    trial_period_days: 14,
    payment_settings: {
      save_default_payment_method: 'on_subscription',
    },
  });

  res.json(subscription);
});
```
