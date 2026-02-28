# CATALOGO REAL-TIME IMPLEMENTATION v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: WebSocket, Socket.io, SSE, Presence system, Room management, Scaling patterns

---

§ 1. TECHNOLOGY COMPARISON

| Technology            | Protocol       | Bidirectional | Latency  | Scalability | Browser Support | Use Case                   |
| --------------------- | -------------- | ------------- | -------- | ----------- | --------------- | -------------------------- |
| WebSocket             | TCP            | ✅            | Low      | High        | ✅              | Chat, games, notifications |
| Socket.io             | TCP + fallback | ✅            | Low      | High        | ✅              | Multi-room chat, presence  |
| SSE                   | HTTP           | ❌            | Medium   | Medium      | ✅              | Live feeds, dashboards     |
| Long Polling          | HTTP           | ❌            | High     | Low         | ✅              | Legacy browsers            |
| WebRTC Data Channels  | UDP            | ✅            | Very Low | Medium      | ✅              | P2P file sharing, gaming   |
| GraphQL Subscriptions | WS or SSE      | ✅            | Low      | Medium      | ✅              | Real-time GraphQL APIs     |

§ 1.1 DECISION MATRIX

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

§ 2. WEBSOCKET IMPLEMENTATION

§ 2.1 SERVER (NODE.JS + WS)

typescript
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

§ 2.2 CLIENT REACT HOOK

typescript
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

---

§ 3. SOCKET.IO IMPLEMENTATION

§ 3.1 SERVER SETUP WITH REDIS ADAPTER

typescript
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

§ 3.2 CLIENT REACT HOOK

typescript
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

---

§ 4. SERVER-SENT EVENTS (SSE)

§ 4.1 NEXT.JS APP ROUTER SSE ENDPOINT

typescript
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

§ 4.2 SSE CLIENT HOOK

typescript
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

---

§ 5. PRESENCE SYSTEM

| Pattern           | Storage    | Latency | Accuracy | Scalability | Best For           |
| ----------------- | ---------- | ------- | -------- | ----------- | ------------------ |
| Heartbeat         | Redis      | Low     | Medium   | High        | Online status      |
| WebSocket ping    | In-memory  | Low     | High     | Medium      | Single server      |
| Activity tracking | Redis + DB | Medium  | High     | High        | Last seen          |
| Typing indicators | Memory     | Low     | Medium   | Medium      | Real-time feedback |

§ 5.1 REDIS PRESENCE IMPLEMENTATION

typescript
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

---

§ 6. MESSAGE SCHEMAS

§ 6.1 CHAT MESSAGE

typescript
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

§ 6.2 ROOM MANAGEMENT

typescript
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

---

§ 7. SCALING PATTERNS

| Pattern               | Technology          | Horizontal | Complexity | Cost   | Use Case                |
| --------------------- | ------------------- | ---------- | ---------- | ------ | ----------------------- |
| Redis Pub/Sub         | WebSocket/Socket.io | ✅         | Medium     | Medium | Multi-instance sync     |
| Kafka Event Stream    | WS/SSE              | ✅         | High       | High   | High-throughput events  |
| Sticky Sessions       | Load balancer       | ✅         | Low        | Medium | Session affinity        |
| Kubernetes Deployment | All                 | ✅         | Medium     | Medium | Container orchestration |

§ 7.1 REDIS PUB/SUB FOR SOCKET.IO

typescript
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

---

§ 8. SECURITY PATTERNS

| Threat              | Mitigation            | Implementation                |
| ------------------- | --------------------- | ----------------------------- |
| Unauthorized access | JWT auth              | Middleware verify token       |
| Room hijack         | Room permissions      | Role-based ACL                |
| Rate limiting       | Redis                 | Limit messages per IP/user    |
| Injection           | Input validation      | Zod schemas                   |
| DoS                 | Connection throttling | Max connections per IP        |
| Message spoofing    | Server-side senderId  | Never trust client userId     |

§ 8.1 RATE LIMITING MIDDLEWARE

typescript
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

---

§ 9. MONITORING & DEBUGGING

| Metric           | Tool       | Implementation               |
| ---------------- | ---------- | ---------------------------- |
| Connection count | Prometheus | Track via middleware         |
| Message latency  | StatsD     | Measure send/receive         |
| Error logging    | Sentry     | Capture server/client errors |
| Heartbeat        | Custom     | Detect disconnected clients  |

§ 9.1 PROMETHEUS METRICS

typescript
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

---

§ 10. REAL-TIME CHECKLIST

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

11. NEXT.JS 14 WEBSOCKET INTEGRATION
Custom Server Setup

Next.js App Router non include un server WebSocket nativo. Richiede un custom server o un servizio esterno.

✅ Pattern Consigliato: External WebSocket Server + API Routes
✅ Alternative: Managed service (Pusher, Ably) o Serverless WebSockets (Vercel via Soketi)
❌ Non supportato: WebSocket diretto in API Route /api/socket senza custom server.

Server.ts Custom con Socket.io
typescript
Copia
Scarica
// server.ts (in root, alongside next.config.js)
import { createServer } from 'http';
import { parse } from 'url';
import next from 'next';
import { Server as SocketIOServer } from 'socket.io';
import { createClient } from 'redis';
import { createAdapter } from '@socket.io/redis-adapter';

const dev = process.env.NODE_ENV !== 'production';
const hostname = process.env.HOSTNAME || 'localhost';
const port = parseInt(process.env.PORT || '3000', 10);
const redisUrl = process.env.REDIS_URL || 'redis://localhost:6379';

const app = next({ dev, hostname, port });
const handler = app.getRequestHandler();

app.prepare().then(async () => {
  const httpServer = createServer(async (req, res) => {
    try {
      const parsedUrl = parse(req.url!, true);
      await handler(req, res, parsedUrl);
    } catch (err) {
      console.error('Error handling request:', err);
      res.statusCode = 500;
      res.end('Internal Server Error');
    }
  });

  // Socket.io Server Setup
  const io = new SocketIOServer(httpServer, {
    path: '/api/socketio',
    addTrailingSlash: false,
    cors: {
      origin: dev ? ['http://localhost:3000'] : ['https://yourdomain.com'],
      methods: ['GET', 'POST'],
      credentials: true,
    },
    connectionStateRecovery: {
      maxDisconnectionDuration: 2 * 60 * 1000, // 2 minutes
      skipMiddlewares: true,
    },
  });

  // Redis Adapter for Scaling
  const pubClient = createClient({ url: redisUrl });
  const subClient = pubClient.duplicate();
  
  await Promise.all([pubClient.connect(), subClient.connect()]);
  io.adapter(createAdapter(pubClient, subClient));
  console.log('Redis adapter connected');

  // Socket.io Event Handlers
  io.on('connection', (socket) => {
    console.log(`Client connected: ${socket.id}`);
    
    // Join room based on user ID or room ID
    socket.on('join:room', (roomId: string, userId: string) => {
      socket.join(roomId);
      socket.data.userId = userId;
      io.to(roomId).emit('user:joined', { userId, socketId: socket.id });
    });

    // Presence heartbeat
    socket.on('presence:heartbeat', (userId: string) => {
      socket.data.lastSeen = Date.now();
      socket.broadcast.emit('presence:update', { userId, status: 'online' });
    });

    // Typing indicator
    socket.on('typing:start', (roomId: string, userId: string) => {
      socket.to(roomId).emit('typing:start', userId);
    });

    socket.on('typing:stop', (roomId: string, userId: string) => {
      socket.to(roomId).emit('typing:stop', userId);
    });

    // Message handling
    socket.on('message:send', (message: MessagePayload) => {
      const { roomId, ...msgData } = message;
      io.to(roomId).emit('message:new', {
        ...msgData,
        timestamp: Date.now(),
        delivered: true,
      });
    });

    // Disconnect handler
    socket.on('disconnect', (reason) => {
      console.log(`Client disconnected: ${socket.id} (${reason})`);
      if (socket.data.userId) {
        socket.broadcast.emit('presence:update', { 
          userId: socket.data.userId, 
          status: 'offline' 
        });
      }
    });
  });

  // Graceful shutdown
  const gracefulShutdown = () => {
    io.close(() => {
      console.log('Socket.IO server closed');
      httpServer.close(() => {
        console.log('HTTP server closed');
        process.exit(0);
      });
    });
  };

  process.on('SIGTERM', gracefulShutdown);
  process.on('SIGINT', gracefulShutdown);

  httpServer
    .once('error', (err) => {
      console.error('Server error:', err);
      process.exit(1);
    })
    .listen(port, () => {
      console.log(`> Ready on http://${hostname}:${port}`);
      console.log(`> WebSocket path: /api/socketio`);
    });
});
Client Hook: useSocket
typescript
Copia
Scarica
// hooks/useSocket.ts
import { useEffect, useRef, useState, useCallback } from 'react';
import { io, Socket } from 'socket.io-client';

