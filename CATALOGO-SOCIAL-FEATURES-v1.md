# CATALOGO SOCIAL-FEATURES v1
§ DOCUMENTAZIONE TECNICA COMPLETA PER FUNZIONALITÀ SOCIAL IN REACT/NEXT.JS

---

§ §1. SOCIAL FEATURES OVERVIEW

§ **TABELLA COMPARATIVA FUNZIONALITÀ SOCIAL**

| Feature | Complexity | Real-time | Database Load | Cache Strategy | Implementation Time | Best Practices |
|---------|------------|-----------|---------------|----------------|---------------------|----------------|
| **Comments** | Medium | Optional (SSE) | Medium (reads) | Redis cache counts | 2-3 days | Lazy loading replies, rate limiting |
| **Replies (nested)** | High | Optional | High (joins) | Materialized path | 3-5 days | Limit depth, collapse deep threads |
| **Likes/Reactions** | Low | Optional | High writes | Redis + batch sync | 1-2 days | Optimistic UI, debounced updates |
| **Follow/Unfollow** | Low | No | Low | Redis counters | 1 day | Asymmetric follow model |
| **Activity Feed** | High | Yes (WebSocket) | High read/write | Redis feed, 7-day TTL | 4-6 days | Hybrid push-pull, precomputed |
| **Notifications** | Medium | Yes (SSE) | Medium | Redis pub/sub | 2-4 days | Dedupe, aggregate, batch send |
| **Bookmarks/Saves** | Low | No | Low | None needed | 1 day | Simple toggle, collections optional |
| **Share** | Low | No | Low | Analytics only | 1 day | Web Share API fallback |
| **Mentions** | Medium | Yes (WebSocket) | Medium | Bloom filter cache | 2-3 days | Async parsing, user search index |

§ **STACK CONSIGLIATO PER SCALABILITÀ**

typescript
// Tech stack per produzione:
{
  database: "PostgreSQL (Supabase/Neon)", // Per transazioni ACID
  cache: "Redis (Upstash)", // Per counters, feeds, sessions
  realtime: "Pusher/Ably" o "Socket.io + Redis pub/sub", // Per notifiche live
  search: "PostgreSQL full-text" o "Meilisearch", // Per mentions search
  analytics: "PostHog" o "Mixpanel", // Per engagement metrics
  moderation: "Akismet API", // Per spam detection
}

---

§ §2. COMMENTS SYSTEM

§ 2.1 COMMENT DATA MODEL COMPLETO

prisma
// schema.prisma - Comment System
model Comment {
  id          String        @id @default(cuid())
  
  // Contenuto
  content     String        @db.Text
  contentHtml String        @db.Text // Sanitized HTML per rendering sicuro
  contentJson Json?         // Per structured content (mentions, links)
  
  // Autore
  authorId    String
  author      User          @relation(fields: [authorId], references: [id], onDelete: Cascade)
  
  // Target (polymorphic)
  targetType  String        // 'post', 'product', 'article', 'video'
  targetId    String
  
  // Threading (adjacency list)
  parentId    String?
  parent      Comment?      @relation("CommentReplies", fields: [parentId], references: [id])
  replies     Comment[]     @relation("CommentReplies")
  
  // Path per nested queries efficienti
  path        String[]      // ['root-id', 'parent-id', 'comment-id']
  depth       Int           @default(0) // 0 = root comment
  
  // Counters denormalizzati
  likesCount  Int           @default(0)
  repliesCount Int          @default(0)
  score       Int           @default(0) // Per ranking (likes - reports)
  
  // Metadata
  isEdited    Boolean       @default(false)
  isPinned    Boolean       @default(false)
  isFeatured  Boolean       @default(false) // Per admin highlighting
  editCount   Int           @default(0)
  lastEditedAt DateTime?
  
  // SEO e discovery
  slug        String?       @unique // Per permalinks
  metadata    Json?         // Per OpenGraph, rich snippets
  
  // Moderation
  status      CommentStatus @default(VISIBLE)
  hiddenReason String?
  hiddenById  String?
  hiddenBy    User?         @relation("CommentHiddenBy", fields: [hiddenById], references: [id])
  hiddenAt    DateTime?
  
  // Flags per utenti
  isLikedByMe Boolean?      @default(false) // Hydrated a runtime
  
  // Timestamps
  createdAt   DateTime      @default(now())
  updatedAt   DateTime      @updatedAt
  
  // Relations
  mentions    Mention[]     // Per @mentions
  attachments CommentAttachment[]
  reports     Report[]
  likes       Like[]
  reactions   Reaction[]
  
  // Indexes
  @@index([targetType, targetId, createdAt])
  @@index([targetType, targetId, score])
  @@index([authorId, createdAt])
  @@index([parentId])
  @@index([path])
  @@index([status])
  @@index([isPinned])
  
  // Unique constraint
  @@unique([targetType, targetId, slug])
}

