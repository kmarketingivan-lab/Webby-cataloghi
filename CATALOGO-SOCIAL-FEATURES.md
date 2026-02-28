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

## § ADVANCED PATTERNS: SOCIAL FEATURES

### Server Actions con Validazione

```typescript
// app/actions/social-features.ts
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


### SOCIAL FEATURES - Utility Helper #550

```typescript
// lib/utils/social-features-helper-550.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #474

```typescript
// lib/utils/social-features-helper-474.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #277

```typescript
// lib/utils/social-features-helper-277.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #806

```typescript
// lib/utils/social-features-helper-806.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #644

```typescript
// lib/utils/social-features-helper-644.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #383

```typescript
// lib/utils/social-features-helper-383.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #0

```typescript
// lib/utils/social-features-helper-0.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #282

```typescript
// lib/utils/social-features-helper-282.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #762

```typescript
// lib/utils/social-features-helper-762.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #556

```typescript
// lib/utils/social-features-helper-556.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #349

```typescript
// lib/utils/social-features-helper-349.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #581

```typescript
// lib/utils/social-features-helper-581.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #558

```typescript
// lib/utils/social-features-helper-558.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #982

```typescript
// lib/utils/social-features-helper-982.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #758

```typescript
// lib/utils/social-features-helper-758.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #18

```typescript
// lib/utils/social-features-helper-18.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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

  getConfig(): Readonly<SOCIALFEATURESConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### SOCIAL FEATURES - Utility Helper #704

```typescript
// lib/utils/social-features-helper-704.ts
import { z } from "zod";

interface SOCIALFEATURESConfig {
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

export class SOCIALFEATURESProcessor<TInput, TOutput> {
  private config: SOCIALFEATURESConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<SOCIALFEATURESConfig> = {}) {
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
   