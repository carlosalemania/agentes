# Batch Processing Examples

## Example 1: Email Batch Sender with Retry

```javascript
const pLimit = require('p-limit');

class EmailBatchProcessor {
  constructor(options = {}) {
    this.batchSize = options.batchSize || 50;
    this.concurrency = options.concurrency || 5;
    this.retries = options.retries || 3;
    this.checkpointInterval = options.checkpointInterval || 100;
  }

  async sendEmails(recipients) {
    const limit = pLimit(this.concurrency);
    let processed = 0;
    let failed = [];

    for (let i = 0; i < recipients.length; i += this.batchSize) {
      const batch = recipients.slice(i, i + this.batchSize);

      const results = await Promise.allSettled(
        batch.map((recipient) =>
          limit(async () => {
            try {
              await this.sendEmailWithRetry(recipient);
              return { success: true, recipient };
            } catch (error) {
              return { success: false, recipient, error };
            }
          })
        )
      );

      // Collect failures
      results.forEach((result) => {
        if (result.status === 'fulfilled' && !result.value.success) {
          failed.push(result.value);
        }
      });

      processed += batch.length;

      // Checkpoint
      if (processed % this.checkpointInterval === 0) {
        console.log(`Processed ${processed}/${recipients.length}`);
        await this.saveCheckpoint({ processed, failed });
      }

      // Rate limiting - wait between batches
      await this.sleep(1000);
    }

    return { processed, failed };
  }

  async sendEmailWithRetry(recipient) {
    let lastError;

    for (let attempt = 0; attempt < this.retries; attempt++) {
      try {
        await emailService.send({
          to: recipient.email,
          subject: recipient.subject,
          body: recipient.body,
        });
        return;
      } catch (error) {
        lastError = error;
        console.log(`Retry ${attempt + 1} for ${recipient.email}`);
        await this.sleep(1000 * Math.pow(2, attempt));
      }
    }

    throw lastError;
  }

  sleep(ms) {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }

  async saveCheckpoint(data) {
    await fs.promises.writeFile('checkpoint.json', JSON.stringify(data));
  }
}

// Usage
const processor = new EmailBatchProcessor({
  batchSize: 100,
  concurrency: 10,
  retries: 3,
});

const result = await processor.sendEmails(recipients);
console.log(`Sent: ${result.processed}, Failed: ${result.failed.length}`);
```

---

## Example 2: Database Migration

```javascript
class DataMigrationJob {
  constructor(sourceDb, targetDb, options = {}) {
    this.sourceDb = sourceDb;
    this.targetDb = targetDb;
    this.batchSize = options.batchSize || 1000;
    this.cursor = options.startCursor || null;
  }

  async migrate() {
    let totalMigrated = 0;
    let hasMore = true;

    while (hasMore) {
      const batch = await this.fetchBatch();

      if (batch.length === 0) {
        hasMore = false;
        break;
      }

      // Transform data
      const transformed = batch.map((record) => this.transform(record));

      // Insert to target
      await this.targetDb.transaction(async (trx) => {
        for (const record of transformed) {
          await trx('users').insert(record).onConflict('id').merge();
        }
      });

      totalMigrated += batch.length;
      this.cursor = batch[batch.length - 1].id;

      // Save progress
      await this.saveProgress({
        cursor: this.cursor,
        migrated: totalMigrated,
        timestamp: new Date(),
      });

      console.log(`Migrated ${totalMigrated} records`);

      // Avoid overwhelming database
      await this.sleep(100);
    }

    return totalMigrated;
  }

  async fetchBatch() {
    let query = this.sourceDb('old_users')
      .orderBy('id')
      .limit(this.batchSize);

    if (this.cursor) {
      query = query.where('id', '>', this.cursor);
    }

    return await query;
  }

  transform(record) {
    return {
      id: record.id,
      email: record.email_address,
      name: `${record.first_name} ${record.last_name}`,
      created_at: record.signup_date,
      metadata: {
        legacy_id: record.old_id,
        source: 'migration',
      },
    };
  }

  async saveProgress(data) {
    await fs.promises.writeFile('migration-progress.json', JSON.stringify(data));
  }

  sleep(ms) {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

// Usage
const migration = new DataMigrationJob(sourceDb, targetDb, {
  batchSize: 500,
  startCursor: loadCheckpoint()?.cursor,
});

const total = await migration.migrate();
console.log(`Migration complete: ${total} records`);
```

