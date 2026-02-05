---
# Integrato da: 15-OUTPUT-SOCIAL-COMMENTS-NOTIFICATIONS.md
# Data integrazione: 2026-01-29 14:51
# Generato con: Gemini 2.5 Flash / DeepSeek R1
---

```prisma
// FILE 1: prisma/schema-social-comments.prisma
// ~100 righe

// Assuming 'Post' and 'Profile' models are defined elsewhere in the schema
// For example:
// model Post {
//   id        String    @id @default(cuid())
//   authorId  String
//   author    Profile   @relation(fields: [authorId], references: [id])
//   comments  Comment[]
//   // ... other fields
// }
//
// model Profile {
//   id        String    @id @default(cuid())
//   userId    String    @unique
//   user      User      @relation(fields: [userId], references: [id])
//   comments  Comment[]
//   // ... other fields
// }

model Comment {
  id          String    @id @default(cuid())
  postId      String
  post        Post      @relation(fields: [postId], references: [id], onDelete: Cascade)
  authorId    String
  author      Profile   @relation(fields: [authorId], references: [id], onDelete: Cascade)
  
  content     String    @db.Text
  
  // Threading
  parentId    String?
  parent      Comment?  @relation("replies", fields: [parentId], references: [id], onDelete: Cascade)
  replies     Comment[] @relation("replies")
  depth       Int       @default(0) // 0 for top-level, 1 for reply to top-level, etc.
  
  // Engagement
  likesCount  Int       @default(0)
  repliesCount Int      @default(0) // Direct replies count
  
  // Status
  isEdited    Boolean   @default(false)
  isPinned    Boolean   @default(false) // Can be pinned by post author
  
  // Relations
  likes       CommentLike[]
  
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  deletedAt   DateTime? // Soft delete
  
  @@index([postId])
  @@index([authorId])
  @@index([parentId])
  @@index([postId, createdAt]) // For fetching comments for a post
  @@index([postId, isPinned, createdAt]) // For fetching pinned comments first
}

model CommentLike {
  id        String   @id @default(cuid())
  commentId String
  comment   Comment  @relation(fields: [commentId], references: [id], onDelete: Cascade)
  profileId String
  profile   Profile  @relation(fields: [profileId], references: [id], onDelete: Cascade) // Assuming Profile model exists
  
  createdAt DateTime @default(now())
  
  @@unique([commentId, profileId]) // A user can like a comment only once
  @@index([profileId])
}

model Notification {
  id          String           @id @default(cuid())
  recipientId String
  recipient   Profile          @relation(fields: [recipientId], references: [id], onDelete: Cascade) // Assuming Profile model exists
  
  type        NotificationType
  
  // Actor (who triggered the notification)
  actorId     String?
  actor       Profile?         @relation("ActorNotifications", fields: [actorId], references: [id], onDelete: SetNull)
  
  // Related entities
  postId      String?
  post        Post?            @relation(fields: [postId], references: [id], onDelete: SetNull) // Assuming Post model exists
  commentId   String?
  comment     Comment?         @relation(fields: [commentId], references: [id], onDelete: SetNull)
  followId    String?          // ID of the Follow record, if applicable
  
  // Content (for flexibility)
  title       String?          // Optional custom title
  body        String?          // Optional custom body text
  imageUrl    String?          // Optional image URL (e.g., actor's avatar)
  actionUrl   String?          // URL to navigate to when clicking the notification
  
  // Status
  isRead      Boolean          @default(false)
  readAt      DateTime?
  
  createdAt   DateTime         @default(now())
  
  @@index([recipientId])
  @@index([recipientId, isRead]) // For quickly querying unread notifications
  @@index([createdAt])
  @@index([recipientId, createdAt]) // For fetching recent notifications
}

enum NotificationType {
  FOLLOW
  FOLLOW_REQUEST
  FOLLOW_ACCEPTED
  LIKE_POST
  LIKE_COMMENT
  COMMENT // A new comment on a post you own or follow
  REPLY // A reply to your comment
  MENTION // You were mentioned in a post or comment
  REPOST // Your post was reposted
  SYSTEM // General system notification
  POST_CREATED // A user you follow created a new post
}

// Notification preferences for a user
model NotificationPreferences {
  id                      String    @id @default(cuid())
  profileId               String    @unique
  profile                 Profile   @relation(fields: [profileId], references: [id], onDelete: Cascade)

  // Email preferences
  email_newFollower       Boolean   @default(true)
  email_postLike          Boolean   @default(true)
  email_comment           Boolean   @default(true)
  email_reply             Boolean   @default(true)
  email_mention           Boolean   @default(true)
  email_system            Boolean   @default(true)

  // In-app preferences
  inApp_newFollower       Boolean   @default(true)
  inApp_postLike          Boolean   @default(true)
  inApp_comment           Boolean   @default(true)
  inApp_reply             Boolean   @default(true)
  inApp_mention           Boolean   @default(true)
  inApp_system            Boolean   @default(true)

  // Push preferences (if applicable)
  push_newFollower        Boolean   @default(true)
  push_postLike           Boolean   @default(true)
  push_comment            Boolean   @default(true)
  push_reply              Boolean   @default(true)
  push_mention            Boolean   @default(true)
  push_system             Boolean   @default(true)

  createdAt               DateTime  @default(now())
  updatedAt               DateTime  @updatedAt
}

// Model to store push notification tokens for a user
model PushSubscription {
  id          String    @id @default(cuid())
  profileId   String
  profile     Profile   @relation(fields: [profileId], references: [id], onDelete: Cascade)
  token       String    @unique // The actual push token (e.g., FCM, APN)
  platform    String    // 'web', 'ios', 'android'
  deviceInfo  Json?     // Optional: user agent, device model, etc.
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  @@index([profileId])
}
```

```typescript
// FILE 2: src/server/services/comment-service.ts
// ~250 righe
import { PrismaClient, Comment, CommentLike, NotificationType } from '@prisma/client';
import { z } from 'zod';
import { CreateCommentInput, UpdateCommentInput, CommentListParams, ListParams } from '../../lib/validations/comments';
import { PaginatedResult } from '../utils/pagination';
import { NotificationService } from './notification-service'; // Assuming NotificationService exists
import { PostService } from './post-service'; // Assuming PostService exists

// Define a simplified Comment structure for threading
export type CommentWithAuthor = Comment & { author: { id: string; name: string; image?: string }; likes: { profileId: string }[] };
export type CommentThread = CommentWithAuthor & { replies: CommentThread[] };

export class CommentService {
  private prisma: PrismaClient;
  private notificationService: NotificationService;
  private postService: PostService;

  constructor(prisma: PrismaClient, notificationService: NotificationService, postService: PostService) {
    this.prisma = prisma;
    this.notificationService = notificationService;
    this.postService = postService;
  }

  // --- CRUD Operations ---

  /**
   * Creates a new comment or a reply to an existing comment.
   * Automatically updates parent comment's reply count and post's comment count.
   */
  async create(postId: string, authorId: string, data: CreateCommentInput): Promise<Comment> {
    const { content, parentId } = data;
    let depth = 0;

    if (parentId) {
      const parentComment = await this.prisma.comment.findUnique({ where: { id: parentId } });
      if (!parentComment) {
        throw new Error('Parent comment not found.');
      }
      depth = parentComment.depth + 1;
    }

    const newComment = await this.prisma.comment.create({
      data: {
        postId,
        authorId,
        content,
        parentId,
        depth,
      },
    });

    // Update counts
    if (parentId) {
      await this.updateRepliesCount(parentId);
      // Notify parent comment author about the reply
      const parentComment = await this.prisma.comment.findUnique({ where: { id: parentId }, select: { authorId: true } });
      if (parentComment && parentComment.authorId !== authorId) {
        await this.notificationService.notifyReply(parentId, authorId, newComment.id);
      }
    }
    await this.updatePostCommentCount(postId);

    // Notify post author about new comment (if not a reply to their own comment)
    const post = await this.prisma.post.findUnique({ where: { id: postId }, select: { authorId: true } });
    if (post && post.authorId !== authorId && !parentId) { // Only notify for top-level comments
      await this.notificationService.notifyComment(postId, authorId, newComment.id);
    }

    // Handle mentions
    const mentionedUserIds = this.extractMentions(content);
    for (const mentionedId of mentionedUserIds) {
      if (mentionedId !== authorId) { // Don't notify self
        await this.notificationService.notifyMention(postId, mentionedId, authorId, newComment.id);
      }
    }

    return newComment;
  }

  /**
   * Updates an existing comment's content.
   */
  async update(commentId: string, data: UpdateCommentInput): Promise<Comment> {
    const { content } = data;
    return this.prisma.comment.update({
      where: { id: commentId },
      data: {
        content,
        isEdited: true,
      },
    });
  }

  /**
   * Soft deletes a comment.
   * Decrements parent comment's reply count and post's comment count.
   */
  async delete(commentId: string): Promise<void> {
    const comment = await this.prisma.comment.update({
      where: { id: commentId },
      data: { deletedAt: new Date() },
      select: { postId: true, parentId: true },
    });

    if (comment.parentId) {
      await this.updateRepliesCount(comment.parentId);
    }
    await this.updatePostCommentCount(comment.postId);
  }

  /**
   * Retrieves a comment by its ID.
   */
  async getById(id: string): Promise<Comment | null> {
    return this.prisma.comment.findUnique({
      where: { id, deletedAt: null },
      include: {
        author: { select: { id: true, name: true, image: true } },
        likes: { select: { profileId: true } },
      },
    });
  }

  // --- Queries ---

  /**
   * Retrieves top-level comments for a specific post.
   */
  async getByPost(postId: string, params?: CommentListParams): Promise<PaginatedResult<CommentWithAuthor>> {
    const { page = 1, pageSize = 10, sortBy = 'createdAt', sortOrder = 'desc' } = params || {};
    const skip = (page - 1) * pageSize;

    const [comments, total] = await this.prisma.$transaction([
      this.prisma.comment.findMany({
        where: { postId, parentId: null, deletedAt: null },
        include: {
          author: { select: { id: true, name: true, image: true } },
          likes: { select: { profileId: true } },
        },
        orderBy: [
          { isPinned: 'desc' }, // Pinned comments first
          { [sortBy]: sortOrder },
        ],
        skip,
        take: pageSize,
      }),
      this.prisma.comment.count({ where: { postId, parentId: null, deletedAt: null } }),
    ]);

    return {
      data: comments,
      total,
      page,
      pageSize,
      totalPages: Math.ceil(total / pageSize),
    };
  }

  /**
   * Retrieves direct replies to a specific comment.
   */
  async getReplies(commentId: string, params?: ListParams): Promise<PaginatedResult<CommentWithAuthor>> {
    const { page = 1, pageSize = 5, sortBy = 'createdAt', sortOrder = 'asc' } = params || {};
    const skip = (page - 1) * pageSize;

    const [replies, total] = await this.prisma.$transaction([
      this.prisma.comment.findMany({
        where: { parentId: commentId, deletedAt: null },
        include: {
          author: { select: { id: true, name: true, image: true } },
          likes: { select: { profileId: true } },
        },
        orderBy: { [sortBy]: sortOrder },
        skip,
        take: pageSize,
      }),
      this.prisma.comment.count({ where: { parentId: commentId, deletedAt: null } }),
    ]);

    return {
      data: replies,
      total,
      page,
      pageSize,
      totalPages: Math.ceil(total / pageSize),
    };
  }

  /**
   * Retrieves all comments made by a specific author.
   */
  async getByAuthor(authorId: string, params?: ListParams): Promise<PaginatedResult<CommentWithAuthor>> {
    const { page = 1, pageSize = 10, sortBy = 'createdAt', sortOrder = 'desc' } = params || {};
    const skip = (page - 1) * pageSize;

    const [comments, total] = await this.prisma.$transaction([
      this.prisma.comment.findMany({
        where: { authorId, deletedAt: null },
        include: {
          author: { select: { id: true, name: true, image: true } },
          likes: { select: { profileId: true } },
        },
        orderBy: { [sortBy]: sortOrder },
        skip,
        take: pageSize,
      }),
      this.prisma.comment.count({ where: { authorId, deletedAt: null } }),
    ]);

    return {
      data: comments,
      total,
      page,
      pageSize,
      totalPages: Math.ceil(total / pageSize),
    };
  }

  // --- Threading ---

  /**
   * Retrieves comments for a post in a threaded structure.
   * This is a more complex query, often better handled with recursive CTEs in SQL
   * or by fetching top-level and then replies separately in application logic.
   * For simplicity, this example fetches top-level and then a limited depth of replies.
   */
  async getThreadedComments(postId: string, maxDepth: number = 2): Promise<CommentThread[]> {
    const topLevelComments = await this.prisma.comment.findMany({
      where: { postId, parentId: null, deletedAt: null },
      include: {
        author: { select: { id: true, name: true, image: true } },
        likes: { select: { profileId: true } },
      },
      orderBy: [
        { isPinned: 'desc' },
        { createdAt: 'desc' },
      ],
      take: 20, // Limit top-level comments for initial load
    });

    const buildThread = async (comment: CommentWithAuthor, currentDepth: number): Promise<CommentThread> => {
      const replies: CommentThread[] = [];
      if (currentDepth < maxDepth) {
        const directReplies = await this.prisma.comment.findMany({
          where: { parentId: comment.id, deletedAt: null },
          include: {
            author: { select: { id: true, name: true, image: true } },
            likes: { select: { profileId: true } },
          },
          orderBy: { createdAt: 'asc' },
          take: 3, // Limit direct replies shown initially
        });
        for (const reply of directReplies) {
          replies.push(await buildThread(reply as CommentWithAuthor, currentDepth + 1));
        }
      }
      return { ...comment, replies };
    };

    const threadedComments: CommentThread[] = [];
    for (const comment of topLevelComments) {
      threadedComments.push(await buildThread(comment as CommentWithAuthor, 0));
    }

    return threadedComments;
  }

  // --- Engagement ---

  /**
   * Likes a comment. Creates a CommentLike entry and increments likesCount.
   */
  async like(commentId: string, profileId: string): Promise<void> {
    try {
      await this.prisma.commentLike.create({
        data: {
          commentId,
          profileId,
        },
      });
      await this.prisma.comment.update({
        where: { id: commentId },
        data: { likesCount: { increment: 1 } },
      });

      // Notify comment author about the like
      const comment = await this.prisma.comment.findUnique({ where: { id: commentId }, select: { authorId: true, postId: true } });
      if (comment && comment.authorId !== profileId) {
        await this.notificationService.notifyLike(comment.postId, profileId, commentId);
      }
    } catch (error: any) {
      if (error.code === 'P2002') { // Unique constraint failed (already liked)
        console.warn(`Profile ${profileId} already liked comment ${commentId}`);
      } else {
        throw error;
      }
    }
  }

  /**
   * Unlikes a comment. Deletes a CommentLike entry and decrements likesCount.
   */
  async unlike(commentId: string, profileId: string): Promise<void> {
    try {
      await this.prisma.commentLike.delete({
        where: {
          commentId_profileId: {
            commentId,
            profileId,
          },
        },
      });
      await this.prisma.comment.update({
        where: { id: commentId },
        data: { likesCount: { decrement: 1 } },
      });
    } catch (error: any) {
      if (error.code === 'P2025') { // Record not found (not liked)
        console.warn(`Profile ${profileId} had not liked comment ${commentId}`);
      } else {
        throw error;
      }
    }
  }

  /**
   * Checks if a profile has liked a specific comment.
   */
  async isLiked(commentId: string, profileId: string): Promise<boolean> {
    const like = await this.prisma.commentLike.findUnique({
      where: {
        commentId_profileId: {
          commentId,
          profileId,
        },
      },
    });
    return !!like;
  }

  // --- Pinning ---

  /**
   * Pins a comment. Only the post author can pin comments on their post.
   */
  async pin(commentId: string, postAuthorId: string): Promise<void> {
    const comment = await this.prisma.comment.findUnique({
      where: { id: commentId },
      select: { postId: true },
    });

    if (!comment) {
      throw new Error('Comment not found.');
    }

    const post = await this.prisma.post.findUnique({
      where: { id: comment.postId },
      select: { authorId: true },
    });

    if (!post || post.authorId !== postAuthorId) {
      throw new Error('Unauthorized: Only the post author can pin comments.');
    }

    await this.prisma.comment.update({
      where: { id: commentId },
      data: { isPinned: true },
    });
  }

  /**
   * Unpins a comment.
   */
  async unpin(commentId: string): Promise<void> {
    await this.prisma.comment.update({
      where: { id: commentId },
      data: { isPinned: false },
    });
  }

  // --- Stats ---

  /**
   * Recalculates and updates the likesCount and repliesCount for a comment.
   * Useful for data integrity checks.
   */
  async updateStats(commentId: string): Promise<void> {
    const likesCount = await this.prisma.commentLike.count({ where: { commentId } });
    const repliesCount = await this.prisma.comment.count({ where: { parentId: commentId, deletedAt: null } });

    await this.prisma.comment.update({
      where: { id: commentId },
      data: { likesCount, repliesCount },
    });
  }

  /**
   * Recalculates and updates the total comment count for a post.
   */
  async updatePostCommentCount(postId: string): Promise<void> {
    const commentCount = await this.prisma.comment.count({ where: { postId, deletedAt: null } });
    await this.postService.updateCommentCount(postId, commentCount);
  }

  /**
   * Internal helper to update repliesCount for a parent comment.
   */
  private async updateRepliesCount(parentId: string): Promise<void> {
    const repliesCount = await this.prisma.comment.count({ where: { parentId, deletedAt: null } });
    await this.prisma.comment.update({
      where: { id: parentId },
      data: { repliesCount },
    });
  }

  /**
   * Extracts mentioned profile IDs from comment content.
   * Assumes mentions are in the format `@username`.
   * This is a simplified example; a real implementation would need to resolve usernames to IDs.
   */
  private async extractMentions(content: string): Promise<string[]> {
    const mentionRegex = /@([a-zA-Z0-9_]+)/g;
    const matches = content.match(mentionRegex);
    if (!matches) return [];

    const usernames = matches.map(match => match.substring(1));
    // In a real app, you'd query your Profile model to get IDs from usernames
    // For this example, let's assume a dummy mapping or direct ID usage for simplicity
    // e.g., const profiles = await this.prisma.profile.findMany({ where: { username: { in: usernames } }, select: { id: true } });
    // return profiles.map(p => p.id);
    return []; // Placeholder
  }
}
```