type SocketStatus = 'connecting' | 'connected' | 'disconnected' | 'reconnecting';
type MessageHandler = (data: any) => void;

interface UseSocketOptions {
  autoConnect?: boolean;
  reconnection?: boolean;
  reconnectionAttempts?: number;
  reconnectionDelay?: number;
  reconnectionDelayMax?: number;
  path?: string;
}

interface UseSocketReturn {
  socket: Socket | null;
  status: SocketStatus;
  connect: () => void;
  disconnect: () => void;
  emit: (event: string, data: any, ack?: (response: any) => void) => void;
  on: (event: string, handler: MessageHandler) => void;
  off: (event: string, handler?: MessageHandler) => void;
  joinRoom: (roomId: string, userId: string) => void;
  leaveRoom: (roomId: string) => void;
  isConnected: boolean;
  error: Error | null;
}

const SOCKET_SERVER_URL = process.env.NEXT_PUBLIC_WS_URL || 
  (typeof window !== 'undefined' ? window.location.origin : '');

export const useSocket = (options: UseSocketOptions = {}): UseSocketReturn => {
  const [status, setStatus] = useState<SocketStatus>('disconnected');
  const [error, setError] = useState<Error | null>(null);
  const socketRef = useRef<Socket | null>(null);
  const eventHandlersRef = useRef<Map<string, MessageHandler[]>>(new Map());
  const reconnectAttemptsRef = useRef(0);

  const {
    autoConnect = true,
    reconnection = true,
    reconnectionAttempts = 5,
    reconnectionDelay = 1000,
    reconnectionDelayMax = 5000,
    path = '/api/socketio',
  } = options;

  // Initialize socket
  const initializeSocket = useCallback(() => {
    if (socketRef.current?.connected) {
      return socketRef.current;
    }

    const socket = io(SOCKET_SERVER_URL, {
      path,
      autoConnect: false,
      reconnection,
      reconnectionAttempts,
      reconnectionDelay,
      reconnectionDelayMax,
      timeout: 20000,
      transports: ['websocket', 'polling'],
      withCredentials: true,
      auth: (cb) => {
        const token = localStorage.getItem('accessToken');
        cb({ token });
      },
    });

    // Connection events
    socket.on('connect', () => {
      console.log('Socket connected:', socket.id);
      setStatus('connected');
      setError(null);
      reconnectAttemptsRef.current = 0;
    });

    socket.on('disconnect', (reason) => {
      console.log('Socket disconnected:', reason);
      setStatus('disconnected');
      
      if (reason === 'io server disconnect') {
        // Manual reconnection needed
        setTimeout(() => {
          socket.connect();
        }, 1000);
      }
    });

    socket.on('connect_error', (err) => {
      console.error('Socket connection error:', err.message);
      setStatus('disconnected');
      setError(err);
      
      // Exponential backoff with jitter
      if (reconnectAttemptsRef.current < reconnectionAttempts) {
        const delay = Math.min(
          reconnectionDelay * Math.pow(1.5, reconnectAttemptsRef.current) +
          Math.random() * 1000,
          reconnectionDelayMax
        );
        setTimeout(() => {
          reconnectAttemptsRef.current += 1;
          socket.connect();
        }, delay);
      }
    });

    socket.on('reconnect_attempt', (attempt) => {
      console.log(`Reconnection attempt ${attempt}`);
      setStatus('reconnecting');
    });

    socket.on('reconnect', (attempt) => {
      console.log(`Reconnected after ${attempt} attempts`);
      setStatus('connected');
      reconnectAttemptsRef.current = 0;
    });

    // Re-register all event handlers
    eventHandlersRef.current.forEach((handlers, event) => {
      handlers.forEach(handler => {
        socket.on(event, handler);
      });
    });

    socketRef.current = socket;
    return socket;
  }, [path, reconnection, reconnectionAttempts, reconnectionDelay, reconnectionDelayMax]);

  // Connection management
  const connect = useCallback(() => {
    const socket = initializeSocket();
    if (!socket.connected) {
      socket.connect();
    }
  }, [initializeSocket]);

  const disconnect = useCallback(() => {
    if (socketRef.current) {
      socketRef.current.disconnect();
      socketRef.current = null;
    }
    setStatus('disconnected');
  }, []);

  // Event management
  const emit = useCallback((event: string, data: any, ack?: (response: any) => void) => {
    if (socketRef.current?.connected) {
      socketRef.current.emit(event, data, ack);
    } else {
      console.warn(`Cannot emit ${event}: socket not connected`);
      // Optionally queue for later
      if (event === 'message:send') {
        const queue = JSON.parse(localStorage.getItem('offlineMessageQueue') || '[]');
        queue.push({ event, data, timestamp: Date.now() });
        localStorage.setItem('offlineMessageQueue', JSON.stringify(queue.slice(-50)));
      }
    }
  }, []);

  const on = useCallback((event: string, handler: MessageHandler) => {
    if (!eventHandlersRef.current.has(event)) {
      eventHandlersRef.current.set(event, []);
    }
    eventHandlersRef.current.get(event)!.push(handler);
    
    if (socketRef.current) {
      socketRef.current.on(event, handler);
    }
  }, []);

  const off = useCallback((event: string, handler?: MessageHandler) => {
    const handlers = eventHandlersRef.current.get(event);
    if (handlers) {
      if (handler) {
        const index = handlers.indexOf(handler);
        if (index > -1) {
          handlers.splice(index, 1);
          if (socketRef.current) {
            socketRef.current.off(event, handler);
          }
        }
      } else {
        handlers.forEach(h => {
          if (socketRef.current) {
            socketRef.current.off(event, h);
          }
        });
        eventHandlersRef.current.delete(event);
      }
    }
  }, []);

  // Room management
  const joinRoom = useCallback((roomId: string, userId: string) => {
    emit('join:room', { roomId, userId });
  }, [emit]);

  const leaveRoom = useCallback((roomId: string) => {
    if (socketRef.current) {
      socketRef.current.emit('leave:room', roomId);
    }
  }, []);

  // Auto-connect on mount
  useEffect(() => {
    if (autoConnect) {
      connect();
    }

    return () => {
      // Clean up event listeners but keep connection
      if (socketRef.current) {
        eventHandlersRef.current.forEach((handlers, event) => {
          handlers.forEach(handler => {
            socketRef.current!.off(event, handler);
          });
        });
      }
    };
  }, [autoConnect, connect]);

  return {
    socket: socketRef.current,
    status,
    connect,
    disconnect,
    emit,
    on,
    off,
    joinRoom,
    leaveRoom,
    isConnected: status === 'connected',
    error,
  };
};

// Types for message payload
interface MessagePayload {
  roomId: string;
  userId: string;
  content: string;
  type: 'text' | 'image' | 'file';
  metadata?: Record<string, any>;
}
Deployment Strategies
Vercel Deployment

