# Payment Gateway Checklist

## Integration
- [ ] Stripe/PayPal SDK installed
- [ ] API keys configured (test & production)
- [ ] Webhooks endpoint created
- [ ] Webhook signatures verified
- [ ] Error handling implemented

## Payment Flow
- [ ] Payment intent creation
- [ ] Client-side integration (Stripe Elements)
- [ ] 3D Secure (SCA) handling
- [ ] Success/failure handling
- [ ] Receipt generation

## Subscriptions
- [ ] Plan/price creation
- [ ] Subscription creation
- [ ] Trial period handling
- [ ] Upgrade/downgrade flow
- [ ] Cancellation flow
- [ ] Proration logic

## Security
- [ ] PCI compliance verified
- [ ] No card data stored
- [ ] HTTPS enforced
- [ ] CSRF protection
- [ ] Rate limiting

## Refunds & Disputes
- [ ] Refund processing
- [ ] Partial refunds
- [ ] Dispute handling
- [ ] Chargeback tracking

## Monitoring
- [ ] Transaction logging
- [ ] Failed payment alerts
- [ ] Revenue tracking
- [ ] Analytics dashboard
