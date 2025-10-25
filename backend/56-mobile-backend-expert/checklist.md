# Mobile Backend Checklist

## API Design
- [ ] Optimized for mobile bandwidth
- [ ] Cursor-based pagination
- [ ] Response compression enabled
- [ ] Binary protocols considered
- [ ] Batch endpoints available

## Push Notifications
- [ ] FCM configured
- [ ] APNs configured
- [ ] Topic subscriptions working
- [ ] Silent notifications tested
- [ ] Token management implemented

## Deep Linking
- [ ] Universal Links configured (iOS)
- [ ] App Links configured (Android)
- [ ] Dynamic links implemented
- [ ] Fallback URLs working
- [ ] Analytics tracking

## Versioning
- [ ] Version check endpoint
- [ ] Force update mechanism
- [ ] Backward compatibility
- [ ] Feature flags per version
- [ ] Deprecation warnings

## Offline Sync
- [ ] Delta sync implemented
- [ ] Conflict resolution
- [ ] Timestamp tracking
- [ ] Deleted items tracked
- [ ] Background sync

## Performance
- [ ] Response times optimized
- [ ] Payload sizes minimized
- [ ] CDN for static assets
- [ ] Caching strategies
- [ ] Connection pooling
