# Email Service Examples

## Example: Complete Email System

```javascript
// Email service with queue and templates
const { Queue, Worker } = require('bullmq');
const sgMail = require('@sendgrid/mail');
const handlebars = require('handlebars');
const fs = require('fs').promises;

sgMail.setApiKey(process.env.SENDGRID_API_KEY);

class EmailSystem {
  constructor() {
    this.queue = new Queue('emails', {
      connection: { host: 'localhost', port: 6379 },
    });

    this.templates = new Map();
    this.initWorker();
  }

  async loadTemplates() {
    const templates = [
      { name: 'welcome', path: './templates/welcome.hbs' },
      { name: 'reset-password', path: './templates/reset-password.hbs' },
      { name: 'order-confirmation', path: './templates/order-confirmation.hbs' },
    ];

    for (const { name, path } of templates) {
      const source = await fs.readFile(path, 'utf-8');
      this.templates.set(name, handlebars.compile(source));
    }
  }

  initWorker() {
    const worker = new Worker(
      'emails',
      async (job) => {
        const { type, to, data } = job.data;

        switch (type) {
          case 'transactional':
            await this.sendTransactional(to, data);
            break;

          case 'marketing':
            await this.sendMarketing(to, data);
            break;

          default:
            throw new Error(`Unknown email type: ${type}`);
        }
      },
      {
        connection: { host: 'localhost', port: 6379 },
        concurrency: 5,
      }
    );

    worker.on('completed', (job) => {
      console.log(`Email ${job.id} sent`);
    });

    worker.on('failed', (job, err) => {
      console.error(`Email ${job.id} failed:`, err);
    });
  }

  async sendTransactional(to, { template, data }) {
    const html = this.renderTemplate(template, data);

    const msg = {
      to,
      from: 'noreply@example.com',
      subject: data.subject,
      html,
      trackingSettings: {
        clickTracking: { enable: true },
        openTracking: { enable: true },
      },
    };

    await sgMail.send(msg);

    await db.emailLogs.create({
      to,
      template,
      status: 'sent',
      sentAt: new Date(),
    });
  }

  async sendMarketing(to, { campaignId, template, data }) {
    // Check suppression list
    const suppressed = await db.emailSuppressions.findOne({ email: to });

    if (suppressed) {
      throw new Error('Email suppressed');
    }

    const html = this.renderTemplate(template, data);

    const msg = {
      to,
      from: 'marketing@example.com',
      subject: data.subject,
      html,
      customArgs: {
        campaignId,
      },
    };

    await sgMail.send(msg);
  }

  renderTemplate(name, data) {
    const template = this.templates.get(name);

    if (!template) {
      throw new Error(`Template ${name} not found`);
    }

    return template(data);
  }

  // Queue emails
  async queueWelcomeEmail(userId, email) {
    await this.queue.add('send', {
      type: 'transactional',
      to: email,
      data: {
        template: 'welcome',
        subject: 'Welcome to Example App!',
        data: { userId, email },
      },
    });
  }

  async queuePasswordReset(email, resetToken) {
    await this.queue.add('send', {
      type: 'transactional',
      to: email,
      data: {
        template: 'reset-password',
        subject: 'Reset Your Password',
        data: {
          resetUrl: `https://example.com/reset?token=${resetToken}`,
          expiresIn: '1 hour',
        },
      },
    });
  }

  async queueOrderConfirmation(email, order) {
    await this.queue.add('send', {
      type: 'transactional',
      to: email,
      data: {
        template: 'order-confirmation',
        subject: `Order Confirmation #${order.id}`,
        data: {
          orderId: order.id,
          items: order.items,
          total: order.total,
          shippingAddress: order.shippingAddress,
        },
      },
    });
  }
}

// Usage
const emailSystem = new EmailSystem();
await emailSystem.loadTemplates();