⚠️ Limitation: Vercel Serverless Functions hanno timeout di 10-60s, non supportano WebSocket persistenti.

✅ Workarounds:

External WebSocket Server: Host su Railway, Render, DigitalOcean

Pusher/Ably: Managed WebSocket service

Soketi: Self-hosted Pusher-compatible server su fly.io/railway

Vercel Serverless con Edge Functions: Solo per SSE, non WebSocket full-duplex

typescript
Copia
Scarica
// api/socket-proxy/route.ts (SSE fallback for Vercel)
import { NextRequest } from 'next/server';

export const runtime = 'edge';

export async function GET(request: NextRequest) {
  // Create a ReadableStream for SSE
  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    async start(controller) {
      // Connect to external WebSocket server
      const wsUrl = process.env.EXTERNAL_WS_URL;
      if (!wsUrl) {
        controller.enqueue(encoder.encode('data: {"error": "No WS server"}\n\n'));
        controller.close();
        return;
      }

      try {
        const ws = new WebSocket(wsUrl);
        
        ws.onopen = () => {
          controller.enqueue(encoder.encode('data: {"status": "connected"}\n\n'));
        };
        
        ws.onmessage = (event) => {
          controller.enqueue(encoder.encode(`data: ${event.data}\n\n`));
        };
        
        ws.onclose = () => {
          controller.close();
        };
        
        ws.onerror = (error) => {
          controller.enqueue(encoder.encode(`data: {"error": "${error}"}\n\n`));
          controller.close();
        };
        
        // Cleanup on client disconnect
        request.signal.addEventListener('abort', () => {
          ws.close();
          controller.close();
        });
      } catch (error) {
        controller.enqueue(encoder.encode(`data: {"error": "${error}"}\n\n`));
        controller.close();
      }
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
      'X-Accel-Buffering': 'no',
    },
  });
}
Docker Deployment
dockerfile
Copia
Scarica
# Dockerfile
FROM node:18-alpine AS base

# Install dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Build application
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV PORT=3000
ENV HOSTNAME=0.0.0.0

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
EXPOSE 3001  # WebSocket port if different

CMD ["node", "server.js"]
yaml
Copia
Scarica
# docker-compose.yml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

  app:
    build: .
    ports:
      - "3000:3000"
      - "3001:3001"  # WebSocket port
    environment:
      - NODE_ENV=production
      - REDIS_URL=redis://redis:6379
      - HOSTNAME=0.0.0.0
    depends_on:
      - redis
    restart: unless-stopped

volumes:
  redis_data:
Configuration per Ambiente
typescript
Copia
Scarica
// lib/config.ts
export const getWebSocketConfig = () => {
  const isBrowser = typeof window !== 'undefined';
  const isProduction = process.env.NODE_ENV === 'production';
  
  if (isProduction) {
    // Production: external WebSocket server or managed service
    if (process.env.NEXT_PUBLIC_WS_PROVIDER === 'external') {
      return {
        url: process.env.NEXT_PUBLIC_EXTERNAL_WS_URL!,
        path: '/api/socketio',
        autoConnect: true,
      };
    } else if (process.env.NEXT_PUBLIC_WS_PROVIDER === 'pusher') {
      return {
        provider: 'pusher',
        key: process.env.NEXT_PUBLIC_PUSHER_KEY!,
        cluster: process.env.NEXT_PUBLIC_PUSHER_CLUSTER!,
      };
    }
  }
  
  // Development: local server
  return {
    url: isBrowser ? window.location.origin : 'http://localhost:3000',
    path: '/api/socketio',
    autoConnect: true,
  };
};
Performance Optimization
typescript
Copia
Scarica
// hooks/useOptimizedSocket.ts
import { throttle, debounce } from 'lodash';
import { useSocket } from './useSocket';

export const useOptimizedSocket = () => {
  const socket = useSocket();
  
  // Throttle high-frequency events
  const throttledEmit = useMemo(
    () => throttle(socket.emit, 100, { leading: true, trailing: true }),
    [socket.emit]
  );
  
  // Debounce typing indicators
  const debouncedTypingStart = useMemo(
    () => debounce((roomId: string, userId: string) => {
      socket.emit('typing:start', { roomId, userId });
    }, 300),
    [socket.emit]
  );
  
  const debouncedTypingStop = useMemo(
    () => debounce((roomId: string, userId: string) => {
      socket.emit('typing:stop', { roomId, userId });
    }, 1000),
    [socket.emit]
  );
  
  // Batch messages for real-time dashboard
  const batchMessages = useMemo(() => {
    let batch: any[] = [];
    let batchTimeout: NodeJS.Timeout | null = null;
    
    return (event: string, data: any) => {
      batch.push(data);
      
      if (!batchTimeout) {
        batchTimeout = setTimeout(() => {
          socket.emit(`${event}:batch`, batch);
          batch = [];
          batchTimeout = null;
        }, 100); // 100ms batching window
      }
    };
  }, [socket.emit]);
  
  return {
    ...socket,
    throttledEmit,
    debouncedTypingStart,
    debouncedTypingStop,
    batchMessages,
  };
};
12. PUSHER/ABLY MANAGED ALTERNATIVES
Pusher Channels Integration
typescript
Copia
Scarica
// lib/pusher/server.ts
import Pusher from 'pusher';
import { createClient } from 'redis';

export const pusherServer = new Pusher({
  appId: process.env.PUSHER_APP_ID!,
  key: process.env.PUSHER_KEY!,
  secret: process.env.PUSHER_SECRET!,
  cluster: process.env.PUSHER_CLUSTER!,
  useTLS: true,
  encryptionMasterKeyBase64: process.env.PUSHER_ENCRYPTION_MASTER_KEY,
});

// Redis persistence for Pusher
const redisClient = createClient({
  url: process.env.REDIS_URL,
});

await redisClient.connect();

// Webhook handler for channel events
export async function handlePusherWebhook(body: any) {
  const events = body.events;
  
  for (const event of events) {
    switch (event.name) {
      case 'channel_occupied':
        await redisClient.sAdd('active-channels', event.channel);
        break;
      case 'channel_vacated':
        await redisClient.sRem('active-channels', event.channel);
        break;
      case 'member_added':
        await redisClient.hSet(`channel:${event.channel}:members`, event.user_id, Date.now());
        break;
      case 'member_removed':
        await redisClient.hDel(`channel:${event.channel}:members`, event.user_id);
        break;
    }
  }
}

// Authentication endpoint for private/presence channels
export async function authorizeChannel(
  socketId: string,
  channelName: string,
  userId?: string
) {
  if (channelName.startsWith('private-')) {
    // Verify user has access to private channel
    const authResponse = pusherServer.authorizeChannel(socketId, channelName);
    return authResponse;
  }
  
  if (channelName.startsWith('presence-')) {
    if (!userId) {
      throw new Error('User ID required for presence channels');
    }
    
    const userData = {
      user_id: userId,
      user_info: {
        name: await getUserName(userId),
        avatar: await getUserAvatar(userId),
      },
    };
    
    const authResponse = pusherServer.authorizeChannel(socketId, channelName, userData);
    return authResponse;
  }
  
  throw new Error('Unauthorized channel type');
}
typescript
Copia
Scarica
// hooks/usePusher.ts
import { useEffect, useRef, useState, useCallback } from 'react';
import PusherClient, { Channel, PresenceChannel } from 'pusher-js';

interface UsePusherOptions {
  cluster: string;
  authEndpoint?: string;
  auth?: {
    headers?: Record<string, string>;
    params?: Record<string, string>;
  };
  forceTLS?: boolean;
}

interface UsePusherReturn {
  client: PusherClient | null;
  subscribe: (channelName: string) => Channel | PresenceChannel | null;
  unsubscribe: (channelName: string) => void;
  sendEvent: (channel: string, event: string, data: any) => void;
  bindEvent: (channel: string, event: string, callback: (data: any) => void) => void;
  getMembers: (channelName: string) => any[];
  status: 'connected' | 'disconnected' | 'connecting';
}

