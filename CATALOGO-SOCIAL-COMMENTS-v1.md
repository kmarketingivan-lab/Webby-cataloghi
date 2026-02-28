# CATALOGO-SOCIAL-COMMENTS-v1

> **Dominio**: Social
> **Stack**: Next.js, React, TypeScript, Prisma, tRPC, Zod
> **Versione**: 1.0
> **Data**: 2026-02-04

---

§ INDICE

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


---

## COMMENT-SERVICE-COMPLETE

### Panoramica
Service layer completo per operazioni sui commenti: CRUD, threading, likes/reactions, moderazione contenuti, menzioni utente e paginazione infinita.

### Implementazione Completa

```typescript
// src/server/services/comment-service.ts
import { prisma } from "@/lib/prisma";
import { Prisma } from "@prisma/client";

// ============================================================
// TYPES
// ============================================================
interface CreateCommentInput {
  content: string;
  postId: string;
  authorId: string;
  parentId?: string;
}

interface UpdateCommentInput {
  content: string;
  commentId: string;
  authorId: string;
}

interface ListCommentsInput {
  postId: string;
  cursor?: string;
  limit?: number;
  sortBy?: "newest" | "oldest" | "popular";
  parentId?: string | null;
}

interface CommentWithReplies {
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
    name: string;
    image: string | null;
  };
  _count: {
    replies: number;
    likes: number;
  };
  isLiked: boolean;
  replies: CommentWithReplies[];
}

// ============================================================
// COMMENT SERVICE
// ============================================================
export class CommentService {
  // ----------------------------------------------------------
  // CREATE COMMENT
  // ----------------------------------------------------------
  async create(input: CreateCommentInput): Promise<CommentWithReplies> {
    // Validate parent exists and belongs to same post
    if (input.parentId) {
      const parent = await prisma.comment.findUnique({
        where: { id: input.parentId },
        select: { postId: true, parentId: true },
      });

      if (!parent) {
        throw new Error("Parent comment not found");
      }
      if (parent.postId !== input.postId) {
        throw new Error("Parent comment belongs to a different post");
      }
      // Max nesting depth: 3 levels
      if (parent.parentId) {
        const grandparent = await prisma.comment.findUnique({
          where: { id: parent.parentId },
          select: { parentId: true },
        });
        if (grandparent?.parentId) {
          throw new Error("Maximum nesting depth reached (3 levels)");
        }
      }
    }

    // Extract mentions from content
    const mentions = this.extractMentions(input.content);

    // Create comment
    const comment = await prisma.comment.create({
      data: {
        content: input.content,
        postId: input.postId,
        authorId: input.authorId,
        parentId: input.parentId ?? null,
      },
      include: {
        author: { select: { id: true, name: true, image: true } },
        _count: { select: { replies: true, likes: true } },
      },
    });

    // Create notifications for mentions
    if (mentions.length > 0) {
      const mentionedUsers = await prisma.user.findMany({
        where: { name: { in: mentions } },
        select: { id: true },
      });

      await prisma.notification.createMany({
        data: mentionedUsers
          .filter((u) => u.id !== input.authorId)
          .map((u) => ({
            type: "MENTION" as const,
            recipientId: u.id,
            actorId: input.authorId,
            targetType: "comment",
            targetId: comment.id,
          })),
      });
    }

    // Notify post author (if not self-commenting)
    const post = await prisma.post.findUnique({
      where: { id: input.postId },
      select: { authorId: true },
    });

    if (post && post.authorId !== input.authorId) {
      await prisma.notification.create({
        data: {
          type: "COMMENT" as const,
          recipientId: post.authorId,
          actorId: input.authorId,
          targetType: "comment",
          targetId: comment.id,
        },
      });
    }

    // Notify parent comment author (for replies)
    if (input.parentId) {
      const parentComment = await prisma.comment.findUnique({
        where: { id: input.parentId },
        select: { authorId: true },
      });

      if (parentComment && parentComment.authorId !== input.authorId) {
        await prisma.notification.create({
          data: {
            type: "REPLY" as const,
            recipientId: parentComment.authorId,
            actorId: input.authorId,
            targetType: "comment",
            targetId: comment.id,
          },
        });
      }
    }

    return {
      ...comment,
      isLiked: false,
      replies: [],
    };
  }

  // ----------------------------------------------------------
  // UPDATE COMMENT
  // ----------------------------------------------------------
  async update(input: UpdateCommentInput): Promise<CommentWithReplies> {
    const existing = await prisma.comment.findUnique({
      where: { id: input.commentId },
      select: { authorId: true, deletedAt: true },
    });

    if (!existing) throw new Error("Comment not found");
    if (existing.deletedAt) throw new Error("Cannot edit a deleted comment");
    if (existing.authorId !== input.authorId) {
      throw new Error("Not authorized to edit this comment");
    }

    const comment = await prisma.comment.update({
      where: { id: input.commentId },
      data: {
        content: input.content,
        updatedAt: new Date(),
      },
      include: {
        author: { select: { id: true, name: true, image: true } },
        _count: { select: { replies: true, likes: true } },
      },
    });

    return { ...comment, isLiked: false, replies: [] };
  }

  // ----------------------------------------------------------
  // SOFT DELETE
  // ----------------------------------------------------------
  async delete(commentId: string, userId: string, isAdmin = false): Promise<void> {
    const comment = await prisma.comment.findUnique({
      where: { id: commentId },
      select: { authorId: true },
    });

    if (!comment) throw new Error("Comment not found");
    if (!isAdmin && comment.authorId !== userId) {
      throw new Error("Not authorized to delete this comment");
    }

    await prisma.comment.update({
      where: { id: commentId },
      data: {
        deletedAt: new Date(),
        content: "[This comment has been deleted]",
      },
    });
  }

  // ----------------------------------------------------------
  // LIST COMMENTS WITH CURSOR PAGINATION
  // ----------------------------------------------------------
  async list(input: ListCommentsInput) {
    const limit = input.limit ?? 20;

    const orderBy: Prisma.CommentOrderByWithRelationInput =
      input.sortBy === "oldest"
        ? { createdAt: "asc" }
        : input.sortBy === "popular"
          ? { likes: { _count: "desc" } }
          : { createdAt: "desc" };

    const comments = await prisma.comment.findMany({
      where: {
        postId: input.postId,
        parentId: input.parentId ?? null,
        deletedAt: null,
      },
      orderBy,
      take: limit + 1,
      ...(input.cursor && {
        cursor: { id: input.cursor },
        skip: 1,
      }),
      include: {
        author: { select: { id: true, name: true, image: true } },
        _count: { select: { replies: true, likes: true } },
        replies: {
          where: { deletedAt: null },
          take: 3,
          orderBy: { createdAt: "asc" },
          include: {
            author: { select: { id: true, name: true, image: true } },
            _count: { select: { replies: true, likes: true } },
          },
        },
      },
    });

    const hasMore = comments.length > limit;
    const data = hasMore ? comments.slice(0, limit) : comments;

    return {
      comments: data,
      nextCursor: hasMore ? data[data.length - 1].id : null,
      hasMore,
    };
  }

  // ----------------------------------------------------------
  // TOGGLE LIKE
  // ----------------------------------------------------------
  async toggleLike(commentId: string, userId: string): Promise<{ liked: boolean; count: number }> {
    const existing = await prisma.commentLike.findUnique({
      where: { userId_commentId: { userId, commentId } },
    });

    if (existing) {
      await prisma.commentLike.delete({
        where: { id: existing.id },
      });
    } else {
      await prisma.commentLike.create({
        data: { userId, commentId },
      });

      // Notify comment author
      const comment = await prisma.comment.findUnique({
        where: { id: commentId },
        select: { authorId: true },
      });

      if (comment && comment.authorId !== userId) {
        await prisma.notification.create({
          data: {
            type: "LIKE" as const,
            recipientId: comment.authorId,
            actorId: userId,
            targetType: "comment",
            targetId: commentId,
          },
        });
      }
    }

    const count = await prisma.commentLike.count({
      where: { commentId },
    });

    return { liked: !existing, count };
  }

  // ----------------------------------------------------------
  // EXTRACT @MENTIONS
  // ----------------------------------------------------------
  private extractMentions(content: string): string[] {
    const mentionRegex = /@(\w+)/g;
    const matches: string[] = [];
    let match;
    while ((match = mentionRegex.exec(content)) !== null) {
      matches.push(match[1]);
    }
    return [...new Set(matches)];
  }

  // ----------------------------------------------------------
  // GET COMMENT COUNT FOR POST
  // ----------------------------------------------------------
  async getCount(postId: string): Promise<number> {
    return prisma.comment.count({
      where: { postId, deletedAt: null },
    });
  }
}

export const commentService = new CommentService();
```