model CommentAttachment {
  id         String   @id @default(cuid())
  commentId  String
  comment    Comment  @relation(fields: [commentId], references: [id], onDelete: Cascade)
  
  type       String   // 'image', 'video', 'file', 'link'
  url        String
  thumbnail  String?
  metadata   Json?    // { width, height, size, duration, etc }
  
  createdAt  DateTime @default(now())
  
  @@index([commentId])
}

enum CommentStatus {
  VISIBLE
  HIDDEN      // Hidden by moderator
  DELETED     // Soft-deleted
  FLAGGED     // Flagged for review
  PENDING     // Awaiting moderation
  REMOVED     // Removed by author
  SPAM        // Marked as spam
}

// Materialized view per performance (opzionale)
// CREATE MATERIALIZED VIEW comment_threads AS
// WITH RECURSIVE thread_path AS (...)

§ 2.2 COMMENT SERVICE COMPLETO

typescript
// lib/services/comment-service.ts
import { PrismaClient, Comment, CommentStatus, Prisma } from '@prisma/client';
import { z } from 'zod';
import { NotFoundError, ValidationError, PermissionError } from '@/lib/errors';
import { redis } from '@/lib/redis';
import { rateLimit } from '@/lib/rate-limit';
import { sanitizeHtml } from '@/lib/utils/sanitize';
import { extractMentions } from './mention-service';

const prisma = new PrismaClient();

// ============ VALIDATION SCHEMAS ============
const CreateCommentSchema = z.object({
  content: z.string()
    .min(1, 'Comment cannot be empty')
    .max(5000, 'Comment cannot exceed 5000 characters')
    .refine((val) => {
      // Check for spam patterns
      const urlCount = (val.match(/https?:\/\/\S+/g) || []).length;
      return urlCount <= 3; // Max 3 URLs per comment
    }, 'Too many URLs in comment'),
  
  contentHtml: z.string().optional(),
  
  parentId: z.string().cuid().optional(),
  
  attachments: z.array(z.object({
    type: z.enum(['image', 'video', 'file', 'link']),
    url: z.string().url(),
    thumbnail: z.string().url().optional(),
    metadata: z.record(z.any()).optional(),
  })).max(5, 'Maximum 5 attachments per comment').optional(),
  
  metadata: z.record(z.any()).optional(),
});

const UpdateCommentSchema = CreateCommentSchema.partial();

const CommentFiltersSchema = z.object({
  page: z.number().int().positive().default(1),
  limit: z.number().int().min(1).max(100).default(20),
  sortBy: z.enum(['newest', 'oldest', 'top', 'controversial']).default('newest'),
  depth: z.number().int().min(0).max(10).optional(),
  includeReplies: z.boolean().default(false),
  replyDepth: z.number().int().min(1).max(3).default(1),
  authorId: z.string().cuid().optional(),
  status: z.nativeEnum(CommentStatus).optional(),
  excludeAuthorId: z.string().cuid().optional(),
});

type CreateCommentInput = z.infer<typeof CreateCommentSchema>;
type UpdateCommentInput = z.infer<typeof UpdateCommentSchema>;
type CommentFilters = z.infer<typeof CommentFiltersSchema>;

// ============ COMMENT SERVICE ============
export class CommentService {
  private readonly CACHE_TTL = 60 * 5; // 5 minutes
  private readonly REDIS_PREFIX = 'comments';
  
