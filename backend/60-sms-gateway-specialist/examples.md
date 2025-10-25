# SMS Gateway Examples

## Example: Complete OTP System

```javascript
const twilio = require('twilio');
const { Queue, Worker } = require('bullmq');

const client = twilio(
  process.env.TWILIO_ACCOUNT_SID,
  process.env.TWILIO_AUTH_TOKEN
);

class OTPSystem {
  constructor() {
    this.queue = new Queue('sms', {
      connection: { host: 'localhost', port: 6379 },
    });

    this.initWorker();
  }

  initWorker() {
    const worker = new Worker(
      'sms',
      async (job) => {
        const { phoneNumber, message } = job.data;

        await client.messages.create({
          body: message,
          from: process.env.TWILIO_PHONE_NUMBER,
          to: phoneNumber,
        });
      },
      {
        connection: { host: 'localhost', port: 6379 },
        concurrency: 5,
      }
    );
  }

  generateOTP(length = 6) {
    return Math.floor(
      Math.random() * (10 ** length - 10 ** (length - 1)) + 10 ** (length - 1)
    ).toString();
  }

  async sendOTP(phoneNumber) {
    // Check rate limit
    const recentAttempts = await db.otpAttempts.count({
      phoneNumber,
      createdAt: { $gte: new Date(Date.now() - 60000) },
    });

    if (recentAttempts >= 3) {
      throw new Error('Too many attempts. Please wait 1 minute.');
    }

    const otp = this.generateOTP();
    const expiresAt = new Date(Date.now() + 10 * 60 * 1000); // 10 min

    // Store OTP
    await db.otpCodes.create({
      phoneNumber,
      code: otp,
      expiresAt,
      verified: false,
      attempts: 0,
    });

    // Queue SMS
    await this.queue.add('send', {
      phoneNumber,
      message: `Your verification code is: ${otp}\n\nValid for 10 minutes.`,
    });

    // Log attempt
    await db.otpAttempts.create({
      phoneNumber,
      createdAt: new Date(),
    });

    return { success: true, expiresAt };
  }

  async verifyOTP(phoneNumber, code) {
    const otpRecord = await db.otpCodes.findOne({
      phoneNumber,
      code,
      verified: false,
      expiresAt: { $gt: new Date() },
    });

    if (!otpRecord) {
      // Increment attempts
      await db.otpCodes.updateMany(
        {
          phoneNumber,
          verified: false,
          expiresAt: { $gt: new Date() },
        },
        { $inc: { attempts: 1 } }
      );

      return { valid: false, reason: 'Invalid or expired code' };
    }

    if (otpRecord.attempts >= 5) {
      return { valid: false, reason: 'Too many attempts' };
    }

    // Mark verified
    await db.otpCodes.updateOne(
      { _id: otpRecord._id },
      {
        verified: true,
        verifiedAt: new Date(),
      }
    );

    return { valid: true };
  }
}

// API endpoints
const express = require('express');
const app = express();
app.use(express.json());

const otpSystem = new OTPSystem();

app.post('/api/otp/send', async (req, res) => {
  const { phoneNumber } = req.body;

  try {
    const result = await otpSystem.sendOTP(phoneNumber);
    res.json(result);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.post('/api/otp/verify', async (req, res) => {
  const { phoneNumber, code } = req.body;

  const result = await otpSystem.verifyOTP(phoneNumber, code);

  if (!result.valid) {
    return res.status(400).json(result);
  }

  res.json({ success: true });
});
```

## Example: Two-Factor Authentication Flow

