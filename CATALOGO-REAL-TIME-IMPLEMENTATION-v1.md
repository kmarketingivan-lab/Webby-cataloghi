# CATALOGO REAL-TIME IMPLEMENTATION v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: WebSocket, Socket.io, SSE, Presence system, Room management, Scaling patterns

---

## 1. TECHNOLOGY COMPARISON

| Technology            | Protocol       | Bidirectional | Latency  | Scalability | Browser Support | Use Case                   |
| --------------------- | -------------- | ------------- | -------- | ----------- | --------------- | -------------------------- |
| WebSocket             | TCP            | ✅            | Low      | High        | ✅              | Chat, games, notifications |
| Socket.io             | TCP + fallback | ✅            | Low      | High        | ✅              | Multi-room chat, presence  |
| SSE                   | HTTP           | ❌            | Medium   | Medium      | ✅              | Live feeds, dashboards     |
| Long Polling          | HTTP           | ❌            | High     | Low         | ✅              | Legacy browsers            |
| WebRTC Data Channels  | UDP            | ✅            | Very Low | Medium      | ✅              | P2P file sharing, gaming   |
| GraphQL Subscriptions | WS or SSE      | ✅            | Low      | Medium      | ✅              | Real-time GraphQL APIs     |

### 1.1 Decision Matrix

| Requirement          | WebSocket | Socket.io | SSE     | Long Polling |
| -------------------- | --------- | --------- | ------- | ------------ |
| Bidirectional        | ✅        | ✅        | ❌      | ❌           |
| Auto-reconnect       | ❌ Manual | ✅ Built-in| ✅ Built-in | ❌ Manual |
| Fallback support     | ❌        | ✅        | ❌      | ✅           |
| Room/namespace       | ❌ Manual | ✅ Built-in| ❌      | ❌           |
| Binary data          | ✅        | ✅        | ❌      | ❌           |
| Complexity           | Medium    | Low       | Low     | High         |
| Bundle size          | 0 KB      | ~40 KB    | 0 KB    | 0 KB         |

---

## 2. WEBSOCKET IMPLEMENTATION

### 2.1 Server (Node.js + ws)

```typescript
import WebSocket, { WebSocketServer } from 'ws';
import jwt from 'jsonwebtoken';
import { IncomingMessage } from 'http';

interface WSClient extends WebSocket {
  userId?: string;
  rooms: Set<string>;
  isAlive: boolean;
}

const wss = new WebSocketServer({ port: 8080 });
const clients = new Set<WSClient>();

// Connection handler
wss.on('connection', (ws: WSClient, req: IncomingMessage) => {
  // Authentication via protocol header
  const token = req.headers['sec-websocket-protocol'] as string;
  
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as { id: string };
    ws.userId = payload.id;
    ws.rooms = new Set();
    ws.isAlive = true;
  } catch {
    ws.close(4001, 'Unauthorized');
    return;
  }

  // Heartbeat
  ws.on('pong', () => {
    ws.isAlive = true;
  });

  // Message handler
  ws.on('message', (raw) => {
    try {
      const data = JSON.parse(raw.toString());
      
      switch (data.action) {
        case 'join':
          ws.rooms.add(data.roomId);
          break;
        case 'leave':
          ws.rooms.delete(data.roomId);
          break;
        case 'message':
          broadcastToRoom(data.roomId, data, ws);
          break;
        case 'typing':
          broadcastToRoom(data.roomId, { action: 'typing', userId: ws.userId }, ws);
          break;
      }
    } catch (err) {
      ws.send(JSON.stringify({ error: 'Invalid message format' }));
    }
  });

  clients.add(ws);
  
  ws.on('close', () => {
    clients.delete(ws);
  });
});

// Broadcast to room
function broadcastToRoom(roomId: string, data: any, sender?: WSClient) {
  for (const client of clients) {
    if (
      client.readyState === WebSocket.OPEN &&
      client.rooms.has(roomId) &&
      client !== sender
    ) {
      client.send(JSON.stringify(data));
    }
  }
}

// Heartbeat interval (30 seconds)
setInterval(() => {
  wss.clients.forEach((ws) => {
    const client = ws as WSClient;
    if (!client.isAlive) {
      client.terminate();
      return;
    }
    client.isAlive = false;
    client.ping();
  });
}, 30000);
```

### 2.2 Client React Hook

