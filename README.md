# 📡 Socket.IO Complete Guide

A comprehensive guide to building real-time applications with Socket.IO in Node.js and the browser.

## Table of Contents

1. [What is Socket.IO?](#what-is-socketio)
2. [Installation](#installation)
3. [Basic Setup](#basic-setup)
4. [Core Concepts & Methods](#core-concepts--methods)
5. [Advanced Features](#advanced-features)
6. [Scaling & Production](#scaling--production)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

---

## What is Socket.IO?

Socket.IO is a powerful JavaScript library that enables **real-time, bidirectional communication** between clients and servers. It's built on top of WebSockets but provides fallbacks for older browsers and environments that don't support WebSockets.

### Key Features

🔄 **Real-time, Bidirectional Communication** - Both client and server can send messages to each other at any time, without waiting for requests.

⚡ **Event-based Messaging** - Communication happens through named events, making code more organized and maintainable.

🌐 **Protocol Flexibility** - Uses WebSockets when available, automatically falls back to HTTP long-polling, and supports various transport protocols.

🎯 **Automatic Reconnection** - Built-in logic to reconnect automatically when the connection drops unexpectedly.

🔐 **Middleware Support** - Authenticate connections and validate incoming data before processing.

### How Socket.IO Works (Architecture)

```
1. Client initiates connection → Server receives 'connection' event
2. Server assigns unique socket.id to the client
3. Both parties can emit named events to each other
4. Server can send to one client, group of clients (rooms), or broadcast to all
5. Events are asynchronous and can include callbacks (acknowledgements)
```

---

## Installation

### Server-side
```bash
npm install express socket.io http
```

### Client-side (Browser)
Socket.IO automatically serves the client library at `/socket.io/socket.io.js`

**Or install via npm:**
```bash
npm install socket.io-client
```

---

## Basic Setup

### Server Setup (Node.js with Express)

```javascript
// Import required modules
import http from 'http';
import express from 'express';
import { Server } from 'socket.io';

// Initialize Express app
const app = express();

// Create HTTP server (IMPORTANT: Don't use app.listen() directly)
// Socket.IO needs the http.Server instance, not just Express
const server = http.createServer(app);

// Initialize Socket.IO with the HTTP server
// This enables WebSocket connections
const io = new Server(server, {
  cors: {
    origin: "http://localhost:3000", // Allow connections from this origin
    methods: ["GET", "POST"]
  }
});

// Listen for client connections
// This event fires whenever a new client connects to the server
io.on('connection', (socket) => {
  console.log(`✅ User connected: ${socket.id}`);
  
  // Handle client disconnect
  // This event fires when a client disconnects (intentionally or due to network issues)
  socket.on('disconnect', (reason) => {
    console.log(`❌ User disconnected: ${socket.id}`);
    console.log(`Reason: ${reason}`); // 'transport close', 'server namespace disconnect', etc.
  });
});

// Start the server
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`🚀 Server running on http://localhost:${PORT}`);
});
```

### Client Setup (Browser)

```html
<!DOCTYPE html>
<html>
<head>
  <title>Socket.IO Client</title>
</head>
<body>
  <!-- Include the Socket.IO client library -->
  <!-- This script is automatically served by Socket.IO server -->
  <script src="/socket.io/socket.io.js"></script>
  
  <script>
    // Connect to the Socket.IO server
    // If server is on different domain, pass the URL: io('http://localhost:3000')
    const socket = io();

    // Listen for successful connection
    socket.on('connect', () => {
      console.log(`✅ Connected to server with ID: ${socket.id}`);
    });

    // Listen for disconnection
    socket.on('disconnect', (reason) => {
      console.log(`❌ Disconnected: ${reason}`);
    });

    // Listen for connection errors
    socket.on('connect_error', (error) => {
      console.error(`⚠️ Connection Error: ${error.message}`);
    });
  </script>
</body>
</html>
```

---

## Core Concepts & Methods

### 1️⃣ Connection Event

**What it does:** Triggered when a client successfully connects to the server.

**Server side:**
```javascript
io.on('connection', (socket) => {
  // socket is the connection object for this specific client
  console.log(`New client connected with ID: ${socket.id}`);
  
  // Each socket has properties and methods:
  console.log(`Client IP: ${socket.handshake.address}`); // Get client IP
  console.log(`Client headers: ${socket.handshake.headers}`); // Get request headers
  
  // You can also attach data to the socket
  socket.username = 'UserName'; // Store user-specific data
  socket.userId = 123;
});
```

**Key Details:**
- Called once per client when they first connect
- The `socket` parameter represents that specific client's connection
- Each socket has a unique `socket.id`
- You can attach custom properties to the socket object

---

### 2️⃣ Emit (Send Event)

**What it does:** Send data from server to a specific client or from client to server.

**Server → Client:**
```javascript
io.on('connection', (socket) => {
  // Send message to THIS specific client only
  socket.emit('welcome', {
    message: 'Welcome to our server!',
    serverTime: new Date()
  });
  
  // You can emit multiple times
  socket.emit('notification', { type: 'info', text: 'Server is ready' });
});
```

**Client listening:**
```javascript
const socket = io();

// Listen for 'welcome' event from server
socket.on('welcome', (data) => {
  console.log(data.message); // Output: "Welcome to our server!"
  console.log(data.serverTime);
});

socket.on('notification', (data) => {
  console.log(`${data.type}: ${data.text}`); // Output: "info: Server is ready"
});
```

**Client → Server:**
```javascript
const socket = io();

// Emit event from client to server
socket.emit('userAction', {
  action: 'clicked_button',
  timestamp: Date.now()
});
```

**Server listening:**
```javascript
io.on('connection', (socket) => {
  socket.on('userAction', (data) => {
    console.log(`User performed action: ${data.action}`);
    console.log(`Time: ${new Date(data.timestamp)}`);
  });
});
```

---

### 3️⃣ On (Listen for Events)

**What it does:** Listen/subscribe to events from the other side.

**Server listening to client:**
```javascript
io.on('connection', (socket) => {
  // Listen for 'message' event from this specific client
  socket.on('message', (data) => {
    console.log(`Received message: ${data}`);
  });
  
  // You can listen to multiple events
  socket.on('typing', (data) => {
    console.log(`${data.user} is typing...`);
  });
  
  socket.on('stopTyping', () => {
    console.log('User stopped typing');
  });
});
```

**Client listening to server:**
```javascript
const socket = io();

// Listen for 'serverMessage' from server
socket.on('serverMessage', (message) => {
  console.log(`Server says: ${message}`);
});

// Listen for multiple events
socket.on('notification', (notification) => {
  alert(notification.text);
});

socket.on('userJoined', (data) => {
  console.log(`${data.username} joined the chat`);
});
```

---

### 4️⃣ Broadcast

**What it does:** Send a message from one client to ALL OTHER clients (excluding the sender).

**Syntax:** `socket.broadcast.emit('event', data)`

```javascript
io.on('connection', (socket) => {
  socket.on('chat message', (msg) => {
    // Send message to all EXCEPT the sender
    // Useful for: "User is typing...", "User joined", etc.
    socket.broadcast.emit('chat message', {
      user: socket.id,
      message: msg,
      timestamp: new Date()
    });
  });
});
```

**Use Cases:**
- "User X joined the chat" - notify everyone except that user
- "User X is typing" - notify others that someone is typing
- "User X sent a message" - all get it except the sender (who already knows)

**Comparison:**

| Method | Sends to |
|--------|----------|
| `socket.emit()` | Only THIS client |
| `io.emit()` | ALL clients including sender |
| `socket.broadcast.emit()` | ALL clients EXCEPT sender |

---

### 5️⃣ Emit to All

**What it does:** Send a message to ALL connected clients.

**Syntax:** `io.emit('event', data)`

```javascript
io.on('connection', (socket) => {
  socket.on('announcement', (msg) => {
    // Send to EVERY connected client (including sender)
    io.emit('notification', {
      type: 'announcement',
      message: msg,
      timestamp: new Date()
    });
  });
  
  socket.on('userCount', () => {
    // Notify all clients about current user count
    io.emit('userCountUpdate', {
      count: io.engine.clientsCount
    });
  });
});
```

**Use Cases:**
- Server maintenance notifications
- Real-time statistics/dashboards
- Global announcements
- User count updates

---

### 6️⃣ Acknowledgements (Request-Response Pattern)

**What it does:** Send a message and wait for a response from the other side. Similar to async/await or promises.

**Server sending and waiting for response:**
```javascript
io.on('connection', (socket) => {
  // Send data and wait for client to acknowledge
  socket.emit('askForData', { question: 'What is your name?' }, (response) => {
    console.log(`Client responded: ${response.name}`);
    console.log(`Response received at: ${response.timestamp}`);
  });
});
```

**Client receiving and responding:**
```javascript
const socket = io();

// Listen for server asking for data
socket.on('askForData', (data, callback) => {
  console.log(`Server asked: ${data.question}`);
  
  // Respond with data using the callback function
  callback({
    name: 'John',
    timestamp: new Date()
  });
});
```

**Client sending and waiting for response:**
```javascript
const socket = io();

// Emit event with callback function
socket.emit('joinRoomRequest', 
  { roomId: 'room123' }, 
  (response) => {
    if (response.success) {
      console.log(`✅ Successfully joined: ${response.message}`);
    } else {
      console.log(`❌ Failed to join: ${response.message}`);
    }
  }
);
```

**Server responding:**
```javascript
io.on('connection', (socket) => {
  socket.on('joinRoomRequest', (data, callback) => {
    // Validate if room exists
    const roomExists = checkIfRoomExists(data.roomId);
    
    if (roomExists) {
      socket.join(data.roomId);
      callback({
        success: true,
        message: 'Room joined successfully',
        roomId: data.roomId
      });
    } else {
      callback({
        success: false,
        message: 'Room does not exist'
      });
    }
  });
});
```

---

### 7️⃣ Disconnect Event

**What it does:** Fired when a client disconnects from the server.

```javascript
io.on('connection', (socket) => {
  console.log(`User ${socket.id} connected`);
  
  // Handle disconnection
  socket.on('disconnect', (reason) => {
    console.log(`User ${socket.id} disconnected`);
    console.log(`Reason: ${reason}`);
    
    // Clean up user data
    deleteUserSession(socket.id);
    updateUserCount();
    
    // Notify other clients that a user left
    io.emit('userLeft', {
      userId: socket.id,
      message: `A user has left the chat`,
      timestamp: new Date()
    });
  });
});
```

**Possible Disconnect Reasons:**
- `transport close` - Client closed the connection
- `server namespace disconnect` - Server forcefully disconnected
- `server shutting down` - Server is shutting down
- `ping timeout` - No response from client (network issue)
- `transport error` - Some transport error occurred

---

## Advanced Features

### 🏠 Rooms (Critical for Scalability)

**What they are:** Rooms allow you to group sockets and send messages to specific groups. A socket can be in multiple rooms simultaneously.

**Why Rooms Matter:**
- Enable targeted messaging to subsets of users
- More efficient than broadcasting to all
- Essential for chat applications, multiplayer games, etc.

#### Join a Room

```javascript
io.on('connection', (socket) => {
  // User joins a specific room
  socket.on('join-room', (roomId) => {
    socket.join(roomId);
    console.log(`User ${socket.id} joined room: ${roomId}`);
    
    // Notify others in the room that someone joined
    socket.broadcast.to(roomId).emit('userJoined', {
      userId: socket.id,
      message: `A user joined the room`
    });
  });
});
```

**Important Details:**
- `socket.join(roomId)` adds this socket to the specified room
- The room is created automatically if it doesn't exist
- A socket can be in multiple rooms at the same time
- The socket's original connection acts as personal room (useful for direct messages)

#### Emit to a Room

```javascript
io.on('connection', (socket) => {
  socket.on('roomMessage', (roomId, message) => {
    // Send message to ALL sockets in the room (including sender)
    io.to(roomId).emit('message', {
      from: socket.id,
      text: message,
      timestamp: new Date()
    });
  });
  
  socket.on('broadcastInRoom', (roomId, announcement) => {
    // Send to all in room except sender
    socket.broadcast.to(roomId).emit('announcement', announcement);
  });
});
```

#### Leave a Room

```javascript
io.on('connection', (socket) => {
  socket.on('leaveRoom', (roomId) => {
    socket.leave(roomId);
    console.log(`User ${socket.id} left room: ${roomId}`);
    
    // Notify others that user left
    io.to(roomId).emit('userLeft', {
      userId: socket.id,
      message: 'A user left the room'
    });
  });
});
```

**Room Methods:**

| Method | Description |
|--------|-------------|
| `socket.join(room)` | Add socket to a room |
| `socket.leave(room)` | Remove socket from a room |
| `socket.to(room).emit()` | Send to room (all except sender) |
| `io.to(room).emit()` | Send to room (including sender) |
| `socket.rooms` | Get all rooms socket belongs to |
| `io.sockets.adapter.rooms` | Get all rooms on server |

---

### 👥 Namespaces

**What they are:** Namespaces provide separate communication channels. Think of them as different "zones" within your application.

**Why Use Namespaces:**
- Organize different parts of your app (chat, notifications, admin, etc.)
- Isolate logic for different features
- Easier to maintain and scale
- Can have different middleware for different namespaces

```javascript
// Default namespace (/)
io.on('connection', (socket) => {
  console.log('User connected to default namespace');
});

// Create a custom chat namespace
const chatNamespace = io.of('/chat');

chatNamespace.on('connection', (socket) => {
  console.log('User connected to chat namespace');
  
  socket.on('message', (data) => {
    // Only emit to users in /chat namespace
    chatNamespace.emit('message', data);
  });
});

// Create notifications namespace
const notifyNamespace = io.of('/notifications');

notifyNamespace.on('connection', (socket) => {
  console.log('User connected to notifications namespace');
});
```

**Client connecting to namespace:**
```javascript
// Connect to default namespace
const socket = io();

// Connect to custom namespace
const chatSocket = io('/chat');
const notifySocket = io('/notifications');

// Different namespaces work independently
chatSocket.on('message', (data) => {
  console.log('Chat message:', data);
});

notifySocket.on('notification', (data) => {
  console.log('Notification:', data);
});
```

---

### 🔐 Authentication & Authorization

**What it does:** Validate client identity before accepting connections.

**Middleware Authentication:**
```javascript
import jwt from 'jsonwebtoken';

// Middleware runs for every connection
io.use((socket, next) => {
  // Get token from client handshake
  const token = socket.handshake.auth.token;
  
  if (!token) {
    return next(new Error('Authentication failed: No token provided'));
  }
  
  try {
    // Verify JWT token
    const decoded = jwt.verify(token, 'your-secret-key');
    
    // Attach user info to socket
    socket.userId = decoded.userId;
    socket.username = decoded.username;
    
    // Continue to next middleware or connection handler
    next();
  } catch (error) {
    next(new Error('Authentication failed: Invalid token'));
  }
});

io.on('connection', (socket) => {
  console.log(`User ${socket.userId} (${socket.username}) connected`);
  
  // Now socket.userId is available throughout the connection
  socket.on('message', (data) => {
    io.emit('message', {
      userId: socket.userId,
      username: socket.username,
      text: data
    });
  });
});
```

**Client sending token:**
```javascript
const token = localStorage.getItem('authToken');

const socket = io({
  auth: {
    token: token
  }
});

socket.on('connect_error', (error) => {
  if (error.message === 'Authentication failed') {
    console.log('Could not authenticate');
    // Redirect to login
  }
});
```

---

### ⏱️ Disconnection Handling & Auto-Reconnect

**Automatic Reconnection:**
Socket.IO has built-in automatic reconnection with exponential backoff.

```javascript
const socket = io({
  reconnection: true, // Enable reconnection
  reconnectionDelay: 1000, // Wait 1s before 1st retry
  reconnectionDelayMax: 5000, // Max wait is 5s
  reconnectionAttempts: 5 // Try 5 times before giving up
});

socket.on('disconnect', (reason) => {
  console.log(`Disconnected: ${reason}`);
  
  if (reason === 'transport close') {
    console.log('Server disconnected, will try to reconnect...');
  }
});

socket.on('reconnect_attempt', () => {
  console.log('Attempting to reconnect...');
});

socket.on('reconnect', () => {
  console.log('✅ Reconnected to server!');
  // Re-sync data, re-join rooms, etc.
  socket.emit('resync', {userId: socket.id});
});

socket.on('reconnect_error', (error) => {
  console.log(`Reconnection failed: ${error.message}`);
});
```

**Server-side handling missing users:**
```javascript
io.on('connection', (socket) => {
  // When user reconnects, clear their old session
  const previousSocketId = socket.handshake.auth.previousSocketId;
  
  if (previousSocketId) {
    // Migrate room memberships from old socket to new
    const oldSocket = io.sockets.sockets.get(previousSocketId);
    if (oldSocket) {
      // Transfer all rooms
      for (let room of oldSocket.rooms) {
        socket.join(room);
      }
    }
  }
});
```


## Scaling & Production

### Database & Session Management

Store session data in a database rather than server memory:

```javascript
import redis from 'redis';

const redisClient = redis.createClient();

io.on('connection', (socket) => {
  socket.on('storeUserData', async (userData) => {
    // Store in Redis with expiration
    await redisClient.setex(
      `user:${socket.id}`,
      3600, // 1 hour expiration
      JSON.stringify(userData)
    );
  });
});
```

### Redis Adapter for Multiple Servers

When running multiple server instances, use Redis to sync messages:

```javascript
import { createAdapter } from '@socket.io/redis-adapter';
import redis from 'redis';

const io = new Server(server);
const pubClient = redis.createClient();
const subClient = pubClient.duplicate();

Promise.all([pubClient.connect(), subClient.connect()]).then(() => {
  io.adapter(createAdapter(pubClient, subClient));
});
```

### Load Balancing

For production with multiple servers:

1. **Use sticky sessions** - Route same client to same server using IP/session ID
2. **Use Redis adapter** - Sync sockets across servers
3. **Use NGINX or HAProxy** as load balancer

**NGINX config example:**
```nginx
upstream socketio {
  least_conn; # Use least connections algorithm
  server 127.0.0.1:3000;
  server 127.0.0.1:3001;
  server 127.0.0.1:3002;
}

server {
  listen 80;
  location / {
    proxy_pass http://socketio;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_redirect off;
    proxy_buffering off; # Important for real-time
  }
}
```

---

## Best Practices

### 1. ✅ Always Use http.createServer()

```javascript
// ✅ CORRECT
import http from 'http';
import express from 'express';
import { Server } from 'socket.io';

const app = express();
const server = http.createServer(app);
const io = new Server(server);

// ❌ WRONG - Socket.IO needs http.Server
const io = new Server(app); // Won't work properly
```

### 2. ✅ Validate & Sanitize Data

```javascript
socket.on('chatMessage', (data) => {
  // Validate data
  if (!data || typeof data.text !== 'string') {
    socket.emit('error', 'Invalid message format');
    return;
  }
  
  // Sanitize/escape to prevent XSS
  const sanitized = escapeHtml(data.text);
  
  // Process safe data
  io.emit('chatMessage', { text: sanitized });
});
```

### 3. ✅ Implement Rate Limiting

```javascript
import rateLimit from 'socket.io-rate-limit';

const limiter = rateLimit(io, {
  messages: 5, // 5 messages
  interval: 1000 // per 1 second
});

socket.on('message', (data) => {
  if (!limiter.isAllowed(socket.id)) {
    socket.emit('error', 'Rate limit exceeded');
    return;
  }
  // Process message
});
```

### 4. ✅ Use Error Handling

```javascript
io.on('connection', (socket) => {
  try {
    socket.on('message', (data) => {
      // Process message
    });
  } catch (error) {
    console.error(`Error in socket ${socket.id}:`, error);
    socket.emit('error', { message: 'Server error occurred' });
  }
});

// Global error handler
socket.on('error', (error) => {
  console.error(`Socket error: ${error.message}`);
});
```

### 5. ✅ Log Important Events

```javascript
io.on('connection', (socket) => {
  // Log important actions
  console.log(`[${new Date().toISOString()}] User ${socket.id} connected`);
  
  socket.on('disconnect', (reason) => {
    console.log(`[${new Date().toISOString()}] User ${socket.id} disconnected: ${reason}`);
  });
});
```

### 6. ✅ Handle Memory Leaks

```javascript
io.on('connection', (socket) => {
  const timers = [];
  
  socket.on('startTimer', () => {
    const timer = setInterval(() => {
      // Do something
    }, 1000);
    timers.push(timer);
  });
  
  socket.on('disconnect', () => {
    // Clear all timers to prevent memory leaks
    timers.forEach(timer => clearInterval(timer));
  });
});
```

### 7. ✅ Use Rooms for Grouping

```javascript
// Good: Rooms for scaling
socket.join(`room:${roomId}`);
io.to(`room:${roomId}`).emit('message', msg);

// Bad: Broadcasting to all when only some need it
io.emit('message', msg); // Inefficient if many users
```

---

## Troubleshooting

### Connection Issues

**Problem:** Client can't connect to server

**Solutions:**
```javascript
// Server: Enable CORS
const io = new Server(server, {
  cors: {
    origin: "http://localhost:3000",
    methods: ["GET", "POST"]
  }
});

// Client: Set proper URL
const socket = io('http://localhost:3000', {
  transports: ['websocket', 'polling'] // Try multiple transports
});

socket.on('connect_error', (error) => {
  console.error('Connection error:', error.message);
});
```

### 🔴 Port Already in Use

```bash
# Find process using port 3000 (Windows)
netstat -ano | findstr :3000

# Kill process (replace PID with actual)
taskkill /PID <PID> /F
```

### ⚠️ Messages Not Received

**Problem:** Client sends message but server doesn't receive

**Solutions:**
```javascript
// 1. Check if socket.on is listening for correct event
socket.on('message', (data) => { // Must match emit event name
  console.log('Received:', data);
});

// 2. Ensure server is listening
io.on('connection', (socket) => {
  socket.on('message', (data) => { // Must match client emit
    console.log('Received from client:', data);
  });
});

// 3. Check for typos in event names
socket.emit('mesage'); // ❌ Typo: should be 'message'
```

### Slow Performance

**Problem:** Messages delayed, high latency

**Solutions:**
```javascript
// 1. Reduce emit frequency
const throttle = (func, limit) => {
  let inThrottle;
  return (...args) => {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
};

const sendThrottled = throttle(
  () => socket.emit('update', data),
  100 // Limit to 100ms intervals
);

// 2. Use rooms instead of broadcasting
io.to(roomId).emit('message', data); // More efficient

// 3. Monitor with profiling
console.time('messageProcessing');
// ... process message
console.timeEnd('messageProcessing');
```

### Memory Leaks

**Problem:** Server memory usage keeps increasing

**Solutions:**
```javascript
// Clean up on disconnect
socket.on('disconnect', () => {
  // Remove socket from custom storage
  delete userSessions[socket.id];
  
  // Clear any timers
  clearInterval(userIntervals[socket.id]);
  
  // Unsubscribe from rooms
  socket.leaveAll();
});

// Monitor memory
setInterval(() => {
  console.log('Connected sockets:', io.engine.clientsCount);
  console.log('Memory usage:', process.memoryUsage());
}, 10000);
```

---

## Common Methods Reference Table

| Method | Sender | Receiver | Example |
|--------|--------|----------|---------|
| `socket.emit()` | Client/Server | Opposite side (one) | `socket.emit('message', 'hello')` |
| `io.emit()` | Server | All clients | `io.emit('broadcast', data)` |
| `socket.broadcast.emit()` | Client/Server | All except sender | `socket.broadcast.emit('update', data)` |
| `io.to(room).emit()` | Server | All in room | `io.to('room1').emit('msg', data)` |
| `socket.to(room).emit()` | Server | Room except sender | `socket.to('room1').emit('msg', data)` |
| `socket.join(room)` | Client/Server | - | `socket.join('room1')` |
| `socket.leave(room)` | Client/Server | - | `socket.leave('room1')` |
| `socket.on()` | Both | - | `socket.on('event', callback)` |

---

---

## When to Use Socket.IO? ✅ vs ❌

### ✅ Use Socket.IO For:
- Real-time chat applications
- Live notifications & alerts
- Collaborative tools (shared editing, whiteboards)
- Multiplayer games
- Live dashboards & analytics
- Real-time stock tickers
- Live video/audio applications

### ❌ Don't Use Socket.IO For:
- Simple CRUD APIs
- File uploads (use Form data instead)
- Static content serving
- APIs with infrequent requests
- One-way data flows

---

## Additional Resources

- [Socket.IO Official Docs](https://socket.io/docs/)
- [Socket.IO Server API](https://socket.io/docs/v4/server-api/)
- [Socket.IO Client API](https://socket.io/docs/v4/client-api/)
- [Socket.IO Examples](https://github.com/socketio/socket.io/tree/main/examples)

---

**Made with ❤️ for real-time enthusiasts!**