  // ============ CREATE COMMENT ============
  async createComment(
    authorId: string,
    targetType: string,
    targetId: string,
    data: CreateCommentInput,
    options?: {
      ipAddress?: string;
      userAgent?: string;
      bypassRateLimit?: boolean;
    }
  ): Promise<Comment> {
    try {
      // Rate limiting
      if (!options?.bypassRateLimit) {
        const limitKey = `comment:create:${authorId}`;
        const canProceed = await rateLimit.check(limitKey, {
          max: 30, // 30 comments per hour
          window: 60 * 60 * 1000, // 1 hour
        });
        
        if (!canProceed) {
          throw new ValidationError('Rate limit exceeded. Please wait before posting another comment.');
        }
      }
      
      // Validate input
      const validatedData = CreateCommentSchema.parse(data);
      
      // Spam check
      await this.checkForSpam(validatedData.content, authorId, options?.ipAddress);
      
      // Generate sanitized HTML if not provided
      const contentHtml = validatedData.contentHtml || sanitizeHtml(validatedData.content);
      
      // Extract mentions
      const mentions = await extractMentions(validatedData.content);
      
      // Determine if this is a reply
      let parentComment: Comment | null = null;
      let path: string[] = [];
      let depth = 0;
      
      if (validatedData.parentId) {
        parentComment = await prisma.comment.findUnique({
          where: { id: validatedData.parentId },
          include: { author: true },
        });
        
        if (!parentComment) {
          throw new NotFoundError('Parent comment not found');
        }
        
        // Check if parent is on same target
        if (parentComment.targetType !== targetType || parentComment.targetId !== targetId) {
          throw new ValidationError('Parent comment must be on the same content');
        }
        
        // Check depth limit
        if (parentComment.depth >= 10) {
          throw new ValidationError('Maximum comment depth reached');
        }
        
        path = [...parentComment.path, parentComment.id];
        depth = parentComment.depth + 1;
      } else {
        path = []; // Root comment
      }
      
      // Create comment with transaction
      const comment = await prisma.$transaction(async (tx) => {
        // Create comment
        const newComment = await tx.comment.create({
          data: {
            content: validatedData.content,
            contentHtml,
            contentJson: mentions.length > 0 ? { mentions } : undefined,
            authorId,
            targetType,
            targetId,
            parentId: validatedData.parentId,
            path,
            depth,
            status: CommentStatus.VISIBLE,
            metadata: validatedData.metadata,
          },
          include: {
            author: {
              select: {
                id: true,
                name: true,
                username: true,
                image: true,
              },
            },
          },
        });
        
        // Create attachments if any
        if (validatedData.attachments && validatedData.attachments.length > 0) {
          await tx.commentAttachment.createMany({
            data: validatedData.attachments.map(attachment => ({
              commentId: newComment.id,
              ...attachment,
            })),
          });
        }
        
        // Update parent's reply count
        if (parentComment) {
          await tx.comment.update({
            where: { id: parentComment.id },
            data: {
              repliesCount: { increment: 1 },
              updatedAt: new Date(),
            },
          });
        }
        
        // Create mention relationships
        if (mentions.length > 0) {
          const users = await tx.user.findMany({
            where: {
              username: { in: mentions.map(m => m.username) },
            },
            select: { id: true, username: true },
          });
          
          await Promise.all(
            users.map(user =>
              tx.mention.create({
                data: {
                  commentId: newComment.id,
                  mentionedUserId: user.id,
                  sourceUserId: authorId,
                  sourceType: 'comment',
                  sourceId: newComment.id,
                },
              })
            )
          );
        }
        
        // Increment target comment count
        await this.incrementTargetCommentCount(tx, targetType, targetId, 1);
        
        // Create activity
        await tx.activity.create({
          data: {
            actorId: authorId,
            verb: parentComment ? 'replied' : 'commented',
            objectType: 'comment',
            objectId: newComment.id,
            targetType,
            targetId,
            targetPreview: await this.getTargetPreview(tx, targetType, targetId),
          },
        });
        
        return newComment;
      });
      
      // Clear caches
      await this.clearCommentCaches(targetType, targetId, comment.parentId);
      
      // Real-time notification (if implemented)
      await this.notifyNewComment(comment, parentComment);
      
      return comment;
    } catch (error) {
      if (error instanceof z.ZodError) {
        throw new ValidationError('Invalid comment data', error.errors);
      }
      throw error;
    }
  }
  
  // ============ UPDATE COMMENT ============
  async updateComment(
    commentId: string,
    authorId: string,
    data: UpdateCommentInput
  ): Promise<Comment> {
    try {
      // Find comment
      const comment = await prisma.comment.findUnique({
        where: { id: commentId },
        include: { author: true },
      });
      
      if (!comment) {
        throw new NotFoundError('Comment not found');
      }
      
      // Check permission
      if (comment.authorId !== authorId) {
        throw new PermissionError('You can only edit your own comments');
      }
      
      // Check if comment is editable (not too old)
      const hoursSinceCreation = (Date.now() - comment.createdAt.getTime()) / (1000 * 60 * 60);
      if (hoursSinceCreation > 24) {
        throw new ValidationError('Comments can only be edited within 24 hours of creation');
      }
      
      // Validate input
      const validatedData = UpdateCommentSchema.parse(data);
      
      // Generate new content HTML if content changed
      let contentHtml = comment.contentHtml;
      if (validatedData.content && validatedData.content !== comment.content) {
        contentHtml = validatedData.contentHtml || sanitizeHtml(validatedData.content);
        
        // Spam check for updated content
        await this.checkForSpam(validatedData.content, authorId);
      }
      
      // Update comment
      const updatedComment = await prisma.comment.update({
        where: { id: commentId },
        data: {
          content: validatedData.content,
          contentHtml,
          isEdited: true,
          editCount: { increment: 1 },
          lastEditedAt: new Date(),
          metadata: validatedData.metadata,
        },
        include: {
          author: {
            select: {
              id: true,
              name: true,
              username: true,
              image: true,
            },
          },
          attachments: true,
        },
      });
      
      // Clear caches
      await this.clearCommentCaches(
        updatedComment.targetType,
        updatedComment.targetId,
        updatedComment.parentId
      );
      
      return updatedComment;
    } catch (error) {
      if (error instanceof z.ZodError) {
        throw new ValidationError('Invalid comment data', error.errors);
      }
      throw error;
    }
  }
  