---

## Example 3: CSV Processing with Streams

```javascript
const fs = require('fs');
const { pipeline } = require('stream/promises');
const { Transform } = require('stream');
const csv = require('csv-parser');

class CSVBatchProcessor extends Transform {
  constructor(options = {}) {
    super({ objectMode: true });
    this.batchSize = options.batchSize || 100;
    this.processFn = options.processFn;
    this.buffer = [];
    this.processed = 0;
  }

  async _transform(chunk, encoding, callback) {
    this.buffer.push(chunk);

    if (this.buffer.length >= this.batchSize) {
      try {
        await this.processBatch();
        callback();
      } catch (error) {
        callback(error);
      }
    } else {
      callback();
    }
  }

  async _flush(callback) {
    try {
      if (this.buffer.length > 0) {
        await this.processBatch();
      }
      callback();
    } catch (error) {
      callback(error);
    }
  }

  async processBatch() {
    const batch = [...this.buffer];
    this.buffer = [];

    await this.processFn(batch);

    this.processed += batch.length;
    console.log(`Processed ${this.processed} records`);
  }
}

// Usage
await pipeline(
  fs.createReadStream('users.csv'),
  csv(),
  new CSVBatchProcessor({
    batchSize: 500,
    processFn: async (batch) => {
      // Batch insert to database
      await db('users').insert(batch);
    },
  })
);

console.log('CSV processing complete');
```

---

## Example 4: Report Generation Job

```javascript
class ReportGenerationJob {
  constructor(options = {}) {
    this.batchSize = options.batchSize || 1000;
    this.reportDate = options.reportDate || new Date();
  }

  async generate() {
    console.log(`Generating report for ${this.reportDate}`);

    const startTime = Date.now();
    let totalRecords = 0;

    // Step 1: Aggregate data
    const aggregates = await this.aggregateData();

    // Step 2: Generate sections in parallel
    const [sales, users, revenue] = await Promise.all([
      this.generateSalesSection(),
      this.generateUsersSection(),
      this.generateRevenueSection(),
    ]);

    // Step 3: Combine and format
    const report = {
      date: this.reportDate,
      generated_at: new Date(),
      aggregates,
      sections: { sales, users, revenue },
    };

    // Step 4: Save report
    await this.saveReport(report);

    const duration = Date.now() - startTime;
    console.log(`Report generated in ${duration}ms`);

    return report;
  }

  async aggregateData() {
    const [totalSales, totalUsers, totalRevenue] = await Promise.all([
      db('orders').count('* as count').first(),
      db('users').count('* as count').first(),
      db('orders').sum('total as sum').first(),
    ]);

    return {
      total_sales: totalSales.count,
      total_users: totalUsers.count,
      total_revenue: totalRevenue.sum,
    };
  }

  async generateSalesSection() {
    const sales = await db('orders')
      .select(
        db.raw('DATE(created_at) as date'),
        db.raw('COUNT(*) as count'),
        db.raw('SUM(total) as revenue')
      )
      .where('created_at', '>=', this.reportDate)
      .groupBy(db.raw('DATE(created_at)'))
      .orderBy('date');

    return sales;
  }

  async generateUsersSection() {
    const users = await db('users')
      .select(
        db.raw('DATE(created_at) as date'),
        db.raw('COUNT(*) as signups')
      )
      .where('created_at', '>=', this.reportDate)
      .groupBy(db.raw('DATE(created_at)'))
      .orderBy('date');

    return users;
  }

  async generateRevenueSection() {
    const revenue = await db('orders')
      .select('product_id')
      .sum('total as revenue')
      .count('* as orders')
      .where('created_at', '>=', this.reportDate)
      .groupBy('product_id')
      .orderBy('revenue', 'desc')
      .limit(10);

    return revenue;
  }

  async saveReport(report) {
    const filename = `report-${this.reportDate.toISOString().slice(0, 10)}.json`;
    await fs.promises.writeFile(filename, JSON.stringify(report, null, 2));
    console.log(`Report saved to ${filename}`);
  }
}

// Schedule daily
const cron = require('node-cron');

cron.schedule('0 0 * * *', async () => {
  const job = new ReportGenerationJob({
    reportDate: new Date(Date.now() - 24 * 60 * 60 * 1000), // Yesterday
  });

  try {
    await job.generate();
  } catch (error) {
    console.error('Report generation failed:', error);
    // Send alert
  }
});
```