### Varianti e Configurazioni

```typescript
// src/server/services/moderation-service.ts
import { prisma } from "@/lib/prisma";

// ============================================================
// CONTENT MODERATION SERVICE
// ============================================================
interface ModerationResult {
  approved: boolean;
  reason?: string;
  flags: string[];
  confidence: number;
}

export class ModerationService {
  private bannedWords: Set<string>;
  private spamPatterns: RegExp[];

  constructor() {
    this.bannedWords = new Set([
      // In production, load from database/config
    ]);

    this.spamPatterns = [
      /\b(buy now|click here|free money|act now)\b/i,
      /(https?:\/\/[^\s]+){3,}/i, // 3+ URLs
      /(.)\1{10,}/i, // Repeated chars (aaaaaaaaaaa)
      /[A-Z\s]{50,}/i, // Excessive caps
    ];
  }

  async moderate(content: string, authorId: string): Promise<ModerationResult> {
    const flags: string[] = [];
    let confidence = 1.0;

    // Check banned words
    const words = content.toLowerCase().split(/\s+/);
    for (const word of words) {
      if (this.bannedWords.has(word)) {
        flags.push(`banned_word:${word}`);
        confidence = 0;
      }
    }

    // Check spam patterns
    for (const pattern of this.spamPatterns) {
      if (pattern.test(content)) {
        flags.push("spam_pattern");
        confidence = Math.min(confidence, 0.3);
      }
    }

    // Check user reputation
    const user = await prisma.user.findUnique({
      where: { id: authorId },
      select: { createdAt: true },
    });

    if (user) {
      const accountAge = Date.now() - user.createdAt.getTime();
      const hourMs = 60 * 60 * 1000;

      if (accountAge < hourMs) {
        flags.push("new_account");
        confidence = Math.min(confidence, 0.5);
      }
    }

    // Check rate: too many comments in short time
    const recentComments = await prisma.comment.count({
      where: {
        authorId,
        createdAt: { gte: new Date(Date.now() - 5 * 60 * 1000) },
      },
    });

    if (recentComments > 10) {
      flags.push("rate_limit_exceeded");
      confidence = Math.min(confidence, 0.2);
    }

    // Check for duplicate content
    const duplicate = await prisma.comment.findFirst({
      where: {
        authorId,
        content,
        createdAt: { gte: new Date(Date.now() - 60 * 60 * 1000) },
      },
    });

    if (duplicate) {
      flags.push("duplicate_content");
      confidence = Math.min(confidence, 0.1);
    }

    return {
      approved: confidence > 0.4,
      reason: flags.length > 0 ? `Flagged: ${flags.join(", ")}` : undefined,
      flags,
      confidence,
    };
  }

  async reportComment(
    commentId: string,
    reporterId: string,
    reason: string
  ): Promise<void> {
    await prisma.commentReport.create({
      data: {
        commentId,
        reporterId,
        reason,
        status: "PENDING",
      },
    });

    // Auto-hide if 3+ reports
    const reportCount = await prisma.commentReport.count({
      where: { commentId, status: { not: "DISMISSED" } },
    });

    if (reportCount >= 3) {
      await prisma.comment.update({
        where: { id: commentId },
        data: {
          deletedAt: new Date(),
          content: "[This comment has been hidden due to reports]",
        },
      });
    }
  }
}

export const moderationService = new ModerationService();
```

### Edge Cases e Error Handling

```typescript
// src/server/services/rich-text-comment.ts
import DOMPurify from "isomorphic-dompurify";

// ============================================================
// RICH TEXT SANITIZATION
// ============================================================
const ALLOWED_TAGS = [
  "b", "i", "u", "s", "em", "strong",
  "a", "br", "p",
  "ul", "ol", "li",
  "code", "pre",
  "blockquote",
];

const ALLOWED_ATTRIBUTES: Record<string, string[]> = {
  a: ["href", "target", "rel"],
};

export function sanitizeCommentHTML(html: string): string {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS,
    ALLOWED_ATTR: Object.values(ALLOWED_ATTRIBUTES).flat(),
    ALLOW_DATA_ATTR: false,
    ADD_ATTR: ["target"],
  });

  // Force all links to open in new tab with noopener
  return clean.replace(
    /<a\s/g,
    '<a target="_blank" rel="noopener noreferrer" '
  );
}

// ============================================================
// MENTION PARSER
// ============================================================
interface ParsedMention {
  raw: string;
  username: string;
  startIndex: number;
  endIndex: number;
}

export function parseMentions(content: string): ParsedMention[] {
  const regex = /@(\w{2,30})/g;
  const mentions: ParsedMention[] = [];
  let match;

  while ((match = regex.exec(content)) !== null) {
    mentions.push({
      raw: match[0],
      username: match[1],
      startIndex: match.index,
      endIndex: match.index + match[0].length,
    });
  }

  return mentions;
}

export function renderMentions(content: string): string {
  return content.replace(
    /@(\w{2,30})/g,
    '<span class="mention" data-username="$1">@$1</span>'
  );
}

// ============================================================
// CONTENT LENGTH VALIDATION
// ============================================================
export function validateCommentContent(content: string): {
  valid: boolean;
  error?: string;
} {
  const trimmed = content.trim();

  if (trimmed.length === 0) {
    return { valid: false, error: "Comment cannot be empty" };
  }

  if (trimmed.length < 2) {
    return { valid: false, error: "Comment must be at least 2 characters" };
  }

  if (trimmed.length > 10000) {
    return { valid: false, error: "Comment must be under 10,000 characters" };
  }

  // Check if it's just whitespace/newlines
  if (/^\s+$/.test(trimmed)) {
    return { valid: false, error: "Comment cannot be only whitespace" };
  }

  return { valid: true };
}
```

### Errori Comuni da Evitare
- **Nesting infinito**: Limita la profondita di nesting (3 livelli max)
- **Missing sanitization**: Sanitizza SEMPRE l'HTML dei commenti con DOMPurify
- **Self-notification**: Non notificare l'utente per le proprie azioni
- **Soft delete leak**: Filtra sempre `deletedAt: null` nelle query

