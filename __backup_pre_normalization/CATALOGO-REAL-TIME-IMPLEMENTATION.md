# CATALOGO-REAL-TIME-IMPLEMENTATION

ESPANSIONE CATALOGO REAL-TIME IMPLEMENTATION PER NEXT.JS 14
§ SOCKET.IO COMPLETE
Server Setup con Next.js
typescript
// lib/socket-server.ts
import { Server } from 'socket.io';
import { createServer } from 'http';
import { NextApiRequest, NextApiResponse } from 'next';
import type { Server as HTTPServer } from 'http';
import type { Socket as NetSocket } from 'net';

interface SocketServer extends HTTPServer {
  io?: Server;
}

interface SocketWithIO extends NetSocket {
  server: SocketServer;
}

interface NextApiResponseWithSocket extends NextApiResponse {
  socket: SocketWithIO;
}

const SocketHandler = (req: NextApiRequest, res: NextApiResponseWithSocket) => {
  if (!res.socket.server.io) {
    console.log('Inizializzazione Socket.IO server...');
    
    const httpServer: SocketServer = res.socket.server;
    const io = new Server(httpServer, {
      path: '/api/socketio',
      addTrailingSlash: false,
      cors: {
        origin: process.env.NEXT_PUBLIC_APP_URL || '*',
        methods: ['GET', 'POST'],
        credentials: true,
      },
      transports: ['websocket', 'polling'],
      pingTimeout: 60000,
      pingInterval: 25000,
      maxHttpBufferSize: 1e8,
    });

    // Setup event handlers
    io.on('connection', (socket) => {
      console.log(`Client connesso: ${socket.id}`);
      
      // Autenticazione
      const token = socket.handshake.auth.token;
      if (!verifyToken(token)) {
        socket.disconnect(true);
        return;
      }

      // Room management
      socket.on('join:room', (roomId: string) => {
        socket.join(roomId);
        io.to(roomId).emit('user:joined', {
          userId: socket.id,
          timestamp: new Date().toISOString(),
        });
      });

      socket.on('leave:room', (roomId: string) => {
        socket.leave(roomId);
        io.to(roomId).emit('user:left', {
          userId: socket.id,
          timestamp: new Date().toISOString(),
        });
      });

      // Gestione disconnessione
      socket.on('disconnect', (reason) => {
        console.log(`Client disconnesso: ${socket.id}, motivo: ${reason}`);
        
        // Notifica tutti i room dell'utente
        socket.rooms.forEach(room => {
          if (room !== socket.id) {
            io.to(room).emit('user:disconnected', {
              userId: socket.id,
              timestamp: new Date().toISOString(),
            });
          }
        });
      });

      // Error handling
      socket.on('error', (error) => {
        console.error(`Socket error per ${socket.id}:`, error);
      });
    });

    httpServer.io = io;
  }
  
  res.end();
};

function verifyToken(token: string): boolean {
  // Implementa la tua logica di verifica token
  return !!token;
}

export default SocketHandler;
Client Hook useSocket
typescript
// hooks/useSocket.ts
import { io, Socket } from 'socket.io-client';
import { useEffect, useRef, useState, useCallback } from 'react';

type SocketEventMap = {
  [event: string]: (...args: any[]) => void;
};

interface UseSocketOptions {
  url?: string;
  autoConnect?: boolean;
  reconnection?: boolean;
  reconnectionAttempts?: number;
  reconnectionDelay?: number;
  auth?: {
    token?: string;
    [key: string]: any;
  };
}