```typescript
import { useState, useEffect, useRef, useCallback } from 'react';

interface WebSocketMessage {
  action: string;
  [key: string]: any;
}

interface UseWebSocketOptions {
  reconnectInterval?: number;
  maxReconnectAttempts?: number;
}

export function useWebSocket(
  url: string,
  token: string,
  options: UseWebSocketOptions = {}
) {
  const { reconnectInterval = 3000, maxReconnectAttempts = 5 } = options;
  
  const wsRef = useRef<WebSocket | null>(null);
  const reconnectCount = useRef(0);
  const messageQueue = useRef<WebSocketMessage[]>([]);
  
  const [connected, setConnected] = useState(false);
  const [messages, setMessages] = useState<WebSocketMessage[]>([]);

  const connect = useCallback(() => {
    const ws = new WebSocket(url, token);
    wsRef.current = ws;

    ws.onopen = () => {
      setConnected(true);
      reconnectCount.current = 0;
      
      // Flush queued messages
      messageQueue.current.forEach((msg) => {
        ws.send(JSON.stringify(msg));
      });
      messageQueue.current = [];
    };

    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setMessages((prev) => [...prev, data]);
    };

    ws.onclose = () => {
      setConnected(false);
      
      // Auto-reconnect
      if (reconnectCount.current < maxReconnectAttempts) {
        reconnectCount.current++;
        setTimeout(connect, reconnectInterval);
      }
    };

    ws.onerror = () => {
      ws.close();
    };
  }, [url, token, reconnectInterval, maxReconnectAttempts]);

  useEffect(() => {
    connect();
    return () => wsRef.current?.close();
  }, [connect]);

  const sendMessage = useCallback((msg: WebSocketMessage) => {
    if (connected && wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify(msg));
    } else {
      messageQueue.current.push(msg);
    }
  }, [connected]);

  const joinRoom = useCallback((roomId: string) => {
    sendMessage({ action: 'join', roomId });
  }, [sendMessage]);

  const leaveRoom = useCallback((roomId: string) => {
    sendMessage({ action: 'leave', roomId });
  }, [sendMessage]);

  return {
    connected,
    messages,
    sendMessage,
    joinRoom,
    leaveRoom,
  };
}
```

---

## 3. SOCKET.IO IMPLEMENTATION

### 3.1 Server Setup with Redis Adapter

```typescript
import { Server } from 'socket.io';
import { createServer } from 'http';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';
import jwt from 'jsonwebtoken';

const httpServer = createServer();
const io = new Server(httpServer, {
  cors: {
    origin: process.env.CORS_ORIGIN || '*',
    methods: ['GET', 'POST'],
  },
  pingTimeout: 60000,
  pingInterval: 25000,
});

// Redis adapter for horizontal scaling
const pubClient = createClient({ url: process.env.REDIS_URL });
const subClient = pubClient.duplicate();

Promise.all([pubClient.connect(), subClient.connect()]).then(() => {
  io.adapter(createAdapter(pubClient, subClient));
});

// Authentication middleware
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  
  if (!token) {
    return next(new Error('Authentication required'));
  }
  
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as { userId: string };
    socket.data.userId = payload.userId;
    next();
  } catch {
    next(new Error('Invalid token'));
  }
});

// Connection handler
io.on('connection', (socket) => {
  console.log(`User connected: ${socket.data.userId}`);

  // Join room
  socket.on('join', async (roomId: string) => {
    await socket.join(roomId);
    socket.to(roomId).emit('user_joined', {
      userId: socket.data.userId,
      roomId,
    });
  });

  // Leave room
  socket.on('leave', async (roomId: string) => {
    await socket.leave(roomId);
    socket.to(roomId).emit('user_left', {
      userId: socket.data.userId,
      roomId,
    });
  });

  // Send message
  socket.on('message', (data: { roomId: string; content: string }) => {
    const message = {
      id: crypto.randomUUID(),
      roomId: data.roomId,
      senderId: socket.data.userId,
      content: data.content,
      createdAt: new Date().toISOString(),
    };
    
    io.to(data.roomId).emit('message', message);
  });

  // Typing indicator
  socket.on('typing', (roomId: string) => {
    socket.to(roomId).emit('typing', {
      userId: socket.data.userId,
      roomId,
    });
  });

  // Disconnect
  socket.on('disconnect', () => {
    console.log(`User disconnected: ${socket.data.userId}`);
  });
});

httpServer.listen(3000, () => {
  console.log('Socket.io server running on port 3000');
});
```

### 3.2 Client React Hook

