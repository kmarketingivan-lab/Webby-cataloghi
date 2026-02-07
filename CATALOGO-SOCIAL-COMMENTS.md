# CATALOGO-SOCIAL-COMMENTS

ESPANSIONE CATALOGO-SOCIAL-COMMENTS per Next.js 14 + Prisma
§ PRISMA SCHEMA COMPLETO
prisma
// prisma/schema.prisma - Sezione Commenti

model User {
  id            String    @id @default(cuid())
  username      String    @unique
  displayName  String
  email        String    @unique
  image        String?
  bio          String?
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt
  
  // Relazioni
  comments     Comment[]
  replies      Reply[]
  reactions    Reaction[]
  mentions     Mention[]
  reports      Report[]
  commentBans  CommentBan[]
  
  @@map("users")
}

model Comment {
  id            String    @id @default(cuid())
  content       String    @db.Text
  htmlContent   String?   @db.Text
  rawContent    Json?     // Per editor avanzato
  parentId      String?   // Per commenti nidificati
  depth         Int       @default(0)
  path          String[]  // Array di IDs per tree structure
  postId        String?
  pageSlug      String?   // Per commenti su pagine statiche
  userId        String
  edited        Boolean   @default(false)
  editHistory   Json?     // Array di versioni precedenti
  deleted       Boolean   @default(false)
  deletedAt     DateTime?
  deleteReason  String?
  pinned        Boolean   @default(false)
  visibility    Visibility @default(PUBLIC)
  metadata      Json?     // Per dati aggiuntivi (link previews, attachments, etc.)
  
  // Stats
  replyCount    Int       @default(0)
  reactionCount Int       @default(0)
  score         Int       @default(0) // Per ranking
  
  // Timestamps
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  // Relazioni
  user          User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  replies       Reply[]
  reactions     Reaction[]
  mentions      Mention[]
  reports       Report[]
  
  // Self-relation per nested comments
  parent        Comment?  @relation("CommentHierarchy", fields: [parentId], references: [id])
  children      Comment[] @relation("CommentHierarchy")
  
  @@index([postId])
  @@index([userId])
  @@index([createdAt])
  @@index([path])
  @@index([score])
  @@map("comments")
}

model Reply {
  id            String    @id @default(cuid())
  content       String    @db.Text
  commentId     String
  userId        String
  edited        Boolean   @default(false)
  editHistory   Json?
  deleted       Boolean   @default(false)
  deletedAt     DateTime?
  
  // Timestamps
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  // Relazioni
  comment       Comment   @relation(fields: [commentId], references: [id], onDelete: Cascade)
  user          User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  reactions     Reaction[]
  
  @@index([commentId])
  @@index([userId])
  @@map("replies")
}

model Reaction {
  id          String     @id @default(cuid())
  type        ReactionType
  commentId   String?
  replyId     String?
  userId      String
  createdAt   DateTime   @default(now())
  
  // Relazioni
  comment     Comment?   @relation(fields: [commentId], references: [id], onDelete: Cascade)
  reply       Reply?     @relation(fields: [replyId], references: [id], onDelete: Cascade)
  user        User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([commentId, replyId, userId])
  @@index([commentId])
  @@index([replyId])
  @@index([userId])
  @@index([type])
  @@map("reactions")
}

model Mention {
  id           String   @id @default(cuid())
  commentId    String
  mentionedUserId String
  position     Int      // Posizione nel testo
  length       Int      // Lunghezza della menzione
  
  // Relazioni
  comment      Comment  @relation(fields: [commentId], references: [id], onDelete: Cascade)
  mentionedUser User    @relation("MentionedUser", fields: [mentionedUserId], references: [id], onDelete: Cascade)
  
  @@unique([commentId, mentionedUserId, position])
  @@index([mentionedUserId])
  @@map("mentions")
}

model Report {
  id          String     @id @default(cuid())
  commentId   String
  replyId     String?
  userId      String
  reason      ReportReason
  description String?
  status      ReportStatus @default(PENDING)
  resolvedAt  DateTime?
  resolvedBy  String?
  createdAt   DateTime   @default(now())
  
  // Relazioni
  comment     Comment?   @relation(fields: [commentId], references: [id], onDelete: Cascade)
  reply       Reply?     @relation(fields: [replyId], references: [id], onDelete: Cascade)
  user        User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@index([commentId])
  @@index([status])
  @@index([createdAt])
  @@map("reports")
}