### Checklist di Verifica
- [ ] I commenti supportano threading fino a 3 livelli
- [ ] Le menzioni (@username) generano notifiche
- [ ] Il contenuto e sanitizzato per prevenire XSS
- [ ] Il soft delete preserva le risposte ma nasconde il contenuto
- [ ] Il rate limiting previene spam
- [ ] I commenti duplicati sono rilevati

---

## COMMENT-UI-COMPONENTS

### Panoramica
Componenti React completi per il sistema commenti: lista con infinite scroll, form con rich text, item con threading, reactions e azioni di moderazione.

### Implementazione Completa

```typescript
// src/hooks/use-comments.ts
"use client";

import { useCallback, useEffect, useRef, useState } from "react";
import { useInfiniteQuery, useMutation, useQueryClient } from "@tanstack/react-query";

// ============================================================
// TYPES
// ============================================================
interface Comment {
  id: string;
  content: string;
  createdAt: string;
  updatedAt: string;
  deletedAt: string | null;
  author: {
    id: string;
    name: string;
    image: string | null;
  };
  _count: {
    replies: number;
    likes: number;
  };
  isLiked: boolean;
  replies: Comment[];
}

interface CommentsPage {
  comments: Comment[];
  nextCursor: string | null;
  hasMore: boolean;
}

// ============================================================
// COMMENTS HOOK
// ============================================================
export function useComments(postId: string) {
  const queryClient = useQueryClient();

  const commentsQuery = useInfiniteQuery({
    queryKey: ["comments", postId],
    queryFn: async ({ pageParam }) => {
      const params = new URLSearchParams({ postId });
      if (pageParam) params.set("cursor", pageParam);

      const res = await fetch(`/api/comments?${params}`);
      if (!res.ok) throw new Error("Failed to fetch comments");
      return res.json() as Promise<CommentsPage>;
    },
    initialPageParam: undefined as string | undefined,
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
  });

  const addComment = useMutation({
    mutationFn: async (data: { content: string; parentId?: string }) => {
      const res = await fetch("/api/comments", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ ...data, postId }),
      });
      if (!res.ok) throw new Error("Failed to add comment");
      return res.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["comments", postId] });
    },
  });

  const updateComment = useMutation({
    mutationFn: async (data: { commentId: string; content: string }) => {
      const res = await fetch(`/api/comments/${data.commentId}`, {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ content: data.content }),
      });
      if (!res.ok) throw new Error("Failed to update comment");
      return res.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["comments", postId] });
    },
  });

  const deleteComment = useMutation({
    mutationFn: async (commentId: string) => {
      const res = await fetch(`/api/comments/${commentId}`, {
        method: "DELETE",
      });
      if (!res.ok) throw new Error("Failed to delete comment");
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["comments", postId] });
    },
  });

  const toggleLike = useMutation({
    mutationFn: async (commentId: string) => {
      const res = await fetch(`/api/comments/${commentId}/like`, {
        method: "POST",
      });
      if (!res.ok) throw new Error("Failed to toggle like");
      return res.json() as Promise<{ liked: boolean; count: number }>;
    },
    onMutate: async (commentId) => {
      await queryClient.cancelQueries({ queryKey: ["comments", postId] });

      const previous = queryClient.getQueryData(["comments", postId]);

      // Optimistic update
      queryClient.setQueryData(["comments", postId], (old: any) => {
        if (!old) return old;
        return {
          ...old,
          pages: old.pages.map((page: CommentsPage) => ({
            ...page,
            comments: page.comments.map((c: Comment) =>
              c.id === commentId
                ? {
                    ...c,
                    isLiked: !c.isLiked,
                    _count: {
                      ...c._count,
                      likes: c.isLiked ? c._count.likes - 1 : c._count.likes + 1,
                    },
                  }
                : c
            ),
          })),
        };
      });

      return { previous };
    },
    onError: (_err, _commentId, context) => {
      if (context?.previous) {
        queryClient.setQueryData(["comments", postId], context.previous);
      }
    },
  });

  const allComments = commentsQuery.data?.pages.flatMap((p) => p.comments) ?? [];

  return {
    comments: allComments,
    isLoading: commentsQuery.isLoading,
    isFetchingNextPage: commentsQuery.isFetchingNextPage,
    hasNextPage: commentsQuery.hasNextPage,
    fetchNextPage: commentsQuery.fetchNextPage,
    addComment,
    updateComment,
    deleteComment,
    toggleLike,
    totalCount: allComments.length,
  };
}
```

```typescript
// src/components/social/comments/comment-list.tsx
"use client";

import { useComments } from "@/hooks/use-comments";
import { CommentItem } from "./comment-item";
import { CommentForm } from "./comment-form";
import { useInView } from "react-intersection-observer";
import { useEffect } from "react";
import { Loader2, MessageCircle } from "lucide-react";
import { Button } from "@/components/ui/button";

interface CommentListProps {
  postId: string;
  currentUserId?: string;
}

export function CommentList({ postId, currentUserId }: CommentListProps) {
  const {
    comments,
    isLoading,
    isFetchingNextPage,
    hasNextPage,
    fetchNextPage,
    addComment,
    updateComment,
    deleteComment,
    toggleLike,
    totalCount,
  } = useComments(postId);

  // Infinite scroll trigger
  const { ref: loadMoreRef, inView } = useInView({ threshold: 0 });

  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  if (isLoading) {
    return (
      <div className="flex items-center justify-center py-8">
        <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
      </div>
    );
  }

  return (
    <div className="space-y-6">
      {/* Comment count header */}
      <div className="flex items-center gap-2 border-b pb-4">
        <MessageCircle className="h-5 w-5" />
        <h3 className="text-lg font-semibold">
          {totalCount} {totalCount === 1 ? "Comment" : "Comments"}
        </h3>
      </div>

      {/* New comment form */}
      {currentUserId && (
        <CommentForm
          onSubmit={async (content) => {
            await addComment.mutateAsync({ content });
          }}
          isSubmitting={addComment.isPending}
          placeholder="Write a comment..."
        />
      )}

      {/* Comment list */}
      {comments.length === 0 ? (
        <p className="py-8 text-center text-muted-foreground">
          No comments yet. Be the first to comment!
        </p>
      ) : (
        <div className="space-y-4">
          {comments.map((comment) => (
            <CommentItem
              key={comment.id}
              comment={comment}
              currentUserId={currentUserId}
              onReply={async (content) => {
                await addComment.mutateAsync({
                  content,
                  parentId: comment.id,
                });
              }}
              onEdit={async (content) => {
                await updateComment.mutateAsync({
                  commentId: comment.id,
                  content,
                });
              }}
              onDelete={async () => {
                await deleteComment.mutateAsync(comment.id);
              }}
              onLike={async () => {
                await toggleLike.mutateAsync(comment.id);
              }}
              depth={0}
            />
          ))}
        </div>
      )}

      {/* Infinite scroll trigger */}
      {hasNextPage && (
        <div ref={loadMoreRef} className="flex justify-center py-4">
          {isFetchingNextPage ? (
            <Loader2 className="h-5 w-5 animate-spin text-muted-foreground" />
          ) : (
            <Button variant="ghost" size="sm" onClick={() => fetchNextPage()}>
              Load more comments
            </Button>
          )}
        </div>
      )}
    </div>
  );
}
```

