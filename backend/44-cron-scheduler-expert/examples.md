# Cron/Scheduler Examples

## Example 1: Database Backup Job

```javascript
const cron = require('node-cron');
const { exec } = require('child_process');
const { promisify } = require('util');
const execAsync = promisify(exec);

class DatabaseBackupJob {
  constructor() {
    this.backupDir = '/var/backups/db';
  }

  async run() {
    const timestamp = new Date().toISOString().replace(/:/g, '-');
    const filename = `backup-${timestamp}.sql.gz`;
    const filepath = `${this.backupDir}/${filename}`;

    try {
      console.log('Starting database backup...');

      // Backup database
      await execAsync(
        `pg_dump -U postgres mydb | gzip > ${filepath}`
      );

      console.log(`Backup saved to ${filepath}`);

      // Upload to S3
      await this.uploadToS3(filepath, filename);

      // Cleanup old backups (keep last 7 days)
      await this.cleanupOldBackups(7);

      console.log('Backup completed successfully');
    } catch (error) {
      console.error('Backup failed:', error);
      await this.notifyFailure(error);
      throw error;
    }
  }

  async uploadToS3(filepath, filename) {
    const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
    const fs = require('fs');

    const s3 = new S3Client({ region: 'us-east-1' });

    const fileStream = fs.createReadStream(filepath);

    await s3.send(
      new PutObjectCommand({
        Bucket: 'my-backups',
        Key: `database/${filename}`,
        Body: fileStream,
      })
    );

    console.log(`Backup uploaded to S3: ${filename}`);
  }

  async cleanupOldBackups(daysToKeep) {
    const cutoff = Date.now() - daysToKeep * 24 * 60 * 60 * 1000;
    const files = await fs.promises.readdir(this.backupDir);

    for (const file of files) {
      const filepath = `${this.backupDir}/${file}`;
      const stats = await fs.promises.stat(filepath);

      if (stats.mtimeMs < cutoff) {
        await fs.promises.unlink(filepath);
        console.log(`Deleted old backup: ${file}`);
      }
    }
  }

  async notifyFailure(error) {
    // Send email/Slack notification
    await notifyAdmins({
      subject: 'Database Backup Failed',
      message: error.message,
    });
  }
}

// Schedule: Daily at 2 AM
const backup = new DatabaseBackupJob();

cron.schedule('0 2 * * *', async () => {
  await backup.run();
}, {
  timezone: 'America/New_York'
});
```

---

## Example 2: Email Digest with BullMQ

```javascript
const { Queue, Worker } = require('bullmq');

// Queue setup
const digestQueue = new Queue('email-digest', {
  connection: { host: 'localhost', port: 6379 },
});

// Schedule daily digest at 8 AM
await digestQueue.add(
  'daily-digest',
  {},
  {
    repeat: {
      pattern: '0 8 * * *',
      tz: 'America/New_York',
    },
  }
);

// Worker
const worker = new Worker(
  'email-digest',
  async (job) => {
    console.log('Generating daily digest...');

    // Get active users
    const users = await db.users.find({ is_active: true });

    // Process in batches
    const batchSize = 100;
    for (let i = 0; i < users.length; i += batchSize) {
      const batch = users.slice(i, i + batchSize);

      await Promise.all(
        batch.map(async (user) => {
          const digest = await generateDigest(user.id);
          await sendEmail(user.email, 'Daily Digest', digest);
        })
      );

      console.log(`Sent ${i + batch.length}/${users.length} digests`);
    }

    return { sent: users.length };
  },
  {
    connection: { host: 'localhost', port: 6379 },
    concurrency: 5,
  }
);

worker.on('completed', (job, result) => {
  console.log(`Daily digest completed: ${result.sent} emails sent`);
});

worker.on('failed', (job, error) => {
  console.error('Daily digest failed:', error);
  notifyAdmins({ error });
});

async function generateDigest(userId) {
  const [newPosts, mentions, messages] = await Promise.all([
    db.posts.find({ created_at: { $gte: yesterday() } }).limit(5),
    db.mentions.find({ user_id: userId, read: false }),
    db.messages.find({ recipient_id: userId, read: false }),
  ]);

  return {
    newPosts,
    mentions,
    messages,
    unreadCount: mentions.length + messages.length,
  };
}
```

---

## Example 3: Distributed Job with Leader Election