// Send emails
await emailSystem.queueWelcomeEmail('123', 'user@example.com');
await emailSystem.queuePasswordReset('user@example.com', 'reset-token-123');
```

## Example: Email Template (Handlebars)

```handlebars
<!-- templates/welcome.hbs -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body {
      font-family: Arial, sans-serif;
      line-height: 1.6;
      color: #333;
      max-width: 600px;
      margin: 0 auto;
      padding: 20px;
    }
    .header {
      background: #007bff;
      color: white;
      padding: 20px;
      text-align: center;
    }
    .content {
      padding: 20px;
      background: #f9f9f9;
    }
    .button {
      display: inline-block;
      padding: 12px 24px;
      background: #007bff;
      color: white;
      text-decoration: none;
      border-radius: 4px;
      margin: 20px 0;
    }
    .footer {
      text-align: center;
      color: #666;
      font-size: 12px;
      margin-top: 20px;
    }
  </style>
</head>
<body>
  <div class="header">
    <h1>Welcome to Example App!</h1>
  </div>

  <div class="content">
    <p>Hi {{name}},</p>

    <p>Thanks for signing up! We're excited to have you on board.</p>

    <p>Get started by exploring our features:</p>

    <a href="{{appUrl}}" class="button">Get Started</a>

    <p>If you have any questions, feel free to reach out to our support team.</p>

    <p>Best regards,<br>The Example App Team</p>
  </div>

  <div class="footer">
    <p>This email was sent to {{email}}</p>
    <p>
      <a href="{{unsubscribeUrl}}">Unsubscribe</a> |
      <a href="{{privacyUrl}}">Privacy Policy</a>
    </p>
  </div>
</body>
</html>
```

## Example: Webhook Handler with Express

```javascript
// Handle SendGrid webhooks
const express = require('express');
const crypto = require('crypto');

const app = express();

// Verify SendGrid webhook signature
function verifySendGridSignature(req, res, next) {
  const publicKey = process.env.SENDGRID_WEBHOOK_PUBLIC_KEY;
  const signature = req.headers['x-twilio-email-event-webhook-signature'];
  const timestamp = req.headers['x-twilio-email-event-webhook-timestamp'];

  if (!signature || !timestamp) {
    return res.status(401).send('Missing signature headers');
  }

  const payload = timestamp + JSON.stringify(req.body);
  const verify = crypto.createVerify('RSA-SHA256');
  verify.update(payload);

  if (!verify.verify(publicKey, signature, 'base64')) {
    return res.status(401).send('Invalid signature');
  }

  next();
}

app.post(
  '/webhooks/sendgrid',
  express.json(),
  verifySendGridSignature,
  async (req, res) => {
    const events = req.body;

    for (const event of events) {
      try {
        await processEmailEvent(event);
      } catch (error) {
        console.error('Error processing event:', error);
      }
    }

    res.status(200).send('OK');
  }
);

async function processEmailEvent(event) {
  const {
    email,
    event: eventType,
    sg_message_id,
    timestamp,
    campaignId,
  } = event;

  // Log event
  await db.emailEvents.create({
    email,
    eventType,
    messageId: sg_message_id,
    campaignId,
    timestamp: new Date(timestamp * 1000),
    metadata: event,
  });

  // Handle specific events
  switch (eventType) {
    case 'bounce':
      await handleBounce(email, event);
      break;

    case 'dropped':
      console.log(`Email to ${email} was dropped:`, event.reason);
      break;

    case 'spam_report':
      await handleSpamReport(email);
      break;

    case 'unsubscribe':
      await handleUnsubscribe(email);
      break;

    case 'open':
      await incrementOpenCount(email, campaignId);
      break;

    case 'click':
      await trackClick(email, campaignId, event.url);
      break;
  }
}

async function handleBounce(email, event) {
  const isSoftBounce = event.type === 'soft';

  if (!isSoftBounce) {
    // Hard bounce - suppress email
    await db.emailSuppressions.create({
      email,
      reason: 'hard_bounce',
      metadata: event,
      createdAt: new Date(),
    });

    await db.users.update({ email }, { emailStatus: 'bounced' });
  }
}

async function handleSpamReport(email) {
  await db.emailSuppressions.create({
    email,
    reason: 'spam_report',
    createdAt: new Date(),
  });

  await db.users.update({ email }, { emailOptIn: false });
}

async function handleUnsubscribe(email) {
  await db.users.update(
    { email },
    {
      emailOptIn: false,
      unsubscribedAt: new Date(),
    }
  );
}

app.listen(3000);
```