```typescript
// src/components/social/comments/comment-item.tsx
"use client";

import { useState } from "react";
import { cn } from "@/lib/utils";
import { Button } from "@/components/ui/button";
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu";
import {
  Heart,
  MessageCircle,
  MoreHorizontal,
  Pencil,
  Trash2,
  Flag,
  ChevronDown,
  ChevronUp,
} from "lucide-react";
import { CommentForm } from "./comment-form";
import { formatDistanceToNow } from "date-fns";

interface Comment {
  id: string;
  content: string;
  createdAt: string;
  updatedAt: string;
  deletedAt: string | null;
  author: {
    id: string;
    name: string;
    image: string | null;
  };
  _count: {
    replies: number;
    likes: number;
  };
  isLiked: boolean;
  replies: Comment[];
}

interface CommentItemProps {
  comment: Comment;
  currentUserId?: string;
  onReply: (content: string) => Promise<void>;
  onEdit: (content: string) => Promise<void>;
  onDelete: () => Promise<void>;
  onLike: () => Promise<void>;
  depth: number;
}

const MAX_DEPTH = 3;

export function CommentItem({
  comment,
  currentUserId,
  onReply,
  onEdit,
  onDelete,
  onLike,
  depth,
}: CommentItemProps) {
  const [isReplying, setIsReplying] = useState(false);
  const [isEditing, setIsEditing] = useState(false);
  const [showReplies, setShowReplies] = useState(depth < 2);
  const [isSubmitting, setIsSubmitting] = useState(false);

  const isOwner = currentUserId === comment.author.id;
  const isDeleted = !!comment.deletedAt;
  const isEdited = comment.updatedAt !== comment.createdAt;
  const canReply = depth < MAX_DEPTH && currentUserId && !isDeleted;

  const handleReply = async (content: string) => {
    setIsSubmitting(true);
    try {
      await onReply(content);
      setIsReplying(false);
    } finally {
      setIsSubmitting(false);
    }
  };

  const handleEdit = async (content: string) => {
    setIsSubmitting(true);
    try {
      await onEdit(content);
      setIsEditing(false);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <div
      className={cn(
        "group",
        depth > 0 && "ml-6 border-l-2 border-border pl-4"
      )}
    >
      <div className="flex gap-3">
        {/* Avatar */}
        <Avatar className="h-8 w-8 shrink-0">
          <AvatarImage src={comment.author.image ?? undefined} />
          <AvatarFallback>
            {comment.author.name.charAt(0).toUpperCase()}
          </AvatarFallback>
        </Avatar>

        <div className="flex-1 space-y-1">
          {/* Header */}
          <div className="flex items-center gap-2">
            <span className="text-sm font-semibold">{comment.author.name}</span>
            <span className="text-xs text-muted-foreground">
              {formatDistanceToNow(new Date(comment.createdAt), {
                addSuffix: true,
              })}
            </span>
            {isEdited && !isDeleted && (
              <span className="text-xs text-muted-foreground">(edited)</span>
            )}
          </div>

          {/* Content or Edit Form */}
          {isEditing ? (
            <CommentForm
              onSubmit={handleEdit}
              onCancel={() => setIsEditing(false)}
              isSubmitting={isSubmitting}
              initialContent={comment.content}
              placeholder="Edit your comment..."
              submitLabel="Save"
            />
          ) : (
            <p
              className={cn(
                "text-sm",
                isDeleted && "italic text-muted-foreground"
              )}
            >
              {comment.content}
            </p>
          )}

          {/* Actions */}
          {!isDeleted && !isEditing && (
            <div className="flex items-center gap-1 pt-1">
              {/* Like */}
              <Button
                variant="ghost"
                size="sm"
                className="h-7 gap-1 px-2 text-xs"
                onClick={onLike}
              >
                <Heart
                  className={cn(
                    "h-3.5 w-3.5",
                    comment.isLiked && "fill-red-500 text-red-500"
                  )}
                />
                {comment._count.likes > 0 && comment._count.likes}
              </Button>

              {/* Reply */}
              {canReply && (
                <Button
                  variant="ghost"
                  size="sm"
                  className="h-7 gap-1 px-2 text-xs"
                  onClick={() => setIsReplying(!isReplying)}
                >
                  <MessageCircle className="h-3.5 w-3.5" />
                  Reply
                </Button>
              )}

              {/* More menu */}
              <DropdownMenu>
                <DropdownMenuTrigger asChild>
                  <Button
                    variant="ghost"
                    size="sm"
                    className="h-7 w-7 p-0 opacity-0 group-hover:opacity-100"
                  >
                    <MoreHorizontal className="h-3.5 w-3.5" />
                  </Button>
                </DropdownMenuTrigger>
                <DropdownMenuContent align="end">
                  {isOwner && (
                    <>
                      <DropdownMenuItem onClick={() => setIsEditing(true)}>
                        <Pencil className="mr-2 h-4 w-4" />
                        Edit
                      </DropdownMenuItem>
                      <DropdownMenuItem
                        onClick={onDelete}
                        className="text-destructive"
                      >
                        <Trash2 className="mr-2 h-4 w-4" />
                        Delete
                      </DropdownMenuItem>
                    </>
                  )}
                  {!isOwner && (
                    <DropdownMenuItem>
                      <Flag className="mr-2 h-4 w-4" />
                      Report
                    </DropdownMenuItem>
                  )}
                </DropdownMenuContent>
              </DropdownMenu>
            </div>
          )}

          {/* Reply form */}
          {isReplying && (
            <div className="mt-2">
              <CommentForm
                onSubmit={handleReply}
                onCancel={() => setIsReplying(false)}
                isSubmitting={isSubmitting}
                placeholder={`Reply to ${comment.author.name}...`}
                submitLabel="Reply"
              />
            </div>
          )}

          {/* Replies */}
          {comment._count.replies > 0 && (
            <div className="mt-2">
              {comment.replies.length > 0 && !showReplies && (
                <Button
                  variant="ghost"
                  size="sm"
                  className="h-7 gap-1 px-2 text-xs text-primary"
                  onClick={() => setShowReplies(true)}
                >
                  <ChevronDown className="h-3.5 w-3.5" />
                  Show {comment._count.replies}{" "}
                  {comment._count.replies === 1 ? "reply" : "replies"}
                </Button>
              )}

              {showReplies && (
                <>
                  {depth < 2 && comment.replies.length > 0 && (
                    <Button
                      variant="ghost"
                      size="sm"
                      className="h-7 gap-1 px-2 text-xs"
                      onClick={() => setShowReplies(false)}
                    >
                      <ChevronUp className="h-3.5 w-3.5" />
                      Hide replies
                    </Button>
                  )}
                  <div className="space-y-3 mt-2">
                    {comment.replies.map((reply) => (
                      <CommentItem
                        key={reply.id}
                        comment={reply}
                        currentUserId={currentUserId}
                        onReply={onReply}
                        onEdit={onEdit}
                        onDelete={onDelete}
                        onLike={onLike}
                        depth={depth + 1}
                      />
                    ))}
                  </div>
                </>
              )}
            </div>
          )}
        </div>
      </div>
    </div>
  );
}
```

