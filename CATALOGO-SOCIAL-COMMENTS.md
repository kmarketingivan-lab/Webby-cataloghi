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

## § ADVANCED PATTERNS: SOCIAL COMMENTS

### Server Actions con Validazione

```typescript
// app/actions/social-comments.ts
"use server";

import { z } from "zod";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

const ItemSchema = z.object({
  name: z.string().min(2).max(100),
  description: z.string().min(10).max(5000),
  category: z.enum(["general", "premium", "enterprise"]),
  price: z.number().positive().max(999999),
  metadata: z.record(z.string(), z.unknown()).optional(),
  tags: z.array(z.string().max(50)).max(10).optional(),
  isActive: z.boolean().default(true),
});

type ItemInput = z.infer<typeof ItemSchema>;

interface ActionResult<T = unknown> {
  success: boolean;
  data?: T;
  error?: string;
  fieldErrors?: Record<string, string[]>;
}

export async function createItem(formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.safeParse(raw);
  if (!validation.success) {
    return {
      success: false,
      error: "Validation failed",
      fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]>,
    };
  }

  try {
    const { db } = await import("@/lib/db");
    const item = await db.insert("items").values({
      ...validation.data,
      createdAt: new Date(),
      updatedAt: new Date(),
    }).returning();

    revalidatePath("/items");
    return { success: true, data: item[0] };
  } catch (error) {
    console.error("Failed to create item:", error);
    return { success: false, error: "Failed to create item. Please try again." };
  }
}

export async function updateItem(id: string, formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.partial().safeParse(raw);
  if (!validation.success) {
    return { success: false, error: "Validation failed", fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]> };
  }

  try {
    const { db } = await import("@/lib/db");
    const updated = await db.update("items")
      .set({ ...validation.data, updatedAt: new Date() })
      .where({ id })
      .returning();

    if (!updated[0]) return { success: false, error: "Item not found" };

    revalidatePath("/items");
    revalidatePath(`/items/${id}`);
    return { success: true, data: updated[0] };
  } catch (error) {
    console.error("Failed to update item:", error);
    return { success: false, error: "Failed to update item" };
  }
}

export async function deleteItem(id: string): Promise<ActionResult> {
  try {
    const { db } = await import("@/lib/db");
    await db.update("items").set({ deletedAt: new Date() }).where({ id });
    revalidatePath("/items");
    return { success: true };
  } catch (error) {
    console.error("Failed to delete item:", error);
    return { success: false, error: "Failed to delete item" };
  }
}

export async function bulkUpdateItems(
  ids: string[],
  updates: Partial<ItemInput>
): Promise<ActionResult<{ updated: number }>> {
  if (ids.length === 0) return { success: false, error: "No items selected" };
  if (ids.length > 100) return { success: false, error: "Maximum 100 items at once" };

  try {
    const { db } = await import("@/lib/db");
    let updatedCount = 0;
    for (const id of ids) {
      const result = await db.update("items").set({ ...updates, updatedAt: new Date() }).where({ id }).returning();
      if (result[0]) updatedCount++;
    }
    revalidatePath("/items");
    return { success: true, data: { updated: updatedCount } };
  } catch (error) {
    console.error("Bulk update failed:", error);
    return { success: false, error: "Bulk update failed" };
  }
}
```

### Hook useOptimisticList

```typescript
// hooks/useOptimisticList.ts
"use client";

import { useOptimistic, useTransition, useCallback, useState } from "react";

interface ListItem {
  id: string;
  [key: string]: unknown;
}

type OptimisticAction<T> =
  | { type: "add"; item: T }
  | { type: "update"; id: string; updates: Partial<T> }
  | { type: "remove"; id: string }
  | { type: "reorder"; fromIndex: number; toIndex: number };

export function useOptimisticList<T extends ListItem>(initialItems: T[]) {
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState<string | null>(null);

  const [optimisticItems, dispatch] = useOptimistic<T[], OptimisticAction<T>>(
    initialItems,
    (state, action) => {
      switch (action.type) {
        case "add":
          return [...state, action.item];
        case "update":
          return state.map((item) =>
            item.id === action.id ? { ...item, ...action.updates } : item
          );
        case "remove":
          return state.filter((item) => item.id !== action.id);
        case "reorder": {
          const newState = [...state];
          const [moved] = newState.splice(action.fromIndex, 1);
          newState.splice(action.toIndex, 0, moved);
          return newState;
        }
        default:
          return state;
      }
    }
  );

  const addOptimistic = useCallback(
    (item: T, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "add", item });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to add item");
        }
      });
    },
    [dispatch]
  );

  const updateOptimistic = useCallback(
    (id: string, updates: Partial<T>, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "update", id, updates });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to update item");
        }
      });
    },
    [dispatch]
  );

  const removeOptimistic = useCallback(
    (id: string, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "remove", id });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to remove item");
        }
      });
    },
    [dispatch]
  );

  return {
    items: optimisticItems,
    isPending,
    error,
    addOptimistic,
    updateOptimistic,
    removeOptimistic,
  };
}
```