```typescript
// FILE 3: src/server/services/notification-service.ts
// ~300 righe
import { PrismaClient, Notification, NotificationType, NotificationPreferences, PushSubscription } from '@prisma/client';
import { z } from 'zod';
import { CreateNotificationInput, NotificationListParams, UpdateNotificationPreferencesInput } from '../../lib/validations/notifications';
import { PaginatedResult } from '../utils/pagination';

// Placeholder for a real-time communication layer (e.g., WebSocket server)
interface RealtimeService {
  publish: (channel: string, event: string, payload: any) => void;
  subscribe: (channel: string, callback: (payload: any) => void) => () => void;
}

// Placeholder for a push notification service (e.g., Firebase Cloud Messaging)
interface PushNotificationProvider {
  send: (token: string, payload: { title: string; body: string; data?: Record<string, any> }) => Promise<void>;
}

export class NotificationService {
  private prisma: PrismaClient;
  private realtimeService?: RealtimeService; // Optional, for real-time features
  private pushProvider?: PushNotificationProvider; // Optional, for push notifications

  constructor(prisma: PrismaClient, realtimeService?: RealtimeService, pushProvider?: PushNotificationProvider) {
    this.prisma = prisma;
    this.realtimeService = realtimeService;
    this.pushProvider = pushProvider;
  }

  // --- Create Operations ---

  /**
   * Creates a single notification.
   * Publishes to real-time channel and sends push notification if preferences allow.
   */
  async create(data: CreateNotificationInput): Promise<Notification> {
    const { recipientId, type, actorId, postId, commentId, followId, title, body, imageUrl, actionUrl } = data;

    // Check user preferences before creating/sending
    const shouldNotify = await this.shouldNotify(recipientId, type, 'inApp');
    if (!shouldNotify) {
      // console.log(`Notification of type ${type} for ${recipientId} skipped due to preferences.`);
      // We might still create it in DB but mark as 'hidden' or just skip entirely.
      // For now, we skip DB creation if in-app is off.
      // A more robust system might create it but not display it.
      return null as any; // Return null or throw specific error if not created
    }

    const notification = await this.prisma.notification.create({
      data: {
        recipientId,
        type,
        actorId,
        postId,
        commentId,
        followId,
        title,
        body,
        imageUrl,
        actionUrl,
      },
    });

    // Publish to real-time channel
    if (this.realtimeService) {
      this.realtimeService.publish(`notifications:${recipientId}`, 'newNotification', notification);
    }

    // Send push notification
    this.sendPushNotificationToRecipient(recipientId, notification);

    return notification;
  }

  /**
   * Creates multiple notifications in a batch.
   */
  async createMany(notifications: CreateNotificationInput[]): Promise<number> {
    const createdNotifications: Notification[] = [];
    for (const data of notifications) {
      const notification = await this.create(data);
      if (notification) {
        createdNotifications.push(notification);
      }
    }
    return createdNotifications.length;
  }

  // --- Specific Notification Types ---

  async notifyFollow(followerId: string, followingId: string): Promise<void> {
    const followerProfile = await this.prisma.profile.findUnique({ where: { id: followerId }, select: { name: true, image: true } });
    if (!followerProfile) return;

    await this.create({
      recipientId: followingId,
      type: NotificationType.FOLLOW,
      actorId: followerId,
      title: `${followerProfile.name} started following you.`,
      body: `See ${followerProfile.name}'s profile.`,
      imageUrl: followerProfile.image || undefined,
      actionUrl: `/profile/${followerId}`,
    });
  }

  async notifyLike(postId: string, likerId: string, commentId?: string): Promise<void> {
    const likerProfile = await this.prisma.profile.findUnique({ where: { id: likerId }, select: { name: true, image: true } });
    if (!likerProfile) return;

    let recipientId: string | undefined;
    let actionUrl: string | undefined;
    let type: NotificationType;
    let title: string;

    if (commentId) {
      const comment = await this.prisma.comment.findUnique({ where: { id: commentId }, select: { authorId: true } });
      if (!comment) return;
      recipientId = comment.authorId;
      type = NotificationType.LIKE_COMMENT;
      title = `${likerProfile.name} liked your comment.`;
      actionUrl = `/post/${postId}?commentId=${commentId}`;
    } else {
      const post = await this.prisma.post.findUnique({ where: { id: postId }, select: { authorId: true } });
      if (!post) return;
      recipientId = post.authorId;
      type = NotificationType.LIKE_POST;
      title = `${likerProfile.name} liked your post.`;
      actionUrl = `/post/${postId}`;
    }

    if (recipientId && recipientId !== likerId) { // Don't notify self
      await this.create({
        recipientId,
        type,
        actorId: likerId,
        postId,
        commentId,
        title,
        imageUrl: likerProfile.image || undefined,
        actionUrl,
      });
    }
  }

  async notifyComment(postId: string, commenterId: string, commentId: string): Promise<void> {
    const commenterProfile = await this.prisma.profile.findUnique({ where: { id: commenterId }, select: { name: true, image: true } });
    const post = await this.prisma.post.findUnique({ where: { id: postId }, select: { authorId: true } });
    if (!commenterProfile || !post) return;

    if (post.authorId !== commenterId) { // Don't notify self
      await this.create({
        recipientId: post.authorId,
        type: NotificationType.COMMENT,
        actorId: commenterId,
        postId,
        commentId,
        title: `${commenterProfile.name} commented on your post.`,
        imageUrl: commenterProfile.image || undefined,
        actionUrl: `/post/${postId}?commentId=${commentId}`,
      });
    }
  }

  async notifyReply(parentCommentId: string, replierId: string, replyId: string): Promise<void> {
    const replierProfile = await this.prisma.profile.findUnique({ where: { id: replierId }, select: { name: true, image: true } });
    const parentComment = await this.prisma.comment.findUnique({ where: { id: parentCommentId }, select: { authorId: true, postId: true } });
    if (!replierProfile || !parentComment) return;

    if (parentComment.authorId !== replierId) { // Don't notify self
      await this.create({
        recipientId: parentComment.authorId,
        type: NotificationType.REPLY,
        actorId: replierId,
        postId: parentComment.postId,
        commentId: replyId,
        title: `${replierProfile.name} replied to your comment.`,
        imageUrl: replierProfile.image || undefined,
        actionUrl: `/post/${parentComment.postId}?commentId=${replyId}`,
      });
    }
  }

  async notifyMention(postId: string, mentionedId: string, mentionerId: string, commentId?: string): Promise<void> {
    const mentionerProfile = await this.prisma.profile.findUnique({ where: { id: mentionerId }, select: { name: true, image: true } });
    if (!mentionerProfile) return;

    let actionUrl = `/post/${postId}`;
    if (commentId) {
      actionUrl += `?commentId=${commentId}`;
    }

    if (mentionedId !== mentionerId) { // Don't notify self
      await this.create({
        recipientId: mentionedId,
        type: NotificationType.MENTION,
        actorId: mentionerId,
        postId,
        commentId,
        title: `${mentionerProfile.name} mentioned you.`,
        imageUrl: mentionerProfile.image || undefined,
        actionUrl,
      });
    }
  }

  async notifyRepost(postId: string, reposterId: string): Promise<void> {
    const reposterProfile = await this.prisma.profile.findUnique({ where: { id: reposterId }, select: { name: true, image: true } });
    const originalPost = await this.prisma.post.findUnique({ where: { id: postId }, select: { authorId: true } });
    if (!reposterProfile || !originalPost) return;

    if (originalPost.authorId !== reposterId) { // Don't notify self
      await this.create({
        recipientId: originalPost.authorId,
        type: NotificationType.REPOST,
        actorId: reposterId,
        postId,
        title: `${reposterProfile.name} reposted your post.`,
        imageUrl: reposterProfile.image || undefined,
        actionUrl: `/post/${postId}`,
      });
    }
  }

  // --- Queries ---

  /**
   * Retrieves notifications for a specific recipient, with pagination.
   */
  async getByRecipient(recipientId: string, params?: NotificationListParams): Promise<PaginatedResult<Notification & { actor: { id: string; name: string; image?: string } | null; post: { id: string; title?: string } | null; comment: { id: string; content: string } | null }>> {
    const { page = 1, pageSize = 10, isRead, type } = params || {};
    const skip = (page - 1) * pageSize;

    const whereClause: any = { recipientId, deletedAt: null };
    if (isRead !== undefined) {
      whereClause.isRead = isRead;
    }
    if (type) {
      whereClause.type = type;
    }

    const [notifications, total] = await this.prisma.$transaction([
      this.prisma.notification.findMany({
        where: whereClause,
        include: {
          actor: { select: { id: true, name: true, image: true } },
          post: { select: { id: true, title: true } }, // Assuming Post has a title
          comment: { select: { id: true, content: true } },
        },
        orderBy: { createdAt: 'desc' },
        skip,
        take: pageSize,
      }),
      this.prisma.notification.count({ where: whereClause }),
    ]);

    return {
      data: notifications,
      total,
      page,
      pageSize,
      totalPages: Math.ceil(total / pageSize),
    };
  }

  /**
   * Gets the count of unread notifications for a recipient.
   */
  async getUnreadCount(recipientId: string): Promise<number> {
    return this.prisma.notification.count({
      where: { recipientId, isRead: false, deletedAt: null },
    });
  }

  /**
   * Retrieves a limited number of recent unread notifications.
   */
  async getUnread(recipientId: string, limit: number = 5): Promise<(Notification & { actor: { id: string; name: string; image?: string } | null; post: { id: string; title?: string } | null; comment: { id: string; content: string } | null })[]> {
    return this.prisma.notification.findMany({
      where: { recipientId, isRead: false, deletedAt: null },
      include: {
        actor: { select: { id: true, name: true, image: true } },
        post: { select: { id: true, title: true } },
        comment: { select: { id: true, content: true } },
      },
      orderBy: { createdAt: 'desc' },
      take: limit,
    });
  }

  // --- Actions ---

  /**
   * Marks a single notification as read.
   */
  async markAsRead(notificationId: string): Promise<void> {
    await this.prisma.notification.update({
      where: { id: notificationId },
      data: { isRead: true, readAt: new Date() },
    });
  }

  /**
   * Marks all unread notifications for a recipient as read.
   */
  async markAllAsRead(recipientId: string): Promise<number> {
    const { count } = await this.prisma.notification.updateMany({
      where: { recipientId, isRead: false },
      data: { isRead: true, readAt: new Date() },
    });
    return count;
  }

  /**
   * Deletes a single notification.
   */
  async delete(notificationId: string): Promise<void> {
    await this.prisma.notification.delete({ where: { id: notificationId } });
  }

  /**
   * Deletes old notifications for a recipient.
   */
  async deleteOld(recipientId: string, olderThan: Date): Promise<number> {
    const { count } = await this.prisma.notification.deleteMany({
      where: {
        recipientId,
        createdAt: { lt: olderThan },
        isRead: true, // Typically only delete read notifications
      },
    });
    return count;
  }

  // --- Real-time ---

  /**
   * Subscribes a callback to real-time notifications for a recipient.
   * Returns an unsubscribe function.
   */
  async subscribeToNotifications(recipientId: string, callback: (notification: Notification) => void): Promise<() => void> {
    if (!this.realtimeService) {
      console.warn('Realtime service not configured. Real-time notifications will not work.');
      return () => {};
    }
    const channel = `notifications:${recipientId}`;
    this.realtimeService.subscribe(channel, callback);
    return () => this.realtimeService?.publish(channel, 'unsubscribe', {}); // Placeholder for actual unsubscribe
  }

  // --- Preferences ---

  /**
   * Gets a recipient's notification preferences, creating defaults if none exist.
   */
  async getPreferences(recipientId: string): Promise<NotificationPreferences> {
    let preferences = await this.prisma.notificationPreferences.findUnique({
      where: { profileId: recipientId },
    });

    if (!preferences) {
      preferences = await this.prisma.notificationPreferences.create({
        data: { profileId: recipientId }, // Default values are set by Prisma
      });
    }
    return preferences;
  }

  /**
   * Updates a recipient's notification preferences.
   */
  async updatePreferences(recipientId: string, preferences: UpdateNotificationPreferencesInput): Promise<NotificationPreferences> {
    return this.prisma.notificationPreferences.update({
      where: { profileId: recipientId },
      data: preferences,
    });
  }

  /**
   * Checks if a notification of a given type should be sent/displayed based on user preferences.
   * @param recipientId The ID of the user to check preferences for.
   * @param type The type of notification.
   * @param channel The channel to check preferences for ('inApp', 'email', 'push').
   */
  async shouldNotify(recipientId: string, type: NotificationType, channel: 'inApp' | 'email' | 'push'): Promise<boolean> {
    const preferences = await this.getPreferences(recipientId); // This will create defaults if none exist

    const preferenceKey = `${channel}_${type.toLowerCase()}` as keyof NotificationPreferences;
    return preferences[preferenceKey] as boolean;
  }

  // --- Push notifications ---

  /**
   * Sends a push notification to a specific recipient.
   * Fetches all registered push tokens for the recipient and sends to each.
   */
  async sendPushNotificationToRecipient(recipientId: string, notification: Notification): Promise<void> {
    if (!this.pushProvider) {
      console.warn('Push notification provider not configured.');
      return;
    }

    const shouldSendPush = await this.shouldNotify(recipientId, notification.type, 'push');
    if (!shouldSendPush) {
      // console.log(`Push notification of type ${notification.type} for ${recipientId} skipped due to preferences.`);
      return;
    }

    const subscriptions = await this.prisma.pushSubscription.findMany({
      where: { profileId: recipientId },
    });

    for (const sub of subscriptions) {
      try {
        await this.pushProvider.send(sub.token, {
          title: notification.title || `New ${notification.type.toLowerCase()}!`,
          body: notification.body || 'You have a new notification.',
          data: {
            notificationId: notification.id,
            type: notification.type,
            actionUrl: notification.actionUrl || `/notifications`,
            actorId: notification.actorId || undefined,
            postId: notification.postId || undefined,
            commentId: notification.commentId || undefined,
          },
        });
      } catch (error) {
        console.error(`Failed to send push notification to token ${sub.token}:`, error);
        // Optionally, delete invalid tokens here
      }
    }
  }

  /**
   * Registers a push notification token for a recipient.
   */
  async registerPushToken(recipientId: string, token: string, platform: 'web' | 'ios' | 'android', deviceInfo?: any): Promise<PushSubscription> {
    return this.prisma.pushSubscription.upsert({
      where: { token }, // Use token as unique identifier for upsert
      update: {
        profileId: recipientId,
        platform,
        deviceInfo,
      },
      create: {
        profileId: recipientId,
        token,
        platform,
        deviceInfo,
      },
    });
  }
}
```

```typescript
// FILE 4: src/server/trpc/routers/comments.ts
// ~150 righe
import { router, publicProcedure, protectedProcedure } from '../trpc'; // Assuming these are defined
import { z } from 'zod';
import { prisma } from '../../db'; // Assuming prisma client is initialized
import { CommentService } from '../../services/comment-service';
import { NotificationService } from '../../services/notification-service';
import { PostService } from '../../services/post-service';
import {
  createCommentSchema,
  updateCommentSchema,
  getCommentsByPostSchema,
  getRepliesSchema,
  likeCommentSchema,
  pinCommentSchema,
  deleteCommentSchema,
  getCommentByIdSchema,
} from '../../../lib/validations/comments';
import { TRPCError } from '@trpc/server';