model CommentBan {
  id          String    @id @default(cuid())
  userId      String
  bannedBy    String
  reason      String
  expiresAt   DateTime?
  permanent   Boolean   @default(false)
  createdAt   DateTime  @default(now())
  
  // Relazioni
  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  bannedByUser User     @relation("BannedByUser", fields: [bannedBy], references: [id])
  
  @@index([userId])
  @@index([expiresAt])
  @@map("comment_bans")
}

model ThreadSubscription {
  id          String    @id @default(cuid())
  userId      String
  commentId   String
  notifyOn    ThreadNotificationType[]
  createdAt   DateTime  @default(now())
  
  // Relazioni
  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  comment     Comment   @relation(fields: [commentId], references: [id], onDelete: Cascade)
  
  @@unique([userId, commentId])
  @@map("thread_subscriptions")
}

// Enums
enum Visibility {
  PUBLIC
  PRIVATE
  HIDDEN
  ARCHIVED
}

enum ReactionType {
  LIKE
  LOVE
  LAUGH
  SAD
  ANGRY
  THUMBS_UP
  THUMBS_DOWN
  CHECKMARK
  FIRE
  STAR
}

enum ReportReason {
  SPAM
  HARASSMENT
  HATE_SPEECH
  INAPPROPRIATE
  MISINFORMATION
  OTHER
}

enum ReportStatus {
  PENDING
  REVIEWED
  RESOLVED
  DISMISSED
}

enum ThreadNotificationType {
  NEW_REPLY
  NEW_REACTION
  MENTION
  ALL
}
§ TYPESCRIPT TYPES
typescript
// types/comments.ts
export type Comment = {
  id: string;
  content: string;
  htmlContent?: string;
  rawContent?: any;
  parentId?: string;
  depth: number;
  path: string[];
  postId?: string;
  pageSlug?: string;
  userId: string;
  edited: boolean;
  editHistory?: CommentEdit[];
  deleted: boolean;
  deletedAt?: Date;
  deleteReason?: string;
  pinned: boolean;
  visibility: Visibility;
  metadata?: CommentMetadata;
  replyCount: number;
  reactionCount: number;
  score: number;
  createdAt: Date;
  updatedAt: Date;
  
  // Relazioni
  user: User;
  replies?: Reply[];
  reactions?: Reaction[];
  mentions?: Mention[];
  children?: Comment[];
  parent?: Comment;
};

export type Reply = {
  id: string;
  content: string;
  commentId: string;
  userId: string;
  edited: boolean;
  editHistory?: any[];
  deleted: boolean;
  deletedAt?: Date;
  createdAt: Date;
  updatedAt: Date;
  
  user: User;
  reactions?: Reaction[];
};

export type Reaction = {
  id: string;
  type: ReactionType;
  commentId?: string;
  replyId?: string;
  userId: string;
  createdAt: Date;
  
  user?: User;
};

export type Mention = {
  id: string;
  commentId: string;
  mentionedUserId: string;
  position: number;
  length: number;
  
  mentionedUser: User;
};

export type Report = {
  id: string;
  commentId: string;
  replyId?: string;
  userId: string;
  reason: ReportReason;
  description?: string;
  status: ReportStatus;
  resolvedAt?: Date;
  resolvedBy?: string;
  createdAt: Date;
};

export type CommentBan = {
  id: string;
  userId: string;
  bannedBy: string;
  reason: string;
  expiresAt?: Date;
  permanent: boolean;
  createdAt: Date;
};

export type ThreadSubscription = {
  id: string;
  userId: string;
  commentId: string;
  notifyOn: ThreadNotificationType[];
  createdAt: Date;
};

export type CommentWithRelations = Comment & {
  user: User;
  reactions: Reaction[];
  replies: ReplyWithRelations[];
  mentions: MentionWithUser[];
  _count?: {
    replies: number;
    reactions: number;
  };
};

export type ReplyWithRelations = Reply & {
  user: User;
  reactions: Reaction[];
};

export type CommentMetadata = {
  attachments?: Attachment[];
  linkPreviews?: LinkPreview[];
  codeBlocks?: CodeBlock[];
  wordCount?: number;
  readingTime?: number;
};