```typescript
import { useEffect, useState, useCallback, useRef } from 'react';
import { io, Socket } from 'socket.io-client';

interface Message {
  id: string;
  roomId: string;
  senderId: string;
  content: string;
  createdAt: string;
}

interface UseSocketOptions {
  autoConnect?: boolean;
}

export function useSocket(url: string, token: string, options: UseSocketOptions = {}) {
  const { autoConnect = true } = options;
  
  const socketRef = useRef<Socket | null>(null);
  const [connected, setConnected] = useState(false);
  const [messages, setMessages] = useState<Message[]>([]);
  const [typingUsers, setTypingUsers] = useState<Set<string>>(new Set());

  useEffect(() => {
    const socket = io(url, {
      auth: { token },
      autoConnect,
      reconnection: true,
      reconnectionAttempts: 5,
      reconnectionDelay: 1000,
    });

    socketRef.current = socket;

    socket.on('connect', () => setConnected(true));
    socket.on('disconnect', () => setConnected(false));
    
    socket.on('message', (msg: Message) => {
      setMessages((prev) => [...prev, msg]);
    });

    socket.on('typing', ({ userId }: { userId: string }) => {
      setTypingUsers((prev) => new Set(prev).add(userId));
      
      // Clear typing after 3 seconds
      setTimeout(() => {
        setTypingUsers((prev) => {
          const next = new Set(prev);
          next.delete(userId);
          return next;
        });
      }, 3000);
    });

    return () => {
      socket.disconnect();
    };
  }, [url, token, autoConnect]);

  const send = useCallback((roomId: string, content: string) => {
    socketRef.current?.emit('message', { roomId, content });
  }, []);

  const join = useCallback((roomId: string) => {
    socketRef.current?.emit('join', roomId);
  }, []);

  const leave = useCallback((roomId: string) => {
    socketRef.current?.emit('leave', roomId);
  }, []);

  const sendTyping = useCallback((roomId: string) => {
    socketRef.current?.emit('typing', roomId);
  }, []);

  return {
    socket: socketRef.current,
    connected,
    messages,
    typingUsers: Array.from(typingUsers),
    send,
    join,
    leave,
    sendTyping,
  };
}
```

---

## 4. SERVER-SENT EVENTS (SSE)

### 4.1 Next.js App Router SSE Endpoint

```typescript
// app/api/events/route.ts
import { NextRequest } from 'next/server';

export const runtime = 'nodejs';
export const dynamic = 'force-dynamic';

export async function GET(req: NextRequest) {
  const encoder = new TextEncoder();
  
  const stream = new ReadableStream({
    start(controller) {
      // Send initial connection event
      controller.enqueue(
        encoder.encode(`event: connected\ndata: ${JSON.stringify({ status: 'ok' })}\n\n`)
      );

      // Heartbeat every 30 seconds
      const heartbeat = setInterval(() => {
        controller.enqueue(
          encoder.encode(`event: heartbeat\ndata: ${JSON.stringify({ time: Date.now() })}\n\n`)
        );
      }, 30000);

      // Example: Subscribe to Redis pub/sub or other event source
      const sendEvent = (event: string, data: any) => {
        controller.enqueue(
          encoder.encode(`event: ${event}\ndata: ${JSON.stringify(data)}\n\n`)
        );
      };

      // Cleanup on close
      req.signal.addEventListener('abort', () => {
        clearInterval(heartbeat);
        controller.close();
      });
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache, no-transform',
      'Connection': 'keep-alive',
    },
  });
}
```

### 4.2 SSE Client Hook

```typescript
import { useEffect, useState, useCallback, useRef } from 'react';

interface SSEOptions {
  withCredentials?: boolean;
  onError?: (error: Event) => void;
}

export function useSSE<T = any>(url: string, options: SSEOptions = {}) {
  const { withCredentials = false, onError } = options;
  
  const eventSourceRef = useRef<EventSource | null>(null);
  const [connected, setConnected] = useState(false);
  const [events, setEvents] = useState<T[]>([]);
  const [lastEvent, setLastEvent] = useState<T | null>(null);

  const connect = useCallback(() => {
    const es = new EventSource(url, { withCredentials });
    eventSourceRef.current = es;

    es.onopen = () => setConnected(true);

    es.onmessage = (event) => {
      const data = JSON.parse(event.data) as T;
      setEvents((prev) => [...prev, data]);
      setLastEvent(data);
    };

    es.addEventListener('heartbeat', () => {
      // Keep-alive received
    });

    es.onerror = (error) => {
      setConnected(false);
      onError?.(error);
      
      // Auto-reconnect after 5 seconds
      setTimeout(() => {
        es.close();
        connect();
      }, 5000);
    };
  }, [url, withCredentials, onError]);

  useEffect(() => {
    connect();
    
    return () => {
      eventSourceRef.current?.close();
    };
  }, [connect]);

  const close = useCallback(() => {
    eventSourceRef.current?.close();
    setConnected(false);
  }, []);

  return {
    connected,
    events,
    lastEvent,
    close,
  };
}
```

