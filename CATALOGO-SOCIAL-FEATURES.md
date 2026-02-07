# CATALOGO-SOCIAL-FEATURES

CATALOGO-SOCIAL-FEATURES per Next.js 14 + Prisma - ESPANSIONE
§ FOLLOW SYSTEM
Modello Prisma per Follow System
prisma
// schema.prisma
model User {
  id                String    @id @default(cuid())
  // ... altri campi utente
  
  // Relazioni Follow
  followers        Follow[]  @relation("Followers", references: [id])
  following        Follow[]  @relation("Following", references: [id])
  blockedUsers     Block[]   @relation("BlockedBy", references: [id])
  blockingUsers    Block[]   @relation("Blocking", references: [id])
  
  // Follow suggestions
  suggestedFollows SuggestedFollow[]
  
  // Statistiche
  followersCount   Int       @default(0)
  followingCount   Int       @default(0)
  
  // Privacy
  isPrivate        Boolean   @default(false)
  followRequests   FollowRequest[]
}

model Follow {
  id            String   @id @default(cuid())
  followerId    String
  followingId   String
  createdAt     DateTime @default(now())
  accepted      Boolean  @default(true) // Per account privati
  
  // Relazioni
  follower      User     @relation("Followers", fields: [followerId], references: [id], onDelete: Cascade)
  following     User     @relation("Following", fields: [followingId], references: [id], onDelete: Cascade)
  
  @@unique([followerId, followingId])
  @@index([followerId])
  @@index([followingId])
}

model FollowRequest {
  id           String   @id @default(cuid())
  requesterId  String
  targetId     String
  status       RequestStatus @default(PENDING)
  createdAt    DateTime @default(now())
  
  requester    User     @relation("FollowRequestsSent", fields: [requesterId], references: [id], onDelete: Cascade)
  target       User     @relation("FollowRequestsReceived", fields: [targetId], references: [id], onDelete: Cascade)
  
  @@unique([requesterId, targetId])
}

model Block {
  id          String   @id @default(cuid())
  blockerId   String
  blockedId   String
  createdAt   DateTime @default(now())
  reason      String?
  
  blocker     User     @relation("Blocking", fields: [blockerId], references: [id], onDelete: Cascade)
  blocked     User     @relation("BlockedBy", fields: [blockedId], references: [id], onDelete: Cascade)
  
  @@unique([blockerId, blockedId])
}

model SuggestedFollow {
  id          String   @id @default(cuid())
  userId      String
  suggestedId String
  score       Float    @default(0)
  reason      String?  // "mutual_friends", "similar_interests", etc.
  createdAt   DateTime @default(now())
  
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  suggested   User     @relation(fields: [suggestedId], references: [id], onDelete: Cascade)
  
  @@unique([userId, suggestedId])
  @@index([userId, score])
}

enum RequestStatus {
  PENDING
  ACCEPTED
  REJECTED
}
TypeScript Types
typescript
// types/follow.ts
export interface Follow {
  id: string;
  followerId: string;
  followingId: string;
  createdAt: Date;
  accepted: boolean;
}

export interface FollowRequest {
  id: string;
  requesterId: string;
  targetId: string;
  status: RequestStatus;
  createdAt: Date;
  requester?: User;
}

export interface Block {
  id: string;
  blockerId: string;
  blockedId: string;
  createdAt: Date;
  reason?: string;
}

export interface SuggestedFollow {
  id: string;
  userId: string;
  suggestedId: string;
  score: number;
  reason?: string;
  suggested?: User;
}

export interface FollowStats {
  followersCount: number;
  followingCount: number;
  mutualFollowsCount: number;
}

export interface UserWithFollowInfo extends User {
  followers: Follow[];
  following: Follow[];
  followersCount: number;
  followingCount: number;
  isFollowing?: boolean;
  isFollowedBy?: boolean;
  isBlocked?: boolean;
  isBlocking?: boolean;
}

export type FollowAction = 'follow' | 'unfollow' | 'accept' | 'reject' | 'block' | 'unblock';
Logica Follow/Unfollow
typescript
// app/api/follow/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';
import { z } from 'zod';