// Initialize services (dependency injection)
const postService = new PostService(prisma); // Assuming PostService exists
const notificationService = new NotificationService(prisma); // Assuming NotificationService exists
const commentService = new CommentService(prisma, notificationService, postService);

export const commentsRouter = router({
  // Create a new comment or reply
  create: protectedProcedure
    .input(createCommentSchema)
    .mutation(async ({ input, ctx }) => {
      const { postId, content, parentId } = input;
      const authorId = ctx.session.user.profileId; // Assuming session contains profileId

      try {
        const comment = await commentService.create(postId, authorId, { content, parentId });
        return comment;
      } catch (error) {
        console.error('Error creating comment:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to create comment.',
          cause: error,
        });
      }
    }),

  // Update an existing comment
  update: protectedProcedure
    .input(updateCommentSchema)
    .mutation(async ({ input, ctx }) => {
      const { id, content } = input;
      const authorId = ctx.session.user.profileId;

      const existingComment = await commentService.getById(id);
      if (!existingComment || existingComment.authorId !== authorId) {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'You are not authorized to update this comment.',
        });
      }

      try {
        const updatedComment = await commentService.update(id, { content });
        return updatedComment;
      } catch (error) {
        console.error('Error updating comment:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to update comment.',
          cause: error,
        });
      }
    }),

  // Delete a comment
  delete: protectedProcedure
    .input(deleteCommentSchema)
    .mutation(async ({ input, ctx }) => {
      const { id } = input;
      const authorId = ctx.session.user.profileId;

      const existingComment = await commentService.getById(id);
      if (!existingComment) {
        throw new TRPCError({ code: 'NOT_FOUND', message: 'Comment not found.' });
      }

      // Only author or post author can delete
      const post = await prisma.post.findUnique({ where: { id: existingComment.postId }, select: { authorId: true } });
      if (existingComment.authorId !== authorId && post?.authorId !== authorId) {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'You are not authorized to delete this comment.',
        });
      }

      try {
        await commentService.delete(id);
        return { success: true };
      } catch (error) {
        console.error('Error deleting comment:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to delete comment.',
          cause: error,
        });
      }
    }),

  // Get a single comment by ID
  getById: publicProcedure
    .input(getCommentByIdSchema)
    .query(async ({ input }) => {
      const { id } = input;
      const comment = await commentService.getById(id);
      if (!comment) {
        throw new TRPCError({ code: 'NOT_FOUND', message: 'Comment not found.' });
      }
      return comment;
    }),

  // Get comments for a specific post (top-level)
  getByPost: publicProcedure
    .input(getCommentsByPostSchema)
    .query(async ({ input }) => {
      const { postId, page, pageSize, sortBy, sortOrder } = input;
      return commentService.getByPost(postId, { page, pageSize, sortBy, sortOrder });
    }),

  // Get replies for a specific comment
  getReplies: publicProcedure
    .input(getRepliesSchema)
    .query(async ({ input }) => {
      const { commentId, page, pageSize, sortBy, sortOrder } = input;
      return commentService.getReplies(commentId, { page, pageSize, sortBy, sortOrder });
    }),

  // Get threaded comments for a post (initial load)
  getThreaded: publicProcedure
    .input(z.object({ postId: z.string(), maxDepth: z.number().optional().default(2) }))
    .query(async ({ input }) => {
      const { postId, maxDepth } = input;
      return commentService.getThreadedComments(postId, maxDepth);
    }),

  // Like a comment
  like: protectedProcedure
    .input(likeCommentSchema)
    .mutation(async ({ input, ctx }) => {
      const { commentId } = input;
      const profileId = ctx.session.user.profileId;

      try {
        await commentService.like(commentId, profileId);
        return { success: true };
      } catch (error) {
        console.error('Error liking comment:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to like comment.',
          cause: error,
        });
      }
    }),

  // Unlike a comment
  unlike: protectedProcedure
    .input(likeCommentSchema)
    .mutation(async ({ input, ctx }) => {
      const { commentId } = input;
      const profileId = ctx.session.user.profileId;

      try {
        await commentService.unlike(commentId, profileId);
        return { success: true };
      } catch (error) {
        console.error('Error unliking comment:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to unlike comment.',
          cause: error,
        });
      }
    }),

  // Check if current user liked a comment
  isLiked: protectedProcedure
    .input(likeCommentSchema)
    .query(async ({ input, ctx }) => {
      const { commentId } = input;
      const profileId = ctx.session.user.profileId;
      return commentService.isLiked(commentId, profileId);
    }),

  // Pin a comment (only post author)
  pin: protectedProcedure
    .input(pinCommentSchema)
    .mutation(async ({ input, ctx }) => {
      const { commentId, postId } = input;
      const currentProfileId = ctx.session.user.profileId;

      const post = await prisma.post.findUnique({ where: { id: postId }, select: { authorId: true } });
      if (!post || post.authorId !== currentProfileId) {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'Only the post author can pin comments.',
        });
      }

      try {
        await commentService.pin(commentId, currentProfileId);
        return { success: true };
      } catch (error) {
        console.error('Error pinning comment:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to pin comment.',
          cause: error,
        });
      }
    }),

  // Unpin a comment (only post author)
  unpin: protectedProcedure
    .input(pinCommentSchema) // Reusing schema, only commentId is needed for unpin
    .mutation(async ({ input, ctx }) => {
      const { commentId, postId } = input; // postId is used for authorization check
      const currentProfileId = ctx.session.user.profileId;

      const post = await prisma.post.findUnique({ where: { id: postId }, select: { authorId: true } });
      if (!post || post.authorId !== currentProfileId) {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'Only the post author can unpin comments.',
        });
      }

      try {
        await commentService.unpin(commentId);
        return { success: true };
      } catch (error) {
        console.error('Error unpinning comment:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to unpin comment.',
          cause: error,
        });
      }
    }),
});
```

```typescript
// FILE 5: src/server/trpc/routers/notifications.ts
// ~150 righe
import { router, publicProcedure, protectedProcedure } from '../trpc'; // Assuming these are defined
import { z } from 'zod';
import { prisma } from '../../db'; // Assuming prisma client is initialized
import { NotificationService } from '../../services/notification-service';
import {
  getNotificationsSchema,
  markAsReadSchema,
  markAllAsReadSchema,
  deleteNotificationSchema,
  updateNotificationPreferencesSchema,
  registerPushTokenSchema,
} from '../../../lib/validations/notifications';
import { TRPCError } from '@trpc/server';