  // ============ DELETE COMMENT (SOFT DELETE) ============
  async deleteComment(commentId: string, userId: string, isAdmin: boolean = false): Promise<void> {
    try {
      const comment = await prisma.comment.findUnique({
        where: { id: commentId },
        include: {
          replies: { select: { id: true } },
          author: true,
        },
      });
      
      if (!comment) {
        throw new NotFoundError('Comment not found');
      }
      
      // Check permission
      if (!isAdmin && comment.authorId !== userId) {
        throw new PermissionError('You can only delete your own comments');
      }
      
      await prisma.$transaction(async (tx) => {
        // Soft delete the comment
        await tx.comment.update({
          where: { id: commentId },
          data: {
            status: CommentStatus.REMOVED,
            content: '[deleted]',
            contentHtml: '<p>[deleted]</p>',
            authorId: isAdmin ? comment.authorId : userId, // Keep original author if admin deletion
            metadata: {
              ...comment.metadata,
              deletedBy: userId,
              deletedAt: new Date().toISOString(),
              deletedAsAdmin: isAdmin,
            },
          },
        });
        
        // Update parent's reply count if this is a reply
        if (comment.parentId) {
          await tx.comment.update({
            where: { id: comment.parentId },
            data: {
              repliesCount: { decrement: 1 },
            },
          });
        }
        
        // Decrement target comment count
        await this.incrementTargetCommentCount(tx, comment.targetType, comment.targetId, -1);
        
        // If comment has replies, update their status too
        if (comment.replies.length > 0) {
          await tx.comment.updateMany({
            where: { parentId: commentId },
            data: {
              status: CommentStatus.REMOVED,
              content: '[parent deleted]',
              contentHtml: '<p>[parent deleted]</p>',
            },
          });
        }
      });
      
      // Clear caches
      await this.clearCommentCaches(comment.targetType, comment.targetId, comment.parentId);
      
    } catch (error) {
      throw error;
    }
  }
  
  // ============ GET COMMENTS ============
  async getComments(
    targetType: string,
    targetId: string,
    filters: CommentFilters,
    currentUserId?: string
  ): Promise<{
    comments: any[];
    total: number;
    page: number;
    totalPages: number;
    pinnedComments: any[];
  }> {
    try {
      const validatedFilters = CommentFiltersSchema.parse(filters);
      const { page, limit, sortBy, depth, includeReplies, replyDepth, authorId, status, excludeAuthorId } = validatedFilters;
      const skip = (page - 1) * limit;
      
      // Build where clause
      const where: Prisma.CommentWhereInput = {
        targetType,
        targetId,
        status: status || CommentStatus.VISIBLE,
        parentId: depth === 0 ? null : undefined, // If depth=0, only root comments
      };
      
      if (authorId) {
        where.authorId = authorId;
      }
      
      if (excludeAuthorId) {
        where.authorId = { not: excludeAuthorId };
      }
      
      if (depth !== undefined) {
        where.depth = depth;
      }
      
      // Build orderBy
      let orderBy: Prisma.CommentOrderByWithRelationInput;
      switch (sortBy) {
        case 'newest':
          orderBy = { createdAt: 'desc' };
          break;
        case 'oldest':
          orderBy = { createdAt: 'asc' };
          break;
        case 'top':
          orderBy = { score: 'desc' };
          break;
        case 'controversial':
          // Comments with high engagement (likes + replies) but low score
          orderBy = { likesCount: 'desc' };
          break;
        default:
          orderBy = { createdAt: 'desc' };
      }
      
      // Try cache first
      const cacheKey = `${this.REDIS_PREFIX}:${targetType}:${targetId}:${page}:${limit}:${sortBy}:${depth}:${currentUserId || 'guest'}`;
      const cached = await redis.get(cacheKey);
      
      if (cached) {
        return JSON.parse(cached);
      }
      
      // Get pinned comments separately
      const pinnedComments = await prisma.comment.findMany({
        where: {
          ...where,
          isPinned: true,
        },
        include: this.getCommentInclude(currentUserId, includeReplies, replyDepth),
        orderBy: { createdAt: 'desc' },
      });
      
      // Exclude pinned comments from regular results
      where.isPinned = false;
      
      // Execute queries in parallel
      const [comments, total] = await Promise.all([
        prisma.comment.findMany({
          where,
          include: this.getCommentInclude(currentUserId, includeReplies, replyDepth),
          orderBy,
          skip,
          take: limit,
        }),
        prisma.comment.count({ where }),
      ]);
      
      // Hydrate with user-specific data
      const hydratedComments = await this.hydrateComments(comments, currentUserId);
      
      const result = {
        comments: hydratedComments,
        pinnedComments: await this.hydrateComments(pinnedComments, currentUserId),
        total,
        page,
        totalPages: Math.ceil(total / limit),
      };
      
      // Cache result
      await redis.setex(cacheKey, this.CACHE_TTL, JSON.stringify(result));
      
      return result;
    } catch (error) {
      if (error instanceof z.ZodError) {
        throw new ValidationError('Invalid filter parameters', error.errors);
      }
      throw error;
    }
  }
  
