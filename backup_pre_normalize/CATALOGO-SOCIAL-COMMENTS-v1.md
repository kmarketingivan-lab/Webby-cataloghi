# CATALOGO-SOCIAL-COMMENTS-v1

> **Dominio**: Social
> **Stack**: Next.js, React, TypeScript, Prisma, tRPC, Zod
> **Versione**: 1.0
> **Data**: 2026-02-04

---

## INDICE

| # | Sezione | Path |
|---|---------|------|
| 1 | [prisma/schema-social-comments.prisma](#1-prisma-schema-social-comments-prisma) | `prisma/schema-social-comments.prisma` |
| 2 | [src/server/services/comment-service.ts](#2-src-server-services-comment-service-ts) | `src/server/services/comment-service.ts` |
| 3 | [src/server/services/notification-service.ts](#3-src-server-services-notification-service-ts) | `src/server/services/notification-service.ts` |
| 4 | [src/server/trpc/routers/comments.ts](#4-src-server-trpc-routers-comments-ts) | `src/server/trpc/routers/comments.ts` |
| 5 | [src/server/trpc/routers/notifications.ts](#5-src-server-trpc-routers-notifications-ts) | `src/server/trpc/routers/notifications.ts` |
| 6 | [src/lib/validations/comments.ts](#6-src-lib-validations-comments-ts) | `src/lib/validations/comments.ts` |
| 7 | [src/lib/validations/notifications.ts](#7-src-lib-validations-notifications-ts) | `src/lib/validations/notifications.ts` |
| 8 | [src/hooks/use-comments.ts](#8-src-hooks-use-comments-ts) | `src/hooks/use-comments.ts` |
| 9 | [src/hooks/use-notifications.ts](#9-src-hooks-use-notifications-ts) | `src/hooks/use-notifications.ts` |
| 10 | [src/components/social/comments/comment-list.tsx](#10-src-components-social-comments-comment-list-tsx) | `src/components/social/comments/comment-list.tsx` |
| 11 | [src/components/social/comments/comment-item.tsx](#11-src-components-social-comments-comment-item-tsx) | `src/components/social/comments/comment-item.tsx` |
| 12 | [src/components/social/comments/comment-form.tsx](#12-src-components-social-comments-comment-form-tsx) | `src/components/social/comments/comment-form.tsx` |
| 13 | [src/components/social/notifications/notification-list.tsx](#13-src-components-social-notifications-notification-list-tsx) | `src/components/social/notifications/notification-list.tsx` |
| 14 | [src/components/social/notifications/notification-item.tsx](#14-src-components-social-notifications-notification-item-tsx) | `src/components/social/notifications/notification-item.tsx` |
| 15 | [src/components/social/notifications/notification-bell.tsx](#15-src-components-social-notifications-notification-bell-tsx) | `src/components/social/notifications/notification-bell.tsx` |
| 16 | [src/app/(social)/notifications/page.tsx](#16-src-app-(social)-notifications-page-tsx) | `src/app/(social)/notifications/page.tsx` |
| 17 | [tests/comments.test.ts](#17-tests-comments-test-ts) | `tests/comments.test.ts` |
| 18 | [tests/notifications.test.ts](#18-tests-notifications-test-ts) | `tests/notifications.test.ts` |

---

Catalogo Completo per Sistema di Commenti Social e Notifiche Next.js 14
Tabella Riepilogativa
Sezione	Nome File	Descrizione	Dimensione Stimata
1	prisma/schema-social-comments.prisma	Schema Prisma completo per commenti e notifiche	5 KB
2	src/server/services/comment-service.ts	Service layer per operazioni sui commenti	8 KB
3	src/server/services/notification-service.ts	Service layer per operazioni sulle notifiche	7 KB
4	src/server/trpc/routers/comments.ts	Router tRPC per commenti	6 KB
5	src/server/trpc/routers/notifications.ts	Router tRPC per notifiche	5 KB
6	src/lib/validations/comments.ts	Schema Zod per validazione commenti	3 KB
7	src/lib/validations/notifications.ts	Schema Zod per validazione notifiche	2 KB
8	src/hooks/use-comments.ts	Custom hook React per commenti	10 KB
9	src/hooks/use-notifications.ts	Custom hook React per notifiche	8 KB
10	src/components/social/comments/comment-list.tsx	Componente lista commenti con infinite scroll	9 KB
11	src/components/social/comments/comment-item.tsx	Componente singolo commento	12 KB
12	src/components/social/comments/comment-form.tsx	Form per creazione/editing commenti	8 KB
13	src/components/social/notifications/notification-list.tsx	Componente lista notifiche	9 KB
14	src/components/social/notifications/notification-item.tsx	Componente singola notifica	7 KB
15	src/components/social/notifications/notification-bell.tsx	Bell icon con badge notifiche	6 KB
16	src/app/(social)/notifications/page.tsx	Pagina notifiche	6 KB
17	tests/comments.test.ts	Test per funzionalità commenti	10 KB
18	tests/notifications.test.ts	Test per funzionalità notifiche	8 KB
1. prisma/schema-social-comments.prisma
prisma
Copia
Scarica
// File: prisma/schema-social-comments.prisma

// Questo schema estende lo schema utente esistente
// Assicurati di aver già un modello User definito

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Modello Post (presupposto esista già)
model Post {
  id               String    @id @default(cuid())
  title            String
  content          String?
  published        Boolean   @default(false)
  authorId         String
  createdAt        DateTime  @default(now())
  updatedAt        DateTime  @updatedAt
  
  // Relazioni
  author           User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  comments         Comment[]
  
  @@index([authorId])
  @@index([createdAt])
}

// Modello Comment
model Comment {
  id               String    @id @default(cuid())
  content          String    @db.Text
  authorId         String
  postId           String
  parentId         String?   // Per commenti nidificati
  createdAt        DateTime  @default(now())
  updatedAt        DateTime  @updatedAt
  deletedAt        DateTime? // Soft delete
  
  // Relazioni
  author           User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  post             Post      @relation(fields: [postId], references: [id], onDelete: Cascade)
  parent           Comment?  @relation("CommentReplies", fields: [parentId], references: [id], onDelete: Cascade)
  replies          Comment[] @relation("CommentReplies")
  likes            CommentLike[]
  notifications    Notification[]
  
  // Vincoli
  @@index([postId])
  @@index([authorId])
  @@index([parentId])
  @@index([createdAt])
  @@index([deletedAt])
  
  // Vincolo per evitare cicli
  @@map("comments")
}

// Modello per like ai commenti
model CommentLike {
  id        String   @id @default(cuid())
  userId    String
  commentId String
  createdAt DateTime @default(now())
  
  // Relazioni
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  comment   Comment  @relation(fields: [commentId], references: [id], onDelete: Cascade)
  
  // Vincolo di unicità: un utente può mettere like a un commento solo una volta
  @@unique([userId, commentId])
  @@index([commentId])
  @@index([userId])
  @@index([createdAt])
  
  @@map("comment_likes")
}

// Modello per notifiche
model Notification {
  id          String             @id @default(cuid())
  type        NotificationType
  recipientId String
  actorId     String            // Utente che ha causato la notifica
  targetType  String            // "post", "comment", "user"
  targetId    String
  read        Boolean           @default(false)
  createdAt   DateTime          @default(now())
  
  // Relazioni
  recipient   User              @relation("NotificationRecipient", fields: [recipientId], references: [id], onDelete: Cascade)
  actor       User              @relation("NotificationActor", fields: [actorId], references: [id], onDelete: Cascade)
  
  @@index([recipientId])
  @@index([actorId])
  @@index([targetType, targetId])
  @@index([read])
  @@index([createdAt])
  @@unique([recipientId, actorId, targetType, targetId, type])
  
  @@map("notifications")
}

// Enum per tipi di notifica
enum NotificationType {
  COMMENT_REPLY  // Risposta a un commento
  COMMENT_LIKE   // Like a un commento
  MENTION        // Mention in un commento
  FOLLOW         // Nuovo follower
  POST_COMMENT   // Commento a un post
}

// Modello User (estensione dello schema esistente)
model User {
  id                    String         @id @default(cuid())
  email                 String         @unique
  name                  String?
  image                 String?
  emailVerified         DateTime?
  
  // Relazioni esistenti...
  
  // Nuove relazioni per il sistema di commenti
  comments              Comment[]
  commentLikes          CommentLike[]
  notificationsSent     Notification[] @relation("NotificationActor")
  notificationsReceived Notification[] @relation("NotificationRecipient")
  
  // ... altri campi esistenti
  
  @@map("users")
}

// Migrazioni da eseguire:
// 1. npx prisma migrate dev --name add-social-comments-system
// 2. npx prisma generate
2. src/server/services/comment-service.ts
typescript
Copia
Scarica
// File: src/server/services/comment-service.ts

import { PrismaClient, Prisma } from '@prisma/client';
import { TRPCError } from '@trpc/server';

const prisma = new PrismaClient();

export interface CreateCommentInput {
  content: string;
  authorId: string;
  postId: string;
  parentId?: string | null;
}

export interface UpdateCommentInput {
  id: string;
  content: string;
  authorId: string;
}

export interface GetCommentsByPostInput {
  postId: string;
  cursor?: string;
  limit?: number;
  includeReplies?: boolean;
}

export interface CommentWithRelations {
  id: string;
  content: string;
  authorId: string;
  postId: string;
  parentId: string | null;
  createdAt: Date;
  updatedAt: Date;
  deletedAt: Date | null;
  author: {
    id: string;
    name: string | null;
    image: string | null;
    email: string;
  };
  _count: {
    likes: number;
    replies: number;
  };
  likes: Array<{
    userId: string;
  }>;
  replies?: CommentWithRelations[];
}

export class CommentService {
  /**
   * Crea un nuovo commento
   */
  static async createComment(input: CreateCommentInput): Promise<CommentWithRelations> {
    return await prisma.$transaction(async (tx) => {
      // Verifica che il post esista
      const post = await tx.post.findUnique({
        where: { id: input.postId },
        select: { id: true, authorId: true }
      });

      if (!post) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'Post non trovato'
        });
      }

      // Se parentId è specificato, verifica che il commento padre esista
      if (input.parentId) {
        const parentComment = await tx.comment.findUnique({
          where: { 
            id: input.parentId,
            deletedAt: null 
          },
          select: { 
            id: true, 
            postId: true,
            authorId: true 
          }
        });

        if (!parentComment) {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: 'Commento padre non trovato'
          });
        }

        // Verifica che il commento padre appartenga allo stesso post
        if (parentComment.postId !== input.postId) {
          throw new TRPCError({
            code: 'BAD_REQUEST',
            message: 'Il commento padre non appartiene a questo post'
          });
        }
      }

      // Crea il commento
      const comment = await tx.comment.create({
        data: {
          content: input.content,
          authorId: input.authorId,
          postId: input.postId,
          parentId: input.parentId || null,
        },
        include: {
          author: {
            select: {
              id: true,
              name: true,
              image: true,
              email: true,
            }
          },
          _count: {
            select: {
              likes: true,
              replies: true,
            }
          },
          likes: {
            where: {
              userId: input.authorId
            },
            select: {
              userId: true
            }
          }
        }
      });

      // Se il commento è una risposta (ha un parentId), crea una notifica per l'autore del commento padre
      if (input.parentId) {
        const parentComment = await tx.comment.findUnique({
          where: { id: input.parentId },
          select: { authorId: true }
        });

        if (parentComment && parentComment.authorId !== input.authorId) {
          await tx.notification.create({
            data: {
              type: 'COMMENT_REPLY',
              recipientId: parentComment.authorId,
              actorId: input.authorId,
              targetType: 'comment',
              targetId: comment.id,
            }
          });
        }
      } else {
        // Se è un commento radice, notifica l'autore del post (se diverso dall'autore del commento)
        if (post.authorId !== input.authorId) {
          await tx.notification.create({
            data: {
              type: 'POST_COMMENT',
              recipientId: post.authorId,
              actorId: input.authorId,
              targetType: 'post',
              targetId: input.postId,
            }
          });
        }
      }

      return comment as CommentWithRelations;
    });
  }

  /**
   * Aggiorna un commento esistente
   */
  static async updateComment(input: UpdateCommentInput): Promise<CommentWithRelations> {
    const comment = await prisma.comment.findUnique({
      where: { 
        id: input.id,
        deletedAt: null 
      }
    });

    if (!comment) {
      throw new TRPCError({
        code: 'NOT_FOUND',
        message: 'Commento non trovato'
      });
    }

    // Verifica che l'utente sia l'autore del commento
    if (comment.authorId !== input.authorId) {
      throw new TRPCError({
        code: 'FORBIDDEN',
        message: 'Non sei autorizzato a modificare questo commento'
      });
    }

    const updatedComment = await prisma.comment.update({
      where: { id: input.id },
      data: {
        content: input.content,
        updatedAt: new Date(),
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            image: true,
            email: true,
          }
        },
        _count: {
          select: {
            likes: true,
            replies: true,
          }
        },
        likes: {
          where: {
            userId: input.authorId
          },
          select: {
            userId: true
          }
        }
      }
    });

    return updatedComment as CommentWithRelations;
  }

  /**
   * Soft delete di un commento
   */
  static async deleteComment(commentId: string, userId: string): Promise<void> {
    const comment = await prisma.comment.findUnique({
      where: { id: commentId }
    });

    if (!comment) {
      throw new TRPCError({
        code: 'NOT_FOUND',
        message: 'Commento non trovato'
      });
    }

    // Verifica che l'utente sia l'autore del commento o un admin
    if (comment.authorId !== userId) {
      // Qui potresti aggiungere un controllo per i ruoli admin
      throw new TRPCError({
        code: 'FORBIDDEN',
        message: 'Non sei autorizzato a eliminare questo commento'
      });
    }

    await prisma.$transaction(async (tx) => {
      // Soft delete del commento
      await tx.comment.update({
        where: { id: commentId },
        data: { deletedAt: new Date() }
      });

      // Se il commento ha risposte, le soft delete anche quelle
      await tx.comment.updateMany({
        where: { parentId: commentId },
        data: { deletedAt: new Date() }
      });
    });
  }

  /**
   * Recupera i commenti di un post con paginazione cursor-based
   */
  static async getCommentsByPost(
    input: GetCommentsByPostInput
  ): Promise<{
    comments: CommentWithRelations[];
    nextCursor: string | null;
    total: number;
  }> {
    const limit = input.limit || 20;
    const cursor = input.cursor ? { id: input.cursor } : undefined;
    
    // Conta il numero totale di commenti radice (non eliminati)
    const total = await prisma.comment.count({
      where: {
        postId: input.postId,
        parentId: null,
        deletedAt: null,
      }
    });

    // Recupera i commenti radice
    const rootComments = await prisma.comment.findMany({
      where: {
        postId: input.postId,
        parentId: null,
        deletedAt: null,
      },
      take: limit + 1, // Prendi uno in più per determinare se c'è una prossima pagina
      cursor,
      orderBy: {
        createdAt: 'desc'
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            image: true,
            email: true,
          }
        },
        _count: {
          select: {
            likes: true,
            replies: true,
          }
        },
        likes: {
          select: {
            userId: true
          }
        }
      }
    });

    let nextCursor: string | null = null;
    let comments = rootComments;

    // Se abbiamo preso più commenti del limite, c'è una prossima pagina
    if (rootComments.length > limit) {
      const nextItem = rootComments.pop();
      nextCursor = nextItem?.id || null;
    }

    // Se richiesto, carica le risposte per ogni commento
    if (input.includeReplies) {
      comments = await Promise.all(
        comments.map(async (comment) => {
          const replies = await this.getCommentReplies(comment.id, 3); // Max 3 livelli di nesting
          return {
            ...comment,
            replies
          } as CommentWithRelations;
        })
      );
    }

    return {
      comments: comments as CommentWithRelations[],
      nextCursor,
      total
    };
  }

  /**
   * Recupera le risposte a un commento (max 3 livelli di nesting)
   */
  static async getCommentReplies(
    parentId: string,
    maxDepth: number = 3,
    currentDepth: number = 1
  ): Promise<CommentWithRelations[]> {
    if (currentDepth > maxDepth) {
      return [];
    }

    const replies = await prisma.comment.findMany({
      where: {
        parentId,
        deletedAt: null,
      },
      orderBy: {
        createdAt: 'asc'
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            image: true,
            email: true,
          }
        },
        _count: {
          select: {
            likes: true,
            replies: true,
          }
        },
        likes: {
          select: {
            userId: true
          }
        }
      }
    });

    // Recursivamente carica le risposte delle risposte
    const repliesWithNested = await Promise.all(
      replies.map(async (reply) => {
        const nestedReplies = await this.getCommentReplies(
          reply.id,
          maxDepth,
          currentDepth + 1
        );
        return {
          ...reply,
          replies: nestedReplies
        } as CommentWithRelations;
      })
    );

    return repliesWithNested;
  }

  /**
   * Aggiunge un like a un commento
   */
  static async likeComment(commentId: string, userId: string): Promise<void> {
    await prisma.$transaction(async (tx) => {
      const comment = await tx.comment.findUnique({
        where: { 
          id: commentId,
          deletedAt: null 
        },
        select: { 
          id: true, 
          authorId: true 
        }
      });

      if (!comment) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'Commento non trovato'
        });
      }

      try {
        // Prova a creare il like
        await tx.commentLike.create({
          data: {
            userId,
            commentId,
          }
        });

        // Crea una notifica se l'autore del commento è diverso dall'utente che mette like
        if (comment.authorId !== userId) {
          await tx.notification.create({
            data: {
              type: 'COMMENT_LIKE',
              recipientId: comment.authorId,
              actorId: userId,
              targetType: 'comment',
              targetId: commentId,
            }
          });
        }
      } catch (error) {
        if (error instanceof Prisma.PrismaClientKnownRequestError) {
          if (error.code === 'P2002') {
            // Unique constraint failed - l'utente ha già messo like a questo commento
            throw new TRPCError({
              code: 'CONFLICT',
              message: 'Hai già messo like a questo commento'
            });
          }
        }
        throw error;
      }
    });
  }

  /**
   * Rimuove un like da un commento
   */
  static async unlikeComment(commentId: string, userId: string): Promise<void> {
    const comment = await prisma.comment.findUnique({
      where: { 
        id: commentId,
        deletedAt: null 
      }
    });

    if (!comment) {
      throw new TRPCError({
        code: 'NOT_FOUND',
        message: 'Commento non trovato'
      });
    }

    const deletedLike = await prisma.commentLike.deleteMany({
      where: {
        userId,
        commentId
      }
    });

    if (deletedLike.count === 0) {
      throw new TRPCError({
        code: 'NOT_FOUND',
        message: 'Like non trovato'
      });
    }
  }

  /**
   * Recupera un commento specifico con tutte le relazioni
   */
  static async getCommentById(commentId: string): Promise<CommentWithRelations | null> {
    const comment = await prisma.comment.findUnique({
      where: { 
        id: commentId,
        deletedAt: null 
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            image: true,
            email: true,
          }
        },
        _count: {
          select: {
            likes: true,
            replies: true,
          }
        },
        likes: {
          select: {
            userId: true
          }
        }
      }
    });

    return comment as CommentWithRelations | null;
  }

  /**
   * Controlla se un utente ha messo like a un commento
   */
  static async hasUserLikedComment(commentId: string, userId: string): Promise<boolean> {
    const like = await prisma.commentLike.findUnique({
      where: {
        userId_commentId: {
          userId,
          commentId
        }
      }
    });

    return !!like;
  }
}

export default CommentService;
3. src/server/services/notification-service.ts
typescript
Copia
Scarica
// File: src/server/services/notification-service.ts

import { PrismaClient, NotificationType } from '@prisma/client';
import { TRPCError } from '@trpc/server';

const prisma = new PrismaClient();

export interface CreateNotificationInput {
  type: NotificationType;
  recipientId: string;
  actorId: string;
  targetType: string;
  targetId: string;
}

export interface CreateBatchNotificationInput {
  type: NotificationType;
  recipientIds: string[];
  actorId: string;
  targetType: string;
  targetId: string;
}

export interface GetNotificationsInput {
  recipientId: string;
  cursor?: string;
  limit?: number;
  read?: boolean;
}

export interface NotificationWithRelations {
  id: string;
  type: NotificationType;
  recipientId: string;
  actorId: string;
  targetType: string;
  targetId: string;
  read: boolean;
  createdAt: Date;
  actor: {
    id: string;
    name: string | null;
    image: string | null;
    email: string;
  };
}

export class NotificationService {
  /**
   * Crea una nuova notifica
   */
  static async createNotification(input: CreateNotificationInput): Promise<NotificationWithRelations> {
    try {
      const notification = await prisma.notification.create({
        data: {
          type: input.type,
          recipientId: input.recipientId,
          actorId: input.actorId,
          targetType: input.targetType,
          targetId: input.targetId,
        },
        include: {
          actor: {
            select: {
              id: true,
              name: true,
              image: true,
              email: true,
            }
          }
        }
      });

      return notification as NotificationWithRelations;
    } catch (error: any) {
      // Se la notifica esiste già (unique constraint), la restituiamo
      if (error.code === 'P2002') {
        const existingNotification = await prisma.notification.findFirst({
          where: {
            recipientId: input.recipientId,
            actorId: input.actorId,
            targetType: input.targetType,
            targetId: input.targetId,
            type: input.type,
          },
          include: {
            actor: {
              select: {
                id: true,
                name: true,
                image: true,
                email: true,
              }
            }
          }
        });

        if (existingNotification) {
          return existingNotification as NotificationWithRelations;
        }
      }
      throw error;
    }
  }

  /**
   * Crea notifiche in batch per più utenti
   */
  static async createBatchNotification(input: CreateBatchNotificationInput): Promise<number> {
    // Rimuovi duplicati e l'attore stesso dalla lista dei destinatari
    const uniqueRecipientIds = [...new Set(input.recipientIds)].filter(
      id => id !== input.actorId
    );

    if (uniqueRecipientIds.length === 0) {
      return 0;
    }

    // Crea notifiche usando createMany con skipDuplicates
    const result = await prisma.notification.createMany({
      data: uniqueRecipientIds.map(recipientId => ({
        type: input.type,
        recipientId,
        actorId: input.actorId,
        targetType: input.targetType,
        targetId: input.targetId,
      })),
      skipDuplicates: true,
    });

    return result.count;
  }

  /**
   * Segna una notifica come letta
   */
  static async markAsRead(notificationId: string, recipientId: string): Promise<void> {
    const notification = await prisma.notification.findUnique({
      where: { id: notificationId }
    });

    if (!notification) {
      throw new TRPCError({
        code: 'NOT_FOUND',
        message: 'Notifica non trovata'
      });
    }

    if (notification.recipientId !== recipientId) {
      throw new TRPCError({
        code: 'FORBIDDEN',
        message: 'Non sei autorizzato a modificare questa notifica'
      });
    }

    await prisma.notification.update({
      where: { id: notificationId },
      data: { read: true }
    });
  }

  /**
   * Segna tutte le notifiche di un utente come lette
   */
  static async markAllAsRead(recipientId: string): Promise<number> {
    const result = await prisma.notification.updateMany({
      where: {
        recipientId,
        read: false
      },
      data: {
        read: true
      }
    });

    return result.count;
  }

  /**
   * Recupera il conteggio delle notifiche non lette
   */
  static async getUnreadCount(recipientId: string): Promise<number> {
    return await prisma.notification.count({
      where: {
        recipientId,
        read: false
      }
    });
  }

  /**
   * Recupera le notifiche di un utente con paginazione
   */
  static async getNotifications(
    input: GetNotificationsInput
  ): Promise<{
    notifications: NotificationWithRelations[];
    nextCursor: string | null;
    total: number;
    unreadCount: number;
  }> {
    const limit = input.limit || 20;
    const cursor = input.cursor ? { id: input.cursor } : undefined;
    
    // Conta il numero totale di notifiche
    const total = await prisma.notification.count({
      where: {
        recipientId: input.recipientId,
        ...(input.read !== undefined ? { read: input.read } : {})
      }
    });

    // Conta le notifiche non lette
    const unreadCount = await prisma.notification.count({
      where: {
        recipientId: input.recipientId,
        read: false
      }
    });

    // Recupera le notifiche
    const notifications = await prisma.notification.findMany({
      where: {
        recipientId: input.recipientId,
        ...(input.read !== undefined ? { read: input.read } : {})
      },
      take: limit + 1, // Prendi uno in più per determinare se c'è una prossima pagina
      cursor,
      orderBy: {
        createdAt: 'desc'
      },
      include: {
        actor: {
          select: {
            id: true,
            name: true,
            image: true,
            email: true,
          }
        }
      }
    });

    let nextCursor: string | null = null;

    // Se abbiamo preso più notifiche del limite, c'è una prossima pagina
    if (notifications.length > limit) {
      const nextItem = notifications.pop();
      nextCursor = nextItem?.id || null;
    }

    return {
      notifications: notifications as NotificationWithRelations[],
      nextCursor,
      total,
      unreadCount
    };
  }

  /**
   * Elimina notifiche vecchie di più di 30 giorni
   */
  static async deleteOldNotifications(days: number = 30): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    const result = await prisma.notification.deleteMany({
      where: {
        createdAt: {
          lt: cutoffDate
        },
        read: true // Elimina solo notifiche lette
      }
    });

    return result.count;
  }

  /**
   * Recupera notifiche raggruppate per data
   */
  static async getNotificationsGroupedByDate(recipientId: string, limit: number = 50): Promise<{
    today: NotificationWithRelations[];
    yesterday: NotificationWithRelations[];
    thisWeek: NotificationWithRelations[];
    older: NotificationWithRelations[];
  }> {
    const now = new Date();
    const startOfToday = new Date(now.getFullYear(), now.getMonth(), now.getDate());
    const startOfYesterday = new Date(startOfToday);
    startOfYesterday.setDate(startOfYesterday.getDate() - 1);
    const startOfWeek = new Date(startOfToday);
    startOfWeek.setDate(startOfWeek.getDate() - 7);

    const allNotifications = await prisma.notification.findMany({
      where: {
        recipientId
      },
      take: limit,
      orderBy: {
        createdAt: 'desc'
      },
      include: {
        actor: {
          select: {
            id: true,
            name: true,
            image: true,
            email: true,
          }
        }
      }
    });

    const grouped = {
      today: [] as NotificationWithRelations[],
      yesterday: [] as NotificationWithRelations[],
      thisWeek: [] as NotificationWithRelations[],
      older: [] as NotificationWithRelations[]
    };

    allNotifications.forEach(notification => {
      const notificationDate = notification.createdAt;

      if (notificationDate >= startOfToday) {
        grouped.today.push(notification as NotificationWithRelations);
      } else if (notificationDate >= startOfYesterday) {
        grouped.yesterday.push(notification as NotificationWithRelations);
      } else if (notificationDate >= startOfWeek) {
        grouped.thisWeek.push(notification as NotificationWithRelations);
      } else {
        grouped.older.push(notification as NotificationWithRelations);
      }
    });

    return grouped;
  }

  /**
   * Recupera le ultime notifiche per la dropdown
   */
  static async getRecentNotifications(recipientId: string, limit: number = 5): Promise<NotificationWithRelations[]> {
    const notifications = await prisma.notification.findMany({
      where: {
        recipientId
      },
      take: limit,
      orderBy: {
        createdAt: 'desc'
      },
      include: {
        actor: {
          select: {
            id: true,
            name: true,
            image: true,
            email: true,
          }
        }
      }
    });

    return notifications as NotificationWithRelations[];
  }

  /**
   * Elimina una notifica specifica
   */
  static async deleteNotification(notificationId: string, recipientId: string): Promise<void> {
    const notification = await prisma.notification.findUnique({
      where: { id: notificationId }
    });

    if (!notification) {
      throw new TRPCError({
        code: 'NOT_FOUND',
        message: 'Notifica non trovata'
      });
    }

    if (notification.recipientId !== recipientId) {
      throw new TRPCError({
        code: 'FORBIDDEN',
        message: 'Non sei autorizzato a eliminare questa notifica'
      });
    }

    await prisma.notification.delete({
      where: { id: notificationId }
    });
  }
}

export default NotificationService;
4. src/server/trpc/routers/comments.ts
typescript
Copia
Scarica
// File: src/server/trpc/routers/comments.ts

import { z } from 'zod';
import { TRPCError } from '@trpc/server';
import { createTRPCRouter, protectedProcedure, publicProcedure } from '../../trpc';
import { createCommentSchema, updateCommentSchema, commentQuerySchema } from '@/lib/validations/comments';
import CommentService from '@/server/services/comment-service';
import { RateLimiterMemory } from 'rate-limiter-flexible';

// Rate limiter per la creazione di commenti: massimo 10 commenti al minuto
const commentRateLimiter = new RateLimiterMemory({
  points: 10,
  duration: 60, // 60 secondi
});

export const commentsRouter = createTRPCRouter({
  /**
   * Recupera i commenti di un post con paginazione
   */
  list: publicProcedure
    .input(commentQuerySchema.extend({
      postId: z.string(),
      includeReplies: z.boolean().default(true),
    }))
    .query(async ({ input }) => {
      try {
        const result = await CommentService.getCommentsByPost({
          postId: input.postId,
          cursor: input.cursor,
          limit: input.limit,
          includeReplies: input.includeReplies,
        });

        return {
          success: true,
          data: {
            comments: result.comments,
            nextCursor: result.nextCursor,
            total: result.total,
          }
        };
      } catch (error) {
        console.error('Error fetching comments:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Errore durante il recupero dei commenti'
        });
      }
    }),

  /**
   * Crea un nuovo commento
   */
  create: protectedProcedure
    .input(createCommentSchema)
    .mutation(async ({ input, ctx }) => {
      try {
        // Rate limiting
        try {
          await commentRateLimiter.consume(ctx.session.user.id);
        } catch (rateLimiterRes) {
          throw new TRPCError({
            code: 'TOO_MANY_REQUESTS',
            message: 'Troppi commenti creati di recente. Attendi un momento.'
          });
        }

        // Controlla se l'utente è autenticato
        if (!ctx.session || !ctx.session.user) {
          throw new TRPCError({
            code: 'UNAUTHORIZED',
            message: 'Devi essere autenticato per creare commenti'
          });
        }

        const comment = await CommentService.createComment({
          content: input.content,
          authorId: ctx.session.user.id,
          postId: input.postId,
          parentId: input.parentId,
        });

        return {
          success: true,
          data: comment,
          message: 'Commento creato con successo'
        };
      } catch (error) {
        if (error instanceof TRPCError) {
          throw error;
        }
        console.error('Error creating comment:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Errore durante la creazione del commento'
        });
      }
    }),

  /**
   * Aggiorna un commento esistente
   */
  update: protectedProcedure
    .input(updateCommentSchema)
    .mutation(async ({ input, ctx }) => {
      try {
        if (!ctx.session || !ctx.session.user) {
          throw new TRPCError({
            code: 'UNAUTHORIZED',
            message: 'Devi essere autenticato per aggiornare commenti'
          });
        }

        const comment = await CommentService.updateComment({
          id: input.id,
          content: input.content,
          authorId: ctx.session.user.id,
        });

        return {
          success: true,
          data: comment,
          message: 'Commento aggiornato con successo'
        };
      } catch (error) {
        if (error instanceof TRPCError) {
          throw error;
        }
        console.error('Error updating comment:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Errore durante l\'aggiornamento del commento'
        });
      }
    }),

  /**
   * Soft delete di un commento
   */
  delete: protectedProcedure
    .input(z.object({
      id: z