export type Attachment = {
  id: string;
  url: string;
  type: 'image' | 'video' | 'file';
  name: string;
  size: number;
  thumbnailUrl?: string;
};

export type LinkPreview = {
  url: string;
  title?: string;
  description?: string;
  image?: string;
  siteName?: string;
};

export type CodeBlock = {
  language: string;
  code: string;
  startLine: number;
};

// Types per paginazione
export type PaginatedComments = {
  comments: CommentWithRelations[];
  total: number;
  page: number;
  totalPages: number;
  hasMore: boolean;
};

export type CommentsFilters = {
  page?: number;
  limit?: number;
  sort?: 'newest' | 'oldest' | 'top' | 'controversial';
  depth?: number;
  parentId?: string;
  userId?: string;
  search?: string;
};

// Types per real-time
export type CommentEvent = {
  type: 'NEW_COMMENT' | 'UPDATE_COMMENT' | 'DELETE_COMMENT' | 'NEW_REACTION' | 'TYPING';
  data: any;
  userId?: string;
  timestamp: Date;
};

export type TypingIndicator = {
  userId: string;
  username: string;
  commentId?: string;
  isTyping: boolean;
};
§ COMMENT SYSTEM COMPLETE
Utility Functions per Nested Comments
typescript
// utils/comment-tree.ts
export class CommentTreeBuilder {
  static buildTree(comments: CommentWithRelations[]): CommentWithRelations[] {
    const map = new Map<string, CommentWithRelations>();
    const roots: CommentWithRelations[] = [];

    // Inizializza mappa
    comments.forEach(comment => {
      comment.children = [];
      map.set(comment.id, comment);
    });

    // Costruisci albero
    comments.forEach(comment => {
      if (comment.parentId && map.has(comment.parentId)) {
        const parent = map.get(comment.parentId)!;
        parent.children!.push(comment);
      } else {
        roots.push(comment);
      }
    });

    // Ordina i commenti per score o data
    const sortComments = (comments: CommentWithRelations[]) => {
      comments.sort((a, b) => {
        if (b.pinned && !a.pinned) return 1;
        if (a.pinned && !b.pinned) return -1;
        return new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime();
      });
      
      comments.forEach(comment => {
        if (comment.children && comment.children.length > 0) {
          sortComments(comment.children);
        }
      });
    };

    sortComments(roots);
    return roots;
  }

  static calculateDepth(comment: Comment): number {
    return comment.path.length;
  }

  static buildPath(parentId: string | null, parentPath: string[] = []): string[] {
    if (!parentId) return [];
    return [...parentPath, parentId];
  }
}

// utils/pagination.ts
export class CommentPaginator {
  static async getPaginatedComments(
    prisma: any,
    filters: CommentsFilters
  ): Promise<PaginatedComments> {
    const page = filters.page || 1;
    const limit = filters.limit || 20;
    const skip = (page - 1) * limit;

    const where: any = {
      deleted: false,
      parentId: filters.parentId || null,
    };

    if (filters.postId) {
      where.postId = filters.postId;
    }

    if (filters.userId) {
      where.userId = filters.userId;
    }

    if (filters.search) {
      where.content = {
        contains: filters.search,
        mode: 'insensitive',
      };
    }

    // Ordinamento
    let orderBy: any = {};
    switch (filters.sort) {
      case 'oldest':
        orderBy = { createdAt: 'asc' };
        break;
      case 'top':
        orderBy = { score: 'desc' };
        break;
      case 'controversial':
        // Logica per commenti controversi (alta reazione, ma mix positivo/negativo)
        orderBy = [
          { reactionCount: 'desc' },
          { createdAt: 'desc' }
        ];
        break;
      default: // 'newest'
        orderBy = { createdAt: 'desc' };
    }

    const [comments, total] = await Promise.all([
      prisma.comment.findMany({
        where,
        include: {
          user: {
            select: {
              id: true,
              username: true,
              displayName: true,
              image: true,
            },
          },
          reactions: true,
          mentions: {
            include: {
              mentionedUser: {
                select: {
                  id: true,
                  username: true,
                  displayName: true,
                },
              },
            },
          },
          _count: {
            select: {
              replies: true,
              reactions: true,
            },
          },
        },
        orderBy,
        skip,
        take: limit,
      }),
      prisma.comment.count({ where }),
    ]);

    const totalPages = Math.ceil(total / limit);

    return {
      comments,
      total,
      page,
      totalPages,
      hasMore: page < totalPages,
    };
  }