export function useSocket<T extends SocketEventMap = {}>(
  events?: T,
  options: UseSocketOptions = {}
) {
  const [isConnected, setIsConnected] = useState(false);
  const [reconnectAttempts, setReconnectAttempts] = useState(0);
  const [lastError, setLastError] = useState<Error | null>(null);
  
  const socketRef = useRef<Socket | null>(null);
  const eventHandlersRef = useRef<Map<string, (...args: any[]) => void>>(new Map());

  const {
    url = process.env.NEXT_PUBLIC_SOCKET_URL || '/',
    autoConnect = true,
    reconnection = true,
    reconnectionAttempts = 5,
    reconnectionDelay = 1000,
    auth,
  } = options;

  // Inizializzazione socket
  useEffect(() => {
    const socket = io(url, {
      autoConnect,
      reconnection,
      reconnectionAttempts,
      reconnectionDelay,
      auth,
      transports: ['websocket', 'polling'],
      path: '/api/socketio',
    });

    socketRef.current = socket;

    // Gestione eventi di connessione
    socket.on('connect', () => {
      console.log('Socket.IO connesso:', socket.id);
      setIsConnected(true);
      setReconnectAttempts(0);
      setLastError(null);
    });

    socket.on('disconnect', (reason) => {
      console.log('Socket.IO disconnesso:', reason);
      setIsConnected(false);
    });

    socket.on('connect_error', (error) => {
      console.error('Errore connessione Socket.IO:', error);
      setLastError(error);
    });

    socket.on('reconnect', (attemptNumber) => {
      console.log(`Socket.IO riconnesso dopo ${attemptNumber} tentativi`);
      setReconnectAttempts(attemptNumber);
    });

    socket.on('reconnect_attempt', (attemptNumber) => {
      console.log(`Tentativo riconnessione ${attemptNumber}`);
      setReconnectAttempts(attemptNumber);
    });

    socket.on('reconnect_error', (error) => {
      console.error('Errore riconnessione:', error);
      setLastError(error);
    });

    socket.on('reconnect_failed', () => {
      console.error('Riconnessione fallita dopo massimi tentativi');
    });

    // Registra gli eventi custom
    if (events) {
      Object.entries(events).forEach(([event, handler]) => {
        socket.on(event, handler);
        eventHandlersRef.current.set(event, handler);
      });
    }

    // Cleanup
    return () => {
      if (socket) {
        // Rimuovi tutti i listener
        eventHandlersRef.current.forEach((handler, event) => {
          socket.off(event, handler);
        });
        
        socket.disconnect();
        socketRef.current = null;
      }
    };
  }, [url, autoConnect, reconnection, reconnectionAttempts, reconnectionDelay, auth]);

  // Funzioni helper
  const emit = useCallback(<K extends keyof T>(
    event: K,
    ...args: Parameters<T[K]>
  ) => {
    if (socketRef.current?.connected) {
      socketRef.current.emit(event as string, ...args);
      return true;
    }
    console.warn('Socket non connesso, impossibile emettere evento:', event);
    return false;
  }, []);

  const on = useCallback(<K extends keyof T>(
    event: K,
    handler: T[K]
  ) => {
    if (socketRef.current) {
      socketRef.current.on(event as string, handler);
      eventHandlersRef.current.set(event as string, handler);
    }
  }, []);

  const off = useCallback(<K extends keyof T>(
    event: K,
    handler?: T[K]
  ) => {
    if (socketRef.current) {
      if (handler) {
        socketRef.current.off(event as string, handler);
        eventHandlersRef.current.delete(event as string);
      } else {
        socketRef.current.off(event as string);
        eventHandlersRef.current.delete(event as string);
      }
    }
  }, []);

  const joinRoom = useCallback((roomId: string) => {
    emit('join:room' as any, roomId);
  }, [emit]);

  const leaveRoom = useCallback((roomId: string) => {
    emit('leave:room' as any, roomId);
  }, [emit]);

  const disconnect = useCallback(() => {
    if (socketRef.current) {
      socketRef.current.disconnect();
    }
  }, []);

  const connect = useCallback(() => {
    if (socketRef.current && !socketRef.current.connected) {
      socketRef.current.connect();
    }
  }, []);

  return {
    socket: socketRef.current,
    isConnected,
    reconnectAttempts,
    lastError,
    emit,
    on,
    off,
    joinRoom,
    leaveRoom,
    disconnect,
    connect,
  };
}

// Hook per typing indicator
export function useTypingIndicator(
  roomId: string,
  userId: string,
  username: string
) {
  const { emit, on, off } = useSocket({
    'typing:start': (data: { userId: string; username: string }) => {},
    'typing:stop': (data: { userId: string }) => {},
  });

  const typingTimeoutRef = useRef<NodeJS.Timeout>();

  const startTyping = useCallback(() => {
    emit('typing:start' as any, { roomId, userId, username });
    
    // Rimanda lo stop dopo 3 secondi
    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    }
    
    typingTimeoutRef.current = setTimeout(() => {
      stopTyping();
    }, 3000);
  }, [roomId, userId, username, emit]);

  const stopTyping = useCallback(() => {
    emit('typing:stop' as any, { roomId, userId });
    
    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    }
  }, [roomId, userId, emit]);

  // Cleanup
  useEffect(() => {
    return () => {
      if (typingTimeoutRef.current) {
        clearTimeout(typingTimeoutRef.current);
      }
      stopTyping();
    };
  }, [stopTyping]);

  return { startTyping, stopTyping };
}
Rooms e Namespaces
typescript
// lib/socket-rooms.ts
import { Server, Socket } from 'socket.io';

