# Mobile Backend Examples

## Example: Complete Push Notification System

```javascript
const admin = require('firebase-admin');
const express = require('express');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
});

const app = express();

// Store device token
app.post('/api/devices/register', async (req, res) => {
  const { token, platform, appVersion } = req.body;
  const userId = req.user.id;

  await db.devices.upsert(
    { userId, token },
    {
      userId,
      token,
      platform,
      appVersion,
      registeredAt: new Date(),
    }
  );

  res.json({ success: true });
});

// Subscribe to topics
app.post('/api/notifications/subscribe', async (req, res) => {
  const { topics } = req.body;
  const userId = req.user.id;

  const devices = await db.devices.find({ userId });
  const tokens = devices.map((d) => d.token);

  for (const topic of topics) {
    await admin.messaging().subscribeToTopic(tokens, topic);

    await db.subscriptions.create({
      userId,
      topic,
      subscribedAt: new Date(),
    });
  }

  res.json({ success: true });
});

// Send notification
app.post('/api/notifications/send', async (req, res) => {
  const { userIds, title, body, data } = req.body;

  const devices = await db.devices.find({
    userId: { $in: userIds },
  });

  const tokens = devices.map((d) => d.token);

  const message = {
    tokens,
    notification: { title, body },
    data,
    android: {
      priority: 'high',
      notification: {
        channelId: 'default',
        sound: 'default',
      },
    },
    apns: {
      payload: {
        aps: {
          sound: 'default',
          badge: 1,
        },
      },
    },
  };

  const response = await admin.messaging().sendMulticast(message);

  // Store notification
  await db.notifications.create({
    userIds,
    title,
    body,
    data,
    successCount: response.successCount,
    failureCount: response.failureCount,
    sentAt: new Date(),
  });

  res.json({
    success: true,
    successCount: response.successCount,
    failureCount: response.failureCount,
  });
});
```

## Example: Offline-First Todo App Sync

```javascript
// Sync endpoint
app.post('/api/sync/todos', async (req, res) => {
  const userId = req.user.id;
  const { lastSyncTime, changes } = req.body;

  // Apply client changes
  const conflicts = [];

  for (const todo of changes.created || []) {
    await db.todos.create({
      ...todo,
      userId,
      createdAt: new Date(),
    });
  }

  for (const todo of changes.updated || []) {
    const existing = await db.todos.findOne({
      id: todo.id,
      userId,
    });

    if (!existing) continue;

    // Conflict: server version is newer
    if (existing.updatedAt > new Date(todo.updatedAt)) {
      conflicts.push({
        id: todo.id,
        serverVersion: existing,
        clientVersion: todo,
      });
      continue;
    }

    await db.todos.update(
      { id: todo.id, userId },
      { ...todo, updatedAt: new Date() }
    );
  }

  for (const id of changes.deleted || []) {
    await db.todos.softDelete({ id, userId });
  }

  // Get server changes
  const since = new Date(lastSyncTime);

  const serverChanges = await db.todos.find({
    userId,
    updatedAt: { $gt: since },
  });

  const deleted = await db.todos.find({
    userId,
    deletedAt: { $gt: since },
  });

  res.json({
    changes: serverChanges,
    deleted: deleted.map((t) => t.id),
    conflicts,
    serverTime: new Date().toISOString(),
  });
});
```

## Example: Version Management Middleware

```javascript
const APP_VERSIONS = {
  ios: {
    minVersion: '2.0.0',
    latestVersion: '2.5.0',
    storeUrl: 'https://apps.apple.com/app/id123456789',
  },
  android: {
    minVersion: '2.0.0',
    latestVersion: '2.5.0',
    storeUrl: 'https://play.google.com/store/apps/details?id=com.example.app',
  },
};

function versionMiddleware(req, res, next) {
  const version = req.headers['app-version'];
  const platform = req.headers['platform'];

  if (!version || !platform) {
    return res.status(400).json({
      error: 'Missing version headers',
    });
  }

  const config = APP_VERSIONS[platform];

  if (!config) {
    return res.status(400).json({
      error: 'Invalid platform',
    });
  }

  const currentVersion = parseVersion(version);
  const minVersion = parseVersion(config.minVersion);

  if (currentVersion < minVersion) {
    return res.status(426).json({
      error: 'Update required',
      message: 'Please update to the latest version',
      minVersion: config.minVersion,
      storeUrl: config.storeUrl,
    });
  }

  next();
}

function parseVersion(version) {
  const [major, minor, patch] = version.split('.').map(Number);
  return major * 10000 + minor * 100 + patch;
}

// Apply to all routes
app.use('/api', versionMiddleware);

// Version check endpoint
app.get('/api/version', (req, res) => {
  const platform = req.headers['platform'];
  const version = req.headers['app-version'];

  const config = APP_VERSIONS[platform];
  const currentVersion = parseVersion(version);
  const latestVersion = parseVersion(config.latestVersion);

  res.json({
    current: version,
    latest: config.latestVersion,
    updateAvailable: currentVersion < latestVersion,
    storeUrl: config.storeUrl,
  });
});
```