  // Paginazione per nested comments
  static async getNestedComments(
    prisma: any,
    parentId: string,
    filters: { page?: number; limit?: number } = {}
  ) {
    const page = filters.page || 1;
    const limit = filters.limit || 10;
    const skip = (page - 1) * limit;

    const [comments, total] = await Promise.all([
      prisma.comment.findMany({
        where: {
          parentId,
          deleted: false,
        },
        include: {
          user: true,
          reactions: true,
          _count: {
            select: {
              children: true,
              reactions: true,
            },
          },
        },
        orderBy: {
          score: 'desc',
        },
        skip,
        take: limit,
      }),
      prisma.comment.count({
        where: {
          parentId,
          deleted: false,
        },
      }),
    ]);

    return {
      comments,
      total,
      page,
      hasMore: skip + comments.length < total,
    };
  }
}
API Routes per Comment System
typescript
// app/api/comments/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';
import { CommentPaginator } from '@/utils/pagination';
import { validateComment } from '@/utils/validation';

export async function GET(request: NextRequest) {
  try {
    const searchParams = request.nextUrl.searchParams;
    const filters = {
      postId: searchParams.get('postId'),
      pageSlug: searchParams.get('pageSlug'),
      page: parseInt(searchParams.get('page') || '1'),
      limit: parseInt(searchParams.get('limit') || '20'),
      sort: searchParams.get('sort') as 'newest' | 'oldest' | 'top' | 'controversial',
      parentId: searchParams.get('parentId'),
      userId: searchParams.get('userId'),
      search: searchParams.get('search'),
    };

    const where: any = {
      deleted: false,
      parentId: filters.parentId || null,
    };

    if (filters.postId) {
      where.postId = filters.postId;
    }

    if (filters.pageSlug) {
      where.pageSlug = filters.pageSlug;
    }

    if (filters.userId) {
      where.userId = filters.userId;
    }

    const result = await CommentPaginator.getPaginatedComments(prisma, filters);
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Error fetching comments:', error);
    return NextResponse.json(
      { error: 'Failed to fetch comments' },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  const session = await getServerSession();
  if (!session?.user?.email) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  try {
    const body = await request.json();
    const { content, postId, pageSlug, parentId, rawContent } = body;

    // Validazione
    const validation = validateComment(content);
    if (!validation.valid) {
      return NextResponse.json(
        { error: validation.message },
        { status: 400 }
      );
    }

    // Controlla se l'utente è bannato
    const user = await prisma.user.findUnique({
      where: { email: session.user.email },
    });

    if (!user) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      );
    }

    const activeBan = await prisma.commentBan.findFirst({
      where: {
        userId: user.id,
        OR: [
          { permanent: true },
          { expiresAt: { gt: new Date() } },
        ],
      },
    });

    if (activeBan) {
      return NextResponse.json(
        { error: 'You are banned from commenting' },
        { status: 403 }
      );
    }

    // Calcola path per nested comments
    let path: string[] = [];
    let depth = 0;

    if (parentId) {
      const parentComment = await prisma.comment.findUnique({
        where: { id: parentId },
      });

      if (parentComment) {
        depth = parentComment.depth + 1;
        path = [...parentComment.path, parentId];
      }
    }

    // Processa menzioni
    const mentions = await processMentions(content, user.id);

    // Processa contenuto ricco
    const processedContent = await processRichContent(content, rawContent);

    const comment = await prisma.comment.create({
      data: {
        content: processedContent.text,
        htmlContent: processedContent.html,
        rawContent,
        parentId,
        depth,
        path,
        postId,
        pageSlug,
        userId: user.id,
        metadata: processedContent.metadata,
        mentions: {
          create: mentions,
        },
      },
      include: {
        user: {
          select: {
            id: true,
            username: true,
            displayName: true,
            image: true,
          },
        },
        mentions: {
          include: {
            mentionedUser: {
              select: {
                id: true,
                username: true,
                displayName: true,
              },
            },
          },
        },
      },
    });

    // Invia notifiche per menzioni
    await sendMentionNotifications(mentions, comment.id, user.id);

    // Aggiorna il conteggio dei reply se è una risposta
    if (parentId) {
      await prisma.comment.update({
        where: { id: parentId },
        data: {
          replyCount: { increment: 1 },
        },
      });
    }

    return NextResponse.json(comment, { status: 201 });
  } catch (error) {
    console.error('Error creating comment:', error);
    return NextResponse.json(
      { error: 'Failed to create comment' },
      { status: 500 }
    );
  }
}
§ COMMENT CRUD COMPLETE
typescript
// app/api/comments/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const comment = await prisma.comment.findUnique({
      where: { id: params.id, deleted: false },
      include: {
        user: {
          select: {
            id: true,
            username: true,
            displayName: true,
            image: true,
            bio: true,
          },
        },
        reactions: {
          include: {
            user: {
              select: {
                id: true,
                username: true,
                displayName: true,
                image: true,
              },
            },
          },
        },
        replies: {
          where: { deleted: false },
          include: {
            user: {
              select: {
                id: true,
                username: true,
                displayName: true,
                image: true,
              },
            },
            reactions: true,
          },
          orderBy: { createdAt: 'asc' },
          take: 10,
        },
        mentions: {
          include: {
            mentionedUser: {
              select: {
                id: true,
                username: true,
                displayName: true,
              },
            },
          },
        },
        parent: {
          include: {
            user: {
              select: {
                id: true,
                username: true,
                displayName: true,
              },
            },
          },
        },
        children: {
          where: { deleted: false },
          include: {
            user: {
              select: {
                id: true,
                username: true,
                displayName: true,
              },
            },
            _count: {
              select: {
                children: true,
                reactions: true,
              },
            },
          },
          orderBy: { score: 'desc' },
          take: 5,
        },
        _count: {
          select: {
            replies: true,
            reactions: true,
            children: true,
          },
        },
      },
    });

    if (!comment) {
      return NextResponse.json(
        { error: 'Comment not found' },
        { status: 404 }
      );
    }

    return NextResponse.json(comment);
  } catch (error) {
    console.error('Error fetching comment:', error);
    return NextResponse.json(
      { error: 'Failed to fetch comment' },
      { status: 500 }
    );
  }
}