export class RoomManager {
  private static instance: RoomManager;
  private rooms: Map<string, Set<string>> = new Map(); // roomId -> Set<userId>
  private userRooms: Map<string, Set<string>> = new Map(); // userId -> Set<roomId>

  private constructor() {}

  static getInstance(): RoomManager {
    if (!RoomManager.instance) {
      RoomManager.instance = new RoomManager();
    }
    return RoomManager.instance;
  }

  joinRoom(socket: Socket, roomId: string, userId: string) {
    // Aggiungi socket alla room
    socket.join(roomId);
    
    // Aggiorna stato room
    if (!this.rooms.has(roomId)) {
      this.rooms.set(roomId, new Set());
    }
    this.rooms.get(roomId)!.add(userId);
    
    // Aggiorna stanze dell'utente
    if (!this.userRooms.has(userId)) {
      this.userRooms.set(userId, new Set());
    }
    this.userRooms.get(userId)!.add(roomId);
    
    // Notifica agli altri nella room
    socket.to(roomId).emit('room:user-joined', {
      roomId,
      userId,
      timestamp: new Date().toISOString(),
      totalUsers: this.rooms.get(roomId)!.size,
    });
    
    console.log(`User ${userId} joined room ${roomId}`);
  }

  leaveRoom(socket: Socket, roomId: string, userId: string) {
    // Rimuovi socket dalla room
    socket.leave(roomId);
    
    // Aggiorna stato room
    if (this.rooms.has(roomId)) {
      this.rooms.get(roomId)!.delete(userId);
      
      // Rimuovi room se vuota
      if (this.rooms.get(roomId)!.size === 0) {
        this.rooms.delete(roomId);
      }
    }
    
    // Aggiorna stanze dell'utente
    if (this.userRooms.has(userId)) {
      this.userRooms.get(userId)!.delete(roomId);
      
      // Rimuovi utente se non in nessuna room
      if (this.userRooms.get(userId)!.size === 0) {
        this.userRooms.delete(userId);
      }
    }
    
    // Notifica agli altri nella room
    socket.to(roomId).emit('room:user-left', {
      roomId,
      userId,
      timestamp: new Date().toISOString(),
      totalUsers: this.rooms.get(roomId)?.size || 0,
    });
    
    console.log(`User ${userId} left room ${roomId}`);
  }

  getRoomUsers(roomId: string): string[] {
    return Array.from(this.rooms.get(roomId) || []);
  }

  getUserRooms(userId: string): string[] {
    return Array.from(this.userRooms.get(userId) || []);
  }

  broadcastToRoom(io: Server, roomId: string, event: string, data: any, excludeSocketId?: string) {
    if (excludeSocketId) {
      io.to(roomId).except(excludeSocketId).emit(event, data);
    } else {
      io.to(roomId).emit(event, data);
    }
  }
}

