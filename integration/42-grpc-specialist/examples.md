# gRPC Examples

## Example 1: Unary RPC with Error Handling

```javascript
// server.js
const grpc = require('@grpc/grpc-js');

const userService = {
  async GetUser(call, callback) {
    const { id } = call.request;

    // Validation
    if (!id) {
      return callback({
        code: grpc.status.INVALID_ARGUMENT,
        message: 'User ID is required',
      });
    }

    try {
      const user = await db.users.findById(id);

      if (!user) {
        return callback({
          code: grpc.status.NOT_FOUND,
          message: `User ${id} not found`,
        });
      }

      callback(null, { user });
    } catch (error) {
      callback({
        code: grpc.status.INTERNAL,
        message: 'Database error',
        metadata: grpc.Metadata.fromObject({
          'error-details': error.message,
        }),
      });
    }
  },
};

// client.js
async function getUser(userId) {
  return new Promise((resolve, reject) => {
    const deadline = new Date();
    deadline.setSeconds(deadline.getSeconds() + 5); // 5s timeout

    client.GetUser(
      { id: userId },
      { deadline },
      (error, response) => {
        if (error) {
          if (error.code === grpc.status.NOT_FOUND) {
            reject(new Error('User not found'));
          } else if (error.code === grpc.status.DEADLINE_EXCEEDED) {
            reject(new Error('Request timeout'));
          } else {
            reject(error);
          }
        } else {
          resolve(response.user);
        }
      }
    );
  });
}
```

---

## Example 2: Server Streaming with Pagination

```protobuf
// orders.proto
message ListOrdersRequest {
  string customer_id = 1;
  int32 limit = 2;
  string cursor = 3;
}

message Order {
  string id = 1;
  string customer_id = 2;
  double total = 3;
  int64 created_at = 4;
}
```

```javascript
// server.js
const orderService = {
  async ListOrders(call) {
    const { customer_id, limit = 50, cursor } = call.request;

    try {
      let query = db.orders.find({ customer_id });

      if (cursor) {
        query = query.where('created_at').gt(cursor);
      }

      const orders = await query.limit(limit).sort({ created_at: 1 });

      for (const order of orders) {
        // Check if client cancelled
        if (call.cancelled) {
          break;
        }

        call.write({
          id: order.id,
          customer_id: order.customer_id,
          total: order.total,
          created_at: order.created_at,
        });
      }

      call.end();
    } catch (error) {
      call.emit('error', {
        code: grpc.status.INTERNAL,
        message: error.message,
      });
    }
  },
};

// client.js
function listOrders(customerId) {
  const orders = [];

  return new Promise((resolve, reject) => {
    const call = client.ListOrders({ customer_id: customerId, limit: 100 });

    call.on('data', (order) => {
      orders.push(order);
      console.log('Received order:', order.id);
    });

    call.on('end', () => {
      resolve(orders);
    });

    call.on('error', (error) => {
      reject(error);
    });

    // Cancel after 30 seconds
    setTimeout(() => {
      call.cancel();
    }, 30000);
  });
}
```

---

## Example 3: Client Streaming File Upload

```protobuf
// file.proto
message UploadFileRequest {
  oneof data {
    FileMetadata metadata = 1;
    bytes chunk = 2;
  }
}

message FileMetadata {
  string filename = 1;
  string content_type = 2;
  int64 size = 3;
}

message UploadFileResponse {
  string file_id = 1;
  string url = 2;
}
```