export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const session = await getServerSession();
  if (!session?.user?.email) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  try {
    const user = await prisma.user.findUnique({
      where: { email: session.user.email },
    });

    if (!user) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      );
    }

    const comment = await prisma.comment.findUnique({
      where: { id: params.id },
    });

    if (!comment) {
      return NextResponse.json(
        { error: 'Comment not found' },
        { status: 404 }
      );
    }

    // Controlla permessi
    if (comment.userId !== user.id) {
      return NextResponse.json(
        { error: 'You can only edit your own comments' },
        { status: 403 }
      );
    }

    const body = await request.json();
    const { content, rawContent } = body;

    // Salva la versione precedente nella cronologia
    const editHistory = comment.editHistory || [];
    editHistory.push({
      content: comment.content,
      htmlContent: comment.htmlContent,
      rawContent: comment.rawContent,
      editedAt: new Date(),
    });

    // Limita la cronologia a 10 versioni
    if (editHistory.length > 10) {
      editHistory.shift();
    }

    // Processa nuovo contenuto
    const processedContent = await processRichContent(content, rawContent);

    const updatedComment = await prisma.comment.update({
      where: { id: params.id },
      data: {
        content: processedContent.text,
        htmlContent: processedContent.html,
        rawContent,
        edited: true,
        editHistory,
        metadata: processedContent.metadata,
        updatedAt: new Date(),
      },
      include: {
        user: {
          select: {
            id: true,
            username: true,
            displayName: true,
            image: true,
          },
        },
      },
    });

    return NextResponse.json(updatedComment);
  } catch (error) {
    console.error('Error updating comment:', error);
    return NextResponse.json(
      { error: 'Failed to update comment' },
      { status: 500 }
    );
  }
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const session = await getServerSession();
  if (!session?.user?.email) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  try {
    const user = await prisma.user.findUnique({
      where: { email: session.user.email },
    });

    if (!user) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      );
    }

    const comment = await prisma.comment.findUnique({
      where: { id: params.id },
    });

    if (!comment) {
      return NextResponse.json(
        { error: 'Comment not found' },
        { status: 404 }
      );
    }

    // Controlla permessi (utente o admin)
    const isAdmin = user.role === 'ADMIN';
    if (comment.userId !== user.id && !isAdmin) {
      return NextResponse.json(
        { error: 'You can only delete your own comments' },
        { status: 403 }
      );
    }

    const { reason, hardDelete } = await request.json();

    if (hardDelete && isAdmin) {
      // Eliminazione permanente (solo admin)
      await prisma.comment.delete({
        where: { id: params.id },
      });
    } else {
      // Soft delete
      await prisma.comment.update({
        where: { id: params.id },
        data: {
          deleted: true,
          deletedAt: new Date(),
          deleteReason: reason || 'Deleted by user',
          content: '[This comment has been deleted]',
          htmlContent: '<p>[This comment has been deleted]</p>',
          rawContent: null,
        },
      });
    }

    return NextResponse.json({ success: true });
  } catch (error) {
    console.error('Error deleting comment:', error);
    return NextResponse.json(
      { error: 'Failed to delete comment' },
      { status: 500 }
    );
  }
}
API per Replies
typescript
// app/api/comments/[id]/replies/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const searchParams = request.nextUrl.searchParams;
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '10');
    const skip = (page - 1) * limit;

    const [replies, total] = await Promise.all([
      prisma.reply.findMany({
        where: {
          commentId: params.id,
          deleted: false,
        },
        include: {
          user: {
            select: {
              id: true,
              username: true,
              displayName: true,
              image: true,
            },
          },
          reactions: {
            include: {
              user: {
                select: {
                  id: true,
                  username: true,
                },
              },
            },
          },
        },
        orderBy: { createdAt: 'asc' },
        skip,
        take: limit,
      }),
      prisma.reply.count({
        where: {
          commentId: params.id,
          deleted: false,
        },
      }),
    ]);

    return NextResponse.json({
      replies,
      total,
      page,
      totalPages: Math.ceil(total / limit),
      hasMore: skip + replies.length < total,
    });
  } catch (error) {
    console.error('Error fetching replies:', error);
    return NextResponse.json(
      { error: 'Failed to fetch replies' },
      { status: 500 }
    );
  }
}