// Namespace setup
export function setupNamespaces(io: Server) {
  // Namespace principale
  const mainNamespace = io.of('/');
  
  // Namespace admin
  const adminNamespace = io.of('/admin');
  
  // Autenticazione namespace admin
  adminNamespace.use((socket, next) => {
    const token = socket.handshake.auth.token;
    const userRole = socket.handshake.auth.role;
    
    if (userRole === 'admin') {
      next();
    } else {
      next(new Error('Accesso non autorizzato al namespace admin'));
    }
  });
  
  adminNamespace.on('connection', (socket) => {
    console.log(`Admin connesso: ${socket.id}`);
    
    socket.on('admin:stats', async () => {
      const sockets = await mainNamespace.fetchSockets();
      const rooms = Array.from(io.of('/').adapter.rooms.keys());
      
      socket.emit('admin:stats-response', {
        totalConnections: sockets.length,
        activeRooms: rooms.filter(room => !sockets.some(s => s.id === room)).length,
        timestamp: new Date().toISOString(),
      });
    });
    
    socket.on('admin:broadcast', (message: string) => {
      mainNamespace.emit('admin:announcement', {
        message,
        timestamp: new Date().toISOString(),
      });
    });
  });
  
  // Namespace chat
  const chatNamespace = io.of('/chat');
  
  chatNamespace.on('connection', (socket) => {
    console.log(`Chat namespace - Client connesso: ${socket.id}`);
    
    socket.on('chat:join', (chatId: string) => {
      socket.join(`chat:${chatId}`);
    });
    
    socket.on('chat:message', (data: { chatId: string; message: string }) => {
      chatNamespace.to(`chat:${data.chatId}`).emit('chat:message', {
        ...data,
        userId: socket.id,
        timestamp: new Date().toISOString(),
      });
    });
  });
}
Authentication
typescript
// middleware/socket-auth.ts
import { Socket } from 'socket.io';
import jwt from 'jsonwebtoken';

interface SocketWithUser extends Socket {
  user?: {
    id: string;
    email: string;
    role: string;
  };
}

export function socketAuthMiddleware(socket: SocketWithUser, next: (err?: Error) => void) {
  try {
    // 1. Token da query string
    const tokenFromQuery = socket.handshake.query.token as string;
    
    // 2. Token da auth object
    const tokenFromAuth = socket.handshake.auth.token;
    
    // 3. Token da headers (se configurato)
    const tokenFromHeaders = socket.handshake.headers.authorization?.replace('Bearer ', '');
    
    const token = tokenFromAuth || tokenFromQuery || tokenFromHeaders;
    
    if (!token) {
      throw new Error('Token non fornito');
    }
    
    // Verifica JWT
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as {
      userId: string;
      email: string;
      role: string;
    };
    
    // Aggiungi user al socket
    socket.user = {
      id: decoded.userId,
      email: decoded.email,
      role: decoded.role,
    };
    
    next();
  } catch (error) {
    console.error('Socket auth error:', error);
    next(new Error('Autenticazione fallita'));
  }
}

// Middleware per room authorization
export function roomAuthMiddleware(requiredRole?: string) {
  return (socket: SocketWithUser, roomId: string, next: (err?: Error) => void) => {
    try {
      if (!socket.user) {
        throw new Error('Utente non autenticato');
      }
      
      // Verifica ruolo se specificato
      if (requiredRole && socket.user.role !== requiredRole) {
        throw new Error('Permessi insufficienti');
      }
      
      // Logica aggiuntiva per verificare accesso alla room
      // Es: controllo su database se l'utente può accedere alla room
      
      next();
    } catch (error) {
      console.error('Room auth error:', error);
      next(new Error('Autorizzazione room fallita'));
    }
  };
}
Reconnection Handling
typescript
// lib/socket-reconnection.ts
export class ReconnectionManager {
  private maxAttempts: number;
  private delay: number;
  private currentAttempt: number = 0;
  private timeoutId?: NodeJS.Timeout;
  private onReconnect?: () => void;
  private onMaxAttempts?: () => void;

  constructor(
    maxAttempts: number = 5,
    delay: number = 1000,
    onReconnect?: () => void,
    onMaxAttempts?: () => void
  ) {
    this.maxAttempts = maxAttempts;
    this.delay = delay;
    this.onReconnect = onReconnect;
    this.onMaxAttempts = onMaxAttempts;
  }

  attemptReconnect(connectFn: () => void) {
    if (this.currentAttempt >= this.maxAttempts) {
      console.error('Massimo numero di tentativi di riconnessione raggiunto');
      this.onMaxAttempts?.();
      return;
    }

    this.currentAttempt++;
    
    const delayWithJitter = this.delay * Math.pow(2, this.currentAttempt - 1) + 
      Math.random() * 1000; // Jitter casuale
    
    console.log(`Tentativo riconnessione ${this.currentAttempt} in ${delayWithJitter}ms`);
    
    this.timeoutId = setTimeout(() => {
      try {
        connectFn();
        this.onReconnect?.();
        this.reset();
      } catch (error) {
        console.error('Riconnessione fallita:', error);
        this.attemptReconnect(connectFn);
      }
    }, delayWithJitter);
  }