```typescript
// src/components/social/comments/comment-form.tsx
"use client";

import { useState, useRef, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { Send, X, Loader2 } from "lucide-react";

interface CommentFormProps {
  onSubmit: (content: string) => Promise<void>;
  onCancel?: () => void;
  isSubmitting: boolean;
  initialContent?: string;
  placeholder?: string;
  submitLabel?: string;
}

export function CommentForm({
  onSubmit,
  onCancel,
  isSubmitting,
  initialContent = "",
  placeholder = "Write a comment...",
  submitLabel = "Comment",
}: CommentFormProps) {
  const [content, setContent] = useState(initialContent);
  const textareaRef = useRef<HTMLTextAreaElement>(null);
  const [isFocused, setIsFocused] = useState(false);

  useEffect(() => {
    if (initialContent && textareaRef.current) {
      textareaRef.current.focus();
    }
  }, [initialContent]);

  // Auto-resize textarea
  useEffect(() => {
    const textarea = textareaRef.current;
    if (!textarea) return;
    textarea.style.height = "auto";
    textarea.style.height = `${Math.min(textarea.scrollHeight, 200)}px`;
  }, [content]);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const trimmed = content.trim();
    if (!trimmed || isSubmitting) return;

    await onSubmit(trimmed);
    setContent("");
  };

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === "Enter" && (e.metaKey || e.ctrlKey)) {
      e.preventDefault();
      handleSubmit(e as any);
    }
    if (e.key === "Escape" && onCancel) {
      onCancel();
    }
  };

  const charCount = content.length;
  const maxChars = 10000;
  const isOverLimit = charCount > maxChars;

  return (
    <form onSubmit={handleSubmit} className="space-y-2">
      <div className="relative">
        <Textarea
          ref={textareaRef}
          value={content}
          onChange={(e) => setContent(e.target.value)}
          onFocus={() => setIsFocused(true)}
          onBlur={() => setIsFocused(false)}
          onKeyDown={handleKeyDown}
          placeholder={placeholder}
          className="min-h-[80px] resize-none pr-4"
          disabled={isSubmitting}
          maxLength={maxChars}
        />
      </div>

      {(isFocused || content.length > 0) && (
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-2 text-xs text-muted-foreground">
            <span className={isOverLimit ? "text-destructive" : ""}>
              {charCount}/{maxChars}
            </span>
            <span>Cmd+Enter to submit</span>
          </div>
          <div className="flex gap-2">
            {onCancel && (
              <Button
                type="button"
                variant="ghost"
                size="sm"
                onClick={onCancel}
                disabled={isSubmitting}
              >
                <X className="mr-1 h-4 w-4" />
                Cancel
              </Button>
            )}
            <Button
              type="submit"
              size="sm"
              disabled={!content.trim() || isSubmitting || isOverLimit}
            >
              {isSubmitting ? (
                <Loader2 className="mr-1 h-4 w-4 animate-spin" />
              ) : (
                <Send className="mr-1 h-4 w-4" />
              )}
              {submitLabel}
            </Button>
          </div>
        </div>
      )}
    </form>
  );
}
```

### Errori Comuni da Evitare
- **Re-render loop con infinite scroll**: Usa `useInView` con `threshold: 0` e check `isFetchingNextPage`
- **Optimistic update non rollback**: Salva lo stato precedente e ripristina in caso di errore
- **Textarea resize non controllato**: Usa `scrollHeight` con un max-height
- **Missing keyboard shortcuts**: Aggiungi Cmd+Enter per submit e Escape per cancel

### Checklist di Verifica
- [ ] Il form supporta Cmd+Enter per submit
- [ ] L'infinite scroll carica automaticamente alla fine della lista
- [ ] Il like ha optimistic update con rollback
- [ ] Il reply form si chiude dopo submit
- [ ] Il character count mostra limite e stato
- [ ] I commenti cancellati mostrano placeholder text
- [ ] Le risposte sono indentate con border-left



---

## NOTIFICATION-SYSTEM

### Panoramica
Sistema di notifiche completo: service layer, real-time via SSE, raggruppamento notifiche, mark as read, bell icon con badge, e pagina notifiche con filtri.

### Implementazione Completa

```typescript
// src/server/services/notification-service.ts
import { prisma } from "@/lib/prisma";

// ============================================================
// NOTIFICATION TYPES
// ============================================================
type NotificationType =
  | "COMMENT"
  | "REPLY"
  | "LIKE"
  | "MENTION"
  | "FOLLOW"
  | "SYSTEM";

interface CreateNotificationInput {
  type: NotificationType;
  recipientId: string;
  actorId: string;
  targetType: string;
  targetId: string;
  message?: string;
}

interface NotificationWithActor {
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
    name: string;
    image: string | null;
  };
}

interface GroupedNotification {
  key: string;
  type: NotificationType;
  targetType: string;
  targetId: string;
  actors: Array<{ id: string; name: string; image: string | null }>;
  count: number;
  latestAt: Date;
  read: boolean;
  message: string;
}

// ============================================================
// NOTIFICATION SERVICE
// ============================================================
export class NotificationService {
  // ----------------------------------------------------------
  // CREATE NOTIFICATION (with deduplication)
  // ----------------------------------------------------------
  async create(input: CreateNotificationInput): Promise<void> {
    // Don't notify yourself
    if (input.recipientId === input.actorId) return;

    // Deduplicate: don't send same notification twice within 5 minutes
    const existing = await prisma.notification.findFirst({
      where: {
        type: input.type as any,
        recipientId: input.recipientId,
        actorId: input.actorId,
        targetType: input.targetType,
        targetId: input.targetId,
        createdAt: { gte: new Date(Date.now() - 5 * 60 * 1000) },
      },
    });

    if (existing) return;

    await prisma.notification.create({
      data: {
        type: input.type as any,
        recipientId: input.recipientId,
        actorId: input.actorId,
        targetType: input.targetType,
        targetId: input.targetId,
      },
    });

    // Emit SSE event for real-time
    this.emitToUser(input.recipientId, "new_notification");
  }

  // ----------------------------------------------------------
  // LIST NOTIFICATIONS (with grouping)
  // ----------------------------------------------------------
  async list(
    userId: string,
    options: {
      cursor?: string;
      limit?: number;
      unreadOnly?: boolean;
      type?: NotificationType;
    } = {}
  ) {
    const limit = options.limit ?? 20;

    const notifications = await prisma.notification.findMany({
      where: {
        recipientId: userId,
        ...(options.unreadOnly && { read: false }),
        ...(options.type && { type: options.type as any }),
      },
      orderBy: { createdAt: "desc" },
      take: limit + 1,
      ...(options.cursor && {
        cursor: { id: options.cursor },
        skip: 1,
      }),
      include: {
        actor: { select: { id: true, name: true, image: true } },
      },
    });

    const hasMore = notifications.length > limit;
    const data = hasMore ? notifications.slice(0, limit) : notifications;

    return {
      notifications: data as NotificationWithActor[],
      nextCursor: hasMore ? data[data.length - 1].id : null,
      hasMore,
    };
  }

  // ----------------------------------------------------------
  // GROUP NOTIFICATIONS
  // ----------------------------------------------------------
  async listGrouped(userId: string): Promise<GroupedNotification[]> {
    const notifications = await prisma.notification.findMany({
      where: {
        recipientId: userId,
        createdAt: { gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) },
      },
      orderBy: { createdAt: "desc" },
      take: 100,
      include: {
        actor: { select: { id: true, name: true, image: true } },
      },
    });

    const groups = new Map<string, GroupedNotification>();

    for (const n of notifications) {
      const key = `${n.type}:${n.targetType}:${n.targetId}`;

      if (groups.has(key)) {
        const group = groups.get(key)!;
        if (!group.actors.find((a) => a.id === n.actorId)) {
          group.actors.push(n.actor);
        }
        group.count++;
        if (!n.read) group.read = false;
      } else {
        groups.set(key, {
          key,
          type: n.type as NotificationType,
          targetType: n.targetType,
          targetId: n.targetId,
          actors: [n.actor],
          count: 1,
          latestAt: n.createdAt,
          read: n.read,
          message: this.formatMessage(
            n.type as NotificationType,
            n.actor.name,
            1
          ),
        });
      }
    }

    // Update messages with grouped counts
    for (const group of groups.values()) {
      group.message = this.formatMessage(
        group.type,
        group.actors[0].name,
        group.actors.length
      );
    }

    return Array.from(groups.values());
  }

  // ----------------------------------------------------------
  // FORMAT NOTIFICATION MESSAGE
  // ----------------------------------------------------------
  private formatMessage(
    type: NotificationType,
    actorName: string,
    count: number
  ): string {
    const others = count > 1 ? ` and ${count - 1} others` : "";

    switch (type) {
      case "COMMENT":
        return `${actorName}${others} commented on your post`;
      case "REPLY":
        return `${actorName}${others} replied to your comment`;
      case "LIKE":
        return `${actorName}${others} liked your comment`;
      case "MENTION":
        return `${actorName}${others} mentioned you in a comment`;
      case "FOLLOW":
        return `${actorName}${others} started following you`;
      case "SYSTEM":
        return "System notification";
      default:
        return `${actorName} interacted with your content`;
    }
  }

  // ----------------------------------------------------------
  // MARK AS READ
  // ----------------------------------------------------------
  async markAsRead(notificationId: string, userId: string): Promise<void> {
    await prisma.notification.updateMany({
      where: { id: notificationId, recipientId: userId },
      data: { read: true },
    });
  }

  async markAllAsRead(userId: string): Promise<number> {
    const result = await prisma.notification.updateMany({
      where: { recipientId: userId, read: false },
      data: { read: true },
    });
    return result.count;
  }

  // ----------------------------------------------------------
  // UNREAD COUNT
  // ----------------------------------------------------------
  async getUnreadCount(userId: string): Promise<number> {
    return prisma.notification.count({
      where: { recipientId: userId, read: false },
    });
  }

  // ----------------------------------------------------------
  // SSE EMITTER (simplified)
  // ----------------------------------------------------------
  private clients = new Map<string, Set<ReadableStreamDefaultController>>();

  registerClient(userId: string, controller: ReadableStreamDefaultController): void {
    if (!this.clients.has(userId)) {
      this.clients.set(userId, new Set());
    }
    this.clients.get(userId)!.add(controller);
  }

  unregisterClient(userId: string, controller: ReadableStreamDefaultController): void {
    this.clients.get(userId)?.delete(controller);
    if (this.clients.get(userId)?.size === 0) {
      this.clients.delete(userId);
    }
  }

  private emitToUser(userId: string, event: string): void {
    const controllers = this.clients.get(userId);
    if (!controllers) return;

    const encoder = new TextEncoder();
    const data = encoder.encode(`event: ${event}\ndata: {}\n\n`);

    for (const controller of controllers) {
      try {
        controller.enqueue(data);
      } catch {
        this.unregisterClient(userId, controller);
      }
    }
  }
}

export const notificationService = new NotificationService();
```