// Initialize NotificationService (without real-time/push for tRPC router directly,
// as these are often handled by separate infrastructure or client-side subscriptions)
const notificationService = new NotificationService(prisma);

export const notificationsRouter = router({
  // Get notifications for the current user
  getByRecipient: protectedProcedure
    .input(getNotificationsSchema)
    .query(async ({ input, ctx }) => {
      const recipientId = ctx.session.user.profileId;
      try {
        return await notificationService.getByRecipient(recipientId, input);
      } catch (error) {
        console.error('Error fetching notifications:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to fetch notifications.',
          cause: error,
        });
      }
    }),

  // Get unread notification count for the current user
  getUnreadCount: protectedProcedure
    .query(async ({ ctx }) => {
      const recipientId = ctx.session.user.profileId;
      try {
        return await notificationService.getUnreadCount(recipientId);
      } catch (error) {
        console.error('Error fetching unread notification count:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to fetch unread notification count.',
          cause: error,
        });
      }
    }),

  // Get recent unread notifications for the current user
  getUnread: protectedProcedure
    .input(z.object({ limit: z.number().int().min(1).max(20).optional().default(5) }))
    .query(async ({ input, ctx }) => {
      const recipientId = ctx.session.user.profileId;
      try {
        return await notificationService.getUnread(recipientId, input.limit);
      } catch (error) {
        console.error('Error fetching recent unread notifications:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to fetch recent unread notifications.',
          cause: error,
        });
      }
    }),

  // Mark a single notification as read
  markAsRead: protectedProcedure
    .input(markAsReadSchema)
    .mutation(async ({ input, ctx }) => {
      const { id } = input;
      const recipientId = ctx.session.user.profileId;

      const notification = await prisma.notification.findUnique({ where: { id } });
      if (!notification || notification.recipientId !== recipientId) {
        throw new TRPCError({ code: 'FORBIDDEN', message: 'Not authorized to mark this notification as read.' });
      }

      try {
        await notificationService.markAsRead(id);
        return { success: true };
      } catch (error) {
        console.error('Error marking notification as read:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to mark notification as read.',
          cause: error,
        });
      }
    }),

  // Mark all notifications for the current user as read
  markAllAsRead: protectedProcedure
    .input(markAllAsReadSchema.optional()) // Input is optional, but schema is defined
    .mutation(async ({ ctx }) => {
      const recipientId = ctx.session.user.profileId;
      try {
        const count = await notificationService.markAllAsRead(recipientId);
        return { success: true, count };
      } catch (error) {
        console.error('Error marking all notifications as read:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to mark all notifications as read.',
          cause: error,
        });
      }
    }),

  // Delete a single notification
  delete: protectedProcedure
    .input(deleteNotificationSchema)
    .mutation(async ({ input, ctx }) => {
      const { id } = input;
      const recipientId = ctx.session.user.profileId;

      const notification = await prisma.notification.findUnique({ where: { id } });
      if (!notification || notification.recipientId !== recipientId) {
        throw new TRPCError({ code: 'FORBIDDEN', message: 'Not authorized to delete this notification.' });
      }

      try {
        await notificationService.delete(id);
        return { success: true };
      } catch (error) {
        console.error('Error deleting notification:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to delete notification.',
          cause: error,
        });
      }
    }),

  // Get notification preferences for the current user
  getPreferences: protectedProcedure
    .query(async ({ ctx }) => {
      const profileId = ctx.session.user.profileId;
      try {
        return await notificationService.getPreferences(profileId);
      } catch (error) {
        console.error('Error fetching notification preferences:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to fetch notification preferences.',
          cause: error,
        });
      }
    }),

  // Update notification preferences for the current user
  updatePreferences: protectedProcedure
    .input(updateNotificationPreferencesSchema)
    .mutation(async ({ input, ctx }) => {
      const profileId = ctx.session.user.profileId;
      try {
        return await notificationService.updatePreferences(profileId, input);
      } catch (error) {
        console.error('Error updating notification preferences:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to update notification preferences.',
          cause: error,
        });
      }
    }),

  // Register a push notification token
  registerPushToken: protectedProcedure
    .input(registerPushTokenSchema)
    .mutation(async ({ input, ctx }) => {
      const profileId = ctx.session.user.profileId;
      try {
        const subscription = await notificationService.registerPushToken(profileId, input.token, input.platform, input.deviceInfo);
        return subscription;
      } catch (error) {
        console.error('Error registering push token:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to register push token.',
          cause: error,
        });
      }
    }),
});
```

```typescript
// FILE 6: src/lib/validations/comments.ts
// ~60 righe
import { z } from 'zod';

// Common pagination and sorting parameters
export const listParamsSchema = z.object({
  page: z.number().int().min(1).optional().default(1),
  pageSize: z.number().int().min(1).max(100).optional().default(10),
  sortBy: z.string().optional().default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).optional().default('desc'),
});

export type ListParams = z.infer<typeof listParamsSchema>;

// Input for creating a comment
export const createCommentSchema = z.object({
  postId: z.string().cuid(),
  content: z.string().min(1).max(1000),
  parentId: z.string().cuid().optional(), // For replies
});

export type CreateCommentInput = z.infer<typeof createCommentSchema>;

// Input for updating a comment
export const updateCommentSchema = z.object({
  id: z.string().cuid(),
  content: z.string().min(1).max(1000),
});

export type UpdateCommentInput = z.infer<typeof updateCommentSchema>;

// Input for deleting a comment
export const deleteCommentSchema = z.object({
  id: z.string().cuid(),
});

// Input for getting a single comment
export const getCommentByIdSchema = z.object({
  id: z.string().cuid(),
});

// Input for getting comments by post
export const getCommentsByPostSchema = listParamsSchema.extend({
  postId: z.string().cuid(),
  // Specific sorting for comments on a post, e.g., pinned first
  sortBy: z.enum(['createdAt', 'likesCount']).optional().default('createdAt'),
});

export type CommentListParams = z.infer<typeof getCommentsByPostSchema>;

// Input for getting replies to a comment
export const getRepliesSchema = listParamsSchema.extend({
  commentId: z.string().cuid(),
  sortBy: z.enum(['createdAt', 'likesCount']).optional().default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).optional().default('asc'), // Replies usually ascend
});

// Input for liking/unliking a comment
export const likeCommentSchema = z.object({
  commentId: z.string().cuid(),
});

// Input for pinning/unpinning a comment
export const pinCommentSchema = z.object({
  commentId: z.string().cuid(),
  postId: z.string().cuid(), // Required for authorization check
});
```

```typescript
// FILE 7: src/lib/validations/notifications.ts
// ~50 righe
import { z } from 'zod';
import { NotificationType } from '@prisma/client';
import { listParamsSchema } from './comments'; // Reusing listParamsSchema

// Input for creating a notification
export const createNotificationSchema = z.object({
  recipientId: z.string().cuid(),
  type: z.nativeEnum(NotificationType),
  actorId: z.string().cuid().optional(),
  postId: z.string().cuid().optional(),
  commentId: z.string().cuid().optional(),
  followId: z.string().cuid().optional(),
  title: z.string().max(255).optional(),
  body: z.string().max(1000).optional(),
  imageUrl: z.string().url().optional(),
  actionUrl: z.string().url().optional(),
});

export type CreateNotificationInput = z.infer<typeof createNotificationSchema>;

// Input for getting notifications for a recipient
export const getNotificationsSchema = listParamsSchema.extend({
  isRead: z.boolean().optional(),
  type: z.nativeEnum(NotificationType).optional(),
});

export type NotificationListParams = z.infer<typeof getNotificationsSchema>;

// Input for marking a notification as read
export const markAsReadSchema = z.object({
  id: z.string().cuid(),
});

// Input for marking all notifications as read (no specific input needed, but good to have a schema)
export const markAllAsReadSchema = z.object({});

// Input for deleting a notification
export const deleteNotificationSchema = z.object({
  id: z.string().cuid(),
});

// Input for updating notification preferences
export const updateNotificationPreferencesSchema = z.object({
  email_newFollower: z.boolean().optional(),
  email_postLike: z.boolean().optional(),
  email_comment: z.boolean().optional(),
  email_reply: z.boolean().optional(),
  email_mention: z.boolean().optional(),
  email_system: z.boolean().optional(),
  inApp_newFollower: z.boolean().optional(),
  inApp_postLike: z.boolean().optional(),
  inApp_comment: z.boolean().optional(),
  inApp_reply: z.boolean().optional(),
  inApp_mention: z.boolean().optional(),
  inApp_system: z.boolean().optional(),
  push_newFollower: z.boolean().optional(),
  push_postLike: z.boolean().optional(),
  push_comment: z.boolean().optional(),
  push_reply: z.boolean().optional(),
  push_mention: z.boolean().optional(),
  push_system: z.boolean().optional(),
});

export type UpdateNotificationPreferencesInput = z.infer<typeof updateNotificationPreferencesSchema>;

// Input for registering a push notification token
export const registerPushTokenSchema = z.object({
  token: z.string().min(1),
  platform: z.enum(['web', 'ios', 'android']),
  deviceInfo: z.record(z.string(), z.any()).optional(), // Flexible JSON object
});
```

```typescript
// FILE 8: src/hooks/use-comments.ts
// ~100 righe
import { useState, useEffect, useCallback, useMemo } from 'react';
import { api } from '../utils/api'; // Assuming tRPC client setup
import { CommentThread, CommentWithAuthor } from '../server/services/comment-service';
import { PaginatedResult } from '../server/utils/pagination'; // Assuming PaginatedResult type

interface UseCommentsResult {
  comments: CommentThread[];
  isLoading: boolean;
  isFetchingNextPage: boolean;
  hasNextPage: boolean;
  error: unknown;
  addComment: (content: string, parentId?: string) => Promise<void>;
  updateComment: (commentId: string, content: string) => Promise<void>;
  deleteComment: (commentId: string) => Promise<void>;
  likeComment: (commentId: string) => Promise<void>;
  unlikeComment: (commentId: string) => Promise<void>;
  loadMoreComments: () => Promise<void>;
  loadMoreReplies: (commentId: string) => Promise<void>;
  refetchComments: () => Promise<void>;
}