  reset() {
    this.currentAttempt = 0;
    if (this.timeoutId) {
      clearTimeout(this.timeoutId);
      this.timeoutId = undefined;
    }
  }

  getCurrentAttempt(): number {
    return this.currentAttempt;
  }

  getRemainingAttempts(): number {
    return this.maxAttempts - this.currentAttempt;
  }
}
Error Handling
typescript
// lib/socket-error-handler.ts
export class SocketErrorHandler {
  static handleConnectionError(error: Error, socket?: any): void {
    console.error('Errore connessione socket:', error);
    
    // Invia errore al client se il socket è disponibile
    if (socket && socket.connected) {
      socket.emit('error', {
        type: 'connection_error',
        message: 'Errore di connessione',
        code: 'CONN_ERR',
        timestamp: new Date().toISOString(),
      });
    }
    
    // Log analytics
    this.logError(error, 'connection');
  }

  static handleEventError(error: Error, socket: any, event: string): void {
    console.error(`Errore evento ${event}:`, error);
    
    // Invia errore specifico al client
    socket.emit('error', {
      type: 'event_error',
      event,
      message: error.message,
      code: 'EVENT_ERR',
      timestamp: new Date().toISOString(),
    });
    
    // Log analytics
    this.logError(error, `event:${event}`);
  }

  static handleRoomError(error: Error, socket: any, roomId: string): void {
    console.error(`Errore room ${roomId}:`, error);
    
    socket.emit('error', {
      type: 'room_error',
      roomId,
      message: error.message,
      code: 'ROOM_ERR',
      timestamp: new Date().toISOString(),
    });
    
    this.logError(error, `room:${roomId}`);
  }

  private static logError(error: Error, context: string): void {
    // Invia a servizio di logging
    const logEntry = {
      level: 'error',
      error: error.message,
      stack: error.stack,
      context,
      timestamp: new Date().toISOString(),
      userId: 'socket-user', // Sostituisci con ID utente reale
    };
    
    // Esempio: invio a console o servizio esterno
    console.log('[ERROR LOG]', logEntry);
    
    // Puoi inviare a:
    // 1. Sentry/Rollbar
    // 2. Elasticsearch
    // 3. Database dedicato
    // 4. External API
  }

  static createErrorResponse(
    error: Error,
    customCode?: string,
    metadata?: Record<string, any>
  ) {
    return {
      success: false,
      error: {
        message: error.message,
        code: customCode || 'INTERNAL_ERROR',
        timestamp: new Date().toISOString(),
        ...metadata,
      },
    };
  }
}
§ PUSHER/ABLY
Pusher Channels Setup
typescript
// lib/pusher-server.ts
import PusherServer from 'pusher';
import PusherClient from 'pusher-js';

// Configurazione server-side
export const pusherServer = new PusherServer({
  appId: process.env.PUSHER_APP_ID!,
  key: process.env.NEXT_PUBLIC_PUSHER_KEY!,
  secret: process.env.PUSHER_SECRET!,
  cluster: process.env.NEXT_PUBLIC_PUSHER_CLUSTER!,
  useTLS: true,
});

// Configurazione client-side
export const pusherClient = new PusherClient(
  process.env.NEXT_PUBLIC_PUSHER_KEY!,
  {
    cluster: process.env.NEXT_PUBLIC_PUSHER_CLUSTER!,
    forceTLS: true,
    authEndpoint: '/api/pusher/auth',
    auth: {
      headers: {
        'Content-Type': 'application/json',
      },
    },
    // Configurazione avanzata
    enabledTransports: ['ws', 'wss'], // Forza WebSocket
    disabledTransports: ['xhr_streaming', 'xhr_polling'],
    activityTimeout: 120000, // 2 minuti
    pongTimeout: 30000, // 30 secondi
  }
);

// Helper per pubblicare eventi
export async function triggerPusherEvent(
  channel: string,
  event: string,
  data: any,
  socketId?: string
) {
  try {
    await pusherServer.trigger(channel, event, data, socketId ? { socket_id: socketId } : undefined);
    return { success: true };
  } catch (error) {
    console.error('Errore trigger Pusher:', error);
    return { success: false, error };
  }
}