```javascript
// server.js
const fileService = {
  UploadFile(call, callback) {
    let metadata = null;
    let fileBuffer = Buffer.alloc(0);

    call.on('data', (request) => {
      if (request.metadata) {
        metadata = request.metadata;
      } else if (request.chunk) {
        fileBuffer = Buffer.concat([fileBuffer, request.chunk]);
      }
    });

    call.on('end', async () => {
      try {
        if (!metadata) {
          return callback({
            code: grpc.status.INVALID_ARGUMENT,
            message: 'Missing file metadata',
          });
        }

        // Save file
        const fileId = generateId();
        const filePath = path.join(UPLOAD_DIR, fileId);
        await fs.promises.writeFile(filePath, fileBuffer);

        callback(null, {
          file_id: fileId,
          url: `/files/${fileId}`,
        });
      } catch (error) {
        callback({
          code: grpc.status.INTERNAL,
          message: error.message,
        });
      }
    });

    call.on('error', (error) => {
      console.error('Upload error:', error);
    });
  },
};

// client.js
async function uploadFile(filePath) {
  return new Promise(async (resolve, reject) => {
    const call = client.UploadFile((error, response) => {
      if (error) {
        reject(error);
      } else {
        resolve(response);
      }
    });

    const stats = await fs.promises.stat(filePath);

    // Send metadata
    call.write({
      metadata: {
        filename: path.basename(filePath),
        content_type: 'application/octet-stream',
        size: stats.size,
      },
    });

    // Stream file in chunks
    const CHUNK_SIZE = 64 * 1024; // 64KB
    const stream = fs.createReadStream(filePath, {
      highWaterMark: CHUNK_SIZE,
    });

    stream.on('data', (chunk) => {
      call.write({ chunk });
    });

    stream.on('end', () => {
      call.end();
    });

    stream.on('error', (error) => {
      call.cancel();
      reject(error);
    });
  });
}
```

---

## Example 4: Bidirectional Streaming Chat

```javascript
// server.js
const chatService = {
  Chat(call) {
    const userId = call.metadata.get('user-id')[0];

    // Add to active connections
    activeConnections.set(userId, call);

    call.on('data', (message) => {
      console.log(`${userId}: ${message.text}`);

      // Broadcast to all users
      for (const [id, connection] of activeConnections) {
        if (id !== userId) {
          connection.write({
            user_id: userId,
            text: message.text,
            timestamp: Date.now(),
          });
        }
      }
    });

    call.on('end', () => {
      activeConnections.delete(userId);
      call.end();
    });

    call.on('error', (error) => {
      console.error(`Chat error for ${userId}:`, error);
      activeConnections.delete(userId);
    });
  },
};

// client.js
function startChat(userId) {
  const metadata = new grpc.Metadata();
  metadata.add('user-id', userId);

  const call = client.Chat(metadata);

  call.on('data', (message) => {
    console.log(`${message.user_id}: ${message.text}`);
  });

  call.on('end', () => {
    console.log('Chat ended');
  });

  call.on('error', (error) => {
    console.error('Chat error:', error);
  });

  // Send messages
  const sendMessage = (text) => {
    call.write({
      user_id: userId,
      text,
      timestamp: Date.now(),
    });
  };

  return { sendMessage, end: () => call.end() };
}

// Usage
const chat = startChat('user1');
chat.sendMessage('Hello everyone!');
chat.sendMessage('How are you?');
```

---

## Example 5: Interceptor for Authentication

```javascript
// auth-interceptor.js
function authInterceptor(options, nextCall) {
  return new grpc.InterceptingCall(nextCall(options), {
    start(metadata, listener, next) {
      // Add auth token
      const token = getAuthToken();
      if (token) {
        metadata.add('authorization', `Bearer ${token}`);
      }

      next(metadata, {
        onReceiveStatus(status, next) {
          // Handle auth errors
          if (status.code === grpc.status.UNAUTHENTICATED) {
            console.error('Authentication failed');
            // Refresh token logic here
          }
          next(status);
        },
      });
    },
  });
}

// Server-side auth validation
function validateAuth(call) {
  const metadata = call.metadata;
  const authHeader = metadata.get('authorization')[0];

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    throw {
      code: grpc.status.UNAUTHENTICATED,
      message: 'Missing or invalid authorization',
    };
  }

  const token = authHeader.slice(7);
  const user = verifyToken(token);

  if (!user) {
    throw {
      code: grpc.status.UNAUTHENTICATED,
      message: 'Invalid token',
    };
  }

  return user;
}
```