### Data Table con Sorting, Filtering e Pagination

```typescript
// components/DataTable.tsx
"use client";

import { useState, useMemo, useCallback } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import {
  ChevronLeft,
  ChevronRight,
  ChevronsLeft,
  ChevronsRight,
  ArrowUpDown,
  ArrowUp,
  ArrowDown,
  Search,
  X,
} from "lucide-react";

interface Column<T> {
  key: keyof T & string;
  label: string;
  sortable?: boolean;
  filterable?: boolean;
  render?: (value: T[keyof T], row: T) => React.ReactNode;
  width?: string;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  pageSize?: number;
  searchable?: boolean;
  selectable?: boolean;
  onRowClick?: (row: T) => void;
  onSelectionChange?: (selected: T[]) => void;
  emptyMessage?: string;
}

export function DataTable<T extends { id: string }>({
  data,
  columns,
  pageSize = 10,
  searchable = true,
  selectable = false,
  onRowClick,
  onSelectionChange,
  emptyMessage = "No data found",
}: DataTableProps<T>) {
  const [currentPage, setCurrentPage] = useState(1);
  const [sortKey, setSortKey] = useState<string | null>(null);
  const [sortDirection, setSortDirection] = useState<"asc" | "desc">("asc");
  const [searchQuery, setSearchQuery] = useState("");
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());
  const [filters, setFilters] = useState<Record<string, string>>({});

  const filteredData = useMemo(() => {
    let result = [...data];

    if (searchQuery) {
      const query = searchQuery.toLowerCase();
      result = result.filter((row) =>
        columns.some((col) => {
          const value = row[col.key];
          return value !== null && value !== undefined && String(value).toLowerCase().includes(query);
        })
      );
    }

    for (const [key, filterValue] of Object.entries(filters)) {
      if (!filterValue) continue;
      result = result.filter((row) => String(row[key as keyof T]).toLowerCase().includes(filterValue.toLowerCase()));
    }

    if (sortKey) {
      result.sort((a, b) => {
        const aVal = a[sortKey as keyof T];
        const bVal = b[sortKey as keyof T];
        if (aVal === bVal) return 0;
        if (aVal === null || aVal === undefined) return 1;
        if (bVal === null || bVal === undefined) return -1;
        const comparison = aVal < bVal ? -1 : 1;
        return sortDirection === "asc" ? comparison : -comparison;
      });
    }
    return result;
  }, [data, searchQuery, filters, sortKey, sortDirection, columns]);

  const totalPages = Math.ceil(filteredData.length / pageSize);
  const paginatedData = filteredData.slice((currentPage - 1) * pageSize, currentPage * pageSize);

  const handleSort = useCallback((key: string) => {
    if (sortKey === key) {
      setSortDirection((prev) => (prev === "asc" ? "desc" : "asc"));
    } else {
      setSortKey(key);
      setSortDirection("asc");
    }
    setCurrentPage(1);
  }, [sortKey]);

  const toggleSelection = useCallback((id: string) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id); else next.add(id);
      if (onSelectionChange) {
        onSelectionChange(data.filter((item) => next.has(item.id)));
      }
      return next;
    });
  }, [data, onSelectionChange]);

  const toggleAll = useCallback(() => {
    const allIds = paginatedData.map((item) => item.id);
    const allSelected = allIds.every((id) => selectedIds.has(id));
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (allSelected) { allIds.forEach((id) => next.delete(id)); }
      else { allIds.forEach((id) => next.add(id)); }
      if (onSelectionChange) { onSelectionChange(data.filter((item) => next.has(item.id))); }
      return next;
    });
  }, [paginatedData, selectedIds, data, onSelectionChange]);

  const SortIcon = ({ columnKey }: { columnKey: string }) => {
    if (sortKey !== columnKey) return <ArrowUpDown className="h-3 w-3 ml-1 opacity-50" />;
    return sortDirection === "asc" ? <ArrowUp className="h-3 w-3 ml-1" /> : <ArrowDown className="h-3 w-3 ml-1" />;
  };

  return (
    <Card>
      <CardHeader className="pb-3">
        <div className="flex items-center justify-between">
          <CardTitle className="text-lg">
            {filteredData.length} result{filteredData.length !== 1 ? "s" : ""}
          </CardTitle>
          {searchable && (
            <div className="relative w-64">
              <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" />
              <Input
                placeholder="Search..."
                value={searchQuery}
                onChange={(e) => { setSearchQuery(e.target.value); setCurrentPage(1); }}
                className="pl-9 pr-9"
              />
              {searchQuery && (
                <button onClick={() => setSearchQuery("")} className="absolute right-3 top-1/2 -translate-y-1/2">
                  <X className="h-4 w-4 text-muted-foreground" />
                </button>
              )}
            </div>
          )}
        </div>
        {selectedIds.size > 0 && (
          <div className="flex items-center gap-2 mt-2">
            <Badge variant="secondary">{selectedIds.size} selected</Badge>
            <Button variant="ghost" size="sm" onClick={() => setSelectedIds(new Set())}>Clear</Button>
          </div>
        )}
      </CardHeader>
      <CardContent>
        <div className="overflow-x-auto">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b bg-muted/50">
                {selectable && (
                  <th className="w-10 py-3 px-3">
                    <input type="checkbox" onChange={toggleAll}
                      checked={paginatedData.length > 0 && paginatedData.every((item) => selectedIds.has(item.id))} />
                  </th>
                )}
                {columns.map((col) => (
                  <th key={col.key} className="text-left py-3 px-3 font-medium" style={{ width: col.width }}>
                    {col.sortable ? (
                      <button onClick={() => handleSort(col.key)} className="flex items-center hover:text-foreground">
                        {col.label} <SortIcon columnKey={col.key} />
                      </button>
                    ) : col.label}
                  </th>
                ))}
              </tr>
            </thead>
            <tbody>
              {paginatedData.length === 0 ? (
                <tr><td colSpan={columns.length + (selectable ? 1 : 0)} className="text-center py-12 text-muted-foreground">{emptyMessage}</td></tr>
              ) : (
                paginatedData.map((row) => (
                  <tr key={row.id} onClick={() => onRowClick?.(row)}
                    className={`border-b transition-colors hover:bg-muted/50 ${onRowClick ? "cursor-pointer" : ""} ${selectedIds.has(row.id) ? "bg-primary/5" : ""}`}>
                    {selectable && (
                      <td className="py-3 px-3" onClick={(e) => e.stopPropagation()}>
                        <input type="checkbox" checked={selectedIds.has(row.id)} onChange={() => toggleSelection(row.id)} />
                      </td>
                    )}
                    {columns.map((col) => (
                      <td key={col.key} className="py-3 px-3">
                        {col.render ? col.render(row[col.key], row) : String(row[col.key] ?? "")}
                      </td>
                    ))}
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4">
            <p className="text-sm text-muted-foreground">
              Page {currentPage} of {totalPages}
            </p>
            <div className="flex items-center gap-1">
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(1)} disabled={currentPage === 1}>
                <ChevronsLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p - 1)} disabled={currentPage === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p + 1)} disabled={currentPage === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(totalPages)} disabled={currentPage === totalPages}>
                <ChevronsRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

### API Route con Middleware Pattern

```typescript
// lib/api/middleware.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";