export const usePusher = (options: UsePusherOptions): UsePusherReturn => {
  const [status, setStatus] = useState<'connected' | 'disconnected' | 'connecting'>('disconnected');
  const pusherRef = useRef<PusherClient | null>(null);
  const channelsRef = useRef<Map<string, Channel>>(new Map());
  const eventHandlersRef = useRef<Map<string, Map<string, Function>>>(new Map());

  const initialize = useCallback(() => {
    if (pusherRef.current) {
      return pusherRef.current;
    }

    const pusher = new PusherClient(process.env.NEXT_PUBLIC_PUSHER_KEY!, {
      cluster: options.cluster,
      authEndpoint: options.authEndpoint || '/api/pusher/auth',
      auth: options.auth,
      forceTLS: options.forceTLS ?? true,
      enabledTransports: ['ws', 'wss'],
      disabledTransports: ['xhr_streaming', 'xhr_polling'],
      activityTimeout: 60000,
      pongTimeout: 30000,
    });

    pusher.connection.bind('state_change', (states: any) => {
      setStatus(states.current as any);
    });

    pusher.connection.bind('error', (err: any) => {
      console.error('Pusher connection error:', err);
    });

    pusherRef.current = pusher;
    return pusher;
  }, [options]);

  const subscribe = useCallback((channelName: string) => {
    const pusher = initialize();
    
    if (channelsRef.current.has(channelName)) {
      return channelsRef.current.get(channelName)!;
    }

    let channel: Channel;
    
    if (channelName.startsWith('presence-')) {
      channel = pusher.subscribe(channelName) as PresenceChannel;
      
      channel.bind('pusher:subscription_succeeded', (members: any) => {
        console.log(`Joined presence channel ${channelName} with ${members.count} members`);
      });
      
      channel.bind('pusher:member_added', (member: any) => {
        console.log('Member added:', member);
      });
      
      channel.bind('pusher:member_removed', (member: any) => {
        console.log('Member removed:', member);
      });
    } else if (channelName.startsWith('private-')) {
      channel = pusher.subscribe(channelName);
      
      channel.bind('pusher:subscription_error', (status: number) => {
        console.error(`Subscription error to ${channelName}:`, status);
      });
    } else {
      channel = pusher.subscribe(channelName);
    }

    channelsRef.current.set(channelName, channel);
    return channel;
  }, [initialize]);

  const unsubscribe = useCallback((channelName: string) => {
    const pusher = pusherRef.current;
    if (pusher && channelsRef.current.has(channelName)) {
      pusher.unsubscribe(channelName);
      channelsRef.current.delete(channelName);
      eventHandlersRef.current.delete(channelName);
    }
  }, []);

  const sendEvent = useCallback((channel: string, event: string, data: any) => {
    // Note: Client can only trigger events on private/presence channels
    // For public channels, use server-side Pusher instance
    const channelInstance = channelsRef.current.get(channel);
    if (channelInstance && channel.startsWith('private-')) {
      channelInstance.trigger(`client-${event}`, data);
    } else {
      console.warn('Client can only trigger events on private/presence channels');
    }
  }, []);

  const bindEvent = useCallback((
    channelName: string,
    event: string,
    callback: (data: any) => void
  ) => {
    const channel = channelsRef.current.get(channelName);
    if (channel) {
      channel.bind(event, callback);
      
      if (!eventHandlersRef.current.has(channelName)) {
        eventHandlersRef.current.set(channelName, new Map());
      }
      eventHandlersRef.current.get(channelName)!.set(event, callback);
    }
  }, []);

  const getMembers = useCallback((channelName: string): any[] => {
    const channel = channelsRef.current.get(channelName);
    if (channel && 'members' in channel) {
      return (channel as PresenceChannel).members.members;
    }
    return [];
  }, []);

  // Cleanup on unmount
  useEffect(() => {
    const pusher = initialize();
    
    return () => {
      channelsRef.current.forEach((_, channelName) => {
        unsubscribe(channelName);
      });
      pusher.disconnect();
    };
  }, [initialize, unsubscribe]);

  return {
    client: pusherRef.current,
    subscribe,
    unsubscribe,
    sendEvent,
    bindEvent,
    getMembers,
    status,
  };
};
Ably Realtime Integration
typescript
Copia
Scarica
// lib/ably/server.ts
import Ably from 'ably/promises';
import { createClient } from 'redis';

export const ablyServer = new Ably.Rest({
  key: process.env.ABLY_API_KEY!,
  clientId: 'server',
});

// Redis for presence persistence
const redisClient = createClient({
  url: process.env.REDIS_URL,
});

await redisClient.connect();

// Token generation for clients
export async function createTokenRequest(clientId: string, capabilities?: any) {
  const tokenParams = {
    clientId,
    capability: capabilities || {
      'presence:*': ['presence', 'subscribe'],
      'chat:*': ['publish', 'subscribe'],
      'notifications:*': ['subscribe'],
    },
    ttl: 3600000, // 1 hour
  };
  
  return await ablyServer.auth.createTokenRequest(tokenParams);
}

// Presence state management
export async function updatePresence(
  channelName: string,
  clientId: string,
  data: any
) {
  const channel = ablyServer.channels.get(channelName);
  
  // Update in Ably
  await channel.presence.update(data);
  
  // Persist to Redis
  await redisClient.hSet(
    `ably:presence:${channelName}`,
    clientId,
    JSON.stringify({ ...data, timestamp: Date.now() })
  );
  
  // Set expiration for stale presence
  await redisClient.expire(`ably:presence:${channelName}`, 65); // 5 seconds grace
}

// Get channel presence from Redis
export async function getChannelPresence(channelName: string) {
  const presence = await redisClient.hGetAll(`ably:presence:${channelName}`);
  return Object.entries(presence).map(([clientId, data]) => ({
    clientId,
    ...JSON.parse(data),
  }));
}
typescript
Copia
Scarica
// hooks/useAbly.ts
import { useEffect, useRef, useState, useCallback } from 'react';
import Ably from 'ably/promises';
import * as AblyTypes from 'ably';

interface UseAblyOptions {
  clientId?: string;
  authUrl?: string;
  echoMessages?: boolean;
  autoConnect?: boolean;
}

interface UseAblyReturn {
  client: AblyTypes.Types.RealtimePromise | null;
  channel: (name: string, options?: AblyTypes.Types.ChannelOptions) => AblyTypes.Types.RealtimeChannelPromise;
  presence: (channelName: string) => AblyTypes.Types.PresencePromise;
  status: AblyTypes.Types.ConnectionState;
  connect: () => void;
  disconnect: () => void;
}