// API route per autenticazione
// pages/api/pusher/auth.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { getSession } from 'next-auth/react';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  try {
    const session = await getSession({ req });
    
    if (!session) {
      return res.status(401).json({ message: 'Unauthorized' });
    }

    const socketId = req.body.socket_id;
    const channel = req.body.channel_name;
    
    // Verifica permessi per il canale
    if (channel.startsWith('private-') || channel.startsWith('presence-')) {
      const userData = {
        user_id: session.user.id,
        user_info: {
          name: session.user.name,
          email: session.user.email,
          role: session.user.role,
        },
      };
      
      const authResponse = pusherServer.authorizeChannel(socketId, channel, userData);
      return res.status(200).json(authResponse);
    }
    
    return res.status(400).json({ message: 'Invalid channel type' });
  } catch (error) {
    console.error('Pusher auth error:', error);
    return res.status(500).json({ message: 'Internal server error' });
  }
}
Private Channels
typescript
// hooks/usePrivateChannel.ts
import { useEffect, useRef, useState } from 'react';
import { pusherClient } from '@/lib/pusher-server';
import { Channel } from 'pusher-js';

interface UsePrivateChannelOptions {
  channelName: string;
  events: Record<string, (data: any) => void>;
  onSubscriptionSucceeded?: (members?: any) => void;
  onSubscriptionError?: (error: any) => void;
}

export function usePrivateChannel(options: UsePrivateChannelOptions) {
  const { channelName, events, onSubscriptionSucceeded, onSubscriptionError } = options;
  
  const [isSubscribed, setIsSubscribed] = useState(false);
  const [channel, setChannel] = useState<Channel | null>(null);
  const channelRef = useRef<Channel | null>(null);

  useEffect(() => {
    if (!channelName.startsWith('private-')) {
      console.error('Il nome del canale deve iniziare con "private-"');
      return;
    }

    // Sottoscrivi al canale
    const privateChannel = pusherClient.subscribe(channelName);
    channelRef.current = privateChannel;
    setChannel(privateChannel);

    // Gestisci eventi di sottoscrizione
    privateChannel.bind('pusher:subscription_succeeded', (data: any) => {
      console.log(`Sottoscritto al canale privato: ${channelName}`);
      setIsSubscribed(true);
      onSubscriptionSucceeded?.(data);
    });

    privateChannel.bind('pusher:subscription_error', (error: any) => {
      console.error(`Errore sottoscrizione canale ${channelName}:`, error);
      setIsSubscribed(false);
      onSubscriptionError?.(error);
    });

    // Registra gli eventi custom
    Object.entries(events).forEach(([event, handler]) => {
      privateChannel.bind(event, handler);
    });

    // Cleanup
    return () => {
      if (channelRef.current) {
        // Rimuovi tutti i listener
        Object.entries(events).forEach(([event]) => {
          channelRef.current!.unbind(event);
        });
        
        // Rimuovi listener di sistema
        channelRef.current.unbind('pusher:subscription_succeeded');
        channelRef.current.unbind('pusher:subscription_error');
        
        // Disiscriviti
        pusherClient.unsubscribe(channelName);
        channelRef.current = null;
        setChannel(null);
        setIsSubscribed(false);
      }
    };
  }, [channelName]);

  const triggerClientEvent = (eventName: string, data: any) => {
    if (channelRef.current && isSubscribed) {
      channelRef.current.trigger(`client-${eventName}`, data);
      return true;
    }
    return false;
  };

  return {
    channel,
    isSubscribed,
    triggerClientEvent,
  };
}

// Esempio di utilizzo
export function useUserNotifications(userId: string) {
  const channelName = `private-user-${userId}`;
  
  const { isSubscribed, triggerClientEvent } = usePrivateChannel({
    channelName,
    events: {
      'new-notification': (data) => {
        console.log('Nuova notifica:', data);
        // Gestisci la notifica
      },
      'notification-read': (data) => {
        console.log('Notifica letta:', data);
      },
    },
    onSubscriptionSucceeded: () => {
      console.log('Canale notifiche pronto');
    },
  });

  const sendNotification = (notification: any) => {
    return triggerClientEvent('new-notification', notification);
  };

  return {
    isSubscribed,
    sendNotification,
  };
}
Presence Channels
typescript
// hooks/usePresenceChannel.ts
import { useEffect, useRef, useState } from 'react';
import { pusherClient } from '@/lib/pusher-server';
import { PresenceChannel } from 'pusher-js';