  // ============ GET COMMENT BY ID ============
  async getComment(
    commentId: string,
    currentUserId?: string,
    includeThread: boolean = false
  ): Promise<any> {
    try {
      const comment = await prisma.comment.findUnique({
        where: { id: commentId },
        include: {
          author: {
            select: {
              id: true,
              name: true,
              username: true,
              image: true,
              bio: true,
              followersCount: true,
              followingCount: true,
            },
          },
          attachments: true,
          parent: includeThread
            ? {
                include: {
                  author: {
                    select: {
                      id: true,
                      name: true,
                      username: true,
                      image: true,
                    },
                  },
                },
              }
            : false,
          replies: includeThread
            ? {
                take: 10,
                orderBy: { createdAt: 'asc' },
                include: {
                  author: {
                    select: {
                      id: true,
                      name: true,
                      username: true,
                      image: true,
                    },
                  },
                },
              }
            : false,
          _count: {
            select: {
              replies: true,
              likes: true,
            },
          },
        },
      });
      
      if (!comment) {
        throw new NotFoundError('Comment not found');
      }
      
      if (comment.status !== CommentStatus.VISIBLE) {
        // Only allow viewing if user is author or admin
        const isAuthor = currentUserId === comment.authorId;
        if (!isAuthor) {
          throw new NotFoundError('Comment not found');
        }
      }
      
      // Hydrate with user-specific data
      const hydrated = await this.hydrateComments([comment], currentUserId);
      
      return hydrated[0];
    } catch (error) {
      throw error;
    }
  }
  
  // ============ GET REPLIES ============
  async getReplies(
    commentId: string,
    filters: Omit<CommentFilters, 'depth'>,
    currentUserId?: string
  ): Promise<{
    replies: any[];
    total: number;
    page: number;
    totalPages: number;
  }> {
    try {
      const validatedFilters = CommentFiltersSchema.omit({ depth: true }).parse(filters);
      const { page, limit, sortBy, includeReplies, replyDepth } = validatedFilters;
      const skip = (page - 1) * limit;
      
      const where: Prisma.CommentWhereInput = {
        parentId: commentId,
        status: CommentStatus.VISIBLE,
      };
      
      // Build orderBy
      let orderBy: Prisma.CommentOrderByWithRelationInput;
      switch (sortBy) {
        case 'newest':
          orderBy = { createdAt: 'desc' };
          break;
        case 'oldest':
          orderBy = { createdAt: 'asc' };
          break;
        case 'top':
          orderBy = { likesCount: 'desc' };
          break;
        default:
          orderBy = { createdAt: 'asc' }; // Default chronological for replies
      }
      
      const [replies, total] = await Promise.all([
        prisma.comment.findMany({
          where,
          include: this.getCommentInclude(currentUserId, includeReplies, replyDepth),
          orderBy,
          skip,
          take: limit,
        }),
        prisma.comment.count({ where }),
      ]);
      
      const hydratedReplies = await this.hydrateComments(replies, currentUserId);
      
      return {
        replies: hydratedReplies,
        total,
        page,
        totalPages: Math.ceil(total / limit),
      };
    } catch (error) {
      if (error instanceof z.ZodError) {
        throw new ValidationError('Invalid filter parameters', error.errors);
      }
      throw error;
    }
  }
  
  // ============ PIN COMMENT ============
  async pinComment(commentId: string, userId: string, isAdmin: boolean = false): Promise<void> {
    try {
      const comment = await prisma.comment.findUnique({
        where: { id: commentId },
        include: { author: true },
      });
      
      if (!comment) {
        throw new NotFoundError('Comment not found');
      }
      
      // Check permission
      // Only admin or target owner can pin comments
      const isTargetOwner = await this.isTargetOwner(userId, comment.targetType, comment.targetId);
      if (!isAdmin && !isTargetOwner) {
        throw new PermissionError('Only content owners or admins can pin comments');
      }
      
      // Unpin any currently pinned comments on this target
      await prisma.comment.updateMany({
        where: {
          targetType: comment.targetType,
          targetId: comment.targetId,
          isPinned: true,
        },
        data: { isPinned: false },
      });
      
      // Pin the comment
      await prisma.comment.update({
        where: { id: commentId },
        data: { isPinned: true },
      });
      
      // Clear caches
      await this.clearCommentCaches(comment.targetType, comment.targetId);
      
    } catch (error) {
      throw error;
    }
  }
  