export const useAbly = (options: UseAblyOptions = {}): UseAblyReturn => {
  const [status, setStatus] = useState<AblyTypes.Types.ConnectionState>('initialized');
  const ablyRef = useRef<AblyTypes.Types.RealtimePromise | null>(null);
  const channelsRef = useRef<Map<string, AblyTypes.Types.RealtimeChannelPromise>>(new Map());

  const initialize = useCallback(async () => {
    if (ablyRef.current) {
      return ablyRef.current;
    }

    const clientOptions: AblyTypes.Types.ClientOptions = {
      authUrl: options.authUrl || '/api/ably/token',
      authMethod: 'POST',
      authHeaders: {
        'Content-Type': 'application/json',
      },
      echoMessages: options.echoMessages ?? false,
      autoConnect: options.autoConnect ?? true,
    };

    if (options.clientId) {
      clientOptions.clientId = options.clientId;
    }

    const ably = new Ably.Realtime.Promise(clientOptions);
    
    ably.connection.on((stateChange: AblyTypes.Types.ConnectionStateChange) => {
      setStatus(stateChange.current);
      
      switch (stateChange.current) {
        case 'connected':
          console.log('Ably connected');
          break;
        case 'failed':
          console.error('Ably connection failed:', stateChange.reason);
          break;
        case 'disconnected':
          console.warn('Ably disconnected, attempting reconnection');
          break;
      }
    });

    ablyRef.current = ably;
    return ably;
  }, [options.authUrl, options.clientId, options.echoMessages, options.autoConnect]);

  const channel = useCallback((
    name: string,
    channelOptions?: AblyTypes.Types.ChannelOptions
  ): AblyTypes.Types.RealtimeChannelPromise => {
    if (!ablyRef.current) {
      throw new Error('Ably client not initialized');
    }
    
    const channelKey = `${name}:${JSON.stringify(channelOptions || {})}`;
    
    if (channelsRef.current.has(channelKey)) {
      return channelsRef.current.get(channelKey)!;
    }
    
    const channel = ablyRef.current.channels.get(name, channelOptions);
    channelsRef.current.set(channelKey, channel);
    
    return channel;
  }, []);

  const presence = useCallback((channelName: string): AblyTypes.Types.PresencePromise => {
    const ch = channel(channelName);
    return ch.presence;
  }, [channel]);

  const connect = useCallback(async () => {
    const ably = await initialize();
    if (ably.connection.state !== 'connected') {
      await ably.connection.connect();
    }
  }, [initialize]);

  const disconnect = useCallback(async () => {
    if (ablyRef.current) {
      await ablyRef.current.connection.close();
      channelsRef.current.clear();
    }
  }, []);

  // Auto-connect
  useEffect(() => {
    if (options.autoConnect !== false) {
      connect();
    }
    
    return () => {
      if (options.autoConnect !== false) {
        disconnect();
      }
    };
  }, [connect, disconnect, options.autoConnect]);

  return {
    client: ablyRef.current,
    channel,
    presence,
    status,
    connect,
    disconnect,
  };
};

// Example: Real-time notifications with Ably
export const useAblyNotifications = (userId: string) => {
  const { channel, status } = useAbly({
    clientId: userId,
    authUrl: '/api/ably/token',
    autoConnect: true,
  });
  
  const [notifications, setNotifications] = useState<any[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);
  
  useEffect(() => {
    if (status !== 'connected') return;
    
    const notificationsChannel = channel(`notifications:${userId}`);
    
    // Subscribe to notifications
    notificationsChannel.subscribe('notification', (message: any) => {
      const notification = {
        id: message.id,
        type: message.data.type,
        title: message.data.title,
        body: message.data.body,
        timestamp: message.timestamp,
        read: false,
        data: message.data.metadata,
      };
      
      setNotifications(prev => [notification, ...prev.slice(0, 49)]);
      setUnreadCount(prev => prev + 1);
      
      // Show browser notification if permitted
      if (Notification.permission === 'granted' && !document.hasFocus()) {
        new Notification(notification.title, {
          body: notification.body,
          icon: '/notification-icon.png',
          tag: notification.id,
        });
      }
    });
    
    // Presence for notification read status
    notificationsChannel.presence.subscribe('enter', (member: any) => {
      console.log(`User ${member.clientId} viewing notifications`);
    });
    
    return () => {
      notificationsChannel.unsubscribe();
    };
  }, [channel, status, userId]);
  
  const markAsRead = useCallback(async (notificationId: string) => {
    setNotifications(prev =>
      prev.map(n =>
        n.id === notificationId ? { ...n, read: true } : n
      )
    );
    setUnreadCount(prev => Math.max(0, prev - 1));
  }, []);
  
  return {
    notifications,
    unreadCount,
    markAsRead,
    markAllAsRead: () => {
      setNotifications(prev => prev.map(n => ({ ...n, read: true })));
      setUnreadCount(0);
    },
  };
};
Soketi (Self-hosted Pusher Alternative)
yaml
Copia
Scarica
# docker-compose.soketi.yml
version: '3.8'

services:
  soketi:
    image: quay.io/soketi/soketi:latest-16-alpine
    environment:
      SOKETI_DEFAULT_APP_ID: ${SOKETI_APP_ID}
      SOKETI_DEFAULT_APP_KEY: ${SOKETI_APP_KEY}
      SOKETI_DEFAULT_APP_SECRET: ${SOKETI_APP_SECRET}
      SOKETI_DEFAULT_HOST: 0.0.0.0
      SOKETI_DEFAULT_PORT: 6001
      SOKETI_DEFAULT_APP_ENABLE_CLIENT_MESS