type Handler = (
  req: NextRequest,
  context: { params: Record<string, string>; user?: { id: string; role: string } }
) => Promise<NextResponse>;

type Middleware = (handler: Handler) => Handler;

export function withValidation<T>(schema: z.ZodSchema<T>, source: "body" | "query" = "body"): Middleware {
  return (handler) => async (req, context) => {
    try {
      let data: unknown;
      if (source === "body") {
        data = await req.json();
      } else {
        const searchParams = Object.fromEntries(req.nextUrl.searchParams);
        data = searchParams;
      }
      const parsed = schema.parse(data);
      (req as any).validated = parsed;
      return handler(req, context);
    } catch (error) {
      if (error instanceof z.ZodError) {
        return NextResponse.json(
          { error: "Validation failed", details: error.errors.map((e) => ({ path: e.path.join("."), message: e.message })) },
          { status: 400 }
        );
      }
      return NextResponse.json({ error: "Invalid request body" }, { status: 400 });
    }
  };
}

export function withAuth(requiredRole?: string): Middleware {
  return (handler) => async (req, context) => {
    const authHeader = req.headers.get("authorization");
    if (!authHeader?.startsWith("Bearer ")) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const token = authHeader.slice(7);
    try {
      const { verifyToken } = await import("@/lib/auth");
      const user = await verifyToken(token);
      if (!user) return NextResponse.json({ error: "Invalid token" }, { status: 401 });
      if (requiredRole && user.role !== requiredRole && user.role !== "admin") {
        return NextResponse.json({ error: "Forbidden" }, { status: 403 });
      }
      context.user = user;
      return handler(req, context);
    } catch {
      return NextResponse.json({ error: "Authentication failed" }, { status: 401 });
    }
  };
}