  // ============ UNPIN COMMENT ============
  async unpinComment(commentId: string, userId: string, isAdmin: boolean = false): Promise<void> {
    try {
      const comment = await prisma.comment.findUnique({
        where: { id: commentId },
      });
      
      if (!comment) {
        throw new NotFoundError('Comment not found');
      }
      
      // Check permission
      const isTargetOwner = await this.isTargetOwner(userId, comment.targetType, comment.targetId);
      if (!isAdmin && !isTargetOwner) {
        throw new PermissionError('Only content owners or admins can unpin comments');
      }
      
      await prisma.comment.update({
        where: { id: commentId },
        data: { isPinned: false },
      });
      
      await this.clearCommentCaches(comment.targetType, comment.targetId);
      
    } catch (error) {
      throw error;
    }
  }
  
  // ============ GET COMMENT COUNT ============
  async getCommentCount(targetType: string, targetId: string): Promise<number> {
    const cacheKey = `${this.REDIS_PREFIX}:count:${targetType}:${targetId}`;
    
    try {
      const cached = await redis.get(cacheKey);
      if (cached) {
        return parseInt(cached, 10);
      }
      
      const count = await prisma.comment.count({
        where: {
          targetType,
          targetId,
          status: CommentStatus.VISIBLE,
          parentId: null, // Only count root comments
        },
      });
      
      await redis.setex(cacheKey, this.CACHE_TTL, count.toString());
      
      return count;
    } catch (error) {
      throw error;
    }
  }
  
  // ============ MODERATION ACTIONS ============
  async hideComment(
    commentId: string,
    moderatorId: string,
    reason: string,
    isAdmin: boolean = false
  ): Promise<void> {
    try {
      const comment = await prisma.comment.findUnique({
        where: { id: commentId },
      });
      
      if (!comment) {
        throw new NotFoundError('Comment not found');
      }
      
      // Only admins can hide comments
      if (!isAdmin) {
        throw new PermissionError('Only admins can hide comments');
      }
      
      await prisma.comment.update({
        where: { id: commentId },
        data: {
          status: CommentStatus.HIDDEN,
          hiddenReason: reason,
          hiddenById: moderatorId,
          hiddenAt: new Date(),
        },
      });
      
      await this.clearCommentCaches(comment.targetType, comment.targetId, comment.parentId);
      
    } catch (error) {
      throw error;
    }
  }
  
  async restoreComment(commentId: string, moderatorId: string, isAdmin: boolean = false): Promise<void> {
    try {
      const comment = await prisma.comment.findUnique({
        where: { id: commentId },
      });
      
      if (!comment) {
        throw new NotFoundError('Comment not found');
      }
      
      if (!isAdmin) {
        throw new PermissionError('Only admins can restore comments');
      }
      
      await prisma.comment.update({
        where: { id: commentId },
        data: {
          status: CommentStatus.VISIBLE,
          hiddenReason: null,
          hiddenById: null,
          hiddenAt: null,
        },
      });
      
      await this.clearCommentCaches(comment.targetType, comment.targetId, comment.parentId);
      
    } catch (error) {
      throw error;
    }
  }
  
  // ============ UTILITY METHODS ============
  private getCommentInclude(
    currentUserId?: string,
    includeReplies: boolean = false,
    replyDepth: number = 1
  ): Prisma.CommentInclude {
    const baseInclude: Prisma.CommentInclude = {
      author: {
        select: {
          id: true,
          name: true,
          username: true,
          image: true,
          bio: true,
          verified: true,
          followersCount: true,
        },
      },
      attachments: true,
      _count: {
        select: {
          replies: true,
          likes: true,
        },
      },
    };
    
    if (includeReplies && replyDepth > 0) {
      return {
        ...baseInclude,
        replies: {
          take: 5, // Limit initial replies
          orderBy: { createdAt: 'asc' },
          include: this.getReplyInclude(replyDepth - 1),
        },
      };
    }
    
    return baseInclude;
  }
  
  private getReplyInclude(depth: number): Prisma.CommentInclude {
    if (depth <= 0) {
      return {
        author: {
          select: {
            id: true,
            name: true,
            username: true,
            image: true,
          },
        },
        _count: {
          select: {
            replies: true,
            likes: true,
          },
        },
      };
    }
    
    return {
      author: {
        select: {
          id: true,
          name: true,
          username: true,
          image: true,
        },
      },
      _count: {
        select: {
          replies: true,
          likes: true,
        },
      },
      replies: {
        take: 3,
        orderBy: { createdAt: 'asc' },
        include: this.getReplyInclude(depth - 1),
      },
    };
  }
  