### Varianti e Configurazioni

```typescript
// src/components/social/notifications/notification-bell.tsx
"use client";

import { useState, useEffect } from "react";
import { Bell } from "lucide-react";
import { Button } from "@/components/ui/button";
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from "@/components/ui/popover";
import { ScrollArea } from "@/components/ui/scroll-area";
import { NotificationItem } from "./notification-item";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { cn } from "@/lib/utils";

// ============================================================
// NOTIFICATION BELL WITH BADGE
// ============================================================
export function NotificationBell() {
  const queryClient = useQueryClient();
  const [isOpen, setIsOpen] = useState(false);

  // Fetch unread count
  const { data: unreadCount = 0 } = useQuery({
    queryKey: ["notifications", "unread-count"],
    queryFn: async () => {
      const res = await fetch("/api/notifications/unread-count");
      if (!res.ok) return 0;
      const data = await res.json();
      return data.count as number;
    },
    refetchInterval: 30_000, // Poll every 30s
  });

  // Fetch notifications
  const { data: notifications = [] } = useQuery({
    queryKey: ["notifications", "list"],
    queryFn: async () => {
      const res = await fetch("/api/notifications?limit=10");
      if (!res.ok) return [];
      const data = await res.json();
      return data.notifications;
    },
    enabled: isOpen,
  });

  // Mark all as read
  const markAllRead = useMutation({
    mutationFn: async () => {
      await fetch("/api/notifications/mark-all-read", { method: "POST" });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["notifications"] });
    },
  });

  // SSE for real-time updates
  useEffect(() => {
    const eventSource = new EventSource("/api/notifications/stream");

    eventSource.addEventListener("new_notification", () => {
      queryClient.invalidateQueries({ queryKey: ["notifications"] });
    });

    eventSource.onerror = () => {
      eventSource.close();
    };

    return () => eventSource.close();
  }, [queryClient]);

  return (
    <Popover open={isOpen} onOpenChange={setIsOpen}>
      <PopoverTrigger asChild>
        <Button variant="ghost" size="icon" className="relative">
          <Bell className="h-5 w-5" />
          {unreadCount > 0 && (
            <span
              className={cn(
                "absolute -right-0.5 -top-0.5 flex items-center justify-center",
                "min-w-[18px] h-[18px] rounded-full bg-destructive px-1",
                "text-[10px] font-bold text-destructive-foreground"
              )}
            >
              {unreadCount > 99 ? "99+" : unreadCount}
            </span>
          )}
        </Button>
      </PopoverTrigger>
      <PopoverContent align="end" className="w-[380px] p-0">
        <div className="flex items-center justify-between border-b px-4 py-3">
          <h4 className="text-sm font-semibold">Notifications</h4>
          {unreadCount > 0 && (
            <Button
              variant="ghost"
              size="sm"
              className="h-7 text-xs"
              onClick={() => markAllRead.mutate()}
            >
              Mark all as read
            </Button>
          )}
        </div>

        <ScrollArea className="max-h-[400px]">
          {notifications.length === 0 ? (
            <div className="flex flex-col items-center justify-center py-8 text-muted-foreground">
              <Bell className="mb-2 h-8 w-8" />
              <p className="text-sm">No notifications yet</p>
            </div>
          ) : (
            <div className="divide-y">
              {notifications.map((notification: any) => (
                <NotificationItem
                  key={notification.id}
                  notification={notification}
                />
              ))}
            </div>
          )}
        </ScrollArea>

        <div className="border-t px-4 py-2">
          <Button
            variant="ghost"
            size="sm"
            className="w-full text-xs"
            asChild
          >
            <a href="/notifications">View all notifications</a>
          </Button>
        </div>
      </PopoverContent>
    </Popover>
  );
}
```