```javascript
const Redis = require('ioredis');
const Redlock = require('redlock');

class DistributedCronJob {
  constructor(jobName, jobFn, cronPattern) {
    this.jobName = jobName;
    this.jobFn = jobFn;
    this.cronPattern = cronPattern;

    this.redis = new Redis();
    this.redlock = new Redlock([this.redis]);
  }

  start() {
    cron.schedule(this.cronPattern, async () => {
      await this.runWithLock();
    });
  }

  async runWithLock() {
    const lockKey = `lock:cron:${this.jobName}`;
    const lockTTL = 300000; // 5 minutes
    let lock;

    try {
      // Try to acquire lock
      lock = await this.redlock.acquire([lockKey], lockTTL);

      console.log(`[${this.jobName}] Lock acquired, running job`);

      // Extend lock periodically
      const extendInterval = setInterval(async () => {
        try {
          await lock.extend(lockTTL);
          console.log(`[${this.jobName}] Lock extended`);
        } catch (error) {
          console.error(`[${this.jobName}] Failed to extend lock`);
          clearInterval(extendInterval);
        }
      }, lockTTL / 2);

      // Run job
      await this.jobFn();

      clearInterval(extendInterval);
      console.log(`[${this.jobName}] Job completed`);
    } catch (error) {
      if (error instanceof Redlock.LockError) {
        console.log(`[${this.jobName}] Already running on another instance`);
      } else {
        console.error(`[${this.jobName}] Job failed:`, error);
        throw error;
      }
    } finally {
      if (lock) {
        await lock.release();
        console.log(`[${this.jobName}] Lock released`);
      }
    }
  }

  stop() {
    this.redis.disconnect();
  }
}

// Usage
const cleanupJob = new DistributedCronJob(
  'cleanup-old-data',
  async () => {
    await db.logs.deleteMany({
      created_at: { $lt: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) },
    });
    console.log('Old data cleaned up');
  },
  '0 3 * * *' // 3 AM daily
);

cleanupJob.start();

// Graceful shutdown
process.on('SIGTERM', () => {
  cleanupJob.stop();
});
```

---

## Example 4: Dynamic Scheduling with Agenda

```javascript
const Agenda = require('agenda');

const agenda = new Agenda({
  db: { address: 'mongodb://localhost/agenda' },
});

// Define job types
agenda.define('send reminder', async (job) => {
  const { userId, message, type } = job.attrs.data;

  await sendNotification(userId, message, type);
  console.log(`Reminder sent to user ${userId}`);
});

agenda.define('subscription expiry warning', async (job) => {
  const { userId } = job.attrs.data;

  const user = await db.users.findById(userId);

  if (user.subscription_expires_at > new Date()) {
    await sendEmail(user.email, 'Subscription Expiring Soon', {
      expiresAt: user.subscription_expires_at,
    });
  }
});

await agenda.start();

// API endpoint to schedule reminders
app.post('/reminders', async (req, res) => {
  const { userId, message, scheduledAt } = req.body;

  // Schedule one-time job
  await agenda.schedule(scheduledAt, 'send reminder', {
    userId,
    message,
    type: 'reminder',
  });

  res.json({ success: true });
});

// Schedule subscription warnings
async function scheduleExpiryWarnings() {
  const users = await db.users.find({
    subscription_expires_at: {
      $gte: new Date(),
      $lte: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // Next 7 days
    },
  });

  for (const user of users) {
    const warningDate = new Date(
      user.subscription_expires_at.getTime() - 3 * 24 * 60 * 60 * 1000
    ); // 3 days before

    await agenda.schedule(warningDate, 'subscription expiry warning', {
      userId: user.id,
    });
  }
}

// Run daily to schedule new warnings
cron.schedule('0 0 * * *', async () => {
  await scheduleExpiryWarnings();
});
```

---

## Example 5: Health Monitoring System

```javascript
class JobHealthMonitor {
  constructor() {
    this.jobs = new Map();
  }

  register(jobName, config) {
    this.jobs.set(jobName, {
      name: jobName,
      expectedInterval: config.expectedInterval,
      lastRun: null,
      lastSuccess: null,
      lastFailure: null,
      consecutiveFailures: 0,
      totalRuns: 0,
      totalFailures: 0,
    });
  }

  recordRun(jobName, success, error = null) {
    const job = this.jobs.get(jobName);
    if (!job) return;

    job.lastRun = new Date();
    job.totalRuns++;

    if (success) {
      job.lastSuccess = new Date();
      job.consecutiveFailures = 0;
    } else {
      job.lastFailure = new Date();
      job.totalFailures++;
      job.consecutiveFailures++;

      if (job.consecutiveFailures >= 3) {
        this.alert(jobName, 'Multiple consecutive failures', error);
      }
    }
  }

  checkHealth() {
    const now = Date.now();

    for (const [name, job] of this.jobs) {
      if (!job.lastRun) continue;

      const elapsed = now - job.lastRun.getTime();

      if (elapsed > job.expectedInterval * 2) {
        this.alert(name, `Job stale (no run in ${elapsed}ms)`);
      }
    }
  }

  getStats() {
    const stats = [];

    for (const [name, job] of this.jobs) {
      stats.push({
        name,
        lastRun: job.lastRun,
        uptime: job.totalRuns > 0 ? 1 - job.totalFailures / job.totalRuns : 0,
        consecutiveFailures: job.consecutiveFailures,
      });
    }

    return stats;
  }

  alert(jobName, message, error = null) {
    console.error(`[ALERT] ${jobName}: ${message}`);
    // Send to monitoring service
  }
}

// Usage
const monitor = new JobHealthMonitor();

monitor.register('daily-backup', {
  expectedInterval: 24 * 60 * 60 * 1000, // 1 day
});

cron.schedule('0 2 * * *', async () => {
  try {
    await runBackup();
    monitor.recordRun('daily-backup', true);
  } catch (error) {
    monitor.recordRun('daily-backup', false, error);
  }
});

// Check health every 5 minutes
cron.schedule('*/5 * * * *', () => {
  monitor.checkHealth();
});

// Expose metrics endpoint
app.get('/metrics/jobs', (req, res) => {
  res.json(monitor.getStats());
});
```