  private async hydrateComments(comments: any[], currentUserId?: string): Promise<any[]> {
    if (comments.length === 0) return [];
    
    // Get likes for current user
    const commentIds = comments.map(c => c.id);
    const userLikes = currentUserId
      ? await prisma.like.findMany({
          where: {
            userId: currentUserId,
            targetType: 'comment',
            targetId: { in: commentIds },
          },
          select: { targetId: true },
        })
      : [];
    
    const likedCommentIds = new Set(userLikes.map(like => like.targetId));
    
    // Hydrate comments
    return comments.map(comment => ({
      ...comment,
      isLikedByMe: likedCommentIds.has(comment.id),
      replies: comment.replies
        ? await this.hydrateComments(comment.replies, currentUserId)
        : undefined,
    }));
  }
  
  private async checkForSpam(content: string, userId: string, ipAddress?: string): Promise<void> {
    // Basic spam detection
    const spamPatterns = [
      /[A-Z]{5,}/g, // Excessive caps
      /(http|https|www\.)\S+/gi, // Multiple URLs
      /(buy|cheap|discount|price|order now)/gi, // Commercial keywords
      /\b(viagra|cialis|pharmacy)\b/gi, // Pharma spam
    ];
    
    let spamScore = 0;
    
    for (const pattern of spamPatterns) {
      const matches = content.match(pattern);
      if (matches) {
        spamScore += matches.length;
      }
    }
    
    if (spamScore > 5) {
      // Check user's history
      const recentComments = await prisma.comment.count({
        where: {
          authorId: userId,
          createdAt: {
            gte: new Date(Date.now() - 5 * 60 * 1000), // Last 5 minutes
          },
        },
      });
      
      if (recentComments > 10) {
        throw new ValidationError('Possible spam detected. Please slow down.');
      }
    }
    
    // Optional: Integrate with Akismet or similar service
    if (process.env.AKISMET_API_KEY) {
      const isSpam = await this.checkWithAkismet(content, userId, ipAddress);
      if (isSpam) {
        throw new ValidationError('Comment flagged as spam');
      }
    }
  }
  
  private async checkWithAkismet(content: string, userId: string, ipAddress?: string): Promise<boolean> {
    // Implementation for Akismet API
    return false; // Placeholder
  }
  
  private async incrementTargetCommentCount(
    tx: Prisma.TransactionClient,
    targetType: string,
    targetId: string,
    increment: number
  ): Promise<void> {
    // This is a simplified example. In reality, you'd have different tables
    // for different target types (posts, products, etc.)
    
    const cacheKey = `${this.REDIS_PREFIX}:count:${targetType}:${targetId}`;
    
    // Update cache
    const currentCount = await redis.get(cacheKey);
    if (currentCount) {
      const newCount = parseInt(currentCount, 10) + increment;
      await redis.setex(cacheKey, this.CACHE_TTL, newCount.toString());
    }
    
    // Here you would update the actual target's comment count
    // Example for posts:
    // await tx.post.update({
    //   where: { id: targetId },
    //   data: { commentsCount: { increment } },
    // });
  }
  
  private async getTargetPreview(
    tx: Prisma.TransactionClient,
    targetType: string,
    targetId: string
  ): Promise<any> {
    // Get preview data for the target
    // This would be different for each target type
    return null;
  }
  
  private async isTargetOwner(userId: string, targetType: string, targetId: string): Promise<boolean> {
    // Check if user owns the target content
    // Implementation depends on your data model
    return false;
  }
  
  private async clearCommentCaches(
    targetType: string,
    targetId: string,
    parentId?: string
  ): Promise<void> {
    const pipeline = redis.pipeline();
    
    // Clear target comment cache
    const pattern = `${this.REDIS_PREFIX}:${targetType}:${targetId}:*`;
    const keys = await redis.keys(pattern);
    keys.forEach(key => pipeline.del(key));
    
    // Clear count cache
    pipeline.del(`${this.REDIS_PREFIX}:count:${targetType}:${targetId}`);
    
    // Clear parent comment cache if exists
    if (parentId) {
      const parentPattern = `${this.REDIS_PREFIX}:*:${parentId}*`;
      const parentKeys = await redis.keys(parentPattern);
      parentKeys.forEach(key => pipeline.del(key));
    }
    
    await pipeline.exec();
  }
  