---

## 5. PRESENCE SYSTEM

| Pattern           | Storage    | Latency | Accuracy | Scalability | Best For           |
| ----------------- | ---------- | ------- | -------- | ----------- | ------------------ |
| Heartbeat         | Redis      | Low     | Medium   | High        | Online status      |
| WebSocket ping    | In-memory  | Low     | High     | Medium      | Single server      |
| Activity tracking | Redis + DB | Medium  | High     | High        | Last seen          |
| Typing indicators | Memory     | Low     | Medium   | Medium      | Real-time feedback |

### 5.1 Redis Presence Implementation

```typescript
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

const PRESENCE_TTL = 60; // seconds
const PRESENCE_KEY = 'presence';

export async function setOnline(userId: string): Promise<void> {
  await redis.zadd(PRESENCE_KEY, Date.now(), userId);
}

export async function setOffline(userId: string): Promise<void> {
  await redis.zrem(PRESENCE_KEY, userId);
}

export async function getOnlineUsers(): Promise<string[]> {
  const threshold = Date.now() - PRESENCE_TTL * 1000;
  
  // Remove stale entries
  await redis.zremrangebyscore(PRESENCE_KEY, '-inf', threshold);
  
  // Get current online users
  return redis.zrange(PRESENCE_KEY, 0, -1);
}

export async function isOnline(userId: string): Promise<boolean> {
  const score = await redis.zscore(PRESENCE_KEY, userId);
  if (!score) return false;
  
  const threshold = Date.now() - PRESENCE_TTL * 1000;
  return parseInt(score) > threshold;
}

export async function getLastSeen(userId: string): Promise<Date | null> {
  const score = await redis.zscore(PRESENCE_KEY, userId);
  return score ? new Date(parseInt(score)) : null;
}

// Heartbeat - call every 30 seconds from client
export async function heartbeat(userId: string): Promise<void> {
  await setOnline(userId);
}
```

---

## 6. MESSAGE SCHEMAS

### 6.1 Chat Message

```typescript
export interface Message {
  id: string;
  roomId: string;
  senderId: string;
  content: string;
  type: 'text' | 'image' | 'file' | 'system';
  metadata?: {
    fileName?: string;
    fileSize?: number;
    mimeType?: string;
    thumbnailUrl?: string;
  };
  replyTo?: string;
  createdAt: Date;
  updatedAt?: Date;
  status: 'sending' | 'sent' | 'delivered' | 'read' | 'failed';
  readBy: string[];
}
```

### 6.2 Room Management

```typescript
export interface Room {
  id: string;
  name: string;
  type: 'direct' | 'group' | 'channel';
  members: RoomMember[];
  createdAt: Date;
  updatedAt: Date;
  lastMessageAt?: Date;
  metadata?: Record<string, any>;
}

export interface RoomMember {
  userId: string;
  role: 'owner' | 'admin' | 'member';
  joinedAt: Date;
  lastReadAt?: Date;
}

// Room service
class RoomService {
  private rooms = new Map<string, Room>();

  create(id: string, name: string, creatorId: string, type: Room['type'] = 'group'): Room {
    const room: Room = {
      id,
      name,
      type,
      members: [{ userId: creatorId, role: 'owner', joinedAt: new Date() }],
      createdAt: new Date(),
      updatedAt: new Date(),
    };
    this.rooms.set(id, room);
    return room;
  }

  join(roomId: string, userId: string): boolean {
    const room = this.rooms.get(roomId);
    if (!room) return false;
    
    if (room.members.some((m) => m.userId === userId)) return true;
    
    room.members.push({ userId, role: 'member', joinedAt: new Date() });
    room.updatedAt = new Date();
    return true;
  }

  leave(roomId: string, userId: string): boolean {
    const room = this.rooms.get(roomId);
    if (!room) return false;
    
    room.members = room.members.filter((m) => m.userId !== userId);
    room.updatedAt = new Date();
    return true;
  }

  getMembers(roomId: string): RoomMember[] {
    return this.rooms.get(roomId)?.members || [];
  }
}
```