export function useComments(postId: string): UseCommentsResult {
  const [comments, setComments] = useState<CommentThread[]>([]);
  const [page, setPage] = useState(1);
  const [hasMoreComments, setHasMoreComments] = useState(true);
  const [isFetchingNextPage, setIsFetchingNextPage] = useState(false);

  // tRPC queries and mutations
  const { data, isLoading, error, refetch } = api.comments.getByPost.useQuery(
    { postId, page, pageSize: 10 },
    {
      enabled: !!postId && hasMoreComments,
      keepPreviousData: true,
      onSuccess: (newData) => {
        if (page === 1) {
          setComments(newData.data.map(c => ({ ...c, replies: [] }))); // Initialize replies as empty
        } else {
          setComments(prev => [...prev, ...newData.data.map(c => ({ ...c, replies: [] }))]);
        }
        setHasMoreComments(newData.page < newData.totalPages);
        setIsFetchingNextPage(false);
      },
      onError: (err) => {
        console.error("Error fetching comments:", err);
        setIsFetchingNextPage(false);
      }
    }
  );

  const createCommentMutation = api.comments.create.useMutation({
    onSuccess: (newComment) => {
      // For simplicity, refetch all comments to ensure correct threading/sorting
      // In a real app, you might optimistically update or insert at the correct position
      refetch();
    },
  });

  const updateCommentMutation = api.comments.update.useMutation({
    onSuccess: () => refetch(),
  });

  const deleteCommentMutation = api.comments.delete.useMutation({
    onSuccess: () => refetch(),
  });

  const likeCommentMutation = api.comments.like.useMutation({
    onSuccess: () => refetch(), // Refetch to update like counts/status
  });

  const unlikeCommentMutation = api.comments.unlike.useMutation({
    onSuccess: () => refetch(), // Refetch to update like counts/status
  });

  const getRepliesQuery = api.comments.getReplies.useQuery; // Use as a function

  const addComment = useCallback(async (content: string, parentId?: string) => {
    await createCommentMutation.mutateAsync({ postId, content, parentId });
  }, [createCommentMutation, postId]);

  const updateComment = useCallback(async (commentId: string, content: string) => {
    await updateCommentMutation.mutateAsync({ id: commentId, content });
  }, [updateCommentMutation]);

  const deleteComment = useCallback(async (commentId: string) => {
    await deleteCommentMutation.mutateAsync({ id: commentId });
  }, [deleteCommentMutation]);

  const likeComment = useCallback(async (commentId: string) => {
    await likeCommentMutation.mutateAsync({ commentId });
  }, [likeCommentMutation]);

  const unlikeComment = useCallback(async (commentId: string) => {
    await unlikeCommentMutation.mutateAsync({ commentId });
  }, [unlikeCommentMutation]);

  const loadMoreComments = useCallback(async () => {
    if (hasMoreComments && !isFetchingNextPage) {
      setIsFetchingNextPage(true);
      setPage(prev => prev + 1);
    }
  }, [hasMoreComments, isFetchingNextPage]);

  const loadMoreReplies = useCallback(async (commentId: string) => {
    // This is a simplified approach. A more robust solution would manage reply pagination per comment.
    // For now, we'll just fetch the next batch of replies and append them.
    // This would typically involve a separate state for each comment's replies.
    // For this example, we'll just refetch the main comments to simulate an update.
    // In a real app, you'd have a `useReplies` hook or similar.
    console.log(`Loading more replies for comment ${commentId}... (simulated refetch)`);
    // A more complex state management would be needed here to update a specific comment's replies array
    // For now, we'll just refetch the main comments to trigger a UI update.
    refetch();
  }, [refetch]);

  const refetchComments = useCallback(async () => {
    setPage(1); // Reset page to 1 for a full refetch
    setComments([]); // Clear existing comments
    setHasMoreComments(true); // Assume there are more comments
    await refetch();
  }, [refetch]);

  return {
    comments,
    isLoading: isLoading && !isFetchingNextPage, // Only show loading for initial fetch
    isFetchingNextPage,
    hasNextPage: hasMoreComments,
    error,
    addComment,
    updateComment,
    deleteComment,
    likeComment,
    unlikeComment,
    loadMoreComments,
    loadMoreReplies,
    refetchComments,
  };
}
```

```typescript
// FILE 9: src/hooks/use-notifications.ts
// ~100 righe
import { useState, useEffect, useCallback } from 'react';
import { api } from '../utils/api'; // Assuming tRPC client setup
import { Notification } from '@prisma/client';
import { PaginatedResult } from '../server/utils/pagination'; // Assuming PaginatedResult type

interface NotificationWithRelations extends Notification {
  actor: { id: string; name: string; image?: string } | null;
  post: { id: string; title?: string } | null;
  comment: { id: string; content: string } | null;
}

interface UseNotificationsResult {
  notifications: NotificationWithRelations[];
  unreadCount: number;
  isLoading: boolean;
  isFetchingNextPage: boolean;
  hasNextPage: boolean;
  error: unknown;
  markAsRead: (notificationId: string) => Promise<void>;
  markAllAsRead: () => Promise<void>;
  deleteNotification: (notificationId: string) => Promise<void>;
  loadMoreNotifications: () => Promise<void>;
  refetchNotifications: () => Promise<void>;
}

export function useNotifications(): UseNotificationsResult {
  const [notifications, setNotifications] = useState<NotificationWithRelations[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);
  const [page, setPage] = useState(1);
  const [hasMoreNotifications, setHasMoreNotifications] = useState(true);
  const [isFetchingNextPage, setIsFetchingNextPage] = useState(false);

  // tRPC queries
  const { data, isLoading, error, refetch } = api.notifications.getByRecipient.useQuery(
    { page, pageSize: 10 },
    {
      enabled: hasMoreNotifications,
      keepPreviousData: true,
      onSuccess: (newData) => {
        if (page === 1) {
          setNotifications(newData.data);
        } else {
          setNotifications(prev => [...prev, ...newData.data]);
        }
        setHasMoreNotifications(newData.page < newData.totalPages);
        setIsFetchingNextPage(false);
      },
      onError: (err) => {
        console.error("Error fetching notifications:", err);
        setIsFetchingNextPage(false);
      }
    }
  );

  const { data: unreadCountData, refetch: refetchUnreadCount } = api.notifications.getUnreadCount.useQuery(
    undefined,
    {
      onSuccess: (count) => setUnreadCount(count),
    }
  );

  // tRPC mutations
  const markAsReadMutation = api.notifications.markAsRead.useMutation({
    onSuccess: () => {
      refetch();
      refetchUnreadCount();
    },
  });

  const markAllAsReadMutation = api.notifications.markAllAsRead.useMutation({
    onSuccess: () => {
      refetch();
      refetchUnreadCount();
    },
  });

  const deleteNotificationMutation = api.notifications.delete.useMutation({
    onSuccess: () => {
      refetch();
      refetchUnreadCount();
    },
  });

  // Actions
  const markAsRead = useCallback(async (notificationId: string) => {
    await markAsReadMutation.mutateAsync({ id: notificationId });
  }, [markAsReadMutation]);

  const markAllAsRead = useCallback(async () => {
    await markAllAsReadMutation.mutateAsync();
  }, [markAllAsReadMutation]);

  const deleteNotification = useCallback(async (notificationId: string) => {
    await deleteNotificationMutation.mutateAsync({ id: notificationId });
  }, [deleteNotificationMutation]);

  const loadMoreNotifications = useCallback(async () => {
    if (hasMoreNotifications && !isFetchingNextPage) {
      setIsFetchingNextPage(true);
      setPage(prev => prev + 1);
    }
  }, [hasMoreNotifications, isFetchingNextPage]);

  const refetchNotifications = useCallback(async () => {
    setPage(1); // Reset page to 1 for a full refetch
    setNotifications([]); // Clear existing notifications
    setHasMoreNotifications(true); // Assume there are more notifications
    await refetch();
    await refetchUnreadCount();
  }, [refetch, refetchUnreadCount]);

  // Real-time subscription (conceptual)
  // useEffect(() => {
  //   // Assuming a real-time client is available and `api.notifications.subscribe` is a tRPC subscription
  //   // This would typically be handled by a separate WebSocket client or tRPC's `useSubscription`
  //   const unsubscribe = api.notifications.subscribe.useSubscription(undefined, {
  //     onData: (newNotification) => {
  //       setNotifications(prev => [newNotification, ...prev]);
  //       setUnreadCount(prev => prev + 1);
  //     },
  //     onError: (err) => {
  //       console.error("Error with notification subscription:", err);
  //     }
  //   });
  //   return () => unsubscribe.unsubscribe();
  // }, []);

  return {
    notifications,
    unreadCount,
    isLoading: isLoading && !isFetchingNextPage,
    isFetchingNextPage,
    hasNextPage: hasMoreNotifications,
    error,
    markAsRead,
    markAllAsRead,
    deleteNotification,
    loadMoreNotifications,
    refetchNotifications,
  };
}
```

```typescript
// FILE 10: src/components/social/comments/comment-list.tsx
// ~150 righe
import React from 'react';
import { useComments } from '../../../hooks/use-comments';
import { CommentItem } from './comment-item';
import { CommentForm } from './comment-form';
import { CommentThread } from '../../../server/services/comment-service'; // Assuming this type is exported

interface CommentListProps {
  postId: string;
}

export const CommentList: React.FC<CommentListProps> = ({ postId }) => {
  const {
    comments,
    isLoading,
    isFetchingNextPage,
    hasNextPage,
    error,
    addComment,
    loadMoreComments,
    updateComment,
    deleteComment,
    likeComment,
    unlikeComment,
    loadMoreReplies, // Passed down to CommentItem
  } = useComments(postId);

  if (isLoading && comments.length === 0) {
    return <div className="text-center py-4 text-gray-500">Loading comments...</div>;
  }

  if (error) {
    return <div className="text-center py-4 text-red-500">Error loading comments: {(error as Error).message}</div>;
  }

  const handleAddComment = async (content: string) => {
    await addComment(content);
  };

  return (
    <div className="space-y-6">
      <h2 className="text-xl font-bold text-gray-800 dark:text-gray-100">Comments</h2>

      <CommentForm postId={postId} onSubmit={handleAddComment} />

      {comments.length === 0 && !isLoading ? (
        <p className="text-center text-gray-500 dark:text-gray-400">No comments yet. Be the first to comment!</p>
      ) : (
        <div className="space-y-4">
          {comments.map((comment) => (
            <CommentItem
              key={comment.id}
              comment={comment}
              postId={postId}
              onReply={addComment}
              onUpdate={updateComment}
              onDelete={deleteComment}
              onLike={likeComment}
              onUnlike={unlikeComment}
              onLoadMoreReplies={loadMoreReplies}
              currentUserId="user-123" // Placeholder for current logged-in user ID
              postAuthorId="post-author-456" // Placeholder for post author ID
            />
          ))}

          {hasNextPage && (
            <div className="text-center mt-4">
              <button
                onClick={loadMoreComments}
                disabled={isFetchingNextPage}
                className="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 disabled:opacity-50"
              >
                {isFetchingNextPage ? 'Loading more...' : 'Load More Comments'}
              </button>
            </div>
          )}
        </div>
      )}
    </div>
  );
};
```

```typescript
// FILE 11: src/components/social/comments/comment-item.tsx
// ~150 righe
import React, { useState } from 'react';
import { CommentThread } from '../../../server/services/comment-service';
import { formatDistanceToNowStrict } from 'date-fns';
import { Avatar, AvatarFallback, AvatarImage } from '../../ui/avatar'; // Assuming a UI library for Avatar
import { Button } from '../../ui/button'; // Assuming a UI library for Button
import { Textarea } from '../../ui/textarea'; // Assuming a UI library for Textarea
import { LikeIcon, ReplyIcon, EditIcon, TrashIcon, PinIcon } from '../../ui/icons'; // Assuming UI icons

interface CommentItemProps {
  comment: CommentThread;
  postId: string;
  onReply: (content: string, parentId: string) => Promise<void>;
  onUpdate: (commentId: string, content: string) => Promise<void>;
  onDelete: (commentId: string) => Promise<void>;
  onLike: (commentId: string) => Promise<void>;
  onUnlike: (commentId: string) => Promise<void>;
  onLoadMoreReplies: (commentId: string) => Promise<void>;
  currentUserId: string; // ID of the currently logged-in user
  postAuthorId: string; // ID of the post's author
  depth?: number; // Current nesting depth
}