interface PresenceUser {
  id: string;
  name: string;
  info?: Record<string, any>;
}

interface UsePresenceChannelOptions {
  channelName: string;
  user: PresenceUser;
  events?: Record<string, (data: any) => void>;
  onUsersChanged?: (users: PresenceUser[]) => void;
}

export function usePresenceChannel(options: UsePresenceChannelOptions) {
  const { channelName, user, events = {}, onUsersChanged } = options;
  
  const [isSubscribed, setIsSubscribed] = useState(false);
  const [members, setMembers] = useState<Map<string, PresenceUser>>(new Map());
  const [onlineCount, setOnlineCount] = useState(0);
  const channelRef = useRef<PresenceChannel | null>(null);

  useEffect(() => {
    if (!channelName.startsWith('presence-')) {
      console.error('Il nome del canale deve iniziare con "presence-"');
      return;
    }

    // Sottoscrivi al canale presence
    const presenceChannel = pusherClient.subscribe(channelName) as PresenceChannel;
    channelRef.current = presenceChannel;

    // Gestisci eventi presence
    presenceChannel.bind('pusher:subscription_succeeded', (membersData: any) => {
      console.log(`Sottoscritto al canale presence: ${channelName}`);
      setIsSubscribed(true);
      
      // Inizializza membri
      const membersMap = new Map<string, PresenceUser>();
      membersData.each((member: any) => {
        membersMap.set(member.id, member.info);
      });
      
      setMembers(membersMap);
      setOnlineCount(membersData.count);
      onUsersChanged?.(Array.from(membersMap.values()));
    });

    presenceChannel.bind('pusher:member_added', (member: any) => {
      console.log(`Utente aggiunto: ${member.id}`);
      
      setMembers(prev => {
        const updated = new Map(prev);
        updated.set(member.id, member.info);
        return updated;
      });
      
      setOnlineCount(prev => prev + 1);
      onUsersChanged?.(Array.from(members.entries()).map(([id, info]) => ({ id, ...info })));
    });

    presenceChannel.bind('pusher:member_removed', (member: any) => {
      console.log(`Utente rimosso: ${member.id}`);
      
      setMembers(prev => {
        const updated = new Map(prev);
        updated.delete(member.id);
        return updated;
      });
      
      setOnlineCount(prev => prev - 1);
      onUsersChanged?.(Array.from(members.entries()).map(([id, info]) => ({ id, ...info })));
    });

    // Registra eventi custom
    Object.entries(events).forEach(([event, handler]) => {
      presenceChannel.bind(event, handler);
    });

    // Cleanup
    return () => {
      if (channelRef.current) {
        // Rimuovi tutti i listener
        Object.entries(events).forEach(([event]) => {
          channelRef.current!.unbind(event);
        });
        
        channelRef.current.unbind('pusher:subscription_succeeded');
        channelRef.current.unbind('pusher:member_added');
        channelRef.current.unbind('pusher:member_removed');
        
        pusherClient.unsubscribe(channelName);
        channelRef.current = null;
        setIsSubscribed(false);
        setMembers(new Map());
        setOnlineCount(0);
      }
    };
  }, [channelName, user.id]);

  const getMember = (userId: string): PresenceUser | undefined => {
    return members.get(userId);
  };

  const isUserOnline = (userId: string): boolean => {
    return members.has(userId);
  };

  const triggerClientEvent = (eventName: string, data: any) => {
    if (channelRef.current && isSubscribed) {
      channelRef.current.trigger(`client-${eventName}`, data);
      return true;
    }
    return false;
  };

  return {
    channel: channelRef.current,
    isSubscribed,
    members: Array.from(members.values()),
    onlineCount,
    getMember,
    isUserOnline,
    triggerClientEvent,
  };
}

