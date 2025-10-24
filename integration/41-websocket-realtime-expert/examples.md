# WebSocket/Real-time Expert - Examples

---

## Example: Live Notifications System

```javascript
// Socket.IO notifications server

io.on('connection', (socket) => {
  const userId = socket.userId;

  // Subscribe to user's notification channel
  socket.join(`user:${userId}`);

  // Also subscribe to user's teams
  const teams = await getUserTeams(userId);
  teams.forEach(team => {
    socket.join(`team:${team.id}`);
  });

  socket.on('disconnect', () => {
    console.log(`User ${userId} disconnected`);
  });
});

// Send notification to specific user
function notifyUser(userId, notification) {
  io.to(`user:${userId}`).emit('notification', notification);
}

// Send notification to team
function notifyTeam(teamId, notification) {
  io.to(`team:${teamId}`).emit('notification', notification);
}

// Usage
notifyUser(123, {
  type: 'mention',
  message: 'You were mentioned in a comment',
  link: '/posts/456'
});
```

---

## Example: Server-Sent Events (SSE)

```javascript
// Express SSE endpoint
app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  const clientId = Date.now();

  const sendEvent = (data) => {
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  };

  // Send initial connection event
  sendEvent({
    type: 'connected',
    clientId: clientId
  });

  // Send updates every 5 seconds
  const interval = setInterval(() => {
    sendEvent({
      type: 'update',
      timestamp: Date.now(),
      data: { value: Math.random() }
    });
  }, 5000);

  req.on('close', () => {
    clearInterval(interval);
    console.log(`Client ${clientId} disconnected`);
  });
});

// Client-side
const eventSource = new EventSource('/events');

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received:', data);
};

eventSource.onerror = (error) => {
  console.error('SSE error:', error);
  eventSource.close();
};
```

---

**Versión:** 1.0.0