```typescript
// src/components/social/notifications/notification-item.tsx
"use client";

import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";
import { cn } from "@/lib/utils";
import {
  Heart,
  MessageCircle,
  AtSign,
  UserPlus,
  Bell,
  Reply,
} from "lucide-react";
import { formatDistanceToNow } from "date-fns";
import Link from "next/link";

interface NotificationItemProps {
  notification: {
    id: string;
    type: string;
    read: boolean;
    createdAt: string;
    targetType: string;
    targetId: string;
    actor: {
      id: string;
      name: string;
      image: string | null;
    };
  };
}

const typeIcons: Record<string, React.ElementType> = {
  COMMENT: MessageCircle,
  REPLY: Reply,
  LIKE: Heart,
  MENTION: AtSign,
  FOLLOW: UserPlus,
  SYSTEM: Bell,
};

const typeColors: Record<string, string> = {
  COMMENT: "text-blue-500",
  REPLY: "text-green-500",
  LIKE: "text-red-500",
  MENTION: "text-purple-500",
  FOLLOW: "text-orange-500",
  SYSTEM: "text-gray-500",
};

const typeMessages: Record<string, string> = {
  COMMENT: "commented on your post",
  REPLY: "replied to your comment",
  LIKE: "liked your comment",
  MENTION: "mentioned you",
  FOLLOW: "started following you",
  SYSTEM: "System notification",
};

function getNotificationLink(targetType: string, targetId: string): string {
  switch (targetType) {
    case "post":
      return `/posts/${targetId}`;
    case "comment":
      return `/comments/${targetId}`;
    case "user":
      return `/users/${targetId}`;
    default:
      return "/notifications";
  }
}

export function NotificationItem({ notification }: NotificationItemProps) {
  const Icon = typeIcons[notification.type] ?? Bell;
  const color = typeColors[notification.type] ?? "text-gray-500";
  const message = typeMessages[notification.type] ?? "interacted with your content";
  const link = getNotificationLink(notification.targetType, notification.targetId);

  return (
    <Link
      href={link}
      className={cn(
        "flex items-start gap-3 px-4 py-3 transition-colors hover:bg-muted/50",
        !notification.read && "bg-primary/5"
      )}
    >
      <div className="relative">
        <Avatar className="h-9 w-9">
          <AvatarImage src={notification.actor.image ?? undefined} />
          <AvatarFallback>
            {notification.actor.name.charAt(0).toUpperCase()}
          </AvatarFallback>
        </Avatar>
        <div
          className={cn(
            "absolute -bottom-1 -right-1 flex h-5 w-5 items-center justify-center rounded-full bg-background",
            color
          )}
        >
          <Icon className="h-3 w-3" />
        </div>
      </div>

      <div className="flex-1 space-y-0.5">
        <p className="text-sm">
          <span className="font-semibold">{notification.actor.name}</span>{" "}
          <span className="text-muted-foreground">{message}</span>
        </p>
        <p className="text-xs text-muted-foreground">
          {formatDistanceToNow(new Date(notification.createdAt), {
            addSuffix: true,
          })}
        </p>
      </div>

      {!notification.read && (
        <div className="mt-2 h-2 w-2 shrink-0 rounded-full bg-primary" />
      )}
    </Link>
  );
}
```

```typescript
// app/api/notifications/stream/route.ts
import { notificationService } from "@/server/services/notification-service";
import { getServerSession } from "next-auth";
import { authOptions } from "@/lib/auth";

// ============================================================
// SSE ENDPOINT FOR REAL-TIME NOTIFICATIONS
// ============================================================
export async function GET() {
  const session = await getServerSession(authOptions);
  if (!session?.user?.id) {
    return new Response("Unauthorized", { status: 401 });
  }

  const userId = session.user.id;

  const stream = new ReadableStream({
    start(controller) {
      // Register client for real-time updates
      notificationService.registerClient(userId, controller);

      // Send initial heartbeat
      const encoder = new TextEncoder();
      controller.enqueue(encoder.encode(":heartbeat\n\n"));

      // Heartbeat every 30 seconds
      const heartbeat = setInterval(() => {
        try {
          controller.enqueue(encoder.encode(":heartbeat\n\n"));
        } catch {
          clearInterval(heartbeat);
          notificationService.unregisterClient(userId, controller);
        }
      }, 30_000);

      // Cleanup on close
      return () => {
        clearInterval(heartbeat);
        notificationService.unregisterClient(userId, controller);
      };
    },
    cancel() {
      // Client disconnected
    },
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache, no-transform",
      Connection: "keep-alive",
    },
  });
}
```

### Edge Cases e Error Handling

```typescript
// src/app/(social)/notifications/page.tsx
"use client";

import { useState } from "react";
import { useInfiniteQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { NotificationItem } from "@/components/social/notifications/notification-item";
import { Button } from "@/components/ui/button";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Loader2, CheckCheck, Bell } from "lucide-react";
import { useInView } from "react-intersection-observer";
import { useEffect } from "react";

// ============================================================
// NOTIFICATIONS PAGE WITH FILTERS
// ============================================================
export default function NotificationsPage() {
  const queryClient = useQueryClient();
  const [filter, setFilter] = useState<string>("all");

  const {
    data,
    isLoading,
    isFetchingNextPage,
    hasNextPage,
    fetchNextPage,
  } = useInfiniteQuery({
    queryKey: ["notifications", "page", filter],
    queryFn: async ({ pageParam }) => {
      const params = new URLSearchParams({ limit: "20" });
      if (pageParam) params.set("cursor", pageParam);
      if (filter === "unread") params.set("unreadOnly", "true");
      if (filter !== "all" && filter !== "unread") {
        params.set("type", filter.toUpperCase());
      }

      const res = await fetch(`/api/notifications?${params}`);
      if (!res.ok) throw new Error("Failed to fetch");
      return res.json();
    },
    initialPageParam: undefined as string | undefined,
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
  });

  const markAllRead = useMutation({
    mutationFn: async () => {
      await fetch("/api/notifications/mark-all-read", { method: "POST" });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["notifications"] });
    },
  });

  const { ref: loadMoreRef, inView } = useInView({ threshold: 0 });

  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  const notifications = data?.pages.flatMap((p) => p.notifications) ?? [];

  return (
    <div className="mx-auto max-w-2xl px-4 py-8">
      <div className="flex items-center justify-between mb-6">
        <h1 className="text-2xl font-bold">Notifications</h1>
        <Button
          variant="outline"
          size="sm"
          onClick={() => markAllRead.mutate()}
          disabled={markAllRead.isPending}
        >
          <CheckCheck className="mr-2 h-4 w-4" />
          Mark all as read
        </Button>
      </div>

      <Tabs value={filter} onValueChange={setFilter}>
        <TabsList className="mb-4">
          <TabsTrigger value="all">All</TabsTrigger>
          <TabsTrigger value="unread">Unread</TabsTrigger>
          <TabsTrigger value="comment">Comments</TabsTrigger>
          <TabsTrigger value="like">Likes</TabsTrigger>
          <TabsTrigger value="mention">Mentions</TabsTrigger>
        </TabsList>

        <TabsContent value={filter}>
          {isLoading ? (
            <div className="flex justify-center py-8">
              <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
            </div>
          ) : notifications.length === 0 ? (
            <div className="flex flex-col items-center justify-center py-16 text-muted-foreground">
              <Bell className="mb-3 h-10 w-10" />
              <p className="text-lg font-medium">No notifications</p>
              <p className="text-sm">
                {filter === "unread"
                  ? "You're all caught up!"
                  : "Nothing here yet."}
              </p>
            </div>
          ) : (
            <div className="divide-y rounded-lg border">
              {notifications.map((notification: any) => (
                <NotificationItem
                  key={notification.id}
                  notification={notification}
                />
              ))}
            </div>
          )}

          {hasNextPage && (
            <div ref={loadMoreRef} className="flex justify-center py-4">
              {isFetchingNextPage && (
                <Loader2 className="h-5 w-5 animate-spin text-muted-foreground" />
              )}
            </div>
          )}
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### Errori Comuni da Evitare
- **Self-notification**: Mai notificare l'utente per le proprie azioni (like al proprio commento)
- **Notification flood**: Raggruppa notifiche simili (3 like -> "User1, User2 and 1 other liked")
- **SSE memory leak**: Registra cleanup dei controller quando il client si disconnette
- **Missing unread indicator**: Il dot blu di unread deve essere visibile anche su mobile
- **Heartbeat mancante**: Invia heartbeat SSE ogni 30s per prevenire timeout proxy

### Checklist di Verifica
- [ ] Le notifiche sono raggruppate per tipo e target
- [ ] Il badge bell mostra il count corretto (max "99+")
- [ ] Mark all as read funziona correttamente
- [ ] Il SSE stream ha heartbeat per mantenere la connessione
- [ ] La pagina notifiche supporta filtri per tipo
- [ ] L'infinite scroll carica le notifiche piu vecchie
- [ ] Le notifiche linkano alla risorsa corretta (post, commento, profilo)



---

## REACTION-SYSTEM

### Panoramica
Sistema di reazioni emoji per commenti e post, con supporto per reaction picker, conteggi aggregati e animazioni.

### Implementazione Completa

```typescript
// src/server/services/reaction-service.ts
import { prisma } from "@/lib/prisma";