// Esempio: Online presence per chat room
export function useChatRoomPresence(roomId: string, currentUser: PresenceUser) {
  const channelName = `presence-chat-${roomId}`;
  
  const { members, onlineCount, isUserOnline } = usePresenceChannel({
    channelName,
    user: currentUser,
    events: {
      'user-typing': (data: { userId: string; username: string }) => {
        // Gestisci typing indicator
      },
      'message-sent': (data: any) => {
        // Aggiorna chat
      },
    },
    onUsersChanged: (users) => {
      console.log(`Utenti online nella chat ${roomId}:`, users.length);
    },
  });

  return {
    onlineUsers: members,
    onlineCount,
    isUserOnline,
  };
}
Client Events
typescript
// lib/pusher-client-events.ts
export class PusherClientEventManager {
  private channel: any;
  private allowedEvents: Set<string>;
  private rateLimit: Map<string, { count: number; timestamp: number }>;

  constructor(channel: any) {
    this.channel = channel;
    this.allowedEvents = new Set();
    this.rateLimit = new Map();
    
    // Configura rate limiting: max 10 eventi per secondo per tipo
    setInterval(() => this.cleanupRateLimit(), 60000); // Cleanup ogni minuto
  }

  allowEvent(eventName: string) {
    this.allowedEvents.add(eventName);
  }

  disallowEvent(eventName: string) {
    this.allowedEvents.delete(eventName);
  }

  trigger(eventName: string, data: any, options?: {
    skipRateLimit?: boolean;
    maxPerMinute?: number;
  }) {
    // Verifica se l'evento è permesso
    if (!this.allowedEvents.has(eventName) && !eventName.startsWith('client-')) {
      console.error(`Evento ${eventName} non permesso per trigger client`);
      return false;
    }

    // Rate limiting
    if (!options?.skipRateLimit) {
      const now = Date.now();
      const key = `${eventName}:${Math.floor(now / 60000)}`; // Chiave per minuto
      
      if (!this.rateLimit.has(key)) {
        this.rateLimit.set(key, { count: 0, timestamp: now });
      }
      
      const limit = this.rateLimit.get(key)!;
      const maxPerMinute = options?.maxPerMinute || 60;
      
      if (limit.count >= maxPerMinute) {
        console.warn(`Rate limit raggiunto per evento ${eventName}`);
        return false;
      }
      
      limit.count++;
    }

    // Trigger dell'evento
    try {
      this.channel.trigger(`client-${eventName}`, data);
      return true;
    } catch (error) {
      console.error(`Errore trigger evento ${eventName}:`, error);
      return false;
    }
  }

  private cleanupRateLimit() {
    const oneMinuteAgo = Date.now()

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-27-REAL-TIME-IMPLEMENTATION
Prompt ID: 27 / 48
Parte: 2
Exported: 2026-02-06T11:48:09.773Z
Characters: 1946
════════════════════════════════════════════════════════════

extApiResponse) {
  if (req.method !== 'GET') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  const { channel } = req.query;
  
  if (!channel || Array.isArray(channel)) {
    return res.status(400).json({ message: 'Channel required' });
  }

  // Imposta headers per SSE
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache, no-transform',
    'Connection': 'keep-alive',
    'X-Accel-Buffering': 'no',
    'Content-Encoding': 'none',
  });

  // Invia commento di keep-alive iniziale
  res.write(': SSE connection established\n\n');
  
  // Salva la connessione
  const connectionId = `${channel}-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  activeConnections.set(connectionId, res);

  console.log(`SSE connection established: ${connectionId} on channel ${channel}`);

  // Gestione eventi per questo canale
  const eventHandler = (data: any) => {
    try {
      const eventData = typeof data === 'string' ? data : JSON.stringify(data);
      res.write(`data: ${eventData}\n\n`);
    } catch (error) {
      console.error('Error sending SSE event:', error);
    }
  };

  // Sottoscrizione agli eventi del canale
  sseEmitter.on(`sse:${channel}`, eventHandler);

  // Invia heartbeat ogni 30 secondi
  const heartbeatInterval = setInterval(() => {
    try {
      res.write(': heartbeat\n\n');
    } catch (error) {
      clearInterval(heartbeatInterval);
    }
  }, 30000);

  // Gestione chiusura connessione
  req.on('close', () => {
    console.log(`SSE connection closed: ${connectionId}`);
    clearInterval(heartbeatInterval);
    activeConnections.delete(connectionId);
    sseEmitter.off(`sse:${channel}`, eventHandler);
    res.end();
  });

  req.on('error', (error) => {
    console.error(`SSE connection error: ${connectionId}`, error);
    clearInterval(heartbeatInterval);
    activeConnections.delete(connectionId);
    sseEmitter.off(`sse