```javascript
// Complete 2FA implementation
class TwoFactorService {
  constructor(otpSystem) {
    this.otpSystem = otpSystem;
  }

  // Step 1: User enables 2FA
  async initiate2FA(userId, phoneNumber) {
    // Validate phone number
    const { isValid, phoneNumber: formatted } = this.validatePhone(phoneNumber);

    if (!isValid) {
      throw new Error('Invalid phone number');
    }

    // Send verification code
    await this.otpSystem.sendOTP(formatted);

    // Update user record
    await db.users.update(
      { _id: userId },
      {
        twoFactorPhone: formatted,
        twoFactorStatus: 'pending',
      }
    );

    return { success: true };
  }

  // Step 2: Verify phone and enable 2FA
  async enable2FA(userId, code) {
    const user = await db.users.findById(userId);

    if (user.twoFactorStatus !== 'pending') {
      throw new Error('2FA setup not initiated');
    }

    const result = await this.otpSystem.verifyOTP(user.twoFactorPhone, code);

    if (!result.valid) {
      return result;
    }

    // Generate backup codes
    const backupCodes = this.generateBackupCodes();

    await db.users.update(
      { _id: userId },
      {
        twoFactorEnabled: true,
        twoFactorStatus: 'active',
        twoFactorBackupCodes: backupCodes.map((code) =>
          bcrypt.hashSync(code, 10)
        ),
      }
    );

    return {
      success: true,
      backupCodes, // Show once to user
    };
  }

  // Step 3: Login with 2FA
  async send2FACode(userId) {
    const user = await db.users.findById(userId);

    if (!user.twoFactorEnabled) {
      throw new Error('2FA not enabled');
    }

    return await this.otpSystem.sendOTP(user.twoFactorPhone);
  }

  async verify2FALogin(userId, code) {
    const user = await db.users.findById(userId);

    // Try OTP first
    const otpResult = await this.otpSystem.verifyOTP(user.twoFactorPhone, code);

    if (otpResult.valid) {
      return { valid: true, method: 'otp' };
    }

    // Try backup codes
    for (const hashedCode of user.twoFactorBackupCodes) {
      if (bcrypt.compareSync(code, hashedCode)) {
        // Remove used backup code
        await db.users.update(
          { _id: userId },
          {
            $pull: { twoFactorBackupCodes: hashedCode },
          }
        );

        return { valid: true, method: 'backup' };
      }
    }

    return { valid: false, reason: 'Invalid code' };
  }

  generateBackupCodes(count = 10) {
    const codes = [];

    for (let i = 0; i < count; i++) {
      const code = Math.random().toString(36).substr(2, 8).toUpperCase();
      codes.push(code);
    }

    return codes;
  }

  validatePhone(phoneNumber) {
    const libphonenumber = require('libphonenumber-js');

    try {
      const parsed = libphonenumber.parsePhoneNumber(phoneNumber);

      return {
        isValid: parsed.isValid(),
        phoneNumber: parsed.format('E.164'),
        country: parsed.country,
      };
    } catch {
      return { isValid: false };
    }
  }
}

// Express routes
app.post('/api/2fa/enable', async (req, res) => {
  const { phoneNumber } = req.body;
  const userId = req.user.id;

  const service = new TwoFactorService(otpSystem);

  try {
    const result = await service.initiate2FA(userId, phoneNumber);
    res.json(result);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.post('/api/2fa/verify-enable', async (req, res) => {
  const { code } = req.body;
  const userId = req.user.id;

  const service = new TwoFactorService(otpSystem);

  const result = await service.enable2FA(userId, code);

  if (!result.success) {
    return res.status(400).json(result);
  }

  res.json(result);
});

app.post('/api/login/2fa', async (req, res) => {
  const { code } = req.body;
  const userId = req.session.pendingUserId;

  const service = new TwoFactorService(otpSystem);

  const result = await service.verify2FALogin(userId, code);

  if (!result.valid) {
    return res.status(400).json(result);
  }

  // Complete login
  req.session.userId = userId;
  delete req.session.pendingUserId;

  res.json({ success: true });
});
```

## Example: SMS Analytics Dashboard

```javascript
// SMS analytics and reporting
class SMSAnalytics {
  async getDashboard(dateRange) {
    const { start, end } = dateRange;

    const [stats, byCountry, byStatus, timeline] = await Promise.all([
      this.getOverallStats(start, end),
      this.getStatsByCountry(start, end),
      this.getStatsByStatus(start, end),
      this.getTimeline(start, end),
    ]);

    return {
      stats,
      byCountry,
      byStatus,
      timeline,
    };
  }

  async getOverallStats(start, end) {
    const result = await db.smsLogs.aggregate([
      {
        $match: {
          sentAt: { $gte: start, $lte: end },
        },
      },
      {
        $group: {
          _id: null,
          total: { $sum: 1 },
          delivered: {
            $sum: {
              $cond: [{ $eq: ['$status', 'delivered'] }, 1, 0],
            },
          },
          failed: {
            $sum: {
              $cond: [{ $eq: ['$status', 'failed'] }, 1, 0],
            },
          },
          totalCost: {
            $sum: { $toDouble: { $ifNull: ['$price', 0] } },
          },
        },
      },
    ]);

    const data = result[0] || { total: 0, delivered: 0, failed: 0, totalCost: 0 };

    return {
      ...data,
      deliveryRate: ((data.delivered / data.total) * 100).toFixed(2) + '%',
      avgCost: (data.totalCost / data.total).toFixed(4),
    };
  }

  async getStatsByCountry(start, end) {
    return await db.smsLogs.aggregate([
      {
        $match: {
          sentAt: { $gte: start, $lte: end },
        },
      },
      {
        $group: {
          _id: '$country',
          count: { $sum: 1 },
          cost: { $sum: { $toDouble: { $ifNull: ['$price', 0] } } },
        },
      },
      {
        $sort: { cost: -1 },
      },
      {
        $limit: 10,
      },
    ]);
  }

  async getTimeline(start, end) {
    return await db.smsLogs.aggregate([
      {
        $match: {
          sentAt: { $gte: start, $lte: end },
        },
      },
      {
        $group: {
          _id: {
            $dateToString: {
              format: '%Y-%m-%d',
              date: '$sentAt',
            },
          },
          count: { $sum: 1 },
          delivered: {
            $sum: {
              $cond: [{ $eq: ['$status', 'delivered'] }, 1, 0],
            },
          },
        },
      },
      {
        $sort: { _id: 1 },
      },
    ]);
  }
}
```