  private async notifyNewComment(comment: Comment, parentComment: Comment | null): Promise<void> {
    // Send real-time notification
    // This could be via WebSocket, Pusher, etc.
    
    if (parentComment && parentComment.authorId !== comment.authorId) {
      // Notify parent comment author
      // await notificationService.create({
      //   userId: parentComment.authorId,
      //   type: 'comment_reply',
      //   data: { commentId: comment.id },
      // });
    }
    
    // Notify target owner (if different from comment author)
    // Notify mentioned users
    // etc.
  }
}

§ 2.3 NESTED COMMENTS IMPLEMENTATION

typescript
// lib/services/comment-tree.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export interface CommentTreeNode {
  id: string;
  content: string;
  authorId: string;
  depth: number;
  path: string[];
  children: CommentTreeNode[];
  [key: string]: any;
}

export class CommentTreeService {
  /**
   * Build a tree structure from flat comments with path data
   */
  static buildTree(comments: any[]): CommentTreeNode[] {
    const map: Record<string, CommentTreeNode> = {};
    const roots: CommentTreeNode[] = [];
    
    // First pass: create map of id -> node
    comments.forEach(comment => {
      map[comment.id] = {
        ...comment,
        children: [],
      };
    });
    
    // Second pass: build tree
    comments.forEach(comment => {
      const node = map[comment.id];
      
      if (comment.parentId) {
        // Add to parent's children
        const parent = map[comment.parentId];
        if (parent) {
          parent.children.push(node);
          parent.children.sort((a, b) => 
            new Date(a.createdAt).getTime() - new Date(b.createdAt).getTime()
          );
        }
      } else {
        // Root comment
        roots.push(node);
      }
    });
    
    // Sort roots by creation date
    roots.sort((a, b) => 
      new Date(a.createdAt).getTime() - new Date(b.createdAt).getTime()
    );
    
    return roots;
  }
  
  /**
   * Get all descendants of a comment (flattened)
   */
  static async getCommentDescendants(commentId: string): Promise<string[]> {
    // Using PostgreSQL recursive query via Prisma raw query
    const result = await prisma.$queryRaw<{ id: string }[]>`
      WITH RECURSIVE comment_tree AS (
        SELECT id, parent_id
        FROM comments
        WHERE id = ${commentId}
        UNION ALL
        SELECT c.id, c.parent_id
        FROM comments c
        INNER JOIN comment_tree ct ON c.parent_id = ct.id
      )
      SELECT id FROM comment_tree WHERE id != ${commentId}
    `;
    
    return result.map(row => row.id);
  }
  
  /**
   * Calculate comment depth (distance from root)
   */
  static calculateDepth(path: string[]): number {
    return path.length;
  }
  
  /**
   * Get thread path for a comment
   */
  static async getThreadPath(commentId: string): Promise<string[]> {
    const comment = await prisma.comment.findUnique({
      where: { id: commentId },
      select: { path: true },
    });
    
    return comment?.path || [];
  }
  
  /**
   * Check if reply depth is within limits
   */
  static async canReplyTo(commentId: string, maxDepth: number = 10): Promise<boolean> {
    const comment = await prisma.comment.findUnique({
      where: { id: commentId },
      select: { depth: true },
    });
    
    if (!comment) return false;
    
    return comment.depth < maxDepth;
  }
  
  /**
   * Collapse deep threads for better UX
   */
  static collapseDeepThreads(tree: CommentTreeNode[], maxDepth: number = 5): CommentTreeNode[] {
    const processNode = (node: CommentTreeNode, currentDepth: number): CommentTreeNode => {
      if (currentDepth >= maxDepth && node.children.length > 0) {
        // Mark as collapsed
        return {
          ...node,
          children: [],
          _isCollapsed: true,
          _collapsedCount: this.countDescendants(node),
        };
      }
      
      return {
        ...node,
        children: node.children.map(child => 
          processNode(child, currentDepth + 1)
        ),
      };
    };
    
    return tree.map(root => processNode(root, 0));
  }
  
  /**
   * Count total descendants of a node
   */
  private static countDescendants(node: CommentTreeNode): number {
    let count = node.children.length;
    
    for (const child of node.children) {
      count += this.countDescendants(child);
    }
    
    return count;
  }
}

---

**NOTA:** Questo è solo il §2 (Comment System) della documentazione completa. Continuerei con le altre sezioni:

- §3: Likes & Reactions System
- §4: Follow System  
- §5: Activity Feed
- §6: Bookmarks/Saved Items
- §7: Mentions System
- §8: Sharing Features
- §9: Moderation & Spam Prevention
- §10: Real-time Updates
- §11: Complete Checklist

Il codice include:
- Modelli Prisma completi con tutte le relazioni
- Service layer con validazione Zod
- Error handling completo
- Rate limiting e spam prevention
- Caching con Redis
- Pattern per nested comments
- Type safety completa
- Transazioni per consistenza