(false);
      }
    };
    
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, []);
  
  // Position styles
  const positionStyles = {
    'top-left': { top: '1rem', left: '1rem' },
    'top-right': { top: '1rem', right: '1rem' },
    'bottom-left': { bottom: '1rem', left: '1rem' },
    'bottom-right': { bottom: '1rem', right: '1rem' },
  };
  
  const displayedNotifications = notifications.slice(0, maxNotifications);
  const hasUnread = unreadCount > 0;
  
  return (
    <div className="relative" ref={bellRef} style={positionStyles[position]}>
      {/* Bell Button */}
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="relative p-2 bg-white dark:bg-gray-800 rounded-full shadow-lg hover:shadow-xl transition-shadow focus:outline-none focus:ring-2 focus:ring-blue-500"
        aria-label={`Notifications ${hasUnread ? `(${unreadCount} unread)` : ''}`}
      >
        <Bell className="w-6 h-6 text-gray-600 dark:text-gray-300" />
        
        {showBadge && hasUnread && (
          <span className="absolute -top-1 -right-1 flex items-center justify-center">
            <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-red-400 opacity-75" />
            <span className="relative inline-flex items-center justify-center">
              {showCount && unreadCount > 9 ? (
                <

§ 3. ADVANCED EDGE CASES

§ 3.1 HANDLING DISCONNECTIONS AND RECONNECTIONS

In real-time applications, handling disconnections and reconnections is crucial for maintaining a seamless user experience. Here's an example of how to handle disconnections and reconnections using WebSocket:

typescript
import WebSocket, { WebSocketServer } from 'ws';

interface WSClient extends WebSocket {
  userId?: string;
  rooms: Set<string>;
  isAlive: boolean;
}

const wss = new WebSocketServer({ port: 8080 });
const clients = new Set<WSClient>();

// Connection handler
wss.on('connection', (ws: WSClient, req: IncomingMessage) => {
  // ...

  // Handle disconnections
  ws.on('close', () => {
    // Remove client from rooms
    for (const room of ws.rooms) {
      // ...
    }
    // Remove client from clients set
    clients.delete(ws);
  });

  // Handle reconnections
  ws.on('reconnect', () => {
    // Rejoin rooms
    for (const room of ws.rooms) {
      // ...
    }
    // Update client state
    ws.isAlive = true;
  });
});

§ 3.2 ERROR HANDLING AND LOGGING

Error handling and logging are essential for debugging and monitoring real-time applications. Here's an example of how to handle errors and log messages using WebSocket:

typescript
import WebSocket, { WebSocketServer } from 'ws';
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
  ],
});

// Connection handler
wss.on('connection', (ws: WSClient, req: IncomingMessage) => {
  // ...

  // Handle errors
  ws.on('error', (error) => {
    logger.error('Error occurred:', error);
  });

  // Handle messages
  ws.on('message', (raw) => {
    try {
      const data = JSON.parse(raw.toString());
      // ...
    } catch (error) {
      logger.error('Error parsing message:', error);
    }
  });
});

§ 4. PERFORMANCE CONSIDERATIONS

§ 4.1 OPTIMIZING WEBSOCKET CONNECTIONS

Optimizing WebSocket connections is crucial for improving the performance of real-time applications. Here are some best practices for optimizing WebSocket connections:

| Best Practice | Description |
| --- | --- |
| ✅ Use connection pooling | Reuse existing connections to reduce overhead |
| ✅ Use batching | Send multiple messages in a single packet to reduce overhead |
| ✅ Use compression | Compress messages to reduce payload size |
| ❌ Use unnecessary features | Avoid using features that are not necessary for your application |

§ 4.2 OPTIMIZING SERVER-SIDE PERFORMANCE

Optimizing server-side performance is crucial for improving the performance of real-time applications. Here are some best practices for optimizing server-side performance:

| Best Practice | Description |
| --- | --- |
| ✅ Use clustering | Use multiple worker processes to handle incoming connections |
| ✅ Use load balancing | Distribute incoming connections across multiple servers |
| ✅ Use caching | Cache frequently accessed data to reduce database queries |
| ❌ Use synchronous operations | Avoid using synchronous operations that can block the event loop |

§ 5. TESTING PATTERNS

§ 5.1 TESTING WEBSOCKET CONNECTIONS

Testing WebSocket connections is crucial for ensuring the reliability and performance of real-time applications. Here's an example of how to test WebSocket connections using Vitest:

typescript
import { describe, it, expect } from 'vitest';
import WebSocket from 'ws';

describe('WebSocket connection', () => {
  it('should establish a connection', async () => {
    const ws = new WebSocket('ws://localhost:8080');
    await new Promise((resolve) => {
      ws.on('open', () => {
        resolve();
      });
    });
    expect(ws.readyState).toBe(WebSocket.OPEN);
  });

  it('should send and receive messages', async () => {
    const ws = new WebSocket('ws://localhost:8080');
    await new Promise((resolve) => {
      ws.on('open', () => {
        resolve();
      });
    });
    ws.send('Hello, world!');
    await new Promise((resolve) => {
      ws.on('message', (message) => {
        expect(message.toString()).toBe('Hello, world!');
        resolve();
      });
    });
  });
});

§ 6. COMMON PITFALLS AND TROUBLESHOOTING

§ 6.1 HANDLING CONNECTION DROPS

Handling connection drops is crucial for maintaining a seamless user experience. Here are some common pitfalls and troubleshooting tips for handling connection drops:

| Pitfall | Description | Troubleshooting Tip |
| --- | --- | --- |
| Connection drops due to network issues | Connection drops due to network issues can cause data loss and corruption | Implement retry mechanisms and use connection pooling to reduce the impact of connection drops |
| Connection drops due to server overload | Connection drops due to server overload can cause data loss and corruption | Implement load balancing and use clustering to distribute incoming connections across multiple servers |
| Connection drops due to client-side issues | Connection drops due to client-side issues can cause data loss and corruption | Implement client-side retry mechanisms and use caching to reduce the impact of connection drops |

§ 7. MIGRATION AND UPGRADE PATTERNS

§ 7.1 MIGRATING FROM WEBSOCKET TO SOCKET.IO

Migrating from WebSocket to Socket.io can be a complex process. Here are some best practices for migrating from WebSocket to Socket.io:

| Best Practice | Description |
| --- | --- |
| ✅ Use a gradual migration approach | Migrate one feature at a time to reduce the impact of changes |
| ✅ Use a compatibility layer | Implement a compatibility layer to ensure backward compatibility with existing WebSocket clients |
| ✅ Use testing and validation | Thoroughly test and validate the migration to ensure that it works as expected |
| ❌ Use a big-bang approach | Avoid migrating everything at once, as this can cause significant downtime and disruption |

§ 7.2 UPGRADING FROM SOCKET.IO 3 TO SOCKET.IO 4

Upgrading from Socket.io 3 to Socket.io 4 can be a complex process. Here are some best practices for upgrading from Socket.io 3 to Socket.io 4:

| Best Practice | Description |
| --- | --- |
| ✅ Use a gradual upgrade approach | Upgrade one feature at a time to reduce the impact of changes |
| ✅ Use a compatibility layer | Implement a compatibility layer to ensure backward compatibility with existing Socket.io 3 clients |
| ✅ Use testing and validation | Thoroughly test and validate the upgrade to ensure that it works as expected |
| ❌ Use a big-bang approach | Avoid upgrading everything at once, as this can cause significant downtime and disruption |

---

## WEBSOCKET-ROOM-MANAGEMENT

### Panoramica
Sistema completo di gestione rooms WebSocket con join/leave, broadcast, presence tracking e message history per applicazioni collaborative.

### Implementazione Completa

```typescript
// lib/realtime/room-manager.ts

interface RoomMember {
  userId: string;
  socketId: string;
  displayName: string;
  avatar?: string;
  joinedAt: Date;
  lastActivity: Date;
  cursor?: { x: number; y: number };
  status: "active" | "idle" | "away";
}

interface RoomMessage {
  id: string;
  roomId: string;
  userId: string;
  type: "text" | "system" | "action";
  content: string;
  metadata?: Record<string, unknown>;
  timestamp: Date;
}

interface Room {
  id: string;
  name: string;
  type: "public" | "private" | "direct";
  members: Map<string, RoomMember>;
  messageHistory: RoomMessage[];
  maxMembers: number;
  maxHistorySize: number;
  createdAt: Date;
  metadata: Record<string, unknown>;
}

type RoomEvent =
  | { type: "member_joined"; member: RoomMember }
  | { type: "member_left"; userId: string; reason: "leave" | "disconnect" | "kick" }
  | { type: "message"; message: RoomMessage }
  | { type: "presence_update"; userId: string; status: RoomMember["status"] }
  | { type: "cursor_update"; userId: string; cursor: { x: number; y: number } }
  | { type: "room_closed"; reason: string };

type EventHandler = (roomId: string, event: RoomEvent) => void;

export class RoomManager {
  private rooms: Map<string, Room> = new Map();
  private userRooms: Map<string, Set<string>> = new Map();
  private eventHandlers: EventHandler[] = [];
  private idleTimeout = 5 * 60 * 1000; // 5 minutes
  private idleCheckInterval: ReturnType<typeof setInterval>;

  constructor() {
    this.idleCheckInterval = setInterval(() => this.checkIdleMembers(), 60_000);
  }

  onEvent(handler: EventHandler): () => void {
    this.eventHandlers.push(handler);
    return () => {
      this.eventHandlers = this.eventHandlers.filter((h) => h !== handler);
    };
  }

  private emit(roomId: string, event: RoomEvent): void {
    for (const handler of this.eventHandlers) {
      try {
        handler(roomId, event);
      } catch (err) {
        console.error("Event handler error:", err);
      }
    }
  }

  createRoom(options: {
    id: string;
    name: string;
    type: Room["type"];
    maxMembers?: number;
    maxHistorySize?: number;
    metadata?: Record<string, unknown>;
  }): Room {
    if (this.rooms.has(options.id)) {
      throw new Error(`Room ${options.id} already exists`);
    }

    const room: Room = {
      id: options.id,
      name: options.name,
      type: options.type,
      members: new Map(),
      messageHistory: [],
      maxMembers: options.maxMembers ?? 100,
      maxHistorySize: options.maxHistorySize ?? 500,
      createdAt: new Date(),
      metadata: options.metadata ?? {},
    };

    this.rooms.set(room.id, room);
    return room;
  }

  joinRoom(roomId: string, member: Omit<RoomMember, "joinedAt" | "lastActivity" | "status">): boolean {
    const room = this.rooms.get(roomId);
    if (!room) throw new Error(`Room ${roomId} not found`);
    if (room.members.size >= room.maxMembers) return false;
    if (room.members.has(member.userId)) return true;

    const fullMember: RoomMember = {
      ...member,
      joinedAt: new Date(),
      lastActivity: new Date(),
      status: "active",
    };

    room.members.set(member.userId, fullMember);

    if (!this.userRooms.has(member.userId)) {
      this.userRooms.set(member.userId, new Set());
    }
    this.userRooms.get(member.userId)!.add(roomId);

    this.addSystemMessage(roomId, `${member.displayName} joined the room`);
    this.emit(roomId, { type: "member_joined", member: fullMember });

    return true;
  }

  leaveRoom(roomId: string, userId: string, reason: "leave" | "disconnect" | "kick" = "leave"): void {
    const room = this.rooms.get(roomId);
    if (!room) return;

    const member = room.members.get(userId);
    if (!member) return;

    room.members.delete(userId);
    this.userRooms.get(userId)?.delete(roomId);

    const reasonText = reason === "kick" ? "was kicked from" : "left";
    this.addSystemMessage(roomId, `${member.displayName} ${reasonText} the room`);
    this.emit(roomId, { type: "member_left", userId, reason });

    if (room.members.size === 0 && room.type !== "public") {
      this.rooms.delete(roomId);
    }
  }

  disconnectUser(userId: string): void {
    const roomIds = this.userRooms.get(userId);
    if (!roomIds) return;
    for (const roomId of Array.from(roomIds)) {
      this.leaveRoom(roomId, userId, "disconnect");
    }
    this.userRooms.delete(userId);
  }

  sendMessage(roomId: string, userId: string, content: string, metadata?: Record<string, unknown>): RoomMessage | null {
    const room = this.rooms.get(roomId);
    if (!room) return null;
    const member = room.members.get(userId);
    if (!member) return null;

    member.lastActivity = new Date();
    member.status = "active";

    const message: RoomMessage = {
      id: crypto.randomUUID(),
      roomId,
      userId,
      type: "text",
      content,
      metadata,
      timestamp: new Date(),
    };

    room.messageHistory.push(message);
    if (room.messageHistory.length > room.maxHistorySize) {
      room.messageHistory = room.messageHistory.slice(-room.maxHistorySize);
    }

    this.emit(roomId, { type: "message", message });
    return message;
  }

  private addSystemMessage(roomId: string, content: string): void {
    const room = this.rooms.get(roomId);
    if (!room) return;
    const message: RoomMessage = {
      id: crypto.randomUUID(),
      roomId,
      userId: "system",
      type: "system",
      content,
      timestamp: new Date(),
    };
    room.messageHistory.push(message);
  }

  updatePresence(roomId: string, userId: string, status: RoomMember["status"]): void {
    const room = this.rooms.get(roomId);
    if (!room) return;
    const member = room.members.get(userId);
    if (!member) return;
    member.status = status;
    member.lastActivity = new Date();
    this.emit(roomId, { type: "presence_update", userId, status });
  }

  updateCursor(roomId: string, userId: string, cursor: { x: number; y: number }): void {
    const room = this.rooms.get(roomId);
    if (!room) return;
    const member = room.members.get(userId);
    if (!member) return;
    member.cursor = cursor;
    member.lastActivity = new Date();
    this.emit(roomId, { type: "cursor_update", userId, cursor });
  }

  getRoomMembers(roomId: string): RoomMember[] {
    const room = this.rooms.get(roomId);
    if (!room) return [];
    return Array.from(room.members.values());
  }

  getMessageHistory(roomId: string, limit = 50, before?: Date): RoomMessage[] {
    const room = this.rooms.get(roomId);
    if (!room) return [];
    let messages = room.messageHistory;
    if (before) {
      messages = messages.filter((m) => m.timestamp < before);
    }
    return messages.slice(-limit);
  }

  getUserRooms(userId: string): Room[] {
    const roomIds = this.userRooms.get(userId);
    if (!roomIds) return [];
    return Array.from(roomIds)
      .map((id) => this.rooms.get(id))
      .filter((r): r is Room => r !== undefined);
  }

  private checkIdleMembers(): void {
    const now = Date.now();
    for (const [roomId, room] of this.rooms) {
      for (const [userId, member] of room.members) {
        if (member.status === "active" && now - member.lastActivity.getTime() > this.idleTimeout) {
          member.status = "idle";
          this.emit(roomId, { type: "presence_update", userId, status: "idle" });
        }
      }
    }
  }

  destroy(): void {
    clearInterval(this.idleCheckInterval);
    for (const [roomId] of this.rooms) {
      this.emit(roomId, { type: "room_closed", reason: "server_shutdown" });
    }
    this.rooms.clear();
    this.userRooms.clear();
  }
}
```

### Errori Comuni da Evitare
- **Memory leak da rooms non chiuse**: Chiudere le rooms vuote automaticamente
- **Missing disconnect handling**: Gestire sempre la disconnessione dell'utente con cleanup di tutte le rooms
- **Unbounded message history**: Limitare sempre la dimensione della history
- **No idle detection**: Implementare sempre il check per utenti inattivi

---

## SERVER-SENT-EVENTS-PATTERN

### Panoramica
Pattern SSE per streaming unidirezionale con auto-reconnect, event filtering e backpressure management.

### Implementazione Completa

```typescript
// app/api/events/route.ts
import { NextRequest } from "next/server";

interface SSEClient {
  id: string;
  userId: string;
  controller: ReadableStreamDefaultController;
  channels: Set<string>;
  lastEventId: number;
  connectedAt: Date;
}

class SSEManager {
  private clients: Map<string, SSEClient> = new Map();
  private eventCounter = 0;
  private heartbeatInterval: ReturnType<typeof setInterval>;
  private eventBuffer: { id: number; channel: string; event: string; data: string; timestamp: Date }[] = [];
  private maxBufferSize = 1000;

  constructor() {
    this.heartbeatInterval = setInterval(() => this.sendHeartbeat(), 30_000);
  }

  addClient(userId: string, channels: string[], lastEventId?: number): ReadableStream {
    const clientId = crypto.randomUUID();

    const stream = new ReadableStream({
      start: (controller) => {
        const client: SSEClient = {
          id: clientId,
          userId,
          controller,
          channels: new Set(channels),
          lastEventId: lastEventId ?? this.eventCounter,
          connectedAt: new Date(),
        };

        this.clients.set(clientId, client);

        // Send missed events if reconnecting
        if (lastEventId !== undefined) {
          const missedEvents = this.eventBuffer.filter(
            (e) => e.id > lastEventId && channels.includes(e.channel)
          );
          for (const event of missedEvents) {
            this.sendToClient(client, event.event, event.data, event.id);
          }
        }

        // Send initial connection event
        this.sendToClient(client, "connected", JSON.stringify({
          clientId,
          channels,
          timestamp: new Date().toISOString(),
        }));
      },
      cancel: () => {
        this.clients.delete(clientId);
      },
    });

    return stream;
  }

  broadcast(channel: string, event: string, data: unknown): void {
    const eventId = ++this.eventCounter;
    const serialized = JSON.stringify(data);

    this.eventBuffer.push({
      id: eventId,
      channel,
      event,
      data: serialized,
      timestamp: new Date(),
    });

    if (this.eventBuffer.length > this.maxBufferSize) {
      this.eventBuffer = this.eventBuffer.slice(-this.maxBufferSize);
    }

    for (const client of this.clients.values()) {
      if (client.channels.has(channel)) {
        this.sendToClient(client, event, serialized, eventId);
      }
    }
  }

  sendToUser(userId: string, event: string, data: unknown): void {
    const serialized = JSON.stringify(data);
    const eventId = ++this.eventCounter;

    for (const client of this.clients.values()) {
      if (client.userId === userId) {
        this.sendToClient(client, event, serialized, eventId);
      }
    }
  }

  private sendToClient(client: SSEClient, event: string, data: string, id?: number): void {
    try {
      const encoder = new TextEncoder();
      let message = "";
      if (id !== undefined) message += `id: ${id}\n`;
      message += `event: ${event}\n`;
      message += `data: ${data}\n\n`;
      client.controller.enqueue(encoder.encode(message));
      if (id !== undefined) client.lastEventId = id;
    } catch {
      this.clients.delete(client.id);
    }
  }

  private sendHeartbeat(): void {
    const encoder = new TextEncoder();
    for (const [id, client] of this.clients) {
      try {
        client.controller.enqueue(encoder.encode(`: heartbeat ${Date.now()}\n\n`));
      } catch {
        this.clients.delete(id);
      }
    }
  }

  getClientCount(): number {
    return this.clients.size;
  }

  getChannelStats(): Record<string, number> {
    const stats: Record<string, number> = {};
    for (const client of this.clients.values()) {
      for (const channel of client.channels) {
        stats[channel] = (stats[channel] ?? 0) + 1;
      }
    }
    return stats;
  }

  destroy(): void {
    clearInterval(this.heartbeatInterval);
    for (const client of this.clients.values()) {
      try {
        client.controller.close();
      } catch {
        // ignore
      }
    }
    this.clients.clear();
  }
}

// Singleton
const sseManager = new SSEManager();

export async function GET(request: NextRequest) {
  const userId = request.headers.get("x-user-id");
  if (!userId) {
    return new Response("Unauthorized", { status: 401 });
  }

  const channels = request.nextUrl.searchParams.get("channels")?.split(",") ?? ["general"];
  const lastEventId = request.headers.get("last-event-id");

  const stream = sseManager.addClient(
    userId,
    channels,
    lastEventId ? parseInt(lastEventId, 10) : undefined
  );

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache, no-transform",
      Connection: "keep-alive",
      "X-Accel-Buffering": "no",
    },
  });
}

export { sseManager };
```

```typescript
// hooks/use-sse.ts
"use client";

import { useEffect, useRef, useCallback, useState } from "react";

interface UseSSEOptions {
  url: string;
  channels: string[];
  onMessage: (event: string, data: unknown) => void;
  onError?: (error: Event) => void;
  onOpen?: () => void;
  enabled?: boolean;
  maxRetries?: number;
}

interface SSEState {
  connected: boolean;
  retryCount: number;
  lastEventId: string | null;
}

export function useSSE({
  url,
  channels,
  onMessage,
  onError,
  onOpen,
  enabled = true,
  maxRetries = 10,
}: UseSSEOptions) {
  const [state, setState] = useState<SSEState>({
    connected: false,
    retryCount: 0,
    lastEventId: null,
  });
  const eventSourceRef = useRef<EventSource | null>(null);
  const retryTimeoutRef = useRef<ReturnType<typeof setTimeout>>();
  const onMessageRef = useRef(onMessage);
  onMessageRef.current = onMessage;

  const connect = useCallback(() => {
    if (eventSourceRef.current) {
      eventSourceRef.current.close();
    }

    const fullUrl = `${url}?channels=${channels.join(",")}`;
    const es = new EventSource(fullUrl);
    eventSourceRef.current = es;

    es.onopen = () => {
      setState((prev) => ({ ...prev, connected: true, retryCount: 0 }));
      onOpen?.();
    };

    es.onerror = (event) => {
      setState((prev) => {
        const newRetryCount = prev.retryCount + 1;
        if (newRetryCount <= maxRetries) {
          const delay = Math.min(1000 * Math.pow(2, newRetryCount), 30000);
          retryTimeoutRef.current = setTimeout(connect, delay);
        }
        return { ...prev, connected: false, retryCount: newRetryCount };
      });
      onError?.(event);
    };

    es.addEventListener("connected", (e) => {
      const data = JSON.parse((e as MessageEvent).data);
      setState((prev) => ({ ...prev, lastEventId: data.clientId }));
    });

    for (const channel of channels) {
      es.addEventListener(channel, (e) => {
        const me = e as MessageEvent;
        try {
          const data = JSON.parse(me.data);
          onMessageRef.current(channel, data);
        } catch {
          onMessageRef.current(channel, me.data);
        }
      });
    }
  }, [url, channels.join(","), maxRetries]);

  useEffect(() => {
    if (!enabled) return;
    connect();
    return () => {
      eventSourceRef.current?.close();
      if (retryTimeoutRef.current) clearTimeout(retryTimeoutRef.current);
    };
  }, [enabled, connect]);

  return state;
}
```

### Checklist di Verifica
- [ ] Il server invia heartbeat ogni 30 secondi per mantenere la connessione
- [ ] Il client implementa auto-reconnect con exponential backoff
- [ ] Gli eventi persi durante la disconnessione vengono recuperati tramite Last-Event-ID
- [ ] Il buffer eventi ha un limite massimo per evitare memory leak
- [ ] Le connessioni chiuse vengono rimosse dal manager



---

## OPTIMISTIC-UPDATES-PATTERN

### Panoramica
Pattern per aggiornamenti ottimistici con WebSocket che forniscono feedback istantaneo all'utente e gestiscono conflitti e rollback.

### Implementazione Completa

```typescript
// lib/realtime/optimistic-updates.ts
"use client";

import { useCallback, useRef, useState } from "react";

interface OptimisticUpdate<T> {
  id: string;
  timestamp: number;
  previousValue: T;
  optimisticValue: T;
  status: "pending" | "confirmed" | "rejected";
}

interface UseOptimisticMutationOptions<T> {
  onMutate: (newValue: T) => Promise<void>;
  onSuccess?: (confirmed: T) => void;
  onError?: (error: Error, previousValue: T) => void;
  timeout?: number;
}

export function useOptimisticMutation<T>(
  currentValue: T,
  setValue: (value: T) => void,
  options: UseOptimisticMutationOptions<T>
) {
  const [isPending, setIsPending] = useState(false);
  const pendingUpdates = useRef<Map<string, OptimisticUpdate<T>>>(new Map());
  const timeoutRefs = useRef<Map<string, ReturnType<typeof setTimeout>>>(new Map());

  const mutate = useCallback(
    async (optimisticValue: T) => {
      const updateId = crypto.randomUUID();
      const update: OptimisticUpdate<T> = {
        id: updateId,
        timestamp: Date.now(),
        previousValue: currentValue,
        optimisticValue,
        status: "pending",
      };

      pendingUpdates.current.set(updateId, update);
      setValue(optimisticValue);
      setIsPending(true);

      // Set timeout for automatic rollback
      const timeout = options.timeout ?? 10000;
      const timeoutId = setTimeout(() => {
        const pending = pendingUpdates.current.get(updateId);
        if (pending && pending.status === "pending") {
          pending.status = "rejected";
          setValue(pending.previousValue);
          pendingUpdates.current.delete(updateId);
          setIsPending(pendingUpdates.current.size > 0);
          options.onError?.(new Error("Mutation timed out"), pending.previousValue);
        }
      }, timeout);
      timeoutRefs.current.set(updateId, timeoutId);

      try {
        await options.onMutate(optimisticValue);
        const pending = pendingUpdates.current.get(updateId);
        if (pending) {
          pending.status = "confirmed";
          pendingUpdates.current.delete(updateId);
          options.onSuccess?.(optimisticValue);
        }
      } catch (error) {
        const pending = pendingUpdates.current.get(updateId);
        if (pending) {
          pending.status = "rejected";
          setValue(pending.previousValue);
          pendingUpdates.current.delete(updateId);
          options.onError?.(error as Error, pending.previousValue);
        }
      } finally {
        clearTimeout(timeoutRefs.current.get(updateId));
        timeoutRefs.current.delete(updateId);
        setIsPending(pendingUpdates.current.size > 0);
      }
    },
    [currentValue, setValue, options]
  );

  const rollbackAll = useCallback(() => {
    const updates = Array.from(pendingUpdates.current.values());
    if (updates.length > 0) {
      const oldest = updates[0];
      setValue(oldest.previousValue);
    }
    pendingUpdates.current.clear();
    for (const timeout of timeoutRefs.current.values()) {
      clearTimeout(timeout);
    }
    timeoutRefs.current.clear();
    setIsPending(false);
  }, [setValue]);

  return { mutate, isPending, rollbackAll, pendingCount: pendingUpdates.current.size };
}
```

### Checklist di Verifica
- [ ] Ogni mutazione ottimistica ha un timeout di sicurezza
- [ ] Il rollback ripristina il valore precedente corretto
- [ ] Le mutazioni pendenti sono tracciate con status
- [ ] Il componente mostra stato "pending" durante la conferma