export const CommentItem: React.FC<CommentItemProps> = ({
  comment,
  postId,
  onReply,
  onUpdate,
  onDelete,
  onLike,
  onUnlike,
  onLoadMoreReplies,
  currentUserId,
  postAuthorId,
  depth = 0,
}) => {
  const [showReplyForm, setShowReplyForm] = useState(false);
  const [isEditing, setIsEditing] = useState(false);
  const [editedContent, setEditedContent] = useState(comment.content);
  const [showReplies, setShowReplies] = useState(false); // State to toggle replies visibility

  const isAuthor = comment.author.id === currentUserId;
  const isPostAuthor = postAuthorId === currentUserId;
  const hasLiked = comment.likes.some(like => like.profileId === currentUserId);

  const handleReplySubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const form = e.target as HTMLFormElement;
    const content = (form.elements.namedItem('replyContent') as HTMLTextAreaElement).value;
    if (content.trim()) {
      await onReply(content, comment.id);
      setShowReplyForm(false);
      form.reset();
    }
  };

  const handleEditSubmit = async () => {
    if (editedContent.trim() && editedContent !== comment.content) {
      await onUpdate(comment.id, editedContent);
      setIsEditing(false);
    } else {
      setIsEditing(false); // Cancel edit if no change or empty
    }
  };

  const handleLikeToggle = async () => {
    if (hasLiked) {
      await onUnlike(comment.id);
    } else {
      await onLike(comment.id);
    }
  };

  const indentation = depth * 1.5; // Adjust as needed for visual nesting

  return (
    <div className={`flex gap-3 ${depth > 0 ? 'ml-6 border-l pl-4 border-gray-200 dark:border-gray-700' : ''}`}>
      <Avatar className="h-8 w-8">
        <AvatarImage src={comment.author.image || '/default-avatar.png'} alt={comment.author.name} />
        <AvatarFallback>{comment.author.name[0]}</AvatarFallback>
      </Avatar>
      <div className="flex-1">
        <div className="flex items-center gap-2 text-sm">
          <span className="font-semibold text-gray-900 dark:text-gray-50">{comment.author.name}</span>
          <span className="text-gray-500 dark:text-gray-400">
            {formatDistanceToNowStrict(new Date(comment.createdAt), { addSuffix: true })}
          </span>
          {comment.isEdited && <span className="text-gray-400 text-xs">(Edited)</span>}
          {comment.isPinned && isPostAuthor && (
            <span className="text-blue-500 text-xs flex items-center gap-1">
              <PinIcon className="h-3 w-3" /> Pinned
            </span>
          )}
        </div>
        {isEditing ? (
          <div className="mt-1">
            <Textarea
              value={editedContent}
              onChange={(e) => setEditedContent(e.target.value)}
              className="w-full text-sm"
            />
            <div className="flex gap-2 mt-2">
              <Button size="sm" onClick={handleEditSubmit}>Save</Button>
              <Button size="sm" variant="ghost" onClick={() => setIsEditing(false)}>Cancel</Button>
            </div>
          </div>
        ) : (
          <p className="text-gray-800 dark:text-gray-200 mt-1 text-sm">{comment.content}</p>
        )}

        <div className="flex items-center gap-3 mt-2 text-xs text-gray-500 dark:text-gray-400">
          <Button variant="ghost" size="sm" onClick={handleLikeToggle} className="flex items-center gap-1">
            <LikeIcon className={`h-4 w-4 ${hasLiked ? 'text-red-500 fill-current' : ''}`} />
            {comment.likesCount > 0 && <span>{comment.likesCount}</span>} Like
          </Button>
          <Button variant="ghost" size="sm" onClick={() => setShowReplyForm(!showReplyForm)} className="flex items-center gap-1">
            <ReplyIcon className="h-4 w-4" /> Reply
          </Button>

          {isAuthor && (
            <>
              <Button variant="ghost" size="sm" onClick={() => setIsEditing(!isEditing)} className="flex items-center gap-1">
                <EditIcon className="h-4 w-4" /> Edit
              </Button>
              <Button variant="ghost" size="sm" onClick={() => onDelete(comment.id)} className="flex items-center gap-1 text-red-500">
                <TrashIcon className="h-4 w-4" /> Delete
              </Button>
            </>
          )}
          {/* Post author specific actions, e.g., pin/unpin */}
          {isPostAuthor && (
            <Button variant="ghost" size="sm" onClick={() => {/* handle pin/unpin */}} className="flex items-center gap-1">
              <PinIcon className="h-4 w-4" /> {comment.isPinned ? 'Unpin' : 'Pin'}
            </Button>
          )}
        </div>

        {showReplyForm && (
          <form onSubmit={handleReplySubmit} className="mt-3">
            <Textarea
              name="replyContent"
              placeholder={`Reply to ${comment.author.name}...`}
              className="w-full text-sm"
              rows={2}
            />
            <div className="flex justify-end gap-2 mt-2">
              <Button type="submit" size="sm">Reply</Button>
              <Button type="button" size="sm" variant="ghost" onClick={() => setShowReplyForm(false)}>Cancel</Button>
            </div>
          </form>
        )}

        {comment.repliesCount > 0 && (
          <div className="mt-3">
            <Button variant="link" size="sm" onClick={() => setShowReplies(!showReplies)} className="text-blue-500 dark:text-blue-400">
              {showReplies ? `Hide replies (${comment.repliesCount})` : `View ${comment.repliesCount} replies`}
            </Button>
          </div>
        )}

        {showReplies && comment.replies.length > 0 && (
          <div className="mt-4 space-y-4">
            {comment.replies.map((reply) => (
              <CommentItem
                key={reply.id}
                comment={reply}
                postId={postId}
                onReply={onReply}
                onUpdate={onUpdate}
                onDelete={onDelete}
                onLike={onLike}
                onUnlike={onUnlike}
                onLoadMoreReplies={onLoadMoreReplies}
                currentUserId={currentUserId}
                postAuthorId={postAuthorId}
                depth={depth + 1}
              />
            ))}
            {/* Implement "Load More Replies" if comment.replies is not exhaustive */}
            {/* For now, assuming `comment.replies` contains all loaded replies up to maxDepth */}
            {/* A more advanced implementation would have a `hasMoreReplies` state per comment */}
            {comment.repliesCount > comment.replies.length && (
              <Button variant="link" size="sm" onClick={() => onLoadMoreReplies(comment.id)} className="text-blue-500 dark:text-blue-400 ml-6">
                Load more replies...
              </Button>
            )}
          </div>
        )}
      </div>
    </div>
  );
};
```

```typescript
// FILE 12: src/components/social/comments/comment-form.tsx
// ~100 righe
import React, { useState } from 'react';
import { Button } from '../../ui/button'; // Assuming a UI library for Button
import { Textarea } from '../../ui/textarea'; // Assuming a UI library for Textarea
import { Avatar, AvatarFallback, AvatarImage } from '../../ui/avatar'; // Assuming a UI library for Avatar
import { useSession } from 'next-auth/react'; // Assuming NextAuth.js for session

interface CommentFormProps {
  postId: string;
  parentId?: string; // Optional, for replying to a specific comment
  onSubmit: (content: string, parentId?: string) => Promise<void>;
  onCancel?: () => void; // Optional, for reply forms to cancel
  placeholder?: string;
  submitButtonText?: string;
}

export const CommentForm: React.FC<CommentFormProps> = ({
  postId,
  parentId,
  onSubmit,
  onCancel,
  placeholder = 'Write a comment...',
  submitButtonText = 'Comment',
}) => {
  const { data: session } = useSession();
  const [content, setContent] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!session?.user?.profileId) {
      setError('You must be logged in to comment.');
      return;
    }
    if (content.trim().length === 0) {
      setError('Comment cannot be empty.');
      return;
    }

    setIsSubmitting(true);
    setError(null);
    try {
      await onSubmit(content, parentId);
      setContent('');
      if (onCancel) onCancel(); // Close form if it's a reply form
    } catch (err) {
      console.error('Failed to submit comment:', err);
      setError('Failed to submit comment. Please try again.');
    } finally {
      setIsSubmitting(false);
    }
  };

  const userAvatar = session?.user?.image || '/default-avatar.png';
  const userNameInitial = session?.user?.name ? session.user.name[0] : '?';

  return (
    <form onSubmit={handleSubmit} className="flex gap-3 p-4 bg-white dark:bg-gray-800 rounded-lg shadow-sm">
      <Avatar className="h-9 w-9">
        <AvatarImage src={userAvatar} alt={session?.user?.name || 'User'} />
        <AvatarFallback>{userNameInitial}</AvatarFallback>
      </Avatar>
      <div className="flex-1">
        <Textarea
          value={content}
          onChange={(e) => setContent(e.target.value)}
          placeholder={placeholder}
          rows={parentId ? 2 : 3}
          className="w-full resize-none border-gray-300 dark:border-gray-600 dark:bg-gray-700 dark:text-gray-100 focus:ring-blue-500 focus:border-blue-500"
          disabled={isSubmitting || !session?.user?.profileId}
        />
        {error && <p className="text-red-500 text-sm mt-1">{error}</p>}
        <div className="flex justify-end gap-2 mt-3">
          {onCancel && (
            <Button type="button" variant="ghost" onClick={onCancel} disabled={isSubmitting}>
              Cancel
            </Button>
          )}
          <Button type="submit" disabled={isSubmitting || content.trim().length === 0 || !session?.user?.profileId}>
            {isSubmitting ? 'Submitting...' : submitButtonText}
          </Button>
        </div>
        {!session?.user?.profileId && (
          <p className="text-sm text-gray-500 dark:text-gray-400 mt-2">
            Please log in to {parentId ? 'reply' : 'comment'}.
          </p>
        )}
      </div>
    </form>
  );
};
```

```typescript
// FILE 13: src/components/social/notifications/notification-list.tsx
// ~120 righe
import React, { useEffect, useRef, useCallback } from 'react';
import { useNotifications } from '../../../hooks/use-notifications';
import { NotificationItem } from './notification-item';
import { Button } from '../../ui/button'; // Assuming a UI library for Button
import { Loader2 } from 'lucide-react'; // Assuming lucide-react for icons

interface NotificationListProps {
  // Can accept filters if needed, e.g., filterType: NotificationType | 'all'
}

export const NotificationList: React.FC<NotificationListProps> = () => {
  const {
    notifications,
    isLoading,
    isFetchingNextPage,
    hasNextPage,
    error,
    markAsRead,
    markAllAsRead,
    deleteNotification,
    loadMoreNotifications,
    refetchNotifications,
  } = useNotifications();

  const observerTarget = useRef(null);

  const handleObserver = useCallback((entries: IntersectionObserverEntry[]) => {
    const target = entries[0];
    if (target.isIntersecting && hasNextPage && !isFetchingNextPage) {
      loadMoreNotifications();
    }
  }, [hasNextPage, isFetchingNextPage, loadMoreNotifications]);

  useEffect(() => {
    const observer = new IntersectionObserver(handleObserver, {
      root: null, // viewport
      rootMargin: '0px',
      threshold: 1.0,
    });

    if (observerTarget.current) {
      observer.observe(observerTarget.current);
    }

    return () => {
      if (observerTarget.current) {
        observer.unobserve(observerTarget.current);
      }
    };
  }, [observerTarget, handleObserver]);

  if (isLoading && notifications.length === 0) {
    return (
      <div className="flex justify-center items-center h-40 text-gray-500 dark:text-gray-400">
        <Loader2 className="h-6 w-6 animate-spin mr-2" /> Loading notifications...
      </div>
    );
  }

  if (error) {
    return <div className="text-center py-4 text-red-500">Error loading notifications: {(error as Error).message}</div>;
  }

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center pb-4 border-b border-gray-200 dark:border-gray-700">
        <h1 className="text-2xl font-bold text-gray-900 dark:text-gray-100">Notifications</h1>
        {notifications.length > 0 && (
          <Button variant="outline" onClick={markAllAsRead} disabled={isLoading || isFetchingNextPage}>
            Mark all as read
          </Button>
        )}
      </div>

      {notifications.length === 0 && !isLoading ? (
        <p className="text-center text-gray-500 dark:text-gray-400 py-8">You don't have any notifications yet.</p>
      ) : (
        <div className="divide-y divide-gray-200 dark:divide-gray-700">
          {notifications.map((notification) => (
            <NotificationItem
              key={notification.id}
              notification={notification}
              onMarkAsRead={markAsRead}
              onDelete={deleteNotification}
            />
          ))}
        </div>
      )}

      {hasNextPage && (
        <div ref={observerTarget} className="flex justify-center py-4">
          {isFetchingNextPage ? (
            <Loader2 className="h-6 w-6 animate-spin text-blue-500" />
          ) : (
            <Button onClick={loadMoreNotifications} variant="ghost" disabled={isFetchingNextPage}>
              Load More
            </Button>
          )}
        </div>
      )}
    </div>
  );
};
```

```typescript
// FILE 14: src/components/social/notifications/notification-item.tsx
// ~100 righe
import React from 'react';
import Link from 'next/link';
import { Notification, NotificationType } from '@prisma/client';
import { formatDistanceToNowStrict } from 'date-fns';
import {
  HeartIcon, UserPlusIcon, MessageSquareIcon, AtSignIcon, Repeat2Icon, InfoIcon, CheckCircleIcon, XCircleIcon
} from 'lucide-react'; // Assuming lucide-react for icons
import { Avatar, AvatarFallback, AvatarImage } from '../../ui/avatar'; // Assuming a UI library for Avatar
import { Button } from '../../ui/button'; // Assuming a UI library for Button

