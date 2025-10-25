# Notification System Examples

## Example: Complete Notification Service

```javascript
class NotificationService {
  async notify(userId, event, data) {
    const user = await db.users.findById(userId);
    const prefs = await db.preferences.findOne({ userId });

    const template = await this.getTemplate(event);
    const content = this.render(template, data);

    const channels = this.getEnabledChannels(prefs);

    for (const channel of channels) {
      await notificationQueue.add('send', {
        userId,
        channel,
        content,
        event,
      });
    }

    // Store for in-app
    await db.notifications.insert({
      userId,
      event,
      content: content.text,
      read: false,
      createdAt: new Date(),
    });
  }

  getEnabledChannels(prefs) {
    const channels = [];
    if (prefs.email) channels.push('email');
    if (prefs.push) channels.push('push');
    if (prefs.sms) channels.push('sms');
    return channels;
  }
}
```