// ============================================================
// REACTION TYPES
// ============================================================
export const REACTIONS = {
  like: { emoji: "👍", label: "Like" },
  love: { emoji: "❤️", label: "Love" },
  laugh: { emoji: "😂", label: "Laugh" },
  wow: { emoji: "😮", label: "Wow" },
  sad: { emoji: "😢", label: "Sad" },
  angry: { emoji: "😡", label: "Angry" },
  fire: { emoji: "🔥", label: "Fire" },
  celebrate: { emoji: "🎉", label: "Celebrate" },
} as const;

export type ReactionType = keyof typeof REACTIONS;

interface ReactionSummary {
  type: ReactionType;
  count: number;
  reacted: boolean;
}

// ============================================================
// REACTION SERVICE
// ============================================================
export class ReactionService {
  async toggle(
    targetType: "post" | "comment",
    targetId: string,
    userId: string,
    reactionType: ReactionType
  ): Promise<{ added: boolean; summary: ReactionSummary[] }> {
    const existing = await prisma.reaction.findFirst({
      where: { targetType, targetId, userId, type: reactionType },
    });

    if (existing) {
      await prisma.reaction.delete({ where: { id: existing.id } });
    } else {
      // Remove any other reaction by same user on same target
      await prisma.reaction.deleteMany({
        where: { targetType, targetId, userId },
      });

      await prisma.reaction.create({
        data: { targetType, targetId, userId, type: reactionType },
      });
    }

    const summary = await this.getSummary(targetType, targetId, userId);
    return { added: !existing, summary };
  }

  async getSummary(
    targetType: "post" | "comment",
    targetId: string,
    userId?: string
  ): Promise<ReactionSummary[]> {
    const reactions = await prisma.reaction.groupBy({
      by: ["type"],
      where: { targetType, targetId },
      _count: { type: true },
    });

    const userReactions = userId
      ? await prisma.reaction.findMany({
          where: { targetType, targetId, userId },
          select: { type: true },
        })
      : [];

    const userReactionTypes = new Set(userReactions.map((r) => r.type));

    return reactions
      .map((r) => ({
        type: r.type as ReactionType,
        count: r._count.type,
        reacted: userReactionTypes.has(r.type),
      }))
      .sort((a, b) => b.count - a.count);
  }
}

export const reactionService = new ReactionService();
```

### Varianti e Configurazioni

```typescript
// src/components/social/reactions/reaction-bar.tsx
"use client";

import { useState } from "react";
import { cn } from "@/lib/utils";
import { Button } from "@/components/ui/button";
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from "@/components/ui/popover";
import { SmilePlus } from "lucide-react";
import { REACTIONS, type ReactionType } from "@/server/services/reaction-service";
import { motion, AnimatePresence } from "framer-motion";

interface ReactionSummary {
  type: ReactionType;
  count: number;
  reacted: boolean;
}

interface ReactionBarProps {
  reactions: ReactionSummary[];
  onToggle: (type: ReactionType) => Promise<void>;
  className?: string;
}

export function ReactionBar({ reactions, onToggle, className }: ReactionBarProps) {
  const [isOpen, setIsOpen] = useState(false);
  const [animatingType, setAnimatingType] = useState<ReactionType | null>(null);

  const handleReact = async (type: ReactionType) => {
    setAnimatingType(type);
    setIsOpen(false);
    await onToggle(type);
    setTimeout(() => setAnimatingType(null), 500);
  };

  return (
    <div className={cn("flex items-center gap-1 flex-wrap", className)}>
      {/* Existing reactions */}
      <AnimatePresence>
        {reactions.map((reaction) => (
          <motion.div
            key={reaction.type}
            initial={{ scale: 0 }}
            animate={{ scale: 1 }}
            exit={{ scale: 0 }}
            layout
          >
            <Button
              variant={reaction.reacted ? "secondary" : "ghost"}
              size="sm"
              className={cn(
                "h-7 gap-1 px-2 text-xs",
                reaction.reacted && "border border-primary/30 bg-primary/10"
              )}
              onClick={() => handleReact(reaction.type)}
            >
              <motion.span
                animate={
                  animatingType === reaction.type
                    ? { scale: [1, 1.5, 1] }
                    : {}
                }
                transition={{ duration: 0.3 }}
              >
                {REACTIONS[reaction.type].emoji}
              </motion.span>
              <span>{reaction.count}</span>
            </Button>
          </motion.div>
        ))}
      </AnimatePresence>

      {/* Add reaction button */}
      <Popover open={isOpen} onOpenChange={setIsOpen}>
        <PopoverTrigger asChild>
          <Button
            variant="ghost"
            size="sm"
            className="h-7 w-7 p-0 text-muted-foreground hover:text-foreground"
          >
            <SmilePlus className="h-4 w-4" />
          </Button>
        </PopoverTrigger>
        <PopoverContent className="w-auto p-2" align="start">
          <div className="flex gap-1">
            {Object.entries(REACTIONS).map(([type, { emoji, label }]) => (
              <button
                key={type}
                onClick={() => handleReact(type as ReactionType)}
                className="rounded-md p-1.5 text-lg transition-transform hover:scale-125 hover:bg-muted"
                title={label}
              >
                {emoji}
              </button>
            ))}
          </div>
        </PopoverContent>
      </Popover>
    </div>
  );
}
```

### Errori Comuni da Evitare
- **Multiple reactions same user**: Permetti una sola reazione per utente per target (rimuovi la precedente)
- **Race condition**: Usa upsert o transazione per prevenire reazioni duplicate
- **Missing animation**: Anima l'aggiunta/rimozione per feedback visivo
- **Popover position**: Assicurati che il reaction picker non venga tagliato dal viewport

### Checklist di Verifica
- [ ] Un utente puo avere una sola reazione per target
- [ ] Il cambio reazione rimuove automaticamente la precedente
- [ ] Le reazioni sono raggruppate per tipo con conteggio
- [ ] L'animazione scala l'emoji quando l'utente reagisce
- [ ] Il reaction picker si chiude dopo la selezione