interface NotificationWithRelations extends Notification {
  actor: { id: string; name: string; image?: string } | null;
  post: { id: string; title?: string } | null;
  comment: { id: string; content: string } | null;
}

interface NotificationItemProps {
  notification: NotificationWithRelations;
  onMarkAsRead: (notificationId: string) => Promise<void>;
  onDelete: (notificationId: string) => Promise<void>;
}

const getNotificationIcon = (type: NotificationType) => {
  switch (type) {
    case NotificationType.FOLLOW:
    case NotificationType.FOLLOW_REQUEST:
    case NotificationType.FOLLOW_ACCEPTED:
      return <UserPlusIcon className="h-5 w-5 text-blue-500" />;
    case NotificationType.LIKE_POST:
    case NotificationType.LIKE_COMMENT:
      return <HeartIcon className="h-5 w-5 text-red-500" />;
    case NotificationType.COMMENT:
    case NotificationType.REPLY:
      return <MessageSquareIcon className="h-5 w-5 text-green-500" />;
    case NotificationType.MENTION:
      return <AtSignIcon className="h-5 w-5 text-purple-500" />;
    case NotificationType.REPOST:
      return <Repeat2Icon className="h-5 w-5 text-yellow-500" />;
    case NotificationType.SYSTEM:
      return <InfoIcon className="h-5 w-5 text-gray-500" />;
    default:
      return <InfoIcon className="h-5 w-5 text-gray-500" />;
  }
};

const getNotificationText = (notification: NotificationWithRelations) => {
  const { type, actor, post, comment, title, body } = notification;
  const actorName = actor?.name || 'Someone';
  const postTitle = post?.title || 'a post';
  const commentContent = comment?.content || 'a comment';

  if (title) return title; // Use custom title if provided

  switch (type) {
    case NotificationType.FOLLOW:
      return `${actorName} started following you.`;
    case NotificationType.FOLLOW_REQUEST:
      return `${actorName} sent you a follow request.`;
    case NotificationType.FOLLOW_ACCEPTED:
      return `${actorName} accepted your follow request.`;
    case NotificationType.LIKE_POST:
      return `${actorName} liked your post "${postTitle}".`;
    case NotificationType.LIKE_COMMENT:
      return `${actorName} liked your comment "${commentContent.substring(0, 30)}...".`;
    case NotificationType.COMMENT:
      return `${actorName} commented on your post "${postTitle}".`;
    case NotificationType.REPLY:
      return `${actorName} replied to your comment "${commentContent.substring(0, 30)}...".`;
    case NotificationType.MENTION:
      return `${actorName} mentioned you in ${comment ? 'a comment' : 'a post'}.`;
    case NotificationType.REPOST:
      return `${actorName} reposted your post "${postTitle}".`;
    case NotificationType.SYSTEM:
      return body || 'You have a system notification.';
    default:
      return 'You have a new notification.';
  }
};

export const NotificationItem: React.FC<NotificationItemProps> = ({ notification, onMarkAsRead, onDelete }) => {
  const notificationText = getNotificationText(notification);
  const notificationIcon = getNotificationIcon(notification.type);
  const actionUrl = notification.actionUrl || `/notifications`;

  const handleItemClick = async () => {
    if (!notification.isRead) {
      await onMarkAsRead(notification.id);
    }
  };

  return (
    <div
      className={`flex items-start gap-4 p-4 rounded-lg transition-colors duration-200
        ${notification.isRead ? 'bg-gray-50 dark:bg-gray-800' : 'bg-blue-50 dark:bg-blue-950 hover:bg-blue-100 dark:hover:bg-blue-900'}
      `}
    >
      <div className="flex-shrink-0">
        {notification.actor?.image ? (
          <Avatar className="h-9 w-9">
            <AvatarImage src={notification.actor.image} alt={notification.actor.name || 'Actor'} />
            <AvatarFallback>{notification.actor.name?.[0] || '?'}</AvatarFallback>
          </Avatar>
        ) : (
          <div className="h-9 w-9 flex items-center justify-center rounded-full bg-gray-200 dark:bg-gray-700">
            {notificationIcon}
          </div>
        )}
      </div>
      <Link href={actionUrl} className="flex-1 cursor-pointer" onClick={handleItemClick}>
        <div className="text-sm text-gray-800 dark:text-gray-200">
          {notificationText}
        </div>
        <div className="text-xs text-gray-500 dark:text-gray-400 mt-1">
          {formatDistanceToNowStrict(new Date(notification.createdAt), { addSuffix: true })}
        </div>
      </Link>
      <div className="flex-shrink-0 flex gap-2 items-center">
        {!notification.isRead && (
          <Button
            variant="ghost"
            size="icon"
            onClick={() => onMarkAsRead(notification.id)}
            title="Mark as read"
          >
            <CheckCircleIcon className="h-4 w-4 text-green-500" />
          </Button>
        )}
        <Button
          variant="ghost"
          size="icon"
          onClick={() => onDelete(notification.id)}
          title="Delete notification"
        >
          <XCircleIcon className="h-4 w-4 text-red-500" />
        </Button>
      </div>
    </div>
  );
};
```

```typescript
// FILE 15: src/components/social/notifications/notification-bell.tsx
// ~80 righe
import React, { useState, useEffect, useRef } from 'react';
import Link from 'next/link';
import { BellIcon, Loader2 } from 'lucide-react'; // Assuming lucide-react for icons
import { useNotifications } from '../../../hooks/use-notifications';
import { Button } from '../../ui/button'; // Assuming a UI library for Button
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '../../ui/dropdown-menu'; // Assuming a UI library for DropdownMenu
import { NotificationItem } from './notification-item'; // Reusing NotificationItem

export const NotificationBell: React.FC = () => {
  const {
    unreadCount,
    notifications: recentUnread, // Renaming for clarity, as this will be a subset
    isLoading,
    error,
    markAsRead,
    markAllAsRead,
    refetchNotifications,
  } = useNotifications(); // useNotifications hook should fetch recent unread for the bell

  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement>(null);

  // When dropdown opens, refetch to ensure latest unread are shown
  useEffect(() => {
    if (isOpen) {
      refetchNotifications(); // This will refetch all notifications, but we only use the first few
    }
  }, [isOpen, refetchNotifications]);

  // Filter for only unread notifications for the bell dropdown
  const unreadNotificationsForBell = recentUnread.filter(n => !n.isRead).slice(0, 5); // Show max 5 recent unread

  return (
    <DropdownMenu open={isOpen} onOpenChange={setIsOpen}>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost" size="icon" className="relative">
          <BellIcon className="h-6 w-6" />
          {unreadCount > 0 && (
            <span className="absolute top-0 right-0 inline-flex items-center justify-center px-2 py-1 text-xs font-bold leading-none text-red-100 bg-red-600 rounded-full transform translate-x-1/2 -translate-y-1/2">
              {unreadCount > 99 ? '99+' : unreadCount}
            </span>
          )}
          <span className="sr-only">View notifications</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent className="w-80 p-0" align="end" ref={dropdownRef}>
        <div className="flex justify-between items-center p-3 border-b border-gray-200 dark:border-gray-700">
          <h3 className="font-semibold text-gray-900 dark:text-gray-100">Notifications</h3>
          {unreadCount > 0 && (
            <Button variant="link" size="sm" onClick={markAllAsRead} disabled={isLoading}>
              Mark all as read
            </Button>
          )}
        </div>
        {isLoading ? (
          <div className="flex justify-center items-center py-4 text-gray-500 dark:text-gray-400">
            <Loader2 className="h-5 w-5 animate-spin mr-2" /> Loading...
          </div>
        ) : error ? (
          <div className="text-center py-4 text-red-500 text-sm">Error loading notifications.</div>
        ) : unreadNotificationsForBell.length === 0 ? (
          <div className="text-center py-4 text-gray-500 dark:text-gray-400 text-sm">No new notifications.</div>
        ) : (
          <div className="divide-y divide-gray-100 dark:divide-gray-800 max-h-96 overflow-y-auto">
            {unreadNotificationsForBell.map((notification) => (
              <NotificationItem
                key={notification.id}
                notification={notification}
                onMarkAsRead={markAsRead}
                onDelete={() => { /* Not exposing delete in bell dropdown for simplicity */ }}
              />
            ))}
          </div>
        )}
        <DropdownMenuSeparator />
        <DropdownMenuItem asChild>
          <Link href="/notifications" className="block text-center py-2 text-blue-600 dark:text-blue-400 hover:bg-gray-50 dark:hover:bg-gray-800">
            View All Notifications
          </Link>
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  );
};
```

```typescript
// FILE 16: src/app/(social)/notifications/page.tsx
// ~60 righe
import React from 'react';
import { NotificationList } from '../../../components/social/notifications/notification-list';
import { getServerAuthSession } from '../../../server/auth'; // Assuming NextAuth.js setup
import { redirect } from 'next/navigation';

export default async function NotificationsPage() {
  const session = await getServerAuthSession();

  if (!session?.user) {
    redirect('/api/auth/signin'); // Redirect unauthenticated users to login
  }

  return (
    <div className="container mx-auto px-4 py-8 max-w-3xl">
      <div className="bg-white dark:bg-gray-800 shadow-md rounded-lg p-6">
        <NotificationList />
      </div>
    </div>
  );
}
```

```typescript
// FILE 17: tests/comments.test.ts
// ~150 righe
import { PrismaClient, Comment, Post, Profile, CommentLike } from '@prisma/client';
import { DeepMockProxy, mockDeep } from 'jest-mock-extended';
import { CommentService } from '../src/server/services/comment-service';
import { NotificationService } from '../src/server/services/notification-service';
import { PostService } from '../src/server/services/post-service';

// Mock Prisma Client
const prismaMock = mockDeep<PrismaClient>();
// Mock NotificationService and PostService
const notificationServiceMock = mockDeep<NotificationService>();
const postServiceMock = mockDeep<PostService>();

let commentService: CommentService;

// Test data
const mockPost: Post = {
  id: 'post1',
  authorId: 'profile1',
  content: 'Test Post',
  createdAt: new Date(),
  updatedAt: new Date(),
  deletedAt: null,
  likesCount: 0,
  commentsCount: 0,
  repostsCount: 0,
};

const mockProfile1: Profile = {
  id: 'profile1',
  userId: 'user1',
  name: 'Author One',
  username: 'authorone',
  email: 'author1@example.com',
  image: null,
  bio: null,
  createdAt: new Date(),
  updatedAt: new Date(),
};

const mockProfile2: Profile = {
  id: 'profile2',
  userId: 'user2',
  name: 'Commenter Two',
  username: 'commentertwo',
  email: 'commenter2@example.com',
  image: null,
  bio: null,
  createdAt: new Date(),
  updatedAt: new Date(),
};

const mockComment1: Comment = {
  id: 'comment1',
  postId: 'post1',
  authorId: 'profile2',
  content: 'This is a test comment.',
  parentId: null,
  depth: 0,
  likesCount: 0,
  repliesCount: 0,
  isEdited: false,
  isPinned: false,
  createdAt: new Date(),
  updatedAt: new Date(),
  deletedAt: null,
};

const mockComment2: Comment = {
  id: 'comment2',
  postId: 'post1',
  authorId: 'profile1',
  content: 'This is a reply to comment 1.',
  parentId: 'comment1',
  depth: 1,
  likesCount: 0,
  repliesCount: 0,
  isEdited: false,
  isPinned: false,
  createdAt: new Date(),
  updatedAt: new Date(),
  deletedAt: null,
};