export async function POST(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const session = await getServerSession();
  if (!session?.user?.email) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  try {
    const user = await prisma.user.findUnique({
      where: { email: session.user.email },
    });

    if (!user) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      );
    }

    const body = await request.json();
    const { content } = body;

    const reply = await prisma.reply.create({
      data: {
        content,
        commentId: params.id,
        userId: user.id,
      },
      include: {
        user: {
          select: {
            id: true,
            username: true,
            displayName: true,
            image: true,
          },
        },
      },
    });

    // Incrementa reply count sul commento
    await prisma.comment.update({
      where: { id: params.id },
      data: {
        replyCount: { increment: 1 },
      },
    });

    return NextResponse.json(reply, { status: 201 });
  } catch (error) {
    console.error('Error creating reply:', error);
    return NextResponse.json(
      { error: 'Failed to create reply' },
      { status: 500 }
    );
  }
}
§ REACTIONS SYSTEM
typescript
// lib/reactions.ts
export class ReactionService {
  static async toggleReaction(
    userId: string,
    targetId: string,
    type: ReactionType,
    targetType: 'comment' | 'reply'
  ) {
    const existingReaction = await prisma.reaction.findFirst({
      where: {
        userId,
        [targetType === 'comment' ? 'commentId' : 'replyId']: targetId,
      },
    });

    if (existingReaction) {
      if (existingReaction.type === type) {
        // Rimuovi reaction
        await prisma.reaction.delete({
          where: { id: existingReaction.id },
        });

        // Decrementa contatore
        await this.updateReactionCount(targetId, targetType, -1);
        
        return { action: 'removed', reaction: null };
      } else {
        // Cambia tipo di reaction
        const updatedReaction = await prisma.reaction.update({
          where: { id: existingReaction.id },
          data: { type },
        });

        return { action: 'updated', reaction: updatedReaction };
      }
    } else {
      // Crea nuova reaction
      const reactionData: any = {
        type,
        userId,
      };
      
      if (targetType === 'comment') {
        reactionData.commentId = targetId;
      } else {
        reactionData.replyId = targetId;
      }

      const newReaction = await prisma.reaction.create({
        data: reactionData,
      });

      // Incrementa contatore
      await this.updateReactionCount(targetId, targetType, 1);

      return { action: 'added', reaction: newReaction };
    }
  }