export function withRateLimit(maxRequests: number = 60, windowMs: number = 60000): Middleware {
  const requests = new Map<string, { count: number; resetAt: number }>();

  return (handler) => async (req, context) => {
    const ip = req.headers.get("x-forwarded-for") || req.headers.get("x-real-ip") || "unknown";
    const now = Date.now();
    const entry = requests.get(ip);

    if (!entry || now > entry.resetAt) {
      requests.set(ip, { count: 1, resetAt: now + windowMs });
    } else if (entry.count >= maxRequests) {
      return NextResponse.json(
        { error: "Too many requests" },
        {
          status: 429,
          headers: {
            "X-RateLimit-Limit": maxRequests.toString(),
            "X-RateLimit-Remaining": "0",
            "X-RateLimit-Reset": new Date(entry.resetAt).toISOString(),
            "Retry-After": Math.ceil((entry.resetAt - now) / 1000).toString(),
          },
        }
      );
    } else {
      entry.count++;
    }

    return handler(req, context);
  };
}

export function withErrorHandler(): Middleware {
  return (handler) => async (req, context) => {
    try {
      return await handler(req, context);
    } catch (error) {
      console.error(`[API Error] ${req.method} ${req.url}:`, error);

      if (error instanceof z.ZodError) {
        return NextResponse.json({ error: "Validation error", details: error.errors }, { status: 400 });
      }

      const message = error instanceof Error ? error.message : "Internal server error";
      const status = (error as any).status || 500;
      return NextResponse.json({ error: message }, { status });
    }
  };
}

export function compose(...middlewares: Middleware[]): Middleware {
  return (handler) => {
    let composed = handler;
    for (let i = middlewares.length - 1; i >= 0; i--) {
      composed = middlewares[i](composed);
    }
    return composed;
  };
}

// Esempio d'uso:
// const handler = compose(withErrorHandler(), withAuth("admin"), withRateLimit(30))(async (req, ctx) => {
//   const items = await db.findMany("items", { userId: ctx.user!.id });
//   return NextResponse.json({ items });
// });
```


### SOCIAL COMMENTS - Utility Helper #178

```typescript
// lib/utils/social-comments-helper-178.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #523

```typescript
// lib/utils/social-comments-helper-523.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #363

```typescript
// lib/utils/social-comments-helper-363.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #419

```typescript
// lib/utils/social-comments-helper-419.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #111

```typescript
// lib/utils/social-comments-helper-111.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #437

```typescript
// lib/utils/social-comments-helper-437.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #279

```typescript
// lib/utils/social-comments-helper-279.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #159

```typescript
// lib/utils/social-comments-helper-159.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #83

```typescript
// lib/utils/social-comments-helper-83.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #357

```typescript
// lib/utils/social-comments-helper-357.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #800

```typescript
// lib/utils/social-comments-helper-800.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #304

```typescript
// lib/utils/social-comments-helper-304.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #848

```typescript
// lib/utils/social-comments-helper-848.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #692

```typescript
// lib/utils/social-comments-helper-692.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #412

```typescript
// lib/utils/social-comments-helper-412.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL COMMENTS - Utility Helper #879

```typescript
// lib/utils/social-comments-helper-879.ts
import { z } from "zod";

interface SOCIALCOMMENTSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class SOCIALCOMMENTSProcessor<TInput, TOutput> {
  private config: SOCIALCOMMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALCOMMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<SOCIALCOMMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```