describe('CommentService', () => {
  beforeEach(() => {
    commentService = new CommentService(prismaMock as unknown as PrismaClient, notificationServiceMock as unknown as NotificationService, postServiceMock as unknown as PostService);
    // Reset mocks before each test
    jest.clearAllMocks();

    // Common mock setups
    (prismaMock.post.findUnique as jest.Mock).mockResolvedValue(mockPost);
    (prismaMock.profile.findUnique as jest.Mock).mockImplementation(({ where }) => {
      if (where.id === 'profile1') return mockProfile1;
      if (where.id === 'profile2') return mockProfile2;
      return null;
    });
    (prismaMock.comment.findUnique as jest.Mock).mockImplementation(({ where }) => {
      if (where.id === 'comment1') return mockComment1;
      if (where.id === 'comment2') return mockComment2;
      return null;
    });
  });

  // --- Create Comment ---
  it('should create a new top-level comment and update post comment count', async () => {
    (prismaMock.comment.create as jest.Mock).mockResolvedValue(mockComment1);
    (prismaMock.comment.count as jest.Mock).mockResolvedValue(1); // For updatePostCommentCount

    const newComment = await commentService.create(mockPost.id, mockProfile2.id, { content: 'New comment' });

    expect(prismaMock.comment.create).toHaveBeenCalledWith({
      data: {
        postId: mockPost.id,
        authorId: mockProfile2.id,
        content: 'New comment',
        parentId: undefined,
        depth: 0,
      },
    });
    expect(postServiceMock.updateCommentCount).toHaveBeenCalledWith(mockPost.id, 1);
    expect(notificationServiceMock.notifyComment).toHaveBeenCalledWith(mockPost.id, mockProfile2.id, newComment.id);
    expect(newComment).toEqual(mockComment1);
  });

  it('should create a reply comment and update parent replies count', async () => {
    (prismaMock.comment.create as jest.Mock).mockResolvedValue(mockComment2);
    (prismaMock.comment.update as jest.Mock).mockResolvedValue({ ...mockComment1, repliesCount: 1 }); // For updateRepliesCount
    (prismaMock.comment.count as jest.Mock).mockResolvedValue(1); // For updateRepliesCount and updatePostCommentCount

    const replyComment = await commentService.create(mockPost.id, mockProfile1.id, { content: 'Reply content', parentId: mockComment1.id });

    expect(prismaMock.comment.create).toHaveBeenCalledWith({
      data: {
        postId: mockPost.id,
        authorId: mockProfile1.id,
        content: 'Reply content',
        parentId: mockComment1.id,
        depth: 1,
      },
    });
    expect(prismaMock.comment.update).toHaveBeenCalledWith({
      where: { id: mockComment1.id },
      data: { repliesCount: { increment: 1 } },
    });
    expect(postServiceMock.updateCommentCount).toHaveBeenCalledWith(mockPost.id, 1); // Assuming 1 total comment for simplicity in this mock
    expect(notificationServiceMock.notifyReply).toHaveBeenCalledWith(mockComment1.id, mockProfile1.id, replyComment.id);
    expect(replyComment).toEqual(mockComment2);
  });

  // --- Update Comment ---
  it('should update a comment', async () => {
    const updatedContent = 'Updated content.';
    const updatedComment = { ...mockComment1, content: updatedContent, isEdited: true };
    (prismaMock.comment.update as jest.Mock).mockResolvedValue(updatedComment);

    const result = await commentService.update(mockComment1.id, { content: updatedContent });

    expect(prismaMock.comment.update).toHaveBeenCalledWith({
      where: { id: mockComment1.id },
      data: { content: updatedContent, isEdited: true },
    });
    expect(result).toEqual(updatedComment);
  });

  // --- Delete Comment ---
  it('should soft delete a comment and update counts', async () => {
    const deletedComment = { ...mockComment1, deletedAt: new Date() };
    (prismaMock.comment.update as jest.Mock).mockResolvedValue(deletedComment);
    (prismaMock.comment.count as jest.Mock).mockResolvedValue(0); // For updatePostCommentCount

    await commentService.delete(mockComment1.id);

    expect(prismaMock.comment.update).toHaveBeenCalledWith({
      where: { id: mockComment1.id },
      data: { deletedAt: expect.any(Date) },
      select: { postId: true, parentId: true },
    });
    expect(postServiceMock.updateCommentCount).toHaveBeenCalledWith(mockPost.id, 0);
  });

  // --- Get Comments ---
  it('should get a comment by ID', async () => {
    (prismaMock.comment.findUnique as jest.Mock).mockResolvedValue({
      ...mockComment1,
      author: mockProfile2,
      likes: [],
    });

    const comment = await commentService.getById(mockComment1.id);
    expect(comment).toBeDefined();
    expect(comment?.id).toBe(mockComment1.id);
  });

  it('should get comments by post ID', async () => {
    (prismaMock.comment.findMany as jest.Mock).mockResolvedValue([mockComment1]);
    (prismaMock.comment.count as jest.Mock).mockResolvedValue(1);

    const result = await commentService.getByPost(mockPost.id);
    expect(result.data).toEqual([mockComment1]);
    expect(result.total).toBe(1);
  });

  it('should get replies for a comment', async () => {
    (prismaMock.comment.findMany as jest.Mock).mockResolvedValue([mockComment2]);
    (prismaMock.comment.count as jest.Mock).mockResolvedValue(1);

    const result = await commentService.getReplies(mockComment1.id);
    expect(result.data).toEqual([mockComment2]);
    expect(result.total).toBe(1);
  });

  // --- Liking Comments ---
  it('should like a comment and increment likesCount', async () => {
    (prismaMock.commentLike.create as jest.Mock).mockResolvedValue({});
    (prismaMock.comment.update as jest.Mock).mockResolvedValue({ ...mockComment1, likesCount: 1 });

    await commentService.like(mockComment1.id, mockProfile1.id);

    expect(prismaMock.commentLike.create).toHaveBeenCalledWith({
      data: { commentId: mockComment1.id, profileId: mockProfile1.id },
    });
    expect(prismaMock.comment.update).toHaveBeenCalledWith({
      where: { id: mockComment1.id },
      data: { likesCount: { increment: 1 } },
    });
    expect(notificationServiceMock.notifyLike).toHaveBeenCalledWith(mockPost.id, mockProfile1.id, mockComment1.id);
  });

  it('should unlike a comment and decrement likesCount', async () => {
    (prismaMock.commentLike.delete as jest.Mock).mockResolvedValue({});
    (prismaMock.comment.update as jest.Mock).mockResolvedValue({ ...mockComment1, likesCount: 0 });

    await commentService.unlike(mockComment1.id, mockProfile1.id);

    expect(prismaMock.commentLike.delete).toHaveBeenCalledWith({
      where: { commentId_profileId: { commentId: mockComment1.id, profileId: mockProfile1.id } },
    });
    expect(prismaMock.comment.update).toHaveBeenCalledWith({
      where: { id: mockComment1.id },
      data: { likesCount: { decrement: 1 } },
    });
  });

  it('should return true if comment is liked by profile', async () => {
    (prismaMock.commentLike.findUnique as jest.Mock).mockResolvedValue({ id: 'like1', commentId: mockComment1.id, profileId: mockProfile1.id });
    const isLiked = await commentService.isLiked(mockComment1.id, mockProfile1.id);
    expect(isLiked).toBe(true);
  });

  it('should return false if comment is not liked by profile', async () => {
    (prismaMock.commentLike.findUnique as jest.Mock).mockResolvedValue(null);
    const isLiked = await commentService.isLiked(mockComment1.id, mockProfile1.id);
    expect(isLiked).toBe(false);
  });

  // --- Pinning Comments ---
  it('should pin a comment if user is post author', async () => {
    (prismaMock.comment.findUnique as jest.Mock).mockResolvedValueOnce({ id: mockComment1.id, postId: mockPost.id }) // For comment lookup
                                                .mockResolvedValueOnce(mockComment1); // For getById in service
    (prismaMock.post.findUnique as jest.Mock).mockResolvedValue({ id: mockPost.id, authorId: mockProfile1.id });
    (prismaMock.comment.update as jest.Mock).mockResolvedValue({ ...mockComment1, isPinned: true });

    await commentService.pin(mockComment1.id, mockProfile1.id);
    expect(prismaMock.comment.update).toHaveBeenCalledWith({
      where: { id: mockComment1.id },
      data: { isPinned: true },
    });
  });

  it('should throw error if non-post author tries to pin a comment', async () => {
    (prismaMock.comment.findUnique as jest.Mock).mockResolvedValueOnce({ id: mockComment1.id, postId: mockPost.id });
    (prismaMock.post.findUnique as jest.Mock).mockResolvedValue({ id: mockPost.id, authorId: mockProfile1.id });

    await expect(commentService.pin(mockComment1.id, mockProfile2.id)).rejects.toThrow('Unauthorized: Only the post author can pin comments.');
  });
});
```

```typescript
// FILE 18: tests/notifications.test.ts
// ~150 righe
import { PrismaClient, Notification, NotificationType, NotificationPreferences, PushSubscription } from '@prisma/client';
import { DeepMockProxy, mockDeep } from 'jest-mock-extended';
import { NotificationService } from '../src/server/services/notification-service';

// Mock Prisma Client
const prismaMock = mockDeep<PrismaClient>();

// Mock RealtimeService and PushNotificationProvider
const realtimeServiceMock = {
  publish: jest.fn(),
  subscribe: jest.fn(() => jest.fn()), // Returns an unsubscribe function
};
const pushProviderMock = {
  send: jest.fn(),
};

let notificationService: NotificationService;

// Test data
const mockProfile1 = { id: 'profile1', name: 'User One', image: 'user1.jpg' };
const mockProfile2 = { id: 'profile2', name: 'User Two', image: 'user2.jpg' };
const mockPost = { id: 'post1', authorId: 'profile1', title: 'My Awesome Post' };
const mockComment = { id: 'comment1', authorId: 'profile1', postId: 'post1', content: 'Great post!' };

const mockNotification: Notification = {
  id: 'notif1',
  recipientId: 'profile1',
  type: NotificationType.LIKE_POST,
  actorId: 'profile2',
  postId: 'post1',
  commentId: null,
  followId: null,
  title: null,
  body: null,
  imageUrl: null,
  actionUrl: '/post/post1',
  isRead: false,
  readAt: null,
  createdAt: new Date(),
};

const mockPreferences: NotificationPreferences = {
  id: 'prefs1',
  profileId: 'profile1',
  email_newFollower: true,
  email_postLike: true,
  email_comment: true,
  email_reply: true,
  email_mention: true,
  email_system: true,
  inApp_newFollower: true,
  inApp_postLike: true,
  inApp_comment: true,
  inApp_reply: true,
  inApp_mention: true,
  inApp_system: true,
  push_newFollower: true,
  push_postLike: true,
  push_comment: true,
  push_reply: true,
  push_mention: true,
  push_system: true,
  createdAt: new Date(),
  updatedAt: new Date(),
};

describe('NotificationService', () => {
  beforeEach(() => {
    notificationService = new NotificationService(prismaMock as unknown as PrismaClient, realtimeServiceMock, pushProviderMock);
    jest.clearAllMocks();

    // Common mock setups
    (prismaMock.profile.findUnique as jest.Mock).mockImplementation(({ where }) => {
      if (where.id === 'profile1') return mockProfile1;
      if (where.id === 'profile2') return mockProfile2;
      return null;
    });
    (prismaMock.post.findUnique as jest.Mock).mockResolvedValue(mockPost);
    (prismaMock.comment.findUnique as jest.Mock).mockResolvedValue(mockComment);
    (prismaMock.notificationPreferences.findUnique as jest.Mock).mockResolvedValue(mockPreferences);
    (prismaMock.notificationPreferences.create as jest.Mock).mockResolvedValue(mockPreferences);
    (prismaMock.pushSubscription.findMany as jest.Mock).mockResolvedValue([]); // No push subscriptions by default
  });

  // --- Create Notification ---
  it('should create a notification and publish to real-time if enabled', async () => {
    (prismaMock.notification.create as jest.Mock).mockResolvedValue(mockNotification);

    const notification = await notificationService.create({
      recipientId: mockProfile1.id,
      type: NotificationType.LIKE_POST,
      actorId: mockProfile2.id,
      postId: mockPost.id,
      actionUrl: '/post/post1',
    });

    expect(prismaMock.notification.create).toHaveBeenCalledWith(expect.objectContaining({
      data: expect.objectContaining({
        recipientId: mockProfile1.id,
        type: NotificationType.LIKE_POST,
      }),
    }));
    expect(realtimeServiceMock.publish).toHaveBeenCalledWith(`notifications:${mockProfile1.id}`, 'newNotification', notification);
    expect(pushProviderMock.send).not.toHaveBeenCalled(); // No push tokens registered
    expect(notification).toEqual(mockNotification);
  });

  it('should not create notification if in-app preferences are off', async () => {
    (prismaMock.notificationPreferences.findUnique as jest.Mock).mockResolvedValue({ ...mockPreferences, inApp_postLike: false });

    const notification = await notificationService.create({
      recipientId: mockProfile1.id,
      type: NotificationType.LIKE_POST,
      actorId: mockProfile2.id,
      postId: mockPost.id,
      actionUrl: '/post/post1',
    });

    expect(prismaMock.notification.create).not.toHaveBeenCalled();
    expect(notification).toBeNull();
  });

  // --- Specific Notification Types ---
  it('should notify a follow', async () => {
    (prismaMock.notification.create as jest.Mock).mockResolvedValue(mockNotification);
    await notificationService.notifyFollow(mockProfile2.id, mockProfile1.id);
    expect(prismaMock.notification.create).toHaveBeenCalledWith(expect.objectContaining({
      data: expect.objectContaining({
        recipientId: mockProfile1.id,
        type: NotificationType.FOLLOW

---
_Modello: gemini-2.5-flash_