  private static async updateReactionCount(
    targetId: string,
    targetType: 'comment' | 'reply',
    increment: number
  ) {
    if (targetType === 'comment') {
      await prisma.comment.update({
        where: { id: targetId },
        data: {
          reactionCount: { increment },
          // Aggiorna score (logica personalizzabile)
          score: { increment: increment > 0 ? 1 : -1 },
        },
      });
    } else {
      // Per replies, se necessario
    }
  }

  static async getReactionSummary(targetId: string, targetType: 'comment' | 'reply') {
    const reactions = await prisma.reaction.groupBy({
      by: ['type'],
      where: {
        [targetType === 'comment' ? 'commentId' : 'replyId']: targetId,
      },
      _count: {
        type: true,
      },
    });

    return reactions.reduce((acc, curr) => {
      acc[curr.type] = curr._count.type;
      return acc;
    }, {} as Record<ReactionType, number>);
  }

  static async getReactionsWithUsers(
    targetId: string,
    targetType: 'comment' | 'reply',
    type?: ReactionType
  ) {
    const where: any = {
      [targetType === 'comment' ? 'commentId' : 'replyId']: targetId,
    };

    if (type) {
      where.type = type;
    }

    return prisma.reaction.findMany({
      where,
      include: {
        user: {
          select: {
            id: true,
            username: true,
            displayName: true,
            image: true,
          },
        },
      },
      orderBy: { createdAt: 'desc' },
      take: 50,
    });
  }
}

// Componente React per Reactions
// components/reactions/ReactionButtons.tsx
'use client';

import { useState } from 'react';
import { ReactionType } from '@/types/comments';
import { useOptimistic } from 'react';

interface ReactionButtonsProps {
  targetId: string;
  targetType: 'comment' | 'reply';
  initialReactions: Record<ReactionType, number>;
  userReaction?: ReactionType | null;
}

const REACTION_TYPES: { type: ReactionType; label: string; emoji: string }[] = [
  { type: 'LIKE', label: 'Like', emoji: '👍' },
  { type: 'LOVE', label: 'Love', emoji: '❤️' },
  { type: 'LAUGH', label: 'Laugh', emoji: '😂' },
  { type: 'SAD', label: 'Sad', emoji: '😢' },
  { type: 'ANGRY', label: 'Angry', emoji: '😠' },
  { type: 'FIRE', label: 'Fire', emoji: '🔥' },
  { type: 'STAR', label: 'Star', emoji: '⭐' },
];

export function ReactionButtons({
  targetId,
  targetType,
  initialReactions,
  userReaction,
}: ReactionButtonsProps) {
  const [reactions, setReactions] = useState(initialReactions);
  const [currentReaction, setCurrentReaction] = useState(userReaction);
  const [isLoading, setIsLoading] = useState(false);

  const [optimisticReactions, addOptimisticReaction] = useOptimistic(
    { reactions, currentReaction },
    (state, newReaction: { type: ReactionType; action: 'add' | 'remove' }) => {
      const updated = { ...state.reactions };
      
      if (newReaction.action === 'add') {
        updated

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-29-SOCIAL-COMMENTS
Prompt ID: 29 / 48
Parte: 2
Exported: 2026-02-06T12:10:05.613Z
Characters: 863
════════════════════════════════════════════════════════════

rom '@/types/comments';

interface ModerationQueueProps {
  initialReports: Report[];
}

export function ModerationQueue({ initialReports }: ModerationQueueProps) {
  const [reports, setReports] = useState(initialReports);
  const [selectedStatus, setSelectedStatus] = useState<ReportStatus>('PENDING');
  const [isLoading, setIsLoading] = useState(false);

  const loadReports = async (status: ReportStatus) => {
    setIsLoading(true);
    try {
      const response = await fetch(`/api/moderation/reports?status=${status}`);
      if (response.ok) {
        const data = await response.json();
        setReports(data.reports);
      }
    } catch (error) {
      console.error('Error loading reports:', error);
    } finally {
      setIsLoading(false);
    }
  };

  const handleStatusChange = async (status: ReportStatus) => {
    setSelectedStatus(status);