---

## 7. SCALING PATTERNS

| Pattern               | Technology          | Horizontal | Complexity | Cost   | Use Case                |
| --------------------- | ------------------- | ---------- | ---------- | ------ | ----------------------- |
| Redis Pub/Sub         | WebSocket/Socket.io | ✅         | Medium     | Medium | Multi-instance sync     |
| Kafka Event Stream    | WS/SSE              | ✅         | High       | High   | High-throughput events  |
| Sticky Sessions       | Load balancer       | ✅         | Low        | Medium | Session affinity        |
| Kubernetes Deployment | All                 | ✅         | Medium     | Medium | Container orchestration |

### 7.1 Redis Pub/Sub for Socket.io

```typescript
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';
import { Server } from 'socket.io';

async function setupRedisAdapter(io: Server) {
  const pubClient = createClient({ url: process.env.REDIS_URL });
  const subClient = pubClient.duplicate();

  await Promise.all([pubClient.connect(), subClient.connect()]);

  io.adapter(createAdapter(pubClient, subClient));

  // Graceful shutdown
  process.on('SIGTERM', async () => {
    await pubClient.quit();
    await subClient.quit();
  });
}
```

---

## 8. SECURITY PATTERNS

| Threat              | Mitigation            | Implementation                |
| ------------------- | --------------------- | ----------------------------- |
| Unauthorized access | JWT auth              | Middleware verify token       |
| Room hijack         | Room permissions      | Role-based ACL                |
| Rate limiting       | Redis                 | Limit messages per IP/user    |
| Injection           | Input validation      | Zod schemas                   |
| DoS                 | Connection throttling | Max connections per IP        |
| Message spoofing    | Server-side senderId  | Never trust client userId     |

### 8.1 Rate Limiting Middleware

```typescript
import Redis from 'ioredis';
import { Socket } from 'socket.io';

const redis = new Redis();

interface RateLimitOptions {
  windowMs: number;
  max: number;
}

export function rateLimitMiddleware(options: RateLimitOptions) {
  return async (socket: Socket, next: (err?: Error) => void) => {
    const key = `ws:rate:${socket.handshake.address}`;
    const count = await redis.incr(key);
    
    if (count === 1) {
      await redis.expire(key, Math.ceil(options.windowMs / 1000));
    }
    
    if (count > options.max) {
      return next(new Error('Rate limit exceeded'));
    }
    
    next();
  };
}
```

---

## 9. MONITORING & DEBUGGING

| Metric           | Tool       | Implementation               |
| ---------------- | ---------- | ---------------------------- |
| Connection count | Prometheus | Track via middleware         |
| Message latency  | StatsD     | Measure send/receive         |
| Error logging    | Sentry     | Capture server/client errors |
| Heartbeat        | Custom     | Detect disconnected clients  |

### 9.1 Prometheus Metrics

```typescript
import { Counter, Gauge, Histogram } from 'prom-client';

export const wsConnections = new Gauge({
  name: 'websocket_connections_total',
  help: 'Total WebSocket connections',
  labelNames: ['status'],
});

export const wsMessages = new Counter({
  name: 'websocket_messages_total',
  help: 'Total WebSocket messages',
  labelNames: ['type', 'direction'],
});

export const wsLatency = new Histogram({
  name: 'websocket_message_latency_seconds',
  help: 'WebSocket message latency',
  buckets: [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1],
});
```

---

## 10. REAL-TIME CHECKLIST

| Category | Item |
| -------- | ---- |
| Server | ✅ WebSocket/Socket.io server running |
| Server | ✅ Redis adapter enabled for scaling |
| Server | ✅ SSE fallback implemented |
| Server | ✅ Heartbeat ping/pong active |
| Security | ✅ Authentication middleware |
| Security | ✅ Authorization per room |
| Security | ✅ Rate limiting active |
| Security | ✅ Input validation on messages |
| Features | ✅ Message schema typed |
| Features | ✅ Room management functional |
| Features | ✅ Typing indicators |
| Features | ✅ Presence system active |
| Features | ✅ Read receipts |
| Client | ✅ Auto-reconnect logic |
| Client | ✅ Message queue for offline |
| Client | ✅ Optimistic updates |
| Scaling | ✅ Horizontal scaling tested |
| Scaling | ✅ Load balancer configured |
| Monitoring | ✅ Metrics configured |
| Monitoring | ✅ Error logging integrated |
