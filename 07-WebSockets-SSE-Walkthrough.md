# WebSockets & Server-Sent Events (SSE) - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung in Echtzeit-Kommunikation](#einführung-in-echtzeit-kommunikation)
2. [HTTP vs WebSockets vs SSE](#http-vs-websockets-vs-sse)
3. [WebSockets Grundlagen](#websockets-grundlagen)
4. [WebSocket-Server mit ws](#websocket-server-mit-ws)
5. [Socket.io](#socketio)
6. [WebSocket-Client (Browser)](#websocket-client-browser)
7. [WebSocket Best Practices](#websocket-best-practices)
8. [Server-Sent Events (SSE) Grundlagen](#server-sent-events-sse-grundlagen)
9. [SSE Server implementieren](#sse-server-implementieren)
10. [SSE Client (Browser)](#sse-client-browser)
11. [Authentication bei WebSockets & SSE](#authentication-bei-websockets--sse)
12. [Skalierung & Load Balancing](#skalierung--load-balancing)
13. [Testing](#testing)
14. [Troubleshooting](#troubleshooting)
15. [Praktische Projekte](#praktische-projekte)

---

## Einführung in Echtzeit-Kommunikation

### Was ist Echtzeit-Kommunikation?

Echtzeit-Kommunikation ermöglicht es, Daten **sofort** zwischen Server und Client auszutauschen, ohne dass der Client ständig nach Updates fragen muss.

### Anwendungsfälle

**Chat-Anwendungen** 💬
- Instant Messaging
- Gruppenchats
- Videokonferenzen

**Live-Updates** 📊
- Aktienkurse
- Sportergebnisse
- Dashboard-Metriken

**Collaborative Tools** 👥
- Google Docs-ähnliche Editoren
- Shared Whiteboards
- Code-Pairing

**Gaming** 🎮
- Multiplayer Games
- Real-time Positioning
- Game State Synchronisation

**Notifications** 🔔
- Push Notifications
- Alert Systems
- Status Updates

**IoT** 🌐
- Sensor-Daten
- Smart Home
- Real-time Monitoring

---

## HTTP vs WebSockets vs SSE

### HTTP (Request-Response)

```
Client ──── Request ───→ Server
Client ←─── Response ─── Server
```

**Eigenschaften:**
- Unidirektional (Client → Server → Client)
- Stateless
- Overhead bei jeder Anfrage
- Polling für "Echtzeit"

**Polling-Problem:**
```javascript
// Ineffizient!
setInterval(() => {
  fetch('/api/messages')
    .then(res => res.json())
    .then(data => updateUI(data));
}, 1000); // Jede Sekunde eine Anfrage
```

### WebSockets (Bidirektional)

```
Client ────── Handshake ─────→ Server
Client ←── Upgrade to WS ───── Server
Client ←──→ Bidirektional ←──→ Server
```

**Eigenschaften:**
- ✅ Bidirektional (Client ↔ Server)
- ✅ Persistent Connection
- ✅ Low Latency
- ✅ Wenig Overhead
- ❌ Komplexer

**Protocol:**
```
ws://example.com/chat
wss://example.com/chat  (verschlüsselt)
```

### Server-Sent Events (Unidirektional)

```
Client ──── Request ───→ Server
Client ←── Stream ←──── Server
```

**Eigenschaften:**
- ✅ Unidirektional (Server → Client)
- ✅ HTTP-basiert
- ✅ Automatische Reconnection
- ✅ Event IDs (Resume möglich)
- ❌ Nur Server → Client

### Vergleich

| Feature | HTTP | WebSocket | SSE |
|---------|------|-----------|-----|
| **Richtung** | Request-Response | Bidirektional | Server → Client |
| **Connection** | Neu bei jeder Anfrage | Persistent | Persistent |
| **Overhead** | Hoch | Niedrig | Mittel |
| **Browser-Support** | 100% | ~98% | ~97% |
| **Complexity** | Einfach | Komplex | Mittel |
| **Auto-Reconnect** | N/A | Manuell | Automatisch |
| **Binary Data** | ✅ | ✅ | ❌ |

### Wann was verwenden?

**WebSockets verwenden:**
- Chat-Anwendungen
- Gaming
- Collaborative Editing
- Bidirektionale Kommunikation benötigt

**SSE verwenden:**
- Live-Updates (Feeds)
- Notifications
- Dashboard-Updates
- Nur Server → Client Kommunikation

**HTTP verwenden:**
- Standard CRUD-Operationen
- Keine Echtzeit-Anforderungen
- File Uploads

---

## WebSockets Grundlagen

### WebSocket Lifecycle

```javascript
// 1. Connection öffnen
const ws = new WebSocket('ws://localhost:8080');

// 2. Connection established
ws.onopen = (event) => {
  console.log('Connected');
  ws.send('Hello Server!');
};

// 3. Nachrichten empfangen
ws.onmessage = (event) => {
  console.log('Received:', event.data);
};

// 4. Fehler behandeln
ws.onerror = (error) => {
  console.error('WebSocket Error:', error);
};

// 5. Connection schließen
ws.onclose = (event) => {
  console.log('Connection closed', event.code, event.reason);
};

// 6. Manuell schließen
ws.close();
```

### WebSocket Frames

WebSocket-Kommunikation erfolgt über **Frames**:

```
Text Frame:    "Hello World"
Binary Frame:  ArrayBuffer, Blob
Control Frame: Ping, Pong, Close
```

### Close Codes

```javascript
1000 // Normal Closure
1001 // Going Away (Browser-Tab geschlossen)
1002 // Protocol Error
1003 // Unsupported Data
1006 // Abnormal Closure (keine Close-Frame)
1007 // Invalid Frame Payload Data
1008 // Policy Violation
1009 // Message Too Big
1011 // Internal Server Error
```

---

## WebSocket-Server mit ws

### Installation

```bash
npm install ws
npm install --save-dev @types/ws
```

### Basis WebSocket-Server

```typescript
// src/server.ts
import { WebSocketServer, WebSocket } from 'ws';

const wss = new WebSocketServer({ port: 8080 });

wss.on('connection', (ws: WebSocket) => {
  console.log('Client connected');

  // Nachricht empfangen
  ws.on('message', (data: Buffer) => {
    const message = data.toString();
    console.log('Received:', message);

    // Antwort senden
    ws.send(`Echo: ${message}`);
  });

  // Fehler behandeln
  ws.on('error', (error) => {
    console.error('WebSocket error:', error);
  });

  // Connection geschlossen
  ws.on('close', () => {
    console.log('Client disconnected');
  });

  // Willkommensnachricht
  ws.send('Welcome to the WebSocket server!');
});

console.log('WebSocket server running on ws://localhost:8080');
```

### Broadcast (Nachricht an alle Clients)

```typescript
wss.on('connection', (ws: WebSocket) => {
  ws.on('message', (data: Buffer) => {
    const message = data.toString();

    // An alle Clients senden
    wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  });
});
```

### Broadcast an alle außer Sender

```typescript
wss.on('connection', (ws: WebSocket) => {
  ws.on('message', (data: Buffer) => {
    const message = data.toString();

    // An alle außer Sender
    wss.clients.forEach((client) => {
      if (client !== ws && client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  });
});
```

### Typed Messages

```typescript
// src/types.ts
export enum MessageType {
  CHAT = 'chat',
  NOTIFICATION = 'notification',
  USER_JOINED = 'user_joined',
  USER_LEFT = 'user_left'
}

export interface ChatMessage {
  type: MessageType;
  userId: string;
  username: string;
  content: string;
  timestamp: Date;
}

// src/server.ts
wss.on('connection', (ws: WebSocket) => {
  ws.on('message', (data: Buffer) => {
    try {
      const message: ChatMessage = JSON.parse(data.toString());

      switch (message.type) {
        case MessageType.CHAT:
          handleChatMessage(message);
          break;
        case MessageType.NOTIFICATION:
          handleNotification(message);
          break;
      }
    } catch (error) {
      ws.send(JSON.stringify({ error: 'Invalid message format' }));
    }
  });
});

function handleChatMessage(message: ChatMessage) {
  wss.clients.forEach((client) => {
    if (client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify(message));
    }
  });
}
```

### WebSocket mit Express

```typescript
import express from 'express';
import { WebSocketServer } from 'ws';
import http from 'http';

const app = express();
const server = http.createServer(app);
const wss = new WebSocketServer({ server });

// REST API
app.get('/api/status', (req, res) => {
  res.json({
    clients: wss.clients.size,
    uptime: process.uptime()
  });
});

// WebSocket
wss.on('connection', (ws) => {
  console.log('New WebSocket connection');

  ws.on('message', (data) => {
    console.log('Received:', data.toString());
  });
});

server.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
  console.log('WebSocket server running on ws://localhost:3000');
});
```

---

## Socket.io

### Was ist Socket.io?

Socket.io ist eine **High-Level WebSocket-Bibliothek** mit vielen Features:

✅ Auto-Reconnection
✅ Room Support
✅ Fallback zu HTTP Long-Polling
✅ Broadcasting
✅ Acknowledgments
✅ Middleware

### Installation

```bash
npm install socket.io
npm install --save-dev @types/socket.io

# Client
npm install socket.io-client
```

### Socket.io Server

```typescript
// src/server.ts
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, {
  cors: {
    origin: 'http://localhost:3000',
    methods: ['GET', 'POST']
  }
});

io.on('connection', (socket) => {
  console.log('User connected:', socket.id);

  // Event empfangen
  socket.on('chat:message', (data) => {
    console.log('Message:', data);

    // An alle senden
    io.emit('chat:message', data);
  });

  // Event mit Acknowledgment
  socket.on('chat:message:ack', (data, callback) => {
    console.log('Message with ack:', data);

    // An alle außer Sender
    socket.broadcast.emit('chat:message', data);

    // Acknowledgment
    callback({ status: 'received', messageId: Date.now() });
  });

  // Disconnect
  socket.on('disconnect', () => {
    console.log('User disconnected:', socket.id);
  });
});

httpServer.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

### Rooms (Channels)

```typescript
io.on('connection', (socket) => {
  // Room beitreten
  socket.on('room:join', (roomName) => {
    socket.join(roomName);
    console.log(`${socket.id} joined ${roomName}`);

    // Alle im Room informieren
    io.to(roomName).emit('user:joined', {
      userId: socket.id,
      room: roomName
    });
  });

  // Room verlassen
  socket.on('room:leave', (roomName) => {
    socket.leave(roomName);
    io.to(roomName).emit('user:left', {
      userId: socket.id,
      room: roomName
    });
  });

  // Nachricht an Room
  socket.on('room:message', ({ room, message }) => {
    io.to(room).emit('room:message', {
      userId: socket.id,
      message,
      timestamp: new Date()
    });
  });
});
```

### Namespaces

```typescript
// Default namespace
io.on('connection', (socket) => {
  // ...
});

// Custom namespace
const chatNamespace = io.of('/chat');
chatNamespace.on('connection', (socket) => {
  console.log('Chat namespace connection:', socket.id);

  socket.on('message', (data) => {
    chatNamespace.emit('message', data);
  });
});

// Admin namespace
const adminNamespace = io.of('/admin');
adminNamespace.on('connection', (socket) => {
  console.log('Admin namespace connection:', socket.id);
});
```

### Middleware

```typescript
// Authentication Middleware
io.use((socket, next) => {
  const token = socket.handshake.auth.token;

  if (!token) {
    return next(new Error('Authentication error'));
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    socket.data.user = decoded;
    next();
  } catch (error) {
    next(new Error('Invalid token'));
  }
});

io.on('connection', (socket) => {
  // socket.data.user ist verfügbar
  console.log('User connected:', socket.data.user);
});
```

### Typed Events

```typescript
// src/types/socket.ts
export interface ServerToClientEvents {
  'chat:message': (data: ChatMessage) => void;
  'user:joined': (data: UserJoinedEvent) => void;
  'user:left': (data: UserLeftEvent) => void;
}

export interface ClientToServerEvents {
  'chat:message': (data: SendMessageData) => void;
  'room:join': (room: string) => void;
  'room:leave': (room: string) => void;
}

export interface InterServerEvents {
  ping: () => void;
}

export interface SocketData {
  user: {
    id: string;
    username: string;
  };
}

// Server mit Types
import { Server } from 'socket.io';

const io = new Server<
  ClientToServerEvents,
  ServerToClientEvents,
  InterServerEvents,
  SocketData
>(httpServer);
```

---

## WebSocket-Client (Browser)

### Native WebSocket

```javascript
// client.js
const ws = new WebSocket('ws://localhost:8080');

ws.onopen = () => {
  console.log('Connected');
  ws.send('Hello Server!');
};

ws.onmessage = (event) => {
  console.log('Message:', event.data);
  
  try {
    const data = JSON.parse(event.data);
    handleMessage(data);
  } catch {
    // Plain text message
    console.log(event.data);
  }
};

ws.onerror = (error) => {
  console.error('Error:', error);
};

ws.onclose = (event) => {
  console.log('Disconnected:', event.code, event.reason);
  
  // Auto-Reconnect
  setTimeout(() => {
    connectWebSocket();
  }, 3000);
};
```

### Socket.io Client

```typescript
// client.ts
import { io, Socket } from 'socket.io-client';

const socket: Socket = io('http://localhost:3000', {
  auth: {
    token: 'your-jwt-token'
  },
  reconnection: true,
  reconnectionDelay: 1000,
  reconnectionAttempts: 5
});

// Connection
socket.on('connect', () => {
  console.log('Connected:', socket.id);
});

// Events empfangen
socket.on('chat:message', (data) => {
  console.log('New message:', data);
  displayMessage(data);
});

// Events senden
function sendMessage(message: string) {
  socket.emit('chat:message', {
    content: message,
    timestamp: new Date()
  });
}

// Mit Acknowledgment
function sendMessageWithAck(message: string) {
  socket.emit('chat:message:ack', { content: message }, (response) => {
    console.log('Message acknowledged:', response);
  });
}

// Room beitreten
function joinRoom(roomName: string) {
  socket.emit('room:join', roomName);
}

// Disconnect
socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
});

// Error
socket.on('connect_error', (error) => {
  console.error('Connection error:', error);
});
```

### React Integration

```typescript
// hooks/useSocket.ts
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';

export function useSocket(url: string) {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    const socketInstance = io(url);

    socketInstance.on('connect', () => {
      setIsConnected(true);
    });

    socketInstance.on('disconnect', () => {
      setIsConnected(false);
    });

    setSocket(socketInstance);

    return () => {
      socketInstance.close();
    };
  }, [url]);

  return { socket, isConnected };
}

// components/Chat.tsx
import { useSocket } from '../hooks/useSocket';

export function Chat() {
  const { socket, isConnected } = useSocket('http://localhost:3000');
  const [messages, setMessages] = useState<Message[]>([]);

  useEffect(() => {
    if (!socket) return;

    socket.on('chat:message', (message: Message) => {
      setMessages((prev) => [...prev, message]);
    });

    return () => {
      socket.off('chat:message');
    };
  }, [socket]);

  const sendMessage = (content: string) => {
    if (!socket) return;

    socket.emit('chat:message', {
      content,
      timestamp: new Date()
    });
  };

  return (
    <div>
      <div>Status: {isConnected ? '🟢 Connected' : '🔴 Disconnected'}</div>
      <MessageList messages={messages} />
      <MessageInput onSend={sendMessage} />
    </div>
  );
}
```

---

## WebSocket Best Practices

### 1. Heartbeat / Ping-Pong

```typescript
// Server
const HEARTBEAT_INTERVAL = 30000; // 30s
const HEARTBEAT_TIMEOUT = 5000;   // 5s

wss.on('connection', (ws: WebSocket) => {
  let isAlive = true;

  ws.on('pong', () => {
    isAlive = true;
  });

  const interval = setInterval(() => {
    if (!isAlive) {
      ws.terminate();
      return;
    }

    isAlive = false;
    ws.ping();
  }, HEARTBEAT_INTERVAL);

  ws.on('close', () => {
    clearInterval(interval);
  });
});
```

### 2. Connection Limits

```typescript
const MAX_CONNECTIONS_PER_IP = 5;
const connectionsByIP = new Map<string, number>();

wss.on('connection', (ws: WebSocket, req) => {
  const ip = req.socket.remoteAddress;

  if (!ip) {
    ws.close(1008, 'Unable to determine IP');
    return;
  }

  const currentConnections = connectionsByIP.get(ip) || 0;

  if (currentConnections >= MAX_CONNECTIONS_PER_IP) {
    ws.close(1008, 'Too many connections');
    return;
  }

  connectionsByIP.set(ip, currentConnections + 1);

  ws.on('close', () => {
    const count = connectionsByIP.get(ip) || 1;
    if (count <= 1) {
      connectionsByIP.delete(ip);
    } else {
      connectionsByIP.set(ip, count - 1);
    }
  });
});
```

### 3. Message Size Limits

```typescript
const MAX_MESSAGE_SIZE = 64 * 1024; // 64KB

wss.on('connection', (ws: WebSocket) => {
  ws.on('message', (data: Buffer) => {
    if (data.length > MAX_MESSAGE_SIZE) {
      ws.send(JSON.stringify({
        error: 'Message too large',
        maxSize: MAX_MESSAGE_SIZE
      }));
      return;
    }

    // Process message
  });
});
```

### 4. Rate Limiting

```typescript
const RATE_LIMIT = 10; // messages per interval
const RATE_INTERVAL = 1000; // 1 second

const messageCounts = new Map<string, { count: number; resetTime: number }>();

wss.on('connection', (ws: WebSocket) => {
  const clientId = generateClientId(ws);

  ws.on('message', (data: Buffer) => {
    const now = Date.now();
    const clientData = messageCounts.get(clientId);

    if (!clientData || now > clientData.resetTime) {
      messageCounts.set(clientId, {
        count: 1,
        resetTime: now + RATE_INTERVAL
      });
    } else {
      clientData.count++;

      if (clientData.count > RATE_LIMIT) {
        ws.send(JSON.stringify({
          error: 'Rate limit exceeded'
        }));
        return;
      }
    }

    // Process message
  });
});
```

### 5. Graceful Shutdown

```typescript
const server = httpServer.listen(3000);

process.on('SIGTERM', () => {
  console.log('SIGTERM received, closing connections...');

  // Keine neuen Connections
  server.close(() => {
    console.log('HTTP server closed');
  });

  // Alle WebSocket Connections schließen
  wss.clients.forEach((ws) => {
    ws.send(JSON.stringify({
      type: 'server:shutdown',
      message: 'Server is shutting down'
    }));
    ws.close(1001, 'Server shutting down');
  });

  // Nach Timeout forcieren
  setTimeout(() => {
    console.log('Forcing shutdown');
    process.exit(0);
  }, 10000);
});
```

---

## Server-Sent Events (SSE) Grundlagen

### Was ist SSE?

Server-Sent Events ist ein **HTTP-basierter Standard** für Server-zu-Client Push-Nachrichten.

**Eigenschaften:**
- Unidirektional (Server → Client)
- HTTP-basiert
- Text-only (UTF-8)
- Automatische Reconnection
- Event IDs für Resume

### SSE Format

```
data: This is a message

data: Multi-line
data: message

event: customEvent
data: {"key": "value"}
id: 123

: This is a comment
```

### SSE vs WebSocket

**SSE verwenden wenn:**
- Nur Server → Client Kommunikation
- Text-basierte Daten
- Einfache Implementation
- HTTP-Infrastruktur nutzen

**WebSocket verwenden wenn:**
- Bidirektionale Kommunikation
- Binäre Daten
- Gaming/Chat
- Niedrige Latenz kritisch

---

## SSE Server implementieren

### Basis SSE-Server

```typescript
// src/server.ts
import express, { Request, Response } from 'express';

const app = express();

app.get('/events', (req: Request, res: Response) => {
  // SSE Headers setzen
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.setHeader('Access-Control-Allow-Origin', '*');

  // Initiale Nachricht
  res.write('data: Connected to SSE\n\n');

  // Interval für Updates
  const intervalId = setInterval(() => {
    const data = {
      time: new Date().toISOString(),
      value: Math.random()
    };

    res.write(`data: ${JSON.stringify(data)}\n\n`);
  }, 1000);

  // Cleanup bei Disconnect
  req.on('close', () => {
    clearInterval(intervalId);
    console.log('Client disconnected');
  });
});

app.listen(3000, () => {
  console.log('SSE Server running on http://localhost:3000');
});
```

### Custom Events

```typescript
app.get('/events', (req: Request, res: Response) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  // Event mit ID
  function sendEvent(event: string, data: any, id?: number) {
    if (id !== undefined) {
      res.write(`id: ${id}\n`);
    }
    res.write(`event: ${event}\n`);
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  }

  // Verschiedene Events
  sendEvent('message', { text: 'Hello' }, 1);
  sendEvent('notification', { type: 'info', text: 'Update available' }, 2);
  sendEvent('userJoined', { username: 'Max' }, 3);
});
```

### SSE mit Client-Management

```typescript
// src/services/sseService.ts
import { Response } from 'express';

class SSEService {
  private clients = new Map<string, Response>();

  addClient(clientId: string, res: Response) {
    this.clients.set(clientId, res);
    
    res.on('close', () => {
      this.clients.delete(clientId);
      console.log(`Client ${clientId} disconnected`);
    });
  }

  sendToClient(clientId: string, event: string, data: any) {
    const client = this.clients.get(clientId);
    if (client) {
      client.write(`event: ${event}\n`);
      client.write(`data: ${JSON.stringify(data)}\n\n`);
    }
  }

  broadcast(event: string, data: any) {
    this.clients.forEach((client) => {
      client.write(`event: ${event}\n`);
      client.write(`data: ${JSON.stringify(data)}\n\n`);
    });
  }

  getClientCount() {
    return this.clients.size;
  }
}

export const sseService = new SSEService();

// Route
app.get('/events', (req: Request, res: Response) => {
  const clientId = req.query.clientId as string || generateId();

  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  sseService.addClient(clientId, res);

  res.write(`data: Connected with ID ${clientId}\n\n`);
});

// Broadcast endpoint
app.post('/api/broadcast', (req: Request, res: Response) => {
  const { event, data } = req.body;
  sseService.broadcast(event, data);
  res.json({ success: true, clients: sseService.getClientCount() });
});
```

### SSE mit Event Replay

```typescript
interface SSEEvent {
  id: number;
  event: string;
  data: any;
  timestamp: Date;
}

class SSEServiceWithReplay {
  private clients = new Map<string, Response>();
  private eventHistory: SSEEvent[] = [];
  private maxHistorySize = 100;
  private currentEventId = 0;

  addClient(clientId: string, res: Response, lastEventId?: string) {
    // Replay verpasste Events
    if (lastEventId) {
      const lastId = parseInt(lastEventId);
      const missedEvents = this.eventHistory.filter(e => e.id > lastId);
      
      missedEvents.forEach(event => {
        this.sendToResponse(res, event.event, event.data, event.id);
      });
    }

    this.clients.set(clientId, res);
    
    res.on('close', () => {
      this.clients.delete(clientId);
    });
  }

  private sendToResponse(res: Response, event: string, data: any, id: number) {
    res.write(`id: ${id}\n`);
    res.write(`event: ${event}\n`);
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  }

  broadcast(event: string, data: any) {
    this.currentEventId++;

    // Event speichern
    this.eventHistory.push({
      id: this.currentEventId,
      event,
      data,
      timestamp: new Date()
    });

    // History limitieren
    if (this.eventHistory.length > this.maxHistorySize) {
      this.eventHistory.shift();
    }

    // An alle Clients senden
    this.clients.forEach((client) => {
      this.sendToResponse(client, event, data, this.currentEventId);
    });
  }
}
```

---

## SSE Client (Browser)

### Native EventSource

```javascript
// client.js
const eventSource = new EventSource('http://localhost:3000/events');

// Standard 'message' Event
eventSource.onmessage = (event) => {
  console.log('Message:', event.data);
  const data = JSON.parse(event.data);
  updateUI(data);
};

// Custom Events
eventSource.addEventListener('notification', (event) => {
  const data = JSON.parse(event.data);
  showNotification(data);
});

eventSource.addEventListener('userJoined', (event) => {
  const data = JSON.parse(event.data);
  console.log('User joined:', data.username);
});

// Connection opened
eventSource.onopen = () => {
  console.log('SSE Connection opened');
};

// Error handling
eventSource.onerror = (error) => {
  console.error('SSE Error:', error);
  
  if (eventSource.readyState === EventSource.CLOSED) {
    console.log('Connection closed, reconnecting...');
  }
};

// Manuell schließen
function closeConnection() {
  eventSource.close();
}
```

### Mit Last-Event-ID

```javascript
// Letzten Event merken
let lastEventId = localStorage.getItem('lastEventId');

const url = lastEventId 
  ? `http://localhost:3000/events?lastEventId=${lastEventId}`
  : 'http://localhost:3000/events';

const eventSource = new EventSource(url);

eventSource.onmessage = (event) => {
  // Event ID speichern
  if (event.lastEventId) {
    lastEventId = event.lastEventId;
    localStorage.setItem('lastEventId', lastEventId);
  }

  const data = JSON.parse(event.data);
  processEvent(data);
};
```

### React Hook

```typescript
// hooks/useSSE.ts
import { useEffect, useState } from 'react';

interface UseSSEOptions {
  url: string;
  events?: { [eventName: string]: (data: any) => void };
  onMessage?: (data: any) => void;
}

export function useSSE({ url, events, onMessage }: UseSSEOptions) {
  const [isConnected, setIsConnected] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const eventSource = new EventSource(url);

    eventSource.onopen = () => {
      setIsConnected(true);
      setError(null);
    };

    eventSource.onerror = (err) => {
      setIsConnected(false);
      setError(new Error('SSE Connection error'));
    };

    if (onMessage) {
      eventSource.onmessage = (event) => {
        try {
          const data = JSON.parse(event.data);
          onMessage(data);
        } catch (err) {
          console.error('Failed to parse SSE data:', err);
        }
      };
    }

    if (events) {
      Object.entries(events).forEach(([eventName, handler]) => {
        eventSource.addEventListener(eventName, (event) => {
          try {
            const data = JSON.parse(event.data);
            handler(data);
          } catch (err) {
            console.error(`Failed to parse ${eventName} data:`, err);
          }
        });
      });
    }

    return () => {
      eventSource.close();
    };
  }, [url]);

  return { isConnected, error };
}

// Verwendung
export function Dashboard() {
  const [metrics, setMetrics] = useState<any[]>([]);
  const [notifications, setNotifications] = useState<any[]>([]);

  const { isConnected, error } = useSSE({
    url: 'http://localhost:3000/events',
    events: {
      metric: (data) => {
        setMetrics(prev => [...prev, data]);
      },
      notification: (data) => {
        setNotifications(prev => [...prev, data]);
      }
    }
  });

  return (
    <div>
      <div>Status: {isConnected ? '🟢' : '🔴'}</div>
      {error && <div>Error: {error.message}</div>}
      <MetricsChart data={metrics} />
      <NotificationList items={notifications} />
    </div>
  );
}
```

---

## Authentication bei WebSockets & SSE

### WebSocket Authentication

#### Mit ws Library

```typescript
import { WebSocketServer } from 'ws';
import jwt from 'jsonwebtoken';
import url from 'url';

const wss = new WebSocketServer({ noServer: true });

// HTTP Server Upgrade Event
server.on('upgrade', (request, socket, head) => {
  const { query } = url.parse(request.url!, true);
  const token = query.token as string;

  if (!token) {
    socket.write('HTTP/1.1 401 Unauthorized\r\n\r\n');
    socket.destroy();
    return;
  }

  try {
    const user = jwt.verify(token, process.env.JWT_SECRET!);
    
    wss.handleUpgrade(request, socket, head, (ws) => {
      (ws as any).user = user;
      wss.emit('connection', ws, request);
    });
  } catch (error) {
    socket.write('HTTP/1.1 401 Unauthorized\r\n\r\n');
    socket.destroy();
  }
});
```

#### Mit Socket.io

```typescript
io.use((socket, next) => {
  const token = socket.handshake.auth.token;

  if (!token) {
    return next(new Error('Authentication error'));
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    socket.data.user = decoded;
    next();
  } catch (error) {
    next(new Error('Invalid token'));
  }
});
```

### SSE Authentication

```typescript
// Middleware für SSE
function authenticateSSE(req: Request, res: Response, next: Function) {
  const token = req.query.token as string || 
                req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const user = jwt.verify(token, process.env.JWT_SECRET!);
    (req as any).user = user;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

app.get('/events', authenticateSSE, (req: Request, res: Response) => {
  const user = (req as any).user;
  
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  res.write(`data: Welcome ${user.username}\n\n`);

  // User-spezifische Events senden
  sseService.addClient(user.id, res);
});
```

---

## Skalierung & Load Balancing

### Problem

WebSockets und SSE sind **stateful connections**. Bei mehreren Server-Instanzen können Clients mit verschiedenen Servern verbunden sein.

```
Client A → Server 1
Client B → Server 2
Client C → Server 1
```

Wenn Server 1 eine Nachricht broadcasten will, erreicht sie nicht Client B!

### Lösung: Redis Pub/Sub

```bash
npm install redis
npm install --save-dev @types/redis
```

```typescript
// src/services/redis.ts
import { createClient } from 'redis';

export const publisher = createClient({
  url: process.env.REDIS_URL || 'redis://localhost:6379'
});

export const subscriber = createClient({
  url: process.env.REDIS_URL || 'redis://localhost:6379'
});

await publisher.connect();
await subscriber.connect();
```

```typescript
// src/server.ts mit Redis
import { publisher, subscriber } from './services/redis';

const wss = new WebSocketServer({ port: 8080 });

// Subscribe to Redis channel
await subscriber.subscribe('chat', (message) => {
  const data = JSON.parse(message);
  
  // Broadcast an lokale Clients
  wss.clients.forEach((client) => {
    if (client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify(data));
    }
  });
});

wss.on('connection', (ws) => {
  ws.on('message', async (data: Buffer) => {
    const message = JSON.parse(data.toString());
    
    // Publish zu Redis (erreicht alle Server)
    await publisher.publish('chat', JSON.stringify(message));
  });
});
```

### Socket.io mit Redis Adapter

```bash
npm install @socket.io/redis-adapter
```

```typescript
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

const pubClient = createClient({ url: 'redis://localhost:6379' });
const subClient = pubClient.duplicate();

await Promise.all([pubClient.connect(), subClient.connect()]);

io.adapter(createAdapter(pubClient, subClient));

// Jetzt funktioniert Broadcasting über alle Server-Instanzen
io.emit('message', data); // Erreicht alle Clients auf allen Servern
```

### Sticky Sessions

Für WebSockets wird **Sticky Sessions** empfohlen:

```nginx
# nginx.conf
upstream websocket_backend {
    ip_hash;  # Sticky sessions
    server backend1:3000;
    server backend2:3000;
    server backend3:3000;
}

server {
    listen 80;

    location / {
        proxy_pass http://websocket_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

---

## Testing

### WebSocket Testing mit Supertest

```bash
npm install --save-dev supertest ws @types/supertest
```

```typescript
// tests/websocket.test.ts
import { WebSocket } from 'ws';
import { server } from '../src/server';

describe('WebSocket Server', () => {
  let ws: WebSocket;

  beforeEach(() => {
    ws = new WebSocket('ws://localhost:8080');
  });

  afterEach(() => {
    ws.close();
  });

  it('should connect successfully', (done) => {
    ws.on('open', () => {
      expect(ws.readyState).toBe(WebSocket.OPEN);
      done();
    });
  });

  it('should receive welcome message', (done) => {
    ws.on('message', (data) => {
      const message = data.toString();
      expect(message).toBe('Welcome to the WebSocket server!');
      done();
    });
  });

  it('should echo messages', (done) => {
    ws.on('open', () => {
      ws.send('Hello Server');
    });

    ws.on('message', (data) => {
      const message = data.toString();
      if (message.startsWith('Echo:')) {
        expect(message).toBe('Echo: Hello Server');
        done();
      }
    });
  });

  it('should handle JSON messages', (done) => {
    const testData = { type: 'test', content: 'Hello' };

    ws.on('open', () => {
      ws.send(JSON.stringify(testData));
    });

    ws.on('message', (data) => {
      const received = JSON.parse(data.toString());
      expect(received.type).toBe('test');
      done();
    });
  });
});
```

### Socket.io Testing

```typescript
// tests/socket.test.ts
import { io as Client, Socket } from 'socket.io-client';
import { server } from '../src/server';

describe('Socket.io', () => {
  let clientSocket: Socket;

  beforeEach((done) => {
    clientSocket = Client('http://localhost:3000');
    clientSocket.on('connect', done);
  });

  afterEach(() => {
    clientSocket.close();
  });

  it('should connect successfully', () => {
    expect(clientSocket.connected).toBe(true);
  });

  it('should receive messages', (done) => {
    clientSocket.emit('chat:message', { content: 'Test' });

    clientSocket.on('chat:message', (data) => {
      expect(data.content).toBe('Test');
      done();
    });
  });

  it('should handle acknowledgments', (done) => {
    clientSocket.emit('chat:message:ack', 
      { content: 'Test' }, 
      (response) => {
        expect(response.status).toBe('received');
        done();
      }
    );
  });
});
```

### SSE Testing

```typescript
// tests/sse.test.ts
import request from 'supertest';
import { app } from '../src/server';

describe('SSE Endpoint', () => {
  it('should set correct headers', async () => {
    const response = await request(app)
      .get('/events')
      .expect(200);

    expect(response.headers['content-type']).toBe('text/event-stream');
    expect(response.headers['cache-control']).toBe('no-cache');
    expect(response.headers['connection']).toBe('keep-alive');
  });

  it('should receive events', (done) => {
    const req = request(app).get('/events');

    let dataReceived = false;

    req.on('data', (chunk) => {
      const data = chunk.toString();
      if (data.includes('data:')) {
        dataReceived = true;
      }
    });

    setTimeout(() => {
      expect(dataReceived).toBe(true);
      done();
    }, 1000);
  });
});
```

---

## Troubleshooting

### Problem: Connection schließt sofort

**WebSocket:**
```typescript
// ❌ Falsch - Server wird nicht am Leben gehalten
const wss = new WebSocketServer({ port: 8080 });

// ✅ Richtig
const server = http.createServer();
const wss = new WebSocketServer({ server });
server.listen(8080);
```

### Problem: CORS Errors

**Socket.io:**
```typescript
// ✅ CORS richtig konfigurieren
const io = new Server(httpServer, {
  cors: {
    origin: ['http://localhost:3000', 'https://yourdomain.com'],
    methods: ['GET', 'POST'],
    credentials: true
  }
});
```

### Problem: Messages kommen nicht an

**Debugging:**
```typescript
wss.on('connection', (ws) => {
  console.log('Client connected');

  ws.on('message', (data) => {
    console.log('Received:', data.toString());
    console.log('Client count:', wss.clients.size);
    console.log('ReadyState:', ws.readyState); // 1 = OPEN
  });
});
```

### Problem: Memory Leaks

**Cleanup vergessen:**
```typescript
// ❌ Falsch
wss.on('connection', (ws) => {
  const interval = setInterval(() => {
    ws.send('ping');
  }, 1000);
  // Interval läuft weiter nach Disconnect!
});

// ✅ Richtig
wss.on('connection', (ws) => {
  const interval = setInterval(() => {
    ws.send('ping');
  }, 1000);

  ws.on('close', () => {
    clearInterval(interval);
  });
});
```

### Problem: SSE funktioniert nicht

**Buffering-Problem:**
```typescript
// ❌ Nginx buffered SSE standardmäßig

// ✅ nginx.conf
location /events {
    proxy_pass http://backend;
    proxy_buffering off;
    proxy_cache off;
    proxy_set_header Connection '';
    proxy_http_version 1.1;
    chunked_transfer_encoding off;
}
```

---

## Praktische Projekte

### Projekt 1: Real-time Chat

```typescript
// src/server.ts
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';
import { v4 as uuidv4 } from 'uuid';

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, {
  cors: { origin: '*' }
});

interface User {
  id: string;
  username: string;
  room: string;
}

interface Message {
  id: string;
  userId: string;
  username: string;
  content: string;
  timestamp: Date;
  room: string;
}

const users = new Map<string, User>();
const rooms = new Map<string, Set<string>>();

io.on('connection', (socket) => {
  console.log('User connected:', socket.id);

  // User registrieren
  socket.on('user:register', ({ username, room }) => {
    const user: User = {
      id: socket.id,
      username,
      room
    };

    users.set(socket.id, user);
    socket.join(room);

    // Room-Set aktualisieren
    if (!rooms.has(room)) {
      rooms.set(room, new Set());
    }
    rooms.get(room)!.add(socket.id);

    // Anderen im Room mitteilen
    socket.to(room).emit('user:joined', {
      userId: socket.id,
      username,
      timestamp: new Date()
    });

    // User-Liste senden
    const roomUsers = Array.from(rooms.get(room) || [])
      .map(id => users.get(id))
      .filter(Boolean);

    io.to(room).emit('room:users', roomUsers);
  });

  // Chat-Nachricht
  socket.on('chat:message', ({ content }) => {
    const user = users.get(socket.id);
    if (!user) return;

    const message: Message = {
      id: uuidv4(),
      userId: socket.id,
      username: user.username,
      content,
      timestamp: new Date(),
      room: user.room
    };

    io.to(user.room).emit('chat:message', message);
  });

  // Typing Indicator
  socket.on('chat:typing', () => {
    const user = users.get(socket.id);
    if (!user) return;

    socket.to(user.room).emit('user:typing', {
      userId: socket.id,
      username: user.username
    });
  });

  // Disconnect
  socket.on('disconnect', () => {
    const user = users.get(socket.id);
    if (user) {
      // Aus Room entfernen
      const roomUsers = rooms.get(user.room);
      if (roomUsers) {
        roomUsers.delete(socket.id);
        if (roomUsers.size === 0) {
          rooms.delete(user.room);
        }
      }

      // Anderen mitteilen
      io.to(user.room).emit('user:left', {
        userId: socket.id,
        username: user.username,
        timestamp: new Date()
      });

      users.delete(socket.id);
    }
  });
});

httpServer.listen(3000, () => {
  console.log('Chat server running on http://localhost:3000');
});
```

```typescript
// client/src/Chat.tsx
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';

export function Chat() {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [messages, setMessages] = useState<Message[]>([]);
  const [users, setUsers] = useState<User[]>([]);
  const [inputValue, setInputValue] = useState('');
  const [typingUsers, setTypingUsers] = useState<Set<string>>(new Set());

  useEffect(() => {
    const socket = io('http://localhost:3000');
    setSocket(socket);

    // Events
    socket.on('chat:message', (message) => {
      setMessages(prev => [...prev, message]);
    });

    socket.on('room:users', (users) => {
      setUsers(users);
    });

    socket.on('user:joined', ({ username }) => {
      console.log(`${username} joined`);
    });

    socket.on('user:left', ({ username }) => {
      console.log(`${username} left`);
    });

    socket.on('user:typing', ({ userId, username }) => {
      setTypingUsers(prev => new Set(prev).add(username));
      setTimeout(() => {
        setTypingUsers(prev => {
          const next = new Set(prev);
          next.delete(username);
          return next;
        });
      }, 3000);
    });

    // Registrieren
    socket.emit('user:register', {
      username: 'Max',
      room: 'general'
    });

    return () => {
      socket.close();
    };
  }, []);

  const sendMessage = () => {
    if (!socket || !inputValue.trim()) return;

    socket.emit('chat:message', { content: inputValue });
    setInputValue('');
  };

  const handleTyping = () => {
    if (!socket) return;
    socket.emit('chat:typing');
  };

  return (
    <div className="chat">
      <div className="sidebar">
        <h3>Users ({users.length})</h3>
        {users.map(user => (
          <div key={user.id}>{user.username}</div>
        ))}
      </div>

      <div className="main">
        <div className="messages">
          {messages.map(msg => (
            <div key={msg.id} className="message">
              <strong>{msg.username}:</strong> {msg.content}
            </div>
          ))}
        </div>

        {typingUsers.size > 0 && (
          <div className="typing">
            {Array.from(typingUsers).join(', ')} is typing...
          </div>
        )}

        <div className="input">
          <input
            value={inputValue}
            onChange={(e) => setInputValue(e.target.value)}
            onKeyDown={(e) => {
              if (e.key === 'Enter') sendMessage();
              else handleTyping();
            }}
            placeholder="Type a message..."
          />
          <button onClick={sendMessage}>Send</button>
        </div>
      </div>
    </div>
  );
}
```

### Projekt 2: Live Dashboard mit SSE

```typescript
// src/server.ts
import express from 'express';
import { sseService } from './services/sse';

const app = express();
app.use(express.json());

// SSE Endpoint
app.get('/api/metrics', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  const clientId = Date.now().toString();
  sseService.addClient(clientId, res);

  res.write(`data: Connected\n\n`);
});

// Simulate metrics
setInterval(() => {
  const metrics = {
    cpu: Math.random() * 100,
    memory: Math.random() * 100,
    requests: Math.floor(Math.random() * 1000),
    timestamp: new Date().toISOString()
  };

  sseService.broadcast('metrics', metrics);
}, 1000);

// Notification endpoint
app.post('/api/notify', (req, res) => {
  const { message, type } = req.body;

  sseService.broadcast('notification', {
    message,
    type,
    timestamp: new Date()
  });

  res.json({ success: true });
});

app.listen(3000);
```

```typescript
// client/src/Dashboard.tsx
import { useEffect, useState } from 'react';

interface Metrics {
  cpu: number;
  memory: number;
  requests: number;
  timestamp: string;
}

export function Dashboard() {
  const [metrics, setMetrics] = useState<Metrics[]>([]);
  const [notifications, setNotifications] = useState<any[]>([]);

  useEffect(() => {
    const eventSource = new EventSource('http://localhost:3000/api/metrics');

    eventSource.addEventListener('metrics', (event) => {
      const data = JSON.parse(event.data);
      setMetrics(prev => [...prev.slice(-50), data]); // Keep last 50
    });

    eventSource.addEventListener('notification', (event) => {
      const data = JSON.parse(event.data);
      setNotifications(prev => [data, ...prev]);
    });

    eventSource.onerror = () => {
      console.error('SSE connection error');
    };

    return () => {
      eventSource.close();
    };
  }, []);

  const latestMetrics = metrics[metrics.length - 1];

  return (
    <div className="dashboard">
      <h1>Live Dashboard</h1>

      <div className="metrics-grid">
        <div className="metric">
          <h3>CPU</h3>
          <div className="value">{latestMetrics?.cpu.toFixed(1)}%</div>
        </div>

        <div className="metric">
          <h3>Memory</h3>
          <div className="value">{latestMetrics?.memory.toFixed(1)}%</div>
        </div>

        <div className="metric">
          <h3>Requests</h3>
          <div className="value">{latestMetrics?.requests}</div>
        </div>
      </div>

      <div className="chart">
        <LineChart data={metrics} />
      </div>

      <div className="notifications">
        <h3>Notifications</h3>
        {notifications.map((notif, i) => (
          <div key={i} className={`notification ${notif.type}`}>
            {notif.message}
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Zusammenfassung

Du hast nun gelernt:

✅ **Echtzeit-Kommunikation**: HTTP Polling vs WebSocket vs SSE
✅ **WebSocket Grundlagen**: Lifecycle, Frames, Close Codes
✅ **ws Library**: Native WebSocket-Server implementieren
✅ **Socket.io**: High-Level Features (Rooms, Namespaces, Middleware)
✅ **WebSocket Client**: Browser & React Integration
✅ **WebSocket Best Practices**: Heartbeat, Rate Limiting, Graceful Shutdown
✅ **Server-Sent Events**: Unidirektionale Push-Kommunikation
✅ **SSE Implementation**: Server & Client mit Event Replay
✅ **Authentication**: JWT-basierte Auth für WebSocket & SSE
✅ **Skalierung**: Redis Pub/Sub für Multi-Server Setup
✅ **Testing**: WebSocket & SSE Tests schreiben
✅ **Troubleshooting**: Häufige Probleme lösen
✅ **Praktische Projekte**: Chat-App & Live Dashboard

### Wann was verwenden?

**WebSockets:**
- Chat-Anwendungen
- Gaming
- Collaborative Tools
- Bidirektionale Kommunikation

**SSE:**
- Live-Feeds
- Dashboard-Updates
- Notifications
- Server → Client nur

**HTTP Polling:**
- Keine Echtzeit-Anforderungen
- Einfache Updates
- Legacy-Browser Support

### Hilfreiche Ressourcen

- [WebSocket MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
- [Socket.io Documentation](https://socket.io/docs/v4/)
- [EventSource MDN](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)
- [ws Library](https://github.com/websockets/ws)

Viel Erfolg mit Echtzeit-Kommunikation! 🚀