const followSchema = z.object({
  targetUserId: z.string().min(1),
  action: z.enum(['follow', 'unfollow', 'block', 'unblock']),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user?.id) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const body = await request.json();
    const { targetUserId, action } = followSchema.parse(body);
    const currentUserId = session.user.id;

    // Controlla se l'utente target esiste
    const targetUser = await prisma.user.findUnique({
      where: { id: targetUserId },
    });

    if (!targetUser) {
      return NextResponse.json({ error: 'User not found' }, { status: 404 });
    }

    // Controlla se è bloccato
    const isBlocked = await prisma.block.findFirst({
      where: {
        OR: [
          { blockerId: targetUserId, blockedId: currentUserId },
          { blockerId: currentUserId, blockedId: targetUserId },
        ],
      },
    });

    if (isBlocked) {
      return NextResponse.json({ error: 'Action not allowed' }, { status: 403 });
    }

    switch (action) {
      case 'follow':
        return await handleFollow(currentUserId, targetUserId, targetUser.isPrivate);
      case 'unfollow':
        return await handleUnfollow(currentUserId, targetUserId);
      case 'block':
        return await handleBlock(currentUserId, targetUserId);
      case 'unblock':
        return await handleUnblock(currentUserId, targetUserId);
      default:
        return NextResponse.json({ error: 'Invalid action' }, { status: 400 });
    }
  } catch (error) {
    console.error('Follow action error:', error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}

async function handleFollow(followerId: string, followingId: string, isPrivate: boolean) {
  // Controlla se esiste già un follow
  const existingFollow = await prisma.follow.findUnique({
    where: {
      followerId_followingId: {
        followerId,
        followingId,
      },
    },
  });

  if (existingFollow) {
    return NextResponse.json({ message: 'Already following' }, { status: 200 });
  }

  if (isPrivate) {
    // Per account privati, crea una richiesta di follow
    const followRequest = await prisma.followRequest.upsert({
      where: {
        requesterId_targetId: {
          requesterId: followerId,
          targetId: followingId,
        },
      },
      create: {
        requesterId: followerId,
        targetId: followingId,
        status: 'PENDING',
      },
      update: {
        status: 'PENDING',
        createdAt: new Date(),
      },
    });

    // Invia notifica
    await createNotification(followingId, 'FOLLOW_REQUEST', {
      requesterId: followerId,
      requestId: followRequest.id,
    });

    return NextResponse.json({ 
      message: 'Follow request sent',
      data: followRequest 
    });
  } else {
    // Per account pubblici, follow diretto
    const follow = await prisma.$transaction(async (tx) => {
      const follow = await tx.follow.create({
        data: {
          followerId,
          followingId,
          accepted: true,
        },
      });

      // Aggiorna i contatori
      await tx.user.update({
        where: { id: followerId },
        data: { followingCount: { increment: 1 } },
      });

      await tx.user.update({
        where: { id: followingId },
        data: { followersCount: { increment: 1 } },
      });

      // Invia notifica
      await createNotification(followingId, 'FOLLOW', {
        followerId,
      });

      return follow;
    });

    return NextResponse.json({ 
      message: 'Followed successfully',
      data: follow 
    });
  }
}

async function handleUnfollow(followerId: string, followingId: string) {
  const result = await prisma.$transaction(async (tx) => {
    // Elimina il follow
    const follow = await tx.follow.delete({
      where: {
        followerId_followingId: {
          followerId,
          followingId,
        },
      },
    });

    // Aggiorna i contatori
    await tx.user.update({
      where: { id: followerId },
      data: { followingCount: { decrement: 1 } },
    });

    await tx.user.update({
      where: { id: followingId },
      data: { followersCount: { decrement: 1 } },
    });

    // Elimina eventuali richieste pendenti
    await tx.followRequest.deleteMany({
      where: {
        requesterId: followerId,
        targetId: followingId,
      },
    });

    return follow;
  });

  return NextResponse.json({ 
    message: 'Unfollowed successfully',
    data: result 
  });
}

// Helper per creare notifiche
async function createNotification(userId: string, type: string, data: any) {
  await prisma.notification.create({
    data: {
      userId,
      type,
      data,
      read: false,
    },
  });
}
Lista Followers/Following
typescript
// app/api/users/[userId]/followers/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { getServerSession } from 'next-auth';

export async function GET(
  request: NextRequest,
  { params }: { params: { userId: string } }
) {
  try {
    const { userId } = params;
    const session = await getServerSession();
    const currentUserId = session?.user?.id;

    // Controlla privacy
    const user = await prisma.user.findUnique({
      where: { id: userId },
      select: { isPrivate: true },
    });

    if (!user) {
      return NextResponse.json({ error: 'User not found' }, { status: 404 });
    }

    // Se l'utente è privato e il visitatore non è follower, restituisci solo il conteggio
    if (user.isPrivate && currentUserId !== userId) {
      const isFollowing = currentUserId ? await prisma.follow.findUnique({
        where: {
          followerId_followingId: {
            followerId: currentUserId,
            followingId: userId,
          },
          accepted: true,
        },
      }) : false;

      if (!isFollowing) {
        const count = await prisma.follow.count({
          where: {
            followingId: userId,
            accepted: true,
          },
        });
        return NextResponse.json({ count, private: true });
      }
    }

    // Recupera followers con informazioni utente
    const followers = await prisma.follow.findMany({
      where: {
        followingId: userId,
        accepted: true,
      },
      include: {
        follower: {
          select: {
            id: true,
            name: true,
            username: true,
            image: true,
            followersCount: true,
            followingCount: true,
            isPrivate: true,
          },
        },
      },
      orderBy: { createdAt: 'desc' },
    });

    // Aggiungi informazioni mutual follows per l'utente corrente
    const followersWithMutualInfo = await Promise.all(
      followers.map(async (follow) => {
        let mutualFollows = 0;
        let isFollowing = false;

        if (currentUserId) {
          // Controlla se l'utente corrente segue questo follower
          isFollowing = !!(await prisma.follow.findUnique({
            where: {
              followerId_followingId: {
                followerId: currentUserId,
                followingId: follow.follower.id,
              },
            },
          }));

          // Calcola mutual follows
          mutualFollows = await prisma.follow.count({
            where: {
              followerId: follow.follower.id,
              following: {
                followers: {
                  some: {
                    followerId: currentUserId,
                  },
                },
              },
            },
          });
        }

        return {
          ...follow,
          follower: {
            ...follow.follower,
            isFollowing,
            mutualFollows,
          },
        };
      })
    );

    return NextResponse.json({
      followers: followersWithMutualInfo,
      count: followers.length,
    });
  } catch (error) {
    console.error('Error fetching followers:', error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}

// app/api/users/[userId]/following/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { userId: string } }
) {
  // Simile alla route followers, ma per following
  // Implementazione simile con where: { followerId: userId }
}
Follow Suggestions
typescript
// app/api/follow/suggestions/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';

export async function GET(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user?.id) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const currentUserId = session.user.id;
    const limit = parseInt(request.nextUrl.searchParams.get('limit') || '10');

    // Strategie di suggerimento multiple
    const suggestions = await getFollowSuggestions(currentUserId, limit);

    return NextResponse.json({ suggestions });
  } catch (error) {
    console.error('Error fetching follow suggestions:', error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}

async function getFollowSuggestions(userId: string, limit: number = 10) {
  // 1. Mutual follows (amici di amici)
  const mutualSuggestions = await prisma.$queryRaw`
    SELECT 
      u.id,
      u.name,
      u.username,
      u.image,
      u.bio,
      COUNT(DISTINCT mf.id) as mutual_count,
      'mutual_friends' as reason
    FROM "User" u
    INNER JOIN "Follow" f1 ON f1.followingId = u.id
    INNER JOIN "Follow" f2 ON f2.followerId = u.id
    LEFT JOIN "Follow" mf ON mf.followingId = f1.followerId 
      AND mf.followerId = ${userId}
    WHERE u.id != ${userId}
      AND u.id NOT IN (
        SELECT followingId FROM "Follow" WHERE followerId = ${userId}
        UNION
        SELECT blockedId FROM "Block" WHERE blockerId = ${userId}
        UNION
        SELECT blockerId FROM "Block" WHERE blockedId = ${userId}
      )
      AND u."isPrivate" = false
    GROUP BY u.id, u.name, u.username, u.image, u.bio
    HAVING COUNT(DISTINCT mf.id) > 0
    ORDER BY mutual_count DESC
    LIMIT ${Math.floor(limit / 2)}
  `;

  // 2. Utenti popolari che non segui
  const popularLimit = limit - (mutualSuggestions as any[]).length;
  let popularSuggestions = [];

  if (popularLimit > 0) {
    popularSuggestions = await prisma.user.findMany({
      where: {
        AND: [
          { id: { not: userId } },
          { isPrivate: false },
          {
            NOT: {
              OR: [
                { followers: { some: { followerId: userId } } },
                { blockingUsers: { some: { blockerId: userId } } },
                { blockedUsers: { some: { blockedId: userId } } },
              ],
            },
          },
        ],
      },
      select: {
        id: true,
        name: true,
        username: true,
        image: true,
        bio: true,
        followersCount: true,
        _count: {
          select: {
            posts: {
              where: { visibility: 'PUBLIC' },
            },
          },
        },
      },
      orderBy: {
        followersCount: 'desc',
      },
      take: popularLimit,
    });

    // Aggiungi reason
    popularSuggestions = popularSuggestions.map(user => ({
      ...user,
      reason: 'popular',
      mutual_count: 0,
    }));
  }

  // 3. Utenti con interessi simili (basato su post like)
  const interestLimit = limit - (mutualSuggestions as any[]).length - popularSuggestions.length;
  let interestSuggestions = [];

  if (interestLimit > 0) {
    interestSuggestions = await prisma.$queryRaw`
      SELECT 
        u.id,
        u.name,
        u.username,
        u.image,
        u.bio,
        COUNT(DISTINCT pl.postId) as common_interests,
        'similar_interests' as reason
      FROM "User" u
      INNER JOIN "PostLike" pl ON pl.userId = u.id
      WHERE u.id != ${userId}
        AND u."isPrivate" = false
        AND u.id NOT IN (
          SELECT followingId FROM "Follow" WHERE followerId = ${userId}
          UNION
          SELECT blockedId FROM "Block" WHERE blockerId = ${userId}
          UNION
          SELECT blockerId FROM "Block" WHERE blockedId = ${userId}
        )
        AND pl.postId IN (
          SELECT postId FROM "PostLike" WHERE userId = ${userId}
        )
      GROUP BY u.id, u.name, u.username, u.image, u.bio
      ORDER BY common_interests DESC
      LIMIT ${interestLimit}
    `;
  }

  // Combina e ordina i suggerimenti
  const allSuggestions = [
    ...(mutualSuggestions as any[]),
    ...popularSuggestions,
    ...(interestSuggestions as any[]),
  ];

  // Ordina per score (logica personalizzabile)
  return allSuggestions.sort((a, b) => {
    const scoreA = calculateSuggestionScore(a);
    const scoreB = calculateSuggestionScore(b);
    return scoreB - scoreA;
  }).slice(0, limit);
}

function calculateSuggestionScore(suggestion: any): number {
  let score = 0;
  
  if (suggestion.reason === 'mutual_friends') {
    score += suggestion.mutual_count * 10;
  }
  
  if (suggestion.reason === 'popular') {
    score += suggestion.followersCount * 0.1;
    score += suggestion._count?.posts * 0.5 || 0;
  }
  
  if (suggestion.reason === 'similar_interests') {
    score += suggestion.common_interests * 15;
  }
  
  return score;
}
Mutual Follows
typescript
// app/api/users/[userId]/mutual-follows/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';

export async function GET(
  request: NextRequest,
  { params }: { params: { userId: string } }
) {
  try {
    const { userId } = params;
    const withUserId = request.nextUrl.searchParams.get('with');
    
    if (!withUserId) {
      return NextResponse.json({ error: 'with parameter is required' }, { status: 400 });
    }

    // Trova utenti seguiti da entrambi
    const mutualFollows = await prisma.$queryRaw`
      SELECT u.*
      FROM "User" u
      INNER JOIN "Follow" f1 ON f1.followingId = u.id AND f1.followerId = ${userId}
      INNER JOIN "Follow" f2 ON f2.followingId = u.id AND f2.followerId = ${withUserId}
      WHERE f1.accepted = true AND f2.accepted = true
      ORDER BY u.followersCount DESC
      LIMIT 50
    `;

    const count = await prisma.$queryRaw`
      SELECT COUNT(*) as count
      FROM "Follow" f1
      INNER JOIN "Follow" f2 ON f2.followingId = f1.followingId
      WHERE f1.followerId = ${userId}
        AND f2.followerId = ${withUserId}
        AND f1.accepted = true
        AND f2.accepted = true
    `;

    return NextResponse.json({
      mutualFollows,
      count: count[0].count,
    });
  } catch (error) {
    console.error('Error fetching mutual follows:', error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}
Block Integration
typescript
// app/api/block/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';
import { z } from 'zod';

const blockSchema = z.object({
  targetUserId: z.string().min(1),
  reason: z.string().optional(),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user?.id) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const body = await request.json();
    const { targetUserId, reason } = blockSchema.parse(body);
    const currentUserId = session.user.id;

    // Controlla se esiste già un blocco
    const existingBlock = await prisma.block.findUnique({
      where: {
        blockerId_blockedId: {
          blockerId: currentUserId,
          blockedId: targetUserId,
        },
      },
    });

    if (existingBlock) {
      return NextResponse.json({ message: 'User already blocked' });
    }

    // Esegui il blocco in una transazione
    const result = await prisma.$transaction(async (tx) => {
      // Crea il blocco
      const block = await tx.block.create({
        data: {
          blockerId: currentUserId,
          blockedId: targetUserId,
          reason,
        },
      });

      // Elimina eventuali follow esistenti in entrambe le direzioni
      await tx.follow.deleteMany({
        where: {
          OR: [
            { followerId: currentUserId, followingId: targetUserId },
            { followerId: targetUserId, followingId: currentUserId },
          ],
        },
      });

      // Elimina richieste di follow
      await tx.followRequest.deleteMany({
        where: {
          OR: [
            { requesterId: currentUserId, targetId: targetUserId },
            { requesterId: targetUserId, targetId: currentUserId },
          ],
        },
      });

      // Aggiorna i contatori follow
      const followCounts = await tx.follow.findMany({
        where: {
          OR: [
            { followerId: currentUserId, followingId: targetUserId },
            { followerId: targetUserId, followingId: currentUserId },
          ],
        },
        select: {
          followerId: true,
          followingId: true,
        },
      });

      for (const follow of followCounts) {
        if (follow.followerId === currentUserId) {
          await tx.user.update({
            where: { id: currentUserId },
            data: { followingCount: { decrement: 1 } },
          });
          await tx.user.update({
            where: { id: targetUserId },
            data: { followersCount: { decrement: 1 } },
          });
        } else {
          await tx.user.update({
            where: { id: targetUserId },
            data: { followingCount: { decrement: 1 } },
          });
          await tx.user.update({
            where: { id: currentUserId },
            data: { followersCount: { decrement: 1 } },
          });
        }
      }

      return block;
    });

    return NextResponse.json({ 
      message: 'User blocked successfully',
      data: result 
    });
  } catch (error) {
    console.error('Block error:', error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}

// app/api/block/[userId]/route.ts per sbloccare
export async function DELETE(
  request: NextRequest,
  { params }: { params: { userId: string } }
) {
  try {
    const session = await getServerSession();
    if (!session?.user?.id) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const { userId: targetUserId } = params;
    const currentUserId = session.user.id;

    const block = await prisma.block.delete({
      where: {
        blockerId_blockedId: {
          blockerId: currentUserId,
          blockedId: targetUserId,
        },
      },
    });

    return NextResponse.json({ 
      message: 'User unblocked successfully',
      data: block 
    });
  } catch (error) {
    console.error('Unblock error:', error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}
§ LIKE SYSTEM
Modello Prisma per Like System
prisma
// schema.prisma
model Post {
  id          String   @id @default(cuid())
  // ... altri campi post
  
  // Relazioni Like
  likes       PostLike[]
  likeCount   Int      @default(0)
  
  // Analytics
  viewCount   Int      @default(0)
  shareCount  Int      @default(0)
  saveCount   Int      @default(0)
}

model PostLike {
  id        String   @id @default(cuid())
  postId    String
  userId    String
  createdAt DateTime @default(now())
  
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([postId, userId])
  @@index([postId])
  @@index([userId])
}

model Comment {
  id        String   @id @default(cuid())
  // ... altri campi commento
  
  // Like per commenti
  likes     CommentLike[]
  likeCount Int      @default(0)
}

model CommentLike {
  id        String   @id @default(cuid())
  commentId String
  userId    String
  createdAt DateTime @default(now())
  
  comment   Comment  @relation(fields: [commentId], references: [id], onDelete: Cascade)
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([commentId, userId])
}
TypeScript Types
typescript
// types/like.ts
export interface PostLike {
  id: string;
  postId: string;
  userId: string;
  createdAt: Date;
  user?: User;
}

export interface CommentLike {
  id: string;
  commentId: string;
  userId: string;
  createdAt: Date;
  user?: User;
}

export interface LikeStats {
  likeCount: number;
  userLiked: boolean;
  firstLikes: User[];
}

export interface LikeNotificationData {
  postId: string;
  postTitle?: string;
  likerId: string;
  likeId: string;
}
Like/Unlike Logic
typescript
// app/api/posts/[postId]/like/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';
import { z } from 'zod';

const likeSchema = z.object({
  action: z.enum(['like', 'unlike']),
});

export async function POST(
  request: NextRequest,
  { params }: { params: { postId: string } }
) {
  try {
    const session = await getServerSession();
    if (!session?.user?.id) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const { postId } = params;
    const body = await request.json();
    const { action } = likeSchema.parse(body);
    const userId = session.user.id;

    // Controlla se il post esiste e se è accessibile
    const post = await prisma.post.findUnique({
      where: { id: postId },
      select: {
        id: true,
        authorId: true,
        visibility: true,
        author: {
          select: {
            isPrivate: true,
            id: true,
          },
        },
      },
    });

    if (!post) {
      return NextResponse.json({ error: 'Post not found' }, { status: 404 });
    }

    // Controlla autorizzazione
    const canView = await checkPostVisibility(post, userId);
    if (!canView) {
      return NextResponse.json({ error: 'Not authorized' }, { status: 403 });
    }

    if (action === 'like') {
      return await handleLike(postId, userId, post.authorId);
    } else {
      return await handleUnlike(postId, userId);
    }
  } catch (error) {
    console.error('Like action error:', error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}

async function handleLike(postId: string, userId: string, authorId: string) {
  try {
    const result = await prisma.$transaction(async (tx) => {
      // Crea il like
      const like = await tx.postLike.create({
        data: {
          postId,
          userId,
        },
      });

      // Aggiorna il contatore
      await tx.post.update({
        where: { id: postId },
        data: { likeCount: { increment: 1 } },
      });

      // Invia notifica all'autore (se non è l'utente stesso)
      if (authorId !== userId) {
        await tx.notification.create({
          data: {
            userId: authorId,
            type: 'POST_LIKE',
            data: {
              postId,
              likerId: userId,
              likeId: like.id,
            },
            read: false,
          },
        });
      }

      return like;
    });

    return NextResponse.json({
      message: 'Post liked successfully',
      data: result,
      likeCount: await getUpdatedLikeCount(postId),
    });
  } catch (error: any) {
    if (error.code === 'P2002') {
      // Unique constraint violation - already liked
      return NextResponse.json({ 
        message: 'Post already liked',
        likeCount: await getUpdatedLikeCount(postId),
      });
    }
    throw error;
  }
}

async function handleUnlike(postId: string, userId: string) {
  const result = await prisma.$transaction(async (tx) => {
    // Elimina il like
    const like = await tx.postLike.delete({
      where: {
        postId_userId: {
          postId,
          userId,
        },
      },
    });

    // Aggiorna il contatore
    await tx.post.update({
      where: { id: postId },
      data: { likeCount: { decrement: 1 } },
    });

    // Elimina notifica correlata
    await tx.notification.deleteMany({
      where: {
        type: 'POST_LIKE',
        userId: like.userId,
        data: {
          path: ['likeId'],
          equals: like.id,
        },
      },
    });

    return like;
  });

  return NextResponse.json({
    message: 'Post unliked successfully',
    data: result,
    likeCount: await getUpdatedLikeCount(postId),
  });
}

async function getUpdatedLikeCount(postId: string) {
  const post = await prisma.post.findUnique({
    where: { id: postId },
    select: { likeCount: true },
  });
  return post?.likeCount || 0;
}

async function checkPostVisibility(post: any, userId: string): Promise<boolean> {
  if (post.visibility === 'PUBLIC') return true;
  if (!userId) return false;
  if (post.authorId === userId) return true;
  if (post.visibility === 'PRIVATE') return false;
  
  if (post.visibility === 'FOLLOWERS') {
    if (post.author.isPrivate) {
      // Per account privati, controlla se il follow è accettato
      const follow = await prisma.follow.findUnique({
        where: {
          followerId_followingId: {
            followerId: userId,
            followingId: post.authorId,
          },
          accepted: true,
        },
      });
      return !!follow;
    } else {
      // Per account pubblici, controlla solo se segue
      const follow = await prisma.follow.findUnique({
        where: {
          followerId_followingId: {
            followerId: userId,
            followingId: post.authorId,
          },
        },
      });
      return !!follow;
    }
  }
  
  return false;
}
Who Liked a Post
typescript
// app/api/posts/[postId]/likes/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { getServerSession } from 'next-auth';

export async function GET(
  request: NextRequest,
  { params }: { params: { postId: string } }
) {
  try {
    const { postId } = params;
    const session = await getServerSession();
    const currentUserId = session?.user?.id;
    
    const cursor = request.nextUrl.searchParams.get('cursor');
    const limit = parseInt(request.nextUrl.searchParams.get('limit') || '20');

    // Controlla se il post esiste
    const post = await prisma.post.findUnique({
      where: { id: postId },
      select: { authorId: true, visibility: true },
    });

    if (!post) {
      return NextResponse.json({ error: 'Post not found' }, { status: 404 });
    }

    // Controlla autorizzazione
    const canView = await checkPostVisibility(post, currentUserId || '');
    if (!canView) {
      return NextResponse.json({ error: 'Not authorized' }, { status: 403 });
    }

    // Recupera likes con informazioni utente
    const likes = await prisma.postLike.findMany({
      where: { postId },
      include: {
        user: {
          select: {
            id: true,
            name: true,
            username: true,
            image: true,
            bio: true,
            followersCount: true,
            isPrivate: true,
          },
        },
      },
      orderBy: { createdAt: 'desc' },
      cursor: cursor ? { id: cursor } : undefined,
      skip: cursor ? 1 : 0,
      take: limit,
    });

    // Aggiungi informazioni follow per l'utente corrente
    const likesWithFollowInfo = await Promise.all(
      likes.map(async (like) => {
        let isFollowing = false;
        let isFollowedBy = false;
        let mutualFollows = 0;

        if (currentUserId) {
          // Controlla relazioni follow
          const [following, followedBy] = await Promise.all([
            prisma.follow.findUnique({
              where: {
                followerId_followingId: {
                  followerId: currentUserId,
                  followingId: like.user.id,
                },
              },
            }),
            prisma.follow.findUnique({
              where: {
                followerId_followingId: {
                  followerId: like.user.id,
                  followingId: currentUserId,
                },
              },
            }),
          ]);

          isFollowing = !!following;
          isFollowedBy = !!followedBy;

          // Calcola mutual follows
          if (isFollowing || isFollowedBy) {
            mutualFollows = await prisma.follow.count({
              where: {
                followerId: like.user.id,
                following: {
                  followers: {
                    some: {
                      followerId: currentUserId,
                    },
                  },
                },
              },
            });
          }
        }

        return {
          ...like,
          user